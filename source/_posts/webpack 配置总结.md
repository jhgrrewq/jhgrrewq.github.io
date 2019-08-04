---
title: webpack 配置总结
tag: 
	- webpack
---

参考: [webpack 官网 配置选项部分](https://webpack.docschina.org/configuration/)

<!-- markdownlint-disable MD010 -->

```js
const path = require('path')

module.exports = {
  // "production" | "development" | "none"
  mode: 'production', // 提供生产环境构建优化
  mode: 'development', // 提供开发环境有用的工具
  mode: 'none', // 不设置默认模式
  // 特定模式告诉 webpack 使用该模式下内置的优化配置

  // string | array | object
  entry: './app/entry',
  entry: ['./app/entry1', './app/entry2'],
  entry: {
    a: './app/entry-a',
    b: ['./app/entry-b1', './app/entry-b2']
  }
  // 应用程序从 entry 开始执行，webpack 开始打包

  output: {
    // webpack 如何输出结果的相关选项

    path: path.resolve(_dirname, 'dist'), // string
    // 所有输出文件的目标路径
    // 必须是绝对路径（使用 Node.js 的 path 模块）

    filename: 'bundle.js', // string
    filename: '[name].js', // 用于多个入口点（entry point）（出口点？）
    filename: '[chunkhash].js', // 用于长效缓存
    // 入口模块（entry chunk）的文件名模板（出口分块？）

    publicPath: '/assets', // string
    // 输出解析文件的目录，url 相对 html 页面

    library: 'MyLibrary', // string
    // 导出库的名称

    libraryTarget: 'umd', // 通用模块定义
    libraryTarget: 'commonjs2', // exported with module.exports
    libraryTarget: 'commonjs-module', // 使用 module.exports 导出
    libraryTarget: 'commonjs', // 使用 exports 的属性导出
    libraryTarget: 'amd', // 使用 amd 定义方法来定义
    libraryTarget: 'this', // 在 this 上设置属性
    libraryTarget: 'var', // 变量定义于根作用域下
    libraryTarget: 'window', // 在 window 对象上设置属性
    libraryTarget: 'global', // 在 globel 对象上设置属性
    // 导出库的类型

    chunkFileName: '[id].js',
    chunkFileName: '[chunkhash].js', // 长效缓存
    // 附加分块的文件名模板

    // ...
  },

  module: {
    // 关于模块配置

    rules: {
      // 模块规则（配置 loader、解析器等选项）

      {
        test: '/\.jsx?$/',
        include: [path.resolve(__dirname, 'app')],
        exclude: [path.resolve(__dirname, 'app/demo-files')],
        // 这里是匹配条件，每个选项都接受一个正则表或字符串
        // test 和 include 具有相同作用，都是必须匹配选项，exclude 是不匹配选项（优先于 test 和 include）
        // 最佳实践：
        // 只在 test 和 文件名匹配 中使用正则表达式
        // 在 include 和 exclude 中使用绝对路径
        // 尽量避免使用 exclude，更倾向于使用 include

        loader: 'babel-loader',
        // 使用的 loader，它相对上下文解析
        // webpack 2 后 '-loader' 后缀不再是可选

        options: {
          presets: ['es2015']
        }
        // loader 的可选项
        // webpack 2 后尽量是用 options 替代 query

      },

      {
        test: '/\.html$/',
        test: '\.html$',

        use: [
          // 应用多个 loader 和选项
          'htmllint-loader',
          {
            loader: 'html-loader',
            options: {
              // ...
            }
          }
        ]
      }
    },

    resolve: {
      // 解析模块请求的选项
      // 不适用于对 loader 解析

      modules: [
        "node_modules",
        path.resolve(__dirname, 'app')
      ],
      // 用于查找模块的目录

      extensions: ['.js', '.json', '.jsx', '.css'],
      // 使用的扩展名

      alias: {
        // 模块别名列表

        "module": "new-module",
        // 起别名: 'module' -> 'new-module' 和 'module/path/file' -> 'new-module/path/file'

        "only-module$": "new-module",
        // 起别名: 'only-module' -> 'new-module', 不匹配 'only-module/path/file' -> 'new-module/path/file'

        "module": path.resolve(__dirname, "app/third/module.js"),
        // 起别名 "module" -> "./app/third/module.js" 和 "module/file" 会导致错误
        // 模块别名相对于当前上下文导入
      }
    },

    devtool: 'source-map', // enum
    // 通过在浏览器调试工具中添加原信息增强调试
    // 牺牲了构建强度的 `source-map` 是最详细的

    context: __dirname, // string(绝对路径！)
    // webpack 主目录
    // entry 和 module.rules.loader 选项
    // 相对于此目录解析

    externals: ['react', /^@angular\//],
    // 不要遵循/打包这些模块，而是在运行环境中从环境中请求他们

    stats: 'errors-only',
    // 精确控制要显示的 bundle 信息

    devServer: {
      proxy: {
        // 服务器 proxy URLS
        '/api': 'http://localhost: 3000', // 'localhost:8080/api/test' -> 'localhost: 3000/api/test'
        '/api2': {
          // 'localhost:8080/api2/test' -> 'localhost:3000/test'
          target: 'http://localhost: 3000',
          changeOrigin: true,
          pathRewrite: {
            '^/api2': ''
          }
        }
      },

      contentBase: path.join(__dirname, 'public') // boolean | string | array, 静态文件目录
      compress: true, // 开启 gzip 压缩
      historyApiFallback: true, // 不匹配的路由指向 index.html，不会报错 404
      historyApiFallback: {
        // 多个匹配路径使用对象设置
        rewrites: [{
          from: '/^\/subpage/',
          to: '/views/subpage/index.html'
        },
        {
          from: '/^\/hello\/.*$/',
          to: function() {
            return '/views/subpage/index.html'
          }
        }]
      },
      hot: true, // 热切换，依赖 HotModuleReplacementPlugin
      https: false,
      // ...
    },

    plugins: [
      // ...
    ]
    // 插件列表

    // ...
  }
}
```

<!-- more -->

## webpack 基本概念

> webpack 是一个 js 应用程序的静态模块打包器，它会递归构建一个依赖关系图，其中包含程序需要的每个模块，然后将这些模块打包成一个或多个 bundle

- 一切都是模块
- 按需加载

webpack 有几个基本概念：

- 入口（entry）
- 输出（output）
- 模块（module，重点理解 loader 加载器）
- 插件（plugins）

### 入口（entry）

进入入口起点后，webpack 会找出哪些模块和库是入口(直接或间接)依赖的

每个依赖项随即被处理，如果依赖项还依赖其他模块，会递归进行处理，最后输出到 bundles 文件中

### 输出（output）

`output` 属性告诉 webpack 在哪里输出它所构建的 bundles，以及如何命名这些文件

### loader

webpack 默认只能处理 es5 语法的 js 文件，loader 让 webpack 能够处理非 js 文件。loader 可以将所有类型文件转换为 webpack 能够处理的有效模块

### 插件（plugins）

loader 被用于转换某些类型的模块，而插件用于执行范围更广的任务。插件的范围包括从打包优化和压缩，一直到重新定义环境中的变量等。

使用一个插件，只需要 `require()` 它，然后用它创建一个实例并添加到 `plugins` 数组中。多数插件可以通过选项 option 自定义

## webpack 使用

### webpack 命令

```js
  --config               Path to the config file
                         // 指定配置文件，默认是 webpack.config.js 或 webpackfile.js
                          [字符串] [默认值: webpack.config.js or webpackfile.js]
  --config-register, -r  Preload one or more modules before loading the webpack
                         configuration        [数组] [默认值: module id or path]
  --config-name          Name of the config to use                      [字符串]
  --env                  Environment passed to the config, when it is a function
                         // 当 webpack 配置是一个函数，环境变量作为参数传入
  --mode                 Enable production optimizations or development hints.
                                   [可选值: "development", "production", "none"]

Basic options:
  --context    The base directory (absolute path!) for resolving the `entry`
               option. If `output.pathinfo` is set, the included pathinfo is
               shortened to this directory.
               // 上下文，默认为当前目录，已经使用项目根目录
                                        [字符串] [默认值: The current directory]
  --entry      The entry point(s) of the compilation.                   [字符串]
  --watch, -w  Enter watch mode, which rebuilds on file change.           [布尔]
               // watch 模式，监听文件改变
  --debug      Switch loaders to debug mode                               [布尔]
  --devtool    A developer tool to enhance debugging.                   [字符串]
  -d           shortcut for --debug --devtool eval-cheap-module-source-map
               --output-pathinfo                                          [布尔]
  -p           shortcut for --optimize-minimize --define
               process.env.NODE_ENV="production"                          [布尔]
  --progress   Print compilation progress in percentage                   [布尔]
               // 输出文件打包构建进度情况

Module options:
  --module-bind       Bind an extension to a loader                     [字符串]
  --module-bind-post  Bind an extension to a post loader                [字符串]
  --module-bind-pre   Bind an extension to a pre loader                 [字符串]

Output options:
  --output, -o                  The output path and file for compilation assets
                                // 指定打包资源输出路径
  --output-path                 The output directory as **absolute path**
                                (required).
                                        [字符串] [默认值: The current directory]
  --output-filename             Specifies the name of each output file on disk.
                                You must **not** specify an absolute path here!
                                The `output.path` option determines the location
                                on disk the files are written to, filename is
                                used solely for naming the individual files.
                                                    [字符串] [默认值: [name].js]
  --output-chunk-filename       The filename of non-entry chunks as relative
                                path inside the `output.path` directory.
        [字符串] [默认值: filename with [id] instead of [name] or [id] prefixed]
  --output-source-map-filename  The filename of the SourceMaps for the
                                JavaScript files. They are inside the
                                `output.path` directory.                [字符串]
  --output-public-path          The `publicPath` specifies the public URL
                                address of the output files when referenced in a
                                browser.                                [字符串]
  --output-jsonp-function       The JSONP function used by webpack for async
                                loading of chunks.                      [字符串]
  --output-pathinfo             Include comments with information about the
                                modules.                                  [布尔]
  --output-library              Expose the exports of the entry point as library
                                                                        [字符串]
  --output-library-target       Type of library
          [字符串] [可选值: "var", "assign", "this", "window", "self", "global",
      "commonjs", "commonjs2", "commonjs-module", "amd", "umd", "umd2", "jsonp"]

Advanced options:
  --records-input-path       Store compiler state to a json file.       [字符串]
  --records-output-path      Load compiler state from a json file.      [字符串]
  --records-path             Store/Load compiler state from/to a json file. This
                             will result in persistent ids of modules and
                             chunks. An absolute path is expected. `recordsPath`
                             is used for `recordsInputPath` and
                             `recordsOutputPath` if they left undefined.[字符串]
  --define                   Define any free var in the bundle          [字符串]
  --target                   Environment to build for                   [字符串]
  --cache                    Cache generated modules and chunks to improve
                             performance for multiple incremental builds.
                          [布尔] [默认值: It's enabled by default when watching]
  --watch-stdin, --stdin     Stop watching when stdin stream has ended    [布尔]
  --watch-aggregate-timeout  Delay the rebuilt after the first change. Value is
                             a time in ms.                                [数字]
  --watch-poll               Enable polling mode for watching           [字符串]
  --hot                      Enables Hot Module Replacement               [布尔]
  --prefetch                 Prefetch this request (Example: --prefetch
                             ./file.js)                                 [字符串]
  --provide                  Provide these modules as free vars in all modules
                             (Example: --provide jQuery=jquery)         [字符串]
  --labeled-modules          Enables labeled modules                      [布尔]
  --plugin                   Load this plugin                           [字符串]
  --bail                     Report the first error as a hard error instead of
                             tolerating it.                [布尔] [默认值: null]
  --profile                  Capture timing information for each module.
                                                           [布尔] [默认值: null]
Resolving options:
  --resolve-alias         Redirect module requests                      [字符串]
  --resolve-extensions    Redirect module requests                        [数组]
  --resolve-loader-alias  Setup a loader alias for resolving            [字符串]

Optimizing options:
  --optimize-max-chunks      Try to keep the chunk count below a limit
  --optimize-min-chunk-size  Minimal size for the created chunk
  --optimize-minimize        Enable minimizing the output. Uses
                             optimization.minimizer.                      [布尔]

Stats options:
  --color, --colors               Enables/Disables colors on the console
                                  // 开启 console 颜色输出
                                               [布尔] [默认值: (supports-color)]
  --sort-modules-by               Sorts the modules list by property in module
                                                                        [字符串]
  --sort-chunks-by                Sorts the chunks list by property in chunk
                                                                        [字符串]
  --sort-assets-by                Sorts the assets list by property in asset
                                                                        [字符串]
  --hide-modules                  Hides info about modules                [布尔]
  --display-exclude               Exclude modules in the output         [字符串]
  --display-modules               Display even excluded modules in the output
                                                                          [布尔]
  --display-max-modules           Sets the maximum number of visible modules in
                                  output                                  [数字]
  --display-chunks                Display chunks in the output            [布尔]
  --display-entrypoints           Display entry points in the output      [布尔]
  --display-origins               Display origins of chunks in the output [布尔]
  --display-cached                Display also cached modules in the output
                                                                          [布尔]
  --display-cached-assets         Display also cached assets in the output[布尔]
  --display-reasons               Display reasons about module inclusion in the
                                  output                                  [布尔]
  --display-depth                 Display distance from entry point for each
                                  module                                  [布尔]
  --display-used-exports          Display information about used exports in
                                  modules (Tree Shaking)                  [布尔]
  --display-provided-exports      Display information about exports provided
                                  from modules                            [布尔]
  --display-optimization-bailout  Display information about why optimization
                                  bailed out for modules                  [布尔]
  --display-error-details         Display details about errors            [布尔]
  --display                       Select display preset
               [字符串] [可选值: "", "verbose", "detailed", "normal", "minimal",
                                                          "errors-only", "none"]
  --verbose                       Show more details                       [布尔]
  --info-verbosity                Controls the output of lifecycle messaging
                                  e.g. Started watching files...
                   [字符串] [可选值: "none", "info", "verbose"] [默认值: "info"]
  --build-delimiter               Display custom text after build output[字符串]

选项：
  --help, -h     显示帮助信息                                             [布尔]
  --version, -v  显示版本号                                               [布尔]
  --silent       Prevent output from being displayed in stdout            [布尔]
  --json, -j     Prints the result as JSON.                               [布尔]
```

命令行中常用的几个命令

- `webpack --version` 查看 webpack 版本
- `webpack -h` 查看 webpack 帮助信息
- `webpack <entry-filePath> --output <output-filePath>` 指定入口和输出进行打包
- `webpack --config webpack.js` 指定 webpack 配置文件为 webpack.js 进行打包
- `webpack --env=development` 当 webpack 配置为函数，可以传入环境变量作为参数
- `webpack --watch` 开启 watch 模式，监听文件修改，但是没有文件刷新和热切换
- `webpack --debug` 开启 debug 模式
- `webpack --progress` webpack 打包构建过程中进度情况输出
- `webpack --color` webpack 打包构建过程开启 console 颜色输出

### webpack 配置

当 webpack 配置较多，不适合在命令行中，推荐使用配置文件

- **`webpack`** webpack 命令执行默认执行的是 `webpack.config.js` 或 `webpackfile.js`
- **`webpack --config webpack.js`** 当使用自定义 webpack 配置文件名时，使用 `--config` 指定

### 第三方脚手架

- vue-cli
- angular-cli
- react-starter

## 打包 js 文件

webpack 只能处理 es5 规范的 js 文件，对于其他类型文件需要借助 loader 处理

对于模块规范 es6 commonJS amd，注意 **amd 会把依赖单独打包成一个 chunk 进行引用**

```js
// index.js
const cmj = require('./cmj')
console.log('cmj: ' + cmj)

import esm from './esm'
console.log('esm: ' + esm)

// 注意 amd 依赖会被单独打包成一个 chunk 并进行引入
require(['./amd'], function(amd) {
  console.log('amd: ' + amd)
})
```

```js
Built at: 2018-06-28 10:30:12
        Asset       Size  Chunks             Chunk Names
   0.abf8f.js  152 bytes       0  [emitted]
main.abf8f.js    2.2 KiB       1  [emitted]  main
[0] ./index.js + 1 modules 247 bytes {1} [built]
    | ./index.js 208 bytes [built]
    | ./esm.js 39 bytes [built]
[1] ./cmj.js 25 bytes {1} [built]
[2] ./amd.js 62 bytes {0} [built]
```

## babel

### 基本使用

- 安装 `babel-core` `babel-loader`
- 针对语法转换需要安装 preset `babel-preset-env`(env 会根据环境安装 es2015 es2016 es2017 规范，还可以安装 babel-preset-stag-0/1/2/3 或 babel-preset-react 等)
- 针对函数和方法需要插件 `babel-polyfill` 或 `babel-plugin-transform-runtime`
- **编写配置，babel preset 和 plugin 的配置可以放在 webpack 配置 对应 loader 的 options 中，也可以放在 package.json 中，但是推荐单独作为 `.babelrc` 文件**

```js
// webpack.config.js
module.exports = {
  // ...
  module: {
    rules: [{
      test: /\.js$/,
      exclude: /node_modules/,
      use: [{
        loader: 'babel-loader',
        options: {
          presets: ['env']
        }
      }]
    }]
  }
}
```

```js
// app.js
const arr = [1, 2, 3].map((item) => item * 2)
console.log('test:', Array.from(new Set(arr)))
```

打包后文件可以发现已经被转化为 es5 语法

```js
! function (e) {
  // ...
}([function (e, t, r) {
  "use strict";
  var n = [1, 2, 3].map(function (e) {
    return 2 * e
  });
  console.log("test:", Array.from(new Set(n)))
}]);
}
```

```js
// webpack.config.js
module.exports = {
  // ...
  module: {
    rules: [{
      test: /\.js$/,
      exclude: /node_modules/,
      use: [{
        loader: 'babel-loader',
        options: {
          ['env', {
            targets: {
              browers: ['last 2 version', '> 1%']
            }
          }]
        }
      }]
    }]
  }
}
```

```js
// webpack.config.js
module.exports = {
  // ...
  module: {
    rules: [{
      test: /\.js$/,
      exclude: /node_modules/,
      use: [{
        loader: 'babel-loader',
        options: {
          ['env', {
            targets: {
              chrome: '52'
            }
          }]
        }
      }]
    }]
  }
}
```

```js
! function (e) {
  // ...
}([function (e, t, r) {
  "use strict";
  const n = [1, 2, 3].map(e => 2 * e);
  console.log("test:", Array.from(new Set(n)))
}]);
}
```

### babel-polyfill vs babel-plugin-transfform-runtime

配置了 preset 只是针对语法能进行转换，对于函数和方法需要用到 `babel-polyfill`  或`babel-plugin-transfform-runtime`

#### babel-polyfill

- 全局垫片
- **开发应用使用**，会把整个库打包进文件导致文件体积较大
- **入口文件引入 `import 'babel-polyfill'`**

```js
// 会把所有函数和方法打包进文件，导致文件体积过大
// ...
$export($export.S + $export.F * !USE_NATIVE, 'Object', {
  // 19.1.2.2 Object.create(O [, Properties])
  create: $create,
  // 19.1.2.4 Object.defineProperty(O, P, Attributes)
  defineProperty: $defineProperty,
  // 19.1.2.3 Object.defineProperties(O, Properties)
  defineProperties: $defineProperties,
  // 19.1.2.6 Object.getOwnPropertyDescriptor(O, P)
  getOwnPropertyDescriptor: $getOwnPropertyDescriptor,
  // 19.1.2.7 Object.getOwnPropertyNames(O)
  getOwnPropertyNames: $getOwnPropertyNames,
  // 19.1.2.8 Object.getOwnPropertySymbols(O)
  getOwnPropertySymbols: $getOwnPropertySymbols
});
// ...
/* 326 */
/***/ (function(module, exports, __webpack_require__) {

"use strict";
/* WEBPACK VAR INJECTION */(function(global) {

__webpack_require__(325);

__webpack_require__(128);

__webpack_require__(127);

if (global._babelPolyfill) {
  throw new Error("only one instance of babel-polyfill is allowed");
}
global._babelPolyfill = true;

var DEFINE_PROPERTY = "defineProperty";
function define(O, key, value) {
  O[key] || Object[DEFINE_PROPERTY](O, key, {
    writable: true,
    configurable: true,
    value: value
  });
}

define(String.prototype, "padLeft", "".padStart);
define(String.prototype, "padRight", "".padEnd);

"pop,reverse,shift,keys,values,entries,indexOf,every,some,forEach,map,filter,find,findIndex,includes,join,slice,concat,push,splice,unshift,sort,lastIndexOf,reduce,reduceRight,copyWithin,fill".split(",").forEach(function (key) {
  [][key] && define(Array, key, Function.call.bind([][key]));
});
/* WEBPACK VAR INJECTION */}.call(this, __webpack_require__(124)))

/***/ }),
/* 327 */
/***/ (function(module, exports, __webpack_require__) {

"use strict";


__webpack_require__(326);

var arr = [1, 2, 3].map(function (item) {
  return item * 2;
});
console.log('test:', Array.from(new Set(arr)));

/***/ })
/******/ ]);
```

#### babel-plugin-transform-runtime

- 局部垫片
- **开发库使用**，只会将使用到的函数和方法打包进文件
- **`.babelrc` 文件配置 `plugins`**

```js
// .babelrc
{
  "presets": [
    ["env", {
      "targets": {
        "browers": ["last 2 version"]
      }
    }]
  ],
  "plugins": ["transform-runtime"]
}
```

```js
// 只会将使用到的函数和方法打包进文件
// ...
/* 80 */
/***/ (function(module, exports, __webpack_require__) {
"use strict";
var _set = __webpack_require__(79);

var _set2 = _interopRequireDefault(_set);

var _from = __webpack_require__(46);

var _from2 = _interopRequireDefault(_from);

function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }

// import 'babel-polyfill'
var arr = [1, 2, 3].map(function (item) {
  return item * 2;
});
arr.includes(8);
console.log('test:', (0, _from2.default)(new _set2.default(arr)));

/***/ })
/******/ ]);
```

## 打包公共代码

打包公共代码的优点在于能减少代码冗余，提升加载速度

### wepack.optimize.CommonsChunkPlugin 用法

`wepack.optimize.CommonsChunkPlugin` 插件中 option 配置：

- **`name`** chunk 名称，将提取的公共代码作为一个 chunk，它的名称指定为该属性值
- **`names`** chunk 名称数组，表示根据配置新建插件实例多少次
- **`filename`** 文件名，可以使用 [chunkhash] [hash] 等占位符
- **`minChunks`** 如果是数字，表示达到提取的公共代码出现的次数才提取；如果是 infinite, 创建 common chunk 但是不会放入任何 module；还可以是一个函数，可以自定义提取公共代码逻辑
- **`chunks`** 指定提取公共代码范围
- **`children`** 在子模块中查找共同依赖
- **`deepChildren`** 在所有后代模块中查找共同依赖
- **`async`** 创建异步公共代码块

**使用方法：webpack 配置 `plugin` 中添加 `wepack.optimize.CommonsChunkPlugin` 插件实例，传入 option**

```js
{
  // ...
  plugins: [
    new webpack.optimize.CommonsChunkPlugin(option)
  ]
}
```

### 单页应用场景

```js
// moduleA.js
const moduleA = 'moduleA'
export default moduleA

// subPage.js
import './moduleA'
export default 'subPageA'

// pageA.js
import './moduleA'
import './subPageA'
```

`pageA.js` 作为单页入口，`pageA.js` 和 `subPageA.js` 都引入 `moduleA.js`

```js
// webpack.config.js
const path = require('path')
const webpack = require('webpack')

module.exports = {
  entry: {
    pageA: path.resolve(__dirname, './src/pageA.js')
  },
  output: {
    path:  path.resolve(__dirname, './dist'),
    filename: '[name].[chunkhash:8].js'
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'common',
      miniChunks: 2
    })
  ]
}
```

打包后的 `common.62221439.js` 只有 webpack 运行时代码，并没有打包 `moduleA.js`

```js
             Asset       Size  Chunks             Chunk Names
 pageA.3e073468.js  875 bytes       0  [emitted]  pageA
common.62221439.js    5.81 kB       1  [emitted]  common
   [0] ./src/moduleA.js 46 bytes {0} [built]
   [1] ./src/pageA.js 38 bytes {0} [built]
   [2] ./src/subPageA.js 44 bytes {0} [built]
```

**说明 `CommonsChunkPlugin` 对于只是打包单页面应用是无效，对多入口有效**

### 单页应用 + 第三方依赖场景

`pageA.js` 引入第三方依赖 lodash

```js
// pageA.js
import './moduleA'
import './subPageA'

import lodash from 'lodash'
console.log(lodash)
```

`webpack.config.js` 中第三方依赖单独作为一个入口进行打包

```js
const path = require('path')
const webpack = require('webpack')

module.exports = {
  entry: {
    pageA: path.resolve(__dirname, './src/pageA.js'),
    vendor: ['lodash']
  },
  output: {
    path:  path.resolve(__dirname, './dist'),
    filename: '[name].[chunkhash:8].js'
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      miniChunks: Infinity
    })
  ]
}
```

```js
            Asset     Size  Chunks                    Chunk Names
 pageA.f811a894.js  1.17 kB       0  [emitted]         pageA
vendor.c1d424ab.js   547 kB       1  [emitted]  [big]  vendor
   [0] ./src/moduleA.js 46 bytes {0} [built]
   [2] ./src/pageA.js 87 bytes {0} [built]
   [3] ./src/subPageA.js 44 bytes {0} [built]
   [4] (webpack)/buildin/global.js 509 bytes {1} [built]
   [5] (webpack)/buildin/module.js 517 bytes {1} [built]
   [6] multi lodash 28 bytes {1} [built]
    + 1 hidden module
```

打包后的 `vendor.c1d424ab.js` 将 lodash 进行了打包

### 多页应用 + 第三方依赖 + webpack 生成代码场景

增加 `pageB.js` 变成多页面, 同样依赖 `moduleA.js`

```js
// moduleA.js
const moduleA = 'moduleA'
export default moduleA

// subPage.js
import './moduleA'
export default 'subPageA'

// pageA.js
import './moduleA'
import './subPageA'

import lodash from 'lodash'
console.log(lodash)

// pageB.js
import './moduleA'
```

`webpack.config.js` 中第三方依赖单独作为一个入口进行打包, 入口变成多入口，将提取 webpack 运行时代码，第三方依赖，业务代码共同依赖分别作为单独的 chunk

**业务代码共同依赖需要指定从哪些文件中查找共同依赖**

```js
const path = require('path')
const webpack = require('webpack')

module.exports = {
  entry: {
    pageA: path.resolve(__dirname, './src/pageA.js'),
    pageB: path.resolve(__dirname, './src/pageB.js'),
    vendor: ['lodash']
  },
  output: {
    path:  path.resolve(__dirname, './dist'),
    filename: '[name].[chunkhash:8].js'
  },
  plugins: [
    // 业务代码共同依赖
    // minChunks 指定共同依赖 2 次以上才提取
    // chunks 指定从哪些文件中查找业务代码共同依赖
    new webpack.optimize.CommonsChunkPlugin({
      name: 'common',
      miniChunks: 2,
      chunks: ['pageA', 'pageB']
    }),

    // 第三方依赖单独提取
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      miniChunks: Infinity
    }),

    // webpack 运行时代码单独提取
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      miniChunks: Infinity
    })
  ]
}
```

```js
              Asset       Size  Chunks                    Chunk Names
  vendor.26e18ac4.js     542 kB       0  [emitted]  [big]  vendor
   pageA.e05b1b55.js  966 bytes       1  [emitted]         pageA
   pageB.d911660b.js  296 bytes       2  [emitted]         pageB
  common.12cca4c8.js  231 bytes       3  [emitted]         common
manifest.0f9adf7b.js    5.85 kB       4  [emitted]         manifest
   [0] ./src/moduleA.js 46 bytes {3} [built]
   [2] ./src/pageA.js 87 bytes {1} [built]
   [3] ./src/subPageA.js 44 bytes {1} [built]
   [4] (webpack)/buildin/global.js 509 bytes {0} [built]
   [5] (webpack)/buildin/module.js 517 bytes {0} [built]
   [6] ./src/pageB.js 18 bytes {2} [built]
   [7] multi lodash 28 bytes {0} [built]
    + 1 hidden module
```

打包后的 `common.12cca4c8.js` 将 `moduleA.js` 进行了打包， `vendor.26e18ac4.js` 将 lodash 进行了打包， `manifest.0f9adf7b.js` 将 webpack 运行时代码提取打包

## 代码分割和懒加载

### require.ensure

```js
require.ensure(
  dependencies: String[],
  callback: function(require),
  errorCallback: function(error),
  chunkName: String
)
```

**`require.ensure` 可以将依赖的模块单独打包成一个 chunk，并异步加载**，接受四个参数：

- `dependencise` 字符串或是数组，可以置空，**注意依赖只是被引入，并没有执行，只有在 callback 中 require 后才执行**
- `callback` 回调接受 require 作为参数，只有在回调中 require 依赖的模块才会执行
- `errorCallback` 可选，错误回调
- `chunkName` 给分割的依赖 指定 chunkName

```js
require.include(dependency: String)
```

`require.include` 会将多个模块中引入的公共子模块先引入但并未执行，同样会单独打包

```js
require.include('a');
require.ensure(['a', 'b'], function(require) { /* ... */ });
require.ensure(['a', 'c'], function(require) { /* ... */ });
```

### import()

`import()` 将引入的模块作为依赖单独分割(**可以指定 chunkName，指定为同一个 chunkName 的依赖会被打包在一起**)，并且引入的模块会执行，返回一个 promise

```js
import(
  /* webpackChunkName: "my-chunk-name" */
  /* webpackMode: "lazy" */
  'module'
).then((module) => {
  // ....
})
```

### require.ensure 应用场景

pageA.js 中使用 `require.ensure`

```js
// moduleA.js
const moduleA = 'moduleA'
console.log('moduleA')
export default module

// subPage.js
import './moduleA'
console.log('subPageA')
export default 'subPageA'

// pageA.js
require.ensure([], function(require) {
  var subPageA = require('./subPageA')
}, 'subPageA')

require.ensure([], function(require) {
  var moduleA = require('./moduleA')
}, 'moduleA')

require.ensure([], function(require) {
  var _ = require('lodash')
  console.log(_.join(1, 2))
}, 'vendor')
```

打包后依赖都打包成对应的 chunk

```js
 Asset       Size  Chunks                    Chunk Names
    0.bundle.js     541 kB       0  [emitted]  [big]  vendor
    1.bundle.js  694 bytes    1, 2  [emitted]         subPageA
    2.bundle.js  324 bytes       2  [emitted]         moduleA
pageA.bundle.js    6.65 kB       3  [emitted]         pageA
   [0] ./src/pageA.js 612 bytes {3} [built]
   [1] ./src/moduleA.js 66 bytes {1} {2} [built]
   [2] ./src/subPageA.js 68 bytes {1} [built]
   [4] (webpack)/buildin/global.js 509 bytes {0} [built]
   [5] (webpack)/buildin/module.js 517 bytes {0} [built]
    + 1 hidden module
```

- 如果 `require.ensure` 中指定相同的 chunkName，依赖会打包到同一个 chunk 中

```js
// pageA.js
// subPageA.js 和 moduleA.js 都指定同一个 chunkName
require.ensure([], function(require) {
  var subPageA = require('./subPageA')
}, 'subPageA')

require.ensure([], function(require) {
  var moduleA = require('./moduleA')
}, 'subPageA')
```

等同于

```js
// pageA.js
require.ensure([], function(require) {
  var subPageA = require('./subPageA')
  var moduleA = require('./moduleA')
}, 'subPageA')

// 或者
// require.ensure 的依赖数组可为空，这里依赖只引入，只有在回调用 require 才执行
require.ensure(['./subPageA', './moduleA'], function(require) {
  var subPageA = require('./subPageA')
  var moduleA = require('./moduleA')
}, 'subPageA')
```

打包后效果一样

```js
          Asset       Size  Chunks                    Chunk Names
    0.bundle.js  692 bytes       0  [emitted]         subPageA
    1.bundle.js     541 kB       1  [emitted]  [big]  vendor
pageA.bundle.js    6.65 kB       2  [emitted]         pageA
   [0] ./src/pageA.js 613 bytes {2} [built]
   [1] ./src/moduleA.js 66 bytes {0} [built]
   [2] ./src/subPageA.js 68 bytes {0} [built]
   [4] (webpack)/buildin/global.js 509 bytes {1} [built]
   [5] (webpack)/buildin/module.js 517 bytes {1} [built]
    + 1 hidden module
```

- `require.ensure` 依赖数组中，依赖只是引入，并没有执行，只有在回调中 require 才执行

```js
// subPage.js moduleA.js 中的 console.log() 不会执行
require.ensure(['./subPageA'], function(require) {
  // var subPageA = require('./subPageA')
}, 'subPageA')

require.ensure(['./moduleA'], function(require) {
  // var moduleA = require('./moduleA')
}, 'moduleA')

require.ensure(['lodash'], function(require) {
  // var _ = require('lodash')
  // console.log(_.join(1, 2))
}, 'vendor')
```

### import() 使用场景

```js
// pageA.js
import(/* webpackChunkName: 'vendor' */ 'lodash').then(_ => {
  // console.log(_)
})

import(/* webpackChunkName: 'moduleA' */ './moduleA.js').then(moduleA => {
  // console.log(moduleA)
})

import(/* webpackChunkName: 'subPageA' */ './subPageA.js').then(subPageA => {
  // console.log(subPageA)
})
```

打包后依赖分割打包进对应的 chunk，**这里依赖 import() 就执行了**

```js
          Asset       Size  Chunks                    Chunk Names
    0.bundle.js     541 kB       0  [emitted]  [big]  vendor
    1.bundle.js  703 bytes    1, 2  [emitted]         subPageA
    2.bundle.js  324 bytes       2  [emitted]         moduleA
pageA.bundle.js    6.51 kB       3  [emitted]         pageA
   [0] ./src/pageA.js 657 bytes {3} [built]
   [1] ./src/moduleA.js 66 bytes {1} {2} [built]
   [3] ./src/subPageA.js 68 bytes {1} [built]
   [4] (webpack)/buildin/global.js 509 bytes {0} [built]
   [5] (webpack)/buildin/module.js 517 bytes {0} [built]
    + 1 hidden module
```

同样可以指定相同的 chunkName，将依赖打包进同一个 chunk

```js
import(/* webpackChunkName: 'subPageA' */ './moduleA.js').then(moduleA => {
  // console.log(moduleA)
})

import(/* webpackChunkName: 'subPageA' */ './subPageA.js').then(subPageA => {
  // console.log(subPageA)
})
```

```js
Asset       Size  Chunks                    Chunk Names
    0.bundle.js  701 bytes       0  [emitted]         subPageA
    1.bundle.js     541 kB       1  [emitted]  [big]  vendor
pageA.bundle.js    6.51 kB       2  [emitted]         pageA
   [0] ./src/pageA.js 658 bytes {2} [built]
   [1] ./src/moduleA.js 66 bytes {0} [built]
   [3] ./src/subPageA.js 68 bytes {0} [built]
   [4] (webpack)/buildin/global.js 509 bytes {1} [built]
   [5] (webpack)/buildin/module.js 517 bytes {1} [built]
    + 1 hidden module
```

## tree shaking

