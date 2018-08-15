---
title: vue-router 路由懒加载
tag: 
	- vue
	- vue-router
    - 路由懒加载
---

> 路由懒加载就是将不同路由对应的组件分割成不同的代码块，当路由被访问时才加载对应组件
> **通过 vue 异步组件 结合 webpack 代码分割实现**

<!-- more -->
## 异步组件

异步组件被定义为返回一个 promise 工厂函数，该函数返回的 promise 是 resolve 本身

```javascript
const Foo = () => Promise.resolve({
    ...
    // 定义组件本身
})
```

## webpack 代码分割

早期require时传入resolve

```javascript
resolve => require(['./foo.vue'], resolve)
```

**webpack 2 使用动态 import 语法定义代码分割点**
es6 定义 import() 为一个在运行时能动态加载 es6 模块的方法，
webpack把 import() 作为代码分割点， 将需求模块放到单独的chunk代码块中。import() 返回一个 Promise。 **注意，import() 时必须指向模块的 .default 值，因为它才是 promise 处理后返回的 module 对象** 

```javascript
import('./foo.vue) // 返回 Promise
```

综上，两者结合能定义一个被webpack代码自动分割的异步组件, 其他路由配置不需要改变

```javascript
const Foo = () => import('./foo.vue')

const router = new VueRouter({
    route: [{
        path: '/foo',
        component: Foo
    }]
})
```