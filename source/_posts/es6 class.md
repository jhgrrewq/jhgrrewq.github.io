---
title: es6 class
tag:
	- es6
---

> 参考: [ECMAScript 6 入门 class 部分](http://es6.ruanyifeng.com/#docs/class)

<!-- markdownlint-disable MD010 -->

## 概述

js 中生成实例对象的传统方法是通过构造函数结合原型

<!-- more -->

```js
// 构造函数
function Point(x, y) {
  this.x = x
  this.y = y
}

// 原型方法
Point.prototype.toString = function() {
  return "(" + this.x + ", " + this.y + ")"
}

// 静态属性
Point.info = '生成点对象的构造函数'
// 静态方法
Point.genText = function(text){
  console.log(text)
}

var p = new Point(1, 2)
p.toString() // "(1, 2)"
Point.info // '生成点对象的构造函数'
Point.genText('测试') // '测试'
```

es6 的 class 其实可以看做是一种语法糖

```js
class Point {
  // 定义在原型上，能被继承
  constructor(x, y) {
    // constructor 中的属性和方法是定义在实例自身上
    this.x = x
    this.y = y
  }

  toString() {
    return "(" + this.x + ", " + this.y + ")"
  }

  // 添加 static 关键字是静态方法只能是类调用，实例不能调用
  static genText(text){
    console.log(text)
  }
}

// 静态属性只能这么设置 es6 规定类内部只有静态方法，没有静态属性
Foo.info = '生成点对象的构造函数';
```

在 class 中的 **constructor 方法就是构造方法,指向类分身（构造函数），this 关键字代表实例对象，默认会返回 this（实例对象），也可以返回指定的另外一个对象。** constructor 是默认具有的（本身原型中就有 constructor 属性）

class 中的**方法都是挂载在原型上的，都是公共方法，可以在类外部访问，方法之间不需要逗号分隔**，加了会报错

class **内部只能有静态方法，没有静态属性，静态方法名前添加 static 关键字，静态属性和 es5 一样直接给类或构造函数添加属性**

```js
class Point {
  ...
}

// 等同于
class Point {
  constructor() {
    ...
  }
}

typeof Point // 'function'
// contructor 指向构造函数 类本身
Point = Point.prototype.constructor === Point // true

// class 中定义的方法都是挂载原型上的
Point.prototype = {
  constructor() {},
  toString() {}
}

let p = new Point(1, 2)
p.constructor === Point.prototype.constructor // true

p.toString() // "(1, 2)"

// constructor 定义在实例对象 this 上的属性方法
p.hasOwnProperty('x') // true
p.hasOwnProperty('toString') // false 方法是挂载在构造函数原型上，不是实例对象自身具有
p.__proto__.hasOwnProperty('toString') // true 实例的 __proto__ 指向构造函数的原型
```

## 类的方法是不可枚举的

es6 class 中定义的方法是不可枚举的，与 es5 中的不同

```js
// es6 方法不可枚举
class Point {
  constructor(x, y) {
    // ...
  }

  toString() {
    // ...
  }
}

Object.keys(Point.prototype) // []
Object.getOwnPropertyNames(Point.prototype) // ["constructor", "toString"]

// es5
var Point = function (x, y) {
  // ...
};

Point.prototype.toString = function() {
  // ...
};

Object.keys(Point.prototype)
// ["toString"]
Object.getOwnPropertyNames(Point.prototype) // ["constructor", "toString"]
```

## class 表达式

和函数一样，class 也可以用表达式定义

```js
// 注意这个类的名字是 MyClass 不是 Me，Me 只在 class 内部可用，用来指代当前类
const MyClass = class Me {
  getClassName() {
    return Me.name
  }
}

let inst = new MyClass()
inst.getClassName() // "Me"
Me.name // ReferenceError: Me is not defined

// 如果类内部没有用到，可以省略类名
const MyClass = class {
  ...
}
```

## 不存在变量提升

**类不存在变量提升**，这和 es5 的函数声明变量提升不同。这是因为 es6 中必须保证子类在父类之后定义

```js
// 类不存在变量声明
new Foo()
class Foo {} // ReferenceError: Foo is not defined

// 子类在继承父类时父类已经有定义。但如果存在 class 提示提升，代码会报错。因为 class 被提升到代码头部，而 let 是不会提升的，导致子类继承父类时父类还没有定义
let Foo = class {}
class Bar extends Foo {}
```

## 私有方法和属性

es5 中在构造函数中定义的方法、属性因为函数作用域会会成为私有方法、属性，在构造函数外无法访问到。es6 中类定义的方法都是挂载在原型上的，也不提供私有方法和属性。

私有方法**可利用 Symbol 值的唯一性，将私有方法的名字命名为一个 Symbol 值变通实现**

```js
const bar = Symbol('bar');
const snaf = Symbol('snaf');

export default class myClass{

  // 公有方法
  foo(baz) {
    this[bar](baz);
  }

  // 私有方法
  [bar](baz) {
    return this[snaf] = baz;
  }

  // ...
};
```

私有属性有一个提案，是在属性名前加 #, 当然很自然也可以用来写私有方法

```js
class Foo {
  #a = 0;
  #b;
  #sum() { return #a + #b; }
  printSum() { console.log(#sum()); }
  constructor(a, b) { #a = a; #b = b; }
}
```

## name 属性

name 属性总是返回紧跟在 class 关键字后面的类名

```js
class Foo {}
Foo.name // "Foo"
```

## class 取值函数 getter 和存值函数 setter

和 es5 一样，类内部可以使用 set 和 get 关键字，对某个属性设置取值函数和存值函数，拦截该属性的存取行为

```js
class Foo {
  constructor() {}
  get prop() {
    return 'getter'
  }
  set prop(value) {
    console.log('setter: ' + value)
  }
}

let inst = new Foo()
inst.prop = 1 // setter: 1
inst.prop // getter
```
