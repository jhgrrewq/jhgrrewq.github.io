---
title: es6 class 继承
tag:
	- es6
---

> 参考: [ECMAScript 6 入门 class 继承部分](http://es6.ruanyifeng.com/#docs/class-extends)

<!-- markdownlint-disable MD010 -->

## 概述

es5 原型继承是采用 **父类.call(this,props) 继承父类的属性方法，并通过一个中间构造函数修复原型链**

<!-- more -->

```js
function Parent(name) {
  this.name = name
}
Parent.prototype.print = function() {
  console.log(this.name)
}

function Child(name, gender) {
  // 对象使用属性和方法会先从自身查找，没有再从原型链上找。如果没有下面这一句，所有对象实例都会共享父类的引用类型的属性（如数组、对象）
  Parent.call(this, name)
  this.gender = gender
}

// 利用中间构造函数修复原型链
// new F() 的 __proto__ 指向 F.prototype（Parent.prototype）
// 原本 Child.prototype.constructor === Child, 现在 Child.prototype 重写了代以 F 的实例对象，因此原来 Parent 实例对象的属性方法都存在 Child.prototype 中
function F() {}
F.prototype = Parent.prototype
Child.prototype = new F()
Child.prototype.constructor = Child

// 因为 Child.prototype 进行了重写，因此想要挂载子类原型的属性和方法需要放在使用中间构造函数修复原型链之后
Child.prototype.printGender = function() {
  console.log(this.gender)
}
```

可以将上述中间构造函数封装成一个函数进行调用或者使用 Object.create() 代替

```js
// 将上述中间构造函数封装成一个函数进行调用
function inherits(Child, Parent) {
  var F = function() {}
  F.prototype = Parent.prototype
  Child.prototype = new F()
  Child.prototype.constructor = Child
}
inherits(Child, Parent)

// 使用 Object.create()
Child.prototype = Object.create(Parent.prototype)
// 内部实现是
Object.create = function(o) {
  var F = function() {} // 定义一个隐形构造函数
  F.protytype = o
  return new F() // 返回一个新对象
}
```

es5 继承是**先创建子类实例对象 this，再将父类的方法属性添加到 this 上（Parent.call(this, props)）**

es6 继承是**先创造父类的实例对象 this(因此要先调用 super 方法),再通过子类构造函数修改 this**

- **通过 extends 关键字继承父类**
- **super 关键字表示父类构造函数，用来新建父类的 this 对象**
- **子类必须在 constructor 中调用 super 方法**，否则创建实例报错（因为子类没有自己的 this 对象，而是继承父类的 this 并进行加工，如果不调用 super 方法，子类拿不到 this 对象）
- **子类必须在调用 super 方法才能使用 this**。 子类实例是基于父类实例（this）的加工, 只有调用 super 方法才能返回父类实例（this）

```js
class Parent{
  constructor(name) {
    this.name = name
  }
  print() {
    console.log(this.name)
  }
}
class Child extends Parent{
  constructor(name, gender) {
    super(name) // 相当于调用父类 constructor(name)
    this.gender = gender
  }
  printGender() {
    console.log(this.gender)
  }
}
```

```js
// 因为 constructor 属性本身是默认存在的，而 es6 规定子类 constructor 中必须先调用 super 方法获得父类实例 this
class Child extends Parent{}
// 等同于
class Child extends Parent{
  constructor(...arg) {
    super(...arg)
  }
}

class Parent{}
class Child extends Parent{
  constructor(...arg) {
    // 子类构造函数中必须先调用 super 方法才能使用 this
    console.log(this) // Uncaught ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor
    super(...arg)
    console.log(this)
  }
}
```

**es6 父类的静态方法也会被子类继承**, 这与 es5 不同，因为 es5 使用构造函数和原型继承时，父类的静态方法属性是直接挂在父类构造函数上，子类构造函数上没有定义

```js
class A {
  static print() {
    console.log('a')
  }
}
class B extends A{}
B.print() // 'a'
```

## Object.getPrototypeOf()

Object.getPrototypeOf() 可以从子类中获取父类，该方法可以用来判断一个类是否继承了另一个类

```js
Object.getPrototypeOf(Child) === Parent // true
```

比较一下判断 instanceof 和 typeof

```js
let num = 1
let obj = {}
let arr = []
let fn = function() {}
class Parent{}
class Child extends Parent{}
let c = new Child()

// typeof 用来判断基本类型数据，无法判断引用类型数据
typeof num // 'number'
typeof obj // 'object'
typeof num // 'object'
typeof fn  // 'object'

// instanceof 用来判断引用类型数据实例
obj instanceof Object // true
arr instanceof Array // true
fn instanceof Function // true
// 对象的隐式原型 __proto__ 指向创建它的构造函数的显式原型 prototype
// instanceof 判断对象是否是某个构造函数的实例，是从对象的隐式原型 __proto__ 沿着原型链往上找，同时沿着构造函数的原型 prototype 往下找，看最终是能否找到同一个引用
c instanceof Child // true
c instanceof Parent // true

// 通用的判断引用类型数据的方法 利用 Object.prototype.toString 方法
Object.prototype.toString.call(obj) // "[object Object]"
Object.prototype.toString.call(arr) // "[object Array]"
Object.prototype.toString.call(fn) // "[object Function]"
Object.prototype.toString.call(Child) // "[object Function]" 构造函数就是函数
```

## super 关键字 （可当做函数使用，也可当做对象使用）

### super 当做函数使用

super 当做函数使用，代表父类的构造函数。es6 规定，**子类的构造函数必须执行一次 super 方法，并且 super 方法只能在子类构造函数中调用，在其他地方调用会报错**

super 虽然代表了父类的构造函数，但是返回的是子类的实例，也就是说 **super 内部的 this 指向的是子类**

```js
class Parent{
  constructor() {
    console.log(this)
  }
}
class Child extends Parent{
  constructor() {
    // 调用后 super 内部 this 指向子类 Child
    // 相当于调用 Parent.prototype.constructor.call(this)
    super()
  }
  m() {
    // super 方法只能在子类构造函数中调用
    super() // Uncaught SyntaxError: 'super' keyword unexpected here
  }
}
```

### super 作为对象使用

super 作为对象时，**在普通方法中，指向父类的原型对象 （定义在父类实例上的方法和属性，是无法通过 super 调用的，因为这是 super 指向父类的原型对象，父类原型对象上是没有父类实例上的方法和属性的），在静态方法中，指向父类**

```js
class Parent{
  constructor(name) {
    this.name = name // 定义在父类实例上的属性方法
  }
  print() {
    console.log('print')
  }
}
// 定义在父类原型上的属性方法
Parent.prototype.test = 'test'

class Child extends Parent{
  constructor(name) {
    super(name)
    // super.print() 处于普通方法中，这里 super 当对象使用，指向 Parent.prototype, super.print() 相当于 Parent.prototype.print()
    console.log(super.print()) // 'print'
  }
  p() {
    // 定义在父类实例上的属性方法通过 super 调用无法拿到
    console.log(super.name) // undefined
    console.log(super.test) // 'test'
  }
}

let c = new Child('jack')
c.p()
```

es6 规定，**通过 super 调用父类的方法时，方法内部的 this 指向子类**

```js
class Parent{
  constructor() {
    this.x = 1
  }
  print() {
    console.log(this.x)
  }
}

class Child extends Parent{
  constructor() {
    super()
    this.x = 2
  }
  p() {
    // super.print() 调用等同于 Parent.prototype.print() ，内部的 this 指向子类 Child，因此输出 this.x 是 2
    super.print()
  }
  newP() {
    // 因为 this 指向子类，通过 super 对某个属性赋值，这时候 super 是 this
    super.x = 3 // 等同 this.x = 3
    console.log(super.x) // super.x 等同 A.prototype.x 原型上当然没有，因此是 undefined
    console.log(this.x)
  }
}

let c = new Child('jack')
c.p()
c.newP()
```

**super 作为对象使用在静态方法中，指向父类，而不是父类原型**

```js
class Parent{
  print(msg) {
    console.log('prototype', msg)
  }
  static print(msg) {
    console.log('static', msg)
  }
}

class Child extends Parent{
  print(msg) {
    super.print(msg)
  }
  static print(msg) {
    super.print(msg)
  }
}

Child.print(1); // static 1

var c = new Child();
c.print(2); // prototype 2
```

使用 super 必须**显示指定是作为函数还是对象使用，否则报错**

```js
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super); // Uncaught SyntaxError: 'super' keyword unexpected here
  }
}
```

## 实例的 隐式原型 属性

子类实例的 隐式原型 属性的 隐式原型 属性，指向父类实例的 隐式原型 属性

```js
var p = new Parent()
var c = new Child()
c.__proto__.__proto__ === p.__proto__ // true

// 等同于
Child.prototype.__proto__ === Parent.prototype // true
```

```js
// 考虑从 es5 利用中间构造函数来看
function F() {}
F.prototype = Parent.prototype
Child.prototype = new F()

// 对象的 __proto__ 指向创建它的构造函数的原型 prototype
// 子类实例的 __proto__ 指向 Child.prototype, 也就是 new F()
// new F() 的 __proto__ 指向 F.prototype，也就是Parent.prototype
// Child.prototype 的 __proto__ 指向 Parent.prototype
// 子类的原型的隐式原型是父类的原型
```

子类的原型的隐式原型是父类的原型，当然也就可以通过子类实例的 隐式原型 的 隐式原型 属性给父类原型添加属性和方法

```js
c.__proto__.__proto__.print = function() {
  console.log('test')
}
p.print() // 'test'
```

## 类的 隐式原型 属性

es5 中，每一个对象都有 隐式原型 属性，指向对应构造函数的 prototype 属性。

es6 中 **子类的 隐式原型 属性，总是指向父类**

```js
Child.__proto__ === Parent // true
```

类的继承是通过 Object.setPropertypeOf(Child.prototype, Parent.prototype) 方法实现的

在 es5 继承中，**父类的静态方法和属性（只能父类调用，父类实例无法调用）不能被子类继承**, 但是 es6 中 **子类能继承父类的静态属性和方法**

```js
// Object.setPrototypeOf() 方法的实现
Object.setPrototypeOf = function(obj, proto) {
  obj._proto__ = proto
  return obj
}
```

```js
class A {}
class B {}

// B 的实例继承 A 的实例
Object.setPrototypeOf(B.prototype, A.prototype)
// 等同于
// B 的原型也是对象，它的隐式原型指向 A 的原型，这样 B 的原型能拿到 A 原型上的属性和方法，B 的实例具有了 A 原型上的属性和方法
B.prototype.__proto__ = A.prototype // 继承实现

// B 继承 A 的静态属性
Object.setPrototypeOf(B, A)
// 等同于
// B 作为构造函数本身也是对象，本来函数（对象）的 __proto__ 指向 Funtion.prototype(本身也是一个对象), 因此 B 能拿到 Function 原型上的属性和方法，现在重写指向 A，B 能拿到 A 上的属性和方法
B.__proto__ = A

const b = new B()
```

## 原生构造函数继承

es5 中的原生构造函数 Object Array Function Date Error RegExp Boolean Number String 一般用来生成数据结构，或者作为转换函数。这些构造函数是不能继承的，因此之前不能通过继承原生构造函数定义自己的类

```js
// es5 通过继承原生构造函数自定义子类
function MyArray() {
  Array.call(this,arguments)
}

MyArray.prototype = Object.create(Array.prototype, {
  constructor: {
    value: MyArray,
    writable: true,
    configurable: true,
    enumerable: false
  }
})

var arr = new MyArray()
arr[0] = 'a'
arr.length // 0 无效
```

这是因为子类无法获得原生构造函数内部的属性，原生构造函数会忽略 call 方法传入的 this, 原生构造函数无法绑定 this，因此拿不到内部属性

**es5 的继承实质是先创建子类实例 this，再通过 父类.call(this, props) 将父类属性添加到子类实例上**，由于父类的内部属性无法获取，导致无法继承原生的构造函数

**es6 允许继承原生构造构造函数继承子类，是因为 es6 继承是先通过在子类构造函数中 调用 super(...) 新建父类实例 this，然后再在子类构造函数中修改 this**

```js
class MyArray extends Array {
  constructor(...arg) {
    super(...arg)
  }
}

let arr = new MyArray()
arr[0] = 'a'
arr.length // 1
```

对于 es5 继承原生构造函数，可以通过内部先调用原生构造函数生成对象，再修改对象的原型链

```js
Object.setPrototypeOf = Object.setPrototypeOf || function(obj, proto) {
  obj.__proto__ = proto
  return obj
}

function MyArray() {
  // var arr = new Array()

  // 考虑到需要传递 MyArray 构造函数一些参数
  var bindArr = Array.bind(Array, Array.prototype.slice.call(arguments))
  var arr = new bindArr()

  // 对象原型链有原来指向 Array.prototype 改为 指向 MyArray.prototype
  Object.setPrototypeOf(arr, MyArray.prototype)

  arr.test = 'test'

  return arr
}

// 自定义类的原型链修改
MyArray.prototype = Object.create(Array.prototype, {
  constructor: {
    value: MyArray,
    writable: true,
    configurable: true,
    enumerable: false
  }
})

MyArray.prototype.getLength = function() {
  console.log(this.length)
}

var arr = new MyArray()
arr[0] = 'a'
arr.length // 1
arr.getLength() // 1
```