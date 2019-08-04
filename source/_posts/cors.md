---
title: cors
tag: 
	- cors
---

> 参考: [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)

cors(跨域资源共享 cross-orgin resource sharing)是一个 w3c 标准，它允许浏览器向服务器发出 XMLHttpRequest 请求，克服了 ajax 只能同源使用的限制

<!-- more -->

## 简介

cors 需要浏览器和服务器支持。除了 ie10 以下的浏览器都支持

cors 通信由浏览器自动完成。浏览器一旦发现 ajax 跨域请求，会自动添加一些附加的头信息，有时还会多出一次附加请求

## 两种请求

浏览器将 cors 请求分为两类：**简单请求和非简单请求**

满足以下条件就是简单请求：

- 请求方法是三者之一： `head` `get` `post`
- http 请求头不超过这几种字段：`Accept` `Accept-Language` `Content-Language` `Last-Event-ID` `Content-Type`（`Content-Type` 只限于三个值 `application/x-www-form-urlencoded` `multipart/form-data` `text/plain`）

## 简单请求

**浏览器发现 ajax 跨域请求是简单请求，自动在头信息添加 `Origin` 字段**

```bash
GET /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

`Origin` 字段说明本次请求来自哪个源（协议 + 域名 + 端口），服务端根据该值决定是否同意这次请求

如果 `Origin` 指定的源不在许可范围，服务器会返回正常的 http 请求。浏览器发现响应头信息没有 `Access-Control-Allow-Origin`，会抛出做错误，被 XMLHttpRequest 的 onerror 函数捕获

如果 `Origin` 指定的源在许可返回，服务器返回的响应，会多出几个头信息

```js
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```

- **`Access-Control-Allow-Origin`**

**字段必须，值要么是请求时 `Origin` 字段的值，要么是 *，表示接受任何域名请求**

- `Access-Control-Allow-Credentials`

字段可选，布尔值，表示是否允许发送 Cookie。默认情况下，跨域请求不发送 Cookie。只能设置 true，表示服务器许可，Cookie 可包含在请求中发给服务器；如果服务器不要浏览器发送 Cookie，删除该字段

- `Access-Control-Allow-Expose-Headers`

字段可选，cors 请求时，XMLHttpRequest 对象的 `getResponseHeader()` 方法只能拿到 6 个基本字段：`Cache-Control` `Content-Language` `Content-Type` `Expires` `Last-Modified` `Pragma`。如果想要拿到其他字段，必须在 `Accept-Content-Expose-Headers` 中指定

### withCredentials 属性

cors 请求默认不发送 Cookie 和 http 认证信息，如果要将 Cookie 发送到服务器，一方面要服务器同意，指定 `Access-Control-Allow-Credentials: true`，另一方面要在 ajax 请求中开启 `withCredentials`

```js
var xhr = new XMLHttpRequest()
xhr.withCredentials = true
```

否则服务器同意发送，浏览器也不会发送；或者服务器要求设置 Cookie，浏览器也不会处理

如果省略 `withCredentials`，有时浏览器还会一起发送 Cookie，这时显式关闭 `xhr.withCredentials = false`

注意，服务端设置允许发送 Cookie， `Access-Control-Allow-Origin` 必须指定明确的和请求网页一直的域名；Cookie 仍然遵循同源策略，只有服务器域名设置的 Cookie 才会上传，其他域名的 Cookie 不会上传

## 非简单请求

### 预检请求

非简单请求会在正式通信前增加一次 http 查询请求，称为预检请求

**浏览器先询问服务器，当前网页所在域名是否在服务器的许可名单中，和可使用哪些 http 东西和头信息字段**；只有得到肯定回复，浏览器才会发出正式 XMLHttpRequest 请求，否则会报错

```js
var url = 'http://api.alice.com/cors';
var xhr = new XMLHttpRequest();
xhr.open('PUT', url, true);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send();
```

上述代码请求方法是 put，并且发送一个自定义请求头 `X-Custom-Header`

浏览器发现是一个非简单请求，会自动发出预检请求

```bash
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

**预检请求的请求方法是 options，表示该请求是用来询问**

- **`Allow-Control-Request-Method`**

**字段必须，用来列出浏览器 cors 请求会用到哪些方法**，上例是 put

- **`Allow-Control-Request-Headers`**

**字段是逗号分隔的字符串，指定浏览器 cors 请求会额外发送的头信息字段**

### 预检请求响应

如果服务器端收到预检请求后检查 `Origin` `Allow-Control-Request-Method` `Allow-Control-Request-Headers` 字段后，如否定预检请求，会返回正常 http 响应，但是没有任何 cors 相关头信息字段，浏览器会认定服务器不同意预检请求，会抛出做错误，被 XMLHttpRequest 的 onerror 函数捕获

如果允许跨域请求，会响应

```bash
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

- **`Allow-Control-Allow-Method`**

**字段必须，它的值是一个逗号分隔的字符串，表示服务器支持的所有跨域请求方法**

- **`Allow-Control-Allow-Headers`**

如果浏览器请求头中含有 `Access-Control-Request-Headers` ,则 `Access-Control-Allow-Headers` 字段是必须的，也是逗号分隔的字符串，表示服务器支持的所有头信息字段

- `Allow-Control-Allow-Cridentials`

和简单请求时含义相同

- **`Allow-Control-Max-Age`**

字段可选，用来指定本次预检请求有效期，单位是秒。**在有效期中不用再发出预检请求**

### 浏览器正常请求和响应

**服务器通过预检请求后，每次浏览器正常的 cors 请求都和简单请求一样，会有 `Origin` 字段，服务器的响应也会有 `Access-Control-Allow-Origin` 头信息字段**

## 和 JSONP 对比

JSONP 只支持 get 请求，cors 支持所有类型请求，JSONP 优势在于支持老式浏览器，以及可想不支持 cors 的网站请求数据