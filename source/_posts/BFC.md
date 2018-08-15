---
title: BFC
tag: 
	- BFC
---

> 参考: [10 分钟理解 BFC 原理](https://zhuanlan.zhihu.com/p/25321647)、[前端精选文摘：BFC 神奇背后的原理](http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html)

<!-- markdownlint-disable MD010 -->

## 基本概念

### 定位

普通流

> 在普通流中，元素按照其在 html 中的先后位置从上到下布局。在这个过程中，行内元素水平排列，直到当行被占满然后换行；块级元素则会被渲染为完成的一个新行，除非另外指定，否则所有元素默认都是普通流定位。可以说普通流中元素的位置由该元素在 html 文档中的位置决定

<!-- more -->

浮动

> 浮动元素脱离了文档流。在浮动布局中，元素首先按照普通流的位置出现，然后根据浮动的方向尽可能的向左边或右边偏移

绝对定位

> 绝对定位布局中的元素会整体脱了文档流，因此绝对定位元素不会对其兄弟元素造成影响。而元素具体的位置由绝对定位的坐标决定

### Box

Box 是 css 布局的对象和基本单位。一个页面由多个 Box 组成。元素的类型和 display 属性决定了 Box 的类型，不同类型的 Box 会参与不同的 Fomatting Context（一个决定如何渲染文档的容器），因此 Box 内的元素会以不同的方式渲染

- **block-level box（块级 box）** display 属性为 block, list-item, table 的元素，会生成 block-level box，参与 BFC(block formatting context)
- **inline-level box（行内 box）** display 属性为 inline, inline-block,  inline-table 的元素，会生成 inline-level box，参与 IFC(inline formatting context)

### Formatting Context

> Formatting context 是 W3C CSS2.1 规范中的一个概念。它是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素如何定位，以及和其他元素的关系和相互作用。

最常见的有 BFC 和 IFC

## BFC

BFC（block formatting context）直译为“块级格式化上下文”。**它是一个独立的渲染区域，只有 block-level box (块级 box)参与，它规定了内部的 block-level box 如何布局，并且与这个区域外部毫不相干**

### BFC 布局规则

- 内部的 box 会在垂直方向上一个一个放置
- **box 垂直方向的距离由 margin 决定。属于同一个 BFC 的两个相邻 box 的 margin 会发生重叠**
- 每个元素的 margin box 的左边，与包含块的 border box 左边相接触（对于从左到右的格式化，否则相反）。即时存在浮动也是如此
- **BFC 区域不会和 float box 重叠**
- BFC 就是**页面上一个隔离的独立容器，容器内的子元素不会影响到外面的元素，反之也如此**
- **计算 BFC 高度时，浮动元素也参与计算**

### 触发 BFC

- body 根元素
- 浮动元素: float 除 none 以外的值
- 绝对定位元素： position（absolute、fixed）
- display 为 inline-block、table-cells、flex
- overflow 除 visible 以外的值（hidden、auto、scroll）

## BFC 特定和应用

### 同一个 BFC 下外边距发生折叠

```html
<style>
	p{
		width: 50px;
		height: 50px;
		background: black;
		margin: 50px;
	}
</style>
<body>
	<p></p>
	<p></p>
</body>
```

![](http://ony85apla.bkt.clouddn.com/18-4-8/97844767.jpg)

根据 BFC 布局规则
`box 垂直方向的距离由 margin 决定。属于同一个 BFC 的两个相邻 box 的 margin 会发生重叠`

两个 p 元素处于同一个 BFC 容器中（body 元素），因此 p 元素垂直方向上的 margin 会发生重叠。可以在 p 元素包裹一层容器，并触发该容器为 BFC。这样两个 p 元素不在同一个 BFC 中，就不会发生 margin 重叠

```html
<style>
	div{
		overflow: hidden
	}
	p{
		width: 50px;
		height: 50px;
		background: black;
		margin: 50px;
	}
</style>
<body>
	<div><p></p></div>
	<div><p></p></div>
</body>
```

![](http://ony85apla.bkt.clouddn.com/18-4-8/67780074.jpg)

### 清除浮动

浮动元素会脱离文档流

```js
<style>
	div{
		width: 100px;
		border: 10px solid red;
	}
	p{
		width: 50px;
		height: 50px;
		float: left;
		background: green;
	}
</style>
<body>
  <div>
    <p></p>
  </div>
</body>
```

![](http://ony85apla.bkt.clouddn.com/18-4-8/63050099.jpg)

根据 BFC 布局规则

`计算 BFC 高度时，浮动元素也参与计算`

为清除浮动，通过将外层容器触发为 BFC，这样外层容器在计算高度时候，内部的浮动元素也会参加计算

```html
<style>
	div{
		overflow: hidden;
	}
</style>
```

![](http://ony85apla.bkt.clouddn.com/18-4-8/74010944.jpg)


### BFC 阻止元素被浮动元素覆盖

```js
<body>
  <div style="width: 100px;height: 100px;float: left;background: gray; font-size: 10px;">浮动元素</div>
  <div style="width: 150px;height: 200px;background: red;font-size: 10px;">未设置浮动，未触发 BFC 的元素，浮动元素脱离文档流，自己被浮动元素覆盖，但是文字并没有</div>
</body>
```

![](http://ony85apla.bkt.clouddn.com/18-4-8/21131544.jpg)

根据 BFC 布局规则

`每个元素的 margin box 的左边，与包含块的 border box 左边相接触（对于从左到右的格式化，否则相反）。即时存在浮动也是如此`

浮动元素会脱离文档流，第二个元素部分会被浮动元素覆盖(浮动元素的左边依然会和第二个元素的左边接触)，但是文本不会被浮动元素覆盖（形成绕排）

而根据 BFC 布局规则

`BFC 区域不会和 float box 重叠`

想避免被覆盖，可触发第二个元素为 BFC。可以用来实现两列自适应布局

```html
<body>
  <div style="width: 100px;height: 100px;float: left;background: gray; font-size: 10px;">浮动元素</div>
  <div style="width: 150px;height: 200px;overflow: hidden;background: red;font-size: 10px;">触发 BFC 的元素，浮动元素脱离文档流，自己没有被浮动元素覆盖</div>
</body>
```

![](http://ony85apla.bkt.clouddn.com/18-4-8/41849529.jpg)