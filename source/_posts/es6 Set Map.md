---
title: es6 Set Map
tag:
	- es6
---

<!-- markdownlint-disable MD010 -->

> 参考: [ECMAScript 6 入门 Set 和 Map 数据结构 部分](http://es6.ruanyifeng.com/#docs/set-map)、[向 ES6 看齐，用更好的 Javascript(三)](https://www.cnblogs.com/vicfeel/p/5829568.html)、[es6 数组对象新增方法 Array.from()将两类对象转为真正的数组](http://blog.csdn.net/qq_30100043/article/details/53198243)

## Set

Set **类似数组，但是成员的值都是唯一的，没有重复的值**

- **Set 本身是一个构造函数**，用来生成 Set 数据结构
- 可以通过 **add 方法向 Set 结构加入成员**
- Set 函数可接受一个数组（或具有 iterable 接口的其他数据结构）作为参数，用来初始化
- 向 Set 加入值的时候**不会发生类型转换（但这里区别在于 NaN 等于自身，而精确运算 === 认为 NaN 不等于自身）**
- 两个对象总是不相等的，包括两个空对象

<!-- more -->

```js
var s = new Set() // Set {}
var s = new Set([2, 3, 2]) // Set {2, 3} // 元素去重

s.add(2) // Set {2, 3} 元素唯一
s.add('2') // Set {2, 3, '2'} 添加元素不会进行类型转换

s.add(NaN)
s.add(NaN) // Set {2, 3, '2', NaN} 这里认为 NaN 等于自身，而精确运算 === 认为 NaN 不等于自身

s.add({})
s.add({}) // Set {2, 3, '2', NaN, {}, {}} 两个对象不相等
```

### Set 实例的属性

- **Set.prototype.constructor** 构造函数，默认就是 Set 函数
- **Set.prototype.size** 返回 Set 实例的成员总数

### Set 实例的方法

Set 实例的方法有两大类，操作方法（用于操作数据）和遍历方法（用于遍历成员）

操作方法:

- **add(value)** 添加某个值，返回 Set 结构本身 **可以链式调用**
- **delete(value)** 删除某个值，返回一个布尔值，表示删除是否成功
- **has(value)** 返回一个布尔值，表示该值是否为 Set 的成员
- **clear()** 清除所有成员，没有返回值

```js
let s = new Set()
s.add(1).add(2).add(2)

// size 实例属性
s.size // 2

//  判断该值是否是该 Set 的成员,返回布尔值
s.has(1) // true
s.has(3) // false

var flag = s.delete(2) // true
s.has(2) // false

s.clear()
s // Set {}

// 对于判断是否包含一个键
// 对象的写法
const props = {
	'width': 1,
	'height': 1
}

if (props[someName]) {
	// ...
}

// Set 的写法
const props = new Set()
props.add('width').add('height')

if (props.has(someName)) {
	// ...
}
```

遍历方法 (Set 的**遍历顺序就是插入顺序**)：

- **keys()** 返回键名的遍历器
- **values()** 返回键值的遍历器 **Set 结构默认可遍历，它的默认遍历器生成函数就是它的 values 方法**
- **entries()** 返回键值对的遍历器
- **forEach()** 使用处理函数遍历每个成员,没有返回值。第一个参数就是处理函数，还可以有第二个参数，表示绑定函数内部的 this 对象


keys() values() entries() 方法都是返回遍历器对象，**由于 Set 结构没有键名，或者说键名和键值是同一个值，因此 keys() 和 values() 的行为完全一致**

entries() 返回的遍历器同时包含键名和键值，遍历器如果每次输出会输出一个数组，它的两个成员完全相等

```js
let s = new Set([1, 2, 3])

s.keys() // SetIterator {1, 2, 3}

Set.prototype[Symbol.iterator] = Set.prototype.values // Set 结构默认可遍历，它的默认遍历器生成函数就是它的 values 方法

for (let item of s.keys()) {
	console.log(item) // 1 2 3
}

for (let item of s.values()) {
	console.log(item) // 1 2 3
}

for (let item of s.entries()) {
	console.log(item)
}
// [1, 1]
// [2, 2]
// [3, 3]

s.forEach((value, key) => {
	console.log(key + ' : ' + value)
})
// 1 : 1
// 2 : 2
// 3 : 3
```

### 遍历的应用

数组去重方法：**用含有重复值的数组来生成 Set，可将生成的 Set 作为参数传入 Array.from() 方法，也可以结合 ... 运算符转化为数组** （Array.from 方法可以将 Set 结构转为数组）

```js
let arr = [1, 1, 2]
var a = Array.from(new Set(arr)) // [1, 2]

var a = [...new Set(arr)] // [1, 2]
```

使用 Set 可以方便实现并集、交集和差集

```js
let a = new Set([1, 2, 3])
let b = new Set([2, 3, 4])

// 并集
let union = new Set([...a, ...b]) // Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter(x => b.has(x))) // Set {2, 3}

// 差集
let difference = new Set([...a].filter(x => !b.has(x))) // Set {1}
```

在遍历操作中改变原有的 Set 结构，一种方法是利用原有 Set 结构映射出一个新的 Set 结构，然后赋值给原有 Set 结构，另一种方法是 Array.from()

```js
let set = new Set([1, 2, 3])
set = new Set([...set].map(val => val * 2)) // Set {2, 4, 6}

set = new Set(Array.from(set, val => val * 2)) // Set {2, 4, 6}
```

## Map

**es6 提供的 Map 数据结构，类似对象，也是键值对的结合，但是 "键" 范围不限于字符串，各种类型的值（包括对象）都可以当做键** 区别于js 的对象只能用字符串当做键

- **Map 本身是一个构造函数**，用来生成 Map 数据结构
- Map 函数可以接受一个数组（或具有 iterable 接口, 并且**每个成员都是一个双元素的数组的数据结构**）作为参数，用来初始化
- 因此 **Set 和 Map 都能生成新的 Map**
- 可以通过 **set 方法向 Map 结构加入键值对, get  方法读取键名对应的键值**
- **同一个键多次赋值，后面的值会覆盖前面的值**
- 只有**对同一个对象的引用，Map 解构才视为同一个键**
- **Map 的键实际是和内存地址绑定的，只要内存地址不一样，就是两个键** Map 键不会进行类型转换，对于基本数据类型（数字、字符串、布尔值），两个值严格相等，Map 会视为一个键。undefined 和 null 也是不同的两个键，**NaN 不严格等于自身，但是 Map 将其视为同一个键**

```js
let map = new Map()
map // Map {}

map = new Map([
	['foo', 1]
	['bar', 2]
]) // Map {'foo' => 1, 'bar' => 2}

// 上面 Map 构造函数传入参数是一个数组，数组每个元素都是双元素的数组，其实是通过执行下面的算法
const item = [
	['foo', 1]
	['bar', 2]
]
item.forEach(
	// 解构，通过 set 方法向 Map 添加键值对
	([key, val]) => map.set(key, val)
)

// 通过 Set 生成 Map
const set = new Set([
	['foo', 1]
	['bar', 2]
])
map = new Map(set)
map.get('foo') // 1

// 通过 Map 生成 Map
const m = new Map([['bar', 3]])
map = new Map(m) // Map {'bar' => 3}

map.set(['a'], 2) // Map {'bar' => 3, ['a'] => 2}
map.get(['a']) // undefined 因为这两个数组虽然元素值一样，但是内存地址不一样

map = new Map([
	[undefined, '1'],
	[null, '2']
])  // Map {undefined => '1', null => '2'}

map.set('1', '1').set(1, 1) // 键值不会进行类型转换 Map {undefined => '1', null => '2', '1' => '1', 1 => 1}

map.set(+0, 1)
map.get(-0) // 1

map.set(NaN, 1)
map.get(NaN) // 1 这里 Map 将 NaN 视为同一个键，这个平时 NaN 不严格等于自身不同
```

### Map实例的属性和方法

- **size 属性** 返回 Map 结构的成员总数

```js
const map = new Map()
map.size // 0
```

操作方法（用来操作数据）：

- **set(key, value)** set 方法设置或更新键名 key 队形的键值 为 value，返回整个 Map 结构, **可以链式调用**
- **get(key)** 读取 key 对应的键值，如果找不到 key，返回 undefined
- **has(key)** 返回布尔值，表示某个键是否在当前 Map 对象中
- **delete(key)** 删除某个键，成功返回 true，失败返回 false
- **clear()** 清空所有成员，没有返回值

```js
const m = new Map()
m.set('edition', 6) // 键是字符串 Map {"eidtion" => 6}
m.set(222, 6) // 键是数值 Map {"eidtion" => 6, 222 => 6}
m.set({}, 6) // 键是 对象 Map {"eidtion" => 6, 222 => 6, {} => 6}

const fn = function() {}
m.set(fn, 'fn') // 键值是函数
m.get(h) // "h"
m.get('o') // 用字符串 o 作为键名查找 undefined

m.has(fn) // true
m.has('o') // false

m.delete(fn) // true
m // Map {"eidtion" => 6, 222 => 6, {} => 6}
m.clear()
m // Map {}
```

遍历方法（**Map 的遍历顺序就是插入顺序**）：

- **keys()** 返回键名的遍历器
- **values()** 返回键值的遍历器
- **entries()** 返回所有成员的遍历器 **Map 结构默认可遍历，它的默认遍历器生成函数就是它的 entries 方法**
- **forEach()** 使用处理函数遍历每个成员,没有返回值。第一个参数就是处理函数，还可以有第二个参数，表示绑定函数内部的 this 对象

```js
const map = new Map([
	['y', 'yes'],
	['n', 'no']
])

map.keys() // MapIterator {"y", "n"} 返回键名遍历器
map.values() // MapIterator {"yes", "no"} 返回键值遍历器
map.entries() // MapIterator {"y" => "yes", "n" => "no"} 返回所有成员遍历器

Map.prototype[Symbol.iterator] === Map.prototype.entries // Map 的实例默认可遍历，它的默认遍历器生成函数就是它的 entries 方法

for (let item of map.keys()) {
	console.log(item) // y n
}

for (let item of map.values()) {
	console.log(item) // yes no
}

for (let item of map.entries()) {
	console.log(item)
	// ["y", "yes"]
	// ["n", "no"]
}

map.forEach(function(value, key, map) {
	console.log("key: %s, value: %s", key, value)
	// key: y, value: yes
	// key: n, value: no
})

// forEach 可以接受第二个参数用来绑定 this
const reporter = {
	report: function(key, value) {
		console.log("key: %s, value: %s", key, value)
	}
}
map.forEach(function(value, key, map) {
	this.report(key, value)
}, reporter)
// key: y, value: yes
// key: n, value: no
```

### 数据结构转化

Map 转为数组

- **Array.from(Map)**
- **使用扩展运算符 ...**

```js
const map = new Map()
	.set(true, 7)
	.set(1, '2')

// 第一种方法
let arr = Array.from(map)

// 第二种方法
let arr = [...map] // [[true, y],[1, '2']]
```

数组转为 Map

- 将数组传入 Map 构造函数即可

```js
new Map([
	[true, 7],
	[1, '2']
]) // Map {true => 7, 1 => '2'}
```

- 结合数组的 map 方法、filter 方法实现 Map 的遍历和过滤

```js
const map = new Map()
	.set(1, '1')
	.set(2, '2')
	.set(3, '3')

const map1 = new Map(
	[...map].filter([k, v] => k < 3)
) // Map {1 => '1', 2 => '2'}

const map2 = new Map(
	[...map].map([k, v] => [k * 2, '_' + v])
) // Map {2 => '_a', 4 => '_b', 6 => '_c'}
```

Map 转为对象

- 因为对象的键都是字符串，因此如果 Map 的键都是字符串，可以无损转为对象，如果有非字符串的键名，那这个键名会被转为字符串，再作为对象的键名

```js
function mapToObj(map) {
	let obj = Object.create(null)
	for (let [k, v] of map) {
		obj[k] = v
	}
	return obj
}

const map = new Map()
map.set('y', true).set(true, 1).set({'true': 1}, 1)
// Map 的键先转为字符串，再作为对象的键名
mapToObj(map) // {'y': true, 'true': 1, '[object Object]': 1}
```

对象转为 Map

```js
function objToMap(obj) {
	let m = new Map()
	for (let k in obj) {
		if (obj.hasOwnProperty(k)) {
			m.set(k, obj[k])
		}
	}
	// for (let k of Object.keys(obj)) { m.set(k, obj[k]) }
	return m
}

objToMap({yes: true, no: false})
// Map {"yes" => true, "no" => false}
```

Map 转为 JSON

- 对于键名都是字符串的，可以选择转为 对象 JSON (其实是**先将 Map 转为对象，再通过 JSON.stringify() 转为 JSON**)
- 对于键名有非字符串的，可以选择转为 数组 JSON（其实**通过结合扩展运算符转为数组，再调用 JSON.stringify() 方法转为 JSON**）

```js
// Map 键名都是字符串，先将 Map 转为对象，再调用 JSON.stringify() 方法转为 JSON
function strMapToJson(map) {
	return JSON.stringify(mapToObj(map))
}

let map = new Map().set('yes', true).set('no', false)
strMapToJson(map) // '{"yes": true, "no": false}'

// Map 键名为非字符串，通过结合扩展运算符转为数组，再调用 JSON.stringify() 方法转为 JSON
function mapToArrayJson(map) {
	return JSON.stringify([...map])
}

let map = new Map().set(true, 7).set({foo: 3}, ['abc'])
mapToArrayJson(map) // '[[true, 7], [{"foo": 3}, ["abc"]]]'
```

JSON 转为 Map

- 正常情况下，所有键名都是字符串（其实**先将调用 JSON.parse() 转为对象，再将对象转为 Map**）
- 对于特殊情况整个 JSON 是一个数组，并且每个数组成员本身又是一个有两个成员的数组，可以一一对应转为 Map，这往往是 Map 转为数组 JSON 的逆操作（其实**先将调用 JSON.parse() 转为每个成员都是两个元素的数组的数组，再将数组转为 Map**）

```js
// JSON 键名都是字符串，先将调用 JSON.parse() 转为对象，再将对象转为 Map
function jsonToStrMap(json) {
	return objToMap(JSON.parse(json))
}

jsonToStrMap('{"yes": true, "no": false}')
// Map {'yes' => true, 'no' => false}

// JSON 是一个数组，并且每个数组成员都是一个有两个成员的数组，先将调用 JSON.parse() 转为每个成员都是两个元素的数组的数组，再将数组转为 Map
function jsonToMap(json) {
	return new Map(JSON.parse(json))
}

jsonToMap('[[true,7],[{"foo":3},["abc"]]]')
// Map {true => 7, Object {foo: 3} => ['abc']}
```

## 弱引用集合 WeakSet 和 WeakMap

- **WeakSet 的值和 WeakMap 的键名必须是对象（引用类型）**
- **WeakMap 只支持 new has get set delete**
- **WeakSet 只支持 new has add delete**
- **不可遍历**，除非专门查询或者给出感兴趣的键，否则不能获得一个弱引用集合中的项
- **WeakSet 并不对其中的对象保持强引用**。当 WeakSet 中的一个对象被回收时，它会简单从 WeakSet 中移除。**WeakMap 也类似不会对它的键名保持强引用,但是键值仍然是强引用。** Map 和 Set 都为内部的键或值保持了强引用，即**如果一个对象被移除了，回收机制无法取回它占用的内存，除非在 Map 或 Set 中删除它**


垃圾回收机制依赖引用计数，若一个值的引用次数不为 0，垃圾回收机制就不会释放这块内存。结束使用该值后，有时会忘记取消引用，导致内存无法释放，进而可能引发内存泄露。**WeakSet 里面的引用，都不计入垃圾回收机制。只要这些对象在外部消失，它在 WeakSet 中的引用就自动消失** 由于上面的特点，WeakSet 的成员不适合引用，因为它会随时消失。另外由于 WeakSet 内部有多少成员，取决于垃圾回收机制有没有运行，运行前后可能成员个数是不一样的，而垃圾回收机制何时运行时不可预测的，因此 **es6 规定 WeakSet 不可遍历（没有 keys() values() entries() forEach() 方法），没有 size 属性。同理这些同样适用于 WeakMap 键名对应的对象**

```js
// WeakSet 接受一个数组或类似数组的对象作为参数，实际上任何具有 Iterable 接口的对象，都可以作为 WeakSet 的参数，该数组所有成员都会自定成为 WeakSet 实例对象的成员(这意味着数组成员只能是对象)
const a = [[1, 2], [3, 4]]
const b = [3, 4]
const c = {"2": 3}
const d = {}
let ws = new WeakSet(a) // WeakSet {[1, 2], [3, 4]}
ws = new WeakSet(b) // TypeError: Invalid value used in weak set 参数传递时候，数组的成员自动 WeakSet 实例的成员，因此要求 Iterable 接口对象的成员也是对象

ws.add(1) // TypeError: Invalid value used in weak set 因为 WeakSet 的成员必须是对象（引用类型）

ws.add(b).add(c) // WeakSet {[1, 2], [3, 4], [3, 4], {"2": 3}} 这里 [3, 4] 并不指向同一个内存地址

ws.has(b) // true
ws.has(d) // false

ws.delete(b) // true
ws.has(b) // false

ws.clear // undefined
ws.size // undefined
ws.keys // undefined
ws.values // undefined
ws.entries // undefined
ws.forEach // undefined

// WeakMap 接受接受一个数组（或具有 iterable 的数组, 该数组每个成员都是双元素，并且第一个元素只能是对象）作为参数
let k1 = [1, 2]
let k2 = [2, 3]
let k3 = {foo: 1}
let val = {foo: 2}
let wm = new WeakMap([[k1, 1],[k2, 2]]) // WeakMap {[1, 2] => 1, [2, 3] => 3}
wm.get(k2) // 2

wm.set(k3, 3).get(k3) // 3
wm.set(1, 2) // TypeError: Invalid value used as weak map key 这里 WeakMap 的键必须是对象

wm.set(k3, 4).get(k3) // 4 同一个键的赋值，后面的值会覆盖之前的值

wm.set(k3, val)
val = null
wm.get(k3) // Object {foo: 1} 这里 WeakMap 的键值是强引用

k1 = null
wm.get(k1) // undefined 这里 WeakMap 的键名是弱引用，不计入垃圾回收机制计数，键名引用的对象随时会被收回

wm.has(k3) // true
wm.has({}) // false

wm.delete(k3) // true
wm // WeakMap {[1, 2] => 1, [2, 3] => 3}

wm.clear // undefined
wm.size // undefined
wm.keys // undefined
wm.values // undefined
wm.entries // undefined
wm.forEach // undefined
```

### 应用

有时向在对象上存放一些数据，这会形成对这个对象的引用，一旦不在需要这个对象，必须手动删除该引用，否则垃圾回收机制不会释放内存

```js
// 如通过 arr 数组对该 dom 对象 添加一些文字说明，形成了 arr 对 e1 的引用
const el = document.getElementById('foo')
const arr = [
	[e1, 'foo 元素']
]

// 不需要 el 的时候，必须手动删除引用
arr[0] = null
```

基本上，如果想要往对象上添加数据，又不想干扰垃圾回收机制，可以使用 WeakMap。**WeakMap 的键对应的对象，可能在未来会消失，使用 WeakMap 结构有助于防止内存泄漏。注意 WeakMap 弱引用的只是键名，而不是键值，键值依然是强引用。** 一个典型应用场景是，在页面的 dom 元素上添加数据，使用 WeakMap 结构。当该 dom 元素被清除，其对应的 WeakMap 记录会被自动移除

```js
const wm = new WeakMap()
const ele = document.getElementById('example')
wm.set(ele, 'some info')
wm.get(ele) // "some info"
```

## 数组的扩展

### **Array.of()**

将一组值转为数组。Array.of() 总是返回参数值组成的数组，如果参数值为空，返回空数组。可用来替代 Array() 或 new Array() 因为参数不同造成的重载

```js
// Array() 参数不同造成重载，没有参数返回空数组，只有一个参数返回参数个数的数组，多个参数返回参数值组成的数组
Array() // []
Array(3) // [,,]
Array(3, 4, 5) // [3, 4, 5]

Array.of() // []
Array.of(undefined) // [undefined]
Array.of(3) // [3]
Array.of(3, 4, 5) // [3, 4, 5]
```

可以用下面的方法模拟

```js
function ArrayOf() {
	return Array.prototype.slice.call(arguments)
}
```

### **Array.from()** 

将**类数组对象**（具有数组的结构，但不是数组对象，如 arguments、DOM NodeList）和可遍历对象（具有 iterator 接口的数据结构，如 Map Set）**转为数组**

```js
// 类数组
let arrLike = {
	'0': 0,
	'1': 1,
	length: 2
}

// es5 写法
Array.prototype.slice.call(arrLike) // [0, 1]
// es6 写法
Array.from(arrLike) // [0, 1]

// arguments 对象
function foo() {
	console.log(Array.from(arguments)) // [1, 2, 3]
}
foo(1, 2, 3)

// DOM NodeList
let el = document.querySelectorAll('p')
Array.from(el).forEach(function(p) {
	console.log(p)
})

// Set
let set = new Set().add(1).add(2) // Set {1, 2}
Array.from(set) // [1, 2]

// Map
let map = new Map().set(1, 1).set(2, 3) // Map {1 => 1, 2 => 2}
Array.from(map) // [[1, 1], [2, 2]]
```

Array.from 方法还可以接受**第二个参数，作用类似数组的 map 方法，用来对每个元素进行处理，将处理后的元素放入返回的数组**

```js
Array.from(arrLike, x => x + x) // [0, 2]
// 等同于
Array.from(arrLike).map(x => x + x) // [0, 2]
```

当然，使用**扩展运算符也可以将某些数据结构(一般是具有 iterator 接口的数据结构)转为数组**

```js
function foo() {
	console.log([...arguments]) // [1, 2, 3]
}
foo(1, 2, 3)

[...document.querySelectorAll('div')]

let set = new Set().add(1).add(2) // Set {1, 2}
[...set] // [1, 2]

let map = new Map().set(1, 1).set(2, 3) // Map {1 => 1, 2 => 2}
[...map] // [[1, 1], [2, 2]]
```

### Array.prototype.find()

方法参数传入一个查找函数，返回找到的**第一个元素，找不到返回 undefined**

```js
let arr = [1, 2, 3, 3]
let result1 = arr.find(function(value, index, arr) {
	return value > 2
})
let result2 = arr.find(function(value, index, arr) {
	return value > 3
})

console.log(result1) // 3
console.log(result2) // undefined
```

Array.prototype.findIndex() 方法类似，它返回的是**找到的第一个元素的索引，没有找到返回 -1**

```js
let arr = [1, 2, 3, 3]
let result1 = arr.findIndex(function(value, index, arr) {
	return value > 2
})
let result2 = arr.findIndex(function(value, index, arr) {
	return value > 3
})

console.log(result1) // 2
console.log(result2) // -1
```

Array.prototype.indexOf() 找到一个元素在数组中的位置，没有找到返回 -1（**相同元素返回第一次出现的位置**）
Array.prototype.includes() 一个元素是否在数组中，返回 true 或 false

```js
let arr = [1, 2, 3, 3]
let result1 = arr.indexOf(3)
let result2 = arr.indexOf(4)
let result3 = arr.includes(3)
let result4 = arr.includes(4)

console.log(result1) // 2
console.log(result2) // -1
console.log(result3) // -1
console.log(result4) // -1
```

### Array.prototype.fill()

**给定一个值填充数组，fill() 方法可以传入第二个、第三个参数，分别为填充的起始位置和结束位置(不包含结束位置)**

```js
let arr = [1, 2, 3, 4]
arr.fill(5, 0, 2) // [5, 5, 3, 4]
arr.fill(6) // [6, 6, 6, 6]
```

### Array.prototype.copyWithin(target, start, end)

**将数组某些位置的元素覆盖到自身其他位置上去**

`target` 覆盖的位置下标
`start` 开始位置下标，可以省略，可以为负数
`end` 结束位置下标（不包括），可以省略，可以为负数

```js
let arr = [1, 2, 3, 4, 5, 6, 7, 8]
arr.copyWithin(2, 3, 5) // [1, 2, 4, 5, 5, 6, 7, 8]
```