---
title: vue 2.0 源码学习
tag: 
	- vue
---

> 参考: [Vue全家桶实现原理简要梳理](https://github.com/tsy77/blog/issues/1)、[Vue 2.0 的 virtual-dom 实现简析](https://github.com/DDFE/DDFE-blog/issues/18)、[深入Vue2.0底层思想–模板渲染](http://developer.51cto.com/art/201707/545802.htm)、[Vue.js 技术揭秘](https://ustbhuangyi.github.io/vue-analysis)、[从Vue.js源码看异步更新DOM策略及nextTick](https://juejin.im/post/59c7b25a5188257a125d7a98#heading-4)、[Vue 2.0 的 virtual-dom 实现简析](https://github.com/DDFE/DDFE-blog/issues/18)

本文 vue@2.0.0

## 代码结构

```js
├── compiler/               // 解析 template，生成 render 函数和 ast
│   ├── codegen/                    // 根据 ast 生成 render 函数
│   ├── directives/                 // 解析 ast 中指令，生成对应 render 函数
│   └── parser/                     // 正则遍历 template 字符串，通过栈记录元素关系，生成 ast
├── core                    // Vue 实例相关，vue 源码核心
│   ├── components/                 // 通用组件，keep-alive 组件
│   ├── global-api/                 // 注册 Vue 构造函数上的静态方法，如 Vue.install、Vue.set...
│   ├── instance/                   // 注册 Vue.prototype，以及构造函数
│   ├── observer/                   // 数据双向绑定相关，主要有 watcher observer dep
│   ├── util/                       // 工具
│   └── vdom/                       // vnode 相关，包含 createVnode， patchNode 等
├── platforms               // core 基础上扩展
│   └── web/                        // 将 core 中代码包装成 web 平台所需要的方法，如 Vue.prototype.$mount 实际包装了 core 中的 $mount
│       ├── compiler/
│       ├── runtime/
│       ├── server/
│       └── util/
├── server/                 // ssr 相关，执行 Vue 代码，生成 Vue 实例；输出流或字符串，传递给 renderNode，renderNode 通过 vnode 生成各种 HTML 标签
├── sfc
│   └── parser.js
└── shared                  // 上述公共工具
    └── util.js
```

<!-- more -->

## Vue 构造函数

`src/core/instance/index.js` 定义了 Vue 构造函数，并且**初始化了一些 Vue.prototype 的方法**

```js
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  // 初始化
  this._init(options)
}

initMixin(Vue)        // Vue.prototype._init()...
stateMixin(Vue)       // Vue.prototype.$data/$set/$delete/$watch...
eventsMixin(Vue)      // Vue.prototype.$on/$once/$off/$emit...
lifecycleMixin(Vue)   // Vue.prototype._update()/_mount()/$destroy()...
renderMixin(Vue)      // Vue.prototype._render()...

export default Vue
```

### initMixin()

`initMixin()` 给 Vue.prototype 挂载了一个 `_init()` 方法，方便之后 vue 实例初始化调用

> Vue 初始化主要就是**合并配置(`mergeOptions()`）、初始化生命周期 (`initLifecycle()`)、初始化事件中心(`initEvents()`)、初始化 data props cumputed watch methods(`initState()`)、初始化渲染(`initRender()`)**等

```js
// src/core/instance/init.js
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    // ...
    // 合并参数
    vm.$options = mergeOptions(
      resolveConstructorOptions(vm.constructor),
      options || {},
      vm
    )
    // ...
    initLifecycle(vm) // 初始化生命周期
    initEvents(vm)  // 初始化事件中心
    callHook(vm, 'beforeCreate') // beforeCreate 钩子 数据属性未初始化 vm.$el(vue 实例根 dom 元素) 未初始化
    initState(vm) // 初始化 data props computed watch methods
    callHook(vm, 'created') // created 钩子 数据属性初始化 vm.$el 未初始化
    initRender(vm) // 初始化渲染
    // ...
  }
  // ...
}
```

### stateMixin()

`stateMixin()` 声明了 `Vue.prototype.$data`、`Vue.prototype.$set`、`Vue.prototype.$delete`、`Vue.prototype.$watch`

```js
// src/core/instance/state.js

// ...
export function stateMixin (Vue: Class<Component>) {
  // flow somehow has problems with directly declared definition object
  // when using Object.defineProperty, so we have to procedurally build up
  // the object here.
  const dataDef = {}
  dataDef.get = function () {
    return this._data
  }
  // ...
  // 通过 vm.$data(this.$data) 访问 Vue 实例代理了对其 data 对象属性的访问
  Object.defineProperty(Vue.prototype, '$data', dataDef)

  // 通过 vm.$set(this.$set) 设置值，等同于全局 Vue.set
  Vue.prototype.$set = set
  // 通过 vm.$delete(this.$delete) 删除对象的值，如果对象是响应式，确保删除能更新视图，等同于全局 Vue.delete
  Vue.prototype.$delete = del

  // 通过 vm.$watch(cb) / this.$watch(cb) 观察 vue 实例变化的一个表达式或计算属性函数，回调函数得到的参数是新值和旧值
  Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: Function,
    options?: Object
  ): Function {
    // ...
  }
}
```

### eventsMixin()

`eventsMixin()` 主要定义了 `Vue.prototype.$on`、`Vue.prototype.$off`、`Vue.prototype.$once`、`Vue.prototype.$emit`, 原理是利用发布订阅模式，在 `Vue._events` 中给每一个 event 维护一个订阅队列

```js
// src/core/instance/events.js
// ...
export function eventsMixin (Vue: Class<Component>) {
  Vue.prototype.$on = function (event: string, fn: Function): Component {
    const vm: Component = this
    // Vue._events 给每一个 event 维护一个订阅队列，相关回调 push 进入该队列
    ;(vm._events[event] || (vm._events[event] = [])).push(fn)
    return vm
  }

  Vue.prototype.$once = function (event: string, fn: Function): Component {
    const vm: Component = this
    // 重新封装一个函数，函数内部先取消该订阅回调，再执行传入的 fn
    function on () {
      // 先取消该订阅回调，再执行传入的 fn
      vm.$off(event, on)
      fn.apply(vm, arguments)
    }
    on.fn = fn
    vm.$on(event, on)
    return vm
  }

  Vue.prototype.$off = function (event?: string, fn?: Function): Component {
    // 如果没有参数，清空全部 event 的全部订阅回调（vm._events 设置空对象）
    // 如果没有第二个参数， 对应 event 的订阅队列直接清空（清除所有订阅回调）
    // 如果对应 event 要取消的回调，从 Vue._events 取出该 event 的订阅队列遍历查找后删除
    const vm: Component = this
    // all
    if (!arguments.length) {
      vm._events = Object.create(null)
      return vm
    }
    // specific event
    const cbs = vm._events[event]
    if (!cbs) {
      return vm
    }
    if (arguments.length === 1) {
      vm._events[event] = null
      return vm
    }
    // specific handler
    let cb
    let i = cbs.length
    while (i--) {
      cb = cbs[i]
      if (cb === fn || cb.fn === fn) {
        cbs.splice(i, 1)
        break
      }
    }
    return vm
  }

  Vue.prototype.$emit = function (event: string): Component {
    const vm: Component = this
    let cbs = vm._events[event]
    // 取出 vm._events 对应 event 的订阅回调，并传入参数执行
    if (cbs) {
      cbs = cbs.length > 1 ? toArray(cbs) : cbs
      const args = toArray(arguments, 1)
      for (let i = 0, l = cbs.length; i < l; i++) {
        cbs[i].apply(vm, args)
      }
    }
    return vm
  }
}
```

### lifecycleMixin()

`lifecycleMixin()` **定义了 Vue 中经常使用的 `Vue.prototype.update` 方法，每当组件 data 变化或者其他原因需要重新渲染时候，Vue 会调用该方法，对 `vnode` 进行 diff 和 patch 操作**

```js
// src/core/instance/lifecycle.js

// ...
export function lifecycleMixin (Vue: Class<Component>) {
  Vue.prototype._mount = function (
    el?: Element | void,
    hydrating?: boolean
  ): Component {
    const vm: Component = this
    vm.$el = el // vm.$el 已经初始化
    // 即时 vm 实例选项没有 render，$mount 方法调用时候已经 将 template 属性 或 el 属性的 outerHTML 转为 render 函数（在 有 compiler 版本的 vue 库文件下），并将 render 函数 设置为 vm.$options.render 属性值
    // 如果 vue 实例 render 属性不存在
    if (!vm.$options.render) {
      vm.$options.render = emptyVNode
      if (process.env.NODE_ENV !== 'production') {
        /* istanbul ignore if */
        if (vm.$options.template) {
          // 有 template 属性，说明没能转化为 render 函数，说明使用 运行时版本 vue.js 文件，要么预先转换 template ，要么使用 独立构建版本的 vue.js 文件
          warn(
            'You are using the runtime-only build of Vue where the template ' +
            'option is not available. Either pre-compile the templates into ' +
            'render functions, or use the compiler-included build.',
            vm
          )
        } else {
          warn(
            'Failed to mount component: template or render function not defined.',
            vm
          )
        }
      }
    }
    callHook(vm, 'beforeMount') // beforeMount 钩子 vm.$el 已经初始化
    // 调动 vm._render, 通过 render 函数返回 vnode
    // 首次渲染 和 每次数据属性更新会执行该 watcher 中的回调，执行 vm._update() 对 vnode 进行 diff 和 patch
    vm._watcher = new Watcher(vm, () => {
      vm._update(vm._render(), hydrating)
    }, noop)
    hydrating = false
    // root instance, call mounted on self
    // mounted is called for child components in its inserted hook
    if (vm.$root === vm) {
      vm._isMounted = true
      callHook(vm, 'mounted') // mounted 钩子 dom 已经挂载
    }
    return vm
  }

  // 更新节点
  Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    // 如果该组件已经挂载过了则表明这个步骤是更新过程，触发 beforeUpdate 钩子
    if (vm._isMounted) {
      callHook(vm, 'beforeUpdate')
    }
    const prevEl = vm.$el // vm.$el 访问 vue 实例根 dom 元素（挂载元素）
    const prevActiveInstance = activeInstance
    activeInstance = vm
    const prevVnode = vm._vnode // vm._vnode 存放的是 旧 vnode
    vm._vnode = vnode
    if (!prevVnode) {
      // Vue.prototype.__patch__ is injected in entry points
      // based on the rendering backend used.
      // 如果不存在 prevVnode 说明是初次挂载，直接将 vnode 创建一个真实的 dom 节点渲染到 vm.$el (会被替换)
      // vm.$el 为真实 dom
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating)
    } else {
      // 如果存在 prevVnode，进行新旧 vnode 对比 diff，并将需要更新的 dom 操作 patch 形式打到 prevVnode 上，完成真实 dom 更新
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    activeInstance = prevActiveInstance
    // update __vue__ reference
    // 更新实例对象的 __vue__
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
    if (vm._isMounted) {
      callHook(vm, 'updated')
    }
  }

  Vue.prototype._updateFromParent = function (
    propsData: ?Object,
    listeners: ?Object,
    parentVnode: VNode,
    renderChildren: ?VNodeChildren
  ) {
    const vm: Component = this
    const hasChildren = !!(vm.$options._renderChildren || renderChildren)
    vm.$options._parentVnode = parentVnode
    vm.$options._renderChildren = renderChildren
    // update props
    if (propsData && vm.$options.props) {
      observerState.shouldConvert = false
      if (process.env.NODE_ENV !== 'production') {
        observerState.isSettingProps = true
      }
      const propKeys = vm.$options._propKeys || []
      for (let i = 0; i < propKeys.length; i++) {
        const key = propKeys[i]
        vm[key] = validateProp(key, vm.$options.props, propsData, vm)
      }
      observerState.shouldConvert = true
      if (process.env.NODE_ENV !== 'production') {
        observerState.isSettingProps = false
      }
    }
    // update listeners
    if (listeners) {
      const oldListeners = vm.$options._parentListeners
      vm.$options._parentListeners = listeners
      vm._updateListeners(listeners, oldListeners)
    }
    // resolve slots + force update if has children
    if (hasChildren) {
      vm.$slots = resolveSlots(renderChildren, vm._renderContext)
      vm.$forceUpdate()
    }
  }

  Vue.prototype.$forceUpdate = function () {
    const vm: Component = this
    if (vm._watcher) {
      vm._watcher.update()
    }
  }

  Vue.prototype.$destroy = function () {
    const vm: Component = this
    if (vm._isBeingDestroyed) {
      return
    }
    // 调用 beforeDestroy 钩子
    callHook(vm, 'beforeDestroy')
    // 标识位
    vm._isBeingDestroyed = true
    // remove self from parent
    const parent = vm.$parent
    if (parent && !parent._isBeingDestroyed && !vm.$options.abstract) {
      remove(parent.$children, vm)
    }
    // teardown watchers
    // 该组件中的所有 watcher 会从其所在的 Dep 中释放
    if (vm._watcher) {
      vm._watcher.teardown()
    }
    let i = vm._watchers.length
    while (i--) {
      vm._watchers[i].teardown()
    }
    // remove reference from data ob
    // frozen object may not have observer.
    if (vm._data.__ob__) {
      vm._data.__ob__.vmCount--
    }
    // call the last hook...
    vm._isDestroyed = true
    // 调用 destroyed 钩子
    callHook(vm, 'destroyed')
    // turn off all instance listeners.
    // 移除所有事件监听
    vm.$off()
    // remove __vue__ reference
    if (vm.$el) {
      vm.$el.__vue__ = null
    }
  }
}
```

### renderMixin()

`renderMixin()` 定义了 `Vue.prototype._render` 等方法，`_render` 方法调用实例化时传入的 `render函数`（vue 实例）选项中没有 `render 函数`，会从 `template` 属性或 `el` 属性的 outerHTML 编译成 `render 函数`)，**生成 vnode**

```js
// src/core/instance/lifecycle.js
Vue.prototype._mount = function (
    el?: Element | void,
    hydrating?: boolean
  ): Component {
    // ...
    vm._watcher = new Watcher(vm, () => {
      // 一般数据、组件更新
      vm._update(vm._render(), hydrating)
    }, noop)
    hydrating = false
    // ...
    return vm
  }
```

```js
// src/core/instance/render.js

import config from '../config'
import VNode, { emptyVNode, cloneVNode, cloneVNodes } from '../vdom/vnode'
import { normalizeChildren } from '../vdom/helpers'
import {
  warn, formatComponentName, bind, isObject, toObject,
  nextTick, resolveAsset, _toString, toNumber, looseEqual, looseIndexOf
} from '../util/index'

import { createElement } from '../vdom/create-element'

export function initRender (vm: Component) {
  vm.$vnode = null // the placeholder node in parent tree
  vm._vnode = null // the root of the child tree
  vm._staticTrees = null
  vm._renderContext = vm.$options._parentVnode && vm.$options._parentVnode.context
  vm.$slots = resolveSlots(vm.$options._renderChildren, vm._renderContext)
  // bind the public createElement fn to this instance
  // so that we get proper render context inside it.

  // 除了 vm.$createElement, 之后的 vue 版本还提供了 vm._c 方法，两者支持的参数相同，内部都是调用了 createElement 方法，只是一般 vm.$createElement 一般用于用户手写 render 函数，vm._c 是被 vue 内部被模板编译成的 render 函数使用
  vm.$createElement = bind(createElement, vm)
  if (vm.$options.el) {
    vm.$mount(vm.$options.el)
  }
}

export function renderMixin (Vue: Class<Component>) {
  Vue.prototype.$nextTick = function (fn: Function) {
    nextTick(fn, this)
  }

  Vue.prototype._render = function (): VNode {
    const vm: Component = this
    // 从 vm.$options.render 拿到 render 函数
    const {
      render,
      staticRenderFns,
      _parentVnode
    } = vm.$options

    if (vm._isMounted) {
      // clone slot nodes on re-renders
      for (const key in vm.$slots) {
        vm.$slots[key] = cloneVNodes(vm.$slots[key])
      }
    }

    if (staticRenderFns && !vm._staticTrees) {
      vm._staticTrees = []
    }
    // set parent vnode. this allows render functions to have access
    // to the data on the placeholder node.
    vm.$vnode = _parentVnode
    // render self
    let vnode
    try {
      // 传入 vm.vm.$createElement 执行 render 函数，返回 vnode
      // 一般在 初始化 vue 实例时候写 render 函数 render：h => h(...) h 就是 vm.$createElement 方法
      vnode = render.call(vm._renderProxy, vm.$createElement)
    } catch (e) {
      if (process.env.NODE_ENV !== 'production') {
        warn(`Error when rendering ${formatComponentName(vm)}:`)
      }
      /* istanbul ignore else */
      if (config.errorHandler) {
        config.errorHandler.call(null, e, vm)
      } else {
        if (config._isServer) {
          throw e
        } else {
          setTimeout(() => { throw e }, 0)
        }
      }
      // return previous vnode to prevent render error causing blank component
      vnode = vm._vnode
    }
    // return empty vnode in case the render function errored out
    if (!(vnode instanceof VNode)) {
      if (process.env.NODE_ENV !== 'production' && Array.isArray(vnode)) {
        warn(
          'Multiple root nodes returned from render function. Render function ' +
          'should return a single root node.',
          vm
        )
      }
      vnode = emptyVNode()
    }
    // set parent
    vnode.parent = _parentVnode
    // _render 方法返回 vnode
    return vnode
  }

  // shorthands used in render functions
  // _h 其实就是 createElement 函数
  // vue 新版本 _c 也是 createElement 函数
  Vue.prototype._h = createElement
  // toString for mustaches
  // _s 转为字符串
  Vue.prototype._s = _toString
  // number conversion
  // _n 转为数字
  Vue.prototype._n = toNumber
  // empty vnode
  // _e 空 vnode
  Vue.prototype._e = emptyVNode
  // loose equal
  Vue.prototype._q = looseEqual
  // loose indexOf
  Vue.prototype._i = looseIndexOf

  // render static tree by index
  // _m 通过下标渲染静态节点树 // 传入 index 返回对应的 staticRenderFns 函数并执行
  Vue.prototype._m = function renderStatic (
    index: number,
    isInFor?: boolean
  ): VNode | Array<VNode> {
    let tree = this._staticTrees[index]
    // if has already-rendered static tree and not inside v-for,
    // we can reuse the same tree by doing a shallow clone.
    if (tree && !isInFor) {
      return Array.isArray(tree)
        ? cloneVNodes(tree)
        : cloneVNode(tree)
    }
    // otherwise, render a fresh tree.
    tree = this._staticTrees[index] = this.$options.staticRenderFns[index].call(this._renderProxy)
    if (Array.isArray(tree)) {
      for (let i = 0; i < tree.length; i++) {
        tree[i].isStatic = true
        tree[i].key = `__static__${index}_${i}`
      }
    } else {
      tree.isStatic = true
      tree.key = `__static__${index}`
    }
    return tree
  }
  // ...

  // render v-for
  // _l 渲染 v-for
  Vue.prototype._l = function renderList (
    val: any,
    render: () => VNode
  ): ?Array<VNode> {
    let ret: ?Array<VNode>, i, l, keys, key
    if (Array.isArray(val)) {
      ret = new Array(val.length)
      for (i = 0, l = val.length; i < l; i++) {
        ret[i] = render(val[i], i)
      }
    } else if (typeof val === 'number') {
      ret = new Array(val)
      for (i = 0; i < val; i++) {
        ret[i] = render(i + 1, i)
      }
    } else if (isObject(val)) {
      keys = Object.keys(val)
      ret = new Array(keys.length)
      for (i = 0, l = keys.length; i < l; i++) {
        key = keys[i]
        ret[i] = render(val[key], key, i)
      }
    }
    return ret
  }

  // renderSlot
  // _t 渲染 slot
  Vue.prototype._t = function (
    name: string,
    fallback: ?Array<VNode>
  ): ?Array<VNode> {
    // ...
  }

  // apply v-bind object
  // _b 应用 v-bind 对象
  Vue.prototype._b = function bindProps (
    data: any,
    value: any,
    asProp?: boolean
  ): VNodeData {
    // ...
  }

  // expose v-on keyCodes
  Vue.prototype._k = function getKeyCodes (key: string): any {
    return config.keyCodes[key]
  }
}
// ...
```

## 创建 vue 实例

![](http://ony85apla.bkt.clouddn.com/18-6-13/93728031.jpg)

`new Vue({})` 实际调用了构造函数中的 `this._init()`, `this._init()` 就是调用 `core/instance/init.js` 中定义的 `Vue.prototype._init`

```js
// src/core/instance/init.js
// ...
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // ...
    // expose real self
    vm._self = vm
    // 初始化生命周期
    initLifecycle(vm)
    // 初始化事件中心
    initEvents(vm)
    // 调用 beforeCreate 钩子函数并触发 beforeCreate 钩子事件
    callHook(vm, 'beforeCreate')
    // 初始化 props data methods computed 和 watch
    initState(vm)
    // 调用 created 钩子函数并触发 created 钩子事件
    callHook(vm, 'created')
    // 初始化渲染
    initRender(vm)
  }
  // ...
}
```

### initLifecycle()

`initLifecycle()` 主要是将自己 push 到 parent.$children 中

```js
// src/core/instance/lifecycle.js
// 初始化生命周期
export function initLifecycle (vm: Component) {
  const options = vm.$options

  // locate first non-abstract parent
  // 将 vm 对象存储到 parent 组件中（保证 parent 组件是非抽象组件，如 keep-alive）
  let parent = options.parent
  if (parent && !options.abstract) {
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent
    }
    parent.$children.push(vm)
  }

  vm.$parent = parent
  vm.$root = parent ? parent.$root : vm

  vm.$children = []
  vm.$refs = {}

  vm._watcher = null
  vm._inactive = false
  vm._isMounted = false
  vm._isDestroyed = false
  vm._isBeingDestroyed = false
}
```

### initEvents()

`initEvents()` 主要初始化 `vm._events` 事件中心存放事件

```js
// src/core/instance/events.js
// 初始化事件
export function initEvents (vm: Component) {
  // 在 vm 上创建 _events 对象用来存放事件以及事件对应的订阅队列
  vm._events = Object.create(null)
  // ...
}
```

### initRender()

`initRender()` 定义了 `vm.$createElement` 方法，并**执行 `vm.$mount`**

```js
// 初始化 render
// src/core/instance/render.js
export function initRender (vm: Component) {
  vm.$vnode = null // the placeholder node in parent tree
  vm._vnode = null // the root of the child tree
  vm._staticTrees = null
  vm._renderContext = vm.$options._parentVnode && vm.$options._parentVnode.context
  vm.$slots = resolveSlots(vm.$options._renderChildren, vm._renderContext)
  // bind the public createElement fn to this instance
  // so that we get proper render context inside it.
  // 将 createElement 函数绑定到 vm 实例上
  vm.$createElement = bind(createElement, vm)
  if (vm.$options.el) {
    // 挂载组件
    vm.$mount(vm.$options.el)
  }
}
```

### initState()

`initState()` 主要是 `initProps, initComputed, initData, initMethods,initWatch`，对 props data computed methods watch 等属性进行初始化

```js
// src/core/instance/state.js
// 初始化 props data computed methods watch
export function initState (vm: Component) {
  vm._watchers = []
  initProps(vm)
  initData(vm)
  initComputed(vm)
  initMethods(vm)
  initWatch(vm)
}
```

## 响应式原理

### 响应式对象

#### initData()

以 `initData()` 为例来理解 Vue 的 响应式原理

`initData()` 对 `data` 的初始化做了两件事：

- 对定义 `data` 函数返回对象的遍历，**通过 `proxy` 方法把每一个值 `vm._data.xxx` 代理到 `vm.xxx`**
- **调用 `observe` 方法观测 `data` 变化，将 `data` 变成响应式**

```js
// src/core/instance/state.js
function initData (vm: Component) {
  // 得到 data 数据
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? data.call(vm) // 组件中 data 属性必须是函数类型
    : data || {} // vue 实例中 data 属性

  // 对对象类型进行严格检查，只有对象是纯 js 对象时返回 true
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object.',
      vm
    )
  }
  // proxy data on instance
  // 遍历 data 对象
  const keys = Object.keys(data)
  const props = vm.$options.props
  let i = keys.length

  // 遍历 data 中数据
  while (i--) {
    // 保证 data 中的 key 不和 props 中的 key 重复，props 优先，冲突会报 warning
    if (props && hasOwn(props, keys[i])) {
      process.env.NODE_ENV !== 'production' && warn(
        `The data property "${keys[i]}" is already declared as a prop. ` +
        `Use prop default value instead.`,
        vm
      )
    } else {
      // 将 data 上面的属性代理到 vm 实例上
      proxy(vm, keys[i])
    }
  }
  // observe data
  // 对数据进行添加 getter setter，observe 递归对深层对象的遍历，并添加 getter setter
  observe(data)
  data.__ob__ && data.__ob__.vmCount++
}

// proxy 方法将 data 上属性代理到 vm 上
// 访问 vm[key], 其实代理访问 vm._data[key]
function proxy (vm: Component, key: string) {
  // 判断是否是保留字段
  if (!isReserved(key)) {
    Object.defineProperty(vm, key, {
      configurable: true,
      enumerable: true,
      get: function proxyGetter () {
        return vm._data[key]
      },
      set: function proxySetter (val) {
        vm._data[key] = val
      }
    })
  }
}
```

`observe()` 中 `new Observer()`, 最终会对 `data` 中所有数据调用 `defineReactive` 变成响应式(包括子对象递归 `observe()`)

```js
// src/core/observer/index.js
export function observe (value: any): Observer | void {
  if (!isObject(value)) {
    return
  }
  let ob: Observer | void
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else if (
    observerState.shouldConvert &&
    !config._isServer &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    // 核心代码 实例化一个 Observer 对象
    ob = new Observer(value)
  }
  return ob
}
```

#### Observer 类

`Observer` 类用于给对象添加 getter 和 setter，用于依赖收集和派发更新

- 首先实例化 `Dep` 对象
- 通过执行 `def` 函数把自身实例添加到对象 `value` 的 `__obj__` 属性上（）
- 接着对 `value` 进行判断，对数组会调用 `observeArray` 方法，否则对纯对象调用 `walk` 方法；`observeArray` 遍历数组会再次调用 `observe` 方法， `walk` 方法是遍历对象的 key 调用 `defineReactive` 方法

```js
// src/core/util/lang.js
/**
 * Define a property.
 */
// def(value, '__ob__', this) // 将自身实例添加到数据对象 value 的 __ob__ 属性上
export function def (obj: Object, key: string, val: any, enumerable?: boolean) {
  Object.defineProperty(obj, key, {
    value: val,
    enumerable: !!enumerable,
    writable: true,
    configurable: true
  })
}

// src/core/observer/index.js
export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that has this object as root $data

  constructor (value: any) {
    this.value = value
    // 实例化一个发布者 dep 用于收集订阅者 watcher
    this.dep = new Dep()
    this.vmCount = 0
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      // 如果属性值是 数组，调用 数组 hack 方法变成响应式
      const augment = hasProto
        ? protoAugment
        : copyAugment
      augment(value, arrayMethods, arrayKeys)
      this.observeArray(value)
    } else {
      // 不是数组 new Observer(data) 会调用 walk 函数
      this.walk(value)
    }
  }

  /**
   * Walk through each property and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  // walk 函数会对 data 中每个属性调用 defineReactive 函数，并将其变为响应式
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i], obj[keys[i]])
    }
  }
  // ...
}
```

#### defineReactive

**`defineReactive` 函数主要就是定义一个响应式对象，给对象添加 getter setter**

`defineReactive` 函数首先初始化 `Dep` 对象实例，接着拿到 `obj` 的属性描述符，然后对子对象递归调用 `observe` 方法，保证 `obj` 所有子属性包括嵌套的对象属性都能变成响应式。因此在访问或修改 `obj` 的任意属性包括子属性，都能触发 getter 和 setter

```js
// src/core/observer/index.js
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: Function
) {
  // 实例化一个发布者 dep 用于收集订阅者 watcher
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // 如果之前该对象已经预设了 getter 和 setter 钩子函数则将其去除
  // 新定义的 getter setter 会将其执行，保证不会将原来的覆盖
  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set

  // 对象的子对象递归进行 observe 并返回子节点的 Observer 对象
  let childOb = observe(val)

  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      // 如果对象原本具有 getter 方法则执行
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        // 依赖收集 将 watcher 添加到 dep 中该数据属性对应的订阅者数组
        dep.depend()
        if (childOb) {
          // 子对象进行依赖收集，其实是将同一个 watcher 观察者实例放入一个是当前 dep 中 该数据属性对应的订阅者数组，另一个是子元素 dep 中的 该数据属性对应的订阅者数组
          childOb.dep.depend()
        }
        if (Array.isArray(value)) {
          // 若是数组则要对每个成员进行依赖收集，若数组成员还是数组，则递归
          for (let e, i = 0, l = value.length; i < l; i++) {
            e = value[i]
            e && e.__ob__ && e.__ob__.dep.depend()
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      // 通过 getter 方法获得当前值（如果没有，就是 val），和新值比较，一致则不需要执行下面操作
      const value = getter ? getter.call(obj) : val
      if (newVal === value) {
        return
      }
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        // 如果对象原本用 setter 方法则执行
        setter.call(obj, newVal)
      } else {
        // 赋新值
        val = newVal
      }
      // 新的值同样要进行 observe，保证数据响应式
      childOb = observe(newVal)
      // 派发更新 dep 对象通知所有观察者
      dep.notify()
    }
  })
}
```

### 依赖收集

#### getter

`defineReactive` 方法将普通对象变成响应式对象，**响应式对象 getter 相关的逻辑就是做依赖收集**

下面代码两个关键部分， 一个是 `const dep = new Dep()` 实例化一个 `Dep` 实例，另一个是 `get` 函数中通过 `dep.depend()` 做依赖收集

```js
// src/core/observer/index.js
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: Function
) {
  // 实例化一个发布者 dep 用于收集订阅者 watcher
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // 如果之前该对象已经预设了 getter 和 setter 钩子函数则将其去除
  // 新定义的 getter setter 会将其执行，保证不会将原来的覆盖
  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set

  // 对象的子对象递归进行 observe 并返回子节点的 Observer 对象
  let childOb = observe(val)

  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      // 如果对象原本具有 getter 方法则执行
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        // 依赖收集 将 watcher 添加到 dep 中该数据属性对应的订阅者数组
        dep.depend()
        if (childOb) {
          // 子对象进行依赖收集，其实是将同一个 watcher 观察者实例放入一个是当前 dep 中 该数据属性对应的订阅者数组，另一个是子元素 dep 中的 该数据属性对应的订阅者数组
          childOb.dep.depend()
        }
        if (Array.isArray(value)) {
          // 若是数组则要对每个成员进行依赖收集，若数组成员还是数组，则递归
          for (let e, i = 0, l = value.length; i < l; i++) {
            e = value[i]
            e && e.__ob__ && e.__ob__.dep.depend()
          }
        }
      }
      return value
    },
    // ...
  })
}
```

#### Dep

**`Dep` 类有一个静态属性 `target`, 指向当前正被计算的 `watcher`, 因为同一时间只有一个 `watcher` 被计算，因此 `Dep.target` 指向的 `watcher` 是当前全局唯一的**，自身属性 `subs` 是 `watcher` 数组

`Dep` 和 `watcher` 其实是 观察者 设计模式的实现

```js
// src/core/observer/dep.js
/* @flow */
import type Watcher from './watcher'
import { remove } from '../util/index'

let uid = 0

/**
 * A dep is an observable that can have multiple
 * directives subscribing to it.
 */
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = [] // subs 属性维护 watcher 数组
  }
  // 添加一个 观察者 watcher
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }
  // 移除一个 观察者 watcher
  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }
  // 依赖收集，当存在 Dep.target 时添加观察者 watcher
  depend () {
    if (Dep.target) {
      // 调用 watcher 的 addDep，this 为 dep
      Dep.target.addDep(this)
    }
  }
  // 通知所有观察者，调用 update()
  notify () {
    // stablize the subscriber list first
    // 触发更新，遍历 subs 数组，调用 watcher 的 update 方法
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update() // 遍历调用 watcher 的 update() ，最终调用 watcher 的 run()，执行 callback
    }
  }
}

// the current target watcher being evaluated.
// this is globally unique because there could be only one
// watcher being evaluated at any time.
// Dep.target 指向当前被计算 watcher （全局唯一 watcher） ，同一时间只有一个 watcher 被计算
// 依赖收集完要将 Dep.target 设为 null，防止之后重复添加依赖
Dep.target = null
const targetStack = []

// 将 watcher 观察者实例设置为 Dep.target,用于依赖收集。同时将该实例放入 target 栈中
export function pushTarget (_target: Watcher) {
  if (Dep.target) targetStack.push(Dep.target)
  Dep.target = _target
}

// 将 watcher 观察者实例从 target 栈中取出并设置给 Dep.taget
export function popTarget () {
  Dep.target = targetStack.pop()
}
```

#### watcher

声明组件时使用的 watch，实际调用了 `new Watcher(a, callback)`, watcher 相当于一个观察者。接下来看一下 watcher 代码，看一下它是怎么和 Observer 关联的

```js
// src/core/observer/watcher/js

// ...
// 解析表达式，进行依赖收集的观察者，会在表达式数据更新时触发回调函数。通常用于 $watch api 和指令
export default class Watcher {
  vm: Component;
  getter: Function
  // ...

  constructor (
    vm: Component, // vm
    expOrFn: string | Function, // 表达式
    cb: Function, // 回调
    options?: Object = {}
  ) {
    this.vm = vm
    // _watchers 存放订阅者实例 new Watcher(vm, ....)
    vm._watchers.push(this)
    // options
    this.deep = !!options.deep
    this.user = !!options.user
    this.lazy = !!options.lazy
    this.sync = !!options.sync
    this.expression = expOrFn.toString()
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
      // 初始化渲染 new Watcher(vm, () => vm._update(vm._render(), hydrating))
      // 此时 this.getter = () => vm._update(vm._render(), hydrating)
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = function () {}
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }

    // new Watcher() 时如果不是定义 lazy watcher 会调用 this.get()（pushTaget(this)），就是将自身 watcher 赋给 Dep.target 并调用 this.getter 求值
    this.value = this.lazy
      ? undefined
      : this.get()
  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
  // 获得 getter 值并重新进行依赖收集
  get () {
    // 将自身 watcher 观察者实例设置给 Dep.target，用来依赖收集
    // 相当于 Dep.target = this(watcher)
    pushTarget(this)
    // 调用 this.getter 对于初次渲染 this.getter = () => vm._upddate(vm._render(), hydrating)
    const value = this.getter.call(this.vm, this.vm)
    // "touch" every property so they are all tracked as
    // dependencies for deep watching
    if (this.deep) {
      // 递归每个对象或者数组，触发它们的 getter，使得对象或数组的每一个成员都被依赖收集
      traverse(value)
    }
    // 将之前的观察者实例从 target 栈中取出并设置给 Dep.target
    popTarget()
    this.cleanupDeps()
    return value
  }

  /**
   * Add a dependency to this directive.
   */
  addDep (dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        // 调用 dep 对象添加自身 watcher
        dep.addSub(this)
      }
    }
  }

  // watcher 更新
  update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      // 同步则执行 run 方法 直接渲染视图
      this.run()
    } else {
      // 异步推送到观察者队列中，下一个 tick 时调用
      queueWatcher(this)
    }
  }

  run () {
    // ...
    this.cb.call(this.vm, value, oldValue)
    // ...
  }

  /**
   * Evaluate the value of the watcher.
   * This only gets called for lazy watchers.
   */
  // 获取观察者的值
  evaluate () {
    this.value = this.get()
    this.dirty = false
  }

  /**
   * Clean up for dependency collection.
   */
  cleanupDeps () {
    let i = this.deps.length
    while (i--) {
      const dep = this.deps[i]
      if (!this.newDepIds.has(dep.id)) {
        dep.removeSub(this)
      }
    }
    let tmp = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    this.newDepIds.clear()
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    this.newDeps.length = 0
  }
  // ...
}
```

#### 过程分析

Vue 的 mount 过程核心是通过

```js
vm._watcher = new Watcher(vm, () => {
  vm._update(vm._render(), hydrating)
}, noop)
hydrating = false
```

实例化一个渲染 `watcher` 时，首先进入 `wacher` 构造函数逻辑，执行 `this.get()`

```js
get () {
  pushTarget(this)
  const value = this.getter.call(this.vm, this.vm)
  // "touch" every property so they are all tracked as
  // dependencies for deep watching
  if (this.deep) {
    traverse(value)
  }
  popTarget()
  this.cleanupDeps()
  return value
}
```

首先执行 `pushTarget(this)` 实际上是把将 `Dep.target` 指向的 `watcher` 先压栈(为了恢复用)，接着将**自身赋值给 `Dep.target`**

```js
// src/core/observer/dep.js
export function pushTarget (_target: Watcher) {
  if (Dep.target) targetStack.push(Dep.target)
  Dep.target = _target
}
```

接着执行 `value = this.getter.call(this.vm, this.vm)`, 此时 `this.getter` 对应的就是 `() => vm._update(vm._render(), hydrating)`,
实际就是执行 `vm._update(vm._render(), hydrating)`

它首先执行 `vm._render()`, 内部通过 `render` 函数生成渲染 vnode，在这个过程上会对 vm 上的数据进行访问，就触发了数据对象的 getter

每个对象值的 getter 都有一个 `dep`, 在触发 getter 的时候会调用 `dep.depend()`，也就会执行 `Dep.target.addDep(this)`

```js
// src/core/observer/watcher.js
addDep (dep: Dep) {
  const id = dep.id
  if (!this.newDepIds.has(id)) {
    this.newDepIds.add(id)
    this.newDeps.push(dep)
    if (!this.depIds.has(id)) {
      dep.addSub(this)
    }
  }
}
```

`Dep.target` 已经被赋值为 渲染 `watcher`, 这里主要是做一些逻辑判断（保证同一个数据不会被添加多次）后执行 `dep.addSub(this)`, 就会执行 `this.subs.push(sub)`，也就是把当前 `watcher` 订阅到这个数据持有的 `dep` 的 `subs` 中，目的是为后续数据变化时通知到哪些 `subs` 做准备

在 `vm._render()` 过程中，会触发所有数据的 getter，实际完成了依赖收集的过程。

回到 `get` 方法中

```js
if (this.deep) {
  traverse(value)
}
function traverse (val: any, seen?: Set) {
  let i, keys
  if (!seen) {
    seen = seenObjects
    seen.clear()
  }
  const isA = Array.isArray(val)
  const isO = isObject(val)
  if ((isA || isO) && Object.isExtensible(val)) {
    if (val.__ob__) {
      const depId = val.__ob__.dep.id
      if (seen.has(depId)) {
        return
      } else {
        seen.add(depId)
      }
    }
    if (isA) {
      i = val.length
      while (i--) traverse(val[i], seen)
    } else if (isO) {
      keys = Object.keys(val)
      i = keys.length
      while (i--) traverse(val[keys[i]], seen)
    }
  }
}
```

这里是递归访问 `value`, 触发它所有子项的 `getter`，接下来执行 `popTarget()`，也就是 `Dep.target = targetStack.pop()`。实际上就是把 `Dep.target` 恢复成上一个状态，因为此时当前 vm 的数据依赖已经收集完成，对应的渲染 `Dep.target` 也需要改变。最后执行 `this.cleanupDeps()`

### 派发更新

在给数据设置响应式，getter 进行依赖收集，目的就是修改数据时可以对相关的依赖派发更新

#### setter

setter 的核心逻辑有 2 个，一个是 `childOb = observe(newVal)`，会把新值变为响应式对象，另一个 `dep.notify()` 通知所有观察者

```js
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: Function
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set

  let childOb = observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    // ...
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      if (newVal === value) {
        return
      }
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = observe(newVal)
      dep.notify()
    }
  })
}
```

#### 过程分析

```js
// src/core/observerdep.js/
class Dep {
  // ...
  notify () {
  // stabilize the subscriber list first
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}
```

对组件中的响应的数据进行修改会触发 setter 的逻辑，调用 `dep.notify()` 方法，遍历所有 `subs`，也就是 `watcher` 实例数组，然后调用每一个 `watcher` 的 `update` 方法

```js
class Watcher {
  // ...
  /**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
  update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      // 同步则执行 run 方法 直接渲染视图
      this.run()
    } else {
      // 异步推送到观察者队列中，下一个 tick 时调用
      queueWatcher(this)
    }
  }
}
```

这里针对 `watcher` 的不同状态，会执行不同的逻辑，在一般组件数据更新的场景，会走到最后一个 `queueWatcher(this)` 逻辑

```js
// src/core/observer/scheduler.js
// ...
const queue: Array<Watcher> = []
let has: { [key: number]: ?true } = {}
let circular: { [key: number]: number } = {}
let waiting = false
let flushing = false
let index = 0
/**
 * Push a watcher into the watcher queue.
 * Jobs with duplicate IDs will be skipped unless it's
 * pushed when the queue is being flushed.
 */
// 将一个观察者对象 watcher 推送到观察者队列中，如果队列中已经存在相同的 id 则跳过该观察者，除非是在队列被刷新时推送
export function queueWatcher (watcher: Watcher) {
  // 获取 watcher id
  const id = watcher.id
  // 检验 watcher id，已经存在跳过该观察者，不存在则标记哈希 has 为 true，用于下次检验
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      // 如果没有 flush 掉，watcher 添加到 队列中
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      // 当 flushing 为 true，会从后往前找，找到第一个等待插入的 watcher id 比当前队列中 watcher id 大的位置，把 watcher 按照 id 插入到队列中，因此 queue 队列的长度发生了变化
      let i = queue.length - 1
      while (i >= 0 && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(Math.max(i, index) + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
      waiting = true
      // nextTick 后执行 flushSchedulerQueue
      nextTick(flushSchedulerQueue)
    }
  }
}
```

这里引入一个队列的概念，vue 派发更新的时候，并不会每次数据修改都触发 `watcher` 的回调，而是把这些 `watcher` 先添加都一个队列里，在 `nextTick` 后执行 `flushSchedulerQueue`。同时 id 重复的 `watcher` 不会被多次添加到队列中

```js
// src/core/observer/scheduler.js
// ...
/**
 * Flush both queues and run the watchers.
 */
// nextTick 的回调，在下一个 tick 时 flush 掉队列同时运行 watcher
function flushSchedulerQueue () {
  flushing = true

  // Sort queue before flush.
  // This ensures that:
  // 1. Components are updated from parent to child. (because parent is always
  //    created before the child)
  // 2. A component's user watchers are run before its render watcher (because
  //    user watchers are created before the render watcher)
  // 3. If a component is destroyed during a parent component's watcher run,
  //    its watchers can be skipped.
  /*
  给 queue 队列排序，这样可以保证：
  1.组件更新的顺序是从父组件到子组件，应为父组件总是比子组件先创建
  2.一个组件的 user watcher 比 render watcher 先运行，因为 user watcher 往往比 render watcher 更早创建
  3.如果一个组件在父组件 watcher 运行期间被销毁，它的 watcher 执行将会被跳过
  */
  queue.sort((a, b) => a.id - b.id)

  // do not cache length because more watchers might be pushed
  // as we run existing watchers
  // 这里不用 index = queue.length; index > 0; index-- 的方式是因为不要将 length 进行缓存，因为在执行处理现有 watcher 对象期间，更多的 watcher 可能会被 push 进 queue 队列中
  // 队列遍历 执行 watcher run 方法 更新视图
  for (index = 0; index < queue.length; index++) {
    const watcher = queue[index]
    const id = watcher.id
    // 将 has 的标记清除
    has[id] = null
    // 调用 watcher run 方法 更新视图
    watcher.run()
    // in dev build, check and stop circular updates.
    /*
      在测试环境中，检测 watch 是否在死循环中
      如
      watch: {
        test() { this.test++ }
      }
      持续执行了 100 次 watch 代表可能存在死循环
    */
    if (process.env.NODE_ENV !== 'production' && has[id] != null) {
      circular[id] = (circular[id] || 0) + 1
      if (circular[id] > config._maxUpdateCount) {
        warn(
          'You may have an infinite update loop ' + (
            watcher.user
              ? `in watcher with expression "${watcher.expression}"`
              : `in a component render function.`
          ),
          watcher.vm
        )
        break
      }
    }
  }

  // devtool hook
  /* istanbul ignore if */
  if (devtools && config.devtools) {
    devtools.emit('flush')
  }
  // 状态恢复
  resetSchedulerState()
}
```

`flushSchedulerQueue` **是下一个 tick 时的回调函数，主要目的是执行 `watcher` 的 `run` 方法，用来更新视图**

这里主要有个三个核心逻辑：

- 队列排序

`queue.sort((a, b) => a.id - b.id)` 对队列根据 id 从小到达进行排序，主要是保证：

1.组件更新的顺序是从父组件到子组件，应为父组件总是比子组件先创建
2.用户自定义的 `watcher` 比渲染 `watcher` 先运行，因为 user `watcher` 往往比渲染 `watcher` 更早创建
3.如果一个组件在父组件 `watcher` 运行期间被销毁，它的 `watcher` 执行将会被跳过，所以父组件的 `watcher` 应该先执行

- 队列遍历

对 `queue` 队列排序后，接着进行遍历，拿到对应 `watcher`，执行 `watcher.run()`。这里要注意，在遍历的时候每次都会对 `queue.length` 求值，因为在 `watcher.run()` 时，用户可能会再次添加新的 `watcher`，这样会再次执行 `queueWatcher`

```js
// src/core/observer/scheduler.js
export function queueWatcher (watcher: Watcher) {
  // 获取 watcher id
  const id = watcher.id
  // 检验 watcher id，已经存在跳过该观察者，不存在则标记哈希 has 为 true，用于下次检验
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      // 如果没有 flush 掉，watcher 添加到 队列中
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      // 当 flushing 为 true，会从后往前找，找到第一个等待插入的 watcher id 比当前队列中 watcher id 大的位置，把 watcher 按照 id 插入到队列中，因此 queue 队列的长度发生了变化
      let i = queue.length - 1
      while (i >= 0 && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(Math.max(i, index) + 1, 0, watcher)
    }
    // ...
  }
}
```

当 `flushing` 为 true，会从后往前找，找到第一个等待插入的 `watcher` id 比当前队列中 `watcher` id 大的位置，把 `watcher` 按照 id 插入到队列中，因此 `queue` 队列的长度发生了变化

- 状态恢复

`queueWatcher` 最后执行 `resetSchedulerState()`。逻辑就是清空 `watcher` 队列并把一些变量重置到初始值

```js
// src/core/observer/scheduler.js
/**
 * Reset the scheduler's state.
 */
function resetSchedulerState () {
  // 清空 watcher 队列
  queue.length = 0
  has = {}
  if (process.env.NODE_ENV !== 'production') {
    circular = {}
  }
  waiting = flushing = false
}
```

`watcher.run()` 方法执行先通过 `this.get()` 得到它的当前值，然后判断如果满足新旧值不等、新值是对象类型、`deep` 模式中的任意一个条件，执行 `watcher` 的回调

这里 `watcher` 的回调函数执行时候会把第一个和第二个参数传入新值 `value` 和旧值 `oldValue`, 因此在组件中添加自定义 `watcher` 的时候可以在回调函数的参数中拿到新旧值

对于渲染 `watcher`, 执行 `this.get()`，会执行 `getter` 方法，也就是传入的第二个参数 `() => vm._update(vm._render(), hydrating)`, 因此修改组件相关的响应式数据时，会触发组件重新渲染，重新执行 `patch` 过程

```js
// src/core/observer/watcher.js
export default class Watcher {
  // ...
  /**
   * Scheduler job interface.
   * Will be called by the scheduler.
   */
  run () {
    if (this.active) {
      const value = this.get()
      if (
        value !== this.value ||
        // Deep watchers and watchers on Object/Arrays should fire even
        // when the value is the same, because the value may
        // have mutated.
        isObject(value) ||
        this.deep
      ) {
        // set new value
        const oldValue = this.value
        this.value = value
        if (this.user) {
          try {
            this.cb.call(this.vm, value, oldValue)
          } catch (e) {
            process.env.NODE_ENV !== 'production' && warn(
              `Error in watcher "${this.expression}"`,
              this.vm
            )
            /* istanbul ignore else */
            if (config.errorHandler) {
              config.errorHandler.call(null, e, this.vm)
            } else {
              throw e
            }
          }
        } else {
          this.cb.call(this.vm, value, oldValue)
        }
      }
    }
  }
  // ...
}
```

### nextTick

#### JS 运行机制

**JS 执行是单线程的，它是基于事件循环**。事件循环大致分为以下几个步骤

- **所有同步任务都是在主线程上执行，形成一个执行栈（execution context stack）**
- **主线程之外，还存在一个“任务队列”（task queue）。只要异步任务有了运行结果，会在“任务队列”中放置一个事件**
- **一旦“执行栈”中所有同步任务执行完毕，系统会读取“任务队列”，看有哪些事件。那些对应的异步任务于是结束等待状态，进入执行栈开始执行**
- 主线程不断重复上述三个步骤

![](http://ony85apla.bkt.clouddn.com/18-6-14/25554080.jpg)

**主线程的执行过程就是一个 tick**, 所有的异步结果都是通过任务队列来调度。规范中规定 **异步 task 分为两大类，分别是 macro task（宏任务）和 micro task（微任务），并且每个 macro task（宏任务） 结束后，都要清空所有的 micro task（微任务）**

```js
for (macroTask of macroTaskQueue) {
  // 1. 处理当前 macro task 宏任务
  handleMacroTask()

  // 2. 处理全部 micro task 微任务
  for (microTask of microTaskQueue) {
    handleMicroTask(microTask)
  }
}
```

在浏览器环境中，常见的 macro task（宏任务） 有 `setTimout` `MessageChannel` `postMessage` `setImmediate` `Ajax`; 常见的 micro task（微任务）有 `MutationObserver` `Promise` `process.nextTick`

#### Vue 中 nextTick

`nextTick` 的目的就是产生一个回调函数加入 macro task 或 micro task 中，当执行栈执行完后调用该回调函数，起到异步触发（下一个 tick 触发）的目的

```js
// src/core/util/env.js
/**
 * Defer a task to execute it asynchronously.
 */
/*
  延迟一个任务使其异步执行，在下一个 tick 时执行，一个立即执行函数，返回一个 function
*/
export const nextTick = (function () {
  // 存放异步执行的回调
  const callbacks = []
  // 一个标记位，如果已经有 timerFunc 被推送到任务队列中则不需要重复推送
  let pending = false
  // 一个函数指针，指向函数被推送到任务队列中，等到主线程任务执行完，任务队列中的 timerFunc 被调用
  let timerFunc

  // 下一个 tick 时的回调
  function nextTickHandler () {
    // 一个标记位，标记等待状态（即函数已经被推入到任务队列或主线程，已经在等待当前栈执行完毕去执行）
    pending = false
    // 执行所有 callback
    const copies = callbacks.slice(0)
    callbacks.length = 0
    for (let i = 0; i < copies.length; i++) {
      copies[i]()
    }
  }

  // the nextTick behavior leverages the microtask queue, which can be accessed
  // via either native Promise.then or MutationObserver.
  // MutationObserver has wider support, however it is seriously bugged in
  // UIWebView in iOS >= 9.3.3 when triggered in touch event handlers. It
  // completely stops working after triggering a few times... so, if native
  // Promise is available, we will use it:
  /* istanbul ignore if */

  /*
    一共有 Promise、MutationObserver 和 setTimeout 三种尝试得到 timerFunc 的方法
    优先使用 Promise，在 Promise 不存在的情况下使用 MutationObserver，这两种方法都会在 micro task(微任务)中执行，会比 setTimout（属于 macro task 宏任务） 更早执行
    如果上述两种方法都不执行的环境会使用 setTimeout，在 macro task 尾部推入这个函数，等待调用执行
  */
  if (typeof Promise !== 'undefined' && isNative(Promise)) {
    var p = Promise.resolve() // 返回一个 promise
    timerFunc = () => {
      p.then(nextTickHandler)
      // in problematic UIWebViews, Promise.then doesn't completely break, but
      // it can get stuck in a weird state where callbacks are pushed into the
      // microtask queue but the queue isn't being flushed, until the browser
      // needs to do some other work, e.g. handle a timer. Therefore we can
      // "force" the microtask queue to be flushed by adding an empty timer.
      if (isIOS) setTimeout(noop)
    }
  } else if (typeof MutationObserver !== 'undefined' && (
    isNative(MutationObserver) ||
    // PhantomJS and iOS 7.x
    MutationObserver.toString() === '[object MutationObserverConstructor]'
  )) {
    // use MutationObserver where native Promise is not available,
    // e.g. PhantomJS IE11, iOS7, Android 4.4
    // 新建一个 textNode 的 dom 对象，用 MutationObserver 绑定该 dom 并执行回调函数，在 dom 变化时候回触发回调，该回调会进入主线程（比 macro task 优先执行）
    var counter = 1
    var observer = new MutationObserver(nextTickHandler)
    var textNode = document.createTextNode(String(counter))
    observer.observe(textNode, {
      characterData: true
    })
    timerFunc = () => {
      counter = (counter + 1) % 2
      textNode.data = String(counter)
    }
  } else {
    // fallback to setTimeout
    /* istanbul ignore next */
    // 使用 setTimeout 将回调推入 macro task 队列尾部
    timerFunc = setTimeout
  }

  /*
    推动到队列中下一个 tick 时执行
    cb 回调函数
    ctx 上下文
  */
  return function queueNextTick (cb: Function, ctx?: Object) {
    const func = ctx
      ? function () { cb.call(ctx) }
      : cb
    // cb 存到 callbacks 中
    callbacks.push(func)
    if (!pending) {
      pending = true
      timerFunc(nextTickHandler, 0)
    }
  }
})()
```

之前 `watcher.run()` 异步会执行 `queueWatcher(this)`, 最终会执行 `nextTick(flushSchedulerQueue)`。

`nextTick` 函数是一个立即执行函数，返回 `queueNextTick` 函数。`queueNextTick` 方法的逻辑很简单，就是将传入的 `cb` 放入 `callbacks` 数组，然后执行 `timerFunc`(pending 是一个状态标记，用于保证 timerFunc 在下一个 tick 之前只执行一次)

> **`timerFunc` 其实是一个异步函数的实现，在该异步函数内部遍历 `callbacks` 中的 `cb` 并执行 `cb`。对于不同的环境实现 `timerFunc`, 会按照支持 Promise、MutationObserver、setTimeout 的先后顺序（前两者都属于 micro task 微任务，比 属于 macro task 宏任务 的 setTimeout 执行快）来实现封装一个异步方法。`timerFunc` 作为异步回调最后会加入 micro task 或 macro task 中，在执行栈执行完毕后调用该回调函数，起到了异步触发（下一个 tick）的目的**

`nextTick(flushSchedulerQueue)` 最后不论内部是用 Promise MutationOBserver 还是 setTimeout 封装 `timerFunc`, 它们都会在下一个 tick 执行 `flushSchedulerQueue`, 也就是对 `callbacks` 遍历，执行对应的回调函数

### vue 中检测变化的问题

#### 响应式对象新添加的属性

vue 中使用 `Object.defineProperty` 实现对象的响应式，对于响应式对象对象直接添加新的属性时，因为没有添加 getter setter，是无法做到响应式。vue 提供了 `vm.$set(obj, key, value)` `Vue.set(obj, key, value)` 方法向响应式对象中添加一个属性，并确保这个新属性同样是响应式的，并且触发视图更新

```js
// src/core/observer/index.js
/**
 * Set a property on an object. Adds the new property and
 * triggers change notification if the property doesn't
 * already exist.
 */
export function set (obj: Array<any> | Object, key: any, val: any) {
  if (Array.isArray(obj)) {
    // splice 方法不只是原型数组的原型方法
    // 数组的话通过下标插入
    obj.splice(key, 1, val)
    return val
  }
  // 如果 key 已经存在直接复制返回
  if (hasOwn(obj, key)) {
    obj[key] = val
    return
  }
  const ob = obj.__ob__
  if (obj._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid adding reactive properties to a Vue instance or its root $data ' +
      'at runtime - declare it upfront in the data option.'
    )
    return
  }
  // 不存在，说明之前 obj 不是响应式对象
  if (!ob) {
    obj[key] = val
    return
  }
  // 调用 defineReactive 将对象设置为响应式
  defineReactive(ob.value, key, val)
  // 手动触发依赖通知
  ob.dep.notify()
  return val
}
```

`set` 方法接受三个参数，`obj` 可能是数组或者是普通对象，`key` 代表数组下标或者是对象的键值，`val` 添加的值

- 首先判断 `obj` 数组并且 `key` 是合法下标，则通过 `splice` 去添加进数组然后返回，注意这里的 `splice` 不仅仅是原生数组的原型方法
- 接着判断 `key` 已经存在则直接赋值返回
- 接着在获取 `obj.__ob__` 并赋值给 `ob`, 之前分析它是在 `Observer` 构造函数执行时候初始化，表示 `Observer` 的一个实例，如果它不存在，说明 `obj` 不是一个响应式对象，直接赋值返回
- 最后通过 `defineReactive(ob.value, key, val)` 把新添加的属性变成响应式对象，再通过 `ob.dep.notify()` 手动触发依赖通知

#### 数组

`Object.defineProperty` 无法检测到数组的变化，vue 内部下面对 7 中数组原型方法做了 hack，只支持：

- push
- pop
- shift
- unshift
- reverse
- sort
- splice

对于利用索引值直接设置一个项 `vm.items[index] = newVal`, 可以使用 `Vue.set(example.items, index, newVal)`; 对于修改数组的长度 `vm.items.length = newLength`, 可以使用 `vm.items.splice(newLength)`

```js
// src/core/observer/index.js
export class Observer {
  constructor (value: any) {
    // ...
    if (Array.isArray(value)) {
      // 判断对象是否有 __proto__, 存在 augment 指向 protoAugment，否则指向 copyAugment
      const augment = hasProto
        ? protoAugment
        : copyAugment
      augment(value, arrayMethods, arrayKeys)
      this.observeArray(value)
    } else {
      // ...
    }
  }
}

/**
 * Augment an target Object or Array by intercepting
 * the prototype chain using __proto__
 */
// const arrayKeys = Object.getOwnPropertyNames(arrayMethods)
// protoAugment(value, arrayMethods, arrayKeys)
function protoAugment (target, src: Object) {
  /* eslint-disable no-proto */
  target.__proto__ = src
  /* eslint-enable no-proto */
}

/**
 * Augment an target Object or Array by defining
 * hidden properties.
 *
 * istanbul ignore next
 */
// const arrayKeys = Object.getOwnPropertyNames(arrayMethods)
// copyAugment(value, arrayMethods, arrayKeys)
function copyAugment (target: Object, src: Object, keys: Array<string>) {
  for (let i = 0, l = keys.length; i < l; i++) {
    const key = keys[i]
    def(target, key, src[key])
  }
}
```

实例化 `Observer` 时，它的构造函数中对数组做了处理，首先通过 `hasProto` 判断对象中是否有 `__proto__`, 有则 `augment` 指向 `protoAugment`, 否则 `augment` 指向 `protoAugment`

`protoAugment` 方法直接将 `target.__proto__` 原型修改为 `src`; `copyAugment` 方法是遍历 keys，通过 `def`，也就是 `Object.defineProperty` 定义它自身的属性值。大部分现代浏览器都会执行 `protoAugment`, 它实际就是将 `value` 的原型指向 `arrayMethods`

```js
// src/core/observer/array.js
/*
 * not type checking this file because flow doesn't play well with
 * dynamically accessing methods on Array prototype
 */

import { def } from '../util/index'
// arrayMethods 继承 Array
const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)

/**
 * Intercept mutating methods and emit events
 */
;[
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
.forEach(function (method) {
  // cache original method
  const original = arrayProto[method]
  def(arrayMethods, method, function mutator () {
    // avoid leaking arguments:
    // http://jsperf.com/closure-with-arguments
    let i = arguments.length
    const args = new Array(i)
    while (i--) {
      args[i] = arguments[i]
    }
    // 原有方法先执行
    const result = original.apply(this, args)
    const ob = this.__ob__
    let inserted
    switch (method) {
      case 'push':
        inserted = args
        break
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    // 获取插入值，将添加值变成响应式对象
    if (inserted) ob.observeArray(inserted)
    // notify change
    // 手动触发依赖通知
    ob.dep.notify()
    return result
  })
})

// src/core/observer/index.js
/**
   * Observe a list of Array items.
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
```

`arrayMethods` 首先继承 `Array`，然后对 `push` `pop` `shift` `unshift` `splice` `sort` `reverse` 7 种方法进行重写，**重写的方法先执行原有逻辑，并对增加数组长度的三种方法 `push` `pop` `splice` 进行判断，获取插入的值，然后将新添加的值变成一个响应式对象，并且调用 `ob.dep.notify()` 手动触发依赖通知**

### computed

计算属性一般只用到 getter，但是计算属性并不是简单的 getter，它会更新它的依赖列表并缓存结果。只要依赖不发生变化，访问计算属性会直接返回缓存的结果，而不是调用 getter

vue 实例初始化阶段 `initState` 方法初始化了 `data` `props` `computed` `watch` `methods`

**整个计算属性的初始化就是利用 `Object.defineProperty` 为计算属性对应 key 值添加 getter setter。初始化 `computed` 时候，会先将 `computed` 对应 key 的函数先创建一个 `watcher`, 再对该计算属性使用 `Object.defineProperty` 为计算属性对应 key 添加 getter setter，`watcher` 作为该 key 的 getter**。setter 在平常的开发场景下情况比较少，也不推荐使用。

```js
// src/core/instance/state.js
// ...
export function initState (vm: Component) {
  vm._watchers = []
  initProps(vm)
  initData(vm)
  initComputed(vm)
  initMethods(vm)
  initWatch(vm)
}

const computedSharedDefinition = {
  enumerable: true,
  configurable: true,
  get: noop,
  set: noop
}

function initComputed (vm: Component) {
  const computed = vm.$options.computed
  if (computed) {
    // 遍历 computed 对象
    for (const key in computed) {
      // 拿到计算属性的每一个 userDef
      const userDef = computed[key]
      if (typeof userDef === 'function') {
        // 如果 userDep 不是 function（认为是 getter 函数）, 用 userDep 创建一个 watcher
        computedSharedDefinition.get = makeComputedGetter(userDef, vm)
        computedSharedDefinition.set = noop
      } else {
        // 如果 userDep 不是 function， 尝试获取 userDef 对应的 getter 函数，为该 getter 函数创建一个 watcher
        computedSharedDefinition.get = userDef.get
          ? userDef.cache !== false
            ? makeComputedGetter(userDef.get, vm)
            : bind(userDef.get, vm)
          : noop
        computedSharedDefinition.set = userDef.set
          ? bind(userDef.set, vm)
          : noop
      }
      // 为计算属性对应 key 添加 getter 和 setter
      Object.defineProperty(vm, key, computedSharedDefinition)
    }
  }
}

function makeComputedGetter (getter: Function, owner: Component): Function {
  // 参数 getter 为 用户自定义的计算属性 get 方法
  // 参数 owner 为 当前 vm 实例
  const watcher = new Watcher(owner, getter, noop, {
    // 表明该 watcher 的求值被延迟，不会在初始化时求值
    // 设置 lazy 为 true，也就是设置 dirty 为 true
    lazy: true
  })
  // 返回 computedGetter 方法，访问 vm 上的计算属性会调用该方法
  return function computedGetter () {
    // 当 watcher.dirty 为 true，会调用 watcher.evaluate 对 watcher 进行计算
    if (watcher.dirty) {
      // watcher.evaluate 方法通过 this.get 方法对 watcher 进行求值，接着设置 this.dirty 为 false，当下次访问 vm 上的计算属性时，watcher.dirty 为 false，就不会再次对 watcher 进行求值，因此也不再次访问计算属性的 getter
      watcher.evaluate()
    }
    // 依赖收集
    // 当计算属性的依赖改变时，会触发依赖的 setter，也就会调用 watcher 的 update 方法，将 watcher 的 dirty 属性设置 true。当访问计算属性时会对 watcher 重新求值
    if (Dep.target) {
      watcher.depend()
    }
    return watcher.value
  }
}

// src/core/observer/watcher.js
export default class Watcher {
  // ...

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: Object = {}
  ) {
    // ...
    // computed watcher 初始化设置 this.dirty 为 true
    this.dirty = this.lazy // for lazy watchers
    // ...
    // computed watcher 初始化不会立即求值，但同时持有一个 dep 实例
    this.value = this.lazy
      ? undefined
      : this.get()
  }

  // ...

  update () {
    /* istanbul ignore else */
    if (this.lazy) {
      // lazy watcher 设置 dirty 为 true
      this.dirty = true
    } else if (this.sync) {
      // 同步调用 run 方法
      this.run()
    } else {
      // 异步放入 watcher 队列
      queueWatcher(this)
    }
  }
  
  evaluate () {
    // 通过 this.get 方法对 watcher 进行求值, 同时设置 this.dirty 为 false
    this.value = this.get()
    this.dirty = false
  }
  // ...
}
```

`initComputed` 方法对 `computed` 属性进行遍历，拿到每个计算属性的定义。计算属性的定义可以是一个 function，也可以是一个 object。默认是一个 function，只有 get 方法，如果想设置计算属性的 set 方法或设置 cache 为 false，则要把计算属性顶一个 object。计算属性的 get 方法默认是通过 `makeComputedGetter` 方法实现，除非设置 cache 为 false。最后通过 `Object.defineProperty(this, key, def)` 方法吧每个计算属性绑定到 vm 实力上，访问 vm 上的计算属性会调用 `def.get` 方法

而初始化该 `computed watcher` 实例时，`computed watcher` 不会立即求值。`computed watcher` 初始化设置 `lazy` 为 true，也就是设置 `dirty` 为 true。第一次访问 vm 上的计算属性，会调用计算属性的 getter，此时 `watcher.drity` 为 true。**`watcher.evaluate` 方法通过 `this.get` 方法对 `watcher` 进行求值，接着设置 `this.dirty` 为 false，当下次访问 vm 上的计算属性时，`watcher.dirty` 为 false，就不会再次对 `watcher` 进行求值，因此也不再次访问计算属性的 getter**

为何修改计算属性的依赖时候，getter 方法会再次被调用？**因为在第一次调用计算属性的 getter 时，`watcher.depend()` 会将当前 watcher 订阅到依赖中。当依赖被修改时。会触发依赖的 setter 方法，也就会调用 `watcher` 的 `update` 方法。因为 `computed watcher` 是 `lazy watcher` ，因此这时 `watcher` 的 `dirty` 属性设置 true。当再次访问计算属性，会重新调用 `watcher.evaluate` 方法对 watcher 进行求值，因此计算属性的 getter 方法被再次调用**

## 模板编译

### $mount

`new Vue()` 调用了构造函数中的 `this._init()`, `this._init()` 最后调用了 `initRender()`,  `initRender()` 最后执行 `vm.$mount()`

`$mount` 方法在多个文件中都有定义，`src/entries/web-runtime-with-compiler.js`、`src/entries/web-runtime.js` 等。我们重点分析带有 `compiler` 版本的 `$mount` 实现

```js
new Vue({
  el: '#app'
})

// 等同于
new Vue({}).$mount('#app')
```

```js
// src/entries/web-runtime-with-compiler.js
const mount = Vue.prototype.$mount
Vue.prototype.$mount = function (
  el?: string | Element, 
  // 可以是 css 选择器也可以是 HTML 元素（dom）
  /*
    index.html 模板有 <div id="#app"></div>；入口文件 new Vue({el: '#app'}) 等同于 new Vue({}).$mount('#app')
    或者
    var div = document.createElement('div'); div.id = '#app';document.body.appendChild(div);
    new Vue({el: div}) 等同 new Vue({}).$mount(div)
  */
  hydrating?: boolean
): Component {
  el = el && query(el)

  /* istanbul ignore if */
  // el 不能挂载在 html body 这样的根节点上
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }

  const options = this.$options
  // resolve template/el and convert to render function
  // 如果 vue 实例选项中 render 属性不存在，将 template / el 转化为 render 函数
  if (!options.render) {
    let template = options.template
    let isFromDOM = false
    // 如果 vue 实例选项中存在 template 属性
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          isFromDOM = true
          template = idToTemplate(template)
        }
      } else if (template.nodeType) {
        isFromDOM = true
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      // 如果 vue 实例选项中 template 属性不存在，将 el 的 outerHTMl 作为 模板
      isFromDOM = true
      template = getOuterHTML(el)
    }
    if (template) {
      // 通过 compileToFunctions 方法将 template （无论 template 属性值 还是 el 的 outerHTML）转为 render 函数
      const { render, staticRenderFns } = compileToFunctions(template, {
        warn,
        isFromDOM,
        shouldDecodeTags,
        shouldDecodeNewlines,
        delimiters: options.delimiters
      }, this)
      // 将 render 函数挂载到 vm.$options.render 属性
      options.render = render
      options.staticRenderFns = staticRenderFns
    }
  }
  return mount.call(this, el, hydrating)
}
```

上面的代码首先缓存原先原型上的 `$mount` 方法（`src/entries/web-runtime.js` 中定义），再重新定义该方法

- 首先对 `el` 做出限制，不能挂载在 `body`、`html` 这样的根节点上
- 核心逻辑是，对于没有定义 `render` 方法，会将 `template` 字符串，如果还没有 `template` 属性，会将 `el` 的 outerHTML 转为 `render` 函数

> **Vue 2.0 中所有 vue 组件的渲染都需要 render 函数，不管是单文件组件 .vue 文件，还是 el 或者 template 属性，最终都会通过 compileToFunctions 方法转为 render 函数**

```js
// src/entries/web-runtime.js
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && !config._isServer ? query(el) : undefined
  // 调用 vm._mount
  return this._mount(el, hydrating)
}
```

原先原型上的 `$mount` 方法，第一个参数是 `el`, 在浏览器环境下回调用 `query` 方法转换为 dom 对象，第二个参数和服务端渲染有关，在浏览器环境下不需要传递

`$mount` 方法实际调用 `src/core/instance/lifecycle.js` 下的 `_mount` 方法

```js
// core/instance/lifecycle.js
// ...
Vue.prototype._mount = function (
  el?: Element | void,
  hydrating?: boolean
): Component {
  // ...
  callHook(vm, 'beforeMount')
  // 初始化渲染 watcher，
  vm._watcher = new Watcher(vm, () => {
    // 调动 vm._render, 通过 render 函数返回 vnode
    // 首次渲染 和 每次数据属性更新会执行该 watcher 中的回调，执行 vm._update() 对 vnode 进行 diff 和 patch
    vm._update(vm._render(), hydrating)
  }, noop)
  hydrating = false
  // root instance, call mounted on self
  // mounted is called for child components in its inserted hook
  if (vm.$root === vm) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
// ...
```

`_mount` 方法核心先调用 `vm._render` 方法先生成 vnode，再实例化一个渲染 `watcher`，在它的回调函数中将 `vm._render` 返回的 vnode 传入 `vm._update` 调用更新 dom

> `watcher` 在这里起到两个作用
- **初始化会执行回调函数**
- **当 vm 实例中监听的数据发生变化时会执行回调函数**

函数最后判断为根节点的时候设置 `vm._isMounted` 为 `true`, 表示实例已经挂载，同时执行 `mounted` 钩子函数

### 编译 render 函数

#### render 函数概念

`render 函数` 是通过编译模板(`template` 或 `el` 的 outerHTML)得到，其运行结果是 vnode

```html
<div id="app">
  <header>
    <h1>I am a template!</h1>
  </header>
  <p v-if="message">{{ message }}</p>
  <p v-else>No message.</p>
</div>
```

`Vue.compile(template)` 编译上面的模板会返回一个对象，对象中含有 `render` 和 `staticRenderFns` 两个值

```js
// 生成的 render 函数
(function() {
  with(this){
    return _c( // _c 为 createElement 方法(创建元素)
      'div', // 创建一个 div 元素
      {
        attrs:{"id":"app"} // div 添加属性 id
      },
      [ // _m 为 renderStatic 方法（渲染静态节点）
        _m(0), // 这里为静态节点 header，此处对应 staticRenderFns 数组索引为 0 的 render 函数
        _v(" "), // _v 为 createTextNode 方法(创建文本 dom) 这里为空的文本节点
        (message) // 三元表达式，判断 message 是否存在
          // 存在就创建 p 元素，元素里有文本，值为 toString(message)
          ? _c('p',[_v(_s(message))]) // _s 为 toString 方法（转为字符串）
          // 不存在就创建 p 元素，元素里文本值为 No message.
          :_c('p',[_v("No message.")])
      ]
    )
  }
})
```

除了 `render` 函数，还有一个 `staticRenderFns` 数组，这个数组中的函数和 vnode 中 diff 算法优化有关，会在比编译阶段给之后不会发生变化的 vnode 打上 `static` 为 `true` 的标签，那些被标记为静态节点的 vnode 会单独生成 `staticRenderFns` 函数

```js
(function() { //上面 render 函数 中的 _m(0) 会调用这个方法
  with(this){
    return _c(
      'header',
      [
        _c(
          'h1',
          [
            _v("I'm a template!")
          ]
        )
      ]
    )
  }
})  
```

#### 编译 render 函数过程

```js
// src/entries/web-runtime-with-compiler.js
// ...
const { render, staticRenderFns } = compileToFunctions(template, {
  warn,
  isFromDOM,
  shouldDecodeTags,
  shouldDecodeNewlines,
  delimiters: options.delimiters
}, this)
options.render = render
options.staticRenderFns = staticRenderFns
```

`src/entries/web-runtime-with-compiler.js` 中 `$mount` 方法通过调用  `compileToFunctions` 方法, 把模板 `template` 编译生成 `render` 以及 `staticRenderFns`

```js
// src/platforms/web/compiler/index.js
// ...
import { compile as baseCompile } from 'compiler/index'
// ...
export function compile (
  template: string,
  options?: CompilerOptions
): CompiledResult {
  options = options
    ? extend(extend({}, baseOptions), options)
    : baseOptions
  // 合并参数，返回
  return baseCompile(template, options)
}

export function compileToFunctions (
  template: string,
  options?: CompilerOptions,
  vm?: Component
): CompiledFunctionResult {
  const _warn = (options && options.warn) || warn
  // detect possible CSP restriction
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production') {
    try {
      new Function('return 1')
    } catch (e) {
      if (e.toString().match(/unsafe-eval|CSP/)) {
        _warn(
          'It seems you are using the standalone build of Vue.js in an ' +
          'environment with Content Security Policy that prohibits unsafe-eval. ' +
          'The template compiler cannot work in this environment. Consider ' +
          'relaxing the policy to allow unsafe-eval or pre-compiling your ' +
          'templates into render functions.'
        )
      }
    }
  }
  const key = options && options.delimiters
    ? String(options.delimiters) + template
    : template
  if (cache[key]) {
    return cache[key]
  }
  const res = {}
  // 传入 template，调用 compile 方法编译
  const compiled = compile(template, options)
  // 得到 render 函数的字符串后，通过 new Function 得到真正的渲染函数
  res.render = makeFunction(compiled.render)
  const l = compiled.staticRenderFns.length
  res.staticRenderFns = new Array(l)
  for (let i = 0; i < l; i++) {
    res.staticRenderFns[i] = makeFunction(compiled.staticRenderFns[i])
  }
  if (process.env.NODE_ENV !== 'production') {
    if (res.render === noop || res.staticRenderFns.some(fn => fn === noop)) {
      _warn(
        `failed to compile template:\n\n${template}\n\n` +
        detectErrors(compiled.ast).join('\n') +
        '\n\n',
        vm
      )
    }
  }
  return (cache[key] = res)
}
function makeFunction (code) {
  try {
    // 将函数字符串生成函数并返回
    return new Function(code)
  } catch (e) {
    return noop
  }
}
```

`compileToFunctions` 方法接受三个参数：编译模板 `template`、编译配置 `options`。核心的编译过程就是 `const compiled = compile(template, options)`

`compile` 函数执行的逻辑是先处理配置参数，真正的编译过程是 `return baseCompile(template, options)`

`baseCompile` 方法在 `src/compiler/index.js`

```js
// src/compiler/index.js
export function compile (
  template: string,
  options: CompilerOptions
): CompiledResult {
  // 解析模板字符串生成 AST 树
  const ast = parse(template.trim(), options)
  // 优化语法树
  optimize(ast, options)
  // 生成代码
  const code = generate(ast, options)
  // 输出包含 AST、render 函数字符串、staticRenderFns 字符串的对象
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
}
```

**`parse` 函数（`src/compiler/parse/index.js` 中定义）主要作用是将模板字符串 `template` 里的结构(指令、属性、标签等)转为 AST 形式存进 ASTElement 中，最后解析生成 AST**。整个 parse 的过程是利用**正则表达式顺序解析模板**，当解析到开始标签、闭合标签、文本的时候都会分别执行对应的回调函数，来达到构造 AST 树的目的。AST 元素节点总共有 3 种类型，type 为 1 表示是普通元素，为 2 表示是表达式，为 3 表示是纯文本。

**`optimize` 函数（`src/compiler/optimizer.js` 中定义）主要功能就是标记静态节点，为后面的 `patch` 过程对比新旧 vnode 结构做优化。被标记为 static 的节点在后面的 diff 算法被直接忽略，不做详细比较** (vue 是数据驱动，响应式的，但是应用中的数据并不全是响应式，很多数据是首次渲染就永远不会变化，那么这部分数据生成的 dom 也不会变化，可以再 `diff` `patch` 过程跳过对比)

**`generate` 函数主要功能就是根据 AST 结构拼接生成 `render 函数` 的字符串**

> 总结：render 函数编译过程

- 解析模板字符串（`template` 、`el` outerHTML）生成 AST `const ast = parse(template.trim(), options)`
- 优化语法树  `optimize(ast, options)`
- 生成代码  `const code = generate(ast, options)`

## 虚拟 dom

vue 2.0 引入 虚拟 dom，算法来源于 snabbdom。dom 的操作是十分昂贵的，虚拟 dom 对 dom 做了一层映射，将直接对 dom 的一系列操作，映射到操作虚拟 dom。虚拟 dom 上定义了真实 dom 上的一些关键信息

### vnode

首先来看一下 `src/core/vdom/vnode.js` 中 vnode 的定义

```js
// VNodeData 类型定义
export interface VNodeData {
  key?: string | number;
  slot?: string;
  ref?: string;
  tag?: string;
  staticClass?: string;
  class?: any;
  style?: Object[] | Object;
  props?: { [key: string]: any };
  attrs?: { [key: string]: any };
  domProps?: { [key: string]: any };
  hook?: { [key: string]: Function };
  on?: { [key: string]: Function | Function[] };
  nativeOn?: { [key: string]: Function | Function[] };
  transition?: Object;
  show?: boolean;
  inlineTemplate?: {
    render: Function;
    staticRenderFns: Function[];
  };
  directives?: VNodeDirective[];
  keepAlive?: boolean;
}

// src/core/vdom/vnode.js
export default class VNode {
  // ...
  constructor (
    tag?: string,
    data?: VNodeData,
    children?: Array<VNode> | void,
    text?: string,
    elm?: Node,
    ns?: string | void,
    context?: Component,
    componentOptions?: VNodeComponentOptions
  ) {
    // 当前节点的标签名
    this.tag = tag
    // 当前节点对应的对象，包含具体的一些数据，VNodeData 类型
    this.data = data
    // 当前节点的子节点，是一个数组
    this.children = children
    // 当前节点的文本
    this.text = text
    // 当前虚拟节点对应的真实 dom 节点
    this.elm = elm
    // 当前节点的命名空间
    this.ns = ns
    // 当前节点的编译作用域
    this.context = context
    // 节点的 key 属性，被当做节点的标识，用于优化
    this.key = data && data.key
    // 组件的 options 选项
    this.componentOptions = componentOptions
    this.child = undefined
    // 当前节点的父节点
    this.parent = undefined
    // 是否是原生 html 还是普通文本，innerHTML 时候为 true，textContent 为 false
    this.raw = false
    // 是否为静态节点
    this.isStatic = false
    // 是否作为根节点插入
    this.isRootInsert = true
    // 是否为注释节点
    this.isComment = false
    // 是否为克隆节点
    this.isCloned = false
  }
}
// ...
```

每一个 vnode 都会映射到一个真实 dom 节点，其中有几个比较重要的属性：

- `tag` vnode 的标签属性
- `data` 包含了最终渲染成真实 dom 节点上的 class attribute style 以及绑定的事件等
- `children` vnode 的子节点，是一个数组
- `text` 文本属性
- `elm` vnode 对应的真实 dom 节点
- `key` vnode 的标记，有利于优化

**vnode 简单理解就是使用 js 对象来描述一个 dom 结构**

```js
{
  tag: 'div',
  data: {
    id: 'app',
    class: 'main'
  },
  children: [{
    tag: 'p',
    text: 'hello'
  }]
}
```

最终渲染成的 dom

```html
<div id="app" class="main">
  <p>hello</p>
</div>
```

### createElement

vue 中使用 `createElement` 方法创建 vnode

```js
// src/core/vdom/create-element.js
// ...
// wrapper function for providing a more flexible interface
// without getting yelled at by flow
export function createElement (
  tag: any,
  data: any,
  children: any
): VNode | void {
  if (data && (Array.isArray(data) || typeof data !== 'object')) {
    children = data
    data = undefined
  }
  // make sure to use real instance instead of proxy as context
  return _createElement(this._self, tag, data, children)
}

function _createElement (
  context: Component,
  tag?: string | Class<Component> | Function | Object,
  data?: VNodeData,
  children?: VNodeChildren | void
): VNode | void {
  if (data && data.__ob__) {
    process.env.NODE_ENV !== 'production' && warn(
      `Avoid using observed data object as vnode data: ${JSON.stringify(data)}\n` +
      'Always create fresh vnode data objects in each render!',
      context
    )
    return
  }
  if (!tag) {
    // in case of component :is set to falsy value
    return emptyVNode()
  }
  if (typeof tag === 'string') {
    // tag 为字符串
    let Ctor
    const ns = config.getTagNamespace(tag)
    if (config.isReservedTag(tag)) {
      // platform built-in elements
      // 内置的节点直接创建一个普通的 vnode
      return new VNode(
        tag, data, normalizeChildren(children, ns),
        undefined, undefined, ns, context
      )
    } else if ((Ctor = resolveAsset(context.$options, 'components', tag))) {
      // 如果是已经注册的组件名，通过 createComponent 创建一个组件类型 vnode
      // component
      return createComponent(Ctor, data, context, children, tag)
    } else {
      // unknown or unlisted namespaced elements
      // check at runtime because it may get assigned a namespace when its
      // parent normalizes children
      // 否则创建一个未知类型标签的 vnode
      return new VNode(
        tag, data, normalizeChildren(children, ns),
        undefined, undefined, ns, context
      )
    }
  } else {
    // tag 为 Component 类型 调用 createComponent，创建一个组件类型的 vnode
    // direct component options / constructor
    return createComponent(tag, data, context, children)
  }
}
```

`createElement` 实际调用 `_createElement` 方法，传入 4 个参数，`context` 表示 vnode 的上下文环境，是 `Component` 类型；`tag` 表示标签，可以使一个字符串也可以是一个 `Component`; `data` 表示 vnode 的数据，是一个 `VNodeData` 类型；`children` 表示当前 vnode 子节点，vnode 元素的数组

`_createElement` 根据 `tag` 不同类型做判断，如果是 `string` 类型，则接着判断如果是内置的一些节点，则 `new Vnode()` 直接创建一个普通 vnode，如果是为已注册的组件名，则通过 `createComponent` 创建一个组件类型的 vnode，否则创建一个未知的标签的 vnode。 如果是 `tag` 一个 `Component` 类型，则直接调用 `createComponent` 创建一个组件类型的 vnode 节点

注意 vnode 的子节点也应该是 vnode 类型，因此 `_createElement` 接受的第三个参数 children 也要规范为 vnode 类型，需要调用 `normalizeChildren` 方法

```js
// src/core/vdom/helpers/normalzie-children.js
export function normalizeChildren (
  children: any,
  ns: string | void,
  nestedIndex: number | void
): Array<VNode> | void {
  if (isPrimitive(children)) {
    // render 函数 children 只有一个节点，如果是基础类型，调用 createTextVNode 创建单个简单的文本节点
    return [createTextVNode(children)]
  }
  if (Array.isArray(children)) {
    // 遍历 children，获得单个节点 c，对 c 的类型进行判断
    const res = []
    for (let i = 0, l = children.length; i < l; i++) {
      const c = children[i]
      const last = res[res.length - 1]
      //  nested
      if (Array.isArray(c)) {
        // c 为数组，递归调用 normalizeChildren，并 push 进入 结果数组
        res.push.apply(res, normalizeChildren(c, ns, i))
      } else if (isPrimitive(c)) {
        // c 是基础类型，调用 createTextNode 转为 vnode 类型，并 push 进入 结果数组
        if (last && last.text) {
          last.text += String(c)
        } else if (c !== '') {
          // convert primitive to vnode
          res.push(createTextVNode(c))
        }
      } else if (c instanceof VNode) {
        // 已经是 vnode 类型，如果 children 存在嵌套，会根据 nestedIndex 去更新它的 key
        if (c.text && last && last.text) {
          // 对连续的两个 text 节点会合并为一个 text 节点
          last.text += c.text
        } else {
          // inherit parent namespace
          if (ns) {
            applyNS(c, ns)
          }
          // default key for nested array children (likely generated by v-for)
          if (c.tag && c.key == null && nestedIndex != null) {
            c.key = `__vlist_${nestedIndex}_${i}__`
          }
          res.push(c)
        }
      }
    }
    return res
  }
}
```

`normalizeChilren` 方法的调用场景有 2 种，一种是用户自定义 `render` 函数，当 `children` 只有一个节点，用户可以写成基础类型来创建单个简单的文本节点，这种情况下调用 `createTextVNode` 创建一个文本节点的 vnode；另一个场景是当编译 `slot` `v-for` 产生嵌套数组的时候

`normalizeChilren` 的主要逻辑是遍历 `children`, 获得单个节点 `c` 并判断它的类型，如果 `c` 是一个数组，递归调用 `normalizeChildren`，并 push 进入 结果数组; `c` 是基础类型，调用 `createTextNode` 转为 vnode 类型，并 push 进入 结果数组; `c` 已经是 vnode 类型，如果 `children` 存在嵌套，会根据 nestedIndex 去更新它的 key, 同时对连续的两个 `text` 节点会合并为一个 text 节点

### update 首次渲染

`vm._update` 被调用的时机有 2 个，一个是首次渲染，另一个是数据更新的时候。先看首次渲染

```js
// src/core/instance/lifecycle.js
// ...
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    if (vm._isMounted) {
      callHook(vm, 'beforeUpdate')
    }
    const prevEl = vm.$el
    const prevActiveInstance = activeInstance
    activeInstance = vm
    const prevVnode = vm._vnode
    vm._vnode = vnode
    if (!prevVnode) {
      // preVnode 不存在说明首次渲染
      // Vue.prototype.__patch__ is injected in entry points
      // based on the rendering backend used.
      // vm.$el 为真实 dom，vnode 创建一个新的 dom 替换 vm.$el
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating)
    } else {
      // 数据更新
      // 参数都是 vnode
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    activeInstance = prevActiveInstance
    // update __vue__ reference
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
    if (vm._isMounted) {
      callHook(vm, 'updated')
    }
  }
```

`vm._update()` 中关键的是 `vm.__patche__()` 方法，也是虚拟 dom 中最核心的方法，主要是完成 prevVnode 和 vnode 的 diff 过程并根据需要操作的 vdom 节点 patch，最后生成新的真实 dom 节点并完成视图的更新工作

```js
Vue.prototype.__patch__ = config._isServer ? noop : patch

// platforms/web/runtime/patch.js
import { createPatchFunction } from 'core/vdom/patch'
// ...
export const patch: Function = createPatchFunction({ nodeOps, modules })
```

`vm.__patch__()` 最终是调用 `src/core/vdom/patch.js` 中的 `createPatchFunction` 方法。`createPatchFunction` 方法内部定义了一系列的辅助方法，最终返回一个 `patch` 方法，该方法赋值给 `vm._update` 调用的 `vm.__patch__`

```js
// src/core/vdom/patch.js
// ...
const hooks = ['create', 'update', 'postpatch', 'remove', 'destroy']

export function createPatchFunction (backend) {
  let i, j
  const cbs = {}

  const { modules, nodeOps } = backend

  for (i = 0; i < hooks.length; ++i) {
    cbs[hooks[i]] = []
    for (j = 0; j < modules.length; ++j) {
      if (modules[j][hooks[i]] !== undefined) cbs[hooks[i]].push(modules[j][hooks[i]])
    }
  }
  // ...

  return function patch (oldVnode, vnode, hydrating, removeOnly) {
    let elm, parent
    let isInitialPatch = false
    const insertedVnodeQueue = []
    // 首次渲染 oldVnode 为 vm.$el 是真实 dom
    if (!oldVnode) {
      // empty mount, create new root element
      isInitialPatch = true
      createElm(vnode, insertedVnodeQueue)
    } else {
      const isRealElement = isDef(oldVnode.nodeType)
      if (!isRealElement && sameVnode(oldVnode, vnode)) {
        patchVnode(oldVnode, vnode, insertedVnodeQueue, removeOnly)
      } else {
        if (isRealElement) {
          // mounting to a real element
          // check if this is server-rendered content and if we can perform
          // a successful hydration.
          if (oldVnode.nodeType === 1 && oldVnode.hasAttribute('server-rendered')) {
            oldVnode.removeAttribute('server-rendered')
            hydrating = true
          }
          if (hydrating) {
            if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
              invokeInsertHook(vnode, insertedVnodeQueue, true)
              return oldVnode
            } else if (process.env.NODE_ENV !== 'production') {
              warn(
                'The client-side rendered virtual DOM tree is not matching ' +
                'server-rendered content. This is likely caused by incorrect ' +
                'HTML markup, for example nesting block-level elements inside ' +
                '<p>, or missing <tbody>. Bailing hydration and performing ' +
                'full client-side render.'
              )
            }
          }
          // either not server-rendered, or hydration failed.
          // create an empty node and replace it
          oldVnode = emptyNodeAt(oldVnode)
        }
        elm = oldVnode.elm
        parent = nodeOps.parentNode(elm)

        createElm(vnode, insertedVnodeQueue)

        // component root element replaced.
        // update parent placeholder node element.
        if (vnode.parent) {
          vnode.parent.elm = vnode.elm
          if (isPatchable(vnode)) {
            for (let i = 0; i < cbs.create.length; ++i) {
              cbs.create[i](emptyNode, vnode.parent)
            }
          }
        }

        if (parent !== null) {
          nodeOps.insertBefore(parent, vnode.elm, nodeOps.nextSibling(elm))
          removeVnodes(parent, [oldVnode], 0, 0)
        } else if (isDef(oldVnode.tag)) {
          invokeDestroyHook(oldVnode)
        }
      }
    }

    invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
    return vnode.elm
  }
}
```

#### 过程分析

```js
new Vue({
  el: '#app',
  render: function (createElement) {
    return createElement('div', {
      attrs: {
        id: 'app'
      },
    }, this.message)
  },
  data: {
    message: 'Hello Vue!'
  }
})
```

对于上面的例子，`vm._update` 方法会执行首次渲染流程 `vm.$el = vm.__patch__(vm.$el, vnode, hydrating)`, 这里 `vm.$el` 为 index.html 模板里的 `<div id="app"></div>` 的 dom 对象，`vnode` 为调用 `render` 函数的返回值，`hydrating` 在非服务端渲染情况下为 false

进入 `patch` 函数，重点分析一个关键步骤

```js
const isRealElement = isDef(oldVnode.nodeType)
if (!isRealElement && sameVnode(oldVnode, vnode)) {
  patchVnode(oldVnode, vnode, insertedVnodeQueue, removeOnly)
} else {
  // 首次渲染 oldVnode 一般为一个 dom container，是真实 dom
  // isRealElement 为 true
  if (isRealElement) {
    // mounting to a real element
    // check if this is server-rendered content and if we can perform
    // a successful hydration.
    // 服务端渲染逻辑
    if (oldVnode.nodeType === 1 && oldVnode.hasAttribute('server-rendered')) {
      oldVnode.removeAttribute('server-rendered')
      hydrating = true
    }
    if (hydrating) {
      if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
        invokeInsertHook(vnode, insertedVnodeQueue, true)
        return oldVnode
      } else if (process.env.NODE_ENV !== 'production') {
        warn(
          'The client-side rendered virtual DOM tree is not matching ' +
          'server-rendered content. This is likely caused by incorrect ' +
          'HTML markup, for example nesting block-level elements inside ' +
          '<p>, or missing <tbody>. Bailing hydration and performing ' +
          'full client-side render.'
        )
      }
    }
    // either not server-rendered, or hydration failed.
    // create an empty node and replace it
    // 不是服务端渲染，首次渲染 oldVnode 一般为一个 dom container，是真实 dom, 这里就是将 oldVnode 转为 vnode
    // 原先 oldVNode 指向的 dom 作为 vnode 的 elm 属性值
    oldVnode = emptyNodeAt(oldVnode)
    /*
    function emptyNodeAt (elm) {
      return new VNode(nodeOps.tagName(elm).toLowerCase(), {}, [], undefined, elm)
    }
    */
  }
  elm = oldVnode.elm // 将原先 oldVnode 的 elm 值也就是真实 dom 赋给变量 elm
  parent = nodeOps.parentNode(elm) // 首次渲染，parent 一般是 body

  createElm(vnode, insertedVnodeQueue)

  // component root element replaced.
  // update parent placeholder node element.
  // 新 vnode 去掉符父占位符元素
  if (vnode.parent) {
    vnode.parent.elm = vnode.elm
    if (isPatchable(vnode)) {
      for (let i = 0; i < cbs.create.length; ++i) {
        cbs.create[i](emptyNode, vnode.parent)
      }
    }
  }

  if (parent !== null) {
    // 插入到它的父节点中，原生 dom 操作
    nodeOps.insertBefore(parent, vnode.elm, nodeOps.nextSibling(elm))
    // 删除 oldVnode （vnode 已经生成 dom 插入文档，要把 oldVnode 删除）
    removeVnodes(parent, [oldVnode], 0, 0)
    /*
    function removeVnodes (parentElm, vnodes, startIdx, endIdx) {
      for (; startIdx <= endIdx; ++startIdx) {
        const ch = vnodes[startIdx]
        if (isDef(ch)) {
          if (isDef(ch.tag)) {
            removeAndInvokeRemoveHook(ch)
            invokeDestroyHook(ch)
          } else { // Text node
            nodeOps.removeChild(parentElm, ch.elm)
          }
        }
      }
    }
    */
  } else if (isDef(oldVnode.tag)) {
    invokeDestroyHook(oldVnode)
  }
}
```

这里传入的 `oldVnode` 为一个 dom container，因此 `isRealElement` 为 true，然后通过 `oldVnode = emptyNodeAt(oldVnode)` 将 `oldVnode` 转为 vnode 对象，再调用 `createElm` 方法

`createElm` 方法是**通过虚拟节点创建真实的 dom**

```js
function createElm (vnode, insertedVnodeQueue, nested) {
    let i
    const data = vnode.data
    vnode.isRootInsert = !nested
    if (isDef(data)) {
      if (isDef(i = data.hook) && isDef(i = i.init)) i(vnode)
      // after calling the init hook, if the vnode is a child component
      // it should've created a child instance and mounted it. the child
      // component also has set the placeholder vnode's elm.
      // in that case we can just return the element and be done.
      if (isDef(i = vnode.child)) {
        initComponent(vnode, insertedVnodeQueue)
        return vnode.elm
      }
    }
    const children = vnode.children
    const tag = vnode.tag
    if (isDef(tag)) {
      // vnode tag 是否存在并进行合法性检查
      if (process.env.NODE_ENV !== 'production') {
        if (
          !vnode.ns &&
          !(config.ignoredElements && config.ignoredElements.indexOf(tag) > -1) &&
          config.isUnknownElement(tag)
        ) {
          warn(
            'Unknown custom element: <' + tag + '> - did you ' +
            'register the component correctly? For recursive components, ' +
            'make sure to provide the "name" option.',
            vnode.context
          )
        }
      }
      // 调用平台 dom 操作创建一个占位符元素
      vnode.elm = vnode.ns
        ? nodeOps.createElementNS(vnode.ns, tag)
        : nodeOps.createElement(tag)
      setScope(vnode)
      // 调用 createChildren 创建子元素，并插入到对应父元素中
      createChildren(vnode, children, insertedVnodeQueue)
      if (isDef(data)) {
        // 执行所有 create 钩子并把 vnode push 到 insertedVnodeQueue
        invokeCreateHooks(vnode, insertedVnodeQueue)
      }
    } else if (vnode.isComment) {
      // vnode 不含 tag，注释 创建注释节点
      vnode.elm = nodeOps.createComment(vnode.text)
    } else {
      // 否则创建文本节点
      vnode.elm = nodeOps.createTextNode(vnode.text)
    }
    return vnode.elm
  }
```

`createElm` 主要逻辑是先判断 vnode 是否包含 `tag` 并做合法性检查，如果包含 `tag` 之后调用平台 dom 操作创建一个占位符元素，接着调用 `createChildren` 方法创建子元素

`createChildren` 方法遍历子虚拟节点，递归调用 `createElm`， vnode.elm 作为父容器的 dom 节点占位符传入

```js
// 遍历子虚拟节点，递归调用 createElm， vnode.elm 作为父容器的 dom 节点占位符传入
function createChildren (vnode, children, insertedVnodeQueue) {
  if (Array.isArray(children)) {
    for (let i = 0; i < children.length; ++i) {
      nodeOps.appendChild(vnode.elm, createElm(children[i], insertedVnodeQueue, true))
    }
  } else if (isPrimitive(vnode.text)) {
    nodeOps.appendChild(vnode.elm, nodeOps.createTextNode(vnode.text))
  }
}
```

回到 `createElm`, 接着执行 `invokeCreateHooks`, 执行所有 create 钩子并把 vnode push 到 `insertedVnodeQueue`

```js
function invokeCreateHooks (vnode, insertedVnodeQueue) {
  for (let i = 0; i < cbs.create.length; ++i) {
    cbs.create[i](emptyNode, vnode)
  }
  i = vnode.data.hook // Reuse variable
  if (isDef(i)) {
    if (i.create) i.create(emptyNode, vnode)
    if (i.insert) insertedVnodeQueue.push(vnode)
  }
}
```

回到 `patch` 方法，最后把 dom 插入到它的父节点中并删除 `oldVnode`， 其实就是调用原生 dom api 进行 dom 操作

```js
// component root element replaced.
// update parent placeholder node element.
if (vnode.parent) {
  vnode.parent.elm = vnode.elm
  if (isPatchable(vnode)) {
    for (let i = 0; i < cbs.create.length; ++i) {
      cbs.create[i](emptyNode, vnode.parent)
    }
  }
}

if (parent !== null) {
  // 插入到它的父节点中
  nodeOps.insertBefore(parent, vnode.elm, nodeOps.nextSibling(elm))
  // 删除旧 oldVnode
  removeVnodes(parent, [oldVnode], 0, 0)
} else if (isDef(oldVnode.tag)) {
  invokeDestroyHook(oldVnode)
}

// createPatchFunction 方法 中 定义的 removeVnodes
function removeVnodes (parentElm, vnodes, startIdx, endIdx) {
  for (; startIdx <= endIdx; ++startIdx) {
    const ch = vnodes[startIdx]
    if (isDef(ch)) {
      if (isDef(ch.tag)) {
        removeAndInvokeRemoveHook(ch)
        invokeDestroyHook(ch)
      } else { // Text node
        nodeOps.removeChild(parentElm, ch.elm)
      }
    }
  }
}
```

vue 封装的 dom 辅助操作方法在`src/platforms/web/runtime/node-ops.js`

```js
// src/platforms/web/runtime/node-ops.js
// ...
export function insertBefore (parentNode: Node, newNode: Node, referenceNode: Node) {
  parentNode.insertBefore(newNode, referenceNode)
}

export function appendChild (node: Node, child: Node) {
  node.appendChild(child)
}

export function parentNode (node: Node): ?Node {
  return node.parentNode
}
```

到这里，`patch` 方法的首次渲染就是调用 `createEle` 方法，这里传入的 `parentElm` 是 `oldVnode.elm` 的父元素，例子中为 id 为 #app 的 div 的父元素，也就是 body，实际就是递归创建一个完整的 dom 树并插入到 body 上，并删除 `oldVnode`

#### update 首次渲染总结

- `vm.$el` (真实 dom) 作为 `oldVnode` 用来创建一个 vnode 并替换 `oldVnode` (此时 `oldVnode` 已经不是 dom 而是 vnode)，之前 `vm.$el` 赋值给 `oldVnode` 的 elm 属性
- 调用 `createElm` 对 `vnode` 创建 dom 元素
- 遇到 `vnode` 的 `children`，调用 `createChilden`, 遍历子虚拟节点，递归调用 `createElm` 生成 dom 并插入到其对应父节点中
- 最后将 `vnode` 生成的 dom 插入到它的父元素中，并删除 `oldVnode`

![](http://ony85apla.bkt.clouddn.com/18-6-15/88660125.jpg)

### update 组件更新

```js
// src/core/instance/lifecycle.js
// ...
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
  const vm: Component = this
  // ...
  const prevVnode = vm._vnode
  if (!prevVnode) {
     // 首次渲染
    vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
  } else {
    // 数据更新
    vm.$el = vm.__patch__(prevVnode, vnode)
  }
  // ...
}
```

`update` 方法组件更新时仍然会调用 `patch` 方法

#### patch

```js
// core/vdom/patch.js

export function createPatchFunction (backend) {
  // ...
  // 数据更新时候 oldVnode vnode 都是 vnode 类型
  return function patch (oldVnode, vnode, hydrating, removeOnly) {
    let elm, parent
    let isInitialPatch = false
    const insertedVnodeQueue = []

    if (!oldVnode) {
      // empty mount, create new root element
      // oldVnode 不存在，创建一个真实 dom
      isInitialPatch = true
      createElm(vnode, insertedVnodeQueue)
    } else {
      // 标记旧的 VNode 是否有 nodeType
      // 数据更新是 oldVnode 为 vnode 类型，因此 isRealElement 为 false
      const isRealElement = isDef(oldVnode.nodeType)
      if (!isRealElement && sameVnode(oldVnode, vnode)) {
        // 认为是同一个节点（只是局部更新）就对 oldVnode 和 vnode 进行 diff，并对 oldVnode 打 patch
        patchVnode(oldVnode, vnode, insertedVnodeQueue, removeOnly)
      } else {
        // ...
        // 不是同一节点，逻辑基本和首次渲染一样，直接 vnode 来创建真实 dom，插入到父节点中，删除 oldVnode
      }
    }

    // 调用 insert 钩子
    invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
    return vnode.elm
  }
}
```

和首次渲染不同，`oldVnode` 不为空，并且和 `vnode` 都是 `VNode` 类型

> 判断新旧 `vnode` 是否只是进行局部更新的同一个节点是通过 `sameVnode()` 方法， **只有这 2 个 vnode 的基本属性相同才会认为这 2 个 vnode 只是局部更新，才会进行 diff。如果 2 个 vnode 的基本属性不一致会直接跳过 diff 过程，根据 vnode 新建一个真实 dom，插入到父节点中, 删除 oldVnode**

如 `p` 和 `div` 会认为是不同的结构而不去 diff

```js
// core/instance/vdom/patch.js
// 判断两个 vnode 的 key tag isComment 相同，data 存在
function sameVnode (vnode1, vnode2) {
  return (
    vnode1.key === vnode2.key &&
    vnode1.tag === vnode2.tag &&
    vnode1.isComment === vnode2.isComment &&
    !vnode1.data === !vnode2.data
  )
}
```

> `patch` 过程总结如下：
- 如果 `oldVnode` 不存在，`createEle(vnode)` 为 vnode 直接创建真实 dom
- 否则利用 `sameVnode()` 判断新旧 vnode 是否是同一个节点, **不是同一个节点（不是局部更新，没必要进一步 diff），vnode 直接创建真实 dom，插入到父节点中，删除 oldVnode**
- 否则新旧 vnode 值得比较，调用 `patchVnode()`

#### patchVnode

```js
// src/core/vdom/patch.js
function patchVnode (oldVnode, vnode, insertedVnodeQueue, removeOnly) {
  // ...
  const hasData = isDef(i = vnode.data)
  const elm = vnode.elm = oldVnode.elm // 新旧 vnode 只是局部更新，指向同一个真实 dom
  const oldCh = oldVnode.children
  const ch = vnode.children
  // 如果 vnode 不是文本节点
  if (isUndef(vnode.text)) {
    // 如果 oldVnode 和 vnode 的 children 属性都存在
    if (isDef(oldCh) && isDef(ch)) {
      // oldCh 和 ch 不相同，updateCihldren, 对子节点进行 diff
      if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
    } else if (isDef(ch)) {
      // 如果 vnode 有子节点，oldVnode 没有子节点，如果 oldVnode 是文本几点先将节点文本清除
      if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
      // 然后将 vnode 的 children 添加进去
      addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
    } else if (isDef(oldCh)) {
      // 如果 oldVnode 有子节点，vnode 没有子节点，删除 elm 下的 oldChildren
      removeVnodes(elm, oldCh, 0, oldCh.length - 1)
    } else if (isDef(oldVnode.text)) {
      // oldVnode 是文本节点，就清空节点文本
      nodeOps.setTextContent(elm, '')
    }
  } else if (oldVnode.text !== vnode.text) {
    // 如果 vnode 是文本节点并且新旧文本不同，直接替换文本内容
    nodeOps.setTextContent(elm, vnode.text)
  }
  // ...
}

// src/platforms/web/runtime/node-ops.js
export function setTextContent (node: Node, text: string) {
  node.textContent = text
}
// src/core/vdom/patch.js
function removeVnodes (parentElm, vnodes, startIdx, endIdx) {
  for (; startIdx <= endIdx; ++startIdx) {
    const ch = vnodes[startIdx]
    if (isDef(ch)) {
      if (isDef(ch.tag)) {
        removeAndInvokeRemoveHook(ch)
        invokeDestroyHook(ch)
      } else { // Text node
        nodeOps.removeChild(parentElm, ch.elm)
      }
    }
  }
}
function addVnodes (parentElm, before, vnodes, startIdx, endIdx, insertedVnodeQueue) {
  for (; startIdx <= endIdx; ++startIdx) {
    nodeOps.insertBefore(parentElm, createElm(vnodes[startIdx], insertedVnodeQueue), before)
  }
}
```

`patchVnode` 方法作用就是将新 `vnode` patch 到 `oldVnode` 上

> `patchVnode` 中 diff 过程有几种情况，`oldCh` 为 `oldVnode` 的子节点， `ch` 为 `vnode` 的子节点：
- 首先进行文本节点判断，如果 `oldVnode.text !== vnode.text`, 说明 `vnode` 是文本节点并且新旧文本不同，直接进行文本替换
- 在 `vnode` 不是文本节点的情况下，进入子节点 diff
- 当 `oldCh` 和 `ch` 都存在并且不相同的情况下，调用 `updateChildren()` 对子节点进行 diff
- 如果只有 `ch` 存在，说明旧节点不再需要。如果 `oldVnode` 是文本节点先清空 文本，再调用 `addVnodes()` 将 `ch` 添加到 elm 中
- 如果只有 `oldCh` 存在，表示更新的是空节点，则调用 `removeVnodes` 删除 elm 真实节点下 `oldCh` 子节点
- 如果 `oldVnode` 是文本节点，就清空文本内容

### updateChildren()

```js
function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
  // 为 oldCh 和 newCh 分别建立索引，作为之后遍历的依据
  let oldStartIdx = 0
  let newStartIdx = 0
  let oldEndIdx = oldCh.length - 1
  let oldStartVnode = oldCh[0]
  let oldEndVnode = oldCh[oldEndIdx]
  let newEndIdx = newCh.length - 1
  let newStartVnode = newCh[0]
  let newEndVnode = newCh[newEndIdx]
  let oldKeyToIdx, idxInOld, elmToMove, before

  // removeOnly is a special flag used only by <transition-group>
  // to ensure removed elements stay in correct relative positions
  // during leaving transitions
  const canMove = !removeOnly

  // 直到 oldCh 或 newCh 被遍历完跳出循环
  while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
    if (isUndef(oldStartVnode)) {
      // oldStartVnode 被移除，oldStartIdx 指向下一个节点
      oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
    } else if (isUndef(oldEndVnode)) {
      // oldEndVnode 被移除，oldEndIdx 指向上一个节点
      oldEndVnode = oldCh[--oldEndIdx]
    } else if (sameVnode(oldStartVnode, newStartVnode)) {
      patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)
      oldStartVnode = oldCh[++oldStartIdx]
      newStartVnode = newCh[++newStartIdx]
    } else if (sameVnode(oldEndVnode, newEndVnode)) {
      patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue)
      oldEndVnode = oldCh[--oldEndIdx]
      newEndVnode = newCh[--newEndIdx]
    } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
      patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue)
      // 插到老的结束节点后面
      canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
      oldStartVnode = oldCh[++oldStartIdx]
      newEndVnode = newCh[--newEndIdx]
    } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
      patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue)
      // 插入到老的开始节点前面
      canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
      oldEndVnode = oldCh[--oldEndIdx]
      newStartVnode = newCh[++newStartIdx]
    } else {
      // 如果上述条件都不满足，则开始比较 key 值，首先建立 key 和 index 索引的对应关系
      if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
      idxInOld = isDef(newStartVnode.key) ? oldKeyToIdx[newStartVnode.key] : null
      // 如果 idxInOld 不存在
      // 1. newStartVnode 上存在这个 key，但是 oldKeyToIdx 中不存在
      // 2. newStartVnode 上没有设置 key 属性
      if (isUndef(idxInOld)) { // New element
        // 创建新的 dom 节点
        // 插入到 oldStartVnode.elm 前面
        // 参见 creatElm() 方法
        nodeOps.insertBefore(parentElm, createElm(newStartVnode, insertedVnodeQueue), oldStartVnode.elm)
        newStartVnode = newCh[++newStartIdx]
      } else {
        elmToMove = oldCh[idxInOld]
        /* istanbul ignore if */
        if (process.env.NODE_ENV !== 'production' && !elmToMove) {
          warn(
            'It seems there are duplicate keys that is causing an update error. ' +
            'Make sure each v-for item has a unique key.'
          )
        }
        if (elmToMove.tag !== newStartVnode.tag) {
          // same key but different element. treat as new element
          // 创建新的 dom 节点
          nodeOps.insertBefore(parentElm, createElm(newStartVnode, insertedVnodeQueue), oldStartVnode.elm)
          newStartVnode = newCh[++newStartIdx]
        } else {
          patchVnode(elmToMove, newStartVnode, insertedVnodeQueue)
          oldCh[idxInOld] = undefined
          // 移动 node 节点
          canMove && nodeOps.insertBefore(parentElm, newStartVnode.elm, oldStartVnode.elm)
          newStartVnode = newCh[++newStartIdx]
        }
      }
    }
  }
  // 如果最后遍历的 oldStartIdx 大于 oldEndIdx
  if (oldStartIdx > oldEndIdx) { // 如果是 oldVnode 先被遍历完
    before = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
    // 添加 newVnode 中剩余节点到 parentElm 中
    addVnodes(parentElm, before, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
  } else if (newStartIdx > newEndIdx) { // 如果是 newVnode 新被遍历完，则删除 oldVnode 中所有的节点
    // 删除剩余节点
    removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
  }
}
```

开始遍历 diff 前，给 `oldCh` 和 `ch` 分别分配一个 `startIndex` 和 `endIndex` 作为遍历索引，当 `oldCh` 和 `ch` 遍历完成（遍历完成的条件是 `oldCh` 或 `ch` 的 `startIndex >= endIndex`）

先看下节点属性不带 key 的情况：

- 从第一个节点开始比较，`oldCh` 和 `ch` 的开始节点或结束节点都不存在 `sameVnode`，第一个 diff 后，`ch` 的 `startVnode` 被添加到 `oldStartVnode` 前面， `newStartIndex` 后移一位

![](http://ony85apla.bkt.clouddn.com/18-6-16/83987055.jpg)

- 第二轮 diff，满足 `sameVnode(oldStartVnode, newStartVnode)`, 对这两个 vnode 进行 `patchVnode`, 最后将 patch 打到 `oldStartVnode`，同时 `oldStartIndex` `newStartIndex` 都后移一位

![](http://ony85apla.bkt.clouddn.com/18-6-16/5191444.jpg)

- 第三轮 diff，满足 `sameVnode(oldEndVnode, newStartVnode)`, 对这两个 vnode 进行 `patchVnode`, 最后将 patch 打到 `oldEndVnode`，并完成 `oldEndVnode` 的移位操作，同时 `oldEndIndex` 前移一位 `newStartIndex` 后移一位

![](http://ony85apla.bkt.clouddn.com/18-6-16/59141550.jpg)

- 第四轮 diff 过程同第三轮 diff

![](http://ony85apla.bkt.clouddn.com/18-6-16/19380189.jpg)

- 第五轮 diff 过程同第一轮 diff

![](http://ony85apla.bkt.clouddn.com/18-6-16/99938328.jpg)

- 遍历结束此时 `newStartIndex > newEndIndex`，此时 `oldCh` 存在多余节点，需要进行删除

![](http://ony85apla.bkt.clouddn.com/18-6-16/31595673.jpg)

> **在 vnode 节点属性不带 key 的情况下，每一轮 diff 都是 __开始__ 和 __结束__ 节点进行比较，直到 `oldCh` 或 `ch` 被遍历完.** 当 vnode 带有 key 属性，遍历 diff 过程中，当 `起始点` `结束点`的 搜寻以及 diff 出现无法匹配的情况，会用 key 作为唯一标识进行 diff，提高 diff 效率

带有 key 的 vnode 的 diff 过程

- 注意第一轮 diff 后 `oldCh` 上的 B 节点被删除，但是 `ch` 上 B 节点的 `elm` 属性保持对 `oldCh` 上 B 节点的 `elm` 引用

![](http://ony85apla.bkt.clouddn.com/18-6-16/17332475.jpg)
![](http://ony85apla.bkt.clouddn.com/18-6-16/40488653.jpg)
![](http://ony85apla.bkt.clouddn.com/18-6-16/71819335.jpg)
![](http://ony85apla.bkt.clouddn.com/18-6-16/27358504.jpg)
![](http://ony85apla.bkt.clouddn.com/18-6-16/25299407.jpg)

#### update 数据更新过程总结

组件更新的过程核心就是新旧 vnode diff，对新旧节点相同和不同的情况分别坐不同处理；新旧节点不同的更新流程是创建新节点-》更新父占位符节点-》删除旧节点；新旧节点相同（只是局部更新）的更新流程是去获取他们的 children，根据不同情况做不同的处理。最复杂的情况是新旧节点相同并且都存在子节点，需要执行 `updateChildren`
