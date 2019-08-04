---
title: js 隐式转换
tag: 
	- js
---

> 参考: [前端高手必备：详解 JavaScript 柯里化](https://juejin.im/entry/58b316d78d6d810058678579/)



## 函数的隐式转换

- 在没有定义 toString 和 valueOf 方法时，函数的隐式转换**默认调用 toString 方法将函数内容转为字符串**返回
- 有自定义的 toString 和 valueOf 方法时，函数的隐式转换默认**调用自定义 toString 方法**
- **valueOf 的优先级比 toString 的优先**

```js
function fn() {
  return 20
}
console.log(fn + 10)
/*function fn() {
    return 20;
}10*/

function fn() {
  return 20
}
fn.toString = function() {
  return 20
}
console.log(fn + 10) // 30

function fn() {
  return 20
}
fn.toString = function() {
  return 20
}
fn.valueOf = function() {
  return 10
}
console.log(fn + 10) // 20
```