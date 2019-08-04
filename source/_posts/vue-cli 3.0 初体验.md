---
title: vue-cli 3.0 初体验
tag: 
	- vue
---

> 参考: [vue-cli 英文官方文档](https://github.com/vuejs/vue-cli)、[The npm Blog——新的包名规则](https://zcfy.cc/article/the-npm-blog-new-package-moniker-rules)、[Vue CLI 3.x 简单体验](https://segmentfault.com/a/1190000013090943)、[vue.config.js 官方文档](https://github.com/vuejs/vue-cli/blob/dev/docs/config.md)

vue-cli 3.0 目前仍只是 beta 版本，不建议在生产环境使用

vue-cli 3.0 尽可能简化配置，零配置让用户能快速搭建项目进行开发，但同时这也会导致缺少灵活性

## 安装

```js
npm install -g @vue/cli
```

这里 vue-cli 使用了新的 npm 包的命名方式

- 为了防止命名的包名和 npm 上的包名重复，可使用作用域，将该包作为你 npm 官网用户名下的**私有模块**，命名规则为 `@` + `npm 官网用户名 / 模块名`
- 最后命令行发布包时添加 `--access=public` 将该包变成共有包

<!-- more -->

```js
// 如 package.json 包命名使用作用域，该包变成私有包
{
  "name": "@qwerrghj/vue"
  ...
}

// 命令行发布添加 --access=public 变成公有包
npm publish --access=public
```

## vue-cli 命令

```js
vue -V, vue --version               输出版本号
vue -h, vue --help                  输出使用信息

vue create [options] <app-name>                  使用 <app-name> 创建一个项目，无需选择模板

vue-cli-service add <plugin> [pluginOptions]     在已创建项目中安装插件
vue-cli-service invoke <plugin> [pluginOptions]  在已创建项目中添加插件
vue-cli-service inspect [options] [paths...]     检查 webpack config 配置


vue-cli-service serve [options] [entry]          开发模式下零配置运行一个 js or vue 文件
vue-cli-service build [options] [entry]          生产模式下零配置运行一个 js or vue 文件

vue init <template> <app-name>                   同旧版 api 使用模板创建项目，但是需要安装 @vue/cli-init
```

这里 vue-cli-service 应该是包含了生产和开发环境的配置文件并起了一个 http 服务器

`vue-cli-service serve` 相当于 `npm run dev`, 类似 webpack-dev-server 跑 webpack.dev.config.js
`vue-cli-service build` 相当于 `npm run build`, 类似使用 webpack.prod.config.js 对文件进行 webpack 打包

## 创建项目

使用 `vue create <app-name>` 创建项目会出现提示面板，提供了两种选择，分别是 **default 默认** 和 **Manually 手动模式**

```js
Vue CLI v3.0.0-beta.6
? Please pick a preset: (Use arrow keys)
❯ default (babel, eslint)
  Manually select features
```

### default

选择默认模式会自动在执行目录下使用设置的项目名创建项目、初始化 git 仓库、安装 cli 插件

```js
✨  Creating project in /Users/jackzhang/Desktop/@qwerrghj:vue/myProject.
🗃  Initializing git repository...
⚙  Installing CLI plugins. This might take a while...
```

### Manually

手动模式会继续出现一些选择面板，提示可选择安装一些模块，使用 空格 选中或取消， a 全选，i 反选

```js
Vue CLI v3.0.0-beta.6
? Please pick a preset: Manually select features
? Check the features needed for your project: (Press <space> to select, <a> to t
oggle all, <i> to invert selection)
❯◯ TypeScript
 ◯ Progressive Web App (PWA) Support
 ◯ Router
 ◯ Vuex
 ◯ CSS Pre-processors
 ◯ Linter / Formatter
 ◯ Unit Testing
 ◯ E2E Testing,
```

对于选择安装的模块还会有一些后续选择安装提示，如对于要安装 css 预处理器，会提示选择 sass/scss、less 或 stylus

```js
? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported 
by default): (Use arrow keys)
❯ SCSS/SASS 
  LESS 
  Stylus 
```

可以设置将配置是单独放置成配置文件还是作为 package.json 的字段

```js
? Where do you prefer placing config for Babel, PostCSS, ESLint, etc.? (Use arro
w keys)
❯ In dedicated config files 
  In package.json 
```

甚至还可以将配置保存作为以后项目的配置

```js
? Save this as a preset for future projects? (Y/n) 
```

### 项目结构

使用 default 模式创建的项目结构如下，manually 模式的结构类似，src 源码可以看出这依然是一个单页面项目

![](http://ony85apla.bkt.clouddn.com/18-4-15/4364696.jpg)

## 启动项目

package.json 中 script 字段配置了一些常用命令，发现 `npm run serve` 其实就是 `vue-cli-service serve --open`, 相当于之前 `npm run dev`；对应的 `npm run build` 其实就是 `vue-cli-service build`, 相当于之前 `npm run build`

```js
"scripts": {
  "serve": "vue-cli-service serve --open",
  "build": "vue-cli-service build",
  "lint": "vue-cli-service lint"
}
```

## 项目自定义配置

项目中需要自定义配置可在根目录下创建 vue.config.js 文件

```js
module.exports = {
  // Project deployment base
  // By default we assume your app will be deployed at the root of a domain,
  // e.g. https://www.my-app.com/
  // If your app is deployed at a sub-path, you will need to specify that
  // sub-path here. For example, if your app is deployed at
  // https://www.foobar.com/my-app/
  // then change this to '/my-app/'
  baseUrl: '/',

  // where to output built files
  outputDir: 'dist',

  // whether to use eslint-loader for lint on save.
  // valid values: true | false | 'error'
  // when set to 'error', lint errors will cause compilation to fail.
  lintOnSave: true,

  // use the full build with in-browser compiler?
  // https://vuejs.org/v2/guide/installation.html#Runtime-Compiler-vs-Runtime-only
  compiler: false,

  // tweak internal webpack configuration.
  // see https://github.com/vuejs/vue-cli/blob/dev/docs/webpack.md
  // webpack 配置
  chainWebpack: () => {},
  configureWebpack: () => {},

  // vue-loader options
  // https://vue-loader.vuejs.org/en/options.html
  // vue-loader 配置
  vueLoader: {},

  // generate sourceMap for production build?
  productionSourceMap: true,

  // CSS related options
  css: {
    // extract CSS in components into a single CSS file (only in production)
    extract: true,

    // enable CSS source maps?
    sourceMap: false,

    // pass custom options to pre-processor loaders. e.g. to pass options to
    // sass-loader, use { sass: { ... } }
    loaderOptions: {},

    // Enable CSS modules for all css / pre-processor files.
    // This option does not affect *.vue files.
    modules: false
  },

  // use thread-loader for babel & TS in production build
  // enabled by default if the machine has more than 1 cores
  // 是否使用多线程
  parallel: require('os').cpus().length > 1,

  // split vendors using autoDLLPlugin?
  // can also be an explicit Array of dependencies to include in the DLL chunk.
  // See https://github.com/vuejs/vue-cli/blob/dev/docs/cli-service.md#dll-mode
  // 是否使用 autoDLLPlugin
  dll: false,

  // options for the PWA plugin.
  // see https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-pwa
  pwa: {},

  // configure webpack-dev-server behavior
  // Webpack dev server
  devServer: {
    open: process.platform === 'darwin',
    host: '0.0.0.0',
    port: 8080,
    https: false,
    hotOnly: false,
    // See https://github.com/vuejs/vue-cli/blob/dev/docs/cli-service.md#configuring-proxy
    proxy: null, // string | Object
    before: app => {}
  },

  // options for 3rd party plugins
  pluginOptions: {
    // ...
  }
}
```

如下是一个 vue.config.js 的案例

```js
const path = require('path')
const webpackConfig = {
  resolve: {
    // extensions: ['.js', '.json'],
    alias: {
      "@src": path.resolve(__dirname,'src/')
    }
  },
  plugins:[]
}

module.exports = {
  //configureWebpack 可以是 function 或 Object
  configureWebpack:webpackConfig,
  devServer: {
    open: process.platform === 'darwin',
    host: '0.0.0.0',
    // port: 8080,
    https: false,
    hotOnly: false,
    proxy: {
      '/api':{target:'http://localhost:3030'}
    }, // string | Object
    // before: app => {
    //   // app is an express instance
    //   // console.log(Object.keys(app))
    //   fn(app)
    // }
  }
}

```