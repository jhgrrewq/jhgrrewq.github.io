---
title: webpack plugin 编写
tag: 
	- webpack
---

## 官网案例

~~~javascript
// MyPlugin.js

function MyPlugin(options) {
  // 定义一个构造函数
}

MyPlugin.prototype.apply = function(compiler) {
  // 操作 compiler 实例方法

  compiler.plugin("compile", function(params) {
    console.log("The compiler is starting to compile...");
  });

  compiler.plugin("compilation", function(compilation) {
    console.log("The compiler is starting a new compilation...");

    compilation.plugin("optimize", function() {
      console.log("The compilation is starting to optimize files...");
    });
  });

  compiler.plugin("emit", function(compilation, callback) {
    console.log("The compilation is going to emit files...");
    callback();
  });
};

module.exports = MyPlugin;

~~~

<!--more-->

webpack.config.js 中使用

~~~javascript
plugins: [
        new MyPlugin({options: 'nada'})
    ]
~~~

## 原理

- webpack 中 plugins 的每个元素都是一个插件的对象实例，因此**编写插件js需要写一个插件构造函数，并用 commonJS 风格导出该函数**
- **webpack 安装 plugin 时会自动调用 plugin 实例的 apply方法， 并传入 compiler 对象。 该对象可以获取与 wepack 打包相关的环境和参数。apply方法需要挂载在 plugin 构造函数的原型上**

```javascript
function MyPlugin(options) {

}

MyPlugin.prototype.apply = function(compiler) {

}
```

> **compiler 在开始打包（webpack 启动）时进行实例化，实例对象包含了 webpack 打包相关的环境和参数等配置信息，包括options，loaders，plugins**
> **compilation 每次一个文件变化重新打包会进行实例化，继承自compiler，包含modules和chunks信息**
> 两者的区别在于，compiler 对象代表了整个 webpack 从启动到关闭的生命周期，compilation 只是代表一个新的编译

- **compiler.plugin(事件名称，回调函数)** 调用 compiler 的 plugin方法，用于监听webpack在不同构建阶段广播的事件，在不同的构建阶段注入。**plugin方法第一个参数为事件， 第二个参数为函数，该函数传入webpack在该事件下实例化的对象， 以及下一个回调函数（可以不传，同步事件不传回调函数）**

- plugin方法中监听的 webpack 一些常用的事件

```javascript
make(compilation)
// make 事件： 为 compilation 增加入口等 addEntry(context, entry, name, callback)

compile（params）
// compile 事件： compiler开始编译， 能操作params对象

compilation（compilation）
// compilation 事件： compiler 正在编译， 能操作 compilation 对象

after-compile（compilation）
// compile 事件： 编译完成，并且modules已经封装，接下来要 emit 已经生成好的文件； 能操作 compilation 对象

emit(compilation)
// emit 事件： 开始 emit 已经生成好的文件； 能操作 compilation 对象

after-emit(compilation)
// after-emit 事件： compiler 已经 emit 全部生成好的文件； 能操作 compilation 对象

done(stats: Stats)
// done 事件：完成

```

> **一般监听 compilation 事件， 对 compilation 操作**

```javascript
compiler.plugin("compilation", function(compilation) {});
```

注意：**compiler 或 compilation 对象都能广播一系列事件**，因此在新开发的插件中也能广播出事件（拿到 compiler 或 compilation 对象并操作之），给其他插件监听使用，传给每个插件的 compiler 和 compilation 对象都是同一个引用，也就是说一个插件中修改了 compiler 或 compilation 对象上的属性，会影响到后面的插件。有些事件是异步的，这些异步事件会附带两个参数，第二个参数为回调函数，在插件处理完任务时需要调用回调函数通知 webpack，才会进行下一个处理流程

## 实现 html-webpack-before-plugin

### 需求

在打包的js前面预先加载用户自定义js

```javascrip
<script src='./my.js'></script>
<script src='./bundle.js'></script>

```

### 借助 html-webpack-plugin 

html-webpack-plugin 提供了一些事件给其他 plugin 监听

_Async:_

- html-webpack-plugin-before-html-generation
- html-webpack-plugin-before-html-processing
- html-webpack-plugin-alter-asset-tags
- html-webpack-plugin-after-html-processing
- html-webpack-plugin-after-emit

_Sync:_

- html-webpack-plugin-alter-chunks

### 基本实现

> **监听 html-webpack-plugin-after-html-processing 事件， 能操作 htmlPluginData 对象**
> **htmlPluginData.html 属性 是打包的 html字符串**
> **htmlPluginData.assets 属性 是打包的静态资源，htmlPluginData.assets.js 数组存放打包的 js路径字符串**

```javascript
function MyPlugin(options) {
    this.options = options
}

MyPlugin.prototype.apply = function(compiler) {
    let paths = this.options.paths
    compiler.plugin('compilation', function(compilation) {
        compilation.plugin('html-webpack-plugin-before-html-processing', function(htmlPluginData, callback) {
            for(let i = paths.length - 1; i >= 0; i--) {
                htmlPluginData.assets.js.unshift(paths[i])
            }
            // 将处理后的数据 传入下一个回调
            callback(null, htmlPluginData);
        })
    })
}

module.exports = MyPlugin

```

webpack.config.js

```javascript
...
plugins: [
        new MyPlugin({
            paths: ['./my.js']
        })
    ]
...

```