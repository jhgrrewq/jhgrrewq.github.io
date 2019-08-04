---
title: vue 2.0 生命周期
tag: 
	- vue
	- 生命周期
---

<!-- markdownlint-disable MD010 -->

## vue2.0 生命周期

vue生命周期钩子函数可以在组件特定阶段进行相关操作

| vue2.0 钩子 | 描述 |
| :------: | :------: |
| beforeCreate | **实例刚被创建，数据观测 (data observer) 和 event/watcher 事件配置前**。$el，data属性等不存在 |
| created | 实例已完成：**数据观测 (data observer)，属性和方法的运算，watch/event 事件回调**。 挂载阶段未开始， $el属性不存在 |
| beforeMount | 模板编译/挂载前 |
| mounted | 模板编译/挂载后，dom已经挂载。**不保证全部子组件挂载，如果需要整个视图完全渲染，需要用$nextTick 替换 mounted** |
| beforeUpdate | 组件更新前 |
| updated | 组件更新后 |
| deactivated | 针对 keep-alive，组件停用时调用 |
| activated | 针对 keep-alive，组件激活时调用 |
| beforeDestroy | 实例被销毁前调用，**实例仍然可用** |
| destroyed | 实例被销毁后调用 |

<!-- more -->

![](http://ony85apla.bkt.clouddn.com/18-1-18/56931152.jpg)

```vue
<template>
    <div id="index">
        <p>{{msg}}</p>
        <button @click="change">更新</button>
        <button @click="destroy">销毁</button>
    </div>
</template>

<script>
window.vue = new Vue({
	el: '#index',
	data() {
        return {
            msg: 'test'
        }
	},
	methods: {
		change() {
			this.msg = 'msg change'
		},
		destroy() {
			this.$destroy()
		}
	},
	components: {
	},
	beforeCreate() {
		console.log('beforeCreate 创建前')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	},
	created() {
		console.log('Created 创建后')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	},
	beforeMount() {
		console.log('beforeMount 载入前')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	},
	mounted() {
		console.log('mounted 载入后')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	},
	beforeUpdate() {
		console.log('beforeUpdate 更新前')
		console.log('el_inner: ', this.$el.innerHTML)
		console.log('msg: ', this.msg)
	},
	updated() {
		console.log('updated 更新后')
		console.log('el_inner: ', this.$el.innerHTML)
		console.log('msg: ', this.msg)
	},
	beforeDestroy() {
		console.log('beforeDestroy 销毁前')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	},
	destroyed() {
		console.log('destroyed 销毁后')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	}
})
</script>
```

vue 2.0 生命周期总共分为 8 个阶段: 创建前/后、挂载前/后、更新前/后、销毁前/后

### beforeCreate / created

- beforeCreate: vue 实例挂载元素 **el 和 数据属性（data, computed等）未初始化**

- created: **数据初始化， el 未初始化**

### beforeMount / mounted

- beforeMount: **数据和 el 完成初始化，但还是挂载前的虚拟 dom，dom 中的数据未替换**

- mounted: **dom 完成挂载**

![](http://ony85apla.bkt.clouddn.com/18-1-18/59633767.jpg)

### updated

- beforeUpdate: **数据更新时调用，发生在虚拟 dom 重新渲染之前**

- updated: **数据更新导致虚拟 dom 重新渲染后, 此时dom已经更新**

![](http://ony85apla.bkt.clouddn.com/18-1-18/87920448.jpg)

### destroyed

- beforeDestroy: 实例销毁之前调用。**实例仍然完全可用**

- destroyed: **调用后 Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁**（点击更新按钮已经无效）

![](http://ony85apla.bkt.clouddn.com/18-1-18/85794316.jpg)

> 综上，可以在生命周期钩子做一些相关操作：
beforeCreated: 如加载loading
created: 结束loading, 一些初始化，定义一些不需要加入数据观测的公共数据
mounted: 开启定时器，向后端请求数据
beforeDestroy: 删除确认操作
destroyed: 关闭定时器， 相关清空操作

## vue父子组件生命周期顺序

父组件

```vue
<template>
    <div id="index">
		<p>{{msg}}</p>
		<child></child>
	</div>
</template>

<script> 
import child from './child.vue'
export default {
	data() {
        return {
            msg: 'parent'
        }
	},
	beforeCreate() {
		console.log('parent beforeCreate 创建前')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	},
	created() {
		console.log('parent created 创建后')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	},
	beforeMount() {
		console.log('parent beforeMount 载入前')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	},
	mounted() {
		console.log('parent mounted 载入后')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	},
	components: {
		child
	}
}
</script>
```

子组件

```vue
<template>
    <div id="child">
		<p>{{msg}}</p>
	</div>
</template>

<script>
export default {
	data() {
        return {
            msg: 'child'
        }
	},
	beforeCreate() {
		console.log('child beforeCreate 创建前')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	},
	created() {
		console.log('child created 创建后')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	},
	beforeMount() {
		console.log('child beforeMount 载入前')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	},
	mounted() {
		console.log('child mounted 载入后')
		console.log('el: ', this.$el)
		console.log('msg: ', this.msg)
	}
}
</script>
```

![](http://ony85apla.bkt.clouddn.com/18-1-18/5700414.jpg)

父子组件的执行顺序如下:
**父组件 beforeCreate -> 父组件 created -> 父组件 beforeMount —> 子组件 beforeCreate -> 子组件 created -> 子组件 beforeMount -> 子组件 mounted -> 父组件 mounted**

因为vue生命周期 是实例先创建完成数据,事件回调等初始化， 然后才开始模板编译。父组件beforeMount生命周期后需要编译模板，因为引用子组件，因此进入子组件生命周期。子组件完成挂载后，最后父组件才完成挂载。
