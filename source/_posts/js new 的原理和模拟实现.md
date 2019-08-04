---
title: js new 的原理和模拟实现
tag: 
	- js
---

> 参考: JavaScript 高级程序设计（第 3 版）

<!-- markdownlint-disable MD010 -->

## new 一个对象的原理

```js
function Person(name, age, job) {
	this.name = name
	this.age = age
	this.job = job
}

var p1 = new Person('jack', 20, 'Teacher')
```

<!-- more -->

通过构造函数创建实例必须使用 new 操作符。通过这种方式调用构造函数实际经过一下 4 个步骤：

- **创建一新个对象**
- **将构造函数的作用域赋给新对象（对象能访问构造函数原型的属性方法）**
- **执行构造函数的代码（this 就指向该对象, 为这个新对象添加属性）**
- **返回新对象**

对于构造函数一般没有 return 语句，返回的就是上面构造的新对象；如果有返回值并且返回值是对象，则 this 指向该对象，构造函数返回该对象；如果有返回值并且返回值不是对象而是一般值，则 this 指向函数实例，构造函数返回上面构造的新对象

```js
// 构造函数返回对象, this 指向该对象，返回该对象
function Person(name, age, job) {
	this.name = name
	this.age = age
	this.job = job
	return {
		name: 'test',
		age: 0,
		job: 'job'
	}
}
new Person('jack', 20, 'Teacher') // {name: "test", age: 0, job: "job"}

// 构造函数返回一般值, this 内部创建的对象并隐式返回
function Person(name, age, job) {
	this.name = name
	this.age = age
	this.job = job
	return 2
}
new Person('jack', 20, 'Teacher') // Person {name: "jack", age: 20, job: "Teacher"}
```

## new 模拟实现

因为 new 操作符是保留字，无法覆盖实现，使用一个函数进行模拟，第一个参数是传入的构造函数，其他参数值传入构造函数的参数

```js
function createObj() {
	// 1 新创建一个对象
	var obj = {}
	// 去除第一个参数为构造函数 之后 arguments 只包含需要构造函数的参数
	var Constructor = Array.prototype.shift.call(arguments)
	// 2 将构造函数作用域赋给该对象，对象能访问构造函数原型的属性方法（[[prototype]] 连接）
	obj.__proto__ = Constructor.prototype
	// 3 this 指向函数调用的对象，执行构造函数代码，将属性和方法添加给该对象
	// 这里执行构造函数，可能构造函数有返回值
	var ret = Constructor.apply(obj, arguments)
	// 4 如果构造函数返回值是对象就返回该值，否则返回上面创建的对象
	return ret instanceof Object ? ret : obj
}
```

之前代码测试一下

```js
// 构造函数没有返回值
function Person(name, age, job) {
	this.name = name
	this.age = age
	this.job = job
}
createObj(Person, 'jack', 20, 'Teacher') // Person {name: "jack", age: 20, job: "Teacher"}

// 构造函数有返回值并且为对象
function Person(name, age, job) {
	this.name = name
	this.age = age
	this.job = job
	return {
		name: 'test',
		age: 0,
		job: 'job'
	}
}
createObj(Person, 'jack', 20, 'Teacher') // {name: "test", age: 0, job: "job"}

// 构造函数有返回值并且不为对象
function Person(name, age, job) {
	this.name = name
	this.age = age
	this.job = job
	return 2
}
createObj(Person, 'jack', 20, 'Teacher') // Person {name: "jack", age: 20, job: "Teacher"}
```