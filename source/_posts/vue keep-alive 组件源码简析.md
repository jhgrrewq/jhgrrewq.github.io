---
title: vue keep-alive 组件源码简析
tag: 
	- vue
---

> 参考: [Vue.js 官网 keep-alive](https://cn.vuejs.org/v2/api/#keep-alive)、[Vue keep-alive实践总结](https://www.cnblogs.com/sysuhanyf/p/7454530.html)、[聊聊keep-alive组件的使用及其实现原理.MarkDown](https://github.com/answershuto/learnVue/blob/master/docs/%E8%81%8A%E8%81%8Akeep-alive%E7%BB%84%E4%BB%B6%E7%9A%84%E4%BD%BF%E7%94%A8%E5%8F%8A%E5%85%B6%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86.MarkDown)

## keep-alive 组件实践

> `keep-alive` 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们（主要用于保存组件状态和防止重复渲染）。和 `transition` 相似，`keep-alive` 是一个抽象组件，自身不会渲染一个 dom 元素
> 当组件在 `keep-alive` 内切换，它的 `activated` `deactivated` 两个生命周期钩子函数将会被执行

<!-- more -->

### 更新数据

`keep-alive` 组件生命周期相关钩子：activated、deactivated

- 使用 `keep-alive` 会将数据缓存在内存中，**如果要在每次进入页面获取最新的数据，需要在 `activated` 阶段获取数据**

### props 控制缓存组件

vue@2.1.0 props 新增 include 和 exclude 允许组件有条件缓存

- `include` 逗号分隔的字符串或正则或数组，只有匹配的组件会被缓存
- `exclude` 逗号分隔的字符串或正则或数组，匹配的组件不会被缓存

```html
<!-- 逗号分割字符串 -->
<!-- 将缓存 name 为 a 和 b 的组件 -->
<keep-alive include="a, b">
  <component></component>
</leep-alive>

<!-- 正则 -->
<!-- 需要使用 v-bind -->
<keep-alive :include="/a|b/">
  <component></component>
</leep-alive>

<!-- 数组 -->
<!-- 需要使用 v-bind -->
<keep-alive :include="['a','b']">
  <component></component>
</leep-alive>
```

匹配首先**检查组件自身的 `name` 选项**，如果 `name` 选项不可用，会匹配它的局部注册名称（**父组件 `components` 选项的键值**）。匿名组件不能被匹配

### 结合 router，缓存部分页面

使用 `$route.meta.keepAlive` 属性，需要在 `routes` 中配置原信息 `meta`

```html
<keep-alive>
  <router-view v-if="$route.meta.keepAlive"></router-view>
</leep-alive>
<router-view v-if="!$route.meta.keepAlive"></router-view>
```

```js
export default new Router({
  routes: [{
    path: '/',
    name: 'Hello',
    component: Hello,
    meta: {
      keepAlive: false // 不需要缓存
    }
  }, {
    path: '/page1',
    name: 'Page1',
    component: Page1,
    meta: {
      keepAlive: true // 需要缓存
    }
  }]
})
```

这里 `/page1` 组件页面中输入框输入的内容会被缓存，在跳转 `/` 组件页面后跳转回 `/page1` 时，输入框仍然保留之前输入的内容

### 每个路由的 `beforeRouteLeave(to, from, next)` 钩子中 设置 `to.meta.keepAlive`, 可设置不同页面跳转到同一个页面时该页面是否缓存

如果 b 页面跳转到 a 页面，a页面需要缓存； c 页面跳转到 a 页面，a 页面不缓存

```js
// a 的路由设置
{
  path: '/a',
  name: 'a',
  component: A,
  meta: {
    keepAlive: true // 需要被缓存
  }
}

// b 组件
export default {
  data() {
    return {}
  },
  beforeRouteLeave(to, from, next) {
    // 设置跳转到下一个路由的 meta
    to.meta.keepAlive = true // b 跳转到 a，a需要缓存，不刷新
    next()
  }
}

// c 组件
export default {
  data() {
    return {}
  },
  beforeRouteLeave(to, from, next) {
    // 设置跳转到下一个路由的 meta
    to.meta.keepAlive = false // c 跳转到 a，a不要缓存，刷新
    next()
  }
}
```

## keep-alive 组件源码

### created 钩子

created 钩子会创建 cache 对象，用来缓存 vnode 节点

```js
created () {
  this.cache = Object.create(null)
  this.keys = []
},
```

### props

```js
props: {
  include: patternTypes,
  exclude: patternTypes,
  max: [String, Number]
}
```

### destroyed 钩子

destroyed 钩子会在组件被注销的时候清除 cache 缓存中的组件实例

```js
destroyed () {
  for (const key in this.cache) {
    pruneCacheEntry(this.cache, key, this.keys)
  }
}
```

### render 函数

```js
render () {
  // 得到 slot 插槽中的第一个组件
  const slot = this.$slots.default
  const vnode: VNode = getFirstComponentChild(slot)

  const componentOptions: ?VNodeComponentOptions = vnode && vnode.componentOptions
  if (componentOptions) {
    // check pattern
    // 获取组件名称，优先获取组件 name 字段，否则是组件的 tag
    const name: ?string = getComponentName(componentOptions)
    const { include, exclude } = this
    // name 不在 include 中或者在 exclude 中直接返回 vnode （不从缓存中取）
    if (
      // not included
      (include && (!name || !matches(include, name))) ||
      // excluded
      (exclude && name && matches(exclude, name))
    ) {
      return vnode
    }

    const { cache, keys } = this
    const key: ?string = vnode.key == null
      // same constructor may get registered as different local components
      // so cid alone is not enough (#3269)
      ? componentOptions.Ctor.cid + (componentOptions.tag ? `::${componentOptions.tag}` : '')
      : vnode.key
    // 如果已经缓存则直接从缓存中获取组件实例被 vnode，还未缓存则进行缓存
    if (cache[key]) {
      vnode.componentInstance = cache[key].componentInstance
      // make current key freshest
      remove(keys, key)
      keys.push(key)
    } else {
      cache[key] = vnode
      keys.push(key)
      // prune oldest entry
      if (this.max && keys.length > parseInt(this.max)) {
        pruneCacheEntry(cache, keys[0], keys, this._vnode)
      }
    }
    // keepAlive 标志位
    vnode.data.keepAlive = true
  }
  return vnode || (slot && slot[0])
}
```

对于 props 中 include 和 exclude 支持传入 逗号分隔字符串、正则和数组，匹配缓存组件 name 的方法 如下

```js
function matches (pattern: string | RegExp | Array<string>, name: string): boolean {
  if (Array.isArray(pattern)) {
    return pattern.indexOf(name) > -1
  } else if (typeof pattern === 'string') {
    return pattern.split(',').indexOf(name) > -1
  } else if (isRegExp(pattern)) {
    return pattern.test(name)
  }
  /* istanbul ignore next */
  return false
}
```

### mounted

mounted 中 watch `include` `exclude` 两个属性的改变，在改变的时候修改 cache 中的缓存数据

```js
this.$watch('include', val => {
    pruneCache(this, name => matches(val, name))
  })
  this.$watch('exclude', val => {
    pruneCache(this, name => !matches(val, name))
  })
```

pruneCache 源码

```js
// 修正 cache
function pruneCache (keepAliveInstance: any, filter: Function) {
  const { cache, keys, _vnode } = keepAliveInstance
  for (const key in cache) {
    // 取出 cache 中的 vnode
    const cachedNode: ?VNode = cache[key]
    if (cachedNode) {
      const name: ?string = getComponentName(cachedNode.componentOptions)
      // name 不符合 filter 条件，并且不是目前渲染的 vnode时，销毁 vnode 对应的组件实例
      if (name && !filter(name)) {
        pruneCacheEntry(cache, key, keys, _vnode)
      }
    }
  }
}

// 销毁 vnode 对应的组件实例（Vue 实例）
function pruneCacheEntry (
  cache: VNodeCache,
  key: string,
  keys: Array<string>,
  current?: VNode
) {
  const cached = cache[key]
  if (cached && (!current || cached.tag !== current.tag)) {
    cached.componentInstance.$destroy()
  }
  cache[key] = null
  remove(keys, key)
}
```