---
title: es6 iterator
tag:
	- es6
---

> 参考: [ECMAScript 6 入门 iterator 部分](http://es6.ruanyifeng.com/#docs/iterator)

<!-- markdownlint-disable MD010 -->

iterator 遍历器是一种机制、一种接口，它为各种不同的数据结构提供统一的访问机制。**任何数据结构只要部署 iterator 接口，就可以完成遍历（依次处理该数据结构中的所有成员）。es6 中提供的遍历方法 **for...of 便是自动寻找该对象的 iterator 接口**

<!-- more -->

## iterator 遍历过程

- 创建一个指针对象，指向当前数据结构的起始位置。**遍历器对象本质是一个指针对象**
- 第一次调用指针对象的 **next 方法**，可以将指针指向数据结构的第一个成员
- **每次调用 next 方法，都会返回数据结构当前成员的信息（返回一个包含 value 和 done 属性的对象，value 属性是当前成员的值，done 属性是一个布尔值，表示遍历是否结束）**
- 依次类推，不断调用指针对象的 next 方法，**直到它指向数据结构的结束位置**

```js
// 定义的 genIterator 函数是一个遍历器生成函数，作用是返回一个遍历器对象，对数据结构执行这个函数，就会返回该数据结构的遍历器对象 it
// 如下是针对数组的遍历器生成函数
var it = genIterator([1, 2])

// 遍历器对象（指针对象）的 next 方法用来移动指针，开始时候指针指向数组的起始位置，之后每次调用 next 方法，指针就会指向数组的下一个成员
// next 方法返回一个对象，表示当前数据成员信息，这个对象具有 value 和 done 属性，value 属性返回当前位置的成员，done 属性是一个布尔值，表示遍历是否结束，是否还有必要再次调用 next 方法
// 对遍历器对象来说， done: false 和 value: undefined 属性可省略
it.next() // {value: 1, done: false}
it.next() // {value: 2, done: false}
it.next() // {value: undefined, done: false}

function genIterator(arr) {
  var index = 0
  // 返回遍历器对象
  return {
    next: function() {
      return index < arr.length ?
      {value: arr[index++], done: false} :
      {value: undefined, done: true}
      // 等同 return index < arr.length ? {value: arr[index++]} : {done: true}
    }
  }
}
```

## 默认 iterator 接口

**es6 默认的 iterator 结构部署在数据结构的 Symbol.iterator 属性**，或者说一个数据结构只要具有 Symbol.iterator 属性，就可认为是可遍历的。

**Symbol.iterator 属性本身是一个函数，就是当前数据结构默认的遍历器生成函数，执行这个函数，就会返回一个遍历器。**属性名 Symbol.iterator 本身是一个表达式，返回 Symbol 对象的 iterator 属性，这是一个预定一号的，类型为 Symbol 的特殊值，所以要放在方括号内

一些数据结构默认部署 iterator 接口，可以直接使用 for...of 遍历

- **数组**
- **Set**
- **Map**
- **类数组（如 arguments 对象、dom NodeList 对象）**
- **generator 对象**
- **字符串**

```js
let arr = [1, 2]
let b = 2

for (let item of arr) {
	console.log(item)
	// 1
	// 2
}

let iter = arr[Symbol.iterator]()
iter.next() // {value: 1, done: false}
iter.next() // {value: 2, done: false}
iter.next() // {value: undefined, done: true}

// for of 遍历遍历器，直接将值取出来，值就是对应 next() 返回的 value
let set = new Set(arr)
for (let item of set) {
	console.log(item)
	// 1
	// 2
}

let map = new Map([[arr, 1], [b, 2]])
for (let item of map) {
	console.log(item)
	// [[1, 2], 1]
	// [[2, 1]
}

function test() {
	for (let item of arguments) {
		console.log(item)
		// 1
		// 2
	}
}
test(...arr)

let str = 'str'
for (let item of str) {
	console.log(item)
	// s
	// t
	// r
}
// 对比 for...in 一般用来以任意顺序遍历一个对象的可枚举属性
for (let item in str) {
	console.log(item)
	// 0
	// 1
	// 2
}
```

对于没有部署 iterator 接口的结构（如对象 Object）想要使用 for...of 遍历可以自己部署。定义 iterator 接口实际上是 **用 Symbol.iterator 作为函数名定义一个函数，该函数调用时会返回一个 iterator 对象，该 iterator 对象具有 next() 方法。next() 方法每次调用会返回一个具有 value 和 done 属性的对象，done 属性是布尔值，表示遍历是否结束**

```js
let obj = {
	data: ['yes', 'no'],
	[Symbol.iterator]() {
		const self = this
		// index 作为遍历的下标
		let index = 0
		return {
			// Iterator 接口通过 next() 函数进行遍历，直到 next 函数返回的 done 值为 true
			next() {
				if (index < self.data.length) {
					return {
						value: self.data[index++],
						done: false
					}
				} else {
					return {
						value: undefined,
						done: true
					}
				}
			}
		}
	}
}

for (let item of obj) {
	console.log(item)
	// 'hello'
	// 'world'
}
```

## 默认调用 iterator 接口的场合

- 解构赋值

对数组和 Set 结构进行解构赋值会默认调用 Symbol.iterator 方法

```js
let set = new Set().add(1).add(2).add(3)
let [x, y, z] = set // x = 1; y = 2; z = 3
let [first, ...rest] = set // first = 1; rest = [2, 3]
```

- 扩展运算符

扩展运算符 ... 也会调用默认 Symbol.iterator 接口。只要某个数据结构部署了 iterator 结构，对它使用扩展运算符号，就可将其转为数组

```js
// let arr = [...iterator]
var str = 'hello'
[...str] // ['h','e','l','l','o']

let arr = [1, 2, 3]
[0, ...arr, 4] // [0, 1, 2, 3, 4]
```

- yield*

yield* 后面跟的是一个可遍历的结构，会将它其中的 yield 按照规则一步步执行

```js
let gen = function* () {
  yield 1
  yield* [2, 3]
  yield 4
}

let it = gen()

it.next() // {value: 1, done: false}
it.next() // {value: 2, done: false}
it.next() // {value: 3, done: false}
it.next() // {value: 4, done: false}
it.next() // {value: undefined, done: true}
```

由于数组的遍历会调用遍历器接口，因此所有接受数组作为参数的场合其实都调用了遍历器接口

- for...of
- Array.from()
- Map() Set() WeakSet() WeakMap()
- Promise.all()
- Promise.race()

## 遍历语法的比较

以数组为例，常见遍历的写法是 for 循环

```js
let arr = [1, 2, 3]
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]) // 1 2 3
}
```

数组也提供了 forEach 方法

```js
// 值在前，下标在后
arr.forEach(function(value, index, arr) {
  console.log(value, index, arr)
  // 1 0 [1, 2, 3]
  // 2 1 [1, 2, 3]
  // 3 2 [1, 2, 3]
})
```

for...in 循环可以遍历数组的键名（主要是针对对象用 for...in 遍历可枚举属性，不是遍历数组）

for...in 循环有一些缺点:

- 如果数组的键名是数组，for...in 循环还是以字符串作为键名 "0" "1" "2" 等
- for...in 循环不仅循环数字键名，还会手动添加其他键，包括原型链上的键
- 有些情况下，for...in 循环会以任意顺序遍历键名

```js
for (let item of arr) {
  console.log(item)
  // 0
  // 1
  // 2
}
```

for...of 循环类似 for...in 语法，但没有 for...in 的缺点，并且提供了遍历所有数据结构的统一操作接口

```js
for (let item of arr) {
  console.log(item)
  // 1
  // 2
  // 3
}
```

jQuery 也提供了 $.each(data, function(index, value) {})

```js
// 下标在前
$.each(arr, function(index, value) {
  console.log(index, value)
})

// 也可以处理对象
let obj = {
	a: 1,
	b: 2,
	3: 3
}

$.each(obj, function(key, value) {
	console.log(key, value)
	// 3 3
	// a 1
	// b 2
})

// 当然可以处理类数组
function test() {
	$.each(arguments, function(index, value) {
		console.log(index, value)
		// 0 'a'
		// 1 'b'
		// 2 1
	})
}
test('a', 'b', 1)
```