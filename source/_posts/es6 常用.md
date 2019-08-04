---
title: es6 常用
tag:
	- es6
---

<!-- markdownlint-disable MD010 -->

> 参考: [向 ES6 看齐，用更好的JavaScript（一）](http://www.cnblogs.com/vicfeel/p/5808277.html) [向 ES6 看齐，用更好的JavaScript（二）](http://www.cnblogs.com/vicfeel/p/5822068.html)

es6 常用特性有:

- **let/const 块级作用域**
- **解构赋值**
- **多行字符/模板字符串**
- **函数默认值**
- **箭头函数**
- **class**

## let 和 const 引入块级作用域

以往 js 是不存在块级作用域，只有全局作用域和函数作用域

```js
var a = []
for (var i = 0; i < 3; i++) {
  a[i] = function() {
    console.log(i)
  }
}

a[1]() // 3
```

<!-- more -->

因为没有块级作用域，i 变量仅用于循环，却被泄露成全局变量形成变量污染。上述形成的**闭包**在以往的解决方案有**构造一个立即执行函数，将变量 i 传入**

```js
var a = []
for (var i = 0; i < 3; i++) {
  (function(i) {
    // 立即执行函数形成函数作用域，这里 i 变成局部变量
    a[i] = function() {
      console.log(i)
    }
  })(i)
}

a[1]() // 1
```

es6 的 **let 关键字形成仅作用于该块作用域的局部变量**

```js
var a = []
for (let i = 0; i < 3; i++) {
  a[i] = function() {
    console.log(i)
  }
}

a[1]() // 1
i // Uncaught ReferenceError: i is not defined
```

将上述 es6 代码进行 babel 转换后发现其实也是新定义一个函数，将变量作为参数传入

```js
"use strict";

var a = [];

var _loop = function _loop(i) {
  a[i] = function () {
    console.log(i);
  };
};

for (var i = 0; i < 3; i++) {
  _loop(i);
}

a[1]();
```

使用 let 有两个要注意的地方：

- **let 声明的变量不会变量提升，同理 const class** 需要先定义再使用
- **let 声明的变量不能重复定义**

```js
// let 声明的变量不会变量提升
console.log(a)
let a = '1' // Uncaught ReferenceError: i is not defined

// let 声明的变量不能重复定义
let a = '2' // Uncaught SyntaxError: Identifier 'c' has already been declared
```

const 类似 let 也能形成块级作用域，不过 **const 声明的是常量，不可改变**

```js
if (true) {
  // const 声明的常量不会提升
  console.log(num) // Uncaught ReferenceError: num is not defined
  const num = 1
  // const 声明的常量不可改变
  num = 2 // Uncaught TypeError: Assignment to constant variable.
  // const 声明的常量不能重复定义
  const num = 2 // Uncaught SyntaxError: Identifier 'num' has already been declared
}
// const 关键字形成块级作用域
console.log(num) // Uncaught ReferenceError: num is not defined
```

**babel 对 es6 的 let const 声明的局部变量和常量，如果外部有相同定义，会添加下划线转化为另外的变量和常量**

```js
// es6
if (true) {
  let a = 1
  const b = 2
}
console.log(a)
console.log(b)

// babel 转化 es6
"use strict";

if (true) {
  var _a = 1;
  var _b = 2;
}
console.log(a);
console.log(b);
```

## 变量的结构赋值

### 数组的结构赋值

es6 提供了一种便捷的多变量赋值

```js
var arr = [1,2,3];
var [a,b,c] = arr;
```

经过 babel 转换后其实是通过**数组下标依次向后赋值**

```js
"use strict";

var _ref = [1, 2，3];
  var a = _ref[0];
  var b = _ref[1];
  var c = _ref[2];
```

- 数量不对应（不存在某个元素），会赋值 undefined
- 对应值不是数组会报错

```js
// 数量不对应（不存在某个元素），会赋值 undefined
let [a,b,c] = [1,2];
console.log(a);  //1
console.log(b);  //2
console.log(c);  //undefined

// 右边赋值的必须可遍历
let [x, y] = 1 // Uncaught TypeError: 1 is not iterable
```

简单应用：交换变量值

```js
let p1 = 1
let p2 = 2
[p1, p2] = [p2, p1]
console.log(p1) // 2
console.log(p2) // 1
```

**es6 映射赋值可以使用默认值**

```js
let [a, b = 2] = [1] // b 无对应值，默认为 2
console.log(a) // 1
console.log(b) // 2

// babel 转换
"use strict";

var _ref = [1],
    a = _ref[0],
    _ref$ = _ref[1],
    b = _ref$ === undefined ? 2 : _ref$; // 通过 === 严格相等是否是 undefined，如果是使用默认值

console.log(a);
console.log(b);
```

### 对象的解构赋值

```js
// es6
let {a, b} = {a: 'a', b: 'b'}
console.log(a) // 'a'
console.log(b) // 'b'
```

babel 转换其实是通过**同名属性进行对应**

```js
'use strict';

var _a$b = { a: 'a', b: 'b' },
    a = _a$b.a,
    b = _a$b.b;

console.log(a); // 'a'
console.log(b); // 'b'
```

- 顺序改变不影响（对象的属性本身就是无序的)
- 数量不对应时，属性名对应的属性值为 undefined

