---
title: 解决 ERROR in xxx.js from UglifyJs
tag: 
	- es6
    - babel
---

![](http://ony85apla.bkt.clouddn.com/18-1-19/12173073.jpg)

<!-- more -->

webpack 生产环境打包 npm run build 报错，根本原因是 webpack 打包文件没有成功转化 es6 语法

> **解决方案: 安装 babel-preset-env 插件**

babel-preset-env 插件能自动根据目标运行环境自动启用需要的 babel 插件。 使用该插件，不再需要使用 es20xx presets

## 安装

```javascript
npm i -D babel-preset-env
```

## 配置 webpack-base-conf.js (如果使用 vue-cli 生成的项目)

```javascript
{
    test: /\.js$/,
    use: [{
        loader: 'babel-loader',
        options: {
            presets: ['env']
        }
    }],
    include: [resolve('src'), resolve('test')]
}
```

## 项目根目录配置 .babelrc

.babelrc 文件用于配置转码的规则和插件。 一般带有rc结尾的文件通常代表自动运行加载的文件、配置等。babel6.x 版本后，所有插件都是可插拔的。这意味着安装了babel 之后是不能工作的，还必须配置对应的 .babelrc 文件。

```javascript
{
  "presets": ["env"]
}
```