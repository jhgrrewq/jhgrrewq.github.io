---
title: vue 父组件修改子组件样式
tag: 
	- vue
---

## 问题

父组件添加 scoped 后无法覆盖修改子组件样式， 而去除 scoped 属性虽然可以修改，但是会污染全局样式

> vue 父组件样式中使用 /deep/ 或 >>> 
> __常用 /deep/, 一些css预编译器不能识别 >>>__

```javascript
    子组件根选择器名 /deep/ 子组件后代选择器名 {
        ...
        // 覆盖的样式
    }

```

<!--more-->

## 代码

父组件

```vue
<template>
    ...
    <header></header>
    ...
</template>

<script>
    import header from './header.vue'
    export default {
        ...
        components: {
            header
        }
        ...
    }
</script>

<style scoped>
    .header /deep/ .title{
        color: red
    }
</style>

```

子组件

```vue
<template>
    <div class="header">
        <div class="title">jack</div>
    </div>
</templat

<script>
     export default {
        ...
     }
</script>

<style scoped></style>

```