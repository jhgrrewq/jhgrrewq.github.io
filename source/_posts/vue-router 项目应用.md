---
title: vue-router 项目应用
tag: 
	- vue
	- vue-router
    - 项目
---

体系庞大的单页面项目路由配置也可以进行划分

```html
|- router/
    |- module/
        |- index.js    // 分块的路由配置
        |- card.js     // 分块的路由配置
    |- router.js       // 配置文件
```

<!-- more -->

./app.js 入口文件

```javascript
import Vue from 'vue'
import router from './router/router' // 引入路由配置文件
...

window.vue = new Vue({
    el: '#app',
    ...
    router
})
```

./router/router.js 路由配置文件

```javascript
/**
 * 核心
 * vue/vue-router
 */
import Vue from 'vue'
import VueRouter from 'vue-router'
Vue.use(VueRouter) //调用

/**
 * vuex
 */
// import store from 'vuexStore/store.js'  //引用vuex

/**
 * 引入路由表
 */
import index from './modules/index.js'  //首页
import card from './modules/card.js'    //支付页

/**
 *  配置路由 分块的路由配置是数组，这里用 es6 展开运算符
 */
const routes =  [
  ...index,
  ...card
]

/**
 *  路由对象
 */
const router = new VueRouter({ // 使用配置文件规则
  routes,
  mode:'hash',
  strict: process.env.NODE_ENV !== 'production'
})

/* 路由跳转后 */
router.afterEach(route => {
    window.scrollTo(0,0)  //跳转后返回页面顶部
})

export default router
```

./router/module/index.js 分块路由配置

```javascript
/**
 * 路由懒加载
 *
 */
const index = () => import('../../page/index/index.vue') // 首页

...

export default [
 //顶层路由
 {
   path: '/',
   redirect: '/home'
   component: index
 },
 {
   path: '/home',
   component: index
 }
 ...
]
```
