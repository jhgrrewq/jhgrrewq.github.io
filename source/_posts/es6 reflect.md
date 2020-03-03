---
title: es6 proxy
tag:
	- es6
---

> 参考: [ECMAScript 6 入门 reflect 部分](http://es6.ruanyifeng.com/#docs/reflect)

<!-- markdownlint-disable MD010 -->

## 概述

Reflect 对象的设计有如下目的：

- 将 Object 对象的一些明显属于语言内部方法放到 Reflect 对象上（如 Object.defineProperty）。目前某些方法同时在 Object 对象和 Reflect 对象上部署，未来的新方法将只在 Reflect 对象上部署
- 修改某些 Object 方法的返回结果变得合理。如 Object.defineProperty(obj, name, desc) 在无法定义属性时会抛出错误，Reflect.defineProperty(obj, name, desc) 则返回 false
- 让 Object 操作都变成函数行为。某些 Object 操作是命令式如 name in obj 和 delete obj[name], 而 Reflect.has(obj, name) 和 Reflect.deleteProperty(obj, name) 变成函数行为
- Reflect 对象和 Proxy 对象方法一一对应。因此 Proxy 对象可以方便调用对应的 Reflect 方法，完成默认行为，作为修改行为的基础

```js
// 不使用 Reflect 对象
var loggedObj = new Proxy(obj, {
  get(obj, name) {
    console.log('get', obj, name)
    return obj[name]
  },
  deleteProperty(obj, name) {
    console.log('delete' + name)
    delete obj[name]
  },
  has(obj, name) {
    console.log('has' + name)
    return has(obj, name)
  }
})
// 使用 Reflect 对象，操作更易读
var loggedObj = new Proxy(obj, {
  get(obj, name) {
    console.log('get', obj, name);
    return Reflect.get(obj, name);
  },
  deleteProperty(obj, name) {
    console.log('delete' + name);
    return Reflect.deleteProperty(obj, name);
  },
  has(obj, name) {
    console.log('has' + name);
    return Reflect.has(obj, name);
  }
})
```

## Reflect 对象静态方法

Reflect 对象一共有 13 种静态方法：

- `Reflect.get(obj, name, receiver)`:  **查找并返回 obj 对象的 name 属性，没有则返回 undefined**
- `Reflect.set(obj, name, value, receiver)`: **设置 obj 对象 name 属性等于 value**
- `Reflect.has(obj, name)`:  **对应 name in obj 操作** in 运算符
- Reflect.deleteProperty(obj, name)` : 等同 **delete obj[name] 操作**, 返回一个布尔值
- `Reflect.ownKeys(obj)`: 基本等同 **Object.getOwnPropertyNames(obj) 和 Object.getOwnPropertySymbols(obj) 之和**
- `Reflect.getOwnPropertyDescriptor(obj, name)`: 基本等同 **Object.getOwnPropertyDescriptor(obj, name)，返回属性的描述对象**
- Reflect.defineProperty(obj, name, attributes)`: 基本等同 **Object.defineProperty**
- `Reflect.preventExtensions(obj)`:  对应** Object.preventExtensions(obj), 让一个对象不可扩展，返回一个布尔值，表示是否操作成功**
- `Reflect.getPrototypeOf(obj)`:  **用于读取对象的 `__proto__` 属性，对应 Object.getPrototypeOf(proxy)**
- `Reflect.isExtensible(obj)`: 对应 **Object.isExtensible(obj), 返回一个布尔值, 表示当前对象是否扩展**
- `Reflect.setPrototypeOf(obj, proto)`:  **用于设置目标对象原型，对应 Object.setPrototypeOf(obj, newProto), 返回一个布尔值表示是否设置成功**
- Reflect.apply(func, thisArg, args)`: **等同于 Function.prototype.apply.call(obj, args)，用于绑定 this 对象后执行给定函数**
- `Reflect.construct(obj, args)`：**等同 new obj(...args), 提供一种不适用 new 调用构造函数的方法**

### `Reflect.get(obj, name, receiver)`

`Reflect.get` 方法查找并返回 obj 对象 name 属性，如果没有该属性返回 undefined

```js
var myObj = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar
	}
}
Reflect.get(myObj, 'foo') // 1
Reflect.get(myObj, 'bar') // 2
Reflect.get(myObj, 'baz') // 2
```

- 如果 name 属性部署了读取函数 getter, 读取函数 this 绑定 receiver

```js
var myObj = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar
	}
}
var myReceiveObj = {
	foo: 4,
  bar: 4
}
Reflect.get(myObj, 'baz', myReceiveObj) // 8
```

- 如果第一个参数不是对象，Reflect.get 方法报错

```js
Reflect.get(1, 'foo') // 报错
```



### `Reflect.set(obj, name, value, receiver)` 

`Reflect.set` 方法设置 obj 对象 name 属性等于 value

```js
var myObj = {
  foo: 1,
  set bar(value) {
    return this.foo = value
  }
}
myObj.foo // 1
Reflect.set(myObj, 'foo', 2)
myObj.foo // 2
Reflect.set(myObj, 'bar', 3)
myObj.foo // 3
```

- 如果第一个参数不是对象, `Reflect.set` 会报错

```js
Reflect.set(1, 'foo', {}) // 报错
```

- 如果 name 属性设置了赋值函数，则赋值函数的 this 绑定 receiver

```js
var myObj = {
  foo: 2,
  set bar(value) {
    return this.foo = value
  }
}
var myReceiver = {
  foo: 4
}
Reflect.set(myObj, 'bar', 1, myReceiver)
myObj.foo // 2
myReceiver.foo // 1
```



### `Reflect.has(obj, name)`

`Reflect.has` 方法对应 name in obj 里面 in 运算符

```js
var myObj = {
  foo: 1
}
// 旧写法
'foo' in myObj // true
// 新写法
Reflect.has(myObj, 'foo') // true
```

- 如果 `Reflect.has` 方法第一个参数不是对象，会报错

```js
Reflect.has(1, 'foo') // Reflect.has called on non-object
```



### `Reflect.deleteProperty(obj, name)`

`Reflect.deleteProperty` 方法等同于 delete obj[name], 用于删除对象的属性，返回一个布尔值：如果删除成功，或者被删除属性不存在返回 true; 删除失败，被删除属性依然存在返回 false

```js
var myObj = {
  foo: 'bar'
}
// 旧写法
delete myObj['foo']
// 新写法
Reflect.deleteProperty(myObj, 'foo')
```

- 如果第一个参数不是对象则报错

```js
Reflect.deleteProperty(1, 'foo') // Reflect.deleteProperty called on non-object
```



### `Reflect.apply(func, thisArg, args)`

`Reflect.apply` 方法等同 Function.prototype.apply.call(func, thisArg, args)，用于绑定 this 对象后执行给定函数

一般如果绑定一个函数的 this 对象，可写成 fn.apply(obj, args), 但如果函数定义了自己的 apply 方法，只能写 Function.prototype.apply.call(fn, obj, args), 使用 Reflect 对象能简化操作

```js
var args = [1, 2]
// 旧写法
Object.prototype.toString.call(args)
Math.min.call(Math, args)
// 新写法
Reflect.apply(Object.prototype.toString, args, [])
Reflect.apply(Math.min, Math, args)
```



### `Reflect.contruct(obj, args)`

`Reflect.contruct` 方法等同 new obj(...args), 这提供一种不用 new 来调用构造函数的方法

```js
function Greeting(name) {
  this.name = name
}
// new 的写法
const instance = new Greeting('jack')
// 新写法
const instance = Reflect.construct(Greeting, ['jack'])
```

- 如果第一个参数不是函数会报错

```js
Reflect.construct({}, 'jack') // #<Object> is not a constructor
```



### `Reflect.getPrototypeOf(obj)`

`Reflect.getPrototypeOf` 用于读取对象 `__proto__`, 对应 Object.getPrototypeOf(obj)

```js
var myObj = new Object()
// 旧写法
Object.getPrototypeOf(myObj) === Object.prototype
// 新写法
Reflect.getPrototypeOf(myObj) === Object.prototype
```

- Reflect.getPrototypeOf 和 Object.getPrototypeOf 的一个区别是，如果参数不是对象，Object.getPrototypeOf 会将这个对象转为对象，再运行，而 Reflect.getPrototypeOf 会报错

```js
Object.getPrototypeOf(1) // Number {[[PrimitiveValue]]: 0}
Reflect.getPrototypeOf(1) // Reflect.getPrototypeOf called on non-object
```



### `Relect.setPrototypeOf(obj, newProto)`

`Reflect.setPrototypeOf` 方法用于设置目标对象的原型，对应 Object.setPrototypeOf(obj, newProto), 返回一个布尔值表示是否设置成功

```js
var myObj = {}
// 旧写法
Object.setPrototypeOf(myObj, Array.prototype)
// 新写法
Reflect.setPrototytpeOf(myObj, Array.prototype)
myObj.length // 0
```

- 如果无法设置目标对象原型（如目标对象禁止扩展），Reflect.setPrototypeOf 返回 false

```js
var myObj = {}
Object.preventExtensions(myObj) // Object.freeze(myObj)
Reflect.setPrototypeOf(myObj, null)
```

- 如果第一个参数不是对象，Object.setPrototypeOf 返回第一个参数本身，Reflect.setPrototypeOf 报错

```js
Object.setPrototypeOf(1, {}) // 1
Reflect.setPrototypeOf(1, {}) // Reflect.setPrototypeOf called on non-object
```

- 如果第一个参数是 undefined 或 null，Reflect.setPrototypeOf 和 Object.setPrototypeOf 都会报错

```js
Object.setPrototypeOf(null, {}) // Object.setPrototypeOf called on null or undefined
Reflect.setPrototypeOf(null, {}) // Reflect.setPrototypeOf called on non-object
```



### `Reflect.defineProperty(target, name, attributes)`

`Reflect.defineProperty` 方法等同 Object.defineProperty, 用于为对象定义属性

```js
function fn() {}
// 旧写法
Object.defineProperty(fn, 'foo', {
  value: 1
})
// 新写法
Reflect.defineProperty(fn, 'foo', {
  value； 1
})
```

- 如果第一个参数不是对象会报错

```js
Reflect.defineProperty(1, 'foo', {
  value: 1
})
// Reflect.defineProperty called on non-object
```

- 结合 Proxy.defineProperty 配合使用

```js
var proxy = new Proxy({}, {
  defineProperty(target, name, attibutes) {
    return Reflect.defineProperty(target, name, attibutes)
  }
})
proxy.foo = 1
proxy.foo // 1
```



### `Reflect.getOwnPropetyDescriptor(target, name)`

``Reflect.getOwnPropetyDescriptor` 基本等同 Object.getOwnPropertyDescriptor, 用于得到指定属性的描述对象

