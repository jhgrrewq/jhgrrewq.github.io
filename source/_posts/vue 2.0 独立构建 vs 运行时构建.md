---
title: vue 2.0 独立构建 vs 运行时构建
tag: 
	- vue
	- 构建
---

<!-- markdownlint-disable MD010 -->

> 参考: [Vue.js 官网 对不同构建版本解释](https://cn.vuejs.org/v2/guide/installation.html#%E5%AF%B9%E4%B8%8D%E5%90%8C%E6%9E%84%E5%BB%BA%E7%89%88%E6%9C%AC%E7%9A%84%E8%A7%A3%E9%87%8A)、[从一个奇怪的错误出发理解 Vue 基本概念](https://zhuanlan.zhihu.com/p/25486761)

## 基础知识

在 NPM 包的 `dist/` 目录下有多个 Vue.js 构建版本

|  | UMD | CommonJS | ES Module |
| ---------- | :------- | :--------- | :---------- |
| **完整版** | vue.js | vue.common.js | vue.esm.js |
| **只含运行时版** | vue.runtime.js | vue.runtime.common.js | vue.runtime.esm.js |
| **完整版（生产环境）** | vue.min.js | - | - |
| **只含运行时版（生产环境)** | vue.runtime.min.js | - | - |

<!-- more -->

## 术语

- **完整版** 同时包含编译器和运行时的版本

- **编译器** 用来将模板字符串编译为 javascript 渲染函数（render 函数）的代码

- **运行时** 用来创建 vue 实例、渲染并处理虚拟 dom 等的代码。__基本就是去除编译器的其他一切__

- **UMD** 该版本可以通过 `script` 标签直接用于浏览器中，__一般 cdn 中的版本默认是 运行时 + 编译器的 UMD 版本__

- **CommonJS** 该版本用来配合老的打包工具如 Browserif 或 webpack1，__这些打包工具的默认文件（pkg.main）是只包含运行时的 CommonJS 版本（vue.runtime.common.js）__

- **ES Module** 该版本用来配合现代打包工具如 webpack2 或 rollup，__这些打包工具的默认文件（pkg.main）是只包含运行时的 ES Module 版本（vue.runtime.esm.js）__

## 原理

从上可知，Vue 有两种不同的构建————独立构建和运行时构建。**运行时构建删除了模板编译功能，因此无法支持带 `template` 属性的 Vue 实例选项**

- 如果在 html 文件使用 script 标签引入独立构建的 Vue 文件（ umd 版本），可以使用带 `template` 属性的 Vue 实例选项
- 一般 webpack 打包的项目中 `node_modules/vue/package.json` 文件中，默认已经通过 `main` 字段指定了通过 `import Vue from 'vue'` 或 `require('vue')` 所引入的文件是 `dist/vue.runtime.common.js`, 也就是运行时构建的 Vue 文件, 因此使用带 `template` 属性的 Vue 实例选项会报错

```js
module.exports = {
	// ...
	resolve: {
		extensions: ['.js', '.vue'],
		alias: {
			// 精确匹配
			'vue$': 'vue/dist/vue.common.js'
		}
	},
	// ...
}
```

解决上面 webpack 项目中的问题需要在 webpack 配置中 `resolve` 属性添加 `alias` 设置

## vue 选项对象中 el 属性、template 属性和 render 渲染函数 （需要渲染 dom）

> 当 Vue 选项对象中有 render 渲染函数，Vue 构造函数会直接使用渲染函数渲染 dom （忽略 template 属性和 el 属性）；当选项对象中没有 render 渲染函数时，Vue 构造函数首先将 template 模板编译成渲染函数，然后在渲染 dom；而当 Vue 实例选项中既没有 render 渲染函数，又没有 template 模板，会通过 el 属性获取挂载元素的 outerHTML 作为模板，并编译生成函数

- 总之，渲染 dom 时，render 渲染函数优先级最高，template 次之并且需编译成渲染函数，而挂载点 el 属性对应元素存在并且前两者都不存在，其 outerHTML 才会用于编译并渲染

```js
// index.html
<body>
	<div class="app1">{{msg}}</div>
	<div class="app2">{{msg}}</div>
	<div class="app2">{{msg}}</div>
</body>

// index.js
new Vue({
	el: '.app1',
	data: {
		msg: 'el'
	},
	template: `<div>template</div>`,
	render: h => h('div', {}, 'render')
})
new Vue({
	el: '.app2',
	data: {
		msg: 'el'
	},
	template: `<div>template</div>`
})
new Vue({
	el: '.app3',
	data: {
		msg: 'el'
	}
})
```

- 当然创建 Vue 实例时候可以不包含 el 属性、template 属性和 render 渲染函数。**实际上，如果 Vue 实例选项中没有设置 el 属性也没有显示调用 $mount 方法，是不会触发对 render 渲染函数（无论是手动编写还是编译生成）的检查**

```js
// 常见场景是 eventBus
// event-bus.js
import Vue from 'vue'
export const EventBus = new Vue()

// 组件中使用
import EventBus from 'event-bus.js'
EventBus.$on('event1', function(...args) {
	// ...
})
```

## 什么时候使用独立构建，什么时候使用运行时构建

- 在单文件组件中仍然可以使用 template 模板，是因为webpack 在打包时候已经将单文件组件的模板预编译为 render 函数，因此一般情况使用运行时构建 Vue 版本文件
- 如果使用 `new Vue` 创建 Vue 实例时，模板是从 el 挂载元素提取或使用 template 属性，需要使用独立构建 Vue 版本文件
- 在 script 标签中引入 Vue.js 项目中，任意实例选项或组件选项中包含了 template 模板属性或从 el 挂载元素提取模板时，均需要使用独立构建的 Vue 库