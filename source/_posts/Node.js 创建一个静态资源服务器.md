---
title: Node.js 创建一个静态资源服务器
tag: 
	- Node.js
	- 静态资源服务器
---

<!-- markdownlint-disable MD010 -->

> 原理就是根据请求 url 的不同进行判断，如果是文件则返回文件内容，如果是目录则列出该目录下的文件列表

```tree
├── README.md
├── bin												// 可执行文件目录
│   └── anywhere							// 可执行文件命令文件
├── package.json
└── src
    ├── app.js								// Server 类
    ├── config								// 参数配置
    │   └── defaultConfig.js
    ├── helper
    │   ├── cache.js					// 缓存
    │   ├── compress.js       // 压缩编码
    │   ├── mime.js           // 获取文件 mimeType
    │   ├── range.js          // 部分内容请求
    │   └── route.js          // 服务器回调处理
    ├── index.js              // 读取命令行参数、实例化 Server 并启动应用
    └── template
        └── dir.html          // 文件列表模板
```

<!-- more -->

## 静态资源服务器基本功能

### 入口 app.js

利用 http 模块创建一个 http 服务器

```js
// 入口 app.js
const http = require('http')
const path = require('path')
const config = require('./config/defaultConfig')
const route = require('./helper/route')

const server = http.createServer((req, res) => {
    const filePath = path.join(config.root, req.url)
    console.log(filePath)
    route(req, res, filePath)
})

server.listen(config.port, config.hostname, () => {
    const info =  `http://${config.hostname}:${config.port}`
    console.log(`server is listening at ${info}` )
})
```

### 文件列表模板

使用 handlebars 语法创建文件列表模板

```html
<!-- template/dir.html -->
{{#each files}}
<a href="{{../dir}}/{{file}}">【{{icon}}】{{file}}</a>
{{/each}}
```

### 服务器回调处理

编译模板，同时利用 `fs.stats` 读取 filePath 信息，判断是文件则读取文件流输出到 res，如果目录则组装数据渲染文件列表模板

```js
// helper/route.js
// async await await 跟着promise会等待promise resolve返回结果
// 异步变同步使用async/await 异步封装成promise

const Handlebars = require('handlebars')
const fs = require('fs')
const path = require('path')
const config = require('../config/defaultConfig')
const mime = require('../helper/mime')
const promisify = require('util').promisify
const stat = promisify(fs.stat)
const readdir = promisify(fs.readdir)

const tpl = '../template/dir.html'
const filePath = path.join(__dirname, tpl) // 除了require以外，尽量将路径处理成绝对路径
const source = fs.readFileSync(filePath) // 此处用同步 首先是因为需要编译成模板才能渲染数据，其次放在函数外只需要编译一次，node的require下次就会从缓存加载
const template = Handlebars.compile(source.toString()) // 读文件出来的buffer

module.exports = async function route(req, res, filePath) {
    try{
        const stats = await stat(filePath)
        if (stats.isFile()) {
            const mimeType = mime(filePath)
            res.statusCode = 200
            res.setHeader('Content-type', mimeType)
            fs.createReadStream(filePath).pipe(res)
            // fs.readFile(filePath, (err, data) => {
            //     res.end(data.toString())
            // })  // 将文件全部读入内存再返回 太慢
        } else if (stats.isDirectory()) {
            const files = await readdir(filePath)
            res.statusCode = 200
            res.setHeader('Content-type', 'text/html')
            const dir = path.relative(config.root, filePath) // 返回第二个相对于第一个的路径
            const data = {
                dir: dir ? `/${dir}` : '', // 需要加上相对网站的根路径 /
                files: files.map(file => {
                    return {
                        file,
                        icon: mime(file)
                    }
                })
            }
            res.end(template(data))
        }
    } catch(ex) {
        console.log(ex)
        res.statusCode = 404
        res.setHeader('Content-type', 'text/plain')
        res.end(`${filePath} is not a file or directory`)
    }
}
```

## 压缩编码功能

根据客户端请求头 `Accept-Encoding` 支持的压缩编码，服务器使用 `zlib` 模块的 `createGzip` `createDeflate` 方法对静态资源进行压缩，同时响应头 `Content-Encoding` 写入对应压缩编码格式

```js
// helper/compress.js
const { createGzip, createDeflate } = require('zlib')

module.exports = (rs, req, res) => {
  // 获取客户端允许压缩编码
  const acceptEncoding = req.headers['accept-encoding']
  if (!acceptEncoding || !acceptEncoding.match(/\b(gzip|deflate)\b/)) {
    // 不做压缩
    return rs
  } else if (acceptEncoding.match(/\bgzip\b/)) {
    res.setHeader('Content-Encoding', 'gzip')
    return rs.pipe(createGzip())
  } else if (acceptEncoding.match(/\bdeflate\b/)) {
    res.setHeader('Content-Encoding', 'deflate')
    return rs.pipe(createDeflate())
  }
}
```

```js
// helper/route.js
const compressFile = require('./compress')
// ...
module.exports = async function(req, res, filePath, config) {
  try {
    const stats = await stat(filePath)
    if (stats.isFile()) {
      // 如果是文件返回文件内容
      const mimeType = mime(filePath)
      res.setHeader('Content-Type', mimeType

      const rs = fs.createReadStream(filePath)

      if (config.compress.test(filePath)) {
        rs = compressFile(rs, req, res)
      }

      rs.pipe(res)
    } else if (stats.isDirectory()) {
      // ...
    }
  } catch (error) {
    // ...
  }
}
```

## 缓存

根据客户端请求头 `Expires` `Cache-Control` 判断是否支持强缓存，根据客户端请求头 `If-Modified-Since` `If-None-Match` 和服务端响应头 `Last-Modified` `Etag` 判断协商缓存是否有效，无效如返回状态码 200、响应体为资源内容的响应，有效则返回返回状态码 304、响应体为空的响应

```js
// helper/cache.js
const { cache } = require('../config/defaultConfig')

const refreshRes = (stats, res) => {
  const { maxAge, expires, cacheControl, lastModified, etag } = cache

  if (expires) {
    // Expires 是超时的绝对时间 UTC 字符串
    res.setHeader('Expires', new Date(Date.now() + maxAge * 1000).toUTCString())
  }

  if (cacheControl) {
    res.setHeader('Cache-Control', `public, max-age: ${maxAge}`)
  }

  if (lastModified) {
    // 上次修改的绝对时间
    // 这里用 stats.mtime
    res.setHeader('Last-Modified', stats.mtime.toUTCString())
  }

  if (etag) {
    // 这里是简单用 stats 的 size 和 mtime 生成 etag
    res.setHeader('Etag', `${stats.size}-${stats.mtime.toUTCString()}`)
  }
}

module.exports = function isFresh(stats, req, res) {
  refreshRes(stats, res)

  // 协商缓存 请求头
  const lastModified = req.headers['if-modified-since']
  const etag = req.headers['if-none-match']

  if (!lastModified && !etag) {
    return false
  }

  // 客户端上次修改时间和服务器修改时间对比
  if (lastModified && lastModified !== res.getHeader('last-modified')) {
    return false
  }

  // 客户端带过来 etag 和服务器 etag 对比
  if (etag && etag !== res.getHeader('etag')) {
    return false
  }

  return true
}
```

```js
// helper/route.js
const compressFile = require('./compress')
const isFresh = require('./cache')
// ...
module.exports = async function(req, res, filePath, config) {
  try {
    const stats = await stat(filePath)
    if (stats.isFile()) {
      // 如果是文件返回文件内容
      const mimeType = mime(filePath)
      res.setHeader('Content-Type', mimeType

      if (isFresh(stats, req, res)) {
        // 协商缓存有效，直接返回状态码 304 响应体为空的响应
        res.statusCode = 304
        res.end()
        return
      }

      const rs = fs.createReadStream(filePath)

      if (config.compress.test(filePath)) {
        rs = compressFile(rs, req, res)
      }

      rs.pipe(res)
    } else if (stats.isDirectory()) {
      // ...
    }
  } catch (error) {
    // ...
  }
}
```

## 部分请求

根据客户端请求头 `Range`, 服务器判断如果是部分内容请求，设置响应头， `Content-Range` `Conetnt-Length`, 返回状态码 206、响应体为对应部分内容的响应; 否则返回返回状态码 200、响应体为资源内容的响应

```js
// helper/range.js

// 请求头 Range:bytes=start-end

// 响应头 Content-Range: bytes start-end/total
//       Conetnt-Length: total

module.exports = (totalSize, req, res) => {
  const range = req.headers['range']

  if (!range) {
    return {code: 200}
  }

  const size = range.match(/bytes=(\d*)-(\d*)/)
  const end = size[2] ? parseInt(size[2]) : totalSize - 1
  const start = size[1] ? parseInt(size[1]) : totalSize - end

  if (end < start || start < 0 || end > totalSize) {
    return {code: 200}
  }

  res.setHeader('Content-Range', `bytes ${start}-${end}/${totalSize}`)
  res.setHeader('Content-Length', end - start)

  // 说明是部分内容请求，statusCode 206
  return {
    code: 206,
    start,
    end
  }
}
```

```js
// helper/route.js
const compressFile = require('./compress')
const isFresh = require('./cache')
const range = require('./range')
// ...
module.exports = async function(req, res, filePath, config) {
  try {
    const stats = await stat(filePath)
    if (stats.isFile()) {
      // 如果是文件返回文件内容
      const mimeType = mime(filePath)
      res.setHeader('Content-Type', mimeType

      if (isFresh(stats, req, res)) {
        // 协商缓存有效，直接返回状态码 304 响应体为空的响应
        res.statusCode = 304
        res.end()
        return
      }

      let rs
      const { code, start, end } = range(stats.size, req, res)

      if (code === 200) {
        res.statusCode = 200
        rs = fs.createReadStream(filePath)
      } else if (code === 206) {
        res.statusCode = 206
        // 读取部分流
        rs = fs.createReadStream(filePath, { start, end })
      }

      if (config.compress.test(filePath)) {
        rs = compressFile(rs, req, res)
      }

      rs.pipe(res)
    } else if (stats.isDirectory()) {
      // ...
    }
  } catch (error) {
    // ...
  }
}
```

对于部分内容请求可以用 curl 来测试

```bash
curl -r 0-10  -i http://127.0.0.1:9527/src/app.jsp

HTTP/1.1 206 Partial Content
Content-Type: text/javascript
Expires: Thu, 28 Jun 2018 05:12:00 GMT
Cache-Control: public, max-age: 600
Last-Modified: Wed, 27 Jun 2018 10:00:40 GMT
Etag: 1204-Wed, 27 Jun 2018 10:00:40 GMT
Content-Range: bytes 0-10/1204
Content-Length: 10
Date: Thu, 28 Jun 2018 05:02:00 GMT
Connection: keep-alive

// 构建?
```

## 包装成脚手架

### 使用 yarns 读取命令行参数启动应用

首先将 `app.js` 改造成一个 Server 类并导出，可以被外部文件 `require()`

```js
// app.js
const http = require('http');
const path = require('path');
const conf = require('./config/defaultConfig');
const route = require('./helper/route');

class Server {
  constructor(config) {
    this.conf = Object.assign({}, conf, config);
  }

  start() {
    // 注意 this 指定，回调使用箭头函数
    const server = http.createServer((req, res) => {
      const filePath = path.join(this.conf.root, req.url)
      route(req, res, filePath, this.conf)
    })

    server.listen(this.conf.port, this.conf.hostname, () => {
      const info = `http://${this.conf.hostname}:${this.conf.port}`
      console.log(`The server is listening at ${info}`)
    })
  }
}

module.exports = Server
```

`index.js` 引用 yargs 包读取 命令行中的参并启动应用， yargs 用法详见其官网

```js
// index.js
const yargs = require('yargs')
const Server = require('./app')

const argv = yargs
  .usage('anydoor [options]')
  .options('p', {
    alias: 'port',
    describe: '端口号',
    default: 9527
  })
  .options('h', {
    alias: 'hostname',
    describe: '主机名',
    default: '127.0.0.1'
  })
  .options('d', {
    alias: 'root',
    describe: '根目录',
    default: process.cwd()
  })
  .version()
  .alias('v', 'version')
  .help()
  .argv

const server = new Server(argv)
server.start()
```

### 包装成脚手架并发布

```bash
node src/index.js -p 9999
```

如今已经在命令行中指定参数启动应用，但是要发布到 npm 中以全局模块进行安装，要在 `package.json` 添加

```js
"bin": {
	"anywhere": "bin/anywhere"
}
```

`bin/anywhere` 中指定使用 node 加载启动 `src/index.js`

```js
#!/usr/bin/env node

require('../src/index')
```

`bin/anywhere` 文件还没有执行权限，使用 `chmod +x bin/anywhere` 增加执行权限

现在本地可以使用类似 `bin/anywhere -p 9999` 启动应用

最后 `npm publish` 发布到 npm 中
