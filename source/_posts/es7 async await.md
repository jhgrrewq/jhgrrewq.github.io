---
title: es7 async / await
tag: 
	- async / await
---

处理异步时候，除了使用异步回调，promise，generator 函数 + co，还可以使用最新的 async / await

> async / await 其实一种语法糖

## async 函数执行后会自动返回一个 promise 对象

```js
let fn = async function() {}
console.log(fn()) // async 函数里没有任何代码，执行后直接返回一个 promise 对象
```

<!-- more -->

## await 只能在 async 函数里使用

```js
var fn = function() {
  let result = await Promise.resolve('ok')
  console.log(result)
}
fn() // await is only valid in async function

fn = async function() {
  let result = await Promise.resolve('ok')
  console.log(result)
}
fn() // ok
```

## async 函数执行时，遇到 await 先返回，等异步操作完成

- await 之后不是 promise，等到的就是表达式的值

- await 之后是 promise，**会等到 promise 对象执行成功后 resolve 出来的参数。因此 await 后的 promise 对象不必添加 then 注册成功回调**

```js
(async function() {
  let result1 = await 1
  console.log(result1)
  let result2 = await Promise.resolve('ok')
  console.log(result2)
})()

// 1
// ok
```

## async / await 错误捕获

- **await 后跟 promise 时会直接等到 promise 执行成功 resolve 出来的参数，但是捕获不到失败状态**。因为 async 本身可以返回一个 promise 对象，因此可以给 async 函数添加 then / catch 方法注册成功和失败回调

```js
let p = new Promise(function(resolve, reject) {
  setTimeout(function() {
    let r = Math.random()
    if (r > 0.5) {
      resolve('ok')
    } else {
      reject('err')
    }
  })
})
let fn = async function() {
  let result = await p
  return result // 这里需要返回，对于出错的 promise 无法捕获
}

fn().then(function(data) {
  console.log(data)
}, function(err) {
  console.log(err)
})
```

- 使用 try / catch (因为 async / await 已经将 异步变为 同步书写)

```js
let p = new Promise(function(resolve, reject) {
  setTimeout(function() {
    let r = Math.random()
    if (r > 0.5) {
      resolve('ok')
    } else {
      reject('err')
    }
  })
})
let fn = async function() {
  try{
    let result = await p
    console.log(result)
  } catch(e) {
    console.log(e)
  }
}

fn()
```