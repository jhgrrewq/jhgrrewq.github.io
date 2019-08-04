---
title: webpack2 升级（持续添加）
tag: 
	- webpack
---

> 参考： [webpack官方文档-迁移到新版本部分](https://doc.webpack-china.org/guides/migrating/)

## module.loaders 改为 module.rules

旧的loader配置被更强大的rules系统取代，后者允许配置loader以及其他更多选项。为了兼容旧版，module.loaders语法仍然有效，旧的属性名依然可以被解析。

<!-- more -->

旧写法

```javascript

module: {
    loaders: [{
        test: /.css$/,
        loaders: [
            "style-loader",
            "css-loader?modules=true"
        ]
    }]
}
```

新写法

```javascript
module: {
    rules: {
        test: /\.css$/,
        use: [{
            loader: "style-loader"
        }, {
            loader: "css-loader",
            options: {
                modules: true
            }
        }]
    }
}
```

## 链式loader

在webpack1中，loader可以链式调用，上一个loader的输出被作为下一个loader的输入。**使用rule.use配置选项，use可以设置为一个loader数组。**在webpack1中，loader通常用！连写，这种写法在webpack2中只有在使用旧的module.loaders才有效

旧写法

```javascript
module: {
    loaders: [{
        test: /\.less$/,
        loader: "style-loader!css-loader!less-loader"
    }]
}
```

新写法

```javascript
module: {
    rules: [{
        test: /\.less$/,
        use: [
            "style-loader",
            "css-loader",
            "less-loader"
        ]
    }]
}
```

## 取消在模块命中自动添加loader后缀

在引用loader时， 不能再省略 -loader 后缀

```javascript
module: {
    rules: [{
        test: /\.less$/,
        use: [
-           "style",
+           "style-loader",
-           "css",
+           "css-loader",
-           "less",
+           "less-loader"
        ]
    }]
}
```

当然通过配置 resolveLoader.moduleExtensions 启用旧行为，当官方并不推荐

```javascript
resolveLoader: {
    moduleExtensions: ["-loader"]
}
```

## 通过options配置loader

有两种方法可以配置webpack的loader，典型的options被称为query，是一个可以为添加到loader名之后的字符串，比较像一个query string

```javascript
module: {
    rules: [{
        test: /\.tsx?$/,
        loader: 'ts-loader?' + JSON.stringify({transpileOnly: false})
    }]
}
```

另一种方式是分开来写，写成一个单独的对象，紧跟在loader属性后

```javascript
module: {
    rules: [{
        test: /\.tsx?$/,
        loader: 'ts-loader',
        options: {transpileOnly: false}
    }]
}
```

## ExtractTextWebpackPlugin 破坏性改动

ExtractTextPlugin 需要使用版本2，才能在webpack2下正常运行

```javascript
npm install -D extract-text-webpack-plugin
```

该插件的配置变化只要体现在语法上

**ExtractTextPlugin.extract**

旧写法

```javascript
module: {
    rules: [{
        test: /\.css$/,
        loader: ExtractTextPlugin.extract("style-loader", "css-loader", {publicPath: "./dist"})
    }]
}
```

新写法

```javascript
module: {
    rules: [{
        test: /\.css$/,
        use: ExtractTextPlugin.extract({
            fallback: "style-loader",
            use: "css-loader",
            publicPath: "./dist"
        })
    }]
}

```

**new ExtractTextPlugin({options})**

旧写法

```javascript
plugins: [
    new ExtractTextPlugin("bundle.css", {
        allChunks: true,
        disable: false
    })
]
```

新写法

```javascript
plugins: [
    new ExtractTextPlugin({
        filename: "bundle.css",
        allChunks: true,
        disable: false
    })
]
```

## es6 代码分割

webpack1 中可以使用 require.ensure 作为程序懒加载chunks中的一种方法

```javascript
require.ensure([], function(require){
    var foo = require('./module')
})
```

es6 模块加载规范定义了 import() 方法，可以在运行时（runtime）动态加载es6模块，webpack将 import() 作为分割点并将要请求的模块放置在一个单独的chunk中。import() 接受模块名作为参数，并返回一个 Promise

```javascript
function onClick() {
    import('./module').then(module => {
        return module.default
    }).catch(e => {
        console.log('Chunk loading failed')
    })
}
```