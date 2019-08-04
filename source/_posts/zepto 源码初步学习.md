---
title: zepto 源码初步学习
tag: 
	- zepto
	- 源码
---

## zepto 对象设计 如 $('span')

~~~javascript
var arr = [1,2,3];
arr.__proto__ = {
    addClass: function () {
        console.log(123);
    }，
    …// 更多自定义工具函数
    push: Array.prototype.push,
    ….
};
arr.addClass();
~~~

实例对象的隐式原型原指向Array.prototype, 但这里重新赋值后已不再指向Array.prototype，不再继承Array，因此不是数组
**zepto对象是一个类似数组的非数组，拥有一些数组的方法而已**

<!--more-->

## zepto 核心模块基本结构

~~~javascript
var Zepto = (function(){
    var $, zepto = {};
    …

    zepto.init = function(selector, context){
       …
    }
    $ = function(selector, context){
        return zepto.init(selector,context)
    }
    …
    return $
})()

window.Zepto = Zepto;
window.$ === undefined && window.$ = window.Zepto
~~~

## zepto.init 函数

> 不同条件下对变量dom赋值（**最终赋值给dom的是一数组**）并和selector一起传给zepto.z函数

~~~javascript
zepto.init = function(selector, context){
     var dom;
     // 分情况给dom赋值
     // 1.selector为空
     // 2.selector为字符串，其中又分几种情况
     // 3.selector为函数
     // 4.其他情况，如selector为数组，对象等
     return zepto.Z(dom,selector)
}
~~~

### 无参数 $()

~~~javascript
    // If nothing given, return an empty Zepto collection
    if (!selector) return zepto.Z()
~~~

### selector 参数是字符串 如$('p') $('#content') $('`<div>`') 

~~~javascript
    else if (typeof selector == 'string') {
      selector = selector.trim()
      // If it's a html fragment, create nodes from it
      // Note: In both Chrome 21 and Firefox 15, DOM error 12
      // is thrown if the fragment doesn't begin with <
      if (selector[0] == '<' && fragmentRE.test(selector))
        dom = zepto.fragment(selector, RegExp.$1, context), selector = null
      // If there's a context, create a collection on that context first, and select
      // nodes from there
      else if (context !== undefined) return $(context).find(selector)
      // If it's a CSS selector, use it to select nodes.
      else dom = zepto.qsa(document, selector)
    }
~~~

- 参数为`<div>`，即一个html标签，首先用这个标签创建的dom对象，类似dom = document.createElement('div')，并封装进数组传给dom
- 若第二个参数有值，先根据第二个参数生成zepto对象，再调用find获取 如 $('.item', '#content')
- 反之，则是css选择器，调用的zepto.qsa函数则是对querySelectorAll方法的封装

### selector 参数是函数 如$(function(){}) 

等待dom加载完毕再执行其他函数

~~~javascript
    // If a function is given, call it when the DOM is ready
    else if (isFunction(selector)) return $(document).ready(selector)
~~~

### selector 为zepto对象 如 var a = $('p'); $(a)

调用zepto.isZ函数判断是否是zepto对象，如果是直接返回

~~~javascript
    // If a Zepto collection is given, just return it
    else if (zepto.isZ(selector)) return selector
~~~

### 其他情况

- selector为数组，调用 compact 方法处理一下再赋值dom

~~~javascript
    // normalize array if an array of nodes is given
    if (isArray(selector)) dom = compact(selector)
~~~

- selector为dom节点，**将它作为数组再赋值dom**

~~~javascript
    // Wrap DOM nodes.
    else if (isObject(selector))
      dom = [selector], selector = null
~~~

## zepto.Z函数

dom是一个数组，并且把它的隐式原型赋值$.fn，而这里的$.fn其实就是一个普通的js对象

~~~javascript
  // `$.zepto.Z` swaps out the prototype of the given `dom` array
  // of nodes with `$.fn` and thus supplying all the Zepto functions
  // to the array. Note that `__proto__` is not supported on Internet
  // Explorer. This method can be overriden in plugins.
  zepto.Z = function(dom, selector) {
    dom = dom || []
    dom.__proto__ = $.fn
    dom.selector = selector || ''
    return dom
  }
~~~

**$是一个函数，但同时也是一个对象，也可以给函数添加任意的属性和方法**

~~~javascript
    $ = function(selector, context){
        return zepto.init(selector,context)
    }
    $.fn = {
       // 工具函数
    }
~~~

## 总结

![](http://ony85apla.bkt.clouddn.com/17-11-29/42838935.jpg)

~~~javascript
var Zepto = (function(){
    var $,
        zepto = {}
    // ...省略N行代码...
    zepto.Z = function(dom, selector) {
      dom = dom || []
      dom.__proto__ = $.fn
      dom.selector = selector || ''
      return dom
    }
    zepto.init = function(selector, context) {
        var dom
        // 针对参数情况，分别对dom赋值
        // 最终调用 zepto.Z 返回的数据
        return zepto.Z(dom, selector)
    }
    $ = function(selector, context){
        return zepto.init(selector, context)
    }
    $.fn = {
        // 里面有若干个工具函数
    }
    // ...省略N行代码...
    return $
})()

window.Zepto = Zepto
window.$ === undefined && (window.$ = Zepto)
~~~