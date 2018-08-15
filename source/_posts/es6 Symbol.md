---
title: es6 Symbol
tag:
	- es6
---

<!-- markdownlint-disable MD010 -->

> 参考: [ECMAScript 6 入门 Symbol 部分](http://es6.ruanyifeng.com/#docs/symbol)

## 概述

es5 对象的属性名都是字符串，容易造成属性名的冲突，可能会被重写。**es6 引入一种新的原始数据类型 Symbol，表示独一无二的值**

因此**基本数据类型就有 6 种：Undefined Null Boolean String Number Symbol 再加上引用类型 Object**

<!-- more -->

Symbol 值通过 **Symbol 函数生成，注意 **Symbol 函数不能使用 new 命令，否则会报错，因为生成的 Symbol 是一个基本数据类型的值，不是对象** 由于 Symbol 值不是对象，因此不能添加属性，可以理解他是一种类似于字符串的数据类型

```js
let s = Symbol()
typeof s // "symbol"
s.toString() // "Symbol()"
```

Symbol 函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述，主要是为了在控制台显示或者转为字符串时比较容易区分

```js
let s1 = Symbol('foo')
let s2 = Symbol('bar')

s1 // Symbol(foo)
s2 // Symbol(bar)

s1 === s2 // false 每一个属性值都是不相同的

s1.toString() // "Symbol(foo)"
s2.toString() // "Symbol(bar)"
```

Symbol 值不能和其他类型的值进行计算，Symbol 值可以转化为字符串和布尔值，但是不能转为数值

```js
let s = Symbol()

s.toString() // "Symbol()"
String(s) // "Symbol()"

Boolean(s) // true
!s // false

Number(s) // TypeError: Cannot convert a Symbol value to a number
s + 'test' // TypeError 不能和其他类型值进行计算
```

## 作为属性名的 Symbol

由于每一个属性值都是不相等，因此 Symbol 值可以作为标识符，用于对象的属性名，保证不出现同名的属性

```js
let mySymbol = Symbol()

// 第一种写法
let a = {}
a[mySymbol] = 'Hello'

// 第二种写法
let a = {
  [mySymbol]: 'Hello'
}

// 第三种方法
let a = {}
// 这里将对象的属性名指定为一个 Symbol 而不是常规的字符串
Object.defineProperty(a, mySymbol, {
  value: "Hello"
})

a[mySymbol] // "Hello"
```

**Symbol 值作为对象属性名的时候，不能用点运算符** 因为点运算符后面总是字符串，不会读取 mySymbol 作为标志名所指代的 Symbol 值，导致 a 的属性名实际是一个字符串，而不是一个 Symbol 值。同样在**对象的内部，使用 Symbol 值定义属性的时候， Symbol 值必须放在 方括号中**

```js
let s = Symbol()
let obj = {
  // 如果 s 不放在方括号中，该属性的键名就是字符串 s，而不是 s 代表的 Symbol 值
  [s]: function(arg) {...}
  // 等同于 [s](arg) {...}
}

obj[s](123)
```

Symbol 值作为属性名时，该属性在类外部是可以访问的，但是**不会出现在 for ... in、for ... of 循环中，也不会被 Object.keys()、Object.getOwnPropertyNames() 返回，但是有个 Object.getOwnPropertySymbols() 方法可以获取指定对象的所有 Symbol 属性名**

```js
let a = Symbol('a')
let b = Symbol('b')
let obj = {
  [a]: 'a',
  [b]: 'b',
  c: 'c'
}

for (let key in obj) {
  console.log(key) // 'c' 不会返回 Symbol 属性名
}

Object.keys(obj) // ['c'] 数组不会返回 Symbol 属性名

Object.getOwnPropertyNames(obj) // ['c'] 数组不会返回 Symbol 属性名

Object.getOwnPropertySymbols(obj) //[Symbol(a), Symbol(b)]
```

Symbol 类型还可以**用来定义一组常量，保证这组常量的值都是不相等的**

```js
log.levels = {
  DEBUG: Symbol("debug"),
  INFO: Symbol("info"),
  WARN: Symbol("warn")
}

console.log(log.levels.DEBUG, 'debug message')
console.log(log.levels.INFO, 'info message')
```

## 内置方法

其他内置方法见 [ECMAScript 6 入门 Symbol 部分](http://es6.ruanyifeng.com/#docs/symbol)