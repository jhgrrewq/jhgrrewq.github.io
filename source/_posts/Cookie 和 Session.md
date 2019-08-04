---
title: Cookie 和 Session
tag: 
	- Cookie
	- Session
---

## Cookie

http 是一个无状态的协议，业务中往往需要一定的状态，否则无法区分用户之间的身份，最早的方案就是 Cookie

Cookie 的处理分为如下几步：

- **服务器向客户端发送 Cookie**
- **浏览器保存 Cookie**
- **之后每次浏览器都会将 Cookie 发给服务器（同域）**

<!-- more -->

### 请求头 Cookie

请求中 Cookie 在请求头 `Cookie` 字段中 **Cookie 值的格式是 key=value;key2=value2**

### 响应头 Set-Cookie

响应中 Cookie 在响应头 `Set-Cookie` 字段中

```bash
Set-Cookie: name=value;Path=/;Expires=Sun, 23-Apr-23 09:01:35 GMT; Domain=domain.com
```

`name=value` 是必须包含的部分，其余都是可选参数。几个主要选项如下：

- `Path`: 表示该 Cookie 影响到的路径，当前访问的路径不满足该匹配时，浏览器则不发送这个 Cookie
- `Expires` `Max-Age`: 用来告知浏览器该 Cookie 何时过期。不设置该选项，则关闭浏览器会丢失该 Cookie。如果设置了过期时间，浏览器会把 Cookie 内容写入磁盘并保存。`Expires` 值是一个 UTC 格式的时间字符串，**告诉浏览器该 Cookie 何时过期**；`Max-Age` 则**告诉浏览器多久过期**。如果服务器端和客户端时间不匹配，`Expires` 这种时间设置会造成偏差
- `HttpOnly`: **告知浏览器不允许通过脚本 document.cookie 去更改该 Cookie**
- `Secure`: 当为 true，在 http 中无效，在 https 中有效，表示创建的 Cookie 只能在 https 中被浏览器传递到服务器端进行会话验证，如果是 http 则不会传递该信息，所以很难被监听到

### Cookie 性能影响

同域下，除非 Cookie 过期，客户端每次请求都会带上 Cookie，如果 Cookie 过多将会导致报头过大，并且大多数 Cookie 并不需要每次都带上，会造成带宽的部分浪费

避免方法就是减小 Cookie 的大小。极端情况下根域名设置 Cookie，几乎所有子路径下的请求都会携带 Cookie。这是在设计上可以限定 Cookie 的域，特别是一些静态内容可以使用不同的域名

## Session

Cookie 除了体积过大的问题，还有更严重的问题就是 **Cookie 在前后端都可以精心修改**，因此 Cookie 对于敏感数据的保护是无效的

Session 的数据只保留在服务端，客户端无法修改。如何将每个客户和服务器中的数据一一对应，一般**基于 Cookie 实现用户和数据的映射** 也就是不能将所有数据放在 Cookie 中，但是可以将 口令 放在 Cookie 中，口令是唯一并且不重复的，一旦篡改将失去映射关系

### 基于 Cookie 实现用户和数据的映射

**一旦服务器启用 Session，预定一个键值作为 Session 的口令（token），如 tomcat
中使用 jsessionid。一旦服务器中检查到用户请求 Cookie 中没有携带该值，它就会位置生成一个值，这个值是唯一并且不重复的值，并设定超时时间**

#### 生成 session

```js
var sessions = {}
var key = 'session_id'
var EXPIRES = 20 * 60 * 1000

var generate = function() {
  var session = {}
  session.id = (new Date()).getTime() + Math.random()
  session.cookie = {
    // 设置何时过期
    expire: (new Date()).getTime() + EXPIRES
  }
  session[session.id] = session
  return session
}
```

#### 请求到来检查 Cookie 中的口令和服务端数据，过期重新生成并在响应中给客户端设新值

```js
function(req, res) {
  var id = req.cookies[key]
  if (!id) {
    req.session = generate()
  } else {
    var session = sessions[id]
    if (session) {
      if (session.cookie.expire > (new Date()).getTime()) {
        // 更新超时时间
        session.cookie.expire = (new Date()).getTime() + EXPIRES
        req.session = session
      } esle {
        // 超时，删除旧数据，并重新生成
        delete session[id]
        req.session = generate()
      }
    } else {
      // 如果 session 过期或者口令不对，重新生成 session
      req.session = generate()
    }
  }
  // 在客户端设置新值
  var cookies = res.getHeader('Set-Cookie')
  var session = serialize(key, req.session.id)
  cookies = Array.isArray(cookies) ? cookies.concat(session) : [cookies, session]
  res.setHeader('Set-Cookie', cookies)

  // todo
}
```