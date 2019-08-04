---
title: webpack 抽取公共模块
tag: 
	- webpack
---

<!-- markdownlint-disable MD010 -->

> 参考: [深入理解 webpack 文件打包机制](https://github.com/happylindz/blog/issues/6)、[webpack官方文档 代码分离部分](https://doc.webpack-china.org/guides/code-splitting/)

第三方引用代码库一般很大，没有引入新模块之前一般是不会变动的，因此通常希望将依赖这部分单独打包，利用浏览器缓存提高加载速度。（如果不单独打包，这些第三方依赖库会被打包进业务代码， 而且引用相同依赖的每个文件会重复打包依赖）

<!-- more -->
## 使用 CommonsChunkPlugin

webpack 内置 CommonsChunkPlugin 插件是**将公共依赖的模块提取到已有的入口 chunk 中或者新生成的 chunk 中**。

```javascript
new webpack.optimize.CommonsChunkPlugin({
	name: 'commons', // 该公共代码的 chunk 名为 "commons"
	filename: '[name].bundle.js', // 生成的文件名，实际是 "commons.bundle.js"
	minChunks: 4  // 设定要有4个chunk的js模块才会被纳入到公共代码中，基本设置3-5个合适
})
```

CommonsChunkPlugin 的初始化常用参数如下:

- **name**: 给这个包含公共代码的 chunk 命名（唯一标识）

- **filename**: 如何命名打包后的js文件，可以加上 [name]、[hash]、[chunkhash] 这些变量，最终生成文件的路径是根据 webpack 配置中的 output.path 和 CommonsChunkPlugin 的 filename 拼接

- **minChunks**: 公共代码判定标准: 某个 js 某块被多少个 chunk加载了才算是公共代码， 一般指定 3-5 个

- **chunks**: 表示需要在哪些 chunk 中寻找公共代码进行打包，不设置该参数则默认是所有的 chunk

## 案例

```toc
|- /dist
|- /node_modules
|- /src
    |- indexA.js
    |- indexB.js
|- package.json
|- util.js
|- webpack.config.js
```

```javascript
// ./util.js  相当于引用的第三方库
export const utilA = 'utilA'
export const utilB = 'utilB'

// ./src/indexA.js
import {utilA, utilB} from '../util'
console.log(utilA)
console.log(utilB)

// ./src/indexB.js
import {utilB} from '../util'
console.log(utilB)

// ./webpack.config.js
const path = require('path')
const glob = require('glob')
const webpack = require('webpack')
const htmlWebpackPlugin = require('html-webpack-plugin')

var getEntry = function(globPath) {
    var entries = {}
    // 使用 glob 模块的 sync 方法批量读取路径
    glob.sync(globPath).forEach(function(entry) {
        basename = path.basename(entry, path.extname(entry))
        entries[basename] = entry
    })

    return entries
}

var entries = getEntry('./src/**/*.js');

module.exports = {
    entry: entries,
    output: {
        path: path.resolve(__dirname, './dist'),
        filename: '[name].[chunkhash].js',
    },
    plugins: [
        new webpack.optimize.CommonsChunkPlugin({
            name: 'vendor',
            filename: '[name].[chunkhash].js',
            minChunks: 2
        })
    ]
}

for (var pathname in entries) {
  // 配置生成的html文件，定义路径等
  var conf = {
    filename: pathname + '.html',
    chunks: [pathname, 'vendor'], // 每个html引用的js模块
    inject: true // js插入位置
  };
  // 需要生成几个html文件，就配置几个htmlWebpackPlugin对象
  module.exports.plugins.push(new htmlWebpackPlugin(conf));
}
```

现在已经能够实现抽取公共代码，但是如果修改 ./src/indexB.js

```javascript
import {utilB} from './util'
console.log(utilB + 'plus')
```

公共代码 vendor 的 hash 也发生改变

![](http://ony85apla.bkt.clouddn.com/18-1-23/46507325.jpg)

然而我们只是修改了业务代码，并没有修改公共代码（依赖库）。大多数情况下开发时只是修改业务代码。

## 优化

> 变化原因： webpack 打包后构建过程中会把 Runtime 的部分 加入 vendor 中

CommonsChunkPlugin **只是提取公共的部分而不是 “经常变动的部分”**。
**需要从公共代码块 vendor 中将这些代码抽离到 manifest 代码块中**。这样可以给 vendor 稳定的 hash 值， 每次修改业务代码，这些初始化的代码会发生变化，如果这些代码放在 vendor 文件中，每次都会生成新的 vendor.xxxx.js，不利于持久化缓存。

```javascript
module.exports = {
    ...
    plugins: [
        ...,
        new webpack.optimize.CommonsChunkPlugin({
            name: "manifest",
            chunks: ['vendor']
        })
    ]
    ...
}

...
for (var pathname in entries) {
  // 配置生成的html文件，定义路径等
  var conf = {
    filename: pathname + '.html',
    chunks: [pathname, 'vendor', 'manifest'], // 每个html引用的js模块, 注意顺序, 数组最后的元素最先加载
    inject: true // js插入位置
  };
  // 需要生成几个html文件，就配置几个htmlWebpackPlugin对象
  module.exports.plugins.push(new htmlWebpackPlugin(conf));
}

```

可以发现修改 ./src/indexB.js文件, vendor 的 hash 没有发生改变， manifest 的 hash 改变了。

![](http://ony85apla.bkt.clouddn.com/18-1-23/78464460.jpg)

## 分析 manifest

```javascript
/******/ (function(modules) { // webpackBootstrap
/******/ 	// install a JSONP callback for chunk loading
/******/ 	var parentJsonpFunction = window["webpackJsonp"];
/******/ 	window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules, executeModules) {
/******/ 		// add "moreModules" to the modules object,
/******/ 		// then flag all "chunkIds" as loaded and fire callback
/******/ 		var moduleId, chunkId, i = 0, resolves = [], result;
/******/ 		for(;i < chunkIds.length; i++) {
/******/ 			chunkId = chunkIds[i];
/******/ 			if(installedChunks[chunkId]) {
/******/ 				resolves.push(installedChunks[chunkId][0]);
/******/ 			}
/******/ 			installedChunks[chunkId] = 0;
/******/ 		}
/******/ 		for(moduleId in moreModules) {
/******/ 			if(Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
/******/ 				modules[moduleId] = moreModules[moduleId];
/******/ 			}
/******/ 		}
/******/ 		if(parentJsonpFunction) parentJsonpFunction(chunkIds, moreModules, executeModules);
/******/ 		while(resolves.length) {
/******/ 			resolves.shift()();
/******/ 		}
/******/ 		if(executeModules) {
/******/ 			for(i=0; i < executeModules.length; i++) {
/******/ 				result = __webpack_require__(__webpack_require__.s = executeModules[i]);
/******/ 			}
/******/ 		}
/******/ 		return result;
/******/ 	};
/******/
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// objects to store loaded and loading chunks
/******/ 	var installedChunks = {
/******/ 		3: 0
/******/ 	};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/ 	// This file contains only the entry chunk.
/******/ 	// The chunk loading function for additional chunks
/******/ 	__webpack_require__.e = function requireEnsure(chunkId) {
/******/ 		var installedChunkData = installedChunks[chunkId];
/******/ 		if(installedChunkData === 0) {
/******/ 			return new Promise(function(resolve) { resolve(); });
/******/ 		}
/******/
/******/ 		// a Promise means "currently loading".
/******/ 		if(installedChunkData) {
/******/ 			return installedChunkData[2];
/******/ 		}
/******/
/******/ 		// setup Promise in chunk cache
/******/ 		var promise = new Promise(function(resolve, reject) {
/******/ 			installedChunkData = installedChunks[chunkId] = [resolve, reject];
/******/ 		});
/******/ 		installedChunkData[2] = promise;
/******/
/******/ 		// start chunk loading
/******/ 		var head = document.getElementsByTagName('head')[0];
/******/ 		var script = document.createElement('script');
/******/ 		script.type = 'text/javascript';
/******/ 		script.charset = 'utf-8';
/******/ 		script.async = true;
/******/ 		script.timeout = 120000;
/******/
/******/ 		if (__webpack_require__.nc) {
/******/ 			script.setAttribute("nonce", __webpack_require__.nc);
/******/ 		}
/******/ 		script.src = __webpack_require__.p + "" + chunkId + "." + {"0":"3af03ac6672cba244ada","1":"be4c3be39b0b1fc2988f","2":"32af6f4d365e76b6864f"}[chunkId] + ".js";
/******/ 		var timeout = setTimeout(onScriptComplete, 120000);
/******/ 		script.onerror = script.onload = onScriptComplete;
/******/ 		function onScriptComplete() {
/******/ 			// avoid mem leaks in IE.
/******/ 			script.onerror = script.onload = null;
/******/ 			clearTimeout(timeout);
/******/ 			var chunk = installedChunks[chunkId];
/******/ 			if(chunk !== 0) {
/******/ 				if(chunk) {
/******/ 					chunk[1](new Error('Loading chunk ' + chunkId + ' failed.'));
/******/ 				}
/******/ 				installedChunks[chunkId] = undefined;
/******/ 			}
/******/ 		};
/******/ 		head.appendChild(script);
/******/
/******/ 		return promise;
/******/ 	};
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, {
/******/ 				configurable: false,
/******/ 				enumerable: true,
/******/ 				get: getter
/******/ 			});
/******/ 		}
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";
/******/
/******/ 	// on error function for async loading
/******/ 	__webpack_require__.oe = function(err) { console.error(err); throw err; };
/******/ })
/************************************************************************/
/******/ ([]);
```