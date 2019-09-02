---

title: vue 服务端渲染
tag: 
	- vue ssr
---

> 参考：[vue ssr 官网](https://ssr.vuejs.org/zh/guide/)

## 原理

### 服务器端渲染整体

![24cb61b30fd3554b408c5e4c83704408_meitu_1](/Users/jack/Desktop/ssr/24cb61b30fd3554b408c5e4c83704408_meitu_1.png)

### 服务端渲染不同环境

开发环境

![屏幕快照 2019-08-15 下午5.04.16](/Users/jack/Desktop/ssr/屏幕快照 2019-08-15 下午5.04.16.png)

生产环境

## ![](/Users/jack/Desktop/ssr/屏幕快照 2019-08-15 下午5.04.48.png)

## 实践

服务端渲染 ssr 最终要让一份代码既可以在服务端运行，也可以在客户端运行。需要有两个 `webpack` 入口文件，一个用于浏览器端渲染 `webpack.config.client.js`, 一个用于服务器端渲染 `webpack.config.server.js`, 将它们的公有部分抽取作为 `webpack.config.base.js`, 之后通过` webpack-merge` 合并。同时需要一个 server 提供 http 服务，这里用的 koa

```bash
# 简化的目录
- build // 打包
    - webpack.config.base..js
    - webpack.config.client.js
    - webpack.config.server.js
- src
    - 
    - app.vue
    - index.js
    - client-entry.js   
    - server-entry.js   
    - create-app.js   
- server
		- index.js
```

### 应用实例

在纯客户端应用中，每个用户会在各自浏览器中使用新的应用程序实例。对于服务端渲染，每个请求都应该是全新独立的应用程序实例，以便不会有交叉请求造成的状态污染。因此需要 create-app.js 中对 app.js 修改，包装成一个工厂函数，每次调用会返回一个全新根组件

```js
import Vue from 'vue'
import App from './app.vue'
import createRouter from './router'
import createStore from './store'

import './styles/global.scss'

// 服务端每次请求都会渲染返回一个新的 app router store 实例，避免状态污染
export default () => {
  const router = createRouter()
  const store = createStore()

  const app = new Vue({
    router,
    store,
    render: h => h(App)
  })

  return { app, router, store }
}
```

在浏览器端，直接新建一个根组件将其挂载就能激活

`client-entry.js`

```js
import createApp from './create-app'

const { app } = createApp()

// 服务端渲染好的应用 根元素 添加了一个特殊的属性 <div id="app" data-server-rendered="true">，这让客户端 vue 知道这部分 html 是 Vue 在服务端渲染的，需要用激活模式挂载
// app.$mount('#root') 激活客户端脚本
// 对于没有 data-server-rendered 属性的元素上，还可以向 $mount 函数的 hydrating 参数位置传入 true，来强制使用激活模式
// app.$mount('#root', true)
app.$mount('#root')
```

在服务器端需要返回一个函数，该函数作用是接受一个 `context` 参数，同时每次都返回一个新的根组件

```js
import createApp from './create-app'

// 这里 context 也就是服务端渲染 renderToString 传入 context
export default context => {
  return new Promise((resolve, reject) => {
    const { app, router, store } = createApp()

    // 服务端主动调用
    router.push(context.url)
    // 路由记录被推进去后，所有异步操作完成（如加载异步组件）
    router.onReady(() => {
      const matchedComponents = router.getMatchedComponents()

      if (!matchedComponents.length) {
        return reject(new Error(`no component matched`))
      }
			resolve(app)
    })
  })
}
```

### webpack 配置

#### wepack.config.base.js

```js
const path = require('path')
const createVueLoaderOptions = require('../vue-loader.config')
const isDev = process.env.NODE_ENV === 'development'
const config = {
  target: 'web',
  entry: path.join(__dirname, '../src/client-entry.js'),
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, '../dist'),
    // 开发环境直接将静态文件的 publicPath 指定全
    publicPath: 'http://0.0.0.0:8000/dist/'
  },
  module: {
    rules: [
      {
        test: /\.(vue|js|jsx)$/,
        loader: 'eslint-loader',
        exclude: /node_modules/,
        enforce: 'pre'
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: createVueLoaderOptions(isDev)
      },
      {
        test: /\.jsx$/,
        loader: 'babel-loader'
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        exclude: /node_modules/
      },
      {
        test: /\.(gif|jpg|jpeg|png|svg)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 1024,
              name: 'resources/[path][name].[hash:8].[ext]'
            }
          }
        ]
      }
    ]
  }
}

module.exports = config
```

#### webpack.config.client.js

注意到使用 `vue-server-renderer/client-plugin`  插件，最终会打包出 `vue-ssr-client-manifest.json`， 最后会被服务端 inject 到 html 中

```js
const path = require('path')
const HTMLPlugin = require('html-webpack-plugin')
const webpack = require('webpack')
const merge = require('webpack-merge')
const ExtractPlugin = require('extract-text-webpack-plugin')
const baseConfig = require('./webpack.config.base')

const VueClientPlugin = require('vue-server-renderer/client-plugin')

const isDev = process.env.NODE_ENV === 'development'

const defaultPluins = [
  new webpack.DefinePlugin({
    'process.env': {
      NODE_ENV: isDev ? '"development"' : '"production"'
    }
  }),
  new HTMLPlugin({
    template: path.join(__dirname, '../template.html')
  }),
  // 此插件在输出目录中生成 `vue-ssr-client-manifest.json`。
  new VueClientPlugin()
]

const devServer = {
  port: 8000,
  host: '0.0.0.0',
  overlay: {
    errors: true
  },
  headers: { 'Access-Control-Allow-Origin': '*' },
  historyApiFallback: {
    index: '/dist/index.html'
  },
  proxy: {
    '/api': 'http://127.0.0.1:3000',
    '/user': 'http://127.0.0.1:3000'
  },
  hot: true
}

let config

if (isDev) {
  config = merge(baseConfig, {
    devtool: '#cheap-module-eval-source-map',
    module: {
      rules: [
        {
          test: /\.scss/,
          use: [
            'vue-style-loader',
            'css-loader',
            {
              loader: 'postcss-loader',
              options: {
                sourceMap: true
              }
            },
            'sass-loader'
          ]
        }
      ]
    },
    devServer,
    plugins: defaultPluins.concat([
      new webpack.HotModuleReplacementPlugin(),
      new webpack.NoEmitOnErrorsPlugin()
    ])
  })
} else {
  // 生产环境 css 提取到单独文件
  config = merge(baseConfig, {
    entry: {
      app: path.join(__dirname, '../src/client-entry.js'),
      vendor: ['vue']
    },
    output: {
      filename: '[name].[chunkhash:8].js',
      publicPath: '/dist/'
    },
    module: {
      rules: [
        {
          test: /\.scss/,
          use: ExtractPlugin.extract({
            fallback: 'vue-style-loader',
            use: [
              'css-loader',
              {
                loader: 'postcss-loader',
                options: {
                  sourceMap: true
                }
              },
              'sass-loader'
            ]
          })
        }
      ]
    },
    plugins: defaultPluins.concat([
      new ExtractPlugin('styles.[contentHash:8].css')
    ])
  })
}

module.exports = config
```

#### webpack.config.server.js

注意使用 `vue-server-renderer/server-plugin'` 插件，会输出 `vue-ssr-server-bundle.json`

```js
const path = require('path')
const ExtractPlugin = require('extract-text-webpack-plugin')
const webpack = require('webpack')
const merge = require('webpack-merge')
const baseConfig = require('./webpack.config.base')

const VueServerPlugin = require('vue-server-renderer/server-plugin')

let config

config = merge(baseConfig, {
  // 这允许 webpack 以 Node 适用方式(Node-appropriate fashion)处理动态导入(dynamic import)，
  // 并且还会在编译 Vue 组件时，告知 `vue-loader` 输送面向服务器代码(server-oriented code)
  target: 'node',
  // entry 指向应用的 server-entry 文件
  entry: path.join(__dirname, '../src/server-entry.js'),
  // 对 bundle renderer 提供 source map 支持
  devtool: 'source-map',
  output: {
    // 此处告知 server bundle 使用 Node 风格导出模块(Node-style exports)
    libraryTarget: 'commonjs2',
    filename: 'server-entry.js', // 每个输出 bundle 名称
    path: path.join(__dirname, '../server-build') // 输出目录绝对地址
  },
  // 外置化应用程序依赖模块。可以使服务器构建速度更快，并生成较小的 bundle 文件
  externals: Object.keys(require('../package.json').dependencies),
  module: {
    rules: [
      {
        test: /\.scss/,
        use: ExtractPlugin.extract({
          fallback: 'vue-style-loader',
          use: [
            'css-loader',
            {
              loader: 'postcss-loader',
              options: {
                sourceMap: true
              }
            },
            'sass-loader'
          ]
        })
      }
    ]
  },
  plugins: [
    new ExtractPlugin('styles.[contentHash:8].css'),
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
      'process.env.VUE_ENV': '"server"'
    }),
    // 这是将服务器的整个输出
    // 构建为单个 JSON 文件的插件
    // 默认文件名为 `vue-ssr-server-bundle.json`
    new VueServerPlugin()
    // 可指定 filename
    // new VueServerPlugin({
    //   filename: 'xxxx'
    // })
  ]
})

module.exports = config
```

### 服务器

#### 入口文件

这里重点在要区分不同环境产出 html

```js
const Koa = require('koa')
const app = new Koa()
const bodyparser = require('koa-body')
const apiRouter = require('./router/api')
const staticRouter = require('./router/static')

const isDev = process.env.NODE_ENV === 'development'

// 错误中间件
app.use(async (ctx, next) => {
  try {
    console.log(`request with path ${ctx.path}`)
    await next()
  } catch (error) {
    console.log(error)
    ctx.status = 500
    if (isDev) {
      ctx.body = error.message
    } else {
      ctx.body = 'please try again later!'
    }
  }
})

// 解析post数据并注册路由
app.use(bodyparser())
// 注册 api 接口路由
app.use(apiRouter.routes()).use(apiRouter.allowedMethods())
// 注册静态文件目录
app.use(staticRouter.routes()).use(staticRouter.allowedMethods())

// 根据不同环境产出返回不同 html 页面
const pageRouter = isDev ? require('./router/dev-ssr') : require('./router/ssr')
app.use(pageRouter.routes()).use(pageRouter.allowedMethods())

app.listen(3000, () => console.log('api服务已启动'))
```

#### ssr

```js
const Router = require('koa-router')
const path = require('path')
const fs = require('fs')
const VueServerRender = require('vue-server-renderer')

const serverRender = require('../server-render')

const clientManifest = require('../../dist/vue-ssr-client-manifest.json')

const renderer = VueServerRender
  .createBundleRenderer(
    path.join(__dirname, '../../server-build/vue-ssr-server-bundle.json'),
    {
      inject: false,
      clientManifest
    }
  )

const template = fs.readFileSync(
  path.join(__dirname, '../server-template.ejs'),
  'utf-8'
)

const router = new Router()
router.get('*', async (ctx, next) => {
  await serverRender(ctx, renderer, template)
})

module.exports = router
```

生产环境下客户端清单 `vue-ssr-client-manifest.json` 和 服务端 bundle 文件 `vue-ssr-server-bundle.json`  都已生成，只要调用 `vue-server-renderer`  插件的 `createBundleRenderer(serverBundle, {/* 选项 */})`  得到一个 `renderer`  渲染器，如果传入模板 template， 就能按照官方规则，调用 `renderToString`方法将根据 服务端 bundle 文件 和 模板 template 生成 html 字符串，同时会把客户端 manifest 注入到 html 对应位置（如 style script 等）

```js
const bundle = fs.readFileSync(path.resolve(__dirname, '../../server-build/vue-ssr-server-bundle.json'))
const renderer = require('vue-server-renderer').createBundleRenderer(bundle, {
  template,
  clientManifest: require('../../dist/vue-ssr-client-manifest.json')
})

renderer.renderToString((err, html) => {
    if (err) {
      console.error(err)
      ctx.status = 500
      ctx.body = '服务器内部错误'
    } else {
      ctx.headers['Content-Type'] = 'text/html'
      ctx.body = html
    }
})
```

如果不想按照官方注册规则，生成 renderer 传入 `inject: false` 参数

```js
const renderer = VueServerRender
  .createBundleRenderer(
    path.join(__dirname, '../../server-build/vue-ssr-server-bundle.json'),
    {
      inject: false,
      clientManifest
    }
  )
```

这时 renderToString 回调函数中，传入 context 参数， 最终返回渲染字符串， 传入的 context 对象最终会暴露 [如下方法](https://ssr.vuejs.org/zh/guide/build-config.html#客户端配置-client-config)，方便用户调用用于自定义渲染

```js
const ejs = require('ejs')
module.exports = async (ctx, renderer, template) => {
  ctx.headers['Content-Type'] = 'text/html'
  const context = { url: ctx.path }
  try {
    // renderToString 回调函数中，传入的 context 对象会暴露如下方法
    // https://ssr.vuejs.org/zh/guide/build-config.html#客户端配置-client-config
    const appString = await renderer.renderToString(context)

    const html = ejs.render(template, {
      appString,
      styles: context.renderStyles(),
      scripts: context.renderScripts()
    })

    ctx.body = html
  } catch (error) {
    console.log('render error', error)
    throw error
  }
}
```

#### dev-ssr

```js
const Router = require('koa-router')
const axios = require('axios')
const fs = require('fs')
const path = require('path')
const MemoryFS = require('memory-fs') // 几乎等同 fs，扩展了 fs，文件生成在内存中
const webpack = require('webpack')
const VueServerRenderer = require('vue-server-renderer')

const serverRender = require('../server-render')

const serverConfig = require('../../build/webpack.config.server')

// 编译 webpack, 通过调用 webpack，传入配置对象，得到一个 complier 对象
// 通过对 compiler 对象 watch 或者 run 调用得到打包的 bundle 文件
const serverCompiler = webpack(serverConfig)
const mfs = new MemoryFS()
serverCompiler.outputFileSystem = mfs // 指定 compiler 输出目录

let bundle
// watch 文件改动，文件改动会重新打包生成一个新的 bundle
serverCompiler.watch({}, (err, stats) => {
  if (err) throw err // 打包报错
  stats = stats.toJson() // eslint 报错等
  stats.errors.forEach(err => console.log(err))
  stats.warnings.forEach(warn => console.log(warn))

  // 读取生成的 bundle
  const bundlePath = path.join(
    serverConfig.output.path,
    'vue-ssr-server-bundle.json'
  )

  bundle = JSON.parse(mfs.readFileSync(bundlePath, 'utf-8')) // 从内存中读取 bundle 字符串，vue-server-render 需要 json 形式
  console.log('new bundle generated')
})

const handleSSR = async ctx => {
  // 服务刚起时 webpack 第一次打包比较慢，此时 bundle 可能还不存在
  if (!bundle) {
    ctx.body = 'wait a moment!'
    return
  }

  // 获取客户端打包静态文件
  const { data: clientManifest } = await axios.get(
    `http://0.0.0.0:8000/dist/vue-ssr-client-manifest.json`
    // `http://127.0.0.1:8000/vue-ssr-client-manifest.json`
  )

  // 获取模板路径
  const template = fs.readFileSync(
    path.join(__dirname, '../server-template.ejs'), 'utf-8'
  )

  // 声明 renderer
  // 默认情况下直接传入生成的客户端清单和页面模板
  // const renderer = createBundleRenderer(serverBundle, {
  //   template,
  //   clientManifest
  // })
  // 当提供 template 选项时，资源注入是自动执行的，不按照官方规定注入设置 inject: false
  const renderer = VueServerRenderer
    .createBundleRenderer(bundle, {
      // 不按照官方规定注入
      inject: false,
      clientManifest
    })

  // 渲染页面
  await serverRender(ctx, renderer, template)
}

const router = new Router()
router.get('*', handleSSR)

module.exports = router
```

不同于生产环境的时，这时候的服务端 bundle 文件是需要 watch webpack 每次打包生成得到 bundle，而客户端清单文件需要请求客户端服务器内存中打包的文件















