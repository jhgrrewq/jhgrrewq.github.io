---
title: Node.js 知识点梳理
tag: 
	- Node.js 
	- Node.js 知识点梳理
---

<!-- markdownlint-disable MD010 -->
## 概念和原理

> **Node.js 是基于chrome v8 引擎运行在服务器端的运行时（可理解为类似java的jre）**

<!--more-->

### 特点

#### 单线程

java、php或者.net等服务器端语言，会为每个客户端连接创建一个线程，而每个线程约要耗费2M内存。因此为了让web程序支持更多用户，就需要增加服务器的数量，进而可能需要提升服务器的性能（单核变多核），这无疑提升了服务器的硬件成本。

**Node.js只使用单线程，不会为每个客户端连接创建一个新线程。**当有客户端连接，就触发一个内部事件，通过**非阻塞I/O、事件驱动机制**，让Node.js宏观上并行，实现高并发。

单线程的优点：操作系统没有线程创建、销毁的开销；缺点：某个客户端造成线程崩溃，整个服务就崩溃，其他客户端也崩溃。

![](http://ony85apla.bkt.clouddn.com/18-2-10/7115488.jpg)

#### 非阻塞 I/O

I/O比cpu运算要耗费更长的时间。传统的单线程机制中，I/O执行时，整个线程会暂停等待I/O完成返回结果后，才执行下面的代码，也就是说**I/O阻塞了代码的执行**。

**Node.js采用非阻塞I/O, 遇到I/O读写代码时，立即转而执行其后面的代码，将I/O返回的结果放在回调函数中**，提高了代码的执行效率。**当某个I/O执行完毕，将以事件形式通知执行I/O操作的线程，线程执行该I/O的回调。** 为了处理异步I/O, 线程必须有事件循环机制，不断检查是否有未经处理的事件，依次处理。

![](http://ony85apla.bkt.clouddn.com/18-2-10/12888853.jpg)

#### 事件驱动

在Node中，客户端请求建立连接，提交数据等行为，会触发相应事件。**在Node中，在一个时刻，只能处理一个事件回调， 但在处理一个事件回调的过程中，可以转而处理其他事件（比如有新的客户端连接），然后返回继续执行原事件的回调。这是Node的"事件环"机制**。不管是新的客户端的连接请求，还是老的客户端的I/O执行完成，都会通过事件方式加入事件环，等待调度（通过一定的优先级算法调度）。

![](http://ony85apla.bkt.clouddn.com/18-2-10/57943648.jpg)

> 三者缺一不可：
> - 单线程为了防止线程被I/O阻塞，引入非阻塞I/O; 
> - 非阻塞I/O不会等待I/O语句结束，而是先执行下面代码，但是I/O异步执行结束后需要某种机制通知线程，引入事件驱动；
> - 触发的操作以事件形式加入事件环，等待调度

### 适合开发

- I/O密集，而非cpu密集 （cpu密集同样会阻塞线程）
- 和 web socket 配合开发长连接实时交互应用

## Node.js 内置模块

### fs

Node.js fs 模块中的方法均有同步和异步版本，异步方法最后一个函数为回调函数，回调函数的第一个参数包含了错误信息

**关于文件读取，服务器启动如果是要读取配置文件或者结束时写入状态文件，可以使用同步代码，因为这些代码只是在启动或者结束时执行一次，其余时候尽量使用异步代码，因为 js 单线程的特点，同步代码极容器带来阻塞**

#### 文件读取

- **`fs.readFile(path, [type], callback)` 异步读取文件**

**读取二进制文件，不传入文件编码时，回调拿到的数据是一个 Buffer 对象**

```js
const fs = require('fs')
const result = fs.readFile('./example.txt', function(err, data){
  if(err) {
		console.log(err)
	}else{
		// console.log(data) // 输出二进制buffer
		// Buffer 对象转化 String 调用 toString() 方法
		console.log(data.toString())
	}
})

const result2 = fs.readFile('./example.txt', 'utf-8', function(err, data){
  if(err) {
		console.log(err)
	}else{
		console.log(data)
	}
})
```

- **`fs.readFileSync(path, [type])` 同步读取文件（没有回调）**

```js
const fs = require('fs')
try{
	let data = fs.readFileSync('./example.txt', 'utf-8')
} catch(err) {
	console.log(err)
}
```

#### 文件写入

- **`fs.writeFile(path, data, [type], callback)` 异步写文件**

**如果传入数据是 String 类型，默认 utf-8 编码写入文件；如果传入 Buffer，则写入二进制文件；回调函数只关系成功与否，只需要一个 err 参数**

```js
// 如果文件不存在，则创建文件，否则覆盖其内容
const fs = require('fs')
var data = 'abc'
fs.writeFile('./example.txt', data, (err) => {
	if (err) {
		console.log(err)
	} else {
		console.log('ok')
	}
})
```

- **`fs.writeFileSync(path, data, [type])` 同步写文件**

```js
const fs = require('fs')
try {
	fs.writeFileSync('./example.txt', 'hello', 'utf-8')
	console.log('ok')
} catch(err) {
	console.log(err)
}
```

#### 文件流读取文件

流是一种抽象的数据结构，可以将数据看成数据流。标准输入流：字符从键盘输入到应用程序；标准输出流：字符从应用程序输出到显示器

从文件中读取数据时，可以打开一个文件流，从文件流中读取；向文件中写数据时，只需要向文件流中不断写入数据

**一般文件操作都是将文件整个放入内存，流相当于一点一点读写**

Node.js 中流是一个对象，只需要关注响应流的事件即可：**`data` 事件表示流已经可以读取（`data` 事件可能有多次，每次的 `chunk` 是流的一部分数据），`end` 事件表示该流已经到末尾，已经没有数据可以读取，`error` 事件表示出错, `close` 事件表示不会再有事件抛出**

- **`fs.createReadStream(path, [type])` 创建读流，从某个文件以某种编码读取**

```js
const fs = require('fs')
const rs = fs.createReadStream('./example.txt', 'utf-8')
// 等同于 const rs = fs.createReadStream('./example.txt'); rs.setEncoding('UTF8')
// rs.pipe(process.stdout) // process.stdout 标准输出

// 处理事件 data error end close
rs.on('data', (chunk) => {
	console.log('读取数据：' + chunk) // data 事件表示流可以读取，可能会有多次，每次 chunk 是流的一部分数据
}).on('err', (err) => {
	console.log('错误：' + error)
}).on('end', () => {
	console.log('没有数据') // end 事件表示已经到流末尾，没有数据
}).on('close', () => {
	console.log('已经关闭') // close 事件表示已经关闭，没有事件抛出
})
```

#### 文件流写入文件

- **`fs.createWriteStream(path, [type])` 创建写流，不断调用 write() 方法，最后 end() 结束**
- **`stream.write(data)` data 为 String 类型 或 Buffer**
- **`stream.end()` 关闭流，会触发 finish 事件**

```js
const fs = require('fs')
let data = 'abc'
const ws = fs.createWriteStream('./example.txt')
ws.write(data, 'utf-8') // data 可为 String 类型或 Buffer
ws.end() // 关闭流会触发 finish 事件
ws.on('finish', () => {
	console.log('写入完成')
}).on('error', (err) => {
	console.log(err)
})
```

#### 删除文件

- **`fs.unlink(path, cb)` 异步删除文件**

```js
const fs = require('fs')
fs.unlink('./example.txt', (err) => {
	if (err) {
		console.log(err)
	} else {
		console.log('删除文件成功')
	}
})
```

- **`fs.unlinkSync(path)` 同步删除文件**

```js
const fs = require('fs')
try{
	fs.unlinkSync('./example.txt')
} catch(err) {
	console.log(err)
}
```

#### 文件是否存在

- **`fs.access(path, cb)` 除了判断文件是否存在，还判断文件权限**

```js
const fs = require('fs')
fs.access('./example.txt', (err) => {
	if (err) {
		console.log(err)
	} else {
		console.log('example.txt 存在')
	}
})
```

#### 文件重命名

- **`fs.rename(oldPath, newPath, cb)` 异步文件重命名**

```js
const fs = require('fs')
fs.rename('./example.txt', './example2.txt', err => {
	if(err) return console.log(err)
	console.log('重命名成功')
})
```

- **`fs.renameSync(oldPath, newPath)` 同步文件重命名**

```js
const fs = require('fs')
try{
	fs.renameSync('./example.txt', './example2.txt')
	console.log('重命名成功')
} catch(err) {
	console.log(err)
}
```

#### 创建目录

- **`fs.mkdir(path, [mode], cb)` 异步创建目录，如果目录已经存在会报错, mode 设置目录权限**

```js
const fs = require('fs')
fs.mkdir('./hello', (err) => {
	if (err) {
		console.log(err)
	} else {
		console.log('目录创建成功')
	}
})
```

- **`fs.mkdirSync(path)` 同步创建目录**

```js
const fs = require('fs')
try{
	fs.mkdirSync('./hello')
	console.log('目录创建成功')
} catch(err) {
	console.log(err)
}
```

#### 读取目录

- **`fs.readdir(path, [options], cb)` 异步读取目录**

回调 `cb` 有两个参数 `err` `files`， `files` 为目录下的文件数组列表

```js
const fs = require('fs')
fs.readdir('./', (err, files) => {
	if (err) {
		return console.log(err)
	}
	files.forEach(file => {
		console.log(file)
	})
})
```

- **`fs.readdirSync(path, [options])` 同步读取目录**

```js
const fs = require('fs')
try{
	const files = fs.readdirSync('./')
	files.forEach(file => {
		console.log(file)
	})
} catch(err) {
	console.log(err)
}
```

#### 监听文件修改

- **`fs.watchFile(path, [options], [cb])`**

**实现原理：轮询。每隔一段时间检查文件是否变化。在不同平台上的表现基本一致**

```js
const fs = require('fs')
fs.watchFile('./example.txt', {
	interval: 2000 // 多久检查一次
}, (curr, prev) => {
	// curr, prev 是被监听文件的状态
	console.log('修改时间为：' + curr.mtime)
})
```

- **`fs.watch(path, [options], [cb])`**

#### 修改文件权限

- **`fs.chmod(path, mode, cb)` 异步修改文件权限**

```js
const fs = require('fs')
fs.chmod('./example.txt', '777', (err) => {
	if(err) return console.log(err)
	console.log('权限修改成功')
})
```

- **`fs.chmodSync(path, mode)` 同步修改文件权限**

```js
const fs = require('fs')
try{
	fs.chmodSync('./example.txt', '777')
	console.log('权限修改成功')
} catch(err) {
	console.log(err)
}
```

#### 获取文件状态

- **`fs.stat(path, cb)` 异步获取文件状态**

回调 `cb` 接受 2 个参数 `err` `stats`：`stats` 是一个 `Class: fs.Stats` 的对象实例，有很多方法和属性

```js
{
  dev: 2114,
  ino: 48064969,
  mode: 33188,
  nlink: 1,
  uid: 85,
  gid: 100,
  rdev: 0,
  size: 527,
  blksize: 4096,
  blocks: 8,
  atime: Mon, 10 Oct 2011 23:24:11 GMT, // 访问时间
  mtime: Mon, 10 Oct 2011 23:24:11 GMT,  // 文件内容修改时间
  ctime: Mon, 10 Oct 2011 23:24:11 GMT,  // 文件状态修改时间
  birthtime: Mon, 10 Oct 2011 23:24:11 GMT  // 创建时间
}
```

如

- **`stats.isFile()`** 是否是文件
- **`stats.isDirectory()`** 是否是目录
- **`stats.atime`** 访问时间（access time）
- **`stats.mtime`** 文件内容修改时间（modified time）
- **`stats.ctime`** 文件状态修改时间，比如修改文件所有者、修改权限、重命名等
- **`stats.birthtime`** 创建时间，但是在某些系统上拿不到

```js
const fs = require('fs')
try{
	const files = fs.readdirSync('./')
	files.forEach(file => {
		fs.stat(file, (err, stats) => {
			console.log('文件名：', file)
			console.log('文件大小：', stats.size)
			console.log('创建时间：', stats.birthtime)
			console.log('是否是目录：', stats.isDirectory())
			console.log('是否是文件：', stats.isFile())
		})
	})
} catch(err) {
	console.log(err)
}
```

- **`fs.stat(path)` 同步获取文件状态**

```js
const fs = require('fs')
try{
	const files = fs.readdirSync('./')
	files.forEach(file => {
		const stats = fs.statSync(file)
		console.log('文件名：', file)
		console.log('文件大小：', stats.size)
		console.log('创建时间：', stats.birthtime)
		console.log('是否是目录：', stats.isDirectory())
		console.log('是否是文件：', stats.isFile())
	})
} catch(err) {
	console.log(err)
}
```

### url 模块

![](http://ony85apla.bkt.clouddn.com/18-6-23/96846634.jpg)

#### url 解析

- **`url.parse(urlString, [parseQueryString], [slashesDenoteHos])` 将 url 字符串解析成 object**

参数 `parseQueryString`（默认为 false）：如为 false，则 query 为未解析的字符串，对应值不会 decode；否则 query 为 object，并且其值会被 decode

```js
const { parse } = require('url')
const str = 'http://Chyingp:HelloWorld@ke.qq.com:8080/index.html?nick=%E7%A8%8B%E5%BA%8F%E7%8C%BF%E5%B0%8F%E5%8D%A1#part=1'

console.log(parse(str)) // query 不解析
console.log(parse(str, true)) // query 会被解析

/*
Url {
  protocol: 'http:',
  slashes: true,
  auth: 'Chyingp:HelloWorld',
  host: 'ke.qq.com:8080',
  port: '8080',
  hostname: 'ke.qq.com',
  hash: '#part=1',
  search: '?nick=%E7%A8%8B%E5%BA%8F%E7%8C%BF%E5%B0%8F%E5%8D%A1',
  query: 'nick=%E7%A8%8B%E5%BA%8F%E7%8C%BF%E5%B0%8F%E5%8D%A1',
  pathname: '/index.html',
  path: '/index.html?nick=%E7%A8%8B%E5%BA%8F%E7%8C%BF%E5%B0%8F%E5%8D%A1',
  href: 'http://Chyingp:HelloWorld@ke.qq.com:8080/index.html?nick=%E7%A8%8B%E5%BA%8F%E7%8C%BF%E5%B0%8F%E5%8D%A1#part=1' }
*/
/*
Url {
  protocol: 'http:',
  slashes: true,
  auth: 'Chyingp:HelloWorld',
  host: 'ke.qq.com:8080',
  port: '8080',
  hostname: 'ke.qq.com',
  hash: '#part=1',
  search: '?nick=%E7%A8%8B%E5%BA%8F%E7%8C%BF%E5%B0%8F%E5%8D%A1',
  query: { nick: '程序猿小卡' },
  pathname: '/index.html',
  path: '/index.html?nick=%E7%A8%8B%E5%BA%8F%E7%8C%BF%E5%B0%8F%E5%8D%A1',
  href: 'http://Chyingp:HelloWorld@ke.qq.com:8080/index.html?nick=%E7%A8%8B%E5%BA%8F%E7%8C%BF%E5%B0%8F%E5%8D%A1#part=1' }
*/
```

#### urlObject

- `protocol` 协议，包含了 ：，并且是小写
- `slashes` 如果 ：后面跟着 //，则为 true
- `auth` 认证信息，如果有密码，为 username:possword, 如果没有则为 username。区分大小写
- `host` 主机，包含了端口号，小写
- `hostname` 主机名，不含端口号，小写
- `port` 端口号
- `path` 路径部分，包含 search 部分
- `pathname` 路径部分，不包含 search 部分
- `search` 查询字符串，包含 ？，并且值没有 decode
- `query` 字符串或者对象，如果是字符串则是 search 去掉 ？；如果是对象则是 decode 过的
- `hash` 哈希部分，包含 #
- `href` 原始地址

### events 模块

**所有能触发事件的对象都是 EventEmitter 实例（需要继承 EventEmitter）**

```js
const EventEmitter = require('events')
class CustomEvent extends EventEmitter {}
```

- **`eventEmitter.on(event, fn)` 用于注册监听器，同一个事件可注册多个，当事件触发，事件监听器按照注册的顺序执行**
- **`eventEmitter.emit(event, arg1, ..., argN)` 用于触发事件**
- **`eventEmitter.once(event, fn)` 注册监听器，响应函数只执行一次**
- **`eventEmitter.removeListener(event, fn)` 移除某个事件的某个监听器**
- **`eventEmitter.removeAllListener(event)` 移除某个事件的全部监听器**

```js
const EventEmitter = require('events')
class CustomEvent extends EventEmitter {}
const ce = new CustomEvent()

ce.on('error', (err, time) => {
	console.log(err, time)
})

ce.emit('error', new Error('wrong'), Date.now())

ce.once('test', () => {
	console.log('test')
})

setInterval(() => {
	ce.emit('test')
}, 1000)

function f1() {
	console.log('f1')
}

function f2() {
	console.log('f2')
}

ce.on('remove', f1)
ce.on('remove', f2)

setInterval(() => {
	ce.emit('remove')
}, 2000)

setTimeout(() => {
	ce.removeListener('remove', f2)
}, 1000)
```

### path 模块

path 模块会根据 Node.js 程序运行的操作系统使用对应操作系统风格的路径。想要任何操作系统处理 window 文件路径获得一致结果，使用`path.win32`; 想要任何操作系统处理 POSIX 文件路径获得一致结果,使用 `path.posix`; 如果 `path.win32.basename()`

#### 获取路径/文件名/扩展名

- **`path.dirname(filePath)` 获取所在文件夹路径**

```js
const { dirname } = require('path')
const filePath = '/usr/local/bin/no.txt'
console.log(dirname(filePath)) // /usr/local/bin
```

- **`path.basename(filename)` 获取文件名，严格意义上只是获取路径的最后一部分，并不会判断是否是文件名**

```js
const { basename } = require('path')
const filePath1 = '/usr/local/bin/no.txt'
const filePath2 = '/usr/local/bin'
// 返回路径的最后一部分
console.log(basename(filePath1)) // no.txt
console.log(basename(filePath2)) // bin
```

- **`path.extname(filePath)` 获取扩展名，严格意义上只是从最后一个 `.` 开始截取到最后一个字符；没有 `.` 或者第一个字符是 `.`,直接返回空字符**

```js
const { extname } = require('path')
const filePath = '/usr/local/bin/no.txt'
const filePath2 = '/usr/local/bin'
const filePath3 = '/usr/local/bin/no.min.txt'
const filePath4 = '/usr/local/bin.'
const filePath5 = './usr/local/bin'
console.log(extname(filePath)) // .txt
console.log(extname(filePath2)) //
console.log(extname(filePath3)) // .txt
console.log(extname(filePath4)) // .
console.log(extname(filePath5)) //
```

#### 格式化路径字符串

- **`path.normalize(filePath)`**

```js
const { normalize } = require('path')
console.log(normalize('/usr//local/node')) // /usr/local/node
console.log(normalize('/usr//local/../node')) // /usr/node
```

#### 路径组合

- **`path.join([...paths])` 将 paths 拼接后再调用 normalize**

```js
const { join } = require('path')
console.log(join('/usr', 'local', '/node/')) // /usr/local/node/
console.log(join('/usr', '../local', '/node/')) // /local/node/
```

- **`path.resolve([...paths])` 将相对路径解析为绝对路径**

相当于不断调用系统的 `cd` 命令

```js
const { resolve } = require('path') // /Users/jackzhang/workspace/code/Node/nodejs/Node
console.log(resolve('./'))
```

#### 路径解析

- **`path.parse(path)` 将文件名解析包含 root，dir，ext，name, base 属性的对象**

```js
const { parse } = require('path')
const filePath = '/usr/local/bin/no.txt'
console.log(parse(filePath))
/*
{ root: '/',
  dir: '/usr/local/bin',
  base: 'no.txt',
  ext: '.txt',
  name: 'no' }
*/
```

- **`path.format(obj)` 和 parse 相反，一般用于修改文件名的一些属性生成路径**

优先级 `dir` 比 `root` 高， `base` 比 `name` 和 `ext` 高

```js
const { format } = require('path')
let data1 = {
	root: '/',
  dir: '/usr/local/bin',
  ext: '.txt',
  name: 'no'
}
console.log(format(data1))  // /usr/local/bin/no.txt

let data2 = {
	root: '/',
  dir: '/usr/local/bin',
  base: 'no1.txt',
  ext: '.txt',
  name: 'no' }
console.log(format(data2)) // /usr/local/bin/no1.txt
```

#### 获取相对路径

- **`path.relative(from, to)` 返回从 from 进入 to 的相对路径**

如果 `from` `to` 指向同一个路径，返回空字符串；如果两者任一为空，返回当前工作路径

```js
const { relative } = require('path')
relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb') // ../../impl/bbb
console.log(relative('/data/demo', '/data/demo')) // ''
```

#### 平台相关

- **`path.posix` 平台 linux 下实现的 path 相关属性方法**
- **`path.win32` 平台 win32 下实现的 path 相关属性方法**
- **`path.sep` 路径分割符，linux 上是 `/` windows 上是 `\`**
- **`path.delimiter`  path 设置的分割符，linux 上是 `:` windows 上是 `;`**

```js
console.log('sep:', sep)
console.log('win sep:', win32.sep)
console.log('posix sep:', posix.sep)
console.log('delimiter', delimiter)
console.log('win delimiter', win32.delimiter)
console.log('posix delimiter', posix.delimiter)
cnosole.log(win32.join('/tmp', 'demo'))
// sep: /
// win sep: \
// posix sep: /
// delimiter :
// win delimiter ;
// posix delimiter :
// \\tmp\\demo
```

#### util 模块

`util` 模块提供了常用的函数合集

- **`util.inherits(constructor, superConstructor)` 实现对象的原型继承**

```js
const EventEmitter = require('events')
class CustomEvent extends EventEmitter {}

// 等同于
const util = require('util')
const EventEmitter = require('events')
class CustomEvent {}
util.inherits(CustomEvent, EventEmitter)
```

- **`util.promisify(fn)` 将方法 promisify**

promise 异步变同步书写，除了用一些第三方库 如 bluebird，还可以用 `util.promisify`

```js
const { readFile } = require('fs')
const { promisify } = require('util')
const read = promisify(readFile)
async function test() {
	try{
		let data = await read('./example.txt')
		console.log(data)
	} catch(err) {
		console.log(err)
	}
}
test()
```

### Buffer 模块

Buffer 模块用来处理二级制数据。Buffer 实例类似整数数组（0-255 的 16 进制数），**大小是创建时候确定的，无法调整**

#### Buffer 静态方法

- **`Buffer.alloc()` `Buffer.from()` `Buffer.allocUnsafe()` 创建方法**

```js
const Buffer = require('Buffer')
// 创建一个长度为 5，用 0 填充的 Buffer
console.log(Buffer.allo(5)) // <Buffer 00 00 00 00 00>

// 创建一个长度为 5，用 0x1 填充的 Buffer
console.log(Buffer.allo(5, 1)) // <Buffer 01 01 01 01 01>

// 创建一个长度为 5，但没有初始化的 Buffer 实例
// 该方法比 Buffer.allo() 调用快，但是可能会包含有旧数据，需要调用 fill() 或者 write() 方法 重写
console.log(Buffer.alloUnsafe(5)) // <Buffer ef 2e 3b 0c 00>

// Buffer.from(array) // 返回一个当前数组副本的 Buffer
console.log(Buffer.from([1, 2, 3])) // <Buffer 01 02 03>
// Buffer.from(string, [encoding]) // 返回一个字符串的 Buffer，第二个参数可以指定编码
console.log(Buffer.from('test')) // <Buffer 74 65 73 74>
console.log(Buffer.from('test','base64')) // <Buffer b5 eb 2d>
```

- **`Buffer.byteLength()` 字符串实际占了几个字节**

```js
const Buffer = require('Buffer')
console.log(Buffer.byteLength('test')) // 4
console.log(Buffer.byteLength('测试')) // 6
```

- **`Buffer.isBuffer()` 该对象是否是 Buffer 实例**

```js
const Buffer = require('Buffer')
console.log(Buffer.isBuffer({})) // false
console.log(Buffer.isBuffer(Buffer.from([1]))) // true
```

- **`Buffer.concat()` 拼接 Buffer**

```js
const Buffer = require('Buffer')
const b0 = Buffer.from('this')
const b1 = Buffer.from(' is')
console.log(Buffer.concat([b0, b1]).toString()) // this is
```

#### Buffer 实例方法

- **`buf.length` Buffer 本身占得字节数，和申请创建的空间有关，和 Buffer 内容无关**
- **`buf.toString([, encoding])` 将二进制转化为指定编码字符串，默认转 utf8**
- **`buf.fill(value, start, end)` 填充，默认是用 0 填充，value 是 Number 或 String, start 为开始下标，end 为结束下标（不包括）**
- **`buf.equals()` 比较两个 bufer 内容是否相同**
- **`buf.indexOf()` 类似数组同名用法**

```js
const buf = Buffer.from('this is test')
console.log(buf.length) // 12

const buf2 = Buffer.alloc(10)
buf2[0] = 2
console.log(buf2.length) // 10

console.log(buf.toString('base64')) // dGhpcyBpcyB0ZXN0
console.log(buf.toString()) // this is text

const buf3 = Buffer.allocUnsafe(10)
console.log(buf3) // <Buffer ff ff ff ff ff ff ff ff 01 08>
console.log(buf3.fill(10, 2, 6)) // <Buffer ff ff 0a 0a 0a 0a ff ff 01 08>

const buf4 = Buffer.from('test')
const buf5 = Buffer.from('test')
const buf6 = Buffer.from('tes')
console.log(buf4.equals(buf5)) // true
console.log(buf4.equals(buf6)) // false

console.log(buf4.indexOf('es')) // 1
console.log(buf4.indexOf('esa')) // -1
```

### zlib 模块

浏览器通过 http 请求头部添加 `Accept-Encoding: gzip, deflate` 告诉服务器可以使用 gzip 或 deflate 算法压缩资源，Node.js 中使用 zlib 模块对资源进行压缩

#### 压缩解压

- **zlib.createGzip() 压缩**
- **zlib.createGunzip() 解压**

```js
const http = require('http')
const fs = require('fs')
const zlib = require('zlib')
const filePath = './example.txt'

const server = http.createServer((req, res) => {
  const acceptEncoding = req.headers['accept-encoding']

  if (acceptEncoding.indexOf('gzip') > -1) { // 判断是否需要 gzip 压缩
    // 响应头部 Content-Encoding，告诉浏览器文件被 gzip 压缩过
    res.writeHeader(200, {
      'Content-Encoding': 'gzip'
    })
    fs.createReadStream(filePath).pipe(zlib.createGzip()).pipe(res)
  } else {
    fs.createReadStream(filePath).pipe(res)
  }
}).listen(5000)
```

### string_decoder 模块

string_decoder 模块用于将 Buffer 转成对应的字符串。特殊之处在于，**当传入的 buffer 不完整（如三个字节的字符，只传入了两个），内部会维护一个 internal buffer 将不完整的字节 cache，等到使用者再次调用 `stringDecoder.write(buffer)` 传入剩余字节时，来拼接成完整的字符**

#### 基本用法

- **`decoder.write(buffer)` 调用传入 Buffer 对象，返回对应编码字符串**

```js
const { StringDecoder } = require('string_decoder')
const decoder = new StringDecoder('utf8')

// Buffer.from('你') =》 <Buffer e4 bd a0>
console.log(decoder.write(Buffer.from([0xe4, 0xbd, 0xa0]))) // 你
```

- **`decoder.end([buffer])` 调用时会把内部剩余的 buffer 一次性返回, 相当于 decoder.write(buffer) decoder.end()**

```js
const StringDecoder = require('string_decoder').StringDecoder;
const decoder = new StringDecoder('utf8');

// Buffer.from('你好') => <Buffer e4 bd a0 e5 a5 bd>
let str = decoder.write(Buffer.from([0xe4, 0xbd, 0xa0, 0xe5, 0xa5])); // 好 还差一个字节，因此 decoder.write(xx) 返回 你
console.log(str);  // 你

str = decoder.end(Buffer.from([0xbd]));
console.log(str);  // 好
```

### process 模块

process 模块是 Node.js 的全局模块，可用来获得 node 进程相关的信息

#### 环境变量

- **`process.env` 环境变量**

```js
if (process.env.NODE_ENV === 'development') {
	console.log('开发环境')
} else if (process.env.NODE_ENV === 'development') {
	console.log('生产环境')
}
```

运行命令 `NODE_ENV=development node env.js` 输出 `开发环境`

#### 异步

- **`process.nextTick(fn)` 实现异步，fn 放在 node 事件循环的下一个 tick，比 setTimeout(fn, 0) 性能高**

```js
console.log('start')
setTimeout(() => {
	console.log('timeout')
}, 0)
process.nextTick(() => {
	console.log('tick')
})
console.log('end')
// start
// end
// tick
// timeout
```

#### 获取命令行参数

- **`process.argv` 获取命令行文件名后的参数，返回一个数组，第一个元素是 node 程序安装路径，第二个元素是可执行文件的绝对路径，第三个参数开始是命令行文件名后传入的参数**

一般使用从 `process.argv[2]` 开始

```js
NODE_ENV=dev node 10_argv.js --env production

// argv[0] /usr/local/bin/node
// argv[1] /Users/jackzhang/workspace/code/Node/nodejs/Node/10_argv.js
// argv[2] --env
// argv[3] production
```

- **`process.execArgv` 获得命令行文件名前传入的参数**

`process.execPath` node 程序安装路径，也就是 `process.argv[0]`

```js
NODE_ENV=dev node 10_argv.js --env production

// execArgv []
// execPath /usr/local/bin/node
```

#### 当前工作路径

- **`process.cwd()` 返回 node 命令所在文件夹**

```js
// _dirname _filename 总是返回文件所在文件夹、文件的绝对路径（不管 node 在哪里执行）
// process.cwd() 返回执行 node 命令所在文件夹
// ./ 在 require 方法中都是相对于当前文件所在文件夹，在其他地方相当于 process.cwd(), node 命令执行文件夹

// nodejs/Node/example.js
// 目前在 nodejs 目录，执行 node Node/example.js
// __dirname     /Users/jackzhang/workspace/code/Node/nodejs/Node
// process.cwd() /Users/jackzhang/workspace/code/Node/nodejs
// ./            /Users/jackzhang/workspace/code/Node/nodejs
```

### child_process 模块

严格意义上说 Node.js 并不是真正的单线程架构，自身还有一定的 I/O 线程存在，这些线程由底层 libuv 处理，对于 js 开发者是透明的，只有 C++ 扩展开发时才会关注到。 js 代码永远运行在 V8 引擎上，是单线程的。

对于单进程单线程对多核 CPU 使用不足的问题，需要启动多进程。理想状态下每个进程各自利用一个 CPU，以此实现多核 CPU 的利用

### 异步创建子进程（每个方法都有对应同步版本）

- **`spawn()` 启动一个子进程来执行命令**
- **`exec()` 启动一个子进程来执行命令, 和 spawn() 不同的是，它有一个回调函数通知子进程的状况**
- **`execFile()` 启动一个子进程来执行可执行文件**
- **`fork()` 和 spawn() 类似，不同点在创建的子进程只需要指定要执行的 js 文件模块**

`spawn()` 和 `exec()` `execFile()` 不同的是后两者创建时可以指定 timeout 超时时间，一旦创建的进程运行事件超过设定事件会被杀死

**`exec()` `execFile()` `fork()` 底层都是通过 `spawn()` 实现**

| 类型 | 回调/异常 | 进程类型 | 执行类型 | 可设置超时 |
| -- | -- | -- | -- | -- |
| spawn | 无 | 任意 | 命令 | 否 |
| fork | 无 | Node | js 文件 | 否 |
| exec | 有 | 任意 | 命令 | 是 |
| execFile | 有 | 任意 | 可执行文件 | 是 |

### net 模块（TCP）

http 模块中， http.Server 继承了 net.Server, http 客户端和 http 服务器的通信依赖于 socket（net.Socket）

net 模块主要包含两部分：

- `net.Server` tcp server，内部通过 socket 来实现和客户端的通信
- `net.Socket` tcp/本地 socket 的 node 版实现，它实现**全双工**的 stream 接口

```js
// 服务端代码
const net = require('net')
// 创建一个 tcp 服务器
var server = net.createServer(socket => {
	// 回调中拿到新的连接
	console.log('服务器：收到来自客户端的请求')

	socket.on('data', (data) => {
		console.log('服务端：收到客户端的数据-' + data)
		// 给客户端返回数据
		socket.write('你好，我是服务端')
	})

	socket.on('close', () => {
		console.log('服务端：客户端断开连接')
	})
}).listen(8000, () => {
	console.log('服务端：开始监听来自客户端的请求')
})
```

```js
// 客户端代码
const net = require('net')
const PORT = 8000
const HOST = '127.0.0.1'

// 创建 tcp socket 套接字
// net.createConnection 客户端创建一个新的 net.Socket
var client = net.createConnection(PORT, HOST)

client.on('connect', () => {
	console.log('客户端：已经和服务端建立连接')
})

client.on('data', (data) => {
	console.log('客户端：收到服务端数据-' + data)
})

client.on('close', () => {
	console.log('客户端：连接断开')
})

client.on('error', () => {
	console.log('客户端：客户端错误')
})

client.end('你好，我是客户端') // socket.write() socket.end() 套接字读写
```

#### tcp 服务器事件

通过 `net.createServer()` 创建的服务器是一个 `EventEmitter` 实例，有如下自定义属性

- `listening 事件` 一般通过调用 server.listen() 触发
- `connection 事件` 每个客户端套接字连接到服务端时触发，简洁写法是通过 net.createServer() 最后一个参数传递
- `close 事件` 当服务器关闭触发，在调用 server.close() 后，服务器将停止接受新的套接字连接，但是保持当前存在的连接，**等待所有连接都断开，会触发该事件**
- `error 事件` 当服务器发生异常将会触发该事件

#### 连接事件

服务器可以和多个客户端保持连接，每个连接（套接字 socket）都是可读可写的 stream 对象。**stream 对象可以用于服务器和客户端之间的通信，既可以通过 `data` 事件从一端读取另一端发来的数据，也可以通过 `write()` 方法从一端向另一端发送数据**

- `data 事件` 当一端调用 write() 发送数据，另一端会触发该事件，事件传递的数据就是 write() 发送的数据
- `connect 事件` 该事件用于客户端，当套接字和服务端连接成功触发
- `close 事件` 当套接字完全关闭会触发（当只要一个套接字连接，调用 socket.end() 也会触发）
- `error 事件` 当异常发生会触发
- `timeout 事件` 当一段时间后连接不再活跃，该事件会被触发，通知用户当前该连接已经被闲置

### dgram 模块（UDP）

dgram 模块是对 UDP socket 的一层封装

### http 模块

HTTP 全称是超文本传输协议（Hyper Text Transfer Protocol），构建与 TCP 之上，属于应用层协议

```js
const http = require('http')
const url = require('url')
const querystring = require('querystring')
http.createServer((req, res) => {
	// 获取 get 请求参数
	var urlObj = url.parse(req.url)
	if (urlObj.query) {
		var queryObj = querystring.parse(urlObj.query)
		console.log(JSON.stringify(queryObj))
	}

	// 获取 post 请求参数
	// var body = ''
	// req.on('data', thunk => {
	// 	body += thunk
	// })

	res.writeHeader(200, {
		'Content-Type': 'text/plain',
	})
	res.end('hello')
}).listen(1337)
```

启动上述服务器端代码后，使用 curl 进行一次报文获取，-v 选项可以显示这次网络通信的所有报文信息

```bash
$ curl -v http://127.0.0.1:1337

# tcp 三次握手
* Rebuilt URL to: http://127.0.0.1:1337/
*   Trying 127.0.0.1...
* Connected to 127.0.0.1 (127.0.0.1) port 1337 (#0)

# 客户端向服务器端发送请求报文
> GET / HTTP/1.1
> Host: 127.0.0.1:1337
> User-Agent: curl/7.49.1
> Accept: */*
>

# 服务器完成处理向客户端返回响应报文
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Date: Mon, 25 Jun 2018 04:26:49 GMT
< Connection: keep-alive
< Transfer-Encoding: chunked
<

# 结束会话
* Connection #0 to host 127.0.0.1 left intact
```

#### http 请求

```bash
> GET / HTTP/1.1
> Host: 127.0.0.1:1337
> User-Agent: curl/7.49.1
> Accept: */*
>
```

报文第一个 `GET / HTTP/1.1` 被解析后分为如下属性：

- `req.method` 请求方法，这里值为 GET，常见的有 GET POST DELETE PUT CONNECT 等
- `req.url` 请求路径，值为 /
- `req.httpVersion` http 协议版本，值为 1.1

其余报头是规律的 key:value 形式，被解析后放在 `req.headers` 中

```js
headers:
{ host: '127.0.0.1:1337',
  'user-agent': 'curl/7.49.1',
  accept: '*/*' }
```

#### http 响应

```bash
# 服务器完成处理向客户端返回响应报文
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Date: Mon, 25 Jun 2018 04:26:49 GMT
< Connection: keep-alive
< Transfer-Encoding: chunked
<
```

同理报文头第一个被客户端解析为 `res.httpVersion` `res.statusCode` `res.statusMessage`, 其余报头会变为解析放在 `res.headers`

- **`res.statusCode` 获取或设置状态代码**
- **`res.statusMessage` 获取或设置状态描述信息**

```js
res.statusCode = 200
res.statusMessage = 'ok'

// res.writeHeader 还可以设置响应头部，并且当响应头部发送出去后，res.statusCode/res.statusMessage 会被设置为已经发送出去的 状态代码/状态描述信息
res.writeHeader(200, 'ok')
```

- **`res.setHeader` 可以多次设置响应报头**
- **`res.writeHeader` 只有调用该方法，报头才会写入到连接中**

```js
res.setHeader('Content-Type', 'text/plain')

// res.writeHeader 不但是设置头部，并且当 res.setHeader 设置了 header，res.writeHeader 设置同名 header 会覆盖之前的设置；并且 res.writeHeader 被调用后报头才会写入到连接中
res.writeHeader(200, 'ok', {
	'Content-Type', 'text/html'
})
```

- **`res.removeHeader(key)` 删除 某 header**
- **`res.getHeader(key)` 获取 某 header，这里 key 是小写**

报文体部分

- **`res.write()` 发送数据**
- **`res.end()` 先调用 write() 发送数据，再发送信号告知服务器这次响应结束**

**报头是在报文体发送前发送的**，一旦开始数据的发送， `setHeader` `writeHeader` 将不再生效

#### http 客户端

- **`http.request(options)` 构造 http 客户端**

```js
const http =
```