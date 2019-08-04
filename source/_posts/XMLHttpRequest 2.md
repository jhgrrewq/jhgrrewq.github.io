---
title: XMLHttpRequest 2
tag: 
	- XMLHttpRequest 2
---

> 参考: [XMLHttpRequest Level 2 使用指](http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)

XMLHttpRequest 是一个浏览器接口，使得 js 可以进行 http(s) 通信

## 旧版本 XMLHttpRequest

### 创建一个 xhr 对象

```js
var xhr
if (window.XMLHttpRequest) {
  xhr = new XMLHttpRequest()
} else {
  // ie7+ 都支持原生 XHR 对象，但是 ie5 中是由 MSXML 库中一个 ActiveXObject 创建的
  xhr = new ActiveXObject('Microsoft.XMLHttp')
}
```

<!-- more -->

### 发送请求

```js
xhr.open('GET', 'example.html', false)
```

- `open()` 第一个参数是请求方法，不区分大小写
- `open()` 第二个参数是请求路径，注意**只能是同域**
- `open()` 第三个参数是表示**是否异步请求的布尔值，如果不填写，默认是 true，表示异步发送，false 为同步请求，将会阻塞到请求完成**

```js
xhr.send(null)
```

`send()` 方法接受一个参数作为请求主体发送的数据，调用 `send()` 方法后，请求被发送到服务器

- 如果是 get 请求，`send()` 无参数或者参数是 null
- 如果是 post 请求，`send()` 参数是要发送的数据

### 监听 xhr 对象状态变化

`onreadystatechange` 用于指定回调，检测 xhr 对象的状态变化

`xhr.readyState` 有 5 个取值：

- **`0` 请求没有初始化，没有调用 open()**
- **`1` 请求已经建立，但是没有发送，没有调用 send()**
- **`2` 请求已经发送，正在处理（通常可以从响应中获取头部）**
- **`3` 请求在处理，通常响应中已经有部分数据可用，没有全部完成**
- **`4` 响应已经完成，可以获取并使用服务器的响应**

```js
xhr.onreadystatechange = function() {
  // xhr.readyState: XMLHttpRequest 对象的状态，等于 4 表示数据接收完毕
  // xhr.status: 服务器返回的状态码，返回 200 表示正常
  // xhr.responseText: 服务器返回的文本数据
  // xhr.responseXML: 服务器返回的 XML 数据
  // xhr.statusText: 服务器返回的状态文本
  if (xhr.readyState === 4 && xhr.status === 200) {
    console.log(xhr.responseText)
  } else {
    console.log(xhr.statusText)
  }
}
```

## XMLHttpRequest 2 改进

- 设置 http 请求的超时时间
- 使用 FormData 对象管理表单数据
- 使用上传文件
- 跨域请求，需要浏览器支持，并且服务器要进行对应配置
- 获取服务器端的二进制数据
- 获得数据传输的进度信息

## XMLHttpRequest 2

### 设置 http 请求的超时时间

```js
xhr.timeout = 3000
xhr.ontimeout = function() {
  alert('请求超时！')
}
```

### 使用 FormData 对象管理表单数据

```js
// 新建一个 FormData 对象
var formData = new FormData()
// 为它添加表单项
formData.append('username', 'jack')
formData.append('age', '20')
// 直接发送
xhr.send(formData)
```

FormData 对象也可以用来获取网页表单的值

```js
var form = document.getElementById('form')
// 传入 表单 dom 实例化 FormData
var formData = new FormData(form)
formData.append('secret', '1234') // 添加一个表单项
xhr.open('POST', form.action)
xhr.send(formData)
```

### 上传文件

一般通过 “选择文件” 的表单元素 `input[type="file"]`，装入 FormData 对象中

```js
var formData = new FormData()
for (var i = 0; i < files.length; i++) {
  formData.append('files'+i, files[i])
}

xhr.send(formData)
```

### 接受服务端二进制数据

```js
var xhr = new XMLHttpRequest()
xhr.open('GET', 'path/to/image.png')
// 将 responseType 设为 blob，表示服务器传回的是二进制数据
xhr.responseType = 'blob'
```

### 获得数据传输的进度信息

XMLHttpRequest 2 传输数据会有一个 progress 事件用来返回进度信息

- **下载的 progress 事件属于 XMLHttpRequest 对象**
- **上传的 progress 事件属于 XMLHttpRequest.upload 对象**

```js
// 下载
xhr.onprogress = updateProgress
// 上传
xhr.upload.onprogress = updateProgress

function updateProgress(data) {
  // 如果 data.lengthComputable 不为真，data.total 等于 0
  if (data.lengthComputable) {
    // data.loaded 已经传输的字节
    // data.total 传输的总字节
    var percent = data.loaded / data.total
  }
}
```

和 progress 事件相关的还有其他事件可以分别指定回调

- `load 事件` 传输成功完成
- `abort 事件` 传输被用户取消
- `error 事件` 传输中出现错误
- `loadstart 事件` 传输开始
- `loadend 事件` 传输结束，但是不知道成功还是失败
