---
title: for...in、Object.keys()、Object.getOwnPropertyNames() 比较
tag: 
	- 
---

> 参考: [JavaScript中的可枚举属性与不可枚举属性](https://www.cnblogs.com/kongxy/p/4618173.html)、[详解forin，Object.keys和Object.getOwnPropertyNames的区别](http://yanhaijing.com/javascript/2015/05/09/diff-between-keys-getOwnPropertyNames-forin/)、[Object.create() - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

在 js 中对象的属性分为可枚举和不可枚举，这是由属性描述符 enumberable 的值 true/false 决定的，可枚举性决定了属性是否能被 for...in 和 Object.keys() 中遍历查找

js 基本包装数据类型的原型属性是不可枚举的，如 Object、Number、String 等

```js
let num = new Number()
for (let prop in num) {
  console.log('prop: ' + num[prop]) // 输出为空
}
```

<!-- more -->

## 设置可枚举属性

可以通过 Object.create(prototype, [properticesObject]) 在创建对象时通过第二个参数对象中，传入属性名和对应的属性描述符以及对应的属性值，也可以通过 Object.defineProperty(Obj, propName, discriptor) 在discrptor中设置 propName 的属性描述符以及对应的属性值

```js
let Parent = {}
Parent = Object.create(Object.prototype, {
  a: {
    value: 'a',
    writable: true,
    configurable: true,
    enumerable: true
  },
  b: {
    value: 'b'
  }
}) // Parent {a: 'a', b: 'b'} 其中 a 属性可枚举，b 属性不可枚举

let Child = Object.create(Parent, {
  c: {
    value: 'c',
    writable: true,
    configurable: true,
    enumerable: true
  },
  d: {
    value: 'd'
  }
}) // Child {c: 'c', d: 'd'} 其中 c 属性可枚举，d 属性不可枚举
```

## for...in

for...in 是将**对象和原型链**上的**可枚举属性**遍历

```js
// for...in 输出对象和原型链上的可枚举属性
for (let key in Child) {
  console.log(key) // c a
}
```

因此常常结合 Object.prototype.hasOwnProperty() 筛选对象自身定义的可枚举属性

```js
for (let key in Child) {
  if (Child.hasOwnProperty(key)) {
    console.log(key) // c
  }
}
```

## Object.keys()

Object.keys() 遍历返回包含**对象自身定义**的**可枚举属性的属性名**的数组

```js
Object.keys(Child) // ['c']
```

## Object.getOwnPropertyNames()

Object.getOwnPropertyNames() 遍历返回包含**对象自身定义**的**所有属性的属性名**的数组，不只是可枚举属性名

```js
Object.getOwnPropertyNames(Child) // ['c', 'd']
```

## 补充

**Object.create(prototype, [propertiesObject])**

使用 Object.create() 返回一个创建的新对象，带着指定的原型对象和属性，参数 prototype 是新创建对象的原型对象，参数 propertiesObject 可选，没有指定就为 undefined，否则是要**添加到新创建对象的可枚举属性（即自身定义的属性，而不是其原型链上的可枚举属性）对象的属性描述符和相应的属性名称**。这些属性对应 Object.defineProperty() 的第二个参数

属性描述符包括：

- **value:** 该属性对应的值，默认为 undefined
- **writable:** 当且仅当该属性 writable 为 true，value 才能被赋值运算符改变，默认为 false
- **enumerable:** 当且仅当该属性的 enumerable 为true时，该属性才能够出现在对象的枚举属性中（**属性可被 for...in 和 Object.keys() 遍历查找**）。默认为 false
- **configurable:** 当且仅当该属性的 configurable 为true时，该属性描述符才能被改变，同时**该属性能从对象上删除**。默认为 false（**属性描述符是否可以被修改，该属性是否能从对象上删除**）

但是用 **set 和 get 时候，不能使用 value 和 writable 这两个属性描述符**
- **set:** 一个给属性提供 setter 方法，**方法接受唯一参数，并将该参数的新值赋值给该属性，默认为 undefined**
- **get:** 一个给属性提供 getter 方法，**该方法的返回值被用作属性值，默认为 undefined**

```js
var o
var val
o = Object.create(null) // 创建一个原型为 null 的空对象

o = {}
// 相当于通过字面量的方式创建空对象
o = Object.create(Object.prototype)

o = Object.create(Object.prototype, {
  // a 会成为创建对象的属性
  a: {
    value: 'a',
    writable: true, // 设置为true，value 可以被赋值运算符改变
    enumerable: true, // 设置为 true，该属性为可枚举，可被 for...in 和 Object.keys() 遍历查找
    configurable: true, // 设置为 true，其他属性描述符才能被改变，属性才能从对象上删除
  },
  // b 也会成为创建对象的属性，但是 descriptor 的属性值省略了默认为 false，因此 b 属性不可写、不可枚举、不可配置
  b: {
    value: 'b'
  },
  // c 属性使用 set 和 get 时，不能使用 value 和 writable 这两个属性描述符
  c: {
    set: function(newVal) {
      console.log('setter',newVal)
      val = newVal
    },
    get: function() {
      console.log('getter')
      return val
    },
    configurable: true,
    enumerable: true
  }
}) // {a: 'a', b: 'b'}

o.a = 'c' // {a: 'c', b: 'b'}
o.b = 'd' // {a: 'c', b: 'b'}
o.b // 'b' 属性 b 不可写
o.c = 'c' // setter c // c
o.c // getter // c

Object.keys(o) // ["a", "b"] 属性 a 为可枚举，属性 b 不可枚举, 属性 c 可枚举

delete o.b // false 属性不可配置
o // {a: 'c', b: 'b'}
// b 的属性描述符不可配置
Object.defineProperty(o, "b", {writable: true}) // VM1168:1 Uncaught TypeError: Cannot redefine property: b
Object.defineProperty(o, "b", {configurable : true}) // VM1168:1 Uncaught TypeError: Cannot redefine property: b
Object.defineProperty(o, "b", {enumerable : true}) // VM1168:1 Uncaught TypeError: Cannot redefine property: b
Object.defineProperty(o, "b", {value: 'd'}) // VM1168:1 Uncaught TypeError: Cannot redefine property: b
Object.defineProperty(o, "b", {set: function() {}}) // VM1168:1 Uncaught TypeError: Cannot redefine property: b
Object.defineProperty(o, "b", {get: function() {return 1}}) // VM1168:1 Uncaught TypeError: Cannot redefine property: b
```