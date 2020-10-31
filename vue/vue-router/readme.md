# vue-router


## install

为 `Vue` 实例添加全局插件，用法 `Vue.use(VueRouter)`

- [RouterView](#routerview)
- [RouterLink](#routerlink)

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

### RouterView

`./components/view.js` 里的源码

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


### RouterLink

源码如下
```js
/* @flow */

import { createRoute, isSameRoute, isIncludedRoute } from '../util/route'
import { extend } from '../util/misc'
import { normalizeLocation } from '../util/location'
import { warn } from '../util/warn'

// work around weird flow bug
const toTypes: Array<Function> = [String, Object]
const eventTypes: Array<Function> = [String, Array]

const noop = () => {}

export default {
  name: 'RouterLink',
  props: {
    to: { // 路由参数
      type: toTypes,
      required: true
    },
    tag: { // 用什么标签包裹，默认是a标签
      type: String,
      default: 'a'
    },
    exact: Boolean, // 严格匹配模式
    append: Boolean, // 为true，则会在当前路由path前加上基准路径
    replace: Boolean, // 为true，则则会使用 router.replace 实现页面的刷新，而不是 router.push 的有记录历史的方式
    activeClass: String, // 激活样式
    exactActiveClass: String, // 严格激活样式
    ariaCurrentValue: {
      type: String,
      default: 'page'
    },
    event: { // 通过什么事件事件导航。默认，通过 click 实现导航
      type: eventTypes,
      default: 'click'
    }
  },
  render (h: Function) {
    const router = this.$router
    const current = this.$route // 当前路由信息
    // 查找to对应的路由信息：{ location, route, ref, href, normalizedTo, resolved }
    const { location, route, href } = router.resolve(
      this.to,
      current,
      this.append
    )

    const classes = {}
    // 路由激活样式
    const globalActiveClass = router.options.linkActiveClass
    const globalExactActiveClass = router.options.linkExactActiveClass
    // Support global empty active class
    const activeClassFallback =
      globalActiveClass == null ? 'router-link-active' : globalActiveClass
    const exactActiveClassFallback =
      globalExactActiveClass == null
        ? 'router-link-exact-active'
        : globalExactActiveClass
    const activeClass =
      this.activeClass == null ? activeClassFallback : this.activeClass
    const exactActiveClass =
      this.exactActiveClass == null
        ? exactActiveClassFallback
        : this.exactActiveClass

    const compareTarget = route.redirectedFrom
      ? createRoute(null, normalizeLocation(route.redirectedFrom), null, router)
      : route

    // 设置激活样式
    classes[exactActiveClass] = isSameRoute(current, compareTarget)
    classes[activeClass] = this.exact
      ? classes[exactActiveClass]
      : isIncludedRoute(current, compareTarget)

    const ariaCurrentValue = classes[exactActiveClass] ? this.ariaCurrentValue : null

    const handler = e => {
      if (guardEvent(e)) {
        // 如果 `router-link` 的 `replace` 属性为 true
        if (this.replace) {
          // 替换当前页面内容
          router.replace(location, noop)
        } else {
          // 推入历史纪录栈
          router.push(location, noop)
        }
      }
    }

    // link事件
    const on = { click: guardEvent }
    if (Array.isArray(this.event)) {
      // <router-link to="/detail" :event="['dblclick']">detail</router-link>
      // 会在多种事件下触发跳转
      this.event.forEach(e => {
        on[e] = handler
      })
    } else {
      // 默认为click事件
      // 当然，也可以修改事件触发的类型，比如，<router-link to="/detail" event="dblclick">detail</router-link>，将会在双击时跳转
      on[this.event] = handler
    }

    const data: any = { class: classes }

    const scopedSlot =
      !this.$scopedSlots.$hasNormal &&
      this.$scopedSlots.default &&
      this.$scopedSlots.default({
        href,
        route,
        navigate: handler,
        isActive: classes[activeClass],
        isExactActive: classes[exactActiveClass]
      })

    if (scopedSlot) {
      if (scopedSlot.length === 1) {
        return scopedSlot[0]
      } else if (scopedSlot.length > 1 || !scopedSlot.length) {
        if (process.env.NODE_ENV !== 'production') {
          warn(
            false,
            `RouterLink with to="${
              this.to
            }" is trying to use a scoped slot but it didn't provide exactly one child. Wrapping the content with a span element.`
          )
        }
        return scopedSlot.length === 0 ? h() : h('span', {}, scopedSlot)
      }
    }

    if (this.tag === 'a') {
      data.on = on
      data.attrs = { href, 'aria-current': ariaCurrentValue }
    } else {
      // find the first <a> child and apply listener and href
      // 例子，<router-link to="xxx" tag="span">aaa</router-link>
      // 会先查找a标签
      const a = findAnchor(this.$slots.default)
      if (a) {
        // 找到a标签，则绑定事件，使其具有路由功能
        // in case the <a> is a static node
        a.isStatic = false
        const aData = (a.data = extend({}, a.data))
        aData.on = aData.on || {}
        // transform existing events in both objects into arrays so we can push later
        for (const event in aData.on) {
          const handler = aData.on[event]
          if (event in on) {
            aData.on[event] = Array.isArray(handler) ? handler : [handler]
          }
        }
        // append new listeners for router-link
        for (const event in on) {
          if (event in aData.on) {
            // on[event] is always a function
            aData.on[event].push(on[event])
          } else {
            aData.on[event] = handler
          }
        }

        // 设置属性
        const aAttrs = (a.data.attrs = extend({}, a.data.attrs))
        aAttrs.href = href
        aAttrs['aria-current'] = ariaCurrentValue
      } else {
        // doesn't have <a> child, apply listener to self
        data.on = on
      }
    }

    // 渲染组件
    return h(this.tag, data, this.$slots.default)
  }
}

