---
title: koa2 源码解读
tag: 
	- Node.js
	- koa2
---
<!-- markdownlint-disable MD010 -->

> koa 是由 Express 原班人马打造的，致力于成为一个更小、更富有表现力、更健壮的 Web 框架。 使用 koa 编写 web 应用，通过组合**不同的 generator(koa2 用 async/await)**，可以免除重复繁琐的回调函数嵌套， 并极大地提升错误处理的效率。**koa 不在内核方法中绑定任何中间件**， 它仅仅提供了一个轻量优雅的函数库，使得编写 Web 应用变得得心应手

## 代码结构

```js
- lib/
-- application.js // 入口文件
-- context.js
-- request.js
-- response.js
```

<!-- more -->

## 基础

```js
const http = require('http')

http.createServer(function(req, res, next) {
  // res.writeHead(statusCode, [reason], [headers])
  res.writeHead(200, {'Content-Type': 'text/html;charset=UTF-8'})
  // res.statusCode = 200
  // res.setHead('Content-Type', 'text/html;charset=UTF-8')
  res.end('hi')
}).listen(3000, function() {
  console.log('listen at port 3000')
})
```

使用 Node.js 原生创建一个 http 服务器的代码, 实际上是调用 http.createServer(requestListener), 创建一个 http.Server 的实例，http.Server 继承于 EventEmitter 类。传入的 requestListener 是一个函数，会自动添加到 'request' 事件，最后将 http.Server 的实例绑定到 3000 端口，开始监听来自 3000 端口的请求

requestListener 函数接受两个参数 req 和 res，req 是一个可读流，res 是一个可写流。通过 req 获取该 http 请求的所有信息，同时将数据写入 res 来对请求做出响应

```js
// Node.js 源码 http.js
const server = require('_http_server');
const { Server } = server;
function createServer(requestListener) {
  return new Server(requestListener);
}

// Node.js 源码 _http_server.js
// Server 构造函数 传入 requestListener，如果 requestListener 存在会被添加到 'request' 事件
function Server(requestListener) {
  if (!(this instanceof Server)) return new Server(requestListener);
  net.Server.call(this, { allowHalfOpen: true });

  if (requestListener) {
    this.on('request', requestListener);
  }
  ...
}
```

## application.js

使用 koa2 创建一个 server 代码非常简单

```js
const koa = require('koa')
const app = new koa()

app.use(function (ctx, next) {
  ctx.body = 'hi'
  ctx.type = 'text/html; charset=urf-8'
})

app.listen(3000, () => {
  console.log('listen at port 3000')
})
```

### 构造函数

application.js 实现并导出一个构造函数，该构造函数继承 Emitter 类，context, request, response 是三个字面量创建的对象，封装了一些方法和属性（大部分都是属性的 get 和 set）, 同时将这三个对象作为原型生成的对象缓存在 application 实例的对应属性中

```js
module.exports = class Application extends Emitter {
  constructor() {
    super();

    this.proxy = false;
    this.middleware = []; // 存放中间件数组，每个元素都是一个中间件函数
    this.subdomainOffset = 2;
    this.env = process.env.NODE_ENV || 'development';
    this.context = Object.create(context);
    this.request = Object.create(request);
    this.response = Object.create(response);
  }

  listen(...arg) {
    ...
  }

  use(fn) {
    ...
  }

  callback() {
    ...
  }

  createContext(req, res){
    ...
  }
};

function responed(ctx) {
  ...
}
```

### use(fn)

构造函数原型方法 **use 方法实质是注册一个中间件**，将传入的中间件 push 到自身的 middlewares 数组中。传入的中间件必须是函数，在 koa2 中，如果传入的中间件是 generator 方法，会调用 koa-convert 模块进行转化

```js
use(fn) {
  // 中间件 fn 必须是函数，否则报错
  if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
  // 对 generator 函数进行转换
  if (isGeneratorFunction(fn)) {
    deprecate('Support for generators will be removed in v3. ' +
              'See the documentation for examples of how to convert old middleware ' +
              'https://github.com/koajs/koa/blob/master/docs/migration.md');
    fn = convert(fn);
  }
  debug('use %s', fn._name || fn.name || '-');
  // 将传入的中间件 push 到自身 middleware 数组中
  this.middleware.push(fn);
  // 返回 this 进行链式调用
  return this;
}
```

