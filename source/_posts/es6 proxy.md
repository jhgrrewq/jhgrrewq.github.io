---
title: es6 proxy
tag:
	- es6
---

> 参考: [ECMAScript 6 入门 proxy 部分](http://es6.ruanyifeng.com/#docs/proxy)

<!-- markdownlint-disable MD010 -->

## 概述

Proxy 用于修改某些操作的默认行为，等同于在语言层变进行修改，属于“元编程”，即对编程语言进行编程

oxy 可理解为在目标对象之前进行一层拦截，外界对该对象的访问，必须先通过这层拦截，因此提供一种机制可以对外界的访问进行过滤和改写

<!-- more -->

## Proxy 实例

```js
new Proxy(target, handler)
```

es6 提供 Proxy 构造函数，生成 Proxy 实例。`target` 参数表示要拦截的目标对象， `handler` 参数是一个配置对象，用来定义拦截行为

这里要注意两点：

- Proxy 起作用必须针对 Proxy 实例进行操作，而不是目标对象进行操作
- `handler` 参数如果没有设置任何拦截，就等同于直接通向原对象

```js
var target = {}
var handler = {}
var proxy = new Proxy(target, handler)
target.a // 不起作用
proxy.a = 'b'
target.a // 'b'
```

## Proxy 实例方法

Proxy 共支持 13 种拦截操作：

- `get(target, propKey, receiver)`:  **拦截对象属性的读取，如 proxy.foo 和  proxy['foo']**
- `set(target, propKey, value, receiver)`: **拦截对象属性设置， 如 proxy.foo = v 或 proxy['foo'] = v, 返回一个布尔值**
- `has(target, propKey)`: **拦截 propKey in proxy 操作，返回一个布尔值**
- `deleteProperty(target, propKey)` : **拦截 delete proxy[propKey] 操作，返回一个布尔值**
- `ownKeys(target)`: 拦截 Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in 循环，返回一个数组。该方法返回目标对象所有自身属性的属性名，而 Object.keys() 返回目标对象自身的可遍历属性
- `getOwnPropertyDescriptor(target, propKey)`: 拦截 Object.getOwnPropertyDescriptor(target, propKey)，返回属性的描述对象
- `defineProperty(target, propKey, proDesc)`: 拦截 Object.defineProperty(proxy, propKey, propDesc)、Object.definePropertys(proxy, propDescs)，返回一个布尔值
- `preventExtensions(target)`: 拦截 Object.preventExtensions(proxy), 返回一个布尔值
- `getPrototypeOf(target)`:  拦截 Object.getPrototypeOf(proxy), 返回一个对象
- `isExtensible(target)`: 拦截 Object.isExtensible(proxy), 返回一个布尔值
- `setPrototypeOf(target, proto)`:  拦截 Object.setPrototypeOf(proxy, proto), 返回一个对象。 如果目标对象是函数，还设有两种额外操作可以拦截
- `apply(target, object, args)`: **拦截 Proxy 实例作为函数调用的操作，如 proxy(...args)、proxy.call(object, ...args)、prox.apply(...args)**
- `construct(target, args)`： **拦截 Proxy 实例作为构造函数的操作，如 new proxy(...args)**

### `get()`

`get` 方法可以用于拦截某个属性的读取操作，可接受三个参数，分别为目标对象、属性名和操作行为针对的对象（一般是 proxy 实例本身, 参数可选）

```js
var person = {
  name: '张三'
}
var proxy = new Proxy(person, {
  get: function(target, property) {
    if (property in target) {
      return target[property]
    } else {
      throw new ReferenceError("Property \"" + property + "\" does not exist.")
    }
  }
})
proxy.name // 张三
proxy.age // 抛出错误
```

- `get` 方法可以继承

```js
let proto = new Proxy({}, {
  get(target, propKey, receiver) {
		console.log('get ' + propKey)
    return target[propKey]
  }
})
let obj = Object.create(proto)
obj.foo // get foo
```

- `get` 方法第三个参数总是指向**原始读操作所在的那个对象**，一般情况就是 Proxy 实例

```js
const proxy = new Proxy({} , {
  get(target, propKey, receiver) {
		return receiver
  }
})
proxy.getReceiver === proxy // true
const d = Object.create(proxy)
d.a === d // true (d 对象没有 a 属性，读取 d.a 会去 d 的原型 proxy 对象找，此时 receiver 指向 d)
```

- 如果属性不可配置 （configurable）并且不可写（writable), 则 Proxy 不能修改该属性，否则通过 Proxy 对象访问该属性会报错

