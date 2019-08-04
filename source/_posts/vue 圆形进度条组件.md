---
title: vue 圆形进度条组件
tag: 
	- vue
	- svg
    - 组件
---

> 基于 svg 的 stroke-dasharray 和 stroke-dashoffset

svg 为可缩放矢量图形。 svg 代码以为根标签 svg 开始，包含开始标签和闭合标签。width 属性和 height 属性用于设置该 svg 文档的宽度和高度，version 属性定义所使用的 svg 的版本， xmlns 属性定义 svg 的命名空间。
**viewBox 属性："起始点横坐标，起始点纵坐标，内部宽度，内部高度" 设置视口，其内部宽度高度需要等于 circle 的直径**

<!-- more -->

circle 标签用于创建圆，cx 和 cy 属性定义圆点的 x 和 y 坐标。如果省略 cx 和 cy，圆的中心会被设置为 (0, 0)；r属性定义圆的半径。

svg的 fill 属性定义填充的颜色，stroke 属性定义描边颜色，stroke-width 定义描边宽度，**stroke-dasharray 定义描边虚线长度，stroke-dashoffset 定义描边的偏移。**

```svg
<svg width="83" height="83" version="1.1" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
    <circle r=50 cx=50 cy=50
            stroke-width="50"
            stroke="#D1D3D7"
            fill="transparent"></circle>
    <!-- 基于stroke-dasharray和stroke-dashoffset属性圆形进度条 -->
    <!-- stroke-dasharray 定义描边的虚线长度 -->
    <!-- stroke-dashoffset 定义描边的偏移 -->
    <circle r=50 cx=50 cy=50
            stroke-width="50"
            stroke="#00A5E0"
            fill="transparent"
            stroke-dasharray="314"
            stroke-dashoffset="160"></circle>
</svg>
```

> **原理就是基于 svg 的 stroke-dasharray 和 stroke-dashoffset，stroke-dasharray 等于圆的周长，stroke-dashoffset 定义描边的偏移，百分比的值通过外部计算传入。**

组件可还以从外部 props 传入数据设置该圆形进度条组件的大小，描边颜色等

```vue
<template>
    <div class="progress-circle">
        <!-- 使用svg circle标签创建圆 -->
        <svg :width="radius" :height="radius" version="1.1" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
            <circle class="progress-background"
                    :style="'stroke:'+bgColor+''"
                    r=50
                    cx=50
                    cy=50
                    fill="transparent"></circle>
            <!-- 基于stroke-dasharray和stroke-dashoffset属性圆形进度条 -->
            <!-- stroke-dasharray 定义描边的虚线长度 -->
            <!-- stroke-dashoffset 定义描边的偏移 -->
            <circle class="progress-bar"
                    :style="'stroke:'+fillColor+''"
                    r=50
                    cx=50
                    cy=50
                    fill="transparent"
                    :stroke-dasharray="dasharray"
                    :stroke-dashoffset="dashoffset"></circle>
        </svg>
        <!-- 使用slot填充外部调用progress-circle组件包裹的dom -->
        <slot></slot>
    </div>
</template>

<script>
    export default {
        props: {
            fillColor: {
                type: String,
                default: '#72cfa0'
            },
            bgColor: {
                type: String,
                default: '#72cfa0'
            },
            radius: {
                type: Number,
                default: 50
            },
            percent: {
                type: String,
                default: '0'
            }
        },
        data() {
            return {
                // 计算描边的周长
                dasharray: Math.PI * 100
            }
        },
        computed: {
            dashoffset() {
                return (1 - Number(this.percent)) * this.dasharray
            }
        }
    }
</script>

<style lang="less" scoped type="text/less">
    .progress-circle{
        position: relative;
        circle{
            stroke-width: 5px;
            transform-origin: center;
            &.progress-background{
                transform: scale(0.9);
                stroke-width: 1px;
            }
            &.progress-bar{
                transform: scale(0.85) rotate(-90deg);
            }
        }
    }
</style>
```