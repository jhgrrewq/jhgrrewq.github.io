---
title: js call apply bind
tag: 
	- js
---

> 参考: [JavaScript深入之bind的模拟实现](https://github.com/mqyqingfeng/Blog/issues/12)

<!-- markdownlint-disable MD010 -->

## call apply bind 方法的异同：

- 三者都是改变函数 this 对象指向的
- **三者第一个参数都是 this 要指向的对象（第一个参数不传或者传 null undefined, this 指向 window，非严格模式）**
- 三者都可以使用后续参数传参
- **call apply 是立即调用，bind 是返回对应函数，延迟调用**

<!-- more -->

## call 和 apply

当前对象无某个方法，借 call 和 apply 其他对象的方法，改变函数 this 指向当前传入的对象，使当前对象也能调用这个方法

> 两者作用相同，接受参数的形式不同:
- call **将参数按顺序传递**
- apply **将参数作为数组传递**

对于 js 中参数数量明确时用 call，对于参数数量不确定时使用 apply，将参数 push 进数组传递进去，也可以在函数内部使用 arguments 遍历参数

```js
// 数组合并
let arr1 = [1, 2, 3]
let arr2 = [4, 5]
console.log(Array.prototype.concat.apply(arr1, arr2)) // [1, 2, 3, 4, 5]

// 求最值 Math.min(1, 2, 3, 4, 5) Math.max(1, 2, 3, 4, 5)
let num = [1, 2, 3, 4, 5]
console.log(Math.min.call(Math, 1, 2, 3, 4, 5)) // 1
console.log(Math.min.apply(Math, [1, 2, 3, 4, 5])) // 1

// 将伪数组转化为数组
function test() {
    let arg = Array.prototype.slice.call(arguments)
    console.log(arg instanceof Array)
}
test(1) // true
```

## bind

> bind()方法会创建**返回一个新函数**，称为绑定函数，当调用该绑定函数时，绑定函数会以创建它时传入 **bind()方法的第一个参数作为 this，传入 bind() 方法的第二个以及以后的参数加上绑定函数运行时本身的参数**按顺序作为原函数的参数来调用原函数。

重点在于 bind() 会返回一个新的绑定函数，不会立即执行

### bind 简单实现

```js
Function.prototype.bind = function(ctx) {
    // fn.bind() 这里 this 为 fn
    var self = this
    // 获取 bind 方法第二个之后参数
    var bindArg = Array.prototype.slice(arguments, 1)
    // 返回一个新的绑定函数，该函数返回 绑定后函数的调用
    return function () {
        // 获取绑定函数运行时的参数
        var arg = Array.prototype.slice(arguments)
        return self.apply(ctx, bindArg.concat(arg))
    }
}
```

### 构造函数 bind 实现

bind 返回的函数作用在构造函数上，绑定的 this 会失效，但是传入的参数依然有效

```js
var a = 1
var foo = {
    a: 2
}

function Bar(name, age) {
    console.log(this.a)
    console.log(name)
    console.log(age)
}

Bar.prototype.test = 'test'

var bindBar = Bar.bind(foo, 'jack')

var bar = new bindBar(20)
// undefined 这是 this 指向 bar 对象
// 'jack'
// 20
```

改进一下之前的 bind 方法

```js
Function.prototype.bind = function(ctx) {
    // fn.bind() 这里 this 为 fn
    var self = this
    // 获取 bind 方法第二个之后参数
    var bindArg = Array.prototype.slice(arguments, 1)

    var fNOP = function () {}
    // 返回一个新的绑定函数，该函数返回 绑定后函数的调用
    var fBound =  function () {
        // 获取绑定函数运行时的参数
        var arg = Array.prototype.slice(arguments)
        // 当该绑定函数作为普通函数调用，this 为 window，this instanceof fNOP 为 false，应该将 this 指向 ctx（传入的对象）
        // 当该绑定函数作为构造函数调用时，this 为函数实例，this instanceof fNOP 为 true，将 this 指向该函数实例
        return self.apply(this instanceof fNOP ? this : ctx || window, bindArg.concat(arg))
    }
    // 继承 返回的新绑定函数 fnBound 就拥有了原有对象函数的所有属性方法 （也就是 fn.bind() 的 fn）
    fNOP.prototype = self.prototype
    fnBound.prototype = new fNOP()
    return fBound
}
```

### MDN 上 bind 实现

```js
if (!Function.prototype.bind) {
    Function.prototype.bind = function(oThis) {
        // fn.bind() 这里的 this 为 fn, fn 必须为函数，否则报错
        if (typeof this !== 'function') {
            // closest thing possible to the ECMAScript 5 internal IsCallable function
            throw new TypeError("Function.prototype.bind - what is trying to be bound is not callable");
        }
        // fn.bind() 传入的除去第一个参数（改变 this 指向）的参数，作为参数数组
        let arg = Array.prototype.slice.call(arguments, 1)
        let fnOp = function() {}
        let fnToBind = this

        let fnBound = function() {
            // 新创建一个绑定函数，第一个参数是 bind() 的第一个参数, arg.concat(Array.prototype.slice.call(arguments)) 是将 bind() 的第二个参数以及之后的参数和绑定函数运行时本身的参数传入
            return fnToBind.apply(this instanceof fnOp && oThis ? this : oThis || window,
                arg.concat(Array.prototype.slice.call(arguments)))
        }

        // 继承 返回的新绑定函数 fnBound 就拥有了原有对象函数的所有属性方法 （也就是 fn.bind() 的 fn）
        fnOp.prototype = this.prototype
        fnBound.prototype = new fnOp()

        return fnBound
    }
}
```

```js
var first = function(a){
    console.log(this.x, a);
}
var sec = {
    x:1
}
var third = {
    x:2
}
var func1 = first.bind(sec,1);
func1()
// {x: 1} [1]
// 1
var func2 = first.bind(sec,1).bind(third,2);
func2();
// {x: 2} [2]
// {x: 1} [1, 2]
// 1
```

通过源码分析，bind() 返回新绑定函数，多次绑定，bind(this) this 最终覆盖指向为第一个 bind() 的第一个参数， 而多次 bind() 除去第一个参数的参数会合并成参数数组