```js
const target = Object.defineProperties({}, {
	foo: {
    value: 123,
    writable: false,
    configurable: false
  }
})
const handler = {
  get(target, propKey) {
		return 'abc'
  }
}
const proxy = new Proxy(target, handler)
proxy.foo // TypeError: Invariant check failed
```



### `set()` 

`set` 方法用于拦截某个属性的赋值操作，可接受四个参数，分别为目标对象、属性名、属性值和操作行为针对的对象（一般是 proxy 实例本身, 参数可选）

```js
// 对象内部属性，如属性名第一个字符使用下划线，表示属性不应该被外部使用
const handler = {
  get(target, key) {
    invariant(key, 'get')
    return target[key]
  }
  set(target, key, value) {
    invariant(key, 'set')
    target[key] = value
    return value
  }
}
function invariant(key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`)
  }
}
const target = {}
const proxy = new Proxy(target, handler)
proxy._prop
// Error: Invalid attempt to get private "_prop" property
proxy._prop = 'c'
// Error: Invalid attempt to set private "_prop" property
```

- `set ` 方法第四个参数 receiver 指向**原始操作行为的对象**，一般是 proxy 实例本身

```js
const handler = {
  set(target, propKey, value, receiver) {
    console.log(target, propKey, value, receiver) // {a: 1} 'foo' 'bar' {b: 2}
    target[propKey] = receiver
  }
}
const proxy = new Proxy({a: 1}, handler)
const myObj = {b: 2}
Object.setPrototypeOf(myObj, proxy)
myObj.foo = 'bar'
myObj.foo === myObj // true (myObj 没有 foo 属性，会从原型 proxy 对象找 foo 属性，设置 proxy 对象 foo 属性会触发 set 方法，receiver 指向原始赋值行为所在对象 myObj)
```

- 如果目标对象某个属性不可写且不可配置，`set` 方法将不起作用

```js
const obj = {}
Object.defineProperty(obj, 'foo', {
  value: 'bar',
  writable: false
})
const handler = {
  set(target, propKey, value, receiver) {
		target[propKey] = value
  }
}
const proxy = new Proxy(obj, handler)
proxy.foo = 'baz'
proxy.foo // bar
```

- 严格模式下 `set` 代理如果没有返回 true 会报错

```js
'use strict'
const handler = {
  set(target, propKey, value, receiver) {
    target[propKey] = receiver
    // return false
  }
}
const proxy = new Proxy({}, handler)
proxy.foo = 'foo' // TypeError: 'set' on proxy: trap returned falsish for property 'foo'
```



### `apply()`

`apply` 方法拦截函数的调用、call 和 apply 操作，可接受三个参数，分别是目标对象、目标对象的上下文 this 和目标对象的参数数组

```js
var handler = {
  apply(target, ctx, args) {
    return Reflect.apply(...arguments)
  }
}
function sum(left, right) {
  return left + right
}
var proxy = new Proxy(sum, handler)
proxy(1, 2) // 3
proxy.call(null, 3, 4) // 7
proxy.apply(null, [5, 6]) // 11
```

### `has()`

`has` 方法拦截 hasProperty 操作，即判断对象是否具有某个属性时，这个方法生效，典型操作是 in 运算符

`has` 方法可接受两个参数，分别是目标对象和需查询的属性名

`has` 方法拦截的是 `hasProperty` 操作，而不是 `hasOwnProperty`操作，**即 `has` 方法不判断一个属性是对象自身的属性还是继承的属性**

- 如果源对象不可配置或者禁止扩展， `has` 拦截会报错

```js
var obj = { a: 10 }
Object.preventExtensions(obj)
var proxy = new Proxy(obj, {
	has(target, propKey) {
		return false
  }
})
'a' in proxy // TypeError is thrown 
```

- `for...in` 循环也用到了 in 运算符，但是 `has` 拦截对 for...in 循环不生效

```js
let stu1 = { name: 'jack', num: 59 }
let stu2 = { name: 'jake', num: 98 }
let handler = {
  has(target, propKey) {
    if (propKey === 'num' && target[propKey] < 60) {
      console.log(`${target.name} 不及格`)
      return false
    }
    return propKey in target
  }
}
let proxy1 = new Proxy(stu1, handler)
let proxy2 = new Proxy(stu2, handler)
'num' in proxy1
// jack 不及格
// false
'num' in proxy2
// true
for(let a in proxy1) {
  console.log(proxy1[a])
}
// jack
// 59
for(let a in proxy2) {
  console.log(proxy2[a])
}
// jake
// 98
```



### `construct()`

`construct`  方法用于拦截 new 命令，接受三个参数：目标对象，构造函数的参数对象，以及 new 命令作用的构造函数( 一般是 proxy 实例本身 )

```js
// hanndler 常见写法
var handler = {
	construct(target, args, newTarget) {
		return new target(...args)
  }
}
```

- `construct` 方法返回的必须是一个对象，否则报错

```js
var P = new Proxy(function() {}, {
	construct(target, args) {
    return 1
  }
})
new P() // 报错
```



### `deleteProperty()`

`deleteProperty` 方法用于拦截 delete 操作，

- 若该方法抛出错误或者返回 false，当前属性就无法被 delete 命令删除

```js
var handler = {
  deleteProperty(target, propKey) {
    invariant(propKey, 'delete')
    delete target[propKey]
    return true
  }
}
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
  }
}
var target = { _prop: 'foo' }
var proxy = new Proxy(target, handler)
delete proxy._prop // Error: Invalid attempt to delete private "_prop" property
```

- 目标对象自身的不可配置属性，不能被 `deleteProperty` 方法删除，否则报错

```js
var handler = {
  deleteProperty(target, propKey) {
    delete target[propKey]
    return true
  }
}
var obj = {}
Object.defineProperty(obj, 'prop', {
  value: 'foo',
  configurable: false
})
var proxy = new Proxy(obj, handler)
delete proxy.prop // 'deleteProperty' on proxy: trap returned truish for property 'prop' which is non-configurable in the proxy target
```



### `getOwnPropertyDescriptor()`

`getOwnPropertyDescriptor` 方法拦截 Object.getOwnPropertyDescriptor(), 返回一个属性描述对象或者 undefined

```js
var handler = {
  getOwnPropertyDescriptor(target, propKey) {
		if (key[0] === '_') {
      return 
    }
    return Object.getOwnPropertyDescriptor(target, key)
  }
}
var target = {
  _foo: 'bar',
  baz: 'tar'
}
var proxy = new Proxy(target, handler)
Object.getOwnPropertyDescriptor(proxy, 'wat') // undefined
Object.getOwnPropertyDescriptor(proxy, '_foo') // undefined
Object.getOwnPropertyDescriptor(proxy, 'baz') // { value: 'tar', writable: true, enumerable: true, configurable: true }
```



### `getPrototypeOf()`

`getPrototypeOf` 方法只要用于拦截获取对象原型的操作：

- Object.prototype._proto_
- Object.prototype.isPrototypeOf()
- Object.getPrototypeOf()
- Reflect.getPrototypeOf()
- instanceOf

```js
var proto = {}
var proxy = new Proxy({}, {
  getPrototypeOf(target) {
		return proto
  }
})
Object.getPrototypeOf(proxy) === proto // true
```

- `getPrototypeOf` 方法**返回值必须是对象或者 null**，否则报错

```js
var proto = {}
var proxy = new Proxy({}, {
  getPrototypeOf(target) {
		return 1
  }
})
Object.getPrototypeOf(p) === proto // 'getPrototypeOf' on proxy: trap returned neither object nor null
```

- 如果目标对象不可扩展，`getPrototypeOf` 方法必须返回目标对象的原型对象

### `isExtensible()`

`isExtensible` 方法拦截 Object.isExtensible 操作，**该方法只能返回布尔值，否则返回值会自动转为布尔值**

```js
var proxy = new Proxy({}, {
  isExtensible(target) {
    console.log('called')
    return true
  }
})
Object.isExtensible(proxy) // called // true
```

- 该方法有一个强限制，它的返回值必须和目标哦对象的 isExtensible 属性保持一致，否则就抛出错误

```js
var proxy = new Proxy({}, {
  isExtensible(target) {
    console.log('called')
    return false
  }
})
Object.isExtensible(proxy) // 'isExtensible' on proxy: trap result does not reflect extensibility of proxy target (which is 'true')
```



### `preventExtensions()`

`preventExtensions` 方法拦截 Object.preventExtensions(), **该方法必须返回一个布尔值，否则会自动转为布尔值**

- 该方法只有一个限制，只有目标对象不可扩展（Object.isExtensible(proxy) 为 false), proxy.preventExtensiions 才能返回 true, 否则报错

```js
var proxy = new Proxy({}, {
  preventExtensions(target) {
    return true
  }
})
Object.preventExtensions(proxy) // // Uncaught TypeError: 'preventExtensions' on proxy: trap returned truish but the proxy target is extensible

