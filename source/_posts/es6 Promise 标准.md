---
title: es6 Promise 标准
tag: 
	- Promise
---

> 参考: [八段代码彻底掌握 Promise](https://juejin.im/post/597724c26fb9a06bb75260e8)

<!-- markdownlint-disable MD010 -->

## Promise 的立即执行

```js
var p = new Promise((resolve, reject) => {
  console.log('create new promise')
  resolve('success')
})

console.log('after new promise')

p.then((val) => {
  console.log(val)
})

// create new promise
// after new promise
// success
```

<!-- more -->

**创建 Promise 时，作为参数传入的函数会被立即执行，尽管函数内可能是异步的代码，then 绑定的成功和失败的回调会在 Promise 对象状态改变后异步执行**，因此会先输出 `create new promise`，再输出 `after new promise`。一般是将异步操作封装成 promise，使得变为同步书写

## Promise 三种状态

```js
var p1 = new Promise((resolve, reject) => {
  resolve(1)
})

var p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(2)
  }, 500)
})

var p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(3)
  }, 500)
})

console.log(p1)
console.log(p2)
console.log(p3)

setTimeout(() => {
  console.log(p2)
},1000)

setTimeout(() => {
  console.log(p3)
},1000)

p1.then(value => {
  console.log(value)
})

p2.then(value => {
  console.log(value)
})

p3.catch(err => {
  console.log(err)
})

// Promise {[[PromiseStatus]]:"resolved",[[PromiseValue]]:1}
// Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}
// Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}
// 1
// 2
// 3
// Promise {[[PromiseStatus]]: "resolved", [[PromiseValue]]: 2}
// Promise {[[PromiseStatus]]: "rejected", [[PromiseValue]]: 3}
```

**Promise 有三种状态：pending、resolved、rejected**

- **当 Promise 刚创建完成，处于 pending 状态**
- **当 Promise 中的参数函数中执行 resolve 方法后，Promise 由 pending 状态变为 resolved 状态**
- **当 Promise 中的参数函数中执行 reject 方法后，Promise 状态会从 pending 变为 rejected 状态**


p2 p3 刚创建完成时候，这两个 Promise 参数函数内代码执行的是异步操作，因此 Promise 处于 pending 状态。p1 的参数函数中执行的是同步代码，Promise 刚创建完成，resolve 方法就调用了，因此 p1 的状态变成了 resolved。setTimeout 1s 后调用，p2 p3 已经异步执行完成，状态分别变成了 resolved 和 rejected

## Promise 状态不可逆性

```js
var p1 = new Promise((resolve, reject) => {
  resolve('p1 ok 1')
  resolve('p1 ok 2')
})

var p2 = new Promise((resolve, reject) => {
  resolve('p2 ok 1')
  reject('p2 err')
})

p1.then(value => {
  console.log(value)
})

p2.then(value => {
  console.log(value)
})

// p1 ok 1
// p2 ok 2
```

**Promise 状态一旦变为 resolved 或者 rejected，就不能再改变它的状态和值**，无论后续再怎么调用 resolve 或 reject 方法

## Promise 链式调用

```js
var p = new Promise((resolve, reject) => {
  resolve(1)
}).then(value => {
  console.log(value) // 1
  return value * 2 // return 2, 返回一个 resolved 状态，值为 2 的 Promise
}).then(value => {
  console.log(value) // 2
  // 默认返回 undefined, 返回一个 resolved 状态，值为 undefined 的 Promise
}).then(value => {
  console.log(value) // undefined
  return Promise.resolve('resolve') // 返回一个 resolved 状态，值为 'resolve' 的 Promise
}).then(value => {
  console.log(value) // 'resovle'
  return Promise.reject('reject') // 返回一个 rejected 状态，值为 'reject' 的 Promise
}).then(value => {
  console.log('resolve:' + value)
}, err => {
  console.log('reject:' + err)
})

// 1
// 2
// undefined
// "resolve"
// "reject: reject"
```

Promise 对象的 then 方法返回一个新的 Promise 对象，因此可以通过链式调用 then 方法。then 方法接收两个参数，第一个参数是 Promise 执行成功时候的回调，第二个参数是 Promise 执行失败时候的回调。

**两个函数只有一个会被调用，函数的返回值将被用作创建 then 返回的 Promise 对象。**这两个参数的返回值可以是如下的一种情况：

- **return 一个同步的值 或 undefined（当没有返回一个有效值，默认返回 undefined）：** then 方法返回一个 resolved 状态的 Promise 对象，Promise 对象的值就是这个返回值
- **return 另一个 Promise：** then 方法将根据这个 Promise 的状态和值创建一个新的 Promise 对象返回
- **throw 一个同步异常：** then 方法将返回一个 rejected 状态的 Promise, 值是该异常

## Promise then() 回调异步性

```js
var p = new Promise((resolve, reject) => {
  resolve('ok')
}).then(value => {
  console.log(value)
})

console.log('first')

// first
// ok
```

**Promise 接收的函数参数是立即执行的，但 then 方法中注册的回调函数是异步执行的。**

- **Promise then() 回调函数必须异步执行**，是为了保证代码执行顺序的一致性，**其内部是将回调函数用 setTimeout 包裹了一层**
- **当 Promise 状态变为 resolved（resolve 方法调用后的成功状态），要按照绑定顺序依次执行 onFulfilled 方法（成功回调）**。如果 then 中注册的方法不是异步执行的，就会有 onFulfilled 方法不是按照绑定顺序执行，中间可能插入 then 方法链式调用后返回的新 Promise 对象绑定的回调函数提前执行，这明显不符合预期
- 用成功回调函数为例，如果是异步执行，回调用 setTimeout 包裹了一层，依次 push 进入成功回调数组，当 promise 为 resovled 状态后，会遍历成功回调数组对每个元素方法进行调用，会同样按照顺序排队进入事件循环队列，最后再依次进入主线程执行。但如果是回调数组元素是同步执行，无法保证哪个数组元素会提前执行完毕

## Promise 中的异常

```js
var p1 = new Promise((resolve, reject) => {
  bar()
  resolve(1)
}).then(value => {
  console.log('p1 then value : ' + value)
}, err => {
  console.log('p1 then err : ' + err)
}).then(value => {
  console.log('p1 then value : ' + value)
}, err => {
  console.log('p1 then err : ' + err)
})

var p2 = new Promise((resolve, reject) => {
  resolve(1)
}).then(value => {
  console.log('p2 then value : ' + value)
  bar()
}, err => {
  console.log('p2 then err : ' + err)
}).then(value => {
  console.log('p2 then value : ' + value)
}, err => {
  console.log('p2 then err : ' + err)
  return 1
}).then(value => {
  console.log('p2 then value : ' + value)
}, err => {
  console.log('p2 then err : ' + err)
})

// p1 then err : ReferenceError: bar is not defined
// p2 then value : 1
// p1 then value : undefined
// p2 then err : ReferenceError: bar is not defined
// p2 then value : 1
```

Promise 中的异常由 then 参数的第二个回调函数（Promise 执行失败的回调）或者 catch 注册的回调进行处理，异常信息作为 Promise 的值。**异常一旦得到处理，then 返回的后续 Promise 对象将恢复正常，并被 Promise 执行成功的回调函数处理**

## Promise.resolve()

```js
var p1 = Promise.resolve(1)
var p2 = Promise.resolve(p1)
var p3 = new Promise((resolve, reject) => {
  resolve(1)
})
var p4 = new Promise((resolve, reject) => {
  resolve(p1)
})

console.log(p1 === p2)
console.log(p1 === p3)
console.log(p1 === p4)
console.log(p3 === p4)
```

Promise.resolve() 方法可以接受**一个值**或**一个 Promise 对象**

- **当参数是普通值，它返回一个 resolved 状态的 Promise 对象，对象的值就是这个参数**
- **当参数是一个 Promise 对象，直接返回这个 Promise 参数**

因此 p1 === p2, 但是通过 new 创建的 Promise 对象都是新对象，彼此并不相等

## resolve / reject

```js
var p1 = new Promise((resolve, reject) => {
  resolve(Promise.resolve('resolve'));
})

var p2 = new Promise((resolve, reject) => {
  resolve(Promise.reject('reject'));
})

var p3 = new Promise((resolve, reject) => {
  reject(Promise.resolve('resolve'));
})

p1.then(value => {
  console.log('p1 fulfilled: ' + value);
}, err => {
  console.log('p1 rejected: ' + err);
})

p2.then(value => {
  console.log('p2 fulfilled: ' + value);
}, err => {
  console.log('p2 rejected: ' + err);
})

p3.then(value => {
  console.log('p3 fulfilled: ' + value);
}, err => {
  console.log('p3 rejected: ' + err);
})

// p3 rejected: [object Promise]
// p1 fulfilled: resolve
// p2 rejected: reject
```

**resolve 方法的参数是一个 Promise 对象时，会去获取 Promise 的状态和值，这个过程是异步的**。当 p1 获取到 Promise 对象状态是 resolved，因此 fulfilled 回调被异步执行；p2 同理

**但是 reject 方法的参数会直接传递给 then 方法中的 rejected 回调，不管参数是不是 Promise 对象**。因此 p3 reject 方法接受了一个 resolved 状态的 Promise，then 方法中被调用仍然是 rejected 回调，并且参数就是 reject 方法接受的 Promise 对象