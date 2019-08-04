---
title: es6 class 经 babel 编译分析
tag:
	- es6
---

<!-- markdownlint-disable MD010 -->

> 参考: [揭秘babel的魔法之class魔法处理](https://www.jianshu.com/p/d36fb31f9cff)、[揭秘babel的魔法之class继承的处理2](https://www.jianshu.com/p/95901615f322)

## class

```js
class Person {
    constructor(name) {
        this.name = name;
        this.type="person"
    }
    hello() {
        console.log('hello ' + this.name);
    }
    static fn() {
        console.log('static');
    };
}
```
<!-- more -->

class 声明经过 babel 编译后其实是**定义一个立即执行函数，返回定义好的构造函数**。重点在于理解立即执行函数内部调用的 _classCallCheck() 和 _createClass() 方法

```js
'use strict';

var _createClass = function () {
  function defineProperties(target, props) {
    for (var i = 0; i < props.length; i++) {
      var descriptor = props[i]; 
      descriptor.enumerable = descriptor.enumerable || false;
      descriptor.configurable = true;
      if ("value" in descriptor) 
        descriptor.writable = true;
      Object.defineProperty(target, descriptor.key, descriptor);
    } 
  } 
  return function (Constructor, protoProps, staticProps) { 
    if (protoProps)
      defineProperties(Constructor.prototype, protoProps); 
    if (staticProps)
      defineProperties(Constructor, staticProps); 
    return Constructor;
  }; 
}();

function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

var Person = function () {
  function Person(name) {
    _classCallCheck(this, Person);

    this.name = name;
    this.type = "person";
  }

  _createClass(Person, [{
      key: 'hello',
      value: function hello() {
        console.log('hello ' + this.name);
      }
    }], [{
      key: 'fn',
      value: function fn() {
        console.log('static');
      }
    }]
  );

  return Person;
}();
```

### _classCallCheck()

```js
function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}
```

这个函数确保 class 不能作为一般的函数调用，因为只有 new 构造函数，this 才会指向创建的对象实例

```js
function _classCallCheck(instance, Constructor) {
  console.log(instance, Constructor)
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

function Person() {
  _classCallCheck(this, Person)
}

// 构造函数作为一般函数调用报错
Person() // Uncaught TypeError: Cannot call a class as a function 此时 this 指向 window
new Person() // this 指向创建的对象实例 Person {}
```

### _createClass()

该方法定义一个立即执行函数，**返回一个方法，可以对构造函数本身和其原型扩展属性和方法（内部实质调用 Object.defineProperty()）**

```js
var _createClass = function () {
  function defineProperties(target, props) {
    // 传入的 props 是一个数组，数组每个元素就是一个包含属性描述符的对象
    // 对于每个 props[i]，都要完全拷贝它的 descriptor, 并扩展到target上
    // descriptor 的 key 为属性名, value 为属性值
    for (var i = 0; i < props.length; i++) {
      var descriptor = props[i];
      descriptor.enumerable = descriptor.enumerable || false;
      descriptor.configurable = true;
      if ("value" in descriptor)
        descriptor.writable = true;
      Object.defineProperty(target, descriptor.key, descriptor);
    }
  }
  return function (Constructor, protoProps, staticProps) { 
    // 参数 Constructor 是要扩展的目标对象，其实就是传入的构造函数
    // 参数 protoProps 是在目标对象原型链上添加的属性，是一个数组，数组每个元素是一个包含属性描述符的对象 {value， key} key 值是属性名，value 值是属性值
    // 参数 staticProps 是在目标对象上添加的属性，是一个数组，数组每个元素是一个包含属性描述符的对象 {value， key} key 值是属性名，value 值是属性值
    if (protoProps)
      defineProperties(Constructor.prototype, protoProps); 
    if (staticProps)
      defineProperties(Constructor, staticProps); 
    return Constructor;
  }; 
}();
```

## class 继承

```js
class Person {
  constructor(name) {
    this.name = name;
    this.type="person"
  }
  hello() {
    console.log('hello ' + this.name);
  }
  static fn() {
    console.log('static');
  };
}

class Student extends Person{
  constructor(name, age) {
    super(name)
    this.age = age
  }
}
```

class 继承经过 babel 编译后其实子类构造函数是**定义一个立即执行函数，传入父类构造函数，返回定义好的构造函数**。重点在于理解立即执行函数内部调用的 _inherits() 和 __possibleConstructorReturn() 方法

```js
'use strict';

var _createClass = function () {
  function defineProperties(target, props) {
    for (var i = 0; i < props.length; i++) {
      var descriptor = props[i]; 
      descriptor.enumerable = descriptor.enumerable || false;
      descriptor.configurable = true;
      if ("value" in descriptor) 
        descriptor.writable = true;
      Object.defineProperty(target, descriptor.key, descriptor);
    } 
  } 
  return function (Constructor, protoProps, staticProps) { 
    if (protoProps)
      defineProperties(Constructor.prototype, protoProps); 
    if (staticProps)
      defineProperties(Constructor, staticProps); 
    return Constructor;
  }; 
}();

function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

function _possibleConstructorReturn(self, call) {
  if (!self) {
    throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
  } 
  return call && (typeof call === "object" || typeof call === "function") ? call : self; 
}

function _inherits(subClass, superClass) {
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
  }
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: {
      value: subClass,
      enumerable: false,
      writable: true,
      configurable: true
    }
  }); 
  if (superClass) 
    Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}

var Person = function () {
  function Person(name) {
    _classCallCheck(this, Person);

    this.name = name;
    this.type = "person";
  }

  _createClass(Person, [{
      key: 'hello',
      value: function hello() {
        console.log('hello ' + this.name);
      }
    }], [{
      key: 'fn',
      value: function fn() {
        console.log('static');
      }
    }]
  );

  return Person;
}();

// 实现 Student 构造函数，定义一个自执行函数，传入 Person 构造函数
var Student = function (_Person) {
  // 返回该函数作为 Student 构造函数
  function Student(name, age) {
    // 检测
    _classCallCheck(this, Student);

    var _this = _possibleConstructorReturn(this, (Student.__proto__ || Object.getPrototypeOf(Student)).call(this, name));

    _this.age = age;
    return _this;
  }

  // 实现对父类原型链属性继承
  _inherits(Student, _Person);

  return Student;
}(Person);
```

### _inherits()

```js
function _inherits(subClass, superClass) {
  // superClass 必须为函数，否则报错
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
  }
  // Object.create 第二个参数是为了修复子类的 constructor
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: {
      value: subClass,
      enumerable: false,
      writable: true,
      configurable: true
    }
  });
  // Object.setPrototypeOf 让子类能继承父类的静态属性和方法
  if (superClass)
    Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}
```

该方法实现了对父类原型链属性的继承, **实质是调用 Object.create() 方法实现对父类的原型继承，同时调用 Object.setPrototypeOf() 方法实现对父类静态属性方法继承**

```js
Student.prototype = Object.create(Person.prototype)
Object.setPrototypeOf(Student, Person)
```

### _possibleConstructorReturn()

```js
// 第一个参数是子类实例 this，第二个参数是调用 父类.call(this.props) 返回值（构造函数一般没有返回，返回 undefined, 构造函数有返回值如果是对象，则返回该对象，如果有返回值并且是非对象，则隐式返回 this，也就是该函数实例）
// 如果第二个参数是对象或者函数，就返回第二个参数，否则返回第一个参数
function _possibleConstructorReturn(self, call) {
  if (!self) {
    throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
  }
  return call && (typeof call === "object" || typeof call === "function") ? call : self;
}
```

该方法传入的第二个参数**本质是调用 父类.call(this, props)**

```js
var Student = (function(_Person) {
  function Student(name, age) {
    ...
    // es5 继承就是先创建子类实例，再将父类属性添加到子类上

    // 首先创建子类实例 this
    // Person.call(this, name) 将父类属性添加到子类实例 this
    // _this 指向修改后的子类实例 this
    // _this 实例继续添加属性
    var _this = _possibleConstructorReturn(this, (Student.__proto__ || Object.getPrototypeOf(Student)).call(this, name));

    _this.age = age;
    return _this;

    // (Student.__proto__ || Object.getPrototypeOf(Student)).call(this, name)
    // this.age = age
  }

  _inherits(Student, _Person);

  return Student
})(Person)
```