// Object.isExtensions(proxy) 本身会返回 true, 因此方法会报错，通常要在 proxy.preventExtensions 方法内部调用一次 Object.preventExtensions

var proxy = new Proxy({}, {
	preventExtensions(target) {
    console.log('called')
		Object.preventExtensions(target)
    return true
  }
})
Object.preventExtensions(proxy) // called // Proxy {}
```



### `setPrototypeOf()`

``setPrototypeOf` 方法用于拦截 Object.setPrototypeOf，该方法只能返回布尔值，否则会自动转为布尔值

```js
var handler = {
  setPrototypeOf(target, proto) {
    throw new Error('Changing the prototype is forbidden');
  }
}
var proto = {}
var target = function() {}
var proxy = new Proxy(target, handler)
Object.setPrototypeOf(proxy, proto) // Error: Changing the prototype is forbidden
```

- 如果目标哦对象不可扩展，`setPrototypeOf` 方法不能改变目标对象原型

```js
var obj = {}
Object.preventExtensions(obj)
var handler = {
  setPrototypeOf(target, proto) {
    console.log('set proto')
    return true
  }
}
var proto = {}
var proxy = new Proxy(obj, handler)
Object.setPrototypeOf(proxy, proto) // 'setPrototypeOf' on proxy: trap returned truish for setting a new prototype on the non-extensible proxy target
```



