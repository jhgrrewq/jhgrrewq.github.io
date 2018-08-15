---
title: vuex 多页项目应用
tag: 
	- vuex
    - 项目
---

<!-- markdownlint-disable MD010 -->

对于大型应用，常常将 vuex 相关代码分割到 module 中

```javascript
├── service
│   └── api.js            # 导出 API 模块
│   └── apiConfig.js      # 接口地址配置
│   └── axios.js          # axios 二次封装
│   ├── modules
│       └── apiUser.js    # 分模块 请求
│       └── ...
├── components
│   └── ...
└── vueStore
│   └── store.js          # 组装模块并导出 store 的地方
│   └── actions.js        # 根级别的 action (也可不设置根级别)
│   └── mutations.js      # 根级别的 mutation (也可不设置根级别)
└── views
│   ├── index             # 首页单页面
│       ├── page                    # 每个页面都是单页面应用
│           └── index.html          # 单页面模板
│           └── index.vue           # 单页面入口 js 文件引用组件
│           └── index.js            # 单页面入口 js 文件
│
│           ├── login               # 登录相关组件目录
│               └── login.vue       # 登录子组件
│               └── register.vue    # 注册子组件
│
│       ├── router                  # 单页面路由目录
│           └── router.js           # 组装模块并导出 router 的地方
│           └── modules
│               └── login.js        # 分模块 路由配置
│
│       ├── vuex                    # 单页面 vuex 目录
│           └── store.js            # 组装模块并导出
│           └── modules
│               └── login.js        # 分模块 登录 vuex
│   ├── card             # 购物车单页面
```

<!-- more -->

## api 封装

service/apiConfig.js 接口地址配置

```js
let env = process.env.NODE_ENV
let api = {}
if(env === 'development') {  // 开发环境
    apiUser:'/apiUser',    // 用户服务系统
    // ...
} else if(env === 'testing') { // 测试环境
    apiUser:'http://xxx.xxx.xxx.xxx:xxxx',    //用户服务系统
    // ...
} else {
    apiUser:'http://xxx.xxx.xxx.xxx:xxxx',    //用户服务系统
    // ...
}
export default api
```

service/axios.js 二次封装 axios

```js
import axios from 'axios'
import api from './apiConfig' // 真实接口配置
import store from 'vuexStore/store.js' // 引用 vuex
import qs from 'qs'
import template from 'url-template'

// axios 全局配置
axios.defaults.timeout = 5000;
axios.defaults.headers['Content-Type'] = 'application/json;charset=UTF-8'
// ...

// axios 拦截器
// http request 拦截器
axios.interceptors.request.use(
  config => {
    if (localStorage.aynUserToken) {
      config.headers.common['X-AIYANGNIU-SIGNATURE'] = localStorage.aynUserToken;
    }
    return config;
  },
  err => {
    return Promise.reject(err);
  });

// http response 拦截器
axios.interceptors.response.use(
  response => {
    if (response.data) {
      let code = response.data.code
      switch (code) {
        case 109: // 109 清除token信息并跳转到登录页面
          localStorage.aynUserToken = ''
          // vue.$loginVerify()
          break;
        case 110:
          // vue.$roleVerify()
          break;
      }
    }
    return response;
  },
  error => {
    if (error.response) {
      let status = error.response.status
      switch (status) {
        case 401: // 109 清除token信息并跳转到登录页面
          localStorage.aynUserToken = ''
          // vue.$loginVerify()
          break;
      }
    }
    let err = error.response ? (error.response.data || error.response) : (error.message || error)
    return Promise.reject(err)
  }
);

/* 二次封装 */
export default async(type, apiName, url, data, options) => {

  let vm = this
    //url拼接
  let path = (apiName != '') ? api[apiName] + url : url
    //  console.log(path,"path");
    //url-template
  let UrlTemplate = template.parse(path)
    //  console.log(data,"data");
    //格式化地址,如果data传入的参数是{params:{xx:"xxx",xxx:"xxxx"}}结构的，expand要取{xx:"xxx",xxx:"xxxx"}来格式化url
  if (data && data.hasOwnProperty("params")) {
    path = UrlTemplate.expand(data.params)
    if (!data.params) {
      data.params = {}
    }
    data.params._t = Date.now().toString()
  } else {
    path = UrlTemplate.expand(data)
    if (data) {
      data._t = Date.now().toString()
    }
  }

  //post/put data转form
  if (options && options.form) {
    let opt = {
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }
    data = qs.stringify(data)
    options === {} ? options.headers = opt.headers : options = opt
  }
  //axios
  await axios[type](path, data, options).then((res) => {
    vm.result = res.data
  })
  return vm.result
}
```

service/api.js 导出 API 模块

```js
import * as apiUser from './modules/apiUser'
// ...

export const apiUser = apiUser
// ...
```

service/modules/apiUser.js 分模块 请求

```js
import res from 'service/axios'

// 获取token
export const GetUserToken = (params) => res('post', 'apiUser', '/users/login', params, {form: true})

// 用户信息读取接口
export const GetUserInfo = () => res('get', 'apiUser', '/users/user',{})
// ...
```

## vuex

vuex/mutation-types.js 应用 mutations 常量设置（全局文件）

```js
/**
 * 登录注册
 * index/login
 */
export const USER_GET_TOKEN = 'USER_GET_TOKEN'
export const USER_GET_USERINFO = 'USER_GET_USERINFO'
export const USER_LOGIN_EXIT = 'USER_LOGIN_EXIT'
export const USER_GET_LOGIN_BG = 'USER_GET_LOGIN_BG'
// ...
```

