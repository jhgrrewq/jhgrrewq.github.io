---
title: webpack require 的几种用法
tag: 
	- webpack
---

## commonJS 同步加载

```js
var a = require('../utils/a.js')
console.log(a)
```

commonJS require 引入的模块是同步引入，模块会被打包到引入它的文件中

## commonJS 异步加载

webpack 中的 `require.ensure` 能实现代码切割，并将**依赖的模块全部打包到另外一个文件**，并**异步加载**分块的代码

```js
console.log('start')
require.ensure([], function(require) {
  // 多个依赖的模块的打包到单独的一个文件中
  var a = require('../utils/a.js')
  var b = require('../utils/b.js')
  console.log('require')
})
console.log('end')
// start end require 异步加载
```

## commonJS 预加载懒执行

不同于上面的方法中依赖传入空数组，下面的写法唯一的区别在于一开始模块只是加载并没有执行，真正执行是 `require('../utils/a.js')`

```js
require.ensure(['../utils/a.js'], function(a) {
  console.log(a)
  var a = require('../utils/a.js')
  console.log(a)
})
```

## AMD 异步加载

webpack 同样支持 AMD 写法，同样是将**依赖的模块全部打包到另外一个文件**，并**异步加载**分块的代码，可以认为是 `require.ensure` 的 AMD 写法

```js
console.log('start')
require(['../utils/a.js','../utils/b.js'], function(a, b) {
  // 多个依赖的模块的打包到单独的一个文件中
  console.log('require')
  console.log(a, b)
})
console.log('end')
```