### listen(...arg)

在 Node.js 原生代码中，通过向 http.createServer 传递一个函数来创建一个  http.Server 实例来启动应用，构造函数原型方法 **listen 方法就是对其封装，只不过 http.createServer 会传入自身 callback 的调用**

```js
listen(...args) {
  debug('listen');
  const server = http.createServer(this.callback());
  return server.listen(...args);
}
```

### callback()

this.callback() 返回的肯定是一个函数，而这个函数就是根据 req 获取信息，同时向 res 中写入数据。

该函数的具体做法是，**它基于 req 和 res 封装出中间件需要的 ctx 对象，再将 ctx 传递给中间件组合成的一个嵌套函数。中间件组合的嵌套函数返回一个 promise 实例，等到这个组合函数执行完毕（resolve），通过 ctx 向 res 中写入数据，如果执行过程出错，会调用默认的错误处理函数**

```js
callback() {
  // 将中间件数组传入 compose 函数，得到一个中间件组合的嵌套函数
  const fn = compose(this.middleware);

  // 默认注册错误处理函数
  if (!this.listeners('error').length) this.on('error', this.onerror);

  // 返回的函数，都会先通过 req 和 res 创建一个函数上下文 ctx
  // 再将 ctx 和 中间件组合的嵌套函数传入 this.hanleRequest(ctx, fn)
  const handleRequest = (req, res) => {
    const ctx = this.createContext(req, res);
    return this.handleRequest(ctx, fn);
  };

  return handleRequest;
}

// 中间件组合的嵌套函数返回一个 promise 实例
// ctx 传入中间件组合的嵌套函数，中间件组合的嵌套函数执行完毕（resolved）, 会调用 handleResponse 方法，通过 ctx 向 res 写入数据，如果出错，会调用 onerror 函数
handleRequest(ctx, fnMiddleware) {
  const res = ctx.res;
  res.statusCode = 404;
  // 请求出错则会调用 ctx.onerror
  const onerror = err => ctx.onerror(err);
  // respond 方法就是就是根据 ctx，集中向 res 写入数据，响应 http 请求
  const handleResponse = () => respond(ctx);
  // onFinished 方法用于监听 http response 的结束事件并执行回调
  onFinished(res, onerror);
  return fnMiddleware(ctx).then(handleResponse).catch(onerror);
}

onerror(err) {
  assert(err instanceof Error, `non-error thrown: ${err}`);

  if (404 == err.status || err.expose) return;
  if (this.silent) return;

  const msg = err.stack || err.toString();
  console.error();
  console.error(msg.replace(/^/gm, '  '));
  console.error();
}
```

为方便理解，将一些错误处理函数代码和一些边界条件判断精简后

```js
callback() {
  const fn = compose(this.middleware);

  return (req, res) => {
    const ctx = this.createContext(req, res);
    return (ctx, fn) => {
      ctx.res.statusCode = 404;
      onFinished(res, onerror);
      return fn(ctx).then(() => respond(ctx))
    };
  };
}
```

koa1 的代码也十分类似

```js
app.callback = function(){
  // 设置 this.experimental 可以在中间件中使用 async/await
  if (this.experimental) {
    console.error('Experimental ES7 Async Function support is deprecated. Please look into Koa v2 as the middleware signature has changed.')
  }
  // 将中间件数组传入 compose 函数，得到一个中间件组合的嵌套函数
  // 这里会针对 async/await 和 generator 函数调用不同的模块转化返回中间件组合成的嵌套函数
  var fn = this.experimental
    ? compose_es7(this.middleware)
    : co.wrap(compose(this.middleware));
  var self = this;

  if (!this.listeners('error').length) this.on('error', this.onerror);

  return function(req, res){
    res.statusCode = 404;
    // 同样通过 req 和 res 生成函数上下文 ctx
    var ctx = self.createContext(req, res);
    onFinished(res, ctx.onerror);
    // 中间件组合的嵌套函数传入 ctx，执行完毕（resolve）调用 respond 方法根据 ctx 对 res 写入数据，出错调用默认的错误处理函数
    fn.call(ctx).then(function () {
      respond.call(ctx);
    }).catch(ctx.onerror);
  }
};
```

