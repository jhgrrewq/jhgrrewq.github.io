---
title: js arguments
tag: 
	- js
---

> 参考: [Javascript 高级程序设计（第三版）]()、[JavaScript深入之类数组对象与arguments](https://github.com/mqyqingfeng/Blog/issues/14)

<!-- markdownlint-disable MD010 -->

在函数体内能通过 arguments 对象访问参数数组，从而获取传递给参数的每一个函数。arguments 是一个类数组（不是 Array 实例），可以使用方括号语法访问它的每一个元素，使用 length 属性确定传递进来实参参数。arguments 对象还有一个 callee 属性，该属性是一个指针，指向拥有这个 arguments 对象的函数

<!-- more -->

## length 属性

```js
var fn = function(a, b, c) {
  console.log('形参个数：' + fn.length)
  console.log('实参个数：' + arguments.length)
}

fn(1)
// 形参个数：3
// 实参个数：1
fn(1, 2, 3, 4)
// 形参个数：3
// 实参个数：4
```

## 使用数组方法

两种思路，一种是先将 arguments 转为数组再调用数组方法，另外一种是借用 call apply 函数借用数组的方法

```js
function fn(a, b, c) {
  // 转为数组
  // slice
  var arr = Array.prototype.slice.call(arguments)
  // splice
  var arr = Array.prototype.splice.call(arguments, 0)
  // concat
  var arr = Array.prototype.concat.apply([], arguments)
  var arr = Array.prototype.concat.call([], ...arguments)
  // 使用 es6 结构
  var arr = [...arguments]
  // 使用 Array.from
  var arr = Array.from(arguments)

  var map = arr.map(function(value, index) {
    return value * value
  })
}

function fn(a, b, c) {
  // call apply 借用方法
  var map = Array.prototype.map.call(arguments, function(value, index) {
    return value * value
  })
  console.log(map)
}

fn(1, 2, 3) // [1, 4, 9]
```

## callee 属性

递归函数式在一个函数通过名字调用自身的情况下构成。若函数名在外部被重写，将导致错误（函数名是对函数对象的引用）

```js
function factorial(num) {
  if (num < 1) {
    return 1
  } else {
    return num * factorial(num - 1)
  }
}

factorial(5) // 120

var anotherFactorial = factorial
factorial = null
anotherFactorial(5) // Uncaught TypeError: factorial is not a function
```

arguments 对象 callee 属性是个指针，**指向拥有该 arguments 对象的函数, 可用 arguments.callee 代替函数名写递归函数**

```js
function factorial(num) {
  if (num < 1) {
    return 1
  } else {
    return num * arguments.callee(num - 1)
  }
}

factorial(5) // 120

var anotherFactorial = factorial
factorial = null
anotherFactorial(5) // 120
```

在严格模式下，不能通过脚本访问 arguments.callee。可以使用命名函数表达式来实现同样的效果

```js
'use strict';
var factorial = function f(num) {
  if (num < 1) {
    return 1
  } else {
    return num * f(num - 1)
  }
}

factorial(5) // 120

var anotherFactorial = factorial
factorial = null
anotherFactorial(5)
```

## arguments 和对应参数绑定

- **严格模式，实参和 arguments 不会共享**
- **非严格模式下，传入参数时 arguments 实参和会共享，没有传入参数 arguments 和实参不会共享**

```js
// 非严格模式
(function(name, sex) {
  console.log(name, arguments[0]) // jack jack
  // 传入局部变量
  name = 'name set jack'
  console.log(name, arguments[0]) // name set jack name set jack

  // 改变 arguments
  arguments[0] = 'arguments set jack'
  console.log(name, arguments[0]) // arguments set jack arguments set jack

  // 测试没传入参数
  console.log(sex, arguments[1]) // undefined undefined

  sex = 'sex set male'
  console.log(sex, arguments[1]) // sex set male undefined

  arguments[1] = 'arguments set male'
  console.log(sex, arguments[1]) // sex set male arguments set male
})('jack')
```
