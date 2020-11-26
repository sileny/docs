# vuex 源码全解

Vuex版本为 `3.0.1`

此处省略Vuex的一些官方定义，官方定义，请参考<a href="https://vuex.vuejs.org/zh/" target="_blank">官方文档</a>

Vuex 采用 `单向数据流` 理念，遵循 `单例模式` 设计，使状态数据全局共享，任意组件都可以修改。

### 单项数据流

初始的 `state` 展示到 `view` 上，`view` 上的变更需要 `action` 提交变更，`mutation` 负责数据的变更，变更后的数据将与 `view` 保持一致。

### 单例模式

如何利用单例设计模式实现的，请参考 `Vue.use` 源码
```
  Vue.use = function (plugin: Function | Object) {
    // 获取插件
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    if (installedPlugins.indexOf(plugin) > -1) { // 已经存在
      return this // 返回实例
    }

    // additional parameters
    const args = toArray(arguments, 1)
    args.unshift(this) // 调整顺序
    if (typeof plugin.install === 'function') { // plugin为对象类型，install为function类型
      plugin.install.apply(plugin, args) // 则执行该方法
    } else if (typeof plugin === 'function') { // 为function类型
      plugin.apply(null, args) // 执行该方法
    }
    installedPlugins.push(plugin) // 缓存插件
    return this // 返回当前实例
  }
```


# 对外暴露的核心对象

这里先介绍 `Vuex` 大概是啥样的
```javascript
// 自执行函数
// global 指向 this, factory 指向了 function() { /* coding */ }
(function (global, factory) {
    // common js 运行环境
	typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() :
    // cmd规范
	typeof define === 'function' && define.amd ? define(factory) :
    // 浏览器运行环境
	(global.Vuex = factory());
}(this, (function () { 'use strict';

// ...省略的代码会在下文进行介绍

// Vuex最终是这样的
var index = {
  Store: Store, // 构造器
  install: install, // Vue.use需要一个install方法
  version: '3.0.1',
  mapState: mapState, // 状态(当前的数据)
  mapMutations: mapMutations, // 赋值操作(=)
  mapGetters: mapGetters, // 取值(get)
  mapActions: mapActions, // 方法(提交数据变更)
  createNamespacedHelpers: createNamespacedHelpers // 对外暴露一个对象：{ mapState, mapGetters, mapMutations, mapActions }，和上面紧挨着的四行没有太大的不同
};

// 暴露对象属性方法
return index;

})));
```

# Vuex内部代码块

## 辅助方法