```js
var myObj = {}
Object.defineProperty(myObj, 'hidden', {
  value: true,
  enumrable: true
})
// 旧写法
Object.getOwnPropertyDescriptor(myObj, 'hidden')
// 新写法
Reflect.getOwnPropertyDescriptor(myObj, 'hidden')
```

- 第一个参数不是对象， Reflect.getOwnPropertyDescriptor 报错， Object.getOwnPropertyDescriptor 会返回 undefined

```js
Object.getOwnPropertyDescriptor(1, 'foo') // undefined
Reflect.getOwnPropertyDescriptor(1, 'foo') // Reflect.getOwnPropertyDescriptor called on non-object
```



### `Reflect.isExtensible(target)`

`Reflect.isExtensible` 对应 Object.isExtensible, 返回一个布尔值，表示当前对象是否可扩展

```js
var myObj = {}
// 旧写法
Object.isExtensible(myObj) // true
Reflect.isExtensible(myObj) // true
```

- 如果参数不是对象，Object.isExtensible 返回 false，因为非对象本身不可扩展，Reflect.isExtensible 报错

```js
Object.isExtensible(1) // false
Reflect.isExtensible(1) // 报错
```



### `Reflect.preventExtensions(target)`

`Reflect.preventExtensions` 对应 Object.preventExtensions 方法，用于让一个对象变为不可扩展，返回一个布尔值，表示是否操作成功

