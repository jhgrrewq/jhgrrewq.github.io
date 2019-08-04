---
title: jquery 的 deferred
tag: 
	- jquery
---

> 参考: [jQuery的deferred对象详解](http://www.ruanyifeng.com/blog/2011/08/a_detailed_explanation_of_jquery_deferred_object)

## jquery 1.5 之前

```js
// data1.json
{"data": 1}
// data2.json
{"data": 2}

var p1 = $.ajax({
  url: './data1.json',
  success: function(data) {
    console.log(data)
    p2
  }
})
var p2 = $.ajax({
  url: './data2.json',
  success: function(data) {
    console.log(data)
  }
})
console.log(p2, 'p2') // xhr 对象
// {data: 1}
// {data: 2}
```

<!-- more -->

jquery 1.5 之前 ajax 返回的是一个 xhr 对象，无法使用链式书写，$.ajax() 方法接受一个对象作为参数，对象中 success 属性指定成功回调，error 属性指定失败回调，多重异步回调只能嵌套书写

jquery 1.5 之后引入 deferred 对象，使得 $.ajax() 可以链式调用

## deferred 对象

### jquery deferred api

- **$.Deferred()** 创建一个 deferred 对象
- **deferred.done()** 注册操作成功的回调函数
- **deferred.fail()** 注册操作失败的回调函数
- **deferred.then()** 可同时注册操作成功（第一个参数）和操作失败（第二个参数）的回调函数
- **deferred.resolve()** 手动将 deferred 对象的执行状态改为“已完成”，并异步调用 done 方法注册的回调函数
- **deferred.reject()** 手动将 deferred 对象的执行状态改为“已失败”，并异步调用 fail 方法注册的回调函数
- **$.when()** 可以为多个操作注册回调

### jquery deferred 三种状态

**deferred 对象有三种状态，未完成、已完成、已失败**。

- 如果执行状态是 “已完成”，deferred 对象会异步调用 done 方法注册的回调函数（或者是异步调用 then 方法注册的第一个参数回调）
- 如果执行状态是 “已失败”，deferred 对象会异步调用 fail 方法注册的回调函数（或者是异步调用 then 方法注册的第二个参数回调）

**done、fail、then 方法都是被动监听**，deferred 对象也可以主动改变自身状态，**resolve 方法可以将状态从 “未完成” 变为 “已完成”， reject 方法可以将状态从 “未完成” 变成 “已失败”**

```js
var p1 = $.ajax({
  url: './data1.json'
})
var p2 = $.ajax({
  url: './data2.json'
})

p1.done(function(data) {
  console.log(data)
}).then(function() {
  console.log('success')
}, function() {
  console.log('err')
})

$.when(p1, p2).done(function(data1, data2) {
  console.log(data1[0], data2[0])
})

console.log(p2, 'p2') // deferred 对象
// {data: 1}
// {data: 1} {data: 2}
// success
```

因此可将异步操作进行封装，进而链式操作，而不必在异步函数内部进行嵌套操作

```js
var dtd = $.Deferred() // 创建一个 deferred 对象

function wait(dtd, flag) {
  setTimeout(function() {
    console.log('执行完毕')
    if(flag) {
      // 成功
      dtd.resolve() // 改变状态为 “已完成”
    } else {
      // 失败
      dtd.reject() // 改变状态为 “已失败”
    }
  })
  return dtd
}

wait(dtd, true)
  .done(function() {
    console.log('success1')
  })
  .fail(function() {
    console.log('error1')
  })
  // .then(function() {
  //   console.log('success2')
  // }, function() {
  //   console.log('error2')
  // })

// 执行完毕
// success1
// success2

wait(dtd)
  .done(function() {
    console.log('success1')
  })
  .fail(function() {
    console.log('error1')
  })
  // .then(function() {
  //   console.log('success2')
  // }, function() {
  //   console.log('error2')
  // })

// 执行完毕
// error1
// error2
```

deferred 对象同时具有 resolve reject 主动改变状态的方法，因此在外部可以被改变状态, 这明显不符合预期

```js
var dtd = $.Deferred() // 创建一个 deferred 对象

function wait(dtd, flag) {
  setTimeout(function() {
    console.log('执行完毕')
    dtd.resolve() // 改变状态为 “已完成”
  })
  return dtd
}

wait(dtd).reject()
  .done(function() {
    console.log('success1')
  })
  .fail(function() {
    console.log('error1')
  })
  // .then(function() {
  //   console.log('success2')
  // }, function() {
  //   console.log('error2')
  // })

  // error1
  // 执行完毕
  // error2
```

- **deferred.promise()** 返回另一个 deferred 对象，**该对象只开放与改变状态无关的方法 （done fail then），屏蔽与状态改变有关的方法（resolve reject）**，使得状态不能被外部改变

```js
var dtd = $.Deferred() // 创建一个 deferred 对象

function wait(dtd, flag) {
  setTimeout(function() {
    console.log('执行完毕')
    dtd.resolve() // 改变状态为 “已完成”
  })
  return dtd.promise() // 返回一个新的 deferred 对象，屏蔽改变状态的方法
}

wait(dtd).reject() // 报错 外部无法调用改变状态方法
  .done(function() {
    console.log('success1')
  })
  .fail(function() {
    console.log('error1')
  })
  // .then(function() {
  //   console.log('success2')
  // }, function() {
  //   console.log('error2')
  // })

  // error1
  // 执行完毕
  // error2

wait(dtd)
  .done(function() {
    console.log('success1')
  })
  .fail(function() {
    console.log('error1')
  })
  // .then(function() {
  //   console.log('success2')
  // }, function() {
  //   console.log('error2')
  // })

  // 执行完毕
  // success1
  // success2
```


