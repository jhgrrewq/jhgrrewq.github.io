---
title: vue-cli webpack 配置
tag: 
	- vue
	- vue-cli
---

> 参考: [vue-cli的webpack模板项目配置文件分析](https://blog.csdn.net/hongchh/article/details/55113751)

<!-- markdownlint-disable MD010 -->

vue-cli 提供了很多的模板，通常我们初始化一个 vue 单页面项目都会使用 [webpack 的模板](https://github.com/vuejs-templates/webpack)。然而 webpack 的模板在 1.1.3 版本之后也就是 1.2.0 版本开始，webpack 配置不在使用自定义的服务器，而是引入 webpack-dev-server。想要使用之前的 webpack 模板初始化需要使用命令 `vue init webpack#1.1.3 <project-name>`

<!-- more -->

先以 webpack#1.1.3 版本分析 vue-cli 配置

## webpack#1.1.3

### 文件结构

主要分析开发和构建两个阶段涉及的文件

```tree
├── build
│   ├── build.js
│   ├── check-versions.js
│   ├── dev-client.js
│   ├── dev-server.js
│   ├── utils.js
│   ├── vue-loader.conf.js
│   ├── webpack.base.conf.js
│   ├── webpack.dev.conf.js
│   ├── webpack.prod.conf.js
│   └── webpack.test.conf.js
├── config
│   ├── dev.env.js
│   ├── index.js
│   ├── prod.env.js
│   └── test.env.js
├── ...
└── package.json
```

### 指令

`package.json` 中的 `script` 字段定义了项目的运行脚本

```js
"scripts": {
	"dev": "node build/dev-server.js",
	"start": "npm run dev",
	"build": "node build/build.js",
	"unit": "cross-env BABEL_ENV=test karma start test/unit/karma.conf.js --single-run",
	"e2e": "node test/e2e/runner.js",
	"test": "npm run unit && npm run e2e",
	"lint": "eslint --ext .js,.vue src test/unit/specs test/e2e/specs"
}
```

重点关注 `dev` 和 `build`。运行 `npm run dev` 执行的是 `build/dev-server.js`, 运行 `npm run build` 执行的是 `build/build.js`,

### build 文件夹分析

#### build/dev-serve.js

>该文件主要完成下面几件事：
- 检查 node 和 npm 的版本、引入相关插件和配置
- **webpack 对源码进行打包并返回 compiler 对象**
- **创建 express 服务器**
- **配置开发中间件（wbpack-dev-middleware）和热重载中间件（webpack-hot-middleware）**
- **挂载代理服务和中间件**
- **配置静态资源**
- **启动服务器监听特定端口（8080）**
- 自动打开浏览器并打开特定网址（localhost:8080）

express 服务器提供静态文件服务，还使用 http 请求代理中间件（http-proxy-middleware）

```js
'use strict'
// 检查 Node.js 和 npm 的版本
require('./check-versions')()

// 获取基本配置
const config = require('../config')
// 如果 Node 的环境变量中没有设置当前的环境（NODE_ENV），则使用 config 中的 dev 环境配置作为当前环境
if (!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}

// opn 是一个可以调用软件打开网址、图片、文件等内容的插件
// 这里用它来调用默认浏览器打开 dev-server 监听的端口，如 localhost:8080
const opn = require('opn')
const path = require('path')
const express = require('express')
const webpack = require('webpack')
// http-proxy-middleware 是一个将 http 请求代理到其他服务器的 express 中间件
// 如 localhost:8080/api/test -> localhost:3000/api/test
// 这里使用该插件可以将前端开发中涉及到的请求代理到提供服务的后台服务上
const proxyMiddleware = require('http-proxy-middleware')
// 开发环境下的 webpack 配置
const webpackConfig = (process.env.NODE_ENV === 'testing' || process.env.NODE_ENV === 'production')
  ? require('./webpack.prod.conf')
  : require('./webpack.dev.conf')

// default port where dev server listens for incoming traffic
// dev-server 监听的端口，如果没有在命令行传入端口号，则使用 config.dev.port 设置的端口
const port = process.env.PORT || config.dev.port
// automatically open browser, if not set will be false
// autoOpenBrowser 用于判断是否自定打开浏览器的布尔变量，当配置文件中没有设置自动打开浏览器时值为 false
const autoOpenBrowser = !!config.dev.autoOpenBrowser
// Define HTTP proxies to your custom API backend
// https://github.com/chimurai/http-proxy-middleware
// http 代理表，指定规则，将某些 api 请求代理到相应服务器
const proxyTable = config.dev.proxyTable

// 创建 express 服务器
const app = express()
// webpack 根据配置开始编译打包源码并返回 compiler 对象
const compiler = webpack(webpackConfig)

// webpack-dev-middleware 将 webpack 编译打包后得到的文件存在在内存中
// 将该中间件挂载 express 上使用之后即可得到提供这些编译后的文件服务
const devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath, // 设置访问路径为 webpack 配置中的 output 里对应的路径
  quiet: true // 设置为 true，使其不要在控制台输出日志
})

// webpack-hot-middleware 用于热重载功能的中间件
const hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: false, // 关闭控制台的日志输出
  heartbeat: 2000 // 发送心跳包的频率
})
// force page reload when html-webpack-plugin template changes
// currently disabled until this is resolved:
// https://github.com/jantimon/html-webpack-plugin/issues/680
// webpack (重新) 编译打包后将 js css 等文件 inject 到 html 文件后，通过热重载中间件强制刷新页面
// compiler.plugin('compilation', function (compilation) {
//   compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
//     hotMiddleware.publish({ action: 'reload' })
//     cb()
//   })
// })

// enable hot-reload and state-preserving
// compilation error display
// 挂载热重载中间件
app.use(hotMiddleware)

// proxy api requests
// 根据 proxyTable 中的代理请求配置来设置 express 服务器的 http 代理规则
Object.keys(proxyTable).forEach(function (context) {
  let options = proxyTable[context]
	// 格式化 options，如将 'www.example.com' 变成 {target: 'www.example.com'}
  if (typeof options === 'string') {
    options = { target: options }
  }
  app.use(proxyMiddleware(options.filter || context, options))
})

// handle fallback for HTML5 history API
// 重定向不存在的 URL，用于支持 SPA（单页应用）
// 如使用 vue-router 并开启 history 模式
app.use(require('connect-history-api-fallback')())

// serve webpack bundle output
// 挂载 webpack-dev-middleware 中间件，提供 webpack 编译打包后的文件服务
app.use(devMiddleware)

// serve pure static assets
// 提供 static 文件夹上的静态文件服务
const staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)
app.use(staticPath, express.static('./static'))

// 访问链接
const uri = 'http://localhost:' + port

// 创建 promise，在应用服务启动后 resolve
// 便于外部文件 require 这个 dev-server 后的代码编写
var _resolve
var _reject
var readyPromise = new Promise((resolve, reject) => {
  _resolve = resolve
  _reject = reject
})

var server
var portfinder = require('portfinder')
portfinder.basePort = port

console.log('> Starting dev server...')
// webpack-dev-middleware 等待 webpack 编译打包完成后输出语句到控制台
// 服务正式启动才自动打开浏览器进入页面
devMiddleware.waitUntilValid(() => {
  portfinder.getPort((err, port) => {
    if (err) {
      _reject(err)
    }
    process.env.PORT = port
    var uri = 'http://localhost:' + port
    console.log('> Listening at ' + uri + '\n')
    // when env is testing, don't need open it
    if (autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
      opn(uri)
    }
		// 启动 express 服务器并监听相应端口
    server = app.listen(port)
    _resolve()
  })
})

// 暴露本模块的功能给外部使用
// 如 var devServer = require('./build/dev-server')
// devServer.ready.then(() => {...})
// if (...) {devServer.close()}
module.exports = {
  ready: readyPromise,
  close: () => {
    server.close()
  }
}
```

#### build/webpack.base.conf.js

从 `build/dev-server.js` 代码中，dev-server 使用的 webpack 配置来自 `build/webpack.dev.conf.js` 文件（生产环境下使用的是 `build/webpack.prod.conf.js`），而 `build/webpack.dev.conf.js` 中又引用了 `build/webpack.base.conf.js`

> `webpack.base.conf.js` 主要完成下面这些事情：
- **配置 webpack 编译入口**
- **配置 webpack 输出路径和命名规则**
- **配置 模块 resolve 规则**
- **配置不同类型模块的处理规则**

这个配置值配置了 `.js`、 `.vue` 、图片、字体等几类文件的处理规则，如果需要处理其他文件可以在 `module.rules` 里另行配置

```js
'use strict'
const path = require('path')
const utils = require('./utils')
const config = require('../config')
const vueLoaderConfig = require('./vue-loader.conf')

// 获取绝对路径
function resolve (dir) {
  return path.join(__dirname, '..', dir)
}

module.exports = {
	// webpack 入口文件
  entry: {
    app: './src/main.js'
  },
	// webpack 输出路径和命名规则
  output: {
		// webpack 输出的目标文件夹路径，如 ./dist
    path: config.build.assetsRoot,
		// webpack 输出 bundle 文件命名格式
    filename: '[name].js',
		// webpack 编译输出的发布路径，如 //cdn.xxx.com/app/
    publicPath: process.env.NODE_ENV === 'production'
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
	// 模块 resolve 规则
  resolve: {
    extensions: ['.js', '.vue', '.json'],
		// 别名，方便引用模块
    alias: {
			// import Vue from 'vue' 相当于 import Vue from 'vue/dist/vue.esm.js'
			// 但是对于 import 'vue/xxx' 不匹配
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'),
    }
  },
	// 不同类型模块的处理规则
  module: {
    rules: [
      {// 对 src 和 test 文件夹下的 .js .vue 文件使用 eslint-loader 进行代码规范检查
        test: /\.(js|vue)$/,
        loader: 'eslint-loader',
        enforce: 'pre',
        include: [resolve('src'), resolve('test')],
        options: {
          formatter: require('eslint-friendly-formatter')
        }
      },
      {// 对全部 .vue 文件使用 vue-loader 进行编译
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      {// 对 src 和 test 文件夹下的 .js 文件使用 babel-loader 将 es6+ 代码转为 es5
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test')]
      },
      {// 对图片资源文件使用 url-loader
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
					// 小于 10k 图片转成 base64 编码的 dataURl 字符串协写到代码中
          limit: 10000,
					// 其他图片转移到静态资源文件夹中
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {// 对于多媒体资源文件使用 url-loader
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
					// 小于 10k 资源转成 base64 编码的 dataURl 字符串协写到代码中
          limit: 10000,
					// 其他资源转移到静态资源文件夹中
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {// 对于字体文件使用 url-loader
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
					// 小于 10k 资源转成 base64 编码的 dataURl 字符串协写到代码中
          limit: 10000,
					// 其他资源转移到静态资源文件夹中
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  }
}
```

#### build/webpack.dev.conf.js

> 在 `webpack.base.conf.js` 的基础上，`webpack.dev.conf.js` 上增加了一些开发环境的配置：
- **将 webpack 的热宠在客户端代码添加到每个 entry 对应的应用**
- **合并基础的 webpack 配置**
- **配置样式文件的处理规则，styleLoader...**
- **配置 Source Maps**
- **配置 webpack 插件**

```js
'use strict'
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
// webpack-merge 是一个可以合并数组和对象的插件
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf')
// html-webpack-plugin 用于将 webpack 编译打包后的文件 注入到 html 模板中
// 即自动在 index.html 中加上 <link> <script> 标签引用 webpack 打包后的文件
const HtmlWebpackPlugin = require('html-webpack-plugin')
// friendly-errors-webpack-plugin 用于更友好输出 webpack 的警告、错误等信息
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')

// add hot-reload related code to entry chunks
// 每个入口页面（应用）加上 dev-client，用于和 dev-server 的热更新插件通信，实现热更新
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})

module.exports = merge(baseWebpackConfig, {
  module: {
		// 样式文件的处理规则，对 css/less/sass/scss 等不同内容使用相应的 styleLoaders
		// 由 utils 配置出各种类型的预处理语言需要使用的 loader，如 less 需要使用 less-loader css-loader style-loader
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })
  },
  // cheap-module-eval-source-map is faster for development
	// cheap-module-eval-source-map 在开发环境下更快
  devtool: '#cheap-module-eval-source-map',
	// webpack 插件
  plugins: [
		// webpack.DefinePlugin 该插件暴露变量给全局
    new webpack.DefinePlugin({
      'process.env': config.dev.env
    }),
    // https://github.com/glenjamin/webpack-hot-middleware#installation--usage
		// 开启 webpack 热更新
    new webpack.HotModuleReplacementPlugin(),
		// webpack 编译阶段出错跳过出错阶段，不会阻塞编译，在编译结束后报错
    new webpack.NoEmitOnErrorsPlugin(),
    // https://github.com/ampedandwired/html-webpack-plugin
		// 自动将依赖注入 html 模板，并输出最终的 html 文件到目标文件夹
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    new FriendlyErrorsPlugin()
  ]
})
```

#### build/utils.js

> utils 提供工具函数，包括生成处理各种样式语言的 loader，获取资源文件存放路径的工具函数
- 计算资源文件存放路径
- **生成 cssLoaders 用于加载 .vue 文件中的样式**
- **生成 styleLoaders 用于加载不在 .vue 文件中单独存在的样式文件**

```js
'use strict'
const path = require('path')
const config = require('../config')
// extract-text-webpack-plugin 用来提取 bundle 中特定文本，将提取后的文本单独存在到另外的文件中
// 这里用来提取 css
const ExtractTextPlugin = require('extract-text-webpack-plugin')

// 资源存放的路径
exports.assetsPath = function (_path) {
  const assetsSubDirectory = process.env.NODE_ENV === 'production'
    ? config.build.assetsSubDirectory
    : config.dev.assetsSubDirectory
  return path.posix.join(assetsSubDirectory, _path)
}

// 生成 css sass scss less 等各种用来编写样式的语言对应的 loader 配置
exports.cssLoaders = function (options) {
  options = options || {}

	// css-loader 配置
  const cssLoader = {
    loader: 'css-loader',
    options: {
			// 是否最小化
      minimize: process.env.NODE_ENV === 'production',
			// 是否使用 source-map
      sourceMap: options.sourceMap
    }
  }

  // generate loader string to be used with extract text plugin
	// 生成各种 loader 配置，并且配置了 extract-text-webpack-plugin
  function generateLoaders (loader, loaderOptions) {
		// 默认是 css-loader
    const loaders = [cssLoader]
		// 如果非 css，则增加一个预编译语言的 loader 并设置相关配置属性
		// 如 generateLoaders('less')，会 push 一个 less-loader，less-loader 先将 less 编译成 css，然后再由 css-loader 处理 css
		// 其他 sass scss 等也是一样
    if (loader) {
      loaders.push({
        loader: loader + '-loader',
        options: Object.assign({}, loaderOptions, {
          sourceMap: options.sourceMap
        })
      })
    }

    // Extract CSS when that option is specified
    // (which is the case during production build)
    if (options.extract) {
			// 配置 extract-text-webpack-plugin 提取样式
      return ExtractTextPlugin.extract({
        use: loaders,
        fallback: 'vue-style-loader'
      })
    } else {
			// 无需提取样式则使用 vue-style-loader 配合各种 loader 处理 <style> 里的 样式
      return ['vue-style-loader'].concat(loaders)
    }
  }

  // https://vue-loader.vuejs.org/en/configurations/extract-css.html
	// 得到不同处理样式语言对应的 loader
  return {
    css: generateLoaders(),
    postcss: generateLoaders(),
    less: generateLoaders('less'),
    sass: generateLoaders('sass', { indentedSyntax: true }),
    scss: generateLoaders('sass'),
    stylus: generateLoaders('stylus'),
    styl: generateLoaders('stylus')
  }
}

// Generate loaders for standalone style files (outside of .vue)
// 处理单独的 .css .sass .scss .less 等样式文件规则
// 如 
// {test: '/.\less$/', use: ['vue-style-loader', 'css-loader','less-loader']}
exports.styleLoaders = function (options) {
  const output = []
  const loaders = exports.cssLoaders(options)
  for (const extension in loaders) {
    const loader = loaders[extension]
    output.push({
      test: new RegExp('\\.' + extension + '$'),
      use: loader
    })
  }
  return output
}
```

#### build/vue-loader.conf.js

```js
'use strict'
const utils = require('./utils')
const config = require('../config')
const isProduction = process.env.NODE_ENV === 'production'

module.exports = {
	// 处理 .vue 文件中的样式
  loaders: utils.cssLoaders({
		// 是否打开 source-map
    sourceMap: isProduction
      ? config.build.productionSourceMap
      : config.dev.cssSourceMap,
		// 是否提取样式到单独的文件
    extract: isProduction
  }),
  transformToRequire: {
    video: 'src',
    source: 'src',
    img: 'src',
    image: 'xlink:href'
  }
}
```

#### build/dev-client.js

`build/dev-client.js` 主要写了浏览器端代码，用于实现 webpack 热更新

```js
/* eslint-disable */
'use strict'
// 实现浏览器端的 EventSource 用于和服务器端双向通信
// webpack 热重载客户端跟 dev-server 上的热重载插件之间需要进行双工通信
require('eventsource-polyfill')
// webpack 热重载客户端
var hotClient = require('webpack-hot-middleware/client?noInfo=true&reload=true')

// 客户端收到更新操作，执行页面刷新
hotClient.subscribe(function (event) {
  if (event.action === 'reload') {
    window.location.reload()
  }
})
```

#### build/build.js

分析完开发环境下的配置，看一下构建环境下的配置

> `build/build.js` 主要完成下面几件事情：
- loading 动画
- **删除目标文件夹 如 ./dist**
- **执行 webpack 构建**
- 输出信息

```js
'use strict'
// 检查 Node.js 和 npm 的版本
require('./check-versions')()

process.env.NODE_ENV = 'production'

// 一个可以在终端显示 spinner 的插件
const ora = require('ora')
// rm 用于删除文件和文件夹的插件
const rm = require('rimraf')
const path = require('path')
// chalk 用于在控制台输出带颜色字体的插件
const chalk = require('chalk')
const webpack = require('webpack')
const config = require('../config')
// 生产环境下 webpack 配置
const webpackConfig = require('./webpack.prod.conf')

const spinner = ora('building for production...')
spinner.start() // 开启 loading 动画

// 首先将整个 dist 文件夹和其内容删除
// 删除完成才开始 webpack 的构建打包
rm(path.join(config.build.assetsRoot, config.build.assetsSubDirectory), err => {
  if (err) throw err
  webpack(webpackConfig, function (err, stats) {
    spinner.stop()
    if (err) throw err
    process.stdout.write(stats.toString({
      colors: true,
      modules: false,
      children: false,
      chunks: false,
      chunkModules: false
    }) + '\n\n')

    if (stats.hasErrors()) {
      console.log(chalk.red('  Build failed with errors.\n'))
      process.exit(1)
    }

    console.log(chalk.cyan('  Build complete.\n'))
    console.log(chalk.yellow(
      '  Tip: built files are meant to be served over an HTTP server.\n' +
      '  Opening index.html over file:// won\'t work.\n'
    ))
  })
})
```

#### build/webpack.prod.conf.js

> 生产环境构建的 webpack 配置来自 `build/webpack.prod.conf.js`, 同样是在 `build/webpack.base.conf.js` 基础上增加了一些生产构建的配置:
- **合并基础的 webpack 配置**
- **配置样式文件的处理规则， styleLoaders**
- **配置 webpack 的输出**
- **配置 webpack 插件**
- **gzip 模式下 webpack 插件配置**
- **webpack-bundle 分析**

```js
'use strict'
const path = require('path')
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf')
// copu-webpack-plugin 插件用于将 static 中的静态文件复制到目标文件夹 dist
const CopyWebpackPlugin = require('copy-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const ExtractTextPlugin = require('extract-text-webpack-plugin')
// optimize-css-assets-webpack-plugin 插件用于优化和最小化 css 资源
const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')

const env = process.env.NODE_ENV === 'testing'
  ? require('../config/test.env')
  : config.build.env

const webpackConfig = merge(baseWebpackConfig, {
  module: {
		// 样式文件的处理规则，对 css less sass scss 等不同内容使用相应的 styleLoaders
		// 由 utils 配置出各种类型的预处理语言所需要的 loader
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
	// 是否是使用 source-map
  devtool: config.build.productionSourceMap ? '#source-map' : false,
	// webpack 输出路径和命名规则
  output: {
    path: config.build.assetsRoot,
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
	// webpack 插件
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    new webpack.DefinePlugin({
      'process.env': env
    }),
    // UglifyJs do not support ES6+, you can also use babel-minify for better treeshaking: https://github.com/babel/minify
		// 压缩 js 代码
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    // extract css into its own file
		// 将 css 提取到单独的文件
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    // Compress extracted CSS. We are using this plugin so that possible
    // duplicated CSS from different components can be deduped.
		// 优化最小化 css 代码，如果只是简单实用 extract-text-webpack-plugin 可能会造成 css 重复
    new OptimizeCSSPlugin({
      cssProcessorOptions: {
        safe: true
      }
    }),
    // generate dist index.html with correct asset hash for caching.
    // you can customize output by editing /index.html
    // see https://github.com/ampedandwired/html-webpack-plugin
		// 将打包 文件注入到 index.html
    new HtmlWebpackPlugin({
      filename: process.env.NODE_ENV === 'testing'
        ? 'index.html'
        : config.build.index,
      template: 'index.html',
      inject: true,
      minify: {
				// 删除 index.html 的注释
        removeComments: true,
				// 删除 index.html 的 空格
        collapseWhitespace: true,
				// 删除各种 html 标签属性值的双引号
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
			// 注入依赖时按照依赖的先后顺序进行注入，如先注入 manifest.js，然后 vendor.js，最后 app.js
      chunksSortMode: 'dependency'
    }),
    // keep module.id stable when vender modules does not change
    new webpack.HashedModuleIdsPlugin(),
    // split vendor js into its own file
		// 将所有从 node_modules 中引入 js 提取到 vendor.js，即抽取第三方库文件
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
		// 从 wendor 中提取 manifest，也就是 webpack 运行时代码，防止新引入第三方库文件导致 vendor hash 变化
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    }),
    // copy custom static assets
		// 将 static 文件夹的静态资源复制到 `dist/static`
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})

// 如果开始 gzip 压缩，则利用插件将构建后的文件进行压缩
if (config.build.productionGzip) {
	// compression-webpack-plugin 用于压缩 webpack 的插件
  const CompressionWebpackPlugin = require('compression-webpack-plugin')

  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
			// 压缩算法
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}

// 如果启动了 report，则通过插件给出 webpack 构建打包的文件分析报告
if (config.build.bundleAnalyzerReport) {
  const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}

module.exports = webpackConfig
```

#### build/check-version.js

`build/check-version.js` 文件完成对 Node.js 和 npm 的版本检查

```js
'use strict'
// chalk 插件用于在控制台输出带颜色字体
const chalk = require('chalk')
// semver 用于语法化版本检查插件
const semver = require('semver')
const packageConfig = require('../package.json')
// shelljs 执行 unix 命令行的插件
const shell = require('shelljs')
// 开辟子进程执行指令 cmd 并返回结果
function exec (cmd) {
  return require('child_process').execSync(cmd).toString().trim()
}

// Node 和 npm 版本要求
const versionRequirements = [
  {
    name: 'node',
    currentVersion: semver.clean(process.version),
    versionRequirement: packageConfig.engines.node
  }
]

if (shell.which('npm')) {
  versionRequirements.push({
    name: 'npm',
    currentVersion: exec('npm --version'),
    versionRequirement: packageConfig.engines.npm
  })
}

module.exports = function () {
  const warnings = []
	// 依次判断版本是否符合要求
  for (let i = 0; i < versionRequirements.length; i++) {
    const mod = versionRequirements[i]
    if (!semver.satisfies(mod.currentVersion, mod.versionRequirement)) {
      warnings.push(mod.name + ': ' +
        chalk.red(mod.currentVersion) + ' should be ' +
        chalk.green(mod.versionRequirement)
      )
    }
  }

	// 若有警告会输出到控制台
  if (warnings.length) {
    console.log('')
    console.log(chalk.yellow('To use this template, you must update following to modules:'))
    console.log()
    for (let i = 0; i < warnings.length; i++) {
      const warning = warnings[i]
      console.log('  ' + warning)
    }
    console.log()
    process.exit(1)
  }
}
```

### config 文件夹

#### config/index.js

config 文件夹下的 index.js 定义了许多开发和生产环境下的配置

```js

'use strict'
// Template version: 1.1.3
// see http://vuejs-templates.github.io/webpack for documentation.

const path = require('path')

module.exports = {
	// 构建使用的配置
  build: {
		// 环境变量
    env: require('./prod.env'),
		// html 入口文件
    index: path.resolve(__dirname, '../dist/index.html'),
		// 构建文件的存放路径
    assetsRoot: path.resolve(__dirname, '../dist'),
		// 二级目录，存放静态资源文件的目录，位于 dist 文件夹下
    assetsSubDirectory: 'static',
		// 发布路径，如果构建后的文件用于发布 cdn 或者放到其他域名的服务器，可在此进行设置
		// 设置后构建的文件在注入 index.html 中会带上这里的发布路径
    assetsPublicPath: '/',
		// 是否使用 source-map
    productionSourceMap: true,
    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    // 是否开启 gzip 压缩
		productionGzip: false,
		// gzip 模式下需要压缩的文件的扩展名，设置 css、js 之后只会对 js 和 css 文件进行压缩
    productionGzipExtensions: ['js', 'css'],
    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
		// 是否展示 webpack 构建打包之后的分析报告
    bundleAnalyzerReport: process.env.npm_config_report
  },
	// 开发使用的配置
  dev: {
		// 环境变量
    env: require('./dev.env'),
		// dev-server 监听的端口
    port: process.env.PORT || 8080,
		// 是否自动打开浏览器
    autoOpenBrowser: true,
		// 静态资源文件夹
    assetsSubDirectory: 'static',
		// 发布路径
    assetsPublicPath: '/',
		// 代理配置表，可配置特定请求代理到对应的 api 接口
    proxyTable: {},
    // CSS Sourcemaps off by default because relative paths are "buggy"
    // with this option, according to the CSS-Loader README
    // (https://github.com/webpack/css-loader#sourcemaps)
    // In our experience, they generally work as expected,
    // just be aware of this issue when enabling this option.
		// 是否开启 cssSourceMap
    cssSourceMap: false
  }
}
```

### config/dev.env.js config/prod.env.js config/test.env.js

这三个文件就是设置了环境变量

```js
// config/dev.env.js
'use strict'
const merge = require('webpack-merge')
const prodEnv = require('./prod.env')

module.exports = merge(prodEnv, {
  NODE_ENV: '"development"'
})

// config/prod.env.js
'use strict'
module.exports = {
  NODE_ENV: '"production"'
}

// config/test.env.js
'use strict'
const merge = require('webpack-merge')
const devEnv = require('./dev.env')

module.exports = merge(devEnv, {
  NODE_ENV: '"testing"'
})
```

## webpack#1.2.0

webpack#1.2.0 版本开始，webpack 配置不在使用自定义的服务器，而是引入 webpack-dev-server

### 文件目录

### 文件结构

可以看到文件结构变化不大，删除了 `dev-server` `dev-client` 文件

```tree
├── build
│   ├── build.js
│   ├── check-versions.js
│   ├── utils.js
│   ├── vue-loader.conf.js
│   ├── webpack.base.conf.js
│   ├── webpack.dev.conf.js
│   ├── webpack.prod.conf.js
│   └── webpack.test.conf.js
├── config
│   ├── dev.env.js
│   ├── index.js
│   ├── prod.env.js
│   └── test.env.js
├── ...
└── package.json
```

### 指令

`package.json` 中的 `script` 字段定义了项目的运行脚本。指令主要是 `npm run dev` 起了 webpack-dev-serve 运行开发环境的配置

```js
"scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "unit": "jest test/unit/specs --coverage",
    "e2e": "node test/e2e/runner.js",
    "test": "npm run unit && npm run e2e",
    "lint": "eslint --ext .js,.vue src test/unit/specs test/e2e/specs",
    "build": "node build/build.js"
  },
```

### build/webpack.dev.conf.js

重点分析 `build/webpack.dev.conf.js` 增加的 `devServer` 配置

```js
'use strict'
// ...
const devWebpackConfig = merge(baseWebpackConfig, {
  // ...
  // these devServer options should be customized in /config/index.js
  devServer: {
    // 重定向不存在的 URL，用于支持 SPA（单页应用）
    historyApiFallback: true,
    // 开启热更新
    hot: true,
    // 设置服务器
    host: process.env.HOST || config.dev.host,
    // 设置端口
    port: process.env.PORT || config.dev.port,
    // 是否自动开启浏览器
    open: config.dev.autoOpenBrowser,
    overlay: config.dev.errorOverlay ? {
      warnings: false,
      errors: true,
    } : false,
    // 发布路径
    publicPath: config.dev.assetsPublicPath,
    // 代理路由表
    proxy: config.dev.proxyTable,
    quiet: true, // necessary for FriendlyErrorsPlugin
    watchOptions: {
      poll: config.dev.poll,
    }
  },
  // ...
})
// ...
```
