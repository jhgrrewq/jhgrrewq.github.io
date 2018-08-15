---
title: less
tag: 
	- less
	- css 预编译
---

## 变量 

~~~javascript
@variable：value
~~~

**url中使用** 或者 **样式名中作为差值变量使用**

~~~javascript
@{variable}

// 如 background: url()
@img: '~assets/imgs/a.png'
div{
    background: url('@{img}');
}

// 如生成 类似后缀 col-1 col-2
~~~

> sass中使用$声明变量，和less一样变量名和变量值中间使用冒号； 
stylus中声明变量没有规定，但是@声明变量，stylus会进行编译，但是不会赋值给变量。因此stylus不要用@声明变量。 styles变量和变量值之间必须使用等号

<!--more-->

## 引用

~~~javascript
@import 'xxx.less'
~~~

绝对路径引用 (需要对应在webpack配置中的路径)

~~~javascript
@import '~xxx/xxx.less'
~~~

## mixin

**定义一些属性规则 一次性导入替换到调用的区块**

~~~javascript
.mixinName{
}

.mixinName(varia){
}
~~~

**对于mixin使用传入当前less所在目录下的文件 xxx.png 需要传入~./xxx.png**

~~~javascript
.no-result-icon{
    width: 86px;
    height: 90px;
    margin: 0 auto;
    .bg-image('~./no-result');
    background-size: 86px 90px;
}
~~~

mixin 还可以返回值，但是**返回值只能初始化一次，之后再调用该 mixin 返回都是第一次初始化的值**

```js
.average(@low, @high) {
    @average: (@low + @high) / 2;
}

div{
    .average(100px, 200px);
    width: @average; // 150px
    .average(10px, 20px); // 重新调用无效
    width: @average; // 150px
}
```

## 嵌套

**& 解析的是同一个元素或此元素的伪类(相当于继承)，没有 & 解析是后代元素 (后面不能有空格) 如&:after  &.footer**

## 约束和循环

less中没有‘if/else’语句， 但是有**when关键字**实现约束，通过mixin，调用自身结合约束来实现循环。常见应用是生成栅格系统

mixin 中设置

```javascript
// n, i 为 mixin 的两个参数， i<=n 为条件
.generate-column(@n, @i: 1) when(@i <= @n) {
    .column-@{i} {                      // i 作为插值变量
        width: (@i * 100% / @n);
    }
    .generate-column(@n, (@i + 1));          // i+1 递归调用自身 mixin
};
```

调用并生成css

```javascript
// 调用
.generate-column(4)

// 生成css 代码
.column-1 { width: 25% }
.column-1 { width: 50% }
.column-1 { width: 75% }
.column-1 { width: 100% }
```

## vue中使用

引入（使用vue-cli构建的项目不需在webpack配置loader）

~~~javascript
npm install less less-loader -D
~~~

组件中使用

~~~javascript
<style lang="less"> </style>
~~~

webstorm中还需要修改为

~~~javascript
<style lang="less" rel="stylesheet/less" type="text/less"> </style>
~~~