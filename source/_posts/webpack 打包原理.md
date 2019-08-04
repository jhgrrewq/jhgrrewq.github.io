---
title: webpack 打包原理
tag: 
	- webpack
---

<!-- markdownlint-disable MD010 -->

## webpack 核心概念

- **entry** 一个 module 或 lib 的入口（可以有多个入口）
- **module** 模块，在 webpack 中一切都是模块，一个模块对应一个文件，webpack 会从入口开始递归找出所有依赖的模块
- **chunk** 代码块，多个 module 组成的一个代码块，如将一个 module 和其所有依赖的组合为一个 chunk
- **loader** 文件转换器，用于将 module 内容根据需求转换为新内容，如将 es6 转为 es5， scss 转为 css
- **plugin** 插件，用于扩展 webpack 功能，在 webpack 构建生命周期的节点上加入扩展 hook 为 webpack 加入功能

## webpack 构建流程

从启动 webpack 构建到输出结果是一个串行过程：

- **解析 webpack 配置参数（初始化参数）：** 从配置文件和 shell 语句中读取并合并参数，得到最终的参数
- **开始编译：** 用上一步得到的参数**初始化 Compile 对象**，**加载注册所有配置的插件**（好让插件监听 webpack 构建生命周期的事件节点，做出对应反应），**执行对象的 run 方法开始执行编译**
- **编译模块：** 从配置的 entry 递归解析所有依赖的 module，每找到一个 module 会调用配置的 loader 对 module 构建 AST 语法树并进行转换，再找出当前 module 依赖的模块递归下去，在解析 module 递归过程中根据文件类型和 loader 配置找到合适的 loader 对 module 进行转换
- **完成编译模块：** 根据 entry 和 module 的依赖关系，组装成一个个包含多个 module 的 chunk（module 会以 entry 分组，一个 entry 和依赖的 module 构成 chunk）， 再把每个chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会
- **输出完成：** 在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统

在以上过程中，webpack 会在特定时间点广播特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 webpack 提供的 api 改变 webpack 的运行结果

## webpack 打包原理

通过 webpack 打包构建一个简单的应用，透过这个简单的例子来看一下 webpack 打包的代码，了解 webpack 打包的原理

<!-- more -->

```javascript
// ./hello.js
module.exports = "hello world"

// ./index.js
var text = require('./hello')
console.log(text)

// ./index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <script src="./bundle.js"></script>
</body>
</html>
```

index.html 中的 bundle.js 并不存在, 通过 webpack ./index.js bundle.js 生成打包文件 bundle.js

```javascript
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache 模块缓存对象
/******/ 	var installedModules = {};

/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {

/******/ 		// Check if module is in cache
/******/ 		// 检查 module 是否在 cache中
/******/ 		if(installedModules[moduleId])
/******/ 			return installedModules[moduleId].exports;

/******/ 		// Create a new module (and put it into the cache)
/******/ 		// 新建一个 module 并放入 cache 中
/******/ 		var module = installedModules[moduleId] = {
/******/ 			exports: {},
/******/ 			id: moduleId,
/******/ 			loaded: false
/******/ 		};

/******/ 		// Execute the module function
/******/ 		// 执行 module 函数
/******/ 		// webpack 为文件包裹了工厂方法 function(module, module.exports, __webpack_require__) {}
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

/******/ 		// Flag the module as loaded
/******/ 		// 标记 module 已经加载
/******/ 		module.loaded = true;

/******/ 		// Return the exports of the module
/******/ 		// 返回 module 的导出模块
/******/ 		return module.exports;
/******/ 	}


/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	// 暴露 modules 对象 (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;

/******/ 	// expose the module cache
/******/ 	// 暴露 modules 缓存
/******/ 	__webpack_require__.c = installedModules;

/******/ 	// __webpack_public_path__
/******/ 	// 设置 webpack 公共路径 __webpack_public_path__
/******/ 	__webpack_require__.p = "";

/******/ 	// Load entry module and return exports
/******/ 	// 读取入口模块 并返回 exports 导出
/******/ 	return __webpack_require__(0);
/******/ })
/************************************************************************/    // webpackBootstrap 传入的参数是一个数组
/******/ ([
/* 0 */  // index.js 的工厂方法
/***/ (function(module, exports, __webpack_require__) {

	var text = __webpack_require__(1)
	console.log(text)

/***/ }),
/* 1 */ // hello.js 的工厂方法
/***/ (function(module, exports) {

	module.exports = "hello world"

/***/ })
/******/ ]);
```

分析上述 webpackBootstrap 立即执行函数 发现:

1) **webpack 自动将每个模块文件包裹在一个工厂函数中，并传入默认参数（requirejs 则需要自己为每个模块写工厂函数define([], callback)）, 然后将它们放入一个 modules 数组中， 并取数组的下标作为 modulesId**

2) **将 modules 传入一个立即执行函数中，立即执行函数中包含一个 installModules 对象用于缓存已经加载的模块和一个模块加载函数， 最后加载入口模块并返回**。

_每个模块 webpack 只加载一次，加载过的模块存在在 installModules 中，重复加载的模块下次加载时直接从 installModules 读取_。

3) **模块加载函数（__webpack_require__）加载模块，首先判断 installedModules 是否已经加载，加载过了就直接返回 module.exports，没有加载过该模块就通过 ```modules[moduleId].call(module.exports, module, module.exports, __webpack__require__)``` 执行模块并将 module.exports 返回**

```modules[moduleId].call(module.exports, module, module.exports, __webpack__require__)``` 保证模块加载时 this 指向 module.exports, 并传入默认参数

模块加载函数 ```__webpack_require__``` 提供的功能和 require 是一样的，声明对其他模块的依赖并获得该模块的 exports。不同之处在于 ```__webpack_require__``` 不需要提供模块的相对路径或者其他形式的 ID, 直接传入该模块在 modules 数组中的下标即可。 数组 modules 的下标和真实模块的实现一一对应，省去了模块标识符的解析过程。