vuex/store.js 组装模块并导出 store（全局文件）

```js
/**
 * 核心
 * vue/vuex
 */
import Vue from 'vue'
import Vuex from 'vuex'

import index from 'index/vuex/store'      //商城
import cart from 'cart/vuex/store'        //进货单
// ...

/* 调试 */
const debug = process.env.NODE_ENV !== 'production'

// Vue.use(Vuex)
export default new Vuex.Store({
  // 还可以包括 全局 actions mutations 等 
  // 组合各个模块
  modules: {
    ...index,
    ...cart
    // ...
  },
  strict: debug
})

```

views/index/vuex/store.js 首页单页面 vuex 组装模块导出

```js
/**
 * 引入vuex模块
 */
import login from './modules/login'   //登录注册
// ...

export default {
  login
  // ...
}
```

views/index/vuex/modules/login.js 首页单页面 vuex 分模块 login

```js
// 模块局部状态 state getters mutations actions
import * as types from 'vuexStore/mutation-types'
import {
    user
    // ...
} from 'service/api'

const state = {
    userToken: '', //token
    userInfo: {}, //用户信息
    isGetToken: false, //是否获取token
    loginStatus: false, //登录状态
    loginMessage: '', //登录的提示信息
    // ...
}

// mutaions 参数的 state 是模块局部 state 
// mutations 必须同步的，因此保证 axios 请求完成拿到数据后才能修改状态
const mutations = {
    /* 用户登录,获取token */
    [types.USER_GET_TOKEN](state, info) {
        if (info.code == 101) {
            state.userToken = info.data //token
            localStorage.aynUserToken = info.data //把token放入本地存储
            state.isGetToken = true
        } else {
            state.userToken = ''
            localStorage.aynUserToken = ''
            state.isGetToken = false
        }
        state.loginMessage = info.message //登录的提示信息
    }
    // ...
}

// actions async/await 调用 axios 请求数据，拿到数据后 提交 mutations
const actions = {
    /* 用户登录,获取token */
    getUserToken: async({
        commit
    }, params) => {
        let token = await user.GetUserToken(params) //获取token
        await commit(types.USER_GET_TOKEN, token)
    },
    /* 获取用户信息 */
    getUserInfo: async({
        commit
    }) => {
        let result = await user.GetUserInfo() //获取用户信息
        await commit(types.USER_GET_USERINFO, result)
    }
    // ...
}

// getters 对 state 同名属性简写
const getters = {
    userToken: state => state.userToken,
    userInfo: state => state.userInfo,
    isGetToken: state => state.isGetToken,
    loginStatus: state => state.loginStatus,
    loginMessage: state => state.loginMessage
    // ...
}

export default {
    state,
    mutations,
    getters,
    actions
}
```

views/index/page/index.js 首页单页面入口 js 文件

```js
/* base */
import Vue from 'assets/js/lib' // 引用 vue
/* vuex状态管理 */
import store from 'vuexStore/store'
/* 路由 */
import router from '../router/router' // 引用路由配置文件
/* vuex-router-sync 将router放入store进行管理 */
import { sync } from 'vuex-router-sync'
sync(store, router)

/* UI/动效 */
import iView from 'iview'
import 'iview/dist/styles/iview.css'    // //iview css
import 'src/assets/css/iview-theme.less' //自定义主题
Vue.use(iView)

window.vue = new Vue({
  el:'#index',
  router,
  store
})
```

## 组件中使用

例如 views/index/page/login/login.vue

```js
import {mapGetters, mapActions} from 'vuex'

export default {
    // ...
    data () {
        return {
            loginInfo: {      //登录表单
                username: '',     //用户名
                password: ''      //密码
            }
        }
    },
    computed: {
        ...mapGetters([
            'loginStatus',    // 登录状态
            'isGetToken',     //是否成功获取token
            'loginMessage'      //登录返回的提示信息
            // ...
        ])
    },
    methods: {
        ...mapActions([
            'getUserToken'    // 获取用户 token
        ]),
        async login(){
            await this.getUserToken(this.loginInfo)   // 获取token

            if(this.isGetToken){
                this.$Message.success('登录成功,正在跳转...') //登录信息提示
                setTimeout(()=>{
                    let lastPath = this.$route.query.lastPath || ''
                    // debugger
                    lastPath.indexOf('join')>-1 ? (lastPath="") : void 0

                    window.location.href = lastPath
                },1000)
            }else{
                this.$Message.error(this.loginMessage)  //登录信息提示
            }
        }
    },
    mounted() {
        this.login()
    }
}
```

## 总结

- 大型多页应用，每个页面为单页面情况下，vuex 按页面分模块（更大型情况下，modules 文件夹还可包含多个文件，每个文件 module 包含自己的 state，getters，mutations，actions）并导出

- getters 一般对 state 的同名属性简写

- 将 axios 请求分成两部分， actions 利用 async/await 封装调用 axios 请求数据，获得数据后提交 mutations （保证 mutations 为同步函数，不包含异步操作），mutations 使用保证 axios 请求完成拿到的数据修改状态

- 根目录下放置 mutation-types 常量文件、全局 mutations、全局 actions等，并将组装模块 vuex 生成store 实例

- 在每个单页面入口 js 文件引入 vuex store 实例， 生成 vue 实例

- 组件中一般使用 mapState mapGetters mapActions mapMutations 语法糖操作 vuex