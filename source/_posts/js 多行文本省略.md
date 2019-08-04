---
title: js 多行文本省略
tag: 
	- js
	- 多行文本省略
---

## 单行文本省略

> 设定宽度，不折行，超出宽度隐藏，超出部分用省略号表示

```javascript
p {
    width: 80%;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
}
```

<!--more-->

less中可以处理mixin

```javascrip
.no-wrap(){
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
}

p {
    width: 80%;
    .no-wrap();
}

```

## 多行文本省略

> 设定高度和行高，利用 父元素的 after伪类 绝对定位 省略图标

```javascript
p {
    position: relative;
    overflow: hidden;
    height: 50px;
    line-height: 2;
}

p:after{
    position: absolute;
    bottom: 0;
    right: 0;
    content: '...';
    font-weight: bold;
    padding: 0 80px 1px 45px;
    // background: url('http://newimg88.b0.upaiyun.com/newimg88/2014/09/ellipsis_bg.png') repeat-y;
    // background: -webkit-linear-gradient(left, transparent, @bgColor @gradient);
    // background: -o-linear-gradient(right, transparent, @bgColor @gradient);
    // background: -moz-linear-gradient(right, transparent, @bgColor @gradient);
    background: linear-gradient(to right, transparent, @bgColor @gradient);
}
```

less中可以处理mixin

```javascript
.multi-no-wrap(@left: 80px, @bgColor: #fff, @gradient: 30%) {
  position: relative;
  overflow: hidden;

  &:after {
    content:"...";
    font-weight:bold;
    position:absolute;
    bottom:0;
    right:0;
    padding:0 @left 1px 45px;
    // background: url('http://newimg88.b0.upaiyun.com/newimg88/2014/09/ellipsis_bg.png') repeat-y;
    // background: -webkit-linear-gradient(left, transparent, @bgColor @gradient);
    // background: -o-linear-gradient(right, transparent, @bgColor @gradient);
    // background: -moz-linear-gradient(right, transparent, @bgColor @gradient);
    background: linear-gradient(to right, transparent, @bgColor @gradient);
  }
}

p{
    height: 50px;
    line-height: 2;
    .multi-no-wrap();
}

```