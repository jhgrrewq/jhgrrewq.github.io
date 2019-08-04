---
title: css 布局
tag: 
	- css
---

> 参考: [各种页面常见布局+知名网站实例分析+相关阅读推荐](https://github.com/Sweet-KK/css-layout)

<!-- markdownlint-disable MD010 -->

## 水平居中

### 文本/行内/行内块级

- text-align 属性只能控制（文本、行内元素、行内块级元素）如何相对父元素对齐

```css
#parent{
  text-align: center;
}
```

### 单个块级元素

- margin 有剩余的情况下如果左右 margin 设置 auto，将平分剩余空间

```css
#son{
  width: 100px; /* 必须定宽, 并且宽度必须小于父元素宽 */
  margin: 0 auto;
}
```

### 多个块级元素

- text-align 属性只能控制（文本、行内元素、行内块级元素）如何相对父元素对齐; 子元素改为 行内 或者 行内块级 元素，让其对 text-align 生效

```css
#parent {
  text-align: center;
}
.son {
  display: inline-block;
}
```

<!-- more -->

### 绝对定位实现

- 子元素绝对定位父元素相对定位，top/left/right/bottom 值相对父元素，margin 或 transform 值相对于自身
- 使用 margin-left 需要知道宽度，使用 transform 兼容性不好（ie9+）

```css
#parent4 {
  width: 200px; /* 定宽 */
  height: 200px;
  position: relative; /* 父元素相对定位 */
  background: red;
}
#son {
  width: 30px; /* 定宽 */
  height: 30px;
  position: absolute; /* 子元素绝对定位 */
  left: 50%; /* 父元素宽度一半，这里相当于 left: 100px; */
  transform: translateX(-50%); /* 自身宽度一半，这里相当于 margin-left: 15px; */
  background: blue;
}
```

### 任意个元素(flex)

- 设置当前主轴对齐方式为居中， pc 端兼容性较差（ie10+）, 移动端支持相对较好（Android 4.0+）

```css
#parent{
  display: flex;
  justify-content: center;
}
```

### 总结

- 对于水平居中，优先考虑哪些元素带有自动居中属性。对于 text-align: center, 只对 行内元素有效，因此必须设置子元素 display: inline 或 display: inline-block

- 其次是块级元素要优先考虑 margin: 0 auto, 其次才是绝对定位实现

- 移动端优先考虑 flex

## 垂直居中

### 单行文本/行内元素/行内块级元素

- **line-heght 最终表现是通过 inline box 实现，无论 inline-box 高度到少（比文字大还是比文字小），占据空间是与文字内容水平中垂线垂直，多出文字部分会平均分在文字的上下**

```css
#parent{
  height: 150px;
  line-height: 150px;  /* 必须知道高度，与height等值 */
}
```

### 多行文本/行内元素/行内块级元素

- 同上

```css
#parent{
  height: 150px;
  line-height: 30px;  /* 元素在页面呈现为5行,则line-height的值为 height/5 */
}
```

- 或者使用 padding

```css
#parent{
  height: 150px;
  padding: 20px;
}
```

### 图片

- [CSS深入理解vertical-align和line-height的基友关系](http://www.zhangxinxu.com/wordpress/2015/08/css-deep-understand-vertical-align-and-line-height/)

```css
#parent3 {
  height: 150;
  line-height: 150px;
  font-size: 0; /*父元素需要设置 font-size: 0; 因为子元素为 inline 或 inline-block 时，子元素之间可能存在空格或者换行*/
}
img#son3 {
  vertical-align: center; /* 默认基线对齐，改为 middle */
}
```

### 单个块级元素

#### table 布局

- 表格内容自动垂直居中，无需设置宽高
- 表格容易造成浏览器回流和重绘制，现在主要只做表格使用，而不是用来布局

```html
<table>
  <tr>
    <td>adfasdfasdf</td>
  </tr>
</table>
```

#### table-cell

- css table, 使表格内容对齐方式为 middle
- 不需知道宽高

```css
#parent {
  display: table-cell;
  vertical-align: middle;
}
```

#### 绝对定位实现

- 子元素绝对定位父元素相对定位，top/left/right/bottom 值相对父元素，margin 或 transform 值相对于自身
- 使用 margin-top 需要知道宽度，使用 transform 兼容性不好（ie9+）

```css
#parent4 {
  width: 200px;
  height: 200px; /* 定高 */
  position: relative; /* 父元素相对定位 */
  background: red;
}
#son {
  width: 30px;
  height: 30px; /* 定高 */
  position: absolute; /* 子元素绝对定位 */
  top: 50%; /* 父元素高度一半，这里相当于 top: 100px; */
  transform: translateY(-50%); /* 自身高度一半，这里相当于 margin-top: 15px; */
  background: blue;
}
```

- 绝对定位 + margin: auto 0: 当top、bottom 为 0 时,margin-top & bottom 会无限延伸占满空间并且平分

```css
#parent{position: relative;}
#son{
  position: absolute;
  margin: auto 0;
  top: 0;
  bottom: 0;
  height: 50px;
}
```

#### flex

```css
#parent{
  display: flex;
}
.son {
  align-self: center;
}
```

### 任意个元素(flex)

- 设置 flex 对齐方式， pc 端兼容性较差（ie10+）, 移动端支持相对较好（Android 4.0+）

```css
#parent{
  display: flex;
  align-items: center;
}

/* 或者 改变排列方向*/
#parent{
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* 或者*/
#parent{
  display: flex;
}
.son {
  align-self: center;
}
```

### 总结

- 对于垂直居中，优先考虑 行内元素的 line-height
- 其次考虑能否使用 vertical-align: center
- 然后是绝对定位实现
- 移动端兼容性允许情况下使用 flex

## 水平垂直居中

## 两列布局

### 两列等高

#### 负 margin-bottom + padding-bottom

- **两栏设置浮动，父容器触发 bfc（等同于最高一列高度），两列 padding-bottom 相等，负 margin-bottom 和 padding-bottom 相等就是最高列内容区高度**

```html
<body>
  <div id="left">左列</div>
  <div id="right">右列</div>
</body>
```

```css
#parent1 {
  overflow: hidden;
  background: green;
}
#left1 {
  float: left;
  width: 100px;
  padding-bottom: 999px;
  margin-bottom: -999px;
  background: red;
}
#right1 {
  float: left;
  width: 100px;
  padding-bottom: 999px;
  margin-bottom: -999px;
  background: blue;
}
```

### 左列定宽，右列自适应（右列定宽，左列自适应）

#### float + margin 实现

```html
<body>
  <div id="left">左列定宽</div>
  <div id="right">右列自适应</div>
</body>
```

```css
#left {
  background-color: red;
  float: left;
  width: 100px;
  height: 500px;
}
#right {
  background-color: blue;
  height: 500px;
  margin-left: 100px; /* 大于等于#left的宽度 */
}
```

#### float + overflow 实现

- **右列无需关注宽度，通过 overflow 触发右列形成 BFC**

```html
<body>
  <div id="left">左列定宽</div>
  <div id="right">右列自适应</div>
</body>
```

```css
#left {
  background-color: red;
  float: left;
  width: 100px;
  height: 500px;
}
#right {
  background-color: blue;
  height: 500px;
  overflow: hidden; /* 将右列触发成为 BFC */
}
```

#### table

- 无需关注定宽的宽度，利用单元格自动分配形成自适应

```html
<body>
  <div id="parent">
    <div id="left">左列定宽</div>
    <div id="right">右列自适应</div>
  </div>
</body>
```

```css
#parent{
  display: table;
  width: 100%;
  height: 500px;
}
#left {
  width: 100px;
  background-color: red;
}
#right {
  background-color: blue;
}
#left,#right{
  display: table-cell;  /* 利用单元格自动分配宽度 */
}
```

#### 绝对定位

```html
<body>
  <div id="parent">
    <div id="left">左列定宽</div>
    <div id="right">右列自适应</div>
  </div>
</body>
```

```css
#parent{
  position: relative;  /*子绝父相*/
}
#left {
  position: absolute;
  top: 0;
  left: 0;
  background-color: red;
  width: 100px;
  height: 500px;
}
#right {
  position: absolute;
  top: 0;
  left: 100px;  /* 值大于等于#left的宽度 */
  right: 0;
  /* width: 100% */
  background-color: blue;
  height: 500px;
}
```

#### flex

- 定宽设置 `flex: 0 0 width`, 自适应列设置 `flex: 1` 均分父元素剩余空间

```html
<body>
  <div id="parent">
    <div id="left">左列定宽</div>
    <div id="right">右列自适应</div>
  </div>
</body>
```

```css
#parent{
  width: 100%;
  height: 500px;
  display: flex;
}
#left {
  flex: 0 0 100px;
  width: 100px;
  background-color: red;
}
#right {
  flex: 1; /* 均分了父元素剩余空间 */
  background-color: blue;
}
```

#### grid

```html
<body>
  <div id="parent">
    <div id="left">左列定宽</div>
    <div id="right">右列自适应</div>
  </div>
</body>
```

```css
#parent {
  width: 100%;
  height: 100px;
  display: grid;
  grid-template-columns: 100px 1fr; /* 设定2列, auto 也可换成 1fr */
  /* grid-template-columns: 100px auto; */
}
#left {
  background: red;
}
#right {
  background: blue;
}
```

### 左列不定宽随内容变化，右列自适应（左列自适应，右列不定宽同理）

#### float + overflow 实现

- **左列只设置浮动不设宽度，右列无需关注宽度，通过 overflow 触发右列形成 BFC**

```html
<body>
  <div id="left">左列定宽</div>
  <div id="right">右列自适应</div>
</body>
```

```css
#left {
  background-color: red;
  float: left; /* 设置浮动不设置宽度 */
  height: 500px;
}
#right {
  background-color: blue;
  height: 500px;
  overflow: hidden; /* 将右列触发成为 BFC */
}
```

#### flex

- 左列不设宽度, 自适应列设置 `flex: 1` 均分父元素剩余空间

```html
<body>
  <div id="parent">
    <div id="left">左列定宽</div>
    <div id="right">右列自适应</div>
  </div>
</body>
```

```css
#parent{
  width: 100%;
  height: 500px;
  display: flex;
}
#left {
  background-color: red;
}
#right {
  flex: 1; /* 均分了父元素剩余空间 */
  background-color: blue;
}
```

#### grid

```html
<body>
  <div id="parent">
    <div id="left">左列定宽</div>
    <div id="right">右列自适应</div>
  </div>
</body>
```

```css
#parent {
  display: grid;
  grid-template-columns: auto 1fr; /* auto 和 1fr 换顺序就是左列自适应, 右列不定宽 */
  height: 100px;
}
#left {
  background: red;
}
#right {
  background: blue;
}
```

## 三列布局（两侧定宽，中间自适应）

### table 实现

```html
<body>
<div id="parent">
  <div id="left">左列定宽</div>
  <div id="center">中间自适应</div>
  <div id="right">右列定宽</div>
</div>
</body>
```

```css
#parent {
    width: 100%;
    height: 500px;
    display: table;
}
#left, #right, #center {
  display: table-cell;
}
#left {
  width: 100px;
  background-color: red;
}
#center {
  background-color: green;
}
#right {
  width: 200px;
  background-color: blue;
}
```

### flex 实现

```html
<body>
<div id="parent">
    <div id="left">左列定宽</div>
    <div id="center">中间自适应</div>
    <div id="right">右列定宽</div>
</div>
</body>
```

```css
#parent {
  height: 100px;
  display: flex;
}
#left {
  width: 100px;
  background-color: red;
}
#center {
  flex: 1;  /* 均分 #parent 剩余的部分 */
  background-color: green;
}
#right {
  width: 200px;
  background-color: blue;
}
```

### position 实现

```html
<body>
<div id="parent">
    <div id="left">左列定宽</div>
    <div id="center">中间自适应</div>
    <div id="right">右列定宽</div>
</div>
</body>
```

```css
#parent {
  position: relative; /*子绝父相*/
}
#left {
  position: absolute;
  top: 0;
  left: 0;
  width: 100px;
  height: 100px;
  background-color: red;
}
#center {
  height: 100px;
  margin-left: 100px; /* 大于等于 #left 的宽度,或者给 #parent 添加同样大小的 padding-left */
  margin-right: 200px;  /* 大于等于 #right 的宽度,或者给 #parent 添加同样大小的 padding-right */
  background-color: green;
}
#right {
  position: absolute;
  top: 0;
  right: 0;
  width: 200px;
  height: 100px;
  background-color: blue;
}
```

### grid

```html
<body>
<div id="parent">
    <div id="left">左列定宽</div>
    <div id="center">中间自适应</div>
    <div id="right">右列定宽</div>
</div>
</body>
```

```css
#parent {
  display: grid;
  grid-template-columns: 100px 1fr 200px; /* 1fr 也可换成 auto */
  height: 100px;
}
#left {
  background: red;
}
#right {
  background: blue;
}
#center {
  background: green;
}
```

### 双飞翼 (中间部分优先渲染)

- html 中 center 放在前面，其次是 left right
- left center right 设置 `float: left`
- center 设置 `width: 100%`，占满整个整行
- left 定宽，并且需要使用 **负外边距**，设 `margin-left: -100%;` (将 left 挤到 center 同一行，并盖在 center 的左边并与 center 左对齐 )
- right 定宽，并且需要使用 **负外边距**，设 `margin-left: -200px` (将 right 挤到 center 同一行，负左边距值等于自身宽度 )
- center 左右两边被盖住, center_box 设置外边距 margin-left margin-right 分别等于左右两边的宽度

```html
<body>
<div id="header"></div>
<div id="parent">
  <!--中间栏需要放在前面-->
  <div id="center">
    <div id="center_box"></div>
  </div>
  <div id="left">左列定宽</div>
  <div id="right">右列定宽</div>
</div>
<div id="footer"></div>
</body>
```

```css
#header {
  height: 60px;
  background-color: gray;
}
#left {
  float: left;
  width: 100px;
  height: 100px;
  margin-left: -100%; /* 调整 #left 位置, 值等于自身宽度 */
  background-color: red;
}
#center {
  height: 100px;
  float: left;
  width: 100%;
  background-color: green;
}
#right {
  float: left;
  width: 200px;
  height: 100px;
  margin-left: -200px;  /* 使得 #right 到指定的位置, 值等于自身宽度 */
  background-color: blue;
}
#footer {
  clear: both;  /* 注意清除浮动 */
  height: 60px;
  background-color: gray;
}
```

### 圣杯布局（中间部分优先渲染）

- html 中 center 放在前面，其次是 left right
- left center right 设置 `float: left`
- center 设置 `width: 100%`，占满整个整行
- left 定宽，并且需要使用 **负外边距**，设 `margin-left: -100%;` (将 left 挤到 center 同一行，并盖在 center 的左边并与 center 左对齐 )
- right 定宽，并且需要使用 **负外边距**，设 `margin-left: -200px` (将 right 挤到 center 同一行，负左边距值等于自身宽度 )
- 之后不同于双飞翼，parent（left right center 父元素）设置 `padding: 0 200px 0 100px;`
- center 位置已正确，left 利用相对定位 `position: relative; left: -100px`(相对此时位置 left 偏移等于自身宽度)
- center 位置已正确，right 利用相对定位 `position: relative; right: -200px`(相对此时位置 right 偏移等于自身宽度)

```html
<body>
<div id="header"></div>
<div id="parent">
  <!--中间栏需要放在前面-->
  <div id="center"></div>
  <div id="left">左列定宽</div>
  <div id="right">右列定宽</div>
</div>
<div id="footer"></div>
</body>
```

```css
#header {
  height: 60px;
  background-color: gray;
}
#parent {
  height: 100px;
  padding: 0 200px 0 100px;  /* 为了使#center摆正,左右 padding 分别等于左右盒子的宽,可以结合左右盒子相对定位的left 调整间距*/
}
#left {
  margin-left: -100%;  /*使#left上去一行*/
  position: relative;
  left: -100px;  /*相对定位调整#left的位置,正值大于或等于自身宽度*/
  float: left;
  width: 100px;
  height: 100px;
  background-color: red;
}
#center {
  float: left;
  width: 100%;  /*由于#parent的padding,达到自适应的目的*/
  height: 100px;
  background-color: green;
}
#right {
  position: relative;
  left: 200px; /*相对定位调整#right的位置,大于或等于自身宽度*/
  width: 200px;
  height: 100px;
  margin-left: -200px;  /*使#right上去一行*/
  float: left;
  background-color: blue;
}
#footer{
  height: 60px;
  background-color: gray;
}
```

### grid （上中下，中栏两侧定宽，中间自适应）

```html
<div id="parent">
  <div id="header"></div>
  <div id="left">左列定宽</div>
  <div id="center">中间自适应</div>
  <div id="right">右列定宽</div>
  <div id="footer"></div>
</div>
```

```css
#parent {
  display: grid;
  grid-template-columns: 100px auto 200px;
  grid-template-rows: 60px 100px 60px;
  grid-template-areas:
    'header header header'
    'leftside main rightside'
    'footer footer footer';
}
#header {
  grid-area: header;
  background: gray;
}
#footer {
  grid-area: footer;
  background: gray;
}
#left {
  grid-area: leftside;
  background: red;
}
#center {
  grid-area: main;
  background: green;
}
#right {
  grid-area: rightside;
  background: blue;
}
```

## 多列等宽

### float

- 需要知道列数并设置 width 百分比

```html
<body>
<div id="parent">
  <div class="column">1 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
  <div class="column">2 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
  <div class="column">3 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
  <div class="column">4 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
</div>
</body>
```

```css
.column{
  width: 25%;
  float: left;
  height: 100px;
}
.column:nth-child(odd){
  background-color: red;
}
.column:nth-child(even){
  background-color: blue;
}
```

### table

- 需要知道列数并设置 width 百分比

```html
<body>
<div id="parent">
  <div class="column">1 <p>我是文字我啊打发奥所所所所所所所所所是文字我输文字我是文字我是文字</p></div>
  <div class="column">2 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
  <div class="column">3 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
  <div class="column">4 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
</div>
</body>
```

```css
#parent {
  width: 100%;
  display: table;
}
.column{
  display: table-cell;
  width: 25%;
  height: 100px;
}
.column:nth-child(odd){
  background-color: red;
}
.column:nth-child(even){
  background-color: blue;
}
```

### flex

```html
<body>
<div id="parent">
  <div class="column">1 <p>我是文字我啊打发奥所所所所所所所所所是文字我输文字我是文字我是文字</p></div>
  <div class="column">2 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
  <div class="column">3 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
  <div class="column">4 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
</div>
</body>
```

```css
#parent {
  display: flex;
  width: 100%;
  height: 100px;
}
.column{
  flex: 1;
}
.column:nth-child(odd){
  background-color: red;
}
.column:nth-child(even){
  background-color: blue;
}
```

### grid

- 需要知道列数

```html
<body>
<div id="parent">
  <div class="column">1 <p>我是文字我啊打发奥所所所所所所所所所是文字我输文字我是文字我是文字</p></div>
  <div class="column">2 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
  <div class="column">3 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
  <div class="column">4 <p>我是文字我是文字我输文字我是文字我是文字</p></div>
</div>
</body>
```

```css
#parent {
  width: 100%;
  display: grid;
  grid-template-columns: repeat(4, 1fr); /* 4 为列数 */
}
.column:nth-child(odd){
  background-color: red;
}
.column:nth-child(even){
  background-color: blue;
}
```