```js
var myObj = {}
// 旧写法
Object.preventExtensions(myObj) // {}
Reflect.preventExtensions(myObj) // true
```

- 如果参数不是对象，Object.preventExtensions 在 es5 环境报错，es6 环境返回传入参数，Reflect.preventExtensions 报错

```js
// es5
Object.preventExtensions(1) // 报错
// ES6 环境
Object.preventExtensions(1) // 1
// 新写法
Reflect.preventExtensions(1) // 报错
```



### `Reflect.ownKeys(target)`

`Reflect.ownKeys` 方法返回对象的所有属性，基本等同 Object.getOwnPropertyNames 和 Object.getOwnPropertySymbols 之和

```js
var myObj = {
  foo: 1,
  [Symbol.for('bar')]: 2
}
// 旧写法
Object.getOwnPropertyNames(myObj) // ['foo']
Object.getOwnPropertySymbols(myObj) // [Symbol(baz)]
// 新写法
Reflect.ownKeys(myObj)
// ['foo', Symbol(baz)]
```

- 如果第一个参数不是对象会报错

```js
Reflect.ownKeys(1) // Reflect.ownKeys called on non-object
```



##  使用 Proxy 实现观察者模式

这里简易使用函数自动观察数据对象，一旦对象有变化，函数自动执行

- 定义一个 Set 集合，所有观察者函数放入这个集合
- 函数返回对原始对象的代理，拦击赋值操作
- 拦截 set 操作中会自动执行所有观察者

```js
const queuedObservers = new Set()
const observe = fn => queuedObservers.add(fn)
const observable = obj => new Proxy(obj, {
  set(target, name, value, receiver) {
		const result = Reflect.set(target, name, value, receiver)
    queuedObservers.forEach(observer => observer())
    return result
  }
})

const person = observable({
	name: 'jack',
  age: 20
})
function print() {
  console.log(`${person.name}, ${person.age}`)
}
observe(print)
person.name = 'jake' // jake, 20

```

