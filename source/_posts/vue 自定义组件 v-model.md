---
title: vue 自定义组件 v-model
tag: 
	- vue
---

> 参考: [vue.js 官方文档 自定义组件 v-model 部分](https://cn.vuejs.org/v2/guide/components-custom-events.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BB%84%E4%BB%B6%E7%9A%84-v-model)

## vue 中的 v-model

vue 中 v-model 指令其实是一个语法糖，相当于将 input 标签的 value 属性值绑定到 val 属性，同时监听 input 标签的 input 事件，将输入内容赋值给 val 属性，实现了双向绑定

```html
<input v-model="val">

// 相当于
<input :value="val" @input="val = $event.target.value">
```

<!-- more -->

## 自定义组件 v-model

> 一个组件上的 v-model 默认会利用名为 value 的 prop 和名为 input 的事件，但是像单选框、复选框等类型的输入控件可能会将 value 特性用于不同的目的。model 选项可以用来避免这样的冲突

```js
Vue.component('custom-input', {
	model: {
		props: 'checked',
		event: 'change'
	},
	props: {
		checked: Boolean
	},
	template: `
		<input type="checkbox" :checked="checked" @change="$emit('change', $event.target.checked)">
	`
})
```

```html
<custom-input v-model="test"></custom-input>
```

**组件定义时要用 model 属性定义使用的 props 和 event，且要在 props 属性中声明 model 定义的 props 名**，这里组件定义 model 会使用名为 checked 的 prop 和名为 change 的事件，因此 组件 v-model 绑定的变量 test 的值会被传入 checked 这个 prop， 同时触发 change 事件时候传递新的值的时候，test 属性会被更新