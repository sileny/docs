# vue-router


## install

为 `Vue` 实例添加全局插件，用法 `Vue.use(VueRouter)`

下面是 `install` 源码
```js
import View from './components/view' /* 1 */
import Link from './components/link' /* 2 */

export let _Vue /* 3 */

export function install (Vue) { /* 4 */
  if (install.installed && _Vue === Vue) return
  install.installed = true

  _Vue = Vue

  const isDef = v => v !== undefined

  const registerInstance = (vm, callVal) => { // 为每一个路由实例注入 Vue 实例
    let i = vm.$options._parentVnode
    if (isDef(i) && isDef(i = i.data) && isDef(i = i.registerRouteInstance)) {
      i(vm, callVal)
    }
  }

  Vue.mixin({
    beforeCreate () { // 在Vue初始化时混入属性
      if (isDef(this.$options.router)) { // 首次执行，确保后续能访问到私有属性
        this._routerRoot = this
        this._router = this.$options.router
        this._router.init(this)
        Vue.util.defineReactive(this, '_route', this._router.history.current) // 响应式编程
      } else {
        this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
      }
      registerInstance(this, this)
    },
    destroyed () {
      registerInstance(this)
    }
  })

  // 向 Vue 实例添加 $router 属性，确保每个实例都可以直接使用 `this.$router` 访问到
  Object.defineProperty(Vue.prototype, '$router', {
    get () { return this._routerRoot._router }
  })

  // 向 Vue 实例添加 $route 属性，确保每个实例都可以直接使用 `this.$route` 访问到
  Object.defineProperty(Vue.prototype, '$route', {
    get () { return this._routerRoot._route }
  })

  Vue.component('RouterView', View) // 添加内置组件 `RouterView` 组件
  Vue.component('RouterLink', Link) // 添加内置组件 `RouterLink`

  const strats = Vue.config.optionMergeStrategies
  // use the same hook merging strategy for route hooks
  strats.beforeRouteEnter = strats.beforeRouteLeave = strats.beforeRouteUpdate = strats.created
}

```

- 1

`./components/view.js` 下的源码

