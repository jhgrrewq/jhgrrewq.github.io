---
title: webpack loader 编写
tag:
	- webpack
---
<!-- markdownlint-disable MD010 -->

> 编写的 webpack 扩展如果是想对一个个单独文件进行转换就编写 loader，否则编写 plugin

- **一个 loader 的职责是单一的，只需要完成一种转换** 如果一个源文件需要进行经过多种转换才能正常使用，就通过多个 loader 转换。调用多个 loader 转换一个文件时，每个 loader 会链式顺序执行，第一个 loader 将会拿到需要处理的原内容，上一个 loader 处理后的结果会传给下一个接着处理，最后的 loader 将处理后的最终结果返回给 webpack

- 开发 loader 时请保持其职责的单一性，只需关心输入和输出

<!-- more -->

## loader 基础

简单情况下，**只有一个 loader 作用于资源时，调用 loader 有一个参数：作为字符串的资源文件的内容**

这个 loader 能在这个函数的上下文中 this 方法 loader api

一个同步 loader 可以通过 return 返回这个值，在其他情况下，可以通过 this.callback(err, values...) 函数返回任意数量的值。错误会被传到 this.callback 函数或者在同步 loader 中抛出

这个 loader 应该返回一个或两个值，第一个值是 js 代码产生的字符串或缓冲区，第二个可选的值是 js 对象的 sourcemap

```js
// 定义 loader
module.exports = function(source) {
	// 对 source 进行处理
	return source
}

// 通过 sourcemap 支持定义 loader
module.exports = function(source, map) {
	// ...
	this.callback(null, source, map)
}
```

**大多数 loader 是可以缓存的，调用 cacheable 方法可以将自身标识成可缓存**

```js
module.exports = function(source) {
	this.cacheable && this.cacheable()
	return source
}
```

如下是一个简单的 loader

```js
module.exports = function(source) {
	this.cacheable && this.cacheable()
	return '/*copy@ jack*/' + JSON.stringify(source)
}
```

## fast-vue-md-loader

webpack 默认只能处理 js 文件，对于其他类型的文件需要借助 loader 进行转换

如对于 markdown 文件，要先通过 fast-vue-md-loader 转化为 vue 组件，再通过 vue-loader 转化为 js 文件

```js
// webpack 配置

module: {
	rules: [
		{
			test: /\.md/,
			use: [
				'vue-loader',
				'fast-vue-md-loader'
			]
		},
		{
			test: /\.vue$/,
			use: [
				{
					loader: 'vue-loader',
					options: {
						preserveWhitespace: false,
						extractCSS: true
					}
				}
			]
		},
		...
	]
}
```

fast-vue-md-loader 源码实现是借助 markdown-it 库，通过该库的实例方法 **render() 将传入的 markdown 字符串转化为 包含 html 标签 的字符串，然后写入 vue 组件字符串 template 中的一个 标签中（使用 v-html 指令，跳过转义检查）**，最后导出该 vue 组件模板

```js
const hljs = require('highlight.js');
const MarkdownIt = require('markdown-it');

const wrapper = content => `
<template>
  <section v-html="content" v-once />
</template>
<script>
export default {
  created() {
    this.content = unescape(\`${escape(content)}\`);
  }
};
</script>
`;

const highlight = (str, lang) => lang && hljs.getLanguage(lang) ? hljs.highlight(lang, str, true).value : '';
const parser = new MarkdownIt({
  html: true,
  highlight
});

// 导出一个函数
module.exports = function(source, options) {
  this.cacheable && this.cacheable();

  options = {
    wrapper,
    ...options
  };

	// new MarkdownIt().render('markdownStr') 传入 markdown 字符串，返回含 html 标签的字符串
	// 最后返回 包含有 html 标签 字符串 的 vue 组件字符串
  return options.wrapper(parser.render(source));
}
```