```js
let {a, b, c} = {c: 'c', a: 'a'}
console.log(a) // 'a'
console.log(b) // undefined
console.log(c) // 'c'
```

简单应用：合并两个对象时，相同的属性会被后面的对象覆盖

```js
// 同名属性被后面对象的覆盖
let newObj = {a: 'a', b: 'b'}
let oldObj = {a: 'c', c: 'c'}
console.log({...newObj, ...oldObj}) // {a: 'c', b: 'b', c: 'c'}
```

## 扩展运算符 ...

### 数组（任何可遍历对象，如 伪数组，Map, Set）的应用

将一个**数组(可遍历对象)转换为逗号分隔的参数序列**

```js
console.log(1, ...[2, 3]) //  1, 2, 3
console.log([1, ...[2, 3]]) // [1, 2, 3]
```

babel 转换

```js
"use strict";

var _console;
// 数组作为 apply 方法的第二个参数传入
(_console = console).log.apply(_console, [1].concat([2, 3]));
// 同样是将数组整个进行操作
console.log([1].concat([2, 3]));
```

同理可以无需使用 apply 方法，使用数组作为参数

```js
// es5
var fn = function(x, y, z) {
  console.log(x, y, z) // 1, 2, 3
}
fn.apply(null, [1, 2, 3])

// es6
var fn = function(x, y, z) {
  console.log(x, y, z)
}
fn(...[1, 2, 3])
```

### 对象的应用

```js
let old = {a: 'a', b: 'c'}
let obj = {...old, c: 'c'}
```

babel 转换其实是**调用一个函数（默认使用 Object.assign），将多个对象的属性合并成一个新对象**

只是对**对象本身的属性浅拷贝**

```js
'use strict';

var _extends = Object.assign || function (target) {
  for (var i = 1; i < arguments.length; i++) {
    var source = arguments[i]; for (var key in source) {
      // 对对象自身的属性浅拷贝
      if (Object.prototype.hasOwnProperty.call(source, key){  target[key] = source[key];
      }
    }
  }
  return target;
};

var old = { a: 'a', b: 'c' };
var obj = _extends({}, old, { c: 'c' });
```

## rest (..., 一般用于函数声明的形参)

rest 操作符同样是 ..., 一般用在函数声明的形参中，但是 **rest 操作符在函数声明作为 函数形参 只能放在最后位置**

```js
function func(a, ...args){
  console.log(a)  // 1
  console.log(args); // [2,3,4]
}

func(1,2,3,4)

// rest 作为函数声明 形参只能作为最后一个参数
function f(...vals,v){ }  // Uncaught SyntaxError: Rest parameter must be last formal parameter
```