```js
import { warn } from '../util/warn'
import { extend } from '../util/misc'

export default {
  name: 'RouterView',
  functional: true,
  props: {
    name: {
      type: String,
      default: 'default'
    }
  },
  render (_, { props, children, parent, data }) {
    // used by devtools to display a router-view badge
    data.routerView = true

    // directly use parent context's createElement() function
    // so that components rendered by router-view can resolve named slots
    const h = parent.$createElement
    // 默认为 default，如果指定了名称，则会在具名插槽渲染对应的路由页面(<router-view name="named"></router-view>)
    const name = props.name
    const route = parent.$route
    const cache = parent._routerViewCache || (parent._routerViewCache = {})

    // determine current view depth, also check to see if the tree
    // has been toggled inactive but kept-alive.
    // 判断当前视图是否有嵌套
    let depth = 0
    let inactive = false
    while (parent && parent._routerRoot !== parent) { // parent 为 Vue 实例
      const vnodeData = parent.$vnode ? parent.$vnode.data : {}
      if (vnodeData.routerView) { // 如果是嵌套的路由，则累加depth深度
        depth++
      }
      // 是否使用了 `keep-alive` 组件缓存组件页面，以及该组件是否实现了 `activated` 和 `deactivated` 钩子函数
      if (vnodeData.keepAlive && parent._directInactive && parent._inactive) {
        inactive = true
      }
      parent = parent.$parent
    }
    data.routerViewDepth = depth // 路由嵌套视图深度

    // render previous view if the tree is inactive and kept-alive
    if (inactive) { // 如果使用了 `keep-alive` 缓存了组件页面
      const cachedData = cache[name]
      const cachedComponent = cachedData && cachedData.component
      if (cachedComponent) { // 如果找到对应的缓存的组件
        // #2301
        // pass props
        if (cachedData.configProps) { // 从缓存的数据里找到对应的数据，填充到被缓存的组件里
          fillPropsinData(cachedComponent, data, cachedData.route, cachedData.configProps)
        }
        return h(cachedComponent, data, children) // 渲染组件
      } else {
        // render previous empty view
        return h()
      }
    }

    const matched = route.matched[depth]
    const component = matched && matched.components[name]

    // render empty node if no matched route or no config component
    if (!matched || !component) {
      cache[name] = null
      return h()
    }

    // cache component
    // 如果组件已经加载完毕，则缓存该组件
    cache[name] = { component }

    // attach instance registration hook
    // this will be called in the instance's injected lifecycle hooks
    data.registerRouteInstance = (vm, val) => {
      // val could be undefined for unregistration
      const current = matched.instances[name]
      if (
        (val && current !== vm) ||
        (!val && current === vm)
      ) {
        matched.instances[name] = val
      }
    }

    // also register instance in prepatch hook
    // in case the same component instance is reused across different routes
    ;(data.hook || (data.hook = {})).prepatch = (_, vnode) => {
      matched.instances[name] = vnode.componentInstance
    }

    // register instance in init hook
    // in case kept-alive component be actived when routes changed
    // 当组件被 `keep-alive` 包裹，路由发生变化后，会执行组件实例里的 `activated` 和 `deactivated` 钩子函数，在该钩子函数里可以维护组件的数据状态
    data.hook.init = (vnode) => {
      if (vnode.data.keepAlive &&
        vnode.componentInstance &&
        vnode.componentInstance !== matched.instances[name]
      ) {
        matched.instances[name] = vnode.componentInstance
      }
    }

    const configProps = matched.props && matched.props[name]
    // save route and configProps in cache
    if (configProps) {
      extend(cache[name], {
        route,
        configProps
      })
      fillPropsinData(component, data, route, configProps)
    }

    return h(component, data, children)
  }
}

function fillPropsinData (component, data, route, configProps) {
  // resolve props
  let propsToPass = data.props = resolveProps(route, configProps)
  if (propsToPass) {
    // clone to prevent mutation
    propsToPass = data.props = extend({}, propsToPass)
    // pass non-declared props as attrs
    const attrs = data.attrs = data.attrs || {}
    for (const key in propsToPass) {
      if (!component.props || !(key in component.props)) {
        attrs[key] = propsToPass[key]
        delete propsToPass[key]
      }
    }
  }
}

function resolveProps (route, config) {
  switch (typeof config) {
    case 'undefined':
      return
    case 'object':
      return config
    case 'function':
      return config(route)
    case 'boolean':
      return config ? route.params : undefined
    default:
      if (process.env.NODE_ENV !== 'production') {
        warn(
          false,
          `props in "${route.path}" is a ${typeof config}, ` +
          `expecting an object, function or boolean.`
        )
      }
  }
}

```

`RouterView` 是一个 `functional` 组件，对应的视图默认在 `name` 为 `default` 对应的组件内部。如果指定了路由组件的名称，那么，组件将会显示在对应 `name` 的 `RouterView` 内

`RouterView` 内视图的渲染是通过 `Vue` 的 `createElement` 实现。具体原理如下，

- 每个路由初始化时，都会注入对应的 `roueter` 实例属性，每个路由对应一个组件，会被缓存到 `cache` 里
- 找到当前路由匹配的组件
- 它会先判定当前视图组件的深度(`depth`)以及是否为 `keep-alive` 包裹的组件
- 如果是 `keep-alive` 包裹的组件，且该组件实现了 `activated` 或者 `deactived` 钩子函数
  - 会从缓存的组件里获取数据填充到该组件里，然后，通过 `createElement` 渲染
  - 未匹配到组件，则渲染空内容
- 不是 `keep-alive` 包裹的组件，会先找到对应的组件
  - 如果没有找到对应的组件，则渲染空内容
  - 否则，渲染匹配到的路由组件


- 2
- 3
- 4
