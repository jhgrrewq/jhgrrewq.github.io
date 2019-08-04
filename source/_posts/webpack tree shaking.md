---
title: webpack tree shaking
tag: 
	- webpack
---

> 参考: [webpack 官方文档 tree shaking 部分](https://doc.webpack-china.org/guides/tree-shaking/)

tree shaking 通常是用于描述移除 js 上下文中未引用的代码。

- **它依赖于 es6 模块系统中的静态结构特性，如 import 和 export**。es6 模块的依赖关系是确定的，和运行时状态无关，可以进行静态分析，清除没用的代码。而 commonJS 是动态加载，依赖关系需要运行时才能确定。

- 其次**需要开启 UglifyJsPlugin 插件对代码进行压缩**（也可以使用其他压缩插件）即 webpack 本身不会执行 tree shaking , 需要第三方工具来执行未引用代码的删除工作

<!-- more -->

## 案例

```javascript
// ./text.js
export const text1 = "text1"
export const text2 = "text2"

// ./index
import {text1} from './text'
console.log(text1)

// ./webpack.config.js
const webpack = require("webpack")
const path = require("path")
const UglifyJSPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
    entry: {
        index: path.resolve(__dirname, './index.js')
    },
    output: {
        path: path.resolve(__dirname, './'),
        filename: "bundle.js"
    },
    plugins: [
        // new webpack.optimize.UglifyJsPlugin({
        //     compress: {
        //         warnings: false
        //       }
        // })
    ]
}

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

一开始将webpack.config.js文件中 UglifyJsPlugin 插件代码注释，执行 webpack 命令查看打包的 bundle.js 文件

```javascript
...
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
const text1 = "text1"
/* harmony export (immutable) */ __webpack_exports__["a"] = text1;

const text2 = "text2"
/* unused harmony export text2 */
/***/ })
```

注意 bundle.js 文件 unused harmony export square 注释， text2 变量没有被导出，但是仍然包含在 bundle.js 代码中

## 使用 UglifyJsPlugin 结合 es6 模块导入导出 实现 tree shaking

webpack.config.js 中 开启 UglifyJsPlugin 插件代码

```javascript
module.exports = {
    entry: {
        index: path.resolve(__dirname, './index.js')
    },
    output: {
        path: path.resolve(__dirname, './'),
        filename: "bundle.js"
    },
    plugins: [
        new webpack.optimize.UglifyJsPlugin({})
    ]
}
```

重新执行 webpack 命令， 发现打包后的 bundle.js 中 text2 不会为引入，只有 ```function(e,t,r){"use strict";t.a="text1"}```, 而且整个文件进行了代码压缩，tree shaking 对 bundle 进行了明显的体积优化

## tree shaking 副作用

当 webpack 不确定该代码是否存在于 bundle 中，也就是删除会不会引起其他副作用时，为保险起见会保留该代码块。常见情况有:

- 从第三方模块中调用一个函数，webpack 和 压缩工具无法检查此模块

- 从第三方模块导入的函数被重新导出等
