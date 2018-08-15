---
title: grid 布局
tag: 
  - grid
---

> 参考: [CSS网格布局（Grid）完全教程](https://juejin.im/entry/5aead92f518825673f0b76b0)

grid 是一套二维的页面布局系统

<!-- more -->

## 网格容器

将属性 `display` 设为 `grid` 或 `inline-grid`, 就创建一个网格容器，其**直接子节点**自动成为网格项目

- 网格项目按行排列，网格项目占据整个容器的宽度

<script src='http://runjs.cn/gist/gyqgyorc/all/rdark'></script>

- 网格项目按行排列，**网格项目由自身宽度决定**

<script src='http://runjs.cn/gist/aydfqrvg/all/rdark'></script>

## 网格轨道

属性 `grid-template-columns` 和 `grid-template-rows` 分别定义列轨道和行轨道

属性 `grid-template-rows` 用于定义行的尺寸，即轨道尺寸。轨道尺寸可以是任何**非负的长度值**（px，%，em，等）, 同理 `grid-template-columns`

<script src='http://runjs.cn/gist/pua7owhh/all/rdark'></script>

- **单位 fr 用于表示轨道尺寸配额，表示按照配比分配可用空间**

```css
.container{
  display: grid;
  grid-template-columns: 1fr 2fr;
}

// 类似 flex 布局的 flex: 1，不过 flex 是在子项目中设置 flex: 1, flex: 2 等
```

<script src='http://runjs.cn/gist/56kpi9x9/all/rdark'></script>

- **单位 fr 和其他长度单位混用时，fr 的计算基于其他单位分配后的剩余空间，常用于设置一列定宽，另一列自适应布局**

```css
.container{
  display: grid;
  grid-template-columns: 100px 1fr 2fr;
}

// 这里 1fr = (容器宽度 - 100px)/3
```

<script src='http://runjs.cn/gist/7ls9dqqu/all/rdark'></script>

## 轨道的最大最小尺寸设置

函数 `minmax()` 用于定义轨道最小最大边界值，接受两个参数，第一个参数表示最小轨道尺寸，第二个参数表示最大轨道参数。**长度值 `auto` 表示轨道尺寸可以根据内容大小拉伸或收缩, 长度值为百分比是相对整个容器宽度**

<script src='http://runjs.cn/gist/hgckhxuf/all/rdark'></script>

## 重复轨道

函数 `repeat()` 定义重复的网格轨道，接受两个参数，第一个参数是重复的个数，第二个参数是轨道的尺寸

```css
grid  {
  display: grid;
  grid-template-columns:    repeat(3, 100px);
｝
```

<script src='http://runjs.cn/gist/zs3clzin/all/rdark'></script>

## 网格间隙

属性 `grid-column-gap` 和 `grid-row-gap` 定义网格间隙

- **网格间隙只能创建在行列之间**，项目与边界之间无间隙
- 间隙尺寸可以任何非符的长度值
- 属性 `grid-gap` 是 `grid-row-gap`  和 `grid-column-gap` 的简写形式, 第一个值是行间隙，第二个值是列间隙, 只有一个值表示行间隙和列边界

<script src='http://runjs.cn/gist/37dnx1xb/all/rdark'></script>

## 网格线编号定位项目

网格线表示网格轨道的开始和结束，**每一条网格线编号从 1 开始，以 1 为步长向前编号，包括行列两组网格线**

- 项目默认只跨一行一列
- 属性 `grid-row` 是 `grid-row-start` 和 `grid-row-end` 的简写，如果至指定一个值，表示 `grid-row-start`；如果指定两个值，第一个值是 `grid-row-start`，第二个值是 `grid-row-end`，并且需要用 `/` 连接。同理 `grid-column`
- 属性 `grid-area` 是 `grid-row-start`, `grid-column-start`, `grid-row-end` 和 `grid-column-end` 的简写, 如果四个值制定，第一个值是 `grid-row-start`, 第二个值是 `grid-column-start`, 第三个值是 `grid-row-end`，第四个值是 `grid-column-end`
- 关键字 `span` 可以用来指定跨越的行或列数量

```css
.container{
  display: grid;
}
.item1{
  /* 从第 1 个列网格线开始，到第三个网格线结束，跨 2 列 */
  grid-column-start: 1;
  grid-column-end: 3;
}
.item2{
  /* 从第 3 个列网格线开始，默认跨 1 列，从第 1 个行网格线开始到第 3 个行网格线结束，跨 2 行 */
  grid-column: 3;
  grid-row: 1/3;
}
.item3 {
  /* 从第 1 个列网格线开始，使用 span 关键字，跨 2 列，从第 2 个行网格线开始到第 3 个行网格线结束，跨 1 行 */
  grid-column: 1 / span 2;
  grid-row: 2/3;
}
.item4 {
  /* 从第 1 个列网格线开始，到第 3 个网格线结束，跨 2 列，从第 3 个行网格线开始到第 4 个行网格线结束，跨 1 行 */
  grid-area: 3/1/4/3;
}
```

<script src='http://runjs.cn/gist/ecdot9g6/all/rdark'></script>

## 网格区域命名和定位项目

使用 `grid-template-areas` 给网格区域命名，网格区域名称可以用来定位项目

- **一组网格区域名称需要放在引号中，每个名称之间以空格间隔，每一组也用空格间隔**
- **每一组名称定义一行，每个名称定义一列**
- **网格名称可以在 `grid-row-start` `grid-row-end` `grid-column-start` `grid-column-end` 值中用于定位项目，或者在简写属性 `grid-row` `grid-column` `grid-area` 中**

```css
.container{
  display: grid;
  grid-template-rows: 50px 100px 50px;
  grid-template-columns: 100px 100px 100px;
  grid-template-areas: 'header header header' 'left center right' 'footer footer footer';
}
.header{
  /* 等同 grid-area: header; */
  grid-row-start: header;
  grid-row-end: header;
  grid-column-start: header;
  grid-column-end: header;
}
.left{
  grid-area: left;
}
.center {
  grid-area: center;
}
.right {
  grid-area: right;
}
.footer{
  /* 等同 grid-area: header; */
  grid-row: footer;
  grid-column: footer;
}
```

<script src='http://runjs.cn/gist/atcqmbnn/all/rdark'></script>

## 层叠网格项目

通过项目定位可以使项目层叠，使用属性 `z-index` 可以改变层叠项目的层次

## 网格项目对齐方式

类似 flex，grid 中网格项目可以根据行或列的轴线方向实现多种对齐方式

- 属性 `justify-items` `justify-self` 以行轴为参照对齐项目，属性 `align-items` `align-self` 以行轴为参照对齐项目
- 属性 `justify-items` `align-items` 是容器属性，并支持如下值： `auto` `normal` `start` `end` `center` `stretch` `baseline` `first baseline` `last baseline`

```css
.container{
  display: grid;
  grid-template-rows: 80px 80px;
  grid-template-columns: 1fr 1fr;
  grid-template-areas: "content content"
                "content content";
}
.item {
  grid-area: content;
}
.container1{
  /* 在行轴线起点处对齐 */
  justify-items: start;
}
.container2{
  /* 在行轴线中点处对齐 */
  justify-items: center;
}
.container3{
  /* 在行轴线终点处对齐 */
  justify-items: end;
}
.container4{
  /* 在行轴线方向延伸并铺满整个区域，stretch 是默认值 */
  justify-items: stretch;
}
.container5 {
  /* 项目定位于行轴和列轴线的中间位置，可实现水平垂直居中 */
  justify-items: center;
  align-items: center;
}
```

<script src='http://runjs.cn/gist/lxln089j/all/rdark'></script>

- 属性 `justify-self` `align-self` 是项目自身属性，并支持如下值： `auto` `normal` `start` `end` `center` `stretch` `baseline` `first baseline` `last baseline`

```css
.container{
  display: grid;
  grid-template-rows: 80px 80px;
  grid-template-columns: 1fr 1fr;
  grid-template-areas: "content content"
                "content content";
}
.item {
  grid-area: content;
}
.item1{
  /* 属性 justify-self 定义项目自身在行的轴线方向定义对齐方式 */
  /* 属性 align-self 定义项目自身在列的轴线方向定义对齐方式 */
  justify-self: start;
}
.item2{
  justify-self: center;
}
.item3{
  justify-self: end;
}
.item4{
  justify-self: stretch;
}
.item5{
  justify-self: center;
  align-self: center;
}
```

<script src='http://runjs.cn/gist/ntpu4iqx/all/rdark'></script>