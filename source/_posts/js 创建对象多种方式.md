---
title: js 创建对象的多种方式
tag: 
	- js
---

> 参考: [javascript 高级程序设计第三版]()

<!-- markdownlint-disable MD010 -->

## 工厂模式

```js
function createPerson(name) {
  var obj = new Object()
  obj.name = name
  obj.getName = function() {
    console.log(this.name)
  }
  return obj
}

var person1 = createPerson('jack')
```

- 缺点：对象无法识别，所有实例都指向同一个原型

<!-- more -->

## 构造函数模式

```js
function Person(name) {
  this.name = name
  this.getName = function() {
    console.log(this.name)
  }
}
var person1 = new Person('jack')
```

- 优点：实例可以识别为一个特定类型
- 缺点：每次创建实例时候，每个方法都会被创建一次

## 构造函数模式优化

```js
function Person(name) {
  this.name = name
  this.getName = getName
}

function getName() {
  console.log(this.name)
}

var person1 = new Person('jack')
```

- 优点：解决每个方法都会被重新创建的问题
- 缺点：没有很好的封装性

## 原型模式

```js
function Person(name) {
}

Person.prototype.name = 'jack'
Person.prototype.getName() {
  console.log(this.name)
}

var person1 = new Person('jack')
```

- 优点：方法不会重新创建
- 缺点：所有属性和方法都给共享；不能初始化参数

## 原型模式优化

```js
function Person(name) {
}

Person.prototype = {
  name: 'jack',
  getName: function() {
    console.log(this.name)
  }
}

var person1 = new Person('jack')
```

- 缺点：重新了原型，丢失了 constructor 属性

## 原型模式优化

```js
function Person(name) {
}

Person.prototype = {
  constructor: Person,
  name: 'jack',
  getName: function() {
    console.log(this.name)
  }
}

var person1 = new Person('jack')
```

- 优点: 实例可以通过 constructor 属性找到所属构造函数
- 缺点: 还是存在原型模式的缺点

## 组合模式（构造函数模式和原型模式结合）

```js
function Person(name) {
  this.name = name
}

Person.prototype = {
  constructor: Person,
  getName: function() {
    console.log(this.name)
  }
}

var person1 = new Person('jack')
```

- 优点：该共享的共享，该私有的私有

## 寄生构造函数模式

```js
function Person(name) {
  var obj = new Object()
  obj.name = name
  obj.getName = function () {
    console.log(this.name)
  }

  return obj
}

var person1 = new Person('jack');
console.log(person1 instanceof Person) // false
console.log(person1 instanceof Object)  // true
```

- 缺点：对象无法识别，所有实例都指向同一个原型，本质还是工厂模式