### ownKeys()`

`ownKeys` 方法用来拦截对**对象自身属性的读取操作**：

- Object.getOwnPropertyNames()
- Object.getOwnPropertySymbols()
- Object.keys()
- for...in 循环

使用 Object.keys 方法会有三类属性被 ownKeys 方法自动过滤：

- 目标对象不存在的属性
- 属性名为 Symbol 值
- 不可遍历属性 enumrable

```js
var target = {
  a: 1,
  b: 2,
  c: 3,
  [Symbol.for('secret')]: '4'
}
Object.defineProperty(target, 'key', {
  enumrable: false,
  configurable: true,
  writable: true,
  value: 'static'
})
var handler = {
  ownKeys(target) {
    return ['a', 'd', Symbol.for('secret'), 'key']
  }
}
var proxy = new Proxy(target, handler)
Object.keys(proxy) // ['a'] 过滤掉不存在、属性为为 Symbol、不可遍历属性
```

- `ownKeys`  方法返回的**数组成员只能是字符串或 Symbol 值**，如果有其他类型的值或者返回不是数组，会报错

```js
var proxy = new Proxy({}, {
  ownKeys(target) {
    return [123, true, undefined, null, [], {}]
  }
})
Object.getOwnPropertyNames(proxy) // 123 is not a valid property name
```

- 如果目标对象包含不可配置属性，则该属性必须被 `ownKeys` 方法返回，否则报错

```js
var target = {}
Object.defineProperty(target, 'a', {
  configurable: false,
  enumrable: true,
  value: 10
})
var proxy = new Proxy(target, {
  ownKeys(target) {
    return ['b']
  }
})
Object.getOwnPropertyNames(proxy) // 'ownKeys' on proxy: trap result did not include 'a'
```

- 如果目标对象不可扩展，`ownKeys` 方法返回的数组中必须包含原对象所有属性，并且不能包含更多属性，否则报错

```js
var target = { a: 1 }
Object.preventExtensions(target)
var handler = {
  ownKeys(target) {
		return ['a', 'b']
  }
}
var proxy = new Proxy(target, handler)
Object.getOwnPropertyNames(proxy) // 'ownKeys' on proxy: trap returned extra keys but proxy target is non-extensible
```



## Proxy.revocable()

`Proxy.revocable` 方法返回一个对象，该对象的 proxy 属性是 Proxy 实例，revoke 属性是一个函数，可取消 Proxy 实例

```js
var target = {}
var handler = {}
let {
  proxy,
  revoke
} = Proxy.revocable(target, handler)
proxy.foo = 12
proxy.foo // 12
revoke()
proxy.foo // Cannot perform 'get' on a proxy that has been revoked
```

