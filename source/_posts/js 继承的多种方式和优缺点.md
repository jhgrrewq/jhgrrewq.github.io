---
title: js 继承的多种方式和优缺点
tag: 
	- js
---

> 参考: [javascript 高级程序设计第三版]()、[JavaScript深入之继承的多种方式和优缺点](https://github.com/mqyqingfeng/Blog/issues/16)

<!-- markdownlint-disable MD010 -->

## 原型链继承

```js
function Parent() {
  this.name = 'jack'
}
Person.prototype.getName = function() {
  console.log(this.name)
}
function Child() {

}
Child.prototype = new Parent()

var child1 = new Child()
console.log(child1.getName()) // jack
```

<!-- more -->

缺点

- 引用类型的属性会被所有实例共享

```js
function Parent() {
  this.names = ['jack', 'kevin'] 
}
function Child() {}
Child.prototype = new Parent()

var child1 = new Child()
child1.names.push('jeff')
console.log(child1.names) // ['jack', 'kevin', 'jeff']
var child2 = new Child()
console.log(child2.names) // ['jack', 'kevin', 'jeff']
```

- 创建 Child 实例时候，不能向 Parent 传参

## 借用构造函数（经典继承）

```js
function Parent() {
  this.names = ['jack', 'kevin']
}
function Child() {
  Parent.call(this)
}

var child1 = new Child();
child1.names.push('jeff');
console.log(child1.names); // ['jack', 'kevin', 'jeff']
var child2 = new Child();
console.log(child2.names); // ['jack', 'kevin']
```

优点

- 避免引用类型的属性被所有实例共享
- 可在 Child 中向 Parent 传参

```js
function Parent (name) {
  this.name = name;
}

function Child (name) {
  Parent.call(this, name);
}

var child1 = new Child('kevin');
console.log(child1.name); // kevin
var child2 = new Child('jack');
console.log(child2.name); // jack
```

缺点

- 方法都在构造函数中定义，每次创建实例都会创建一遍方法

## 组合继承（原型链继承和经典继承结合）

```js
function Parent(name) {
  this.name = name
  this.colors = ['red', 'yellow']
}
Parent.prototype.getName = function() {
  console.log(this.name)
}

function Child(name, age) {
  Parent.call(this, name)
  this.age = age
}
Child.prototype = new Parent()
Child.prototype.constructor = Child

var child1 = new Child('jack', 20)
child1.colors.push('blue')
console.log(child1.name) // jack
console.log(child1.colors) // ['red', 'yellow', 'blue']
console.log(child1.age) // 20
var child1 = new Child('kevin', 18)
console.log(child1.name) // kevin
console.log(child1.colors) // ['red', 'yellow']
console.log(child1.age) // 18
```

优点

- 融合原型链继承和构造函数的优点，是 javascript 中最常见的继承模式

## 原型式继承

es5 Object.create 的模拟实现，将传入的对象作为创建对象的原型

```js
function createObj(o) {
  function F() {}
  F.prototype = o
  return new F()
}
```

缺点

- 包含引用类型的属性值都会共享，和原型链继承一样

```js
var person = {
  name: 'jack',
  friends: ['kevin']
}
var person1 = createObj(person)
var person2 = createObj(person)

person1.name = 'person1'
console.log(person2.name) // jack
person1.friends.push('jeff')
console.log(person2.friends) // ['kevin', 'jeff']
```

## 寄生式继承

创建一个仅用于封装继承过程的函数，该函数在内部以某种方式在做增强对象，最后返回对象

```js
function createObj(o) {
  var clone = Object.create(o)
  clone.sayName = function () {
    console.log('hi')
  }
  return clone
}
```

缺点

- 和借用构造函数一样，每次创建对象都会创建一遍方法

## 寄生组合继承

组合继承**最大缺点是会调用两次父构造函数**

```js
function Parent (name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}
Parent.prototype.getName = function () {
  console.log(this.name)
}
function Child (name, age) {
  Parent.call(this, name) // 创建实例实例上第二次调用父构造函数是执行这句代码
  this.age = age
}
Child.prototype = new Parent() // 设置子类实例原型第一次调用父构造函数
var child1 = new Child('kevin', '18') // 创建实例是会第二次调用父构造函数
console.log(child1)
```

使用中间函数进行桥接

```js
function Parent (name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}
Parent.prototype.getName = function () {
  console.log(this.name)
}
function Child (name, age) {
  Parent.call(this, name)
  this.age = age
}

// 中间函数进行桥接
function F() {}
F.prototype = Parent.prototype
Child.prototype = new F()

var child1 = new Child('kevin', '18')
console.log(child1)
```

进一步可以对上述继承方法进行封装

```js
function createObj(o) {
  function F() {}
  F.prototype = o
  return new F()
}

function inhert(child, parent) {
  var proto = createObj(parent.prototype)
  pro.constructor = child
  child.prototype = proto
}

// 使用时
inhert(Child, Parent)
```

寄生组合式继承优点

- 只调用一次 Parent 构造函数，避免在 Parent.prototype 上创建多余的属性
- 保证原型链不变
- 能正常使用 instanceof 和 isPrototypeOf
