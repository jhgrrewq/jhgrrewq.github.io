## 起步项目

- 安装官方脚手架 ` npm install -g create-react-app`
- 创建项目 `create-react-app react-study`
- 启动项目 `npm start`

## 文件结构

```bash
├── README.md              文档
├── public							   静态资源
│   ├── favicon.ico
│   ├── index.html
│   └── manifest.json
└── src									   源码
    ├── App.css
    ├── App.js             根组件
    ├── App.test.js
    ├── index.css          全局样式
    ├── index.js           入口文件
    ├── logo.svg
    └── serviceWorker.js   pwa支持
├── package.json           npm 依赖
```

## CRA （create-react-app）项目真容

使用 `npm run eject` 弹出项目真容，会多出两个目录

该操作不可逆

```bash
├── config
	├── env.js                     处理.env环境变量配置文件
	├── paths.js                   提供各种路径
	├── webpack.config.js          webpack配置文件
	└── webpackDevServer.config.js 测试服务器配置文件
└── scripts                      启动、打包和测试脚本 
	├── build.js                   打包脚本
	├── start.js                   启动脚本
	└── test.js                    测试脚本
```

### env.js 

用来处理 env 文件中配置的变量

```js
// node 运行环境：development production test 等
const NODE_ENV = process.env.NODE_ENV

// 待扫描的文件名数组
const dotenvFiles = [
  `${paths.dotenv}.${NODE_ENV}.local`, // .env.development.local
  `${paths.dotenv}.${NODE_ENV}`,	     // .env.development
  NODE_ENV !== 'test' && `${paths.dotenv}.local`, // .env.local
  paths.dotenv, // .env
].filter(Boolean); // 判断文件存在

// 从 .env* 文件中加载文件变量
dotenvFiles.forEach(dotenvFile => {
  if (fs.existsSync(dotenvFile)) {
    require('dotenv-expand')(
      require('dotenv').config({
        path: dotenvFile,
      })
    );
  }
});
```

### webpack.config.js

- 支持 ts

```js
// Check if TypeScript is setup 启用 ts
const useTypeScript = fs.existsSync(paths.appTsConfig); // 配置 tsconfig.json

// TypeScript type checking ts 相关插件启用
plugins: [
  // ...
  useTypeScript &&
    new ForkTsCheckerWebpackPlugin({
    // ...
  })
]
```

- 支持 sass 和 css 模块化

```js
// style files regexes
const cssRegex = /\.css$/;
const cssModuleRegex = /\.module\.css$/; // css 模块化
const sassRegex = /\.(scss|sass)$/;
const sassModuleRegex = /\.module\.(scss|sass)$/

module: {
  rules: {
    // ...
    {
      // "oneOf" will traverse all following loaders until one will
      // match the requirements. When no loader matches it will fall
      // back to the "file" loader at the end of the loader list.
      // 这里由上至下进行匹配，只有一个匹配后面就不匹配
      oneOf: [
        {
          test: cssRegex,
          exclude: cssModuleRegex, // 优先处理 css ，排除模块化 css 文件
          use: getStyleLoaders({
            importLoaders: 1,
            sourceMap: isEnvProduction && shouldUseSourceMap,
          }),
          sideEffects: true
        },
        {
          test: cssModuleRegex,
          use: getStyleLoaders({
            importLoaders: 1,
            sourceMap: isEnvProduction && shouldUseSourceMap,
            modules: {
              getLocalIdent: getCSSModuleLocalIdent,
            },
          })
        }
      ]
  	}
}
```

- 入口

```js
entry: [
      // Include an alternative client for WebpackDevServer. A client's job is to
      // connect to WebpackDevServer by a socket and get notified about changes.
      // When you save a file, the client will either apply hot updates (in case
      // of CSS changes), or refresh the page (in case of JS changes). When you
      // make a syntax error, this client will display a syntax error overlay.
      // Note: instead of the default WebpackDevServer client, we use a custom one
      // to bring better experience for Create React App users. You can replace
      // the line below with these two lines if you prefer the stock client:
      // require.resolve('webpack-dev-server/client') + '?/',
      // require.resolve('webpack/hot/dev-server'),
      isEnvDevelopment &&
        require.resolve('react-dev-utils/webpackHotDevClient'), // 开发环境 热更新文件
     	paths.appIndexJs // src/index 入口文件
    ].filter(Boolean),
```

## React 和 ReactDom

```js
import React from 'react'
import ReactDom from 'react-dom'

// babel-loader 可转换 jsx -> vdom, 需要使用 React.createElement() 等
// <h1>test</h1> -> React.createElement('h1', 'test') 
const jsx = <h1>test</h1>
ReactDOM.render(jsx, document.querySelector('#root'))
```

- React 负责逻辑控制， 数据 -> vdom
- ReactDom 渲染实际 dom，vdom -> dom，如果是移动端，要用别的库渲染
- React 可用 jsx 来描述 UI

## JSX

jsx 是一种 js 语法扩展，能很好描述 UI

- 使用 表达式 `{}`
- 表达式可以是 `变量` 、`常量` 、`函数` 、jsx 
- 条件可以通过三元表达式等实现
- 数组元素可通过 map 等函数实现
- class for 等属于保留字，class 用 className 替代， for 用 htmlFor 替代
- style 或 class 可以使用 css 模块化

```js
import style from './index.module.css'
<h1 className={ style.img }>test</h1>
```

## 补充知识

### dotenv 从文件加载环境变量

`require('dotenv').config()` 会默认读取项目根目录下的 .env 文件

```js
# 可传入路径和编码方式
config({
  path: '',
  Encoding: '' 
})
```

