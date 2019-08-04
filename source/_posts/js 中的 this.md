---
title: js 中的 this
tag: 
	- js
	- this
---

> 总原则: **this 对象是在运行时基于函数的执行环境绑定的，谁调用 this 就指向谁。**在全局函数中，this 指向 window（非严格模式），而函数作为某个对象的方法调用时候，this 指向那个对象，不过，匿名函数的执行环境具有全局性，this 对象通常指向 window。在通过 call apply bind 等方法改变函数执行环境的情况下， this 会指向其他对象。通过 new 一个构造函数，this 指向 new 出来的对象

## 默认绑定全局变量

当函数单独定义和调用时，this 绑定全局变量（非严格模式）

### 全局

**this 为 window（非严格模式）**

```javascript
console.log(this === window) // true
```

<!-- more -->

### 普通函数调用

**this 为window（非严格模式，属于全局的普通函数调用）**

```javascript
function f() {
    console.log(this) // window
}

f() // 相当于 window.f()
```

### 普通函数嵌套调用

**this 为 window （非严格模式，虽然在 obj.fn 内部，但还是普通函数）**

```javascript
var obj = {
    fn: function() {
        function f() {
            console.log(this) // window
        }
        f() // 相当于 window.fn()
    }
}

obj.fn()
```

此时 this 想要保证还是指向该对象，有三种方法：

- **将外部 this 保存在另外一个变量中，嵌套内部函数通过作用域链可以访问上一层函数的变量**

```javascript
var obj = {
    fn: function() {
        let that = this
        function f() {
            console.log(that) // obj {fn: function}
        }
        f()
    }
}

obj.fn()
```

- **内部函数 bind(this) 再进行调用**

```javascript
var obj = {
    fn: function() {
        function f() {
            console.log(this) // obj {fn: function}
        }
        f.bind(this)()
    }
}

obj.fn()
```

- **使用 es6 箭头函数**

-- **箭头函数本身没有自己的 this**
-- **内部 this 在函数声明时确定**
-- **内部 this 为外部代码块的 this，如果外层也是箭头函数，再往上层查找，this 绑定后就不再改变**
-- **箭头函数不可以做构造函数**
-- **箭头函数本身没有 arguments**
-- **如果箭头函数外层是函数，在函数调用时会把外部的 arguments 拿来**

```javascript
var obj = {
    fn: function() {
        var f = () => {
            console.log(this) // obj {fn: function}
        }
        f()
    }
}

obj.fn()
```

## 隐式调用

隐式调用是说函数调用时拥有一个上下文对象，仿佛该函数属于这个对象

### 函数作为对象的一个属性，而且是作为对象的属性调用

**this 为调用的最后一个对象（函数作为对象的一个属性，而且是作为对象属性调用）**

```javascript
var obj = {
    a: 1,
    fn: function() {
        console.log(this.a)
    }
}

obj.fn() // 1


var obj = {
    a: 1,
    b: {
        a: 2,
        fn: function() {
            console.log(this.a) // 2
        }
    }
}
obj.b.fn() // 2 this 指向最后一个调用的对象
```

事件绑定中的 this

```js
<input type="button" id="app">click me</input>

let el = document.querySelector('#app')
el.addEventListener('click', function() {
    // 相当于 el.onclick = function() {console.log(this)} 隐式调用
    // dom 本身
    console.log(this)
})
el.addEventListener('click', () => {
    // 箭头函数 this 为外层作用域 this
    // window (非严格模式)
    console.log(this)
})

// jquery
$('#app').on('click', function (){
    // this 为 dom 本身
    // $(this) 为 dom 对应的 jq 对象
    console.log(this)
    console.log($(this))
})
```

### 函数被赋值给另一个变量，没有作为对象的属性调用

**this 为window（非严格模式，函数也是一个对象，这里 f 和 obj.fn 都是对内存对象的引用）**

```javascript
var obj = {
    fn: function() {
        console.log(this)
    }
}

var f = obj.fn()
f() // window, 相当于 window.f()
```

## 显式绑定

**函数用 call apply bind 调用, 函数传入第一个参数为函数上下文对象并赋给 this。** 如果传入的是简单值，this 绑定为对相应对象，如果传入 null，this 默认绑定全局变量

```javascript
var x = 20
var obj = {
    x: 10
}
var fn = function() {
    console.log(this.x)
}

fn.call(obj)  // 10
fn.call(null) // 20 传入 null 时 this 指向 window(非严格模式)
```

## 构造函数

### 直接调用构造函数

**this 为window（非严格模式，属于全局的构造函数直接调用）**

```javascript
function F() {
    this.name= 'jack'
    console.log(this) // window
}

F() // 相当于 window.F()
```

### 用构造函数 new 一个对象

**this 为 new 出来的对象（包括整个原型链中 this 都指向该对象）**

```javascript
function F() {
    this.name= 'jack'
    console.log(this) // F {name: 'jack'}
}

new F()
```

new 操作符创建一个对象的原理：

- **new 创建一个对象**
- **将构造函数的作用域赋给新对象（因此 this 指向这个新对象），添加 [[prototype]] 连接**
- **执行构造函数中的代码（属性和方法被加入到 新对象）**
- **返回新对象**

对于 new，如果构造函数没有返回值，就返回上面对象；如果构造函数有返回值并且返回值是对象，则 this 指向的就是返回的对象；如果构造函数有返回值并且为一般值， this 指向的是函数的实例

```js
// 构造函数返回值为非对象
function F() {
    this.name= 'jack'
    return 2
}
var f = new F()
f.name // 'jack'

// 构造函数返回值为对象
function F() {
    this.name= 'jack'
    return {
        name: 'kevin'
    }
}
var f = new F()
f.name // 'kevin'
```