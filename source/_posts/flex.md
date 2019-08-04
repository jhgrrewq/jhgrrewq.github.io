---
title: flex 布局
tag: 
	- css3
	- flex
---

> 参考: [Flex 布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

<!-- markdownlint-disable MD010 -->

## flex 弹性布局

![](http://ony85apla.bkt.clouddn.com/18-6-2/9723494.jpg)

<!-- more -->

任何容器都可设置弹性布局，行内元素也可。webkit 内核浏览器要加上 `-webkit` 前缀

- **设置 flex 布局后，子元素的 float clear vertical-align 属性失效**

```css
.box {
	display: flex;
	// display: inline-flex;
	display: -webkit-flex; // safari
}
```

## 基本概念

![](http://ony85apla.bkt.clouddn.com/18-6-2/94929667.jpg)

- `flex 容器（flex container）` 采用 flex 布局的元素
- `flex 项目（flex item）` 所有子元素自动成为容器成员
- 容器有两个主轴，水平主轴（main axis）和 垂直交叉轴（cross axis），主轴开始位置（和容器交叉点）为 `main start`，结束位置为 `main end`；交叉轴开始位置（和容器交叉点）为 `cross start`，结束位置为 `cross end`
- **项目默认沿主轴排列**，单个项目占据主轴空间叫 `main size`，占据交叉轴空间叫 `cross size

## 容器属性

容器有 6 个属性：

- `flex-direction`
- `flex-wrap`
- `flex-flow`
- `justify-content`
- `align-items`
- `align-content`

### flex-content (决定主轴顺序，就是项目的排列顺序)

```css
.box {
	flex-direction: row | row-reverse | culumn | column-reverse
}
```

- `flex-direction: row 和**不设置（默认）**` 横向从左到右

![](http://ony85apla.bkt.clouddn.com/18-6-2/91973021.jpg)

- `flex-direction: row-reverse` 横向从右到左

![](http://ony85apla.bkt.clouddn.com/18-6-2/41872058.jpg)

- `flex-direction: column` 纵向从上到下

![](http://ony85apla.bkt.clouddn.com/18-6-2/30643464.jpg)

- `flex-direction: column` 纵向从下到上

![](http://ony85apla.bkt.clouddn.com/18-6-2/85540885.jpg)

### flex-wrap (定义一条轴线排列不下，超出如何换行)

```css
.box {
	flex-wrap: nowrap | wrap | wrap-reverse
}
```

- `flex-wrap: nowrap` **默认不换行**

![](http://ony85apla.bkt.clouddn.com/18-6-2/89652330.jpg)

- `flex-wrap: wrap` 换行

![](http://ony85apla.bkt.clouddn.com/18-6-2/2977256.jpg)

- `flex-wrap: wrap-reverse` 换行相反方向

![](http://ony85apla.bkt.clouddn.com/18-6-2/60761591.jpg)

### flex-flow（flex-direction 和 flex-wrap 简写，默认 row nowrap）

```css
.box {
	flex-flow: <flex-direction> || <flex-wrap>
}
```

### justify-content（定义项目在主轴上的对齐方式）

```css
.box {
	justify-content: flex-start | flex-end | center | space-between | space-around
}
```

假设主轴从左到右

- `justify-content: flex-start` **默认左对齐**

![](http://ony85apla.bkt.clouddn.com/18-6-2/12496932.jpg)

- `justify-content: flex-end` 右对齐

![](http://ony85apla.bkt.clouddn.com/18-6-2/43709152.jpg)

- `justify-content: center` 居中

![](http://ony85apla.bkt.clouddn.com/18-6-2/75426758.jpg)

- `justify-content: space-between` **两端对齐，项目之间间隔相等**

![](http://ony85apla.bkt.clouddn.com/18-6-2/72218223.jpg)

- `justify-content: space-around` **项目两侧间隔相等**

![](http://ony85apla.bkt.clouddn.com/18-6-2/84745699.jpg)

### align-items (定义项目在交叉轴上的对齐方式)

```css
.box {
	align-items: flex-start | flex-end | center | baseline | stretch
}
```

假设交叉轴从上到下

- `align-items: flex-start` 交叉轴起点对齐

![](http://ony85apla.bkt.clouddn.com/18-6-2/41368961.jpg)

- `align-items: center` 交叉轴中点对齐

![](http://ony85apla.bkt.clouddn.com/18-6-2/64584779.jpg)

- `align-items: flex-end` 交叉轴终点对齐

![](http://ony85apla.bkt.clouddn.com/18-6-2/60046906.jpg)

- `align-items: baseline` 交叉轴第一行文字基线对齐

![](http://ony85apla.bkt.clouddn.com/18-6-2/5947862.jpg)

- `align-items: stretch` 如果项目没有设置高度或者设为 `auto`，将占据整个容器高度

![](http://ony85apla.bkt.clouddn.com/18-6-2/12165396.jpg)

### align-content (定义对根轴线的对齐方式，如果项目只有一根轴线则不起作用)

```css
.box {
	align-content: flex-start | flex-end | center | space-between | space-around
}
```

假设交叉轴从上到下

- `align-content: flex-start` 交叉轴起点对齐

![](http://ony85apla.bkt.clouddn.com/18-6-2/93159124.jpg)

- `align-content: center` 交叉轴中点对齐

![](http://ony85apla.bkt.clouddn.com/18-6-2/48709014.jpg)

- `align-content: flex-end` 交叉轴终点对齐

![](http://ony85apla.bkt.clouddn.com/18-6-2/79772444.jpg)

- `align-content: space-between` 交叉轴两端能对齐，轴线之间间隔平均分布

![](http://ony85apla.bkt.clouddn.com/18-6-2/87105794.jpg)

- `align-content: space-around` 每条轴线两侧间隔都相等

![](http://ony85apla.bkt.clouddn.com/18-6-2/93742673.jpg)

- `align-content: stretch` 轴线占满整个交叉轴

![](http://ony85apla.bkt.clouddn.com/18-6-2/96242810.jpg)

## 项目属性

项目有 6 个属性

- `order`
- `flex-grow`
- `flex-shrink`
- `flex-basis`
- `flex`
- `align-self`

### order (定义项目的排列顺序，数值越小越靠前，默认为 0)

```css
.item {
	order: <interger>
}
```

![](http://ony85apla.bkt.clouddn.com/18-6-2/91731496.jpg)

### flex-grow (定义项目的放大比例，默认为 0，即如果存在剩余空间也不放大)

```css
.item {
	flex-grow: <interger> // default: 0
}
```

- **如果所有项目的 `flex-grow` 为 1, 等分剩余空间**

![](http://ony85apla.bkt.clouddn.com/18-6-2/50544174.jpg)

- **如果一个项目的 `flex-grow` 为 2, 其他为 1，前者占据的剩余空间将比其他的多一半，其他情况一次类推**

![](http://ony85apla.bkt.clouddn.com/18-6-2/30099558.jpg)

### flex-shrink (定义项目的缩小比例，默认为 1，即如果空间不足将缩小)

```css
.item {
	flex-shrink: <interger> // default: 1
}
```

- **如果所有项目的 `flex-shrink` 为 1, 当空间不足将等比缩小**

- **如果一个项目的 `flex-shrink` 为 0, 其他为 1，如果空间不足，前者不缩小**

![](http://ony85apla.bkt.clouddn.com/18-6-2/60049657.jpg)

### flex-basis (分配多余空间之前项目占据株洲空间，默认值 auto，也就是项目的本来大小)

```css
.item {
	flex-basis: <interger> | auto // default: auto
}
```

- **可设置 `width` `height` 属性一样的值（如 350px）,则项目将占据固定空间**

### flex (flex-grow/flex-shrink/flex-basis 简写，默认是 0 1 auto， 后两个属性可选)

- **建议优先使用该属性，而不是写三个分离的属性**
- **`flex: auto` 等同于 1 1 auto**
- **`flex: none` 等同于 0 0 auto**

### align-self (允许单个项目和其他项目不同的对齐方式，可覆盖容器设置的 `align-items`, 默认为 auto，表示继承父元素的 `align-items`, 若没有父元素等同 `stretch`)

```css
.item {
	align-self: auto | flex-start | flex-end | center | baseline | stretch
}
```

![](http://ony85apla.bkt.clouddn.com/18-6-2/92816364.jpg)