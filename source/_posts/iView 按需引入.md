---
title: iView 按需引入
tag: 
	- iView
---

iView 一般直接使用在入口文件中配置

```javascript
import iView from 'iview'
import 'iview/dist/styles/iview.css'     //iview css
import 'src/assets/css/iview-theme.less' //自定义主题

Vue.use(iView)
```

<!-- more -->

按需加载组件可以有效减少文件体积，而且并不是 iView 的所有组件都使用到。需要借助插件 babel-plugin-import

### .babelrc 配置

```javascript
{
    ...
    "plugin": [["import", {
        "libraryName": "iview",
        "libraryDirectory": "src/component"
    }]]
}
```

### 入口文件仍然要导入样式

```javascript
import 'iview/dist/styles/iview.css'     //iview css
```

### 按需引入组件

```javascript
// 如在入口文件中
import { BackTop } from 'iview'
Vue.component('BackTop', BackTop)

// 在组件中 用法和引用自定义组件相同
import { BackTop } from 'iview'
export default {
    ...
    components: {
        BackTop
    }
}
```

### 打包编译

如果没有在 webpack 的 modules 中对 loader 进行配置，生产环境打包 'iview/src' 目录下 js 文件会出现错误 "ERROR in xxx.js from UglifyJs"

```javascript
// webpack.base.conf.js
{
    test: /iview.src.*?js$/,
    loader: 'babel-loader'  // 使用 babel-loader
}
```