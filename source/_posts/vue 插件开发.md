---
title: vue 插件开发
tag: 
	- vue
	- 插件
---

<!-- markdownlint-disable MD010 -->

> 参考: [vue 官方文档 插件部分](https://cn.vuejs.org/v2/guide/plugins.html)

## 开发插件

**Vue.js 的插件应当有一个公开的 install 方法。该方法的第一个参数是 vue 构造器， 第二个参数是一个可选的选项对象**

<!-- more -->

```javascript
MyPlugin.install = function(Vue, options) {
	// 1. 添加全局方法或者属性（直接添加在 Vue 构造函数上）
	Vue.myGlobalMethod = function() {
		// 逻辑...
	}

	// 2. 添加实例方法 （挂载在 Vue 原型上）
	Vue.prototype.$myMethod = function(methodOptions) {
		// 逻辑...
	}

	// 3. 注入组件
	Vue.mixin({
		created() {
			// 逻辑...
		}
		...
	})

	// 4. 添加全局资源
	Vue.directive('my-directive', {
		bind(el, binding, vnode, oldVnode){
			// 逻辑...
		}
	})
}
```

## 使用插件

使用全局方法 Vue.use() 使用插件

```javascript
Vue.use(MyPlugin)
```

也可传入选项对象：

```javascript
Vue.use(MyPlugin, {someOption: true})
```

**Vue.use 会自动阻止多次注册相同的插件，届时指挥注册一次插件**。

## 案例

如对于 axios 库，可以在入口文件引入，将其挂载在 Vue 构造函数原型上 (虽然不是插件用法)，这样在可以获取到 Vue 实例的地方都可以调用（相当于全局调用）

```javascript
// 入口文件 app.js
import Vue from 'vue'
import axios from 'axios'

Vue.prototype.$axios = axios

// 组件内使用
this.$axios.get....
```

上述代码有些弊端，别人可以覆盖你的方法。**通常使用 Object.defineProperty, 该方法可以定义一个 descriptor 属性。而使用descriptor定义的属性默认是只读的**

```javascript
// 入口文件 app.js
import Vue from 'vue'
import axios from 'axios'

Object.defineProperty(Vue.prototype, '$axios', {value: axios})

// 组件内使用
this.$axios.get....
```

更进一步， 可以对 axios 进行二次封装，处理成一个插件，在多个 vue 项目中使用

```javascript
// ./axios.js
import axios from 'axios'

export default {
	install: function(Vue) {
		Object.defineProperty(Vue.prototype, '$axios', {value: axios})
	}
}

// ./app.js
import Vue from 'vue'
import axiosPlugin from './axios'
Vue.use(axiosPlugin)

new Vue({
	mounted() {
		console.log(this.$axios)
	}
})
```

## 补充

Object.defineProperty(obj, propName, descriptor)
在给定对象上修改现有属性或添加一个新属性，并返回该对象（descriptor 可设置 get 和 set 方法、枚举等）

discriptor 属性：

- **value**: 设置属性的值

- **configurable**: 是否可以删除目标属性或者再次修改属性 true/false

- **enumberable**: 是否可以被枚举（使用for in或Object.keys）true/false

- **writable**: 是否可以被重写 true/false

```javascript
var o = {}

var bValue
Object.defineProperty(o, "b", {
	get: function() {
		return bValue
	},
	set: function(newVal) {
		bValue = newValue
	}
})

o.b = 38 // 对象o拥有了属性b，值为38 而且属性b被进行数据劫持

```