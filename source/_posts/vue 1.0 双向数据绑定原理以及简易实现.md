---
title: vue 1.0 双向数据绑定原理以及实现
tag: 
	- vue
	- 数据绑定
---

<!-- markdownlint-disable MD010 -->

> 参考: [面试官系列(4): 实现双向绑定Proxy比defineproperty优劣如何?](https://juejin.im/post/5acd0c8a6fb9a028da7cdfaf)

## 动态数据绑定练习

### 动态数据绑定一

<img width="500" height="500" src="http://ony85apla.bkt.clouddn.com/17-11-30/38125022.jpg">

<!--more-->

#### 代码

~~~javascript
function Observer(data){
    this.data = data;
    this.walk(data);
}
Observer.prototype = {
    walk: function(data){
        for(let key in data){
            // for in 会把对象的原型链上所有可枚举属性列出来，而我们只想要遍历对象本身的属性
            if(data.hasOwnProperty(key)) {
                // 如果没有遍历到最底层，继续new Observer
                if(data[key] instanceof Object){
                    new Observer(data[key])
                }
                this.defineReactive(data, key, data[key]);
            }
        }
    },
    defineReactive: function(data, key, val){
        Object.defineProperty(data, key, {
            enumberable: true,
            configurable: true,
            get: function(){
                console.log('你访问了 ' + key);
                return val
            },
            set: function(newVal){
                if(newVal === val){
                    return
                }
                val = newVal;
                // 同样 设置的值为object 需要new Observer将数据进行劫持
                if(newVal instanceof Object){
                    new Observer(val)
                }

                console.log('你设置了 '+ key + ', 新的值' + newVal);

            }
        })
    }
}

let app1 = new Observer({ name: 'youngwind', age: 110});
let app2 = new Observer({ university: 'bupt', major: 'computer'});
app1.data.name;
app1.data.age = 100;
app2.data.university;
app2.data.major = 'science';
~~~

#### 知识点

**Object.defineProperty**

Vue.js 是通过它实现双向绑定的。通过这个方法，直接在一个对象上定义一个新的属性，或者是修改已有的属性。最终这个方法会返回该对象。它接受3个参数，而且都是必填的: `object` 目标对象、`propertyname` 需要定义的属性或方法的名字、`descriptor` 属性描述符

### 动态数据绑定二

![](http://ony85apla.bkt.clouddn.com/17-11-30/33729754.jpg)

#### 代码

~~~javascript
function Observer(data){
    this.data = data;
    this.walk(data);
    // 每次new一个Observer实例就new一个Event实例来管理Observer实例中的所有事件，通过$watch来为Observer实例注册监听事件，每当属性改变触发事件队列
    this.eventBus = new Event();
}

Observer.prototype = {
    walk: function(data){
        for(let key in data){
            // for in 会把对象的原型链上所有可枚举属性列出来，而我们只想要遍历对象本身的属性
            if(data.hasOwnProperty(key)) {
                // 如果没有遍历到最底层，继续new Observer
                if(data[key] instanceof Object){
                    new Observer(data[key])
                }
                this.defineReactive(data, key, data[key]);
            }
        }
    },
    defineReactive: function(data, key, val){
        let that = this
        Object.defineProperty(data, key, {
            enumberable: true,
            configurable: true,
            get: function(){
                console.log('你访问了 ' + key);
                return val
            },
            set: function(newVal){
                if(newVal === val){
                    return
                }
                val = newVal;
                // 同样 设置的值为object 需要new Observer将数据进行劫持
                if(newVal instanceof Object){
                    new Observer(val)
                }
                // 属性改变触发事件队列
                that.eventBus.emit(key, newVal);
                console.log('你设置了 '+ key + ', 新的值' + newVal);
            }
        })
    },
    $watch: function(key, callback){
        this.eventBus.on(key, callback);
    }
}

function Event(){
    this.events = {}; // 缓存列表
}

Event.prototype = {
    on: function(key, callback){
        if(!this.events[key]){
           this.events[key] = [callback];
         }
        this.events[key].push(callback)
    },
    off: function(key){
        for(let attr in this.events){
            if(this.events.hasOwnProperty(key) && key === attr){
                delete this.events[key];
            }
        }
    },
    emit: function(){
        let key = arguments[0];
        let rest = Array.prototype.slice.call(arguments,1);
        this.events[key] && this.events[key].forEach(function(fn){
            fn.apply(null,rest);
        })
    },
    // es6
    emit: function(key, …arg){
        this.events[key] && this.events[key].forEach(function(fn){
            fn(…arg)
        })
    }
}

let app1 = new Observer({ name: 'youngwind', age: 110});
let app2 = new Observer({ university: 'bupt', major: 'computer'});
app1.data.name;
app1.data.age = 100;
app2.data.university;
app2.data.major = 'science';
app1.$watch('age', function(age){
    console.log('我的年龄变了，现在是：'+age+'岁了’);
});
app1.data.age = 10;
~~~

#### 知识点

- 使用观察者设计模式，初始化一个缓存对象
- 调用 `Object.defineProperty` 的 `get` 钩子时，将依赖数据属性的对应回调存入缓存对象对应数据属性键的数组中
- 调用`Object.defineProperty` 的 `set` 钩子时，遍历调用缓存对象对应数据属性键的数组中的回调

### 动态数据绑定三

<img width="700" height="700" src="http://ony85apla.bkt.clouddn.com/17-11-30/23737266.jpg">

#### 代码

~~~javascript
let _default = {
    el: 'body',
    data: {}
}

var Vue = function(options) {
    this.extend(this, _default, options);
    this.el = document.querySelector(this.el);
    this.compile();
}

Vue.prototype = {
    extend: function() {
        let obj = arguments[0]
        for (let i = 1; i < arguments.length; i++) {
            for (let key in arguments[i]) {
                if (arguments[i].hasOwnProperty(key)) {
                    obj[key] = arguments[i][key]
                }
            }
        }
    },
    compile: function() {
        new Compile(this.el, this)
    }
}

var Compile = function(el, vm) {
    this.el = el;
    this.fragment = null;
    this.vm = vm;
    this.init();
}

Compile.prototype = {
    init: function() {
        if (this.el) {
            // 都是对 fragment 操作 
            this.fragment = this.node2Fragment(this.el);
            this.compileElement(this.fragment); // 因为是频繁操作，因此不在dom上直接操作
            this.el.appendChild(this.fragment);
        } else {
            console.log('dom元素不存在！')
        }
    },
    node2Fragment: function(el) {
        // 生成一个空白的文档片段节点
        let fragment = document.createDocumentFragment();
        let child;
        while (child = el.firstChild) {
            fragment.appendChild(child);
        }
        return fragment
    },
    compileElement: function(el) {
        let childNodes = el.childNodes;
        let reg = /\{\{(.*)\}\}/;
        let self = this;

        // childNodes 是只读的类数组，需要转换为数组才能调用forEach方法
        Array.prototype.slice.call(childNodes).forEach(function(node) {
            let text = node.textContent;
            // if (self.isElementNode(node)) { // 判断是否是元素结点
            //     self.compile(node);
            // } else
            if (self.isTextNode(node) && reg.test(text)) { // 判断是否是文本节点
                self.compileText(node, reg.exec(text)[1]);
            } else if (node.childNodes && node.childNodes.length) { // 还有子节点
                self.compileElement(node);
            }
        })
    },
    compileText: function(node, exp) {
        console.log(node, exp)
        let arr = exp.split('.');
        let innerText = '';
        if (arr && arr.length) {
            for (let key in this.vm.data) {
                if (this.vm.data.hasOwnProperty(key) && key === arr[0]) {
                    let data = this.vm.data[key];
                    for (let i = 1; i < arr.length; i++) {
                        data = data[arr[i]]
                    }
                    innerText = data;
                }
            }
        }
        this.updateText(node, innerText)
    },
    updateText: function(node, innerText) {
        node.textContent = typeof innerText === 'undefined' ? '' : innerText;
    },
    isElementNode: function(node) {
        return node.nodeType === 1
    },
    isTextNode: function(node) {
        return node.nodeType === 3
    }
}
~~~

#### 知识点

添加模板解析，页面初始渲染时对 dom 的节点分类处理，使用不同的方法更新节点

## 双向数据绑定

### 基本概念

Vue 三要素

- `响应式` 如何监听数据变化
- `模板引擎` 如何解析模板
- `渲染` Vue 如何将监听到的数据变化和解析后的 html 进行渲染

实现数据双向绑定的方法：

- `基于观察者模式` KnockoutJS
- `基于脏检查` Angular
- `基于数据劫持`

> 目前业界分为以 react 为首的单向数据绑定，和以 angular、vue 为主的双向数据绑定。其实三大框架都能显示双向绑定和单向绑定，比如 react 可以手动绑定 onChange 和 value 实现双向绑定，vue 也加入了 props 这种单向 api。相对于其他双向绑定的方法，数据劫持无需显示调用。如 vue 运用数据劫持 + 观察者模式（发布订阅模式），直接通知变化并驱动视图

### vue 基于数据劫持和观察者模式实现双向绑定

> **Vue 目前是 `Object.defineProperty` 进行数据劫持 + 观察者模式，Vue 3.0 会使用 `Proxy` 取代 `Object.defineProperty`**

![](http://ony85apla.bkt.clouddn.com/18-6-11/7627095.jpg)

- `Observer 数据监听器` 能够对数据对象的所有属性进行监听(对数据对象递归遍历并添加 getter setter)，如有变动可拿到最新值（赋值触发 setter），**内部采用 Object.defineProperty 的 getter 和 setter 来实现**
- `Compiler 指令解析器` 对每个元素结点的**指令**进行扫描解析，**根据指令模板将模板变量替换成数据，以及为每个指令对应的节点绑定相应的更新函数，添加监听数据的订阅者，一旦数据变动，收到通知，更新视图**
- `Watcher 订阅者` 作为连接 Observer 和 Compile 的桥梁，**在自身实例化时往 `Dep` 中添加自身，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数**
- `Dep 消息订阅器` 内部维护了一个数组，用来**收集订阅者（Watcher），数据变动触发notify 函数，再调用订阅者的 update 方法**

从图中可以看出，当执行 `new Vue()` 时，Vue 就进入了初始化阶段，一方面 Vue 会遍历 `data` 选项中的属性，并用 `Object.defineProperty` 将它们转为 `getter/setter`，实现数据变化监听功能；另一方面，Vue 的指令编译器 `Compiler` 对元素节点的指令进行扫描和解析，初始化视图，并订阅 `Watcher` 来更新视图， 此时 `Wather` 会将自己添加到消息订阅器中(`Dep`), 初始化完毕。
当数据发生变化时，`Observer` 中的 `setter` 方法被触发，`setter` 会立即调用 `Dep.notify()`，`Dep` 开始遍历所有的订阅者，并调用订阅者的 `update` 方法，订阅者收到通知后对视图进行相应的更新。

#### Dep 发布者（负责存储订阅者和消息的分发）

```js
// 构建发布者 针对不同的 exp 存储
function Dep() {
	this.deps = {}; // 缓存列表
}
Dep.prototype.on = function(exp, watcher) {
	if (this.deps[exp]) {
		this.deps[exp].push(watcher);
	} else {
		this.deps[exp] = [watcher];
	}
}
Dep.prototype.notify = function(exp) {
	this.deps[exp] && this.deps[exp].forEach(watcher => {
		watcher.update();
	})
}
Dep.target = null;
```

#### Observer 监听者（用于监听属性值的变化）

```js
function Observer(data) {
	this.data = data;
	this.walk(data);
	// 每次实例化一个 Observer 就会实例化一个 Dep，用 dep 实例来管理所有的订阅器回调，每当属性变化遍历触发订阅器回调
	this.dep = new Dep();
}

// 遍历data，如果是 key 值是对象继续递归遍历
Observer.prototype.walk = function(data) {
	let childObj = {}
	Object.keys(data).forEach(key => {
		// 判断 key 的值是否是对象
		if (Object.prototype.toString.call(data[key]) === '[object Object]') {
			// 是对象继续遍历
			childObj = observe(data[key])
		}
		// 否则对 key 属性添加 getter setter
		this.defineReative(data, key, data[key]);
	})
}
Observer.prototype.defineReative = function(data, key, val) {
	Object.defineProperty(data, key, {
		get: () => {
			// 属性获取 将 watcher 添加到缓存列表
			// 设置getter 时，Dep.target 指向当前 watcher
			if (Dep.target) {
				this.dep.on(Dep.target.exp, Dep.target);
			}
			return val
		},
		set: newVal => {
			if (val === newVal) {
				return
			}
			val = newVal

			// 如果设置的新值是一个对象，添加 添加 getter setter
			if (Object.prototype.toString.call(newVal) === '[object Object]') {
				childObj = observe(newVal);
			}
			// 属性变化 遍历订阅器回调 触发watcher改变视图
			this.dep.notify(key);
		}
	})
}
function observe(data) {
	if (!data || typeof data !== 'object') {
		return 
	}
	return new Observer(data)
}
```

#### watcher 订阅者

```js
// 订阅者
function Watcher(vm, exp, cb) {
	this.vm = vm; // vm 实例
	this.exp = exp;  // 订阅 数据属性
	this.cb = cb;		// 订阅回调
	this.value = this.get(); // 实例化时候自动将自身添加到订阅器缓存列表
}

Watcher.prototype = {
	get: function() {
		Dep.target = this; // 将自身赋值
		let value = this.vm[this.exp]; // 调用数据劫持属性 get 方法
		Dep.target = null;
		return value
	},
	update: function() {
		// 新值 value
		let value = this.vm[this.exp];
		let oldValue = this.value;
		if (value === oldValue) {
			return
		}
		this.value = value;
		// 调用回调
		this.cb.call(this.vm, value, oldValue)
	}
}
```

#### compiler 编译器

```js
function Compiler(vm, el) {
	this.el = el;
	this.vm = vm;
	this.fragment = null;
	this.init();
}

Compiler.prototype = {
	// 不在dom上直接操作，而是在文档片段节点中操作，在 appendChild 到原来的dom上
	init: function() {
		if (this.el) {
			this.fragment = this.node2Fragment(this.el);
			this.compile(this.fragment);
			this.el.appendChild(this.fragment);
		} else {
			console.log('dom节点不存在')
		}
	},
	node2Fragment: function(el) {
		let child, fragment = document.createDocumentFragment();
		// 不断将 dom 移到 fragment 中
		while (child = el.firstChild) {
			fragment.appendChild(child);
		}
		return fragment
	},
	compile: function(root) {
		let children = root.children;
		for (let i = 0; i < children.length; i++) {
			let node = children[i]
			// 元素节点 进而判断属性
			if (this.isElementNode(node)) {
				this.compileElement(node)
			}
			// 文本内容
			if (/\{\{([^\}]+)\}\}/.test(node.textContent)) {
				this.compileText(node, node.textContent, /\{\{([^\}]+)\}\}/.exec(node.textContent)[1])
			}
			if (node.children.length) {
				console.log(node)
				// 子节点继续遍历
				this.compile(node)
			}
		}
	},
	compileElement: function(node) {
		let nodeAttrs = node.attributes
		Array.prototype.forEach.call(nodeAttrs, attr => {
			let attrName = attr.name
			let exp = attr.nodeValue

			// 判断 v-model 
			if (attrName.indexOf('v-model') > -1) {
				this.compileModel(node, this.vm, exp)
			}
			if (attrName.indexOf('v-on') > -1 || /^@/.test(attrName)) {
				let event = attrName.indexOf('v-on') > -1
					? attrName.slice(5)
					: /^@/.test(attrName) ? attrName.slice(2) : ''
				this.compileEvent(node, this.vm, exp, event)
			}

			// 处理后去除该属性
			node.removeAttribute(attrName)
		})
	}, 
	// 处理文本节点
	compileText: function(node, expTextContent, exp) {
		console.log(node, expTextContent, exp)
		this.updateText(node, expTextContent, this.vm[exp])
		// 初始化 Watcher 
		// 绑定 数据属性 和对应更新回调
		new Watcher(this.vm, exp, val => {
			this.updateText(node, expTextContent, this.vm[exp])
		})
	},
	// 处理 v-model 属性节点
	compileModel: function(node, vm, exp) {
		this.updateModel(node, vm[exp])
		// 初始化 Watcher 
		// 绑定 数据属性 和对应更新回调
		new Watcher(this.vm, exp, val => {
			this.updateModel(node, vm[exp])
		})

		// 上面只是做了数据到视图的单向绑定，视图对数据的修改是通过绑定节点的 input 事件
		node.addEventListener('input', e => {
			let value = e.target.value 
			if (this.vm[exp] === value) return
			this.vm[exp] = value
		}, false)
	},
	// 处理事件属性节点
	compileEvent: function(node, vm, exp, event) {
		let cb = vm.methods && vm.methods[exp]
		// 事件监听
		cb && node.addEventListener(event, cb.bind(vm), false)
	},
	// 更新 model
	updateModel: function(node, val) {
		node.value = typeof val === 'undefined' ? '' : val
	},
	// 更新 text
	updateText: function(node, expTextContent, val) {
		val = typeof val === 'undefined' ? '' : val
		node.textContent = expTextContent.replace(/\{\{([^\}]+)\}\}/, val)
	},
	// 判断是否是元素结点
	isElementNode: function(node) {
		return node.nodeType === 1
	}
}
```

#### vue 构造函数

```js
// 'use strict'
(function(global, factory) {
	// 判断是否是模块导出 或者 amd 或者 浏览器环境使用
	typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() :
		typeof define === 'function' && define.amd ? define(factory) :
		(global.SelfVue = factory());
}(this, function() {

	let SelfVue = function SelfVue(options = {}) {
		// 把 options 上的属性添加到 this 实例上
		Object.assign(this, options)
		this.data = this.data || {}
		this.el = document.querySelector(this.el || 'body')

		// 代理 SelfVue.data[key] 可以通过SelfVue[key]访问 如访问 this.title 等同 this.data.title
		Object.keys(this.data).forEach(key => {
			this.proxy(key);
		})
		// 监听所有数据变化
		observe(this.data)
		// new Observer(this.data);
		// 提取模板指令，初始化视图，并绑定订阅
		new Compiler(this, this.el);
		// 所有准备完毕后 
		this.mounted.call(this);
	}

	SelfVue.prototype.proxy = function(key) { 
		Object.defineProperty(this, key, {
			enumberable: false,
			configurable: true,
			get: () => {
				return this.data[key];
			},
			set: newVal => {
				if (this.data[key] === newVal) {
					return
				}
				this.data[key] = newVal;
			}
		})
	}

	return SelfVue
}))
```

[完整代码](https://github.com/jhgrrewq/myVue)

### Object.defineProperty 缺陷

#### 需要递归遍历进行数据劫持（对象的值还是对象）

#### 无法监听数组变化

Vue 支持以下 7 种方法来改变数组

- `pop()`
- `push()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

本质上是内部将无法监听数组的情况 hack 处理

```js
const aryMethods = ['push', 'pop', 'shift', 'unshift', 'reverse', 'sort', 'splice']
const arrayAugmentations = []

aryMethods.forEach(method => {
  // 原生 Array 原型方法
  let original = Array.prototype[method]
  // 将 push pop 等封装好的方法定义在对象 arrayAugmentations 属性上
  arrayAugmentations[method] = function() {
    // 调用对应原生方法并返回结果
    console.log('我被改变了')
    return original.apply(this, arguments)
  }
})

let list = ['a', 'b', 'c']
// 将要监听的数组的原型指针指向上面定义的空数组对象
list.__proto__ = arrayAugmentations
list.push('d') // 我被改变了 4

// 这里 list2 原型链没有被改变
let list2 = ['a', 'b', 'c']

list2.push('d') // 4
```

## Proxy 实现双向绑定

Proxy 在 es6 规范中正式发布，它在目标对象之前架设一层拦截，外接对该对象的访问，都必须通过这层拦截，因此提供一种机制，可以对外籍的访问进行过滤和修改

### Proxy 可以直接监听对象而非属性

Proxy 直接劫持整个对象，并返回一个新对象

### Proxy 可以直接监听数组的变化

对数组进行操作时（push pop splice 等），会触发对应的方法名称和 length 变化