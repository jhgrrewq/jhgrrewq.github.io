---
title: vue-router 源码初步学习
tag: 
	- vue
	- vue-router
---

> 参考: [vue-router源码分析-整体流程](https://github.com/DDFE/DDFE-blog/issues/9)、[vue-router源码分析](https://www.cnblogs.com/caizhenbo/p/7297730.html)

本文分析 vue-router@2.0.3

## 使用

先看一下 vue-router 官网的 demo

```html
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```

<!-- more -->

```js
// 0. 如果使用模块化机制编程，导入Vue和VueRouter，要调用 Vue.use(VueRouter)

// 1. 定义（路由）组件。
// 可以从其他文件 import 进来
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
// 我们晚点再讨论嵌套路由。
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. 创建 router 实例，然后传 `routes` 配置
// 你还可以传别的配置参数, 不过先这么简单着吧。
const router = new VueRouter({
  routes // （缩写）相当于 routes: routes
})

// 4. 创建和挂载根实例。
// 记得要通过 router 配置参数注入路由，
// 从而让整个应用都有路由功能
const app = new Vue({
  router
}).$mount('#app')

// 现在，应用已经启动了！
```

通过注入路由器，可以在任何组件内通过 this.$router 访问路由器，也可以通过 this.$route 访问当前路由

```js
// Home.vue
export default {
  computed: {
    username () {
      // 我们很快就会看到 `params` 是什么
      return this.$route.params.username
    }
  },
  methods: {
    goBack () {
      window.history.length > 1
        ? this.$router.go(-1)
        : this.$router.push('/')
    }
  }
}
```

## 代码结构

![](http://ony85apla.bkt.clouddn.com/18-4-17/18725369.jpg)

这里我们先关注几个和主要流程相关的部分

- `components 目录` 定义了 router-view 和 router-link 组件
- `history 目录` 封装了 前端路由 三种模式
- `create-matcher.js`
- `create-route-map.js`
- `index.js` vue-router 入口文件
- `install.js` vue-router 安装模块

## 插件安装

vue 使用插件是调用 Vue.use(plugin), 而 use() 方法内部会调用 plugin 的 install 方法（没有则将 plugin 自身作为函数进行调用）

入口文件 index.js 会导出 `VueRouter` 对象， 这里延续了 vue 插件的写法，对外暴露的插件对象含有的 install 方法用来安装插件的具体逻辑，同时判断如果是浏览器环境则自动安装插件

```js
/* @flow */

// 引入 install 模块
import { install } from './install'
import { createMatcher } from './create-matcher'
import { HashHistory } from './history/hash'
import { HTML5History } from './history/html5'
import { AbstractHistory } from './history/abstract'
// ...

export default class VueRouter {
  static install: () => void;
  // ...
}

// 赋值 install 属性
VueRouter.install = install

// 浏览器环境自动安装插件
if (inBrowser && window.Vue) {
  window.Vue.use(VueRouter)
}
```

>install 模块做了如下操作
>- **对 vue 实例进行 beforeCreate 钩子混入，在 vue 生命周期阶段会被调用**
>- **通过 Vue.prototype 定义 $route $route, 全局可以通过 vm.$router（this.$router） vm.$route (this.$route) 访问**
>- **全局注册 route-view router-link 组件**

```js
// 引入 router-view router-link 组件
import View from './components/view'
import Link from './components/link'

export let _Vue

export function install (Vue) {
  if (install.installed) return
  install.installed = true

  _Vue = Vue

  // this.$root 访问根实例
  // 注入 $route $route， 全局可以通过 vm.$router vm.$route 访问
  Object.defineProperty(Vue.prototype, '$router', {
    get () { return this.$root._router }
  })

  Object.defineProperty(Vue.prototype, '$route', {
    get () { return this.$root._route }
  })

  // 混入 beforeCreate
  Vue.mixin({
    beforeCreate () {
      // 判断 初始化根实例 是否有 router 选项
      if (this.$options.router) {
        // 赋值 _router
        this._router = this.$options.router
        // 初始化 init
        this._router.init(this)
        // 定义 响应式 _route 对象, 一旦 _route 发生改变，会通知 router-view 执行 render 方法
        Vue.util.defineReactive(this, '_route', this._router.history.current)
      }
    }
  })

  // 全局注册 router-view router-link 组件
  Vue.component('router-view', View)
  Vue.component('router-link', Link)
}
```

## 生成 router 实例

之前 官网 demo 中, 安装完 VueRouter 插件后，会实例化一个 router 对象，并将路由配置的数组作为参数传入

```js
const router = new VueRouter({
  routes 
})
```

> 生成 router 实例过程中进行了 两部操作
- **根据配置数组（传入的 routes）生成路由配置表**
- **根据不同的模式生成监控路由变化的 history 对象**

注：history/base.js 封装了基本 history 操作的 history 类；history/hash.js，history/html5.js 和 history/abstract.js 继承了 base，只是根据不同的模式封装了一些基本操作

index.js 导出 VueRouter 类

```js
/* @flow */

// ...
import { createMatcher } from './create-matcher'
import { HashHistory } from './history/hash'
import { HTML5History } from './history/html5'
import { AbstractHistory } from './history/abstract'
// ...

export default class VueRouter {
  //...

  constructor (options: RouterOptions = {}) {
    this.app = null
    this.options = options
    this.beforeHooks = []
    this.afterHooks = []
    // 创建 match 匹配函数，生成匹配表
    this.match = createMatcher(options.routes || [])

    // 路由模式 默认是 hash 模式
    let mode = options.mode || 'hash'
    // 浏览器不支持 history 模式需要退回 hash 模式
    this.fallback = mode === 'history' && !supportsHistory
    if (this.fallback) {
      mode = 'hash'
    }
    // 非浏览器
    if (!inBrowser) {
      mode = 'abstract'
    }
    this.mode = mode

    // 根据 mode 实例化具体的 History
    switch (mode) {
      case 'history':
        this.history = new HTML5History(this, options.base)
        break
      case 'hash':
        this.history = new HashHistory(this, options.base, this.fallback)
        break
      case 'abstract':
        this.history = new AbstractHistory(this)
        break
      default:
        assert(false, `invalid mode: ${mode}`)
    }
  }
  
  // 全局进入守卫
  beforeEach (fn: Function) {
    this.beforeHooks.push(fn)
  }

  // 全局离开守卫
  afterEach (fn: Function) {
    this.afterHooks.push(fn)
  }

  // 调用 history 对象 push 方法
  push (location: RawLocation) {
    this.history.push(location)
  }

  // 调用 history 对象 replace 方法
  replace (location: RawLocation) {
    this.history.replace(location)
  }

  // 调用 history 对象 replace 方法
  go (n: number) {
    this.history.go(n)
  }

  back () {
    this.go(-1)
  }

  forward () {
    this.go(1)
  }

  // ...
}
// ...
```

### 生成路由配置表

在上述构造函数中有一行代码 `this.match = createMatcher(options.routes || [])`, 将传入 routes 配置数组处理结果赋给 router 实例的 matcher 属性。生成路由配置表是通过 `create-matcher.js`

```js
/* @flow */

// ...
import { createRouteMap } from './create-route-map'
// ...

export function createMatcher (routes: Array<RouteConfig>): Matcher {
  // 创建 路由 map
  const { pathMap, nameMap } = createRouteMap(routes)

  function match (
    raw: RawLocation,
    currentRoute?: Route,
    redirectedFrom?: Location
  ): Route {
    const location = normalizeLocation(raw, currentRoute)
    const { name } = location

    // 命名路由处理
    if (name) {
      const record = nameMap[name]
      const paramNames = getParams(record.path)

      // ...

      if (record) {
        location.path = fillParams(record.path, location.params, `named route "${name}"`)
        // 创建 route
        return _createRoute(record, location, redirectedFrom)
      }
    } else if (location.path) {
      // 普通路由处理
      location.params = {}
      for (const path in pathMap) {
        // 从 pathMap 中找到匹配的路由创建新的 route
        if (matchRoute(path, location.params, location.path)) {
          return _createRoute(pathMap[path], location, redirectedFrom)
        }
      }
    }
    // no match
    return _createRoute(null, location)
  }

  function redirect (
    record: RouteRecord,
    location: Location
  ): Route {
    // ...
  }

  function alias (
    record: RouteRecord,
    location: Location,
    matchAs: string
  ): Route {
    // ...
  }

  function _createRoute (
    record: ?RouteRecord,
    location: Location,
    redirectedFrom?: Location
  ): Route {
    // ...
  }

  return match
}
// ...
```

`create-Matcher.js` 再一次将 routes 配置数组 传给了 `create-route-map.js` 进一步处理并生成 路由 map，最终返回 match 函数, match 函数调用会返回新的 route 对象

`create-route-map.js` 将 routes 配置数组遍历，**根据 name 和 path 将每一个路由项处理为一个 routeRecord，并分别保存到 pathMap 和 nameMap 中，并最终导出 包含 pathMap 和 nameMap 的对象**

```js
/* @flow */

import { assert, warn } from './util/warn'
import { cleanPath } from './util/path'

// 创建 路由 map
export function createRouteMap (routes: Array<RouteConfig>): {
  pathMap: Dictionary<RouteRecord>,
  nameMap: Dictionary<RouteRecord>
} {
  // path 路由 map
  const pathMap: Dictionary<RouteRecord> = Object.create(null)
  // name 路由 map
  const nameMap: Dictionary<RouteRecord> = Object.create(null)

  // 遍历路由配置对象 添加路由记录
  routes.forEach(route => {
    addRouteRecord(pathMap, nameMap, route)
  })

  return {
    pathMap,
    nameMap
  }
}

// 添加路由记录 方法
function addRouteRecord (
  pathMap: Dictionary<RouteRecord>,
  nameMap: Dictionary<RouteRecord>,
  route: RouteConfig,
  parent?: RouteRecord,
  matchAs?: string
) {
  const { path, name } = route
  assert(path != null, `"path" is required in a route configuration.`)

  // 封装 route 记录
  const record: RouteRecord = {
    path: normalizePath(path, parent), // 路径
    components: route.components || { default: route.component }, // 关联组件
    instances: {}, // 实例
    name, // 名字
    parent, // 父级 route
    matchAs,
    redirect: route.redirect, // 跳转
    beforeEnter: route.beforeEnter, // 进入守卫
    meta: route.meta || {} // meta 参数
  }

  // 嵌套子路由
  if (route.children) {
    // Warn if route is named and has a default child route.
    // If users navigate to this route by name, the default child will
    // not be rendered (GH Issue #629)
    if (process.env.NODE_ENV !== 'production') {
      if (route.name && route.children.some(child => /^\/?$/.test(child.path))) {
        warn(false, `Named Route '${route.name}' has a default child route.
          When navigating to this named route (:to="{name: '${route.name}'"), the default child route will not be rendered.
          Remove the name from this route and use the name of the default child route for named links instead.`
        )
      }
    }
    // 递归 收集子路由
    route.children.forEach(child => {
      addRouteRecord(pathMap, nameMap, child, record)
    })
  }

  // 别名
  if (route.alias !== undefined) {
    if (Array.isArray(route.alias)) {
      route.alias.forEach(alias => {
        addRouteRecord(pathMap, nameMap, { path: alias }, parent, record.path)
      })
    } else {
      addRouteRecord(pathMap, nameMap, { path: route.alias }, parent, record.path)
    }
  }

  // 按照 path 存储
  pathMap[record.path] = record
  // 按照 name 存储
  if (name) {
    if (!nameMap[name]) {
      nameMap[name] = record
    } else {
      warn(false, `Duplicate named routes definition: { name: "${name}", path: "${record.path}" }`)
    }
  }
}

function normalizePath (path: string, parent?: RouteRecord): string {
  path = path.replace(/\/$/, '')
  if (path[0] === '/') return path
  if (parent == null) return path
  return cleanPath(`${parent.path}/${path}`)
}
```

### 不同的模式生成 history 对象

VueRouter 构造函数除了根据传入 routes 配置数组生成路由配置表，同时还根据不同模式生成 history 对象。而所有 history 类都继承 `history/base.js`

```js
/* @flow */

// ...

export class History {
  router: VueRouter; // router 对象
  base: string; // 基准路径
  current: Route; // 当前路由
  pending: ?Route;
  cb: (r: Route) => void; // 回调

  // 子类实现
  // implemented by sub-classes
  go: (n: number) => void;
  push: (loc: RawLocation) => void;
  replace: (loc: RawLocation) => void;
  ensureURL: (push?: boolean) => void;

  constructor (router: VueRouter, base: ?string) {
    this.router = router
    this.base = normalizeBase(base) // 返回基准路径
    // start with a route object that stands for "nowhere"
    this.current = START // 设置当前 route
    this.pending = null
  }

  // history 对象 cb 属性是在 install.js 中注册的
  listen (cb: Function) {
    this.cb = cb
  }

  // 路由转化函数，传入 location cb
  // 找到匹配路由需要过渡的则 更新 route 执行传入的回调，更新地址
  transitionTo (location: RawLocation, cb?: Function) {
    // 通过 match 函数生成新的 route，找到匹配路由
    const route = this.router.match(location, this.current)
    // 过渡
    this.confirmTransition(route, () => {
      // 更新 route
      this.updateRoute(route)
      // 每次更新 route 都会执行回调，也就是改变 _route
      cb && cb(route)
      // 更新地址
      this.ensureURL()
      // 子类实现的更新 url 地址
      // 对于 hash 模式的话 就是更新 hash 的值
      // 对于 history 模式的话 就是利用 pushstate / replacestate 来更新
      // 浏览器地址
    })
  }

  // 过渡
  confirmTransition (route: Route, cb: Function) {
    const current = this.current
    // 如果前后是同一个路由，直接返回
    if (isSameRoute(route, current)) {
      this.ensureURL()
      return
    }

    // ...
  }

  // 更新路由
  updateRoute (route: Route) {
    const prev = this.current // 跳转前路由
    this.current = route // 准备跳转路由
    this.cb && this.cb(route) // 执行回调，每次更新都会调用
    this.router.afterHooks.forEach(hook => {
      hook && hook(route, prev)
    })
  }
}

// 返回基准路径
function normalizeBase (base: ?string): string {
  if (!base) {
    if (inBrowser) {
      // respect <base> tag
      const baseEl = document.querySelector('base')
      base = baseEl ? baseEl.getAttribute('href') : '/'
    } else {
      base = '/'
    }
  }
  // make sure there's the starting slash
  if (base.charAt(0) !== '/') {
    base = '/' + base
  }
  // remove trailing slash
  return base.replace(/\/$/, '')
}
// ...
```

`history/base.js` 实现了基本history的操作，`history/hash.js`，`history/html5.js` 和 `history/abstract.js` 继承了base，只是根据不同的模式封装了一下几个函数的基本操作

```js
// 子类实现
// implemented by sub-classes
go: (n: number) => void;
push: (loc: RawLocation) => void;
replace: (loc: RawLocation) => void;
ensureURL: (push?: boolean) => void;
```

接下来看一下各个 history 类对于 transitionTo() 的调用以及上述方法的封装

#### `history/hash.js`

hash 模式**通过监听 hashchange 事件，push 和 replace 方法通过更新 url 中的hash**

```js
/* @flow */

import type VueRouter from '../index'
import { History } from './base'
import { getLocation } from './html5'
import { cleanPath } from '../util/path'

export class HashHistory extends History {
  constructor (router: VueRouter, base: ?string, fallback: boolean) {
    super(router, base)

    // check history fallback deeplinking
    if (fallback && this.checkFallback()) {
      return
    }

    ensureSlash()
  }

  checkFallback () {
    const location = getLocation(this.base)
    if (!/^\/#/.test(location)) {
      window.location.replace(
        cleanPath(this.base + '/#' + location)
      )
      return true
    }
  }

  // 注册响应 hashchange 事件的回调
  onHashChange () {
    if (!ensureSlash()) {
      return
    }
    this.transitionTo(getHash(), route => {
      // 作为传入 transitionTo 的回调，将 route.fullPath 替换掉 getHash() 拿到的 hash
      replaceHash(route.fullPath)
    })
  }

  // push 方法
  push (location: RawLocation) {
    this.transitionTo(location, route => {
      pushHash(route.fullPath)
    })
  }

  // replace 方法
  replace (location: RawLocation) {
    this.transitionTo(location, route => {
      replaceHash(route.fullPath)
    })
  }
 
  // go 方法
  go (n: number) {
    window.history.go(n)
  }
  
  // ensureURL 方法 更新 url 通过更新 hash 的值
  ensureURL (push?: boolean) {
    const current = this.current.fullPath
    if (getHash() !== current) {
      push ? pushHash(current) : replaceHash(current)
    }
  }
}

function ensureSlash (): boolean {
  const path = getHash()
  if (path.charAt(0) === '/') {
    return true
  }
  replaceHash('/' + path)
  return false
}

// 获取 url 中 hash
export function getHash (): string {
  // We can't use window.location.hash here because it's not
  // consistent across browsers - Firefox will pre-decode it!
  const href = window.location.href
  const index = href.indexOf('#')
  return index === -1 ? '' : href.slice(index + 1)
}

// push hash
function pushHash (path) {
  window.location.hash = path
}

// replace hash
function replaceHash (path) {
  const i = window.location.href.indexOf('#')
  window.location.replace(
    window.location.href.slice(0, i >= 0 ? i : 0) + '#' + path
  )
}
```

#### `history/html5.js`

html5 模式**通过监听 popstate 事件，push 和 replace 方法通过调用 window.history 的 pushState 和 replaceState  方法**


```js
/* @flow */

import type VueRouter from '../index'
import { assert } from '../util/warn'
import { cleanPath } from '../util/path'
import { History } from './base'
import {
  saveScrollPosition,
  getScrollPosition,
  isValidPosition,
  normalizePosition,
  getElementPosition
} from '../util/scroll-position'

const genKey = () => String(Date.now())
let _key: string = genKey()

export class HTML5History extends History {
  constructor (router: VueRouter, base: ?string) {
    super(router, base)

    const expectScroll = router.options.scrollBehavior

    // 监听 popstate 事件
    window.addEventListener('popstate', e => {
      _key = e.state && e.state.key
      const current = this.current
      this.transitionTo(getLocation(this.base), next => {
        if (expectScroll) {
          this.handleScroll(next, current, true) // 处理滚动
        }
      })
    })

    if (expectScroll) {
      window.addEventListener('scroll', () => {
        saveScrollPosition(_key) // 保留当前滚动位置，用于返回时候复位
      })
    }
  }

  // go 方法
  go (n: number) {
    window.history.go(n)
  }

  // push 方法 内部调用 history html5 api pushState 方法
  push (location: RawLocation) {
    const current = this.current
    this.transitionTo(location, route => {
      pushState(cleanPath(this.base + route.fullPath))
      this.handleScroll(route, current, false)
    })
  }

  // replace 方法 内部调用 history html5 api replaceState 方法
  replace (location: RawLocation) {
    const current = this.current
    this.transitionTo(location, route => {
      replaceState(cleanPath(this.base + route.fullPath))
      this.handleScroll(route, current, false)
    })
  }

  // ensureURL 方法 更新 url 通过 pushstate / replacestate 来更新
  ensureURL (push?: boolean) {
    if (getLocation(this.base) !== this.current.fullPath) {
      const current = cleanPath(this.base + this.current.fullPath)
      push ? pushState(current) : replaceState(current)
    }
  }

  // ...
}

// 获取 url 中除了 base 的部分
export function getLocation (base: string): string {
  let path = window.location.pathname
  if (base && path.indexOf(base) === 0) {
    path = path.slice(base.length)
  }
  return (path || '/') + window.location.search + window.location.hash
}

// pushState 方法内部调用 window.history.pushState
function pushState (url: string, replace?: boolean) {
  // try...catch the pushState call to get around Safari
  // DOM Exception 18 where it limits to 100 pushState calls
  const history = window.history
  try {
    if (replace) {
      history.replaceState({ key: _key }, '', url)
    } else {
      _key = genKey()
      history.pushState({ key: _key }, '', url)
    }
    saveScrollPosition(_key)
  } catch (e) {
    window.location[replace ? 'assign' : 'replace'](url)
  }
}

// pushState 方法内部调用 window.history.replaceState
function replaceState (url: string) {
  pushState(url, true)
}
```

#### `history/abstract.js`

abstract 类用于 非浏览器端，通过栈数据结构来模拟路由路径

```js
/* @flow */

import type VueRouter from '../index'
import { History } from './base'

export class AbstractHistory extends History {
  index: number;
  stack: Array<Route>;

  constructor (router: VueRouter) {
    super(router)
    this.stack = []
    this.index = -1 // 当前 currentRoute 下标，当前激活默认是 stack 最后一个
  }

  // push 方法 添加 route 记录
  push (location: RawLocation) {
    this.transitionTo(location, route => {
      this.stack = this.stack.slice(0, this.index + 1).concat(route)
      this.index++
    })
  }

  // replace 方法 替换 route 记录
  replace (location: RawLocation) {
    this.transitionTo(location, route => {
      this.stack = this.stack.slice(0, this.index).concat(route)
    })
  }

  go (n: number) {
    const targetIndex = this.index + n
    if (targetIndex < 0 || targetIndex >= this.stack.length) {
      return
    }
    const route = this.stack[targetIndex]
    this.confirmTransition(route, () => {
      this.index = targetIndex
      this.updateRoute(route)
    })
  }

  ensureURL () {
    // noop
  }
}
```

## Vue 实例化

在官网 demo 中, 初始化 vue 实例 options 中传入 router

```js
new Vue({
  router,
  template: `
    <div id="app">
      <h1>Basic</h1>
      <ul>
        <li><router-link to="/">/</router-link></li>
        <li><router-link to="/foo">/foo</router-link></li>
        <li><router-link to="/bar">/bar</router-link></li>
        <router-link tag="li" to="/bar">/bar</router-link>
      </ul>
      <router-view class="view"></router-view>
    </div>
  `
}).$mount('#app')
```

而之前 `install.js` 中, 判断初始化根实例 是否有 router 选项，有就说明带有路由配置的 router 实例被创建了，这时候会继续 执行 `this._router.init(this)` 和定义 响应式 `_route` 对象

```js
//...

// 混入 beforeCreate
Vue.mixin({
  beforeCreate () {
    // 判断 初始化根实例 是否有 router 选项
    if (this.$options.router) {
      // 赋值 _router
      this._router = this.$options.router
      // 初始化 init
      this._router.init(this)
      // 定义 响应式 _route 对象
      Vue.util.defineReactive(this, '_route', this._router.history.current)
    }
  }
})
// ...
```

看 `index.js` 中 VueRouter 构造函数中的 init() 方法

- 其实就是针对 HTML5History 和 HashHistory 特殊处理，需要当前浏览器地址栏里的 path 或者 hash 来激活对应的路由，这里就是通过调用 transitionTo() 方法
- 同时调用 history 对象 listen() 方法注册 callback，而通过 `history/base.js` 可知 transitionTo() 中，history 更新后会调用这个 callback, **这个 callback 作用就是更新当前实例 _route 值**

```js
// ...
export default class VueRouter {
  // ...
  init (app: any /* Vue component instance */) {
    // ...
    this.app = app

    const history = this.history


    // 初始化时，hash 模式和 html5 模式有可能存在进入的不是默认页，需要根据当前浏览器地址栏里的 path 或者 hash 来激活对应的路由，此时就是通过调用 transitionTo 来达到目的
    if (history instanceof HTML5History) {
      history.transitionTo(getLocation(history.base))
    } else if (history instanceof HashHistory) {
      history.transitionTo(getHash(), () => {
        // 监听 hashchange 事件并执行 onHashChange 方法
        window.addEventListener('hashchange', () => {
          history.onHashChange()
        })
      })
    }

    // 注册当前 history 对象的 cb
    // history/base 中 listen (cb: Function) {this.cb = cb}
    // cb 会在 history 更新后调用
    // cb 作用在于更新当前实例 _route
    history.listen(route => {
      this.app._route = route
    })
  }
// ...
}
// ...
```

对于 `history.listen(route => {this.app._route = route})` ，history 更新后会调用这个 callback, 这个 callback 作用就是更新当前实例 _route 值，联系之前 `install.js` 定义响应式的 _route 对象 `Vue.util.defineReactive(this, '_route', this._router.history.current)`, `_route` 属性的值就是 `this._router.history.current` ，也就是当前 history 实例的当前活动路由对象。**这意味着每当 history 更新后会更新实例的 _route，会触发更新机制，继而会触发 应用实例的 render 重新渲染**

## router-view 组件

`components/view.js` 定义了 router-view 组件，其逻辑就是拿到匹配的组件进行渲染

```js
export default {
  name: 'router-view',
  functional: true, // 功能组件，单纯渲染
  props: {
    name: {
      type: String,
      default: 'default' // 默认 default 默认命名视图 name
    }
  },
  // h 函数（createElement 函数生成 vnode）
  render (h, { props, children, parent, data }) {
    // 解决深度嵌套
    data.routerView = true
    // route 对象
    const route = parent.$route
    // 缓存
    const cache = parent._routerViewCache || (parent._routerViewCache = {})
    let depth = 0
    let inactive = false

    // 当前组件深度
    while (parent) {
      if (parent.$vnode && parent.$vnode.data.routerView) {
        depth++
      }
      // 处理 keep-alive 逻辑
      if (parent._inactive) {
        inactive = true
      }
      parent = parent.$parent
    }

    data.routerViewDepth = depth
    // 得到匹配的当前组件层级的 路由记录
    const matched = route.matched[depth]
    if (!matched) {
      return h()
    }

    // 得到待渲染组件
    const name = props.name
    // const matched = route.matched[depth]
    const component = inactive
      ? cache[name]
      : (cache[name] = matched.components[name])

    if (!inactive) {
      // 非 keep-alive 模式，每次都要设置钩子
      // 进而更新匹配的实例元素
      const hooks = data.hook || (data.hook = {})
      hooks.init = vnode => {
        matched.instances[name] = vnode.child
      }
      hooks.prepatch = (oldVnode, vnode) => {
        matched.instances[name] = vnode.child
      }
      hooks.destroy = vnode => {
        if (matched.instances[name] === vnode.child) {
          matched.instances[name] = undefined
        }
      }
    }

    // 调用 createElement 函数渲染匹配组件
    return h(component, data, children)
  }
}
```

## router-link 组件

`components/link.js` 定义了 router-link 组件, 就是**在点击时根据设置的 to 值调用 router 的 push 或 replace 方法来更新路由；同时会检查自身是否于当前路由匹配（严格匹配和包含匹配）来决定自身是否添加 activeClass**

```js
/* @flow */

import { cleanPath } from '../util/path'
import { createRoute, isSameRoute, isIncludedRoute } from '../util/route'
import { normalizeLocation } from '../util/location'
import { _Vue } from '../install'

// work around weird flow bug
const toTypes: Array<Function> = [String, Object]

export default {
  name: 'router-link',
  props: {
    to: { // 目标路由链接
      type: toTypes,
      required: true
    },
    tag: { // router-link 被渲染成的 html 标签
      type: String,
      default: 'a'
    },
    // 完整模式，如果为 true 意味着 绝对相等的路由才会添加 activeClass，否则是包含关系
    exact: Boolean,
    // 在当前（相对）路径中附加路径
    append: Boolean,
    // 如果为 true 则调用 router.replace() 替换历史操作
    replace: Boolean,
    // 链接激活时使用的 css 类名
    activeClass: String
  },
  render (h: Function) {
    // 得到 router 实例和当前激活的 route 对象
    const router = this.$router
    const current = this.$route
    const to = normalizeLocation(this.to, current, this.append)
    // 根据当前目标链接和当前的 router 匹配结果
    const resolved = router.match(to, current)
    const fullPath = resolved.redirectedFrom || resolved.fullPath
    const base = router.history.base
    // 创建 href
    const href = createHref(base, fullPath, router.mode)
    const classes = {}
    // 激活 class：优先当前组件上获取，要么是 router 设置的 linkActiveClass，默认是 'router-link-active'
    const activeClass = this.activeClass || router.options.linkActiveClass || 'router-link-active'
    // 比较目标
    // 因为有命名路由，因此不一定有 path
    const compareTarget = to.path ? createRoute(null, to) : resolved
    // 如果严格模式，判断是否是相同路由（path query params hash）
    // 否则走包含逻辑（path 包含、query 包含、hash 为空或相同）
    classes[activeClass] = this.exact
      ? isSameRoute(current, compareTarget)
      : isIncludedRoute(current, compareTarget)

    // 事件绑定
    const on = {
      click: (e) => {
        // don't redirect with control keys
        /* istanbul ignore if */
        // 忽略功能键的点击
        if (e.metaKey || e.ctrlKey || e.shiftKey) return
        // don't redirect when preventDefault called
        /* istanbul ignore if */
        // 已阻止返回
        if (e.defaultPrevented) return
        // don't redirect on right click
        /* istanbul ignore if */
        // 右击
        if (e.button !== 0) return
        // don't redirect if `target="_blank"`
        /* istanbul ignore if */
        // `target="_blank"` 忽略
        const target = e.target.getAttribute('target')
        if (/\b_blank\b/i.test(target)) return

        // 阻止默认行为 防止跳转
        e.preventDefault()
        // router-link 点击会触发 route 改变
        if (this.replace) {
          // replace 逻辑
          router.replace(to)
        } else {
          // push 逻辑
          router.push(to)
        }
      }
    }

    // 创建元素需要附加的数据
    const data: any = {
      class: classes
    }

    if (this.tag === 'a') {
      data.on = on
      data.attrs = { href }
    } else {
      // find the first <a> child and apply listener and href
      // 找到第一个 <a> 并给这个元素事件绑定和 href 属性
      const a = findAnchor(this.$slots.default)
      if (a) {
        // in case the <a> is a static node
        a.isStatic = false
        const extend = _Vue.util.extend
        const aData = a.data = extend({}, a.data)
        aData.on = on
        const aAttrs = a.data.attrs = extend({}, a.data.attrs)
        aAttrs.href = href
      } else {
        // doesn't have <a> child, apply listener to self
        // 没有 <a> 就给当前元素自身绑定事件
        data.on = on
      }
    }
    // 创建元素
    return h(this.tag, data, this.$slots.default)
  }
}

function findAnchor (children) {
  if (children) {
    let child
    for (let i = 0; i < children.length; i++) {
      child = children[i]
      if (child.tag === 'a') {
        return child
      }
      if (child.children && (child = findAnchor(child.children))) {
        return child
      }
    }
  }
}

function createHref (base, fullPath, mode) {
  // hash 形式 '/#/xxx'
  // h5 形式 '/xxx'
  var path = mode === 'hash' ? '/#' + fullPath : fullPath
  return base ? cleanPath(base + path) : path
}
```

## 总结

>### 安装插件

- **混入 `beforeCreate` 生命周期处理，初始化 `_router`，`_route` 等数据** (`_router` 为 `this.$option.router`；`this._router.init()`; 设置 `_route` 响应式对象，值为 `this._router.history.current`)
- **全局设置访问 `$router` 和 `$route`**(`this.$router` 访问的是 `this._router`, `this.$route` 访问的是 `this._route`)
- **全局注册 router-link 和 router-view 两个组件，router-link 用于触发路由的变化，router-view 用于触发对应路由视图的变化**

>### 根据路由配置生成 router 实例

- **根据 routes 配置数组生成路由配置记录表**(包括对嵌套路由的递归，命名路由的处理)
- **根据不同模式生成 history 对象**（定义 `listen` 方法注册回调，每次 route 更新都会执行；`transitionTo` 方法会会先生成一个新的 route 对象，判断 route 是否改变，改变的话更新 `history.current` , 执行回调触发根组件的 `_route` 的变化（`route => {this.app._route = route}`）），根据不同的 history 调用不同 api 更新 url)

>### 将 router 实例注入 vue 根实例

- **根据 `beforeCreate` 混入，为根 vue 对象设置了劫持字段 `_route`**
- **调用 `init()` 函数，完成首次路由的渲染，首次渲染的调用路径是调用 `history.transitionTo` 方法，根据 router 的 `match` 函数，生成一个新的 route 对象；接着通过 `confirmTransition` 对比一下新生成的 route 和当前的 route 对象是否改变，改变的话触发 `updateRoute`，更新 `hsitory.current` 属性，执行回调触发根组件的 `_route` 的变化（`route => {this.app._route = route}`）,从而导致组件的调用 render 函数，更新 router-view**
- **另外一种更新路由的方式是主动触发:** router-link 绑定了 click 方法，触发 `history.push` 或者 `history.replace`, 从而触发 `history.transitionTo`, 同时会监控 hashchange 和 popstate 来对路由变化作对用的处理

![](http://ony85apla.bkt.clouddn.com/18-4-17/34082640.jpg)