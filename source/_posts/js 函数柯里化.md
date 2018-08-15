---
title: js 函数柯里化
tag: 
	- js
---

> 参考: [前端高手必备：详解 JavaScript 柯里化](https://juejin.im/entry/58b316d78d6d810058678579/)

## 利用 call/apply 封装 map

map 方法是在遍历数组时，将每个数组元素传入回调进行处理，最后返回一个新数组

```js
// 给数组原型添加方法
Array.prototype._map(fn, context) {
  context = context || window
  let temp = []
  if (typeof fn === 'function') {
    for (let i = 0; i < this.length; i++) {
      // fn 传入 三个参数分别是 数组当前遍历元素，数组当前遍历元素下标，处理的数组
      // 将 fn 调用后的结果放入新数组中
      temp.push(fn.call(context, this[i], i, this))
    }
  } else {
    console.log('')
  }
  return temp
}
```

<!-- more -->

## 柯里化

### 从一个例子引入

```js
add(1)(2)(3) // 6
add(1, 2, ,3) // 6
add(1, 2)(3) // 6
```

add 方法每执行一次，会返回一个相同的函数并计算剩余的参数，先实现最简单的情况

```js
function add(a) {
  return function(b) {
    return a + b
  }
}
console.log(add(1)(2)) // 3
```

原理就是**利用闭包，将所有参数搜集起来，最后集中到最后返回的函数中进行计算**并返回结果

```js
function add() {
  // 第一次调用，使用一个数组存放参数
  let _args = Array.prototype.slice.call(arguments)

  // 内部声明一个函数，利用闭包保存 _args 并搜集所有参数，每次调用会返回该函数
  let adder = function() {
    let _adder = function() {
      Array.prototype.push.apply(_args, Array.prototype.slice.call(arguments))
      return _adder
    }

    // 利用隐式转换特性，当最后执行时隐式转换，并计算最终值返回
    _adder.toString = function() {
      return _args.reduce(function(a, b) {
        return a + b
      })
    }
    return _adder
  }

  return adder.apply(null, Array.prototype.slice.call(arguments))
}

console.log(add(1, 2, 3, 4, 5)) // 15
console.log(add(1, 2, 3, 4)(5)) // 15
console.log(add(1)(2)(3)(4)(5)) // 15
```

### 柯里化概念

> 柯里化又称为部分求值，是把接受多个参数的函数变换成接受单一参数的函数，并且返回一个**新的函数，新函数接受剩余参数并计算返回结果**

- 接受单一参数，通常以回调函数来解决
- 将部分参数通过回调函数的方式传入函数中
- 返回一个新函数，用于处理想要传入的参数

### 柯里化通用式

```js
function curry(fn) {
  // 第一次调用使用一个数组存放参数 (这里第一个参数是回调)
  let args = Array.prototype.slice.call(arguments, 1)

  // 返回一个新函数，利用闭包保存 args 并搜集所有参数
  return function() {
    // 将所有参数搜集到一个数组中，方便最后统一计算
    let _args = args.concat(Array.prototype.slice.call(arguments))
    // 统一传入 fn 进行调用
    return fn.apply(null, _args)
  }
}

var sum = curry(function() {
  let args = [].slice.call(arguments)
  return args.reduce(function(a, b) {
    return a + b
  })
}, 10)
console.log(sum(1, 2)) // 13
console.log(sum(1, 2, 3)) // 16
```

### 柯里化和 bind

对于 bind，是返回一个新的绑定函数，绑定函数再次调用其实是 借助 apply 或 call 方法

```js
// fn.bind(context, args1, arg2)
Function.prototype._bind = function(context) {
  context = context || window
  // this 这里为 fn
  let _this = this
  // 保存参数
  let args = Array.prototype.slice.call(arguments, 1)

  // 返回新的绑定函数, 利用闭包保存 args 并搜集所有参数
  return function() {
    // 将所有参数搜集到一个数组中，方便最后统一计算
    // 绑定函数调用是借助 apply 方法
    let _args = args.concat(Array.prototype.slice.call(arguments))
    return _this.apply(context, args)
  }
}
```

对于 bind 的调用者可能是构造函数，因此进行一下改造

```js
// fn.bind(context, args1, arg2)
Function.prototype._bind = function(context) {
  context = context || window
  // this 这里为 fn
  let _this = this
  // 保存参数
  let args = Array.prototype.slice.call(arguments, 1)

  let fnop = function() {}

  // 返回新的绑定函数
  let fBound =  function() {
    // 绑定函数调用是借助 apply 方法
    let _args = args.concat(Array.prototype.slice.call(arguments))
    return _this.apply(this instanceof fnop ? this : context, _args)
  }

  // 继承
  fnop.prototype = _this.prototype
  fBound.prototype = new fnop()

  return fBound
}
```