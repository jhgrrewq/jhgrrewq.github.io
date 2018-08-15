---
title: axios 登录拦截
tag: 
	- axios
---

> 项目地址：https://github.com/jhgrrewq/vue-login.git

该项目是利用了 github 的 personal token 来替代 用户名密码 访问 github 的仓库列表

- 需要先在 github [生成 token](https://github.com/settings/tokens/new)

- 之后可以通过 **设置请求 header 中的 Authorization 字段** 发起请求

```js
// 请求用户的仓库列表
curl -v -H "Authorization: token TOKEN" https://api.github.com/user/repos
```

<!-- more -->

## 技术栈

- vue-cli
- vue
- vue-router
- vuex
- axios
- less

## 登录逻辑

前端路由需要设置 meta 字段。需要在路由配置时自定义字段requireAuth，用来判断路由是否需要登录

```js
const router = new Router({
  routes: [
    {
      path: '/',
      name: 'index',
      component: Index
    },
    {
      path: '/login',
      name: 'login',
      component: Login
    },
    {
      path: '/project',
      name: 'project',
      component: Project,
      // 元数据 需要登录验证
      meta: {
        requiresAuth: true
      }
    }
  ]
})
```

routes 配置中的每个路由对象为 路由记录。一个路由匹配到的所有路由记录会暴露为 $route 对象（还有在导航守卫中的路由对象）的 $route.matched 数组。因此，我们需要通过 beforeEach() 对路由进行判断，遍历 $route.matched 来检查路由记录中的 meta 字段

```js
// 全局导航守卫
router.beforeEach((to, from, next) => {
  // to, from 都是路由对象
  if (to.matched.some(r => r.meta.requiresAuth)) {
  // if (to.meta && to.meta.requiresAuth)) {
    // 元数据需要登录验证
    if (localStorage.getItem('token')) {
      // 验证有 token
      next()
    } else {
      // 跳转到登录，同时将当前的路由作为参数传递
      next({
        path: '/login',
        query: {
          redirect: to.fullPath
        }
      })
    }
  } else {
    next()
  }
})
```

到此只是在前端对路由进行了简单的控制，还无法真正阻止用户访问需要登录权限的路由。因为有时 远程 token 失效，但是本地的 token 还保存，因此需要 **axios 的 http 拦截器结合 后端接口返回的 http 状态码**来判断

## axios 二次封装

主要是 axios http 拦截器设置，需要结合 http 状态码进行判断

```js
import axios from 'axios'
import router from '@/router'
import store from '@/store/index'
import * as types from '@/store/mutation-types'

// axios 基础配置
axios.defaults.timeout = 5000
axios.defaults.baseURL = 'https://api.github.com'

// http request 拦截器
axios.interceptors.request.use(
  config => {
    // 本地存在 token，请求头带上 token
    if (localStorage.getItem('token')) {
      config.headers.Authorization = `token ${localStorage.getItem('token')}`
    }
    return config
  },
  error => {
    return Promise.reject(error)
  }
)

// http response 拦截器
axios.interceptors.response.use(
  response => {
    return response
  },
  error => {
    if (error.response) {
      let status = error.response.status
      switch (status) {
        case 401:
          // 清除 token 并跳转到 登录
          store.commit(types.LOGOUT)
          router.push({
            path: '/login'
          })
      }
    }
    return Promise.reject(error.response.data)
  }
)

export default axios
```

## 项目运行和构建

```js
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```
