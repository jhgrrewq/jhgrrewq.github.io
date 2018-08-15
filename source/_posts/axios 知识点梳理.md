---
title: axios 知识点梳理
tag: 
	- axios
---

<!-- markdownlint-disable MD010 -->

> 参考: [Axios 中文说明](https://www.kancloud.cn/yunye/axios/234845)

axios 是一个基于 promise 的 http 库，可以用在浏览器和 Node.js 中 （可以对比 jquery 的 ajax 学习）

<!-- more -->

## 基本用法

### 发送 get 请求

- **axios.get(url[, config])** 参数对象作为 params 属性传递

```javascript
// 为给定 Id 的 user 创建请求
axios.get('/user?ID=12345')
.then(function(res){
	console.log(res)
}).catch(function(e){
	console.log(e)
})

// 上面的请求 参数对象作为 params 属性传递
axios.get('/user', {
	params: {
		ID: 12345
	}
}).then(function(res){
	console.log(res)
}).catch(function(e){
	console.log(e)
})
```

### 发送 post 请求

- **axios.post(url[, config])** 参数作为对象传递

```javascript
axios.post('/user', {
	firstName: 'jack',
	lastName: 'zhang'
}).then(function(res){
	console.log(res)
}).catch(function(e){
	console.log(e)
})
```

### 并发请求

- **axios.all(promises).then(axios.spread(function(res1,res2....){}))**

```javascript
function getUserAccount() {
	return axios.get('/user/12345')
}

function getUserPermissions() {
	return axios.get('/user/12345/permissions')
}

axios.all([getUserAccount, getUserPermissions])
.then(axios.spread(function(res1, res2){
	// 所有请求完成
}))
```

axios.all 源码内部返回 Promise.all([promise1, promise2,...]).then(function(values){})
**values 是一个结果数组**

```javascript
// Expose all/spread
axios.all = function all(promises) {
  return Promise.all(promises)
};
```

axios.all 的 then() 其实全部 promise 已经执行完毕，返回结果数组，axios.spread 是将数组解构成一个个参数，传入回调

```javascript
/** 通常使用 `Function.prototype.apply`.
 *
 *  function f(x, y, z) {}
 *  var args = [1, 2, 3];
 *  f.apply(null, args);
 *
 * 相当于
 *  spread(function(x, y, z) {})([1, 2, 3]);
*/

axios.spread = function(callback) {
  // 多重函数嵌套
  return function wrap(arr) {
    return callback.apply(null, arr)
  }
```

看一个例子

```javascript
var promise1 = Promise.resolve(3);
var promise2 = 42;
var promise3 = new Promise(function(resolve, reject) {
	setTimeout(resolve, 100, 'foo');
});

axios.all([promise1, promise2, promise3]).then(function(res){
	// 这里相当于 Promise.all([promise1, promise2, promise3]).then(function(res){})
	// res 为一个结果数组，传入axios.spread, 会被解构
	axios.spread(function(res1,res2,res3){
		console.log(res1,res2,res3)
	})(res)
})

// 也可以这么写
axios.all([promise1, promise2, promise3]).then(
	// axios.spread = function(callback) { return function wrap(arr) { return callback.apply(null, arr) } }
	// axios.spread 内部先return 一层包裹函数 function wrap(arr) {} 并传入 Promise.all().then（）拿到的结果数组
	axios.spread(function(res1,res2,res3){
		console.log(res1,res2,res3)
	})
)
```

## axios api

### axios 可通过配置（config）发送请求

**axios(config)**

```javascript
// 发送 post 请求
axios({
	method: 'post',
	url: '/user/12345',
	data: {
		firstName: 'jack',
		lastName: 'zhang'
	}
})
```

**axios(url[,config])**

```javascript
// 发送 get 请求（默认方法）
axios('/user/12345')
```

### 请求别名，对所有支持的方式都提供了方便的别名

```javascript
axios.request(config)
axios.get(url[, config])
axios.delete(url[, config])
axios.head(url[, config])
axios.options(url[, config])
axios.post(url[, data[, config]])
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])
```

### 创建一个 axios 实例，并且可以自定义其配置

- axios.create(config)

```javascript
var instance = axios.create({
  baseURL: 'https://some-domain.com/api/',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});
```

## 请求配置

创建请求时可用的配置选项只有 url 是必须的，如果没有指定请求方法 method, 默认是 get 方法

```javascript
// 仅截取一些常用配置
{
	// `url` 是用于请求的服务器 URL
	url: '/user',

	// `method` 是创建请求时使用的方法
	method: 'get', // 默认是 get

	// `baseURL` 将自动加在 `url` 前面，除非 `url` 是一个绝对 URL。
	// 它可以通过设置一个 `baseURL` 便于为 axios 实例的方法传递相对 URL
	baseURL: 'https://some-domain.com/api/',

	// `transformRequest` 允许在向服务器发送前，修改请求数据
	// 只能用在 'PUT', 'POST' 和 'PATCH' 这几个请求方法
	// 后面数组中的函数必须返回一个字符串，或 ArrayBuffer，或 Stream
	transformRequest: [function (data) {
		// 对 data 进行任意转换处理

		return data;
	}],

	// `transformResponse` 在传递给 then/catch 前，允许修改响应数据
	transformResponse: [function (data) {
		// 对 data 进行任意转换处理

		return data;
	}],

	// `headers` 是即将被发送的自定义请求头
	headers: {'X-Requested-With': 'XMLHttpRequest'},

	// `params` 是即将与请求一起发送的 URL 参数
	// 必须是一个无格式对象(plain object)或 URLSearchParams 对象
	params: {
		ID: 12345
	},

	// `paramsSerializer` 是一个负责 `params` 序列化的函数
	// (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
	paramsSerializer: function(params) {
		return Qs.stringify(params, {arrayFormat: 'brackets'})
	},

	// `data` 是作为请求主体被发送的数据
	// 只适用于这些请求方法 'PUT', 'POST', 和 'PATCH'
	// 在没有设置 `transformRequest` 时，必须是以下类型之一：
	// - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
	// - 浏览器专属：FormData, File, Blob
	// - Node 专属： Stream
	data: {
		firstName: 'Fred'
	},

	// `timeout` 指定请求超时的毫秒数(0 表示无超时时间)
	// 如果请求话费了超过 `timeout` 的时间，请求将被中断
	timeout: 1000,

	// `withCredentials` 表示跨域请求时是否需要使用凭证
	withCredentials: false, // 默认的

	// `adapter` 允许自定义处理请求，以使测试更轻松
	// 返回一个 promise 并应用一个有效的响应 (查阅 [response docs](#response-api)).
	adapter: function (config) {
		/* ... */
	},

	// `auth` 表示应该使用 HTTP 基础验证，并提供凭据
	// 这将设置一个 `Authorization` 头，覆写掉现有的任意使用 `headers` 设置的自定义 `Authorization`头
	auth: {
		username: 'janedoe',
		password: 's00pers3cret'
	},

	// `responseType` 表示服务器响应的数据类型，可以是 'arraybuffer', 'blob', 'document', 'json', 'text', 'stream'
	responseType: 'json', // 默认的

	// `xsrfCookieName` 是用作 xsrf token 的值的cookie的名称
	xsrfCookieName: 'XSRF-TOKEN', // default

	// `xsrfHeaderName` 是承载 xsrf token 的值的 HTTP 头的名称
	xsrfHeaderName: 'X-XSRF-TOKEN', // 默认的

	// `onUploadProgress` 允许为上传处理进度事件
	onUploadProgress: function (progressEvent) {
		// 对原生进度事件的处理
	},

	// `onDownloadProgress` 允许为下载处理进度事件
	onDownloadProgress: function (progressEvent) {
		// 对原生进度事件的处理
	},

	// `maxContentLength` 定义允许的响应内容的最大尺寸
	maxContentLength: 2000,

	// `validateStatus` 定义对于给定的HTTP 响应状态码是 resolve 或 reject  promise 。如果 `validateStatus` 返回 `true` (或者设置为 `null` 或 `undefined`)，promise 将被 resolve; 否则，promise 将被 rejecte
	validateStatus: function (status) {
		return status >= 200 && status < 300; // 默认的
	},

	// `maxRedirects` 定义在 node.js 中 follow 的最大重定向数目
	// 如果设置为0，将不会 follow 任何重定向
	maxRedirects: 5, // 默认的

	// `httpAgent` 和 `httpsAgent` 分别在 node.js 中用于定义在执行 http 和 https 时使用的自定义代理。允许像这样配置选项：
	// `keepAlive` 默认没有启用
	httpAgent: new http.Agent({ keepAlive: true }),
	httpsAgent: new https.Agent({ keepAlive: true }),

	// 'proxy' 定义代理服务器的主机名称和端口
	// `auth` 表示 HTTP 基础验证应当用于连接代理，并提供凭据
	// 这将会设置一个 `Proxy-Authorization` 头，覆写掉已有的通过使用 `header` 设置的自定义 `Proxy-Authorization` 头。
	proxy: {
		host: '127.0.0.1',
		port: 9000,
		auth: : {
			username: 'mikeymike',
			password: 'rapunz3l'
		}
	},

	// `cancelToken` 指定用于取消请求的 cancel token
	// （查看后面的 Cancellation 这节了解更多）
	cancelToken: new CancelToken(function (cancel) {
	})
}
```

## 配置默认值 defaults

### 全局 axios 默认值

```javascript
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
```

### 自定义实例默认值

```javascript
// 创建实例时设置配置的默认值
var instance = axios.create({
  baseURL: 'https://api.example.com'
});

// 在实例已创建后修改默认值
instance.defaults.headers.common['Authorization'] = AUTH_TOKEN;
```

### 配置优先级

- 配置以一个优先顺序进行合并

- 顺序: 在源码 lib/defaults 中找到的默认值 -> 实例的 defaults 属性 -> 请求的 config 参数

- 后者会覆盖前者

```javascript
// 使用由库提供的配置的默认值来创建实例
// 此时超时配置的默认值是 `0`
var instance = axios.create();

// 覆写库的超时默认值
// 现在，在超时前，所有请求都会等待 2.5 秒
instance.defaults.timeout = 2500;

// 为已知需要花费很长时间的请求覆写超时设置
instance.get('/longRequest', {
  timeout: 5000
});
```

源码分析

```javascript
// ./lib/axios.js
var defaults = require('./defaults');

axios.create = function create(instanceConfig) {
  // utils.merge(obj1, obj2,...) 相当于 Object.assign() 合并对象
  // 这里是生成 axios 实例时 配置的 config 覆盖 lib/defaults.js 的默认配置
  return createInstance(utils.merge(defaults, instanceConfig));
};

// ./lib/core/Axios.js
function Axios(instanceConfig) {
  this.defaults = instanceConfig;
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}

Axios.prototype.request = function request(config) {
	...
	// 明显看出 请求时 config 会覆盖 创建 axois 实例的 配置， 更会覆盖 lib/defaults.js 的默认配置
	config = utils.merge(defaults, this.defaults, { method: 'get' }, config);
	...
}
```

## 拦截器

- **axios.interceptors.request.use()** 请求拦截器

```javascript
axios.interceptors.request.use(function(config) {
	// 在发送请求前做些什么
	return config
}, function(err) {
	// 对请求错误做些什么
	return Promise.reject(err)
})
```

- **axios.interceptors.response.use()** 响应拦截器

```javascript
axios.interceptors.response.use(function(response) {
	// 对响应数据做些什么
	return response
}, function(err) {
	// 对响应错误做些什么
	return Promise.reject(err)
})
```

- **eject()** 取消拦截器

```javascript
var myInterceptor = axios.interceptors.request.use(function() {
	/*...*/
})

axios.interceptors.request.eject(myIntercetor)
```

- 给自定义实例添加拦截器

```javascript
var instance = axios.create()
instance.interceptor.request.use(function(){/*...*/})
```

**添加拦截器后，请求响应流程: 请求拦截器 -> 发起请求 —> 接受响应 -> 响应拦截器 -> 处理响应数据**

源码解析

```javascript
// ./lib/core/interceptorManager.js
...
this.handlers = [];

InterceptorManager.prototype.use = function use(fulfilled, rejected) {
	// fulfilled 处理 Promise.then()
	// rejected 处理 Promise.reject()
	this.handlers.push({
		fulfilled: fulfilled,
		rejected: rejected
	});
	return this.handlers.length - 1;
};

InterceptorManager.prototype.eject = function eject(id) {
	// 将对应拦截器设为 null
	if (this.handlers[id]) {
		this.handlers[id] = null;
	}
};

InterceptorManager.prototype.forEach = function forEach(fn) {
	// 遍历 将拦截器传入函数进行调用
	utils.forEach(this.handlers, function forEachHandler(h) {
		if (h !== null) {
			fn(h);
		}
	});
};
...

// ./lib/core/Axios.js
function Axios(instanceConfig) {
  this.defaults = instanceConfig;
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}

Axios.prototype.request = function request(config) {
	...
	// Hook up interceptors middleware
	// dispatchRequest 作为请求配发的封装
  	var chain = [dispatchRequest, undefined];
  	var promise = Promise.resolve(config);

	this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
		// chain 为 [interceptor.fulfilled, interceptor.rejected, dispatchRequest, undefined]
		// 相当发起请求前先执行请求拦截器
		chain.unshift(interceptor.fulfilled, interceptor.rejected);
	});

	this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
		// chain 为 [dispatchRequest, undefined, interceptor.fulfilled, interceptor.rejected]
		// 相当接受响应后先执行响应拦截器，再处理响应数据
		chain.push(interceptor.fulfilled, interceptor.rejected);
	});

	// chain 为 [requesr_interceptor.fulfilled, requesr_interceptor.rejected, dispatchRequest, undefined, response_interceptor.fulfilled, response_interceptor.rejected]
	...
	return promise
}
```

## 取消 （通过一个 cancel token 取消请求）

- 通过 CancelToken.source 工厂函数创建一个 cancel token

CanToken.source().token 其实就是 new CancelToken(function executor(c) {cancel = c})
	

```javascript
var CancelToken = axios.CancelToken
var source = CancelToken.source()

axios.get('/user/12345', {
	cancelToken: source.token
}).catch(function(thrown) {
	if (axios.isCancel(thrown)) {
		console.log('request throw', thrown.message)
	} else {
		// 处理错误
	}
})
```

- 给CancelToken构造函数传一个 executor function 来创建一个 cancel token

```javascript
var CancelToken = axios.CancelToken
var cancel

axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    // executor 函数接收一个 cancel 函数作为参数
    cancel = c;
  })
});

// 取消请求
cancel();
```

源码分析

```javascript
// ./lib/adaptor/xhr.js ./lib/adaptor/http.js
module.exports = function xhrAdapter(config) {
	return new Promise(function dispatchXhrRequest(resolve, reject) {
		...
		if (config.cancelToken) {
			// 处理取消 请求配置中有 cancelToken 选项
			config.cancelToken.promise.then(function onCanceled(cancel) {
				if (!request) {
					return;
				}
				// 其实是调用 xhr.abort()
				request.abort();
				reject(cancel);
				// 清除 request
				request = null;
			});
		}
		...
		// 发送请求
		request.send(requestData);
	}
}

// ./lib/cancel/CancelToken.js
function CancelToken(executor) {
	if (typeof executor !== 'function') {
		throw new TypeError('executor must be a function.');
	}

	var resolvePromise;
	this.promise = new Promise(function promiseExecutor(resolve) {
		resolvePromise = resolve;
	});

	var token = this;
	executor(function cancel(message) {
		if (token.reason) {
			// Cancellation has already been requested
			return;
		}

		token.reason = new Cancel(message);
		resolvePromise(token.reason);
	});
}

CancelToken.source = function source() {
	// CanToken.source().token 其实就是 new CancelToken(function executor(c) {cancel = c})
	var cancel;
	var token = new CancelToken(function executor(c) {
		cancel = c;
	});
	return {
		token: token,
		cancel: cancel
	};
};

```