---
title: js call apply 模拟实现
tag: 
	- js
---

> 参考:[JavaScript深入之call和apply的模拟实现](https://github.com/mqyqingfeng/Blog/issues/11)

<!-- narkdownlint-disable MD010 -->

## 原理

```js
var a = 2
var foo = {
  a: 1
}
var bar = function(name) {
  console.log(this.a)
  console.log(name)
}

bar.call(foo, 'jack') // 1 'jack'
bar.call(null, 'kevin') // 2 'kevin'

// 函数也是可以有返回值
var bar = function(name) {
  return {
    a: this.a,
    name: name
  }
}

var p1 = bar.call(foo, 'jack') // {a:1, name:'jack'}
var p2 = bar.call(null, 'kevin') // {a:2,name:'kevin'}

```

当前对象没有某个方法，借用 call 和 apply 其他对象的方法，改变函数 this 指向传入的对象，使当前对象也能调用该方法

模拟实现需要三个步骤：

- **将借用的方法设为当前对象的属性**
- **当前对象调用该方法**
- **删除该属性**
- **有函数返回值就返回**

## call 模拟实现

该方法能被函数调用，需要挂载在 Function 原型上

```js
// 方法必须是挂载在 Function 原型上，才能被所有函数实例调用
Function.prototype.newCall = function (ctx) {
  // ctx 为传入的当前对象 ctx 不传或者传 undefined null 则为 window
  ctx = ctx || window
  // 通过 this 拿到该借用的方法
  // 1 首先该方法设为当前对象的属性
  ctx.fn = this
  // 取出需要传入的参数 从第二个参数开始
  let args = []
  for (let i = 1; i < arguments.length; i++) {
    args[i - 1] = arguments[i]
  }
  // let args = Array.prototype.slice.call(arguments, 1)
  // 2 调用该方法 使用 es6 解构传入参数
  var ret
  if (args && args.length > 0) {
    ret = ctx.fn(...args)
  } else {
    ret = ctx.fn()
  }
  // 3 删除该属性
  delete ctx.fn
  // 返回函数返回值
  return ret
}

var a = 2
var foo = {
  a: 1
}
var bar = function(name) {
  console.log(this.a)
  console.log(name)
}

bar.newCall(foo, 'jack') // 1 'jack'
bar.newCall(null, 'kevin') // 2 'kevin'

var bar = function(name) {
  return {
    a: this.a,
    name: name
  }
}

var p1 = bar.newCall(foo, 'jack') // {a:1, name:'jack'}
var p2 = bar.newCall(null, 'kevin') // {a:2, name:'kevin'}
```

## apply 模拟实现

apply 的实现类似 call，第二个参数是参数数组

```js
// 方法必须是挂载在 Function 原型上，才能被所有函数实例调用
Function.prototype.newApply = function (ctx, arr) {
  // ctx 为传入的当前对象 ctx 不传或者传 undefined null 则为 window
  ctx = ctx || window
  // 通过 this 拿到该借用的方法
  // 1 首先该方法设为当前对象的属性
  ctx.fn = this
  // 2 调用该方法 使用 es6 解构传入参数
  var ret
  if (arr && arr.length > 0) {
    ret = ctx.fn(...arr)
  } else {
    ret = ctx.fn()
  }
  // 3 删除该属性
  delete ctx.fn
  // 返回函数返回值
  return ret
}

var a = 2
var foo = {
  a: 1
}
var bar = function(name) {
  console.log(this.a)
  console.log(name)
}

bar.newApply(foo, ['jack']) // 1 'jack'
bar.newApply(null, ['kevin']) // 2 'kevin'

var bar = function(name) {
  return {
    a: this.a,
    name: name
  }
}

var p1 = bar.newApply(foo, ['jack']) // {a:1, name:'jack'}
var p2 = bar.newApply(null, ['kevin']) // {a:2, name:'kevin'}
```