function guardEvent (e) {
  // don't redirect with control keys
  if (e.metaKey || e.altKey || e.ctrlKey || e.shiftKey) return
  // don't redirect when preventDefault called
  if (e.defaultPrevented) return
  // don't redirect on right click
  if (e.button !== undefined && e.button !== 0) return
  // don't redirect if `target="_blank"`
  if (e.currentTarget && e.currentTarget.getAttribute) {
    const target = e.currentTarget.getAttribute('target')
    if (/\b_blank\b/i.test(target)) return
  }
  // this may be a Weex event which doesn't have this method
  if (e.preventDefault) {
    e.preventDefault()
  }
  return true
}

function findAnchor (children) {
  if (children) {
    let child
    for (let i = 0; i < children.length; i++) {
      child = children[i]
      // 找到a标签则返回该节点
      if (child.tag === 'a') {
        return child
      }
      if (child.children && (child = findAnchor(child.children))) {
        return child
      }
    }
  }
}

```

首先，找到与当前路径匹配的路由信息，设定路由激活样式

`event` 属性默认为 `click`，表示当点击时触发跳转。可以指定为别的事件类型，比如，双击的时候跳转，只需要将 `event="dblclick"` 即可。也可以指定多种事件类型触发跳转，比如，`<router-link to="/detail" :event="['click', 'contextmenu']">detail</router-link>`，可以在 `click` 或 `contextmenu` 的时候实现跳转。

`tag` 用来指定路由渲染的元素。默认是通过 `a` 渲染。可以指定为别的元素，该元素可以嵌套 `a` 元素，如果该元素内部有 `a` 元素，那么，会先找到 `a` 元素，然后，为这个 `a` 元素设置属性，绑定事件，使其具有路由功能。

`replace` 属性为 `true`，则，使用 `router.replace` 的方式替换页面内容，否则，使用 `router.push` 推入历史纪录栈


- 3
- 4