- [applyMixin (Vue)](#applyMixin)
- devtoolPlugin (store)
- [forEachValue (obj, fn)](#forEachValue)
- isObject (obj)
- isPromise (val)
- assert (condition, msg)
- update (path, targetModule, newModule) 更新模块（在 `Store` 的`hotUpdate`方法内有引用）
- assertRawModule (path, rawModule) 模块断言，判断类型是否符合
- [makeAssertionMessage (path, key, type, value, expected)](#断言消息)
- [genericSubscribe (fn, subs)](#subscribe)
- resetStore (store, hot) 重置状态
- [resetStoreVM (store, state, hot)](#resetStoreVM)
- [installModule (store, rootState, path, module, hot)](#installModule)
- [makeLocalContext (store, namespace, path)](#makeLocalContext)
- makeLocalGetters (store, namespace)
- [registerMutation (store, type, handler, local)](#registerMutation)
- [registerAction (store, type, handler, local)](#registerAction)
- [registerGetter (store, type, rawGetter, local)](#registerGetter)
- [enableStrictMode (store)](#strict)
- getNestedState (state, path) 获取嵌套的 `state`
- unifyObjectStyle (type, payload, options)

  `unifyObjectStyle` 定义 `dispatch` 和 `commit` 的数据格式
  ```javascript
  function unifyObjectStyle (type, payload, options) {
    if (isObject(type) && type.type) {
      options = payload;
      payload = type;
      type = type.type;
    }
  
    {
      assert(typeof type === 'string', ("Expects string as the type, but found " + (typeof type) + "."));
    }
  
    // 数据格式
    return { type: type, payload: payload, options: options }
  }
  ```

- install (_Vue)
- var mapState = normalizeNamespace(function (namespace, states) {})
- var mapMutations = normalizeNamespace(function (namespace, mutations) {})
- var mapGetters = normalizeNamespace(function (namespace, getters) {})
- var mapActions = normalizeNamespace(function (namespace, actions) {})
- var createNamespacedHelpers = function (namespace)
- normalizeMap(map)
- normalizeNamespace(fn)
- getModuleByNamespace(store, helper, namespace)



## 按模块划分

- [Module](#Module) 处理 `state`、子模块(`_children`)、原始的模块数据(`_rawModule`)
- [ModuleCollection](#ModuleCollection) 收集模块信息
- [Store](#Store)


# StoreOptions&lt;S&gt;

`StoreOptions` 是这样的
```
export interface StoreOptions<S> {
  state?: S;
  getters?: GetterTree<S, S>;
  actions?: ActionTree<S, S>;
  mutations?: MutationTree<S>;
  modules?: ModuleTree<S>;
  plugins?: Plugin<S>[];
  strict?: boolean;
}g
```

# demo

实例一个 `store`
```javascript
// store.js
import Vue from 'vue';
import Vuex from 'vuex';
import getters from './getters';
import cover from './modules/cover';
import PRODUCT_NEW from '@/constants/product-new';
import { getStyleId, setStyleId } from '@/utils/utils';

Vue.use(Vuex);

const store = new Vuex.Store({
  modules: { // 子模块
    cover
  },
  getters,
  state: {
    PRODUCT_NEW,
    styleId: parseInt(getStyleId(), 10)
  },
  mutations: {
    SET_STYLE_ID(state, styleId) {
      state.styleId = styleId;
    }
  },
  actions: {
    SetStyleId({ commit }, styleId) {
      setStyleId(styleId);
      commit('SET_STYLE_ID', styleId);
    }
  }
});

export default store;
```

将 `store` 注入到 `Vue` 实例上

```javascript
import store from './store'

new Vue({
  el: '#app',
  store
})
```

下面是分步解析运行过程

## Vue.use(Vuex)

`Vue.use(Vuex)` 会自动找到 `Vuex` 的 `install` 方法，向 `Vue` 实例注入 `Vuex` 插件，如果已经注册过，将不再注册。

`install` 方法的源码如下，
```javascript
function install (_Vue) {
  if (Vue && _Vue === Vue) { // 已经存在 Vue 变量
    {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      );
    }
    return
  }
  Vue = _Vue; // 给全局变量赋值
  applyMixin(Vue);
}
```
该方法在内部维护一个全局变量 `Vue`，如果 `Vue` 变量已经存在，则表明已经注册到 `Vue实例` 上，给出错误提示

## applyMixin

`applyMixin` 方法在内部做了版本兼容处理，在 `2` 以上版本里，直接将 `vuexInit` 方法混入到 `Vue实例` 上的钩子函数里。
这样，就可以在组件执行生命周期函数(`beforeCreate`)的之后，通过 `this.` 拿到 `$store`。

```javascript
var applyMixin = function (Vue) {
  var version = Number(Vue.version.split('.')[0]);

  if (version >= 2) {
    // 将 `vuexInit` 方法混入到钩子函数里
    Vue.mixin({ beforeCreate: vuexInit });
  } else {
    // 兼容处理不做解释说明
    // override init and inject vuex init procedure
    // for 1.x backwards compatibility.
    var _init = Vue.prototype._init;
    Vue.prototype._init = function (options) {
      if ( options === void 0 ) options = {};

      options.init = options.init
        ? [vuexInit].concat(options.init)
        : vuexInit;
      _init.call(this, options);
    };
  }

  /**
   * Vuex init hook, injected into each instances init hooks list.
   */

  function vuexInit () {
    // 从Vue实例上获取属性
    var options = this.$options;
    // store injection
    if (options.store) {
      // 如果当前Vue实例上有类型为function的store，则执行该function
      //    将函数执行的结果赋值给当前Vue实例的$store属性上
      // 否则，将options的sotre直接赋值给$store
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store;
    } else if (options.parent && options.parent.$store) { // 父节点和父节点的$store存在
      this.$store = options.parent.$store; // 则将父节点的$store赋值给当前Vue实例
    }
  }
};
```

>生命周期函数的执行标志：`callHook(vm: Component, hook: String)`，该生命周期方法在 `Vue.prototype._init` 内部执行，
`callHook(vm, 'beforeCreate')`，具体的`callHook` 方法的逻辑可以参考 [instance](./../vue/instance.md)

## 实例化Store对象

[StoreOptions&lt;S&gt;](#StoreOptionss) 包含的可选参数有：
- `state` 数据状态
- `getters` getter数据
- `actions` 提交变更操作
- `mutations` 执行数据变更
- `modules` 子模块
- `plugins` 插件
- `strict` 严格模式

创建 `Store` 实例的方式 `new Vuex.Store(StoreOptions<S>)`，`Store` 源码以及解释如下
```javascript
var Store = function Store (options) {
  var this$1 = this; // 当前Store实例
  if ( options === void 0 ) options = {}; // 设置默认值

  // Auto install if it is not done yet and `window` has `Vue`.
  // To allow users to avoid auto-installation in some cases,
  // this code should be placed here. See #731
  if (!Vue && typeof window !== 'undefined' && window.Vue) {
    install(window.Vue);
  }

  {
    assert(Vue, "must call Vue.use(Vuex) before creating a store instance.");
    assert(typeof Promise !== 'undefined', "vuex requires a Promise polyfill in this browser.");
    assert(this instanceof Store, "Store must be called with the new operator.");
  }

  // 获取插件
  var plugins = options.plugins; if ( plugins === void 0 ) plugins = [];
  // 使 Vuex store 进入严格模式，在严格模式下，任何 mutation 处理函数以外修改 Vuex state 都会抛出错误
  var strict = options.strict; if ( strict === void 0 ) strict = false;

  // 获取状态，不存在该参数，那么，则赋值为空对象
  var state = options.state; if ( state === void 0 ) state = {};
  // 如果state为函数类型
  if (typeof state === 'function') {
    // 则获取执行的结果，如果执行的结果为空，则赋值为空对象
    state = state() || {};
  }

  // store internal state内部状态
  this._committing = false;
  this._actions = Object.create(null);
  this._actionSubscribers = [];
  this._mutations = Object.create(null);
  // 缓存getters
  this._wrappedGetters = Object.create(null);
  // 收集模块 -- 先注册根模块，然后，注册子模块（查看_modules和register）
  this._modules = new ModuleCollection(options);
  this._modulesNamespaceMap = Object.create(null);
  this._subscribers = []; // 收集订阅者
  this._watcherVM = new Vue(); // Vue实例

  // bind commit and dispatch to self
  var store = this;
  var ref = this;
  var dispatch = ref.dispatch;
  var commit = ref.commit;
  this.dispatch = function boundDispatch (type, payload) {
    return dispatch.call(store, type, payload)
  };
  this.commit = function boundCommit (type, payload, options) {
    return commit.call(store, type, payload, options)
  };

  // strict mode
  this.strict = strict;

  // init root module.
  // this also recursively registers all sub-modules
  // and collects all module getters inside this._wrappedGetters
  installModule(this, state, [], this._modules.root);

  // initialize the store vm, which is responsible for the reactivity
  // (also registers _wrappedGetters as computed properties)
  resetStoreVM(this, state);

  // apply plugins调用插件
  plugins.forEach(function (plugin) { return plugin(this$1); });

  if (Vue.config.devtools) {
    devtoolPlugin(this);
  }
};
```

### plugins

需要处理的业务可以放到这里，在 `plugins.forEach(function (plugin) { return plugin(this$1); });` 会被执行，执行时传入了 `this$1`，
`this$1` 指向了 `Store` 实例，也就意味着，在这里，你可以为当前 `Store` 实例扩展属性。

### strict

```javascript
if (store.strict) { // 严格模式
  enableStrictMode(store);
}

function enableStrictMode (store) {
  // 监听state
  store._vm.$watch(function () { return this._data.$$state }, function () {
    {
      // 如果为false，则会报错
      assert(store._committing, "Do not mutate vuex store state outside mutation handlers.");
    }
  }, { deep: true, sync: true });
}

function assert (condition, msg) {
  // condition为false将会报错
  if (!condition) { throw new Error(("[vuex] " + msg)) }
}
```
严格模式下，会深度监听 `state`，如果不是是使用 `commit` 提交的，那么，会报错

### _committing

`_committing` 标记来自 `commit` 的变更

这里采用了 `备忘录` 设计模式

如果 `_committing` 是被 `_withCommit` 修改的，当前实例的 `_committing` 会被修改为 `true`，
之后再还原为 `false`，不还原，严格模式下，下次再次提交状态修改的时候会报错。
```javascript
this._committing = false; // 初始默认为false

Store.prototype._withCommit = function _withCommit (fn) {
  var committing = this._committing;
  this._committing = true;
  fn();
  this._committing = committing;
};
```

### _modules

`_modules` 负责收集模块

从 `StoreOptions<S>` 里收集模块
```javascript
// options 是 StoreOptions<S> 类型
this._modules = new ModuleCollection(options);

var ModuleCollection = function ModuleCollection (rawRootModule) {
  // register root module (Vuex.Store options)
  // 注册根模块
  this.register([], rawRootModule, false);
};
```
先注册根模块

`register` 方法

1. 内部先进行模块断言，判断 `StoreOptions<S>` 的 `actions`、`getters`、`modules`、`mutations`、`state` 等的类型是否满足要求，
2. 保留原始的模块信息，获取state
3. 如果第一次注册，则默认为根模块
4. 不是第一次注册，则添加子模块
5. 存在子 `modules`，则按照 `runtime` 的状态继续递归注册模块(子模块)
```javascript
ModuleCollection.prototype.register = function register (path, rawModule, runtime) {
    var this$1 = this;
    if ( runtime === void 0 ) runtime = true;

  {// 进行模块断言（断言模块的类型是否满足要求）
    assertRawModule(path, rawModule);
  }

  // 保存原始模块信息，获取state
  var newModule = new Module(rawModule, runtime);
  // 根模块
  if (path.length === 0) {
    this.root = newModule;
  } else {
    // 添加子模块
    var parent = this.get(path.slice(0, -1));
    parent.addChild(path[path.length - 1], newModule);
  }

  // register nested modules
  // 注册.modules的模块
  if (rawModule.modules) {
    forEachValue(rawModule.modules, function (rawChildModule, key) {
      this$1.register(path.concat(key), rawChildModule, runtime);
    });
  }
};

var Module = function Module (rawModule, runtime) {
  this.runtime = runtime; // 运行时
  this._children = Object.create(null); // 保存子模块
  this._rawModule = rawModule; // 保留原始的模块信息 StoreOptions<S>
  var rawState = rawModule.state; // 获取state
  // state可以为function类型，如果是function类型，则得到函数执行的结果，否则为对象类型
  this.state = (typeof rawState === 'function' ? rawState() : rawState) || {};
};
```

### _watcherVM

`Store` 的 `watch` 借助了 `Vue` 的 `$watch` 来实现
```javascript
Store.prototype.watch = function watch (getter, cb, options) {
    var this$1 = this;

  {
    assert(typeof getter === 'function', "store.watch only accepts a function.");
  }
  return this._watcherVM.$watch(function () { return getter(this$1.state, this$1.getters); }, cb, options)
};
```

### installModule

为所有的 `module` 安装 `dispatch`、`commit`、`getters`、`state` 等属性方法，
如果存在[子模块](#StoreOptionss)，递归安装 `dispatch`、`commit`、`getters`、`state` 等属性方法。

  - 子模块
  ```javascript
  const store = new Vuex.Store({
    modules: { // modules就是子模块
      cover
    }
    // ....
  });
  ```

下面是 `installModule` 源码
```javascript
function installModule (store, rootState, path, module, hot) {
  // 根节点
  var isRoot = !path.length;
  // 从注册的模块里获取命名空间
  var namespace = store._modules.getNamespace(path);

  // register in namespace map
  if (module.namespaced) {
    store._modulesNamespaceMap[namespace] = module;
  }

  // set state
  // 如果是子模块
  if (!isRoot && !hot) {
    var parentState = getNestedState(rootState, path.slice(0, -1));
    var moduleName = path[path.length - 1];
    store._withCommit(function () {
      // 为子模块添加state
      Vue.set(parentState, moduleName, module.state);
    });
  }

  // 生成本地的 `dispatch, commit, getters, state`
  var local = module.context = makeLocalContext(store, namespace, path);

  // 安装 mutation(类型为function)
  module.forEachMutation(function (mutation, key) {
    var namespacedType = namespace + key;
    registerMutation(store, namespacedType, mutation, local);
  });

  // 安装 action(类型为function)
  module.forEachAction(function (action, key) {
    var type = action.root ? key : namespace + key;
    var handler = action.handler || action;
    registerAction(store, type, handler, local);
  });

  // 安装 getter(类型为function)，其中getter会被缓存起来，功能和computed类似
  module.forEachGetter(function (getter, key) {
    var namespacedType = namespace + key;
    registerGetter(store, namespacedType, getter, local);
  });

  module.forEachChild(function (child, key) {
    // 安装子模块
    installModule(store, rootState, path.concat(key), child, hot);
  });
}
```

### registerMutation

注册 `mutation`

源码如下，
```javascript
function registerMutation (store, type, handler, local) {
  var entry = store._mutations[type] || (store._mutations[type] = []);
  entry.push(function wrappedMutationHandler (payload) {
    handler.call(store, local.state, payload);
  });
}
```
会将案例里的 `SET_STYLE_ID` 丢到 `entry` 数组里。


`handler` 会在运行时指向对应的方法。

对应的方法是 `StoreOptions.mutations` 里的被调用的方法。

如果，执行的是 `this.$store.dispatch('SetStyleId', 99)`，会先经过 `actions` 里的 `SetStyleId` 方法，该方法内部提交了 `commit('SET_STYLE_ID', styleId)` 的修改，则 `99` 会赋值给 `styleId`，然后，调用 `mutations` 里的 `SET_STYLE_ID` 方法，`SET_STYLE_ID` 方法在内部修改 `state` 的 `styleId`，`Vue` 内部实现了响应式编程，因此，视图里的数据也会刷新。

### registerAction

将 `actions` 里的函数以 `方法名` 为 `key` 的形式存储到 `entry`，在 `dispatch` 的时候执行该函数
```javascript
function registerAction (store, type, handler, local) {
  var entry = store._actions[type] || (store._actions[type] = []);
  entry.push(function wrappedActionHandler (payload, cb) {
    // 执行hanlder
    var res = handler.call(store, {
      dispatch: local.dispatch,
      commit: local.commit,
      getters: local.getters,
      state: local.state,
      rootGetters: store.getters,
      rootState: store.state
    }, payload, cb);
    // 返回Prommise类型的res
    if (!isPromise(res)) {
      res = Promise.resolve(res);
    }
    if (store._devtoolHook) {
      return res.catch(function (err) {
        store._devtoolHook.emit('vuex:error', err);
        throw err
      })
    } else {
      return res
    }
  });
}
```

当 `this.$store.dispatch('SetStyleId', 123)` 的时候，会找到 `SetStyleId` 对应的函数，也就是 `registerAction` 内部的 `handler`，

`handler` 就是 `actions` 里的方法
```javascript
actions: {
  SetStyleId({ commit }, styleId) {
    setStyleId(styleId);
    commit('SET_STYLE_ID', styleId);
  }
}
```
然后，`commit('SET_STYLE_ID', styleId)` 会找到 `mutations` 里 `SET_STYLE_ID` 对应的方法，该方法在内部执行数据变更。


### registerGetter

该方法会将 `store` 和 `local` 里的 `state` 和 `getter` 缓存到 `_wrappedGetters` 里
```javascript
function registerGetter (store, type, rawGetter, local) {
  if (store._wrappedGetters[type]) {
    {
      console.error(("[vuex] duplicate getter key: " + type));
    }
    return
  }
  store._wrappedGetters[type] = function wrappedGetter (store) {
    return rawGetter(
      local.state, // local state
      local.getters, // local getters
      store.state, // root state
      store.getters // root getters
    )
  };
}
```

`_wrappedGetters` 缓存的数据怎么拿到呢？

往下看

1. 存到 `getters` 里
```javascript
// getters.js
export default {
  COVER: state => state.cover.COVER,
  COVER_TIME: state => state.cover.COVER_TIME,
  COVER_COUNT: state => state.cover.COVER_COUNT,
  PRODUCT_NEW: state => state.PRODUCT_NEW,
  styleId: state => state.styleId
};
```

2. 通过 `computed` 得到

```javascript
import { mapGetters } from 'vuex';

export default {
  name: '组件名',
  computed: {
    // ...
    ...mapGetters(['styleId'])
  }
}
```

3、看下 `mapGetters` 的原理，在[这里](#mapGetters)


这只是实现了方法缓存，没有被访问（在页面上展示或者在业务里被访问）

如何访问数据，往下看

4、访问数据

4.1、直接在页面上访问
```
<div>{{ styleId }}</div>
```

4.2、在业务里访问
```
this.styleId
```

### makeLocalContext

生成本地的 `dispatch, commit, getters, state`
```javascript
/**
 * make localized dispatch, commit, getters and state
 * if there is no namespace, just use root ones
 */
function makeLocalContext (store, namespace, path) {
  var noNamespace = namespace === '';

  var local = {
    dispatch: noNamespace ? store.dispatch : function (_type, _payload, _options) {
      var args = unifyObjectStyle(_type, _payload, _options);
      var payload = args.payload;
      var options = args.options;
      var type = args.type;

      if (!options || !options.root) {
        type = namespace + type;
        if ("development" !== 'production' && !store._actions[type]) {
          console.error(("[vuex] unknown local action type: " + (args.type) + ", global type: " + type));
          return
        }
      }

      // 最终还是调用了 Store 原型上的 dispatch 方法
      return store.dispatch(type, payload)
    },

    commit: noNamespace ? store.commit : function (_type, _payload, _options) {
      var args = unifyObjectStyle(_type, _payload, _options);
      var payload = args.payload;
      var options = args.options;
      var type = args.type;

      if (!options || !options.root) {
        type = namespace + type;
        if ("development" !== 'production' && !store._mutations[type]) {
          console.error(("[vuex] unknown local mutation type: " + (args.type) + ", global type: " + type));
          return
        }
      }

      store.commit(type, payload, options);
    }
  };

  // getters and state object must be gotten lazily
  // because they will be changed by vm update
  Object.defineProperties(local, {
    getters: {
      get: noNamespace
        ? function () { return store.getters; }
        : function () { return makeLocalGetters(store, namespace); }
    },
    state: {
      get: function () { return getNestedState(store.state, path); }
    }
  });

  return local
}
```


### resetStoreVM

```javascript
function resetStoreVM (store, state, hot) {
  var oldVm = store._vm;

  // bind store public getters
  // 还原 getter
  store.getters = {};
  // 获取缓存的getter
  var wrappedGetters = store._wrappedGetters;
  // copy属性
  var computed = {};
  forEachValue(wrappedGetters, function (fn, key) {
    // use computed to leverage its lazy-caching mechanism
    computed[key] = function () { return fn(store); };
    Object.defineProperty(store.getters, key, {
      get: function () { return store._vm[key]; },
      enumerable: true // for local getters
    });
  });

  // use a Vue instance to store the state tree
  // suppress warnings just in case the user has added
  // some funky global mixins
  var silent = Vue.config.silent;
  Vue.config.silent = true;
  store._vm = new Vue({
    data: {
      // 将当前Store实例的state给了Vue的$$state属性
      // 注意，这里的data不是function，所以，是全局共享的数据，一处修改，处处生效
      $$state: state
    },
    computed: computed // 将copy的属性赋值给Vue的computed
  });
  Vue.config.silent = silent;

  // enable strict mode for new vm
  if (store.strict) { // 此处的解释往上看
    enableStrictMode(store);
  }

  if (oldVm) {
    if (hot) {
      // dispatch changes in all subscribed watchers
      // to force getter re-evaluation for hot reloading.
      // 销毁store._vm的数据
      store._withCommit(function () {
        oldVm._data.$$state = null;
      });
    }
    // 销毁store._vm
    Vue.nextTick(function () { return oldVm.$destroy(); });
  }
}
```

至此，一个 `Store` 的实例算是完成了。

上面的读完，可以理解以下内容：

1. state
2. getters
3. modules
4. plugins
5. strict

下面看看如何进行数据的变更操作

# Vuex的数据变更

[数据流](#单项数据流)

## dispatch

`$store.dispatch` 触发数据变更

先看一段代码，理解是如何变更数据
```javascript
const setId = () => this.$store.dispatch('SetStyleId', 123);
setTimeout(setId, 3000);

// 变更完数据后想干点啥
setId().then(() => console.log('做点可爱的事情吧....'));

// 多了一个参数，但是没啥用，知道用处的请告诉我下，谢谢
setTimeout(() => this.$store.dispatch('SetId', 123, {
  root: true // false
}), 5000);
```

为何如此神奇？

`Dispatch` 定义了可选的参数，返回值都为 `Promise` 类型
```javascript
interface Dispatch {
  (type: string, payload?: any, options?: DispatchOptions): Promise<any>;
  <P extends Payload>(payloadWithType: P, options?: DispatchOptions): Promise<any>;
}
```

`dispatch` 的实现有两处：

1. 在 `Store` 的原型上
```javascript
Store.prototype.dispatch = function dispatch (_type, _payload) {
    var this$1 = this;

  // check object-style dispatch
  var ref = unifyObjectStyle(_type, _payload);
    var type = ref.type;
    var payload = ref.payload;

  var action = { type: type, payload: payload };
  var entry = this._actions[type];
  if (!entry) {
    {
      console.error(("[vuex] unknown action type: " + type));
    }
    return
  }

  // 告诉订阅者提交状态变更
  this._actionSubscribers.forEach(function (sub) { return sub(action, this$1.state); });

  // 返回 Promise 对象
  return entry.length > 1
    ? Promise.all(entry.map(function (handler) { return handler(payload); }))
    : entry[0](payload)
};
```

2. 在本地属性上

具体实现在 `makeLocalContext` 内部，下面只是其中的部分代码。
```javascript
dispatch: noNamespace ? store.dispatch : function (_type, _payload, _options) {
  // ...
  return store.dispatch(type, payload)
}
```

## action

先看下 [registerAction](#registerAction) 源码，该方法实现了 `action` 的注册，
```javascript
var res = handler.call(store, { // 回调返回的参数
  dispatch: local.dispatch,
  commit: local.commit,
  getters: local.getters,
  state: local.state,
  rootGetters: store.getters,
  rootState: store.state
}, payload, cb);
```
`actions` 里的每一个函数在 `dispatch` 的时候都当作 `handler`，`dispatch('SetStyleId', 123)` 会在 `actions` 里找到与 `SetStyleId` 对应的 `function`，然后，以 `Promise.all` 返回一个 `Promise` 对象。

`commit('SET_STYLE_ID', styleId)` 会找到 `mutations` 里与 `SET_STYLE_ID` 对应的方法，并且标记数据变更来自 `commit`，修改并重置 `_committing` 的值，然后，执行 `SET_STYLE_ID` 方法。

`SET_STYLE_ID` 方法的执行，触发了响应式机制，实现视图与数据保持一致。

## commit

提交数据和变更
```javascript
Store.prototype.commit = function commit (_type, _payload, _options) {
    var this$1 = this;

  // check object-style commit
  var ref = unifyObjectStyle(_type, _payload, _options);
    var type = ref.type;
    var payload = ref.payload;
    var options = ref.options;

  var mutation = { type: type, payload: payload };
  // 判断提交的类型是否存在
  var entry = this._mutations[type];
  if (!entry) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(("[vuex] unknown mutation type: " + type));
    }
    return
  }
  // 存在，则遍历数组执行数据变更方法
  this._withCommit(function () {
    entry.forEach(function commitIterator (handler) {
      handler(payload);
    });
  });
  this._subscribers.forEach(function (sub) { return sub(mutation, this$1.state); });

  if (
    process.env.NODE_ENV !== 'production' &&
    options && options.silent
  ) {
    console.warn(
      "[vuex] mutation type: " + type + ". Silent option has been removed. " +
      'Use the filter functionality in the vue-devtools'
    );
  }
};
```


## mutation

负责数据变更

参考 [registerMutation](#registerMutation)


# 访问Store模块

只介绍暴露出来的几个比较有意思的
- [mapState](#mapState)
- [mapMutations](#mapMutations)
- [mapGetters](#mapGetters)
- [mapActions](#mapActions)

## mapState

返回的是一个对象，该对象保存的是以被访问的属性名为 `key`，函数返回值为 `函数执行的结果` 或 `state.` 的结果
```javascript
var mapState = normalizeNamespace(function (namespace, states) {
  // 返回对象
  var res = {};
  // 将数组转换成对象数组进行遍历
  normalizeMap(states).forEach(function (ref) {
    var key = ref.key;
    var val = ref.val;

    // 把要被访问的属性以states的成员为key，function为值的形式缓存到res对象里
    res[key] = function mappedState () {
      // 获取state
      var state = this.$store.state;
      // 获取getters
      var getters = this.$store.getters;
      // 获取命名空间
      if (namespace) {
        // 根据命名空间获取模块
        var module = getModuleByNamespace(this.$store, 'mapState', namespace);
        if (!module) {
          return
        }
        // 获取模块上的state
        state = module.context.state;
        // 获取模块上的getters
        getters = module.context.getters;
      }
      // 返回值是从state里取得
      return typeof val === 'function'
        ? val.call(this, state, getters)
        : state[val]
    };
    // mark vuex getter for devtools
    res[key].vuex = true;
  });
  return res
});
```
内部通过函数取值或直接从 `state` 里取值

```vue
<template>
  <div id="app">
    {{ styleId() }}
  </div>
</template>

<script>
import { mapState } from 'vuex';

export default {
  name: 'index',
  data() {
    return {
      ...mapState(['styleId'])
    };
  }
};
</script>
```

`mapState(['styleId'])` 获取的是 `res` 对象上的方法，通过模板表达式 `{{ styleId }}` 获取的是方法，需要执行后的结果才能展示数据



## mapMutations

返回的是一个对象。该对象保存的是以被访问的属性名为 `key`，函数返回值值为 `Promise` 对象
```javascript
var mapMutations = normalizeNamespace(function (namespace, mutations) {
  var res = {}; // 返回值
  // 将数组转换成对象数组进行遍历
  normalizeMap(mutations).forEach(function (ref) {
    var key = ref.key;
    var val = ref.val;

    // 把要被访问的属性以mutations的成员为key，function为值的形式缓存到res对象里
    res[key] = function mappedMutation () {
      // 获取传递的参数
      var args = [], len = arguments.length;
      while ( len-- ) args[ len ] = arguments[ len ];

      // 获取当前实例上的 commit 方法
      var commit = this.$store.commit;
      // 存在命名空间
      if (namespace) {
        // 获取命名空间上的模块
        var module = getModuleByNamespace(this.$store, 'mapMutations', namespace);
        if (!module) {
          return
        }
        // 获取模块上的 commit 方法
        commit = module.context.commit;
      }
      return typeof val === 'function'
        ? val.apply(this, [commit].concat(args)) // 如果传入的mutations的成员为functionl；类型
        : commit.apply(this.$store, [val].concat(args))
    };
  });
  return res
});
```
内部通过 `commit` 提交数据变更请求，执行注册过的 `mutation`，而后，根据响应式编程机制，异步刷新 `mapGetters` 或 `getters` 里的属性。

通过案例分解
```vue
<template>
  <div id="app">
    {{ styleId }}
  </div>
</template>

<script>
import { mapGetters, mapMutations } from 'vuex';

export default {
  name: 'index',
  computed: {
    ...mapGetters(['styleId'])
  },
  mounted() {
    this.$nextTick(() => {
      this.SET_STYLE_ID(98765);
    });
  },
  methods: {
    ...mapMutations(['SET_STYLE_ID'])
  }
};
</script>
```

`mapGetters(['styleId'])` 负责从缓存的 `res` 里获取对应的 `function`，`{{ styleId }}` 会执行这个方法。

`mapMutations(['SET_STYLE_ID'])` 同理，从缓存的对象里获取对应的 `function`，`this.SET_STYLE_ID(98765)` 直接修改数据，其内部仍然是通过 `commit` 提交数据变更请求，而后，执行数据变更，`computed` 会得到计算后新的值


## mapGetters

返回的是一个对象，该对象保存的是以被访问的属性名为 `key`，函数返回值值为 `this.$store.getters[val]` 的函数里，使用的时候，再去执行该函数。
```javascript
var mapGetters = normalizeNamespace(function (namespace, getters) {
  var res = {};
  // 将数组转换成对象数组进行遍历
  normalizeMap(getters).forEach(function (ref) {
    var key = ref.key;
    var val = ref.val;

    val = namespace + val;
    // 把要被访问的属性以getters的成员为key，function为值的形式缓存到res对象里
    res[key] = function mappedGetter () {
      if (namespace && !getModuleByNamespace(this.$store, 'mapGetters', namespace)) {
        return
      }
      if ("development" !== 'production' && !(val in this.$store.getters)) {
        console.error(("[vuex] unknown getter: " + val));
        return
      }
      // 返回值是从getters里取得
      return this.$store.getters[val]
    };
    // mark vuex getter for devtools
    res[key].vuex = true;
  });
  return res
});

function normalizeMap (map) {
  return Array.isArray(map)
    ? map.map(function (key) { return ({ key: key, val: key }); }) // 将数组转换成对象数组
    : Object.keys(map).map(function (key) { return ({ key: key, val: map[key] }); })
}
```

`mapGetters(['styleId'])` 实际上把 `['styleId']` 传递给了 `getters`，如下图

![mapGetters](./img/mapGetters.png)


## mapActions

返回的是一个对象，该对象保存的是以被访问的属性名为 `key`，函数返回值为 `Promise` 对象
```javascript
var mapActions = normalizeNamespace(function (namespace, actions) {
  var res = {};
  normalizeMap(actions).forEach(function (ref) {
    var key = ref.key;
    var val = ref.val;

    // 把要被访问的属性以actions的成员为key，function为值的形式缓存到res对象里
    res[key] = function mappedAction () {
      // 获取传递的参数
      var args = [], len = arguments.length;
      while ( len-- ) args[ len ] = arguments[ len ];

      // 获取dispatch方法
      var dispatch = this.$store.dispatch;
      // 是否存在命名空间
      if (namespace) {
        // 根据命名空间获取模块
        var module = getModuleByNamespace(this.$store, 'mapActions', namespace);
        if (!module) {
          return
        }
        // 重新获取当前模块运行环境上的dispatch
        dispatch = module.context.dispatch;
      }
      return typeof val === 'function'
        ? val.apply(this, [dispatch].concat(args)) // actions成员为function类型则
        : dispatch.apply(this.$store, [val].concat(args))
    };
  });
  return res
});
```
内部借助了 `dispatch` 实现了 `action` 触发的变更请求。

下面通过案例理解 `mapActions`
```vue
<template>
  <div id="app">
    {{ styleId }}
  </div>
</template>

<script>
import { mapActions } from 'vuex';

export default {
  name: 'index',
  mounted() {
    this.$nextTick(() => {
      this.SetStyleId(124578);
    });
  },
  methods: {
    ...mapActions(['SetStyleId'])
  }
};
</script>
```

`mapActions(['SetStyleId'])` 把要被访问的属性以 `actions` 的成员为 `key`，`function` 为值的形式缓存到对象里

`this.SetStyleId(124578)` 会判断 `key` 的类型，如果为 `function` 类型，执行该方法，否则，借助 `Store` 实例的 `dispatch` 上的方法，传入参数 `124578`（参数也可以是数组类型），然后，执行该方法，同样，返回一个 `Promise` 对象。


# 状态订阅

数据发生变更，可以知道是什么原因产生的

## subscribe

`subscribe` 监听数据变更（`mutation`）

监听方法采用 `观察者模式` 实现
```javascript
Store.prototype.subscribe = function subscribe (fn) {
  return genericSubscribe(fn, this._subscribers)
};

function genericSubscribe (fn, subs) {
  if (subs.indexOf(fn) < 0) {
    subs.push(fn);
  }
  return function () {
    var i = subs.indexOf(fn);
    if (i > -1) {
      subs.splice(i, 1);
    }
  }
}
```
先将需要订阅的方法丢到数组里，然后，执行对应的方法。

如何使用？
```javascript
setTimeout(() => {
  this.$store.dispatch('SetStyleId', 123);
}, 5000);
this.$store.subscribe((mutation, state) => {
  console.log(mutation, state);
});
```

5秒之后会接收到数据变更，控制台打印如下信息

![subscribe](./img/subscribe.png)

## subscribeAction

`subscribeAction` 监听 `action`
```javascript
setTimeout(() => {
  this.$store.dispatch('SetStyleId', 123);
}, 5000);
this.$store.subscribe((mutation, state) => {
  console.log(mutation, state);
});
this.$store.subscribeAction((action, state) => {
  console.log(action, state);
});
```

请查看控制台信息输出

# registerModule

`registerModule` 动态注册模块

下面是完整代码
```vue
<template>
  <div id="app">
    {{ styleId }}
    {{ $store.state.dynamicModule.dynamicNumber }}
  </div>
</template>

<script>
import { mapGetters } from 'vuex';

const DynamicStoreModule = {
  install(instance) {
    instance.$store.registerModule(['dynamicModule'], {
      namespaced: true,
      state: {
        dynamicNumber: Date.now()
      },
      actions: {
        setTest: ({ commit }, val) => {
          commit('putTest', val);
        }
      },
      mutations: {
        putTest: (state, val) => {
          state.dynamicNumber = val;
        }
      }
    });
  },
  uninstall(instance) {
    instance.$store.unregisterModule(['dynamicModule']);
  }
};

export default {
  name: 'index',
  computed: {
    ...mapGetters(['styleId'])
  },
  created() {
    DynamicStoreModule.install(this);
  },
  beforeDestory() {
    DynamicStoreModule.uninstall(this);
  },
  mounted() {
    console.log(this);
  }
};
</script>
```

本来不打算做解释的，只说一下感觉：数据共享度不够高，或许刚刚好，或许有点画地为牢。

# unregisterModule

参考[registerModule](#registerModule)

# forEachValue

高频方法，下面来一睹芳容
```javascript
function forEachValue (obj, fn) {
  Object.keys(obj).forEach(function (key) { return fn(obj[key], key); });
}
```

- `obj` 为 `Object` 类型，`Object.keys` 对象转换成数组
- `fn` 是写法惯例，就是 `function` 简写，有些人喜欢称之为 `callback`
- `fn(obj[key], key)` 以 `obj` 的键值对为参数执行


# Module

负责模块的增删改以及模块访问、遍历

- `addChild(key, module)` 添加子模块
- `removeChild(key)` 删除子模块
- `getChild(key)` 获取子模块
- `update(rawModule)` 更新模块的 `actions`、`mutations`、`getters`
- `forEachChild(fn)`
- `forEachGetter(fn)`
- `forEachAction(fn)`
- `forEachMutation(fn)`
- `Object.defineProperties( Module.prototype, prototypeAccessors$1 )` 扩展 `Module` 属性

# ModuleCollection

- `get` 获取模块
- `getNamespace` 获取模块的命名空间
- `update` 更新收集的模块（被 `Store` 实例调用：`hotUpdate`）
- `register` 注册，源码解析参考[这里](#_modules)
- `unregister` 卸载


# Store

- `commit` 参考[commit](#commit)
- `dispatch` 参考[dispatch](#dispatch)
- `subscribe` 参考[subscribe](#subscribe)
- `subscribeAction` 和 `subscribe` 类似，参考[subscribe](#subscribe)
- `watch` 是 `Vue` 实例的 `watch`
- `replaceState` 使用 `commit` 替换状态(`state`)

  ```javascript
  Store.prototype.replaceState = function replaceState (state) {
      var this$1 = this;
  
    this._withCommit(function () {
      this$1._vm._data.$$state = state;
    });
  };
  ```

- `registerModule` 参考[registerModule](#registerModule)
- `unregisterModule` 参考[registerModule](#registerModule)
- `hotUpdate` 更新模块
- `_withCommit` 参考[_committing](#_committing)
- `Object.defineProperties( Store.prototype, prototypeAccessors )`

  重写了 `get`、`set` 方法，`get` 是从 `Store` 实例的 `_vm` 属性里获取 `$$state` 数据，具体实现参考[resetStoreVM](#resetStoreVM)
  ```javascript
  var prototypeAccessors = { state: { configurable: true } };
  
  prototypeAccessors.state.get = function () {
    return this._vm._data.$$state
  };
  
  prototypeAccessors.state.set = function (v) {
    {
      assert(false, "Use store.replaceState() to explicit replace store state.");
    }
  };
  ```