### respond(ctx)

respond 方法是 application 模块的私有方法，就是根据 ctx ，向 res 集中写入数据，响应 http 请求

```js
function respond(ctx) {
  if (false === ctx.respond) return;

  const res = ctx.res;
  if (!ctx.writable) return;

  let body = ctx.body;
  const code = ctx.status;

  // ignore body
  if (statuses.empty[code]) {
    // strip headers
    ctx.body = null;
    return res.end();
  }

  if ('HEAD' == ctx.method) {
    if (!res.headersSent && isJSON(body)) {
      ctx.length = Buffer.byteLength(JSON.stringify(body));
    }
    return res.end();
  }

  // status body
  if (null == body) {
    // body null， 状态码默认是 404， message 默认是 not found
    body = ctx.message || String(code);
    if (!res.headersSent) {
      ctx.type = 'text';
      ctx.length = Buffer.byteLength(body);
    }
    return res.end(body);
  }

  // responses 对 body 不同类型，返回不同数据
  if (Buffer.isBuffer(body)) return res.end(body);
  if ('string' == typeof body) return res.end(body);
  if (body instanceof Stream) return body.pipe(res);

  // body: json
  body = JSON.stringify(body);
  if (!res.headersSent) {
    ctx.length = Buffer.byteLength(body);
  }
  res.end(body);
}
```

### createContext(req, res)

其实是将context, request, response 作为原型创建的对象指定为 app 中对应的属性，将原生 req 和 res 赋给相应的属性

```js
 createContext(req, res) {
    const context = Object.create(this.context);
    const request = context.request = Object.create(this.request);
    const response = context.response = Object.create(this.response);
    // 为了方便调用，context request respones 都有相应 app req res 对象属性，记住 context.respone / context.request 访问的是 koa 封装的 response / request 对象属性方法, context.req / context.res 访问的是 Node.js 封装的 req / res 对象属性方法 
    context.app = request.app = response.app = this;
    context.req = request.req = response.req = req;
    context.res = request.res = response.res = res;
    request.ctx = response.ctx = context;
    request.response = response;
    response.request = request;
    context.originalUrl = request.originalUrl = req.url;
    context.cookies = new Cookies(req, res, {
      keys: this.keys,
      secure: request.secure
    });
    request.ip = request.ips[0] || req.socket.remoteAddress || '';
    context.accept = request.accept = accepts(req);
    context.state = {};
    return context;
  }
```

## context.js

该模块中，将 request 对象和 response 对象的一些方法和属性委托给了 context 对象, 并导出 context 对象

```js
var proto = module.exports = {

  assert: httpAssert,

  throw: function(){
    throw createError.apply(null, arguments);
  },

  onerror: function(err){
    ...
  }
};

// 将 request 对象和 response 对象的一些方法和属性委托给 context 对象
delegate(proto, 'response')
  .method('attachment')
  .method('redirect')
  .method('remove')
  .method('vary')
  .method('set')
  .method('append')
  .access('status')
  .access('message')
  .access('body')
  .access('length')
  .access('type')
  .access('lastModified')
  .access('etag')
  .getter('headerSent')
  .getter('writable');

delegate(proto, 'request')
  .method('acceptsLanguages')
  .method('acceptsEncodings')
  .method('acceptsCharsets')
  .method('accepts')
  .method('get')
  .method('is')
  .access('querystring')
  .access('idempotent')
  .access('socket')
  .access('search')
  .access('method')
  .access('query')
  .access('path')
  .access('url')
  .getter('origin')
  .getter('href')
  .getter('subdomains')
  .getter('protocol')
  .getter('host')
  .getter('hostname')
  .getter('header')
  .getter('headers')
  .getter('secure')
  .getter('stale')
  .getter('fresh')
  .getter('ips')
  .getter('ip');
```

第三方模块 delegates 是将 koa 的 request 和 response 模块的方法属性委托到 context 对象中, 因此如下一些写法是等价的:

- ctx.body === ctx.response.body
- ctx.status === ctx.response.status
- ctx.href === ctx.request.href
- ctx.search === ctx.request.search

