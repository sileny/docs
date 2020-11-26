# store

Vuex 源码分析

- [vuex源码全解](./vuex.md)
- [state](#state)
- [getters](#getters)
- [actions](#actions)
- [mutations](#mutations)
- [modules](#modules)
- [plugins](#plugins)
- [strict](#strict)

## StoreOptions&lt;S&gt;

`StoreOptions` 模型如下，也就是一个完整的 `Store` 构造
```
export interface StoreOptions<S> {
  state?: S;
  getters?: GetterTree<S, S>;
  actions?: ActionTree<S, S>;
  mutations?: MutationTree<S>;
  modules?: ModuleTree<S>;
  plugins?: Plugin<S>[];
  strict?: boolean;
}
```


## demo

本案例完整的 `store`，在下面的 `actions`、`mutations` 等地方省略
```javascript
import Vue from 'vue';
import Vuex from 'vuex';
import getters from './getters';
import cover from './modules/cover';
import { getStyleId, setStyleId } from '@/utils/utils';

Vue.use(Vuex);

const store = new Vuex.Store({
  modules: {
    cover
  },
  getters,
  state: {
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


### state

`state` 一般而言，是一个对象，用来存储数据。

数据的存储格式为 `键值对` 的形式。
```javascript
const store = new Vuex.Store({
  state: {
    styleId: parseInt(getStyleId(), 10)
  }
};
```

当然，`state` 还可以单独写在一个文件里，假设这个文件为 `state.js`，其内容，可以如下，
```javascript
// state.js
const state = {
  styleId: parseInt(getStyleId(), 10)
};
```
写好的 `state` 可以在 `store` 里引入
```javascript
// store.js
import state from './state.js';

const store = new Vuex.Store({
  state
};
```

高端的 `state` 的写法
```javascript
const store = new Vuex.Store({
  state: () => ({
    styleId: parseInt(getStyleId(), 10)
  })
};
```
与第一个代码片段没有什么太大的不同，唯一的不同在于 `state` 是 `function` 类型。

为啥，这都行？可能 `function` 是 `一等公民`。也有可能是 `actions` 和 `mutations`、`plugins` 等都是 `function` 类型吧，然后，`state` 也不能显得那么死板或者特立独行，干脆也支持 `function`，然后，在内部做了函数类型判断，这样，大家显得步调一致。

如何保持步调一致？源码如下，
```javascript
var Module = function Module (rawModule, runtime) {
  // ...
  var rawState = rawModule.state; // 获取state
  // state可以为function类型，如果是function类型，则得到函数执行的结果，否则为对象类型
  this.state = (typeof rawState === 'function' ? rawState() : rawState) || {};
};
```
好了，现在真相大白

总之，`state` 理解成对象就行了。

再高端一点的玩法，看[这里](vuex.md#mapState)


#### <s>说明
```
var Store = function Store (options) {
  // ...
  var state = options.state; if ( state === void 0 ) state = {};
  if (typeof state === 'function') {
    state = state() || {};
  }

  // ...
  installModule(this, state, [], this._modules.root);

  // ...
  resetStoreVM(this, state);
  // ...
}
```
这里并没有将 `state` 指向 `Store` 实例，而是以传参的形式，给了 `installModule` 和 `resetStoreVM` 方法。

- 为啥不直接将 `state` 指向 `this`

  - 最终的处理逻辑与 `installModule` 和 `resetStoreVM` 类似，复用了这俩方法
  - 指向 `this` 后代码变得不美观、容易被覆盖
</s>

### getters

负责收集需要访问的数据方法，可以通过 `computed` 计算出结果

```javascript
// getters.js
export default {
  styleId: state => state.styleId
};
```

入口程序
```javascript
import Vue from 'vue';
import App from './App.vue';
import store from './store/store';

Vue.config.productionTip = false;

new Vue({
  store,
  render: h => h(App)
}).$mount('#app');
```

页面访问
```vue
<template>
  <div id="app">
    {{ styleId }}
  </div>
</template>

<script>
import { mapGetters } from 'vuex';

export default {
  name: 'index',
  computed: {
    ...mapGetters(['styleId'])
  }
};
</script>
```

具体的原理参考[installModule](vuex.md#installmodule) 和 [registerGetter](vuex.md#registergetter)

### actions

写法参见[demo](#demo)

提交数据变更操作

```vue
<template>
  <div id="app">
    {{ styleId }}
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex';

export default {
  name: 'index',
  computed: {
    ...mapGetters(['styleId'])
  },
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

具体运行原理参考 [action](vuex.md#action)


### mutations

写法参见[demo](#demo)

负责执行数据变更操作的业务

先注册到 `Store` 实例上，然后，在需要变更的时候执行变更操作，原理参考[这里](vuex.md#mapMutations)


### modules

写法参见[demo](#demo)

负责递归收集子模块

原理参考[实例化Store对象](vuex.md#实例化Store对象) 和 [installModule](vuex.md#installModule)


### plugins

自己玩玩看

也可以参考[plugins](vuex.md#plugins)


### strict

严格模式下，约束修改是否来自 `commit`

原理参考 [strict](vuex.md#strict) 和 [committing](vuex.md#_committing)