babel 转换其实是对 arguments 的模拟

```js
"use strict";

function func(a) {
  console.log(a);

  for (var _len = arguments.length, args = Array(_len > 1 ? _len - 1 : 0), _key = 1; _key < _len; _key++) {
    args[_key - 1] = arguments[_key];
  }

  console.log(args);
}

func(1, 2, 3, 4);
```

## 函数默认参数

es6 之前是无法对函数设置默认参数的，往往是在函数体内进行短路赋值

```js
function(name) {
  name = name || 'defaultName'
  // ...
}
```

es6 提供了函数默认参数

```js
function f(name, age = 23) {
  console.log(name + ':' + age)
}
f('jack') // jack:23
```

babel 转换 其实是**参数进行 === 严格判断是否相等 undefined，是就使用默认参数**

```js
'use strict';

function f(name) {
  var age = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : 23;

  console.log(name + ':' + age);
}
f('jack');
```

## 箭头函数

es6 箭头函数其实是一个语法糖

```js
var add = (a, b) => a + b
// 等同于
var add = function add(a, b) {
    return a + b;
}

// 箭头函数结合 解构赋值
var plus = ({name, age}) => name + age
var person = {
  name:'jack',
  age:23
};
plus(person)

// babel 转换
'use strict';

var plus = function plus(_ref) {
  var name = _ref.name,
    age = _ref.age;
  return name + age;
};
var person = {
  name: 'jack',
  age: 23
};
plus(person);
```

传统函数中 this 对象的指向是可变的

```js
var obj = {
  fn: function() {
    setTimeout(function() {
      console.log(this)
    })
  }
}
// 此时 this 不指向 obj 对象
obj.fn() // window
```

上述代码中传统做法往往通过将 外层 this 暂存一个变量，或者通过 bind 进行绑定

```js
// 将外层 this 暂存
var obj = {
  fn: function() {
    // 将外层 this 暂存
    var that = this
    setTimeout(function() {
      console.log(that)
    })
  }
}
obj.fn() // obj {fn: f}

// 通过 bind 绑定
var obj = {
  fn: function() {
    setTimeout(function() {
      console.log(this)
    }.bind(this))
  }
}
obj.fn() // obj {fn: f}
```

es6 **箭头函数 this 对象是绑定为外层代码块 this**, babel 转化实质也是将外层 this 暂存

```js
var obj = {
  fn: function() {
    setTimeout(() => {
      console.log(this)
    })
  }
}
obj.fn() // obj {fn: f}

// babel 转换
"use strict";

var obj = {
  fn: function fn() {
    var _this = this;

    setTimeout(function () {
      console.log(_this);
    });
  }
};
obj.fn();
```

## 模板字符串

es6 模板字符串避免变量拼接字符串, 模板使用 **``**, 插入变量和变量表达式或者函数通过 **${变量/变量表达式}**

```js
var data = {a: 'a'}
console.log(`this is ${a}`) // 'this is a'

console.log(`this is ${(function() {return 'now'})()}`) // 'now'
```

babel 转换后其实是将变量和变量运算表达式进行输出再拼接字符串

```js
'use strict';

var data = { a: 'a' };
console.log('this is ' + a);

console.log('this is ' + function () {
  return 'now';
}());
```

- 对于插入的变量是数组或者对象，会调用 `toString()` 方法进行转换

```js
var data = {a: 'a'}
console.log(`this is ${data}`) // 'this is [object Object]'

var arr = ['1', 1]
console.log(`this is ${arr}`) // 'this is 1,1'
```

- **模板字符串还允许嵌套**，因此对于数组或者对象能结合 `map` 方便进行转换

```js
var data = {a: 'a', b: 'b'}

var tpl = `
  ${Object.keys(data).map((key) => {
    return `${key}: ${data[key]}`
  })}
`
console.log(tpl) // a: a,b: b
```