```js
// 导出一个构造函数
module.exports = Delegator

function Delegator(proto, target) {
  // 调用直接生成返回一个实例
  if (!(this instanceof Delegator)) return new Delegator(proto, target);
  // 将传入的 proto target 缓存到自身同名属性
  this.proto = proto;
  this.target = target;
  this.methods = [];
  this.getters = [];
  this.setters = [];
  this.fluents = [];
}

Delegator.prototype.method = function(name){
  var proto = this.proto;
  var target = this.target;
  this.methods.push(name);

  // 将 target 的 method 直接挂在到 proto 同名属性
  proto[name] = function(){
    return this[target][name].apply(this[target], arguments);
  };

  return this;
};

Delegator.prototype.access = function(name){
  // 将 target 的 属性 直接挂在到 proto 同名属性，该属性是 getter 也是 setter
  return this.getter(name).setter(name);
};

Delegator.prototype.getter = function(name){
  var proto = this.proto;
  var target = this.target;
  this.getters.push(name);

  // 设置 proto 同名属性的 getter
  proto.__defineGetter__(name, function(){
    return this[target][name];
  });

  return this;
};

Delegator.prototype.setter = function(name){
  var proto = this.proto;
  var target = this.target;
  this.setters.push(name);

  // 设置 proto 同名属性的 setter
  proto.__defineSetter__(name, function(val){
    return this[target][name] = val;
  });

  return this;
};

...
```

## request.js

request 模块导出一个对象，对象的属性设置 getter 和 setter，封装了 Node.js 中 req 对象的一些属性方法

同理 response.js

```js
module.exports = {
  get header() {
    return this.req.headers;
  },
  get headers() {
    return this.req.headers;
  },

  get url() {
    return this.req.url;
  },
  set url(val) {
    this.req.url = val;
  }
  ...
}
```

## 中间件

对于每一个请求都会调用中间件进行处理。在 koa 中，中间件通过 use 方法注册，koa2 中间件函数不能是 generator 函数，可以使用 async / await

**在 koa 中，中间件的执行顺序是 "洋葱模型结构"，嵌套调用**，第一个中间件调用第二个中间件，第二个中间件调用第三个中间件，依次类推。**上游的中间件必须等到下游的中间件返回结果才能继续执行**

koa 中可以注册多个中间件，**但如果多个中间件中，其中一个中间件缺少 await next()，后面的中间件都不会执行**

```js
// 每个中间件都有 await next()
const koa = require('koa')
const app = new koa()

app.use(async (ctx, next) => {
  console.log('before 1')
  await next()
  console.log('after 1')
})

app.use(async function (ctx, next) {
  console.log('before 2')
  await next()
  console.log('after 2')
})

app.listen(3000, () => {
  console.log('new')
})

// before 1
// before 2
// after 2
// after 1
```

```js
// 第一个中间件缺少 await next()，后面的中间件不会执行
app.use(async (ctx, next) => {
  console.log('before 1')

  console.log('after 1')
})

app.use(async function (ctx, next) {
  console.log('before 2')
  await next()
  console.log('after 2')
})

// before 1
// after 1
```

koa2 源码使用 **koa-compose 模块对全部注册的中间件组合生成一个嵌套函数，该函数返回一个 promise 实例**

```js
function compose (middleware) {
  // 检验，传入参数必须是中间件数组，并且每个元素都是函数，否则报错
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  // 返回一个函数
  return function (context, next) {
    let index = -1
    return dispatch(0)

    function dispatch (i) {
      // 检验
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      // 首先获取第一个中间件函数
      index = i
      let fn = middleware[i]

      // 当最后一个中间件时, 后面已经没有中间件， next 为 undefined，fn = next = undefined，返回 Promise.resolve()
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()

      try {
        // 对中间件函数进行调用，传入 ctx 和 next
        // 返回一个 promise 实例，并将结果 resolve
        return Promise.resolve(fn(context, function next () {
          // 遇到 await next(), 调用 dispatch(i + 1)，调用下一个中间件
          return dispatch(i + 1)
        }))
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

## 源码学习心得

- 通过 README.md 或者 api 文档熟悉 模块用法
- 先从入口文件入手（package.json 的 main字段 或者 项目默认的入口文件）
- **了解整体架构流程，不要一上来就扣细节**
- 通过比对 api 的用法梳理 模块 或 函数 的功能和流程
- **通过过滤掉一些边界条件和检测代码，只保留主干来了解核心代码**
