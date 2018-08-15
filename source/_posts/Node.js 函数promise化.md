---
title: Node.js 函数promise化
tag: 
	- Node.js
	- promise
---

## 概念

> 指将带有 callback 函数重新变成 promise 来实现

## 使用

> 需要引入一个 第三方 promise 库如 bluebird，**利用其 promisify 和 promisifyall 方法**

<!-- more -->

### **promisify 方法**

需要 nodeCallback（错误优先的回调）回调函数转为 promise（nodejs标准包的异步回调几乎是nodeCallback）

~~~javascript
var Promise = require('bluebird')
var fs = require('fs')

// 异步回调形式
fs.readFile('./test.js',function(err, data){
    console.log(data)
})

// promisify 形式
const readFileAsync = Promise.promisify(fs.readFile)

readFileAsync('./test.js').then(function(data){
    console.log(data)
}).catch(e){
    console.log(e)
}
~~~

### **promisifyAll 方法**

**将一个库的所有方法全部转化为 promise 实现，并将原有函数名加上 Async**

~~~javascript
var Promise = require('bluebird')

// promisifyall 形式
const fs = Promise.promisifyAll(require('fs'))

fs.readFileAsync('./test.js').then(function(data){
    console.log(data)
}).catch(e){
    console.log(e)
}
~~~

## 实现自己的 promisify

### **单一函数封装**

~~~javascript
// 如对fs的异步读取文件函数封装promise
// 返回一个promise 同时针对回调的不同参数返回不同的处理结果，方便之后调用

var promisify = function(fpath, encoding){
    return new Promise(function(resolve, reject){
        fs.readFile(fpath, encoding, function(err, result){
            if(err) return reject(err)
            else return resolve(result)
        })
    })
}
~~~

### **函数一般化**
 > 将需要转化的函数当成一个方法传入，因为不同函数接受的参数不同，需要用到methods.apply()

~~~javascript
var promisify = function (method, ctx) {
    // 返回一个函数，函数内部返回一个 promise
    return function () {
        // 获取method调用的需要参数
        var args = Array.prototype.slice.call(arguments, 0);
        // use runtime this if ctx not provided
        ctx = ctx || this;

        //返回一个新的Promise对象
        return new Promise(function (resolve, reject) {
            // 除了函数传入的参数以外还需要一个callback函数来供异步方法调用
            // 因为要返回一个 promise 这里的回调是为了能在方法调用后改变 promise 状态
            var callback = function () {
                return function (err, result) {
                    if (err) {
                        return reject(err);
                    }
                    return resolve(result);
                }
            }
            args.push(callback());
            // 调用method
            method.apply(ctx, args);
        });
    };
};

// 调用
var readFileAsync = promisify(fs.readFile)
readFileAsync('./test.txt', 'utf8').then(function(data){
  console.log(data)
})
~~~