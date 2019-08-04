---
title: axios 二次封装
tag: 
	- axios
---

<!-- markdownlint-disable MD010 -->

一般项目往往要对 axios 库进行二次封装，添加一些自定义配置和拦截器等

<!-- more -->
## 案例

./service/axios.js

```javascript
/**
*  说明：axios 二次封装
*  参数：type       //请求的HTTP方法，例如：'GET', 'POST'或其他HTTP方法
*       apiName    //接口地址
*       url        //接口模块
*       options    //配置参数,例如传{params={}}
*/

import axios from 'axios'
import api from 'service/apiConfig' //真实接口配置
import store from 'vuexStore/store.js' //引用vuex
import qs from 'qs'
import template from 'url-template'

/**  axios基础配置 */
axios.defaults.timeout = 5000;
axios.defaults.headers['Content-Type'] = 'application/json;charset=UTF-8'

// http request 拦截器
// 一般设置 token
axios.interceptors.request.use(
  config => {
    if (localStorage.aynUserToken) {
      config.headers.common['X-AIYANGNIU-SIGNATURE'] = localStorage.aynUserToken;
    }
    return config;
  },
  err => {
    return Promise.reject(err);
  }
);

// http response 拦截器
axios.interceptors.response.use(
  response => {
    if (response.data) {
      let code = response.data.code
      switch (code) {
        case 109: // 109 清除token信息并跳转到登录页面
          localStorage.aynUserToken = ''
        //   vue.$loginVerify()
          break;
        case 110:
        //   vue.$roleVerify()
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
        //   vue.$loginVerify()
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
  // url拼接
  let path = (apiName != '') ? api[apiName] + url : url
  // url-template
  let UrlTemplate = template.parse(path)
  // 格式化地址,若data传入参数是{params:{xx:"xxx",xxx:"xxxx"}}结构的，expand要取{xx:"xxx",xxx:"xxxx"}来格式化url
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

  // 设置 {form: true}  post/put data转参数拼接字符串
  if (options && options.form) {
    let opt = {
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }
    // 利用 qs 库的stringify() 方法拼接参数字符串
    data = qs.stringify(data)
    options === {} ? options.headers = opt.headers : options = opt
  }
  // axios
  await axios[type](path, data, options).then((res) => {
    // res.data 才是响应数据
    vm.result = res.data
  })
  return vm.result
}
```

调用

- get/delete 传数据 {} 或者 {params}

- post/put 传输数据 params（对象）

- 如果是表单提交 {form: true}

```javascript
import res from 'service/axios'

/* 加入进货单、加入采购申请 */
export const AddToCart = (params) => res('post', 'apiItem', '/carts/add', params, {form:true})

/* 获取进货单列表 */
export const GetCartList = () => res('get', 'apiItem', '/carts/list', {})
/* 更新购物车商品数量 */
export const CartUpdate = (params) => res('put','apiItem', '/carts/update/{id}',params, {form:true})

/* 根据购物车记录编号(列表时可批量)删除 */
export const CartDelete = (params) => res('delete','apiItem', '/carts/delete',{params})
...
```

## 补充

url-template 库使用

```javascript
var template = require('url-template');

var emailUrl = template.parse('/{email}/{folder}/{id}');

// Returns '/user@domain/test/42'
emailUrl.expand({
  email: 'user@domain',
  folder: 'test',
  id: 42
});
```