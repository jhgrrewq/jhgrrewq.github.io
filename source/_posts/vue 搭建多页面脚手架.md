---
title: vue 搭建多页面脚手架
tag: 
	- vue
	- 多页面
	- 脚手架
---

> 使用 vue-cli 改造多页面脚手架，实质是改造成多入口和多页面输出。遍历读取入口文件，并通过HtmlWebpackPlugin对入口文件产出对应html

## 改造 vue-cli 目录结构

在 src 目录下新建文件夹如 module 存放页面，页面也按照文件夹存放，每个页面相关的资源放在对应的页面文件夹下

<!--more-->

~~~
-- src
--- components
--- module
---- index             // 首页
------ assets          // 首页资源目录
------ router          // 首页路由
------ App.vue
------ index.js        // 首页入口文件，必须和首页模板同名
------ index.html      // 首页模板

---- test              // 测试页
------ test.js         // 测试页入口文件，必须和测试页模板同名
------ test.html       // 测试页模板
~~~ 

## 改造 build/utils.js

使用 glob 模块， 增加获取多级入口文件的方法

```javascript
const glob = require('glob');

//获取多级的入口文件
exports.getMultiEntry = function (globPath) {
  var entries = {},
    basename, tmp, pathname;

  glob.sync(globPath).forEach(function (entry) {
    basename = path.basename(entry, path.extname(entry));
    tmp = entry.split('/').splice(-4);

    var pathsrc = tmp[0]+'/'+tmp[1];
    if( tmp[0] == 'src' ){
      pathsrc = tmp[1];
    }
    pathname = pathsrc + '/' + basename; // 正确输出js和html的路径
    entries[pathname] = entry;
  });

  return entries;
}
```

## 改造 build/webpack.base.conf.js

改造多入口

```javascript
const entries = utils.getMultiEntry('./src/module/**/*.js'); // 获得入口js文件

...

module.exports = {
    ...
    entry: entries,
    ...
}

...
```

## 改造 build/webpack.dev.conf.js , build/webpack.prod.conf.js

通过HtmlWebpackPlugin对每个入口文件产出对应html，module.exports.plugins每次push一个 HtmlWebpackPlugin 实例

- webpack.dev.conf.js （**将原有plugin配置中 HtmlWebpackPlugin 配置代码注释**）

```javascript
//构建生成多页面的HtmlWebpackPlugin配置，主要是循环生成
var pages = utils.getMultiEntry('./src/module/**/*.html');
  console.log(pages)

for (var pathname in pages) {
  // 配置生成的html文件，定义路径等
  var conf = {
    filename: pathname + '.html',
    template: pages[pathname], // 模板路径
    chunks: [pathname, 'vendors', 'manifest'], // 每个html引用的js模块
    inject: true // js插入位置
  };
  // 需要生成几个html文件，就配置几个HtmlWebpackPlugin对象 
  devWebpackConfig.plugins.push(new HtmlWebpackPlugin(conf))
}
```

- webpack.prod.conf.js （ **将原有plugin配置中 HtmlWebpackPlugin 配置代码注释, 此处 HtmlWebpackPlugin 的配置 conf 增加了hash** ）

```javascript
//构建生成多页面的HtmlWebpackPlugin配置，主要是循环生成
var pages =  utils.getMultiEntry('./src/module/**/*.html');
for (var pathname in pages) {

  var conf = {
    filename: pathname + '.html',
    template: pages[pathname], // 模板路径
    chunks: ['vendor',pathname], // 每个html引用的js模块
    inject: true,              // js插入位置
    hash:true
  };

  webpackConfig.plugins.push(new HtmlWebpackPlugin(conf));
}
```

> **可以参考我的 [github](https://github.com/jhgrrewq/vue-cli-multi)**