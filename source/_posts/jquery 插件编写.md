---
title: jquery 插件编写
tag: 
	- jquery
---
## jquery 源码总体结构分析

```javascript
(function(window, undefined) {
    var jQuery = function(selector, context) {
        // 通常创建一个对象或实例是通过new 一个构造函数，但是如果构造函数有返回值，new创建的对象会被丢弃，返回值将作为new表达式的值。这里jQuery()内部new创建并返回另一个构造函数的实例，省去了构造函数jQuery()前面的new，也就是可以直接省略new写jQuery()
        return new jQuery.fn.init(selector, context, rootjQuery)
    }
    // 一些局部变量声明

    jQuery.fn = jQuery.prototype = {
        // jQuery对象是类数组
        construtor: jQuery,
        init: function(selector, context, rootjQuery) {...}
        // 一堆原型属性和方法
    }

    // 关键 jQuery.fn.init实例化对象能拥有 jQuery 原型上的属性和方法
    jQuery.fn.init.prototype = jQuery.fn
    // 多个对象合并
    jQuery.extend = jQuery.fn.extend = function() {...}

    // 当只有一个参数是扩展jQuery全局对象
    jQuery.extend({
        // 一堆静态属性和方法
    })

})(window)
```

<!--more-->

> 只要将调用jquery插件的方法挂载在jquery原型上即可

## 实现

- $.fn.function

将插件方法 挂载在原型上

- $.fn.extend({functionName: function})

$.fn.extend方法当只有一个参数是在扩展原型

```javascript
// 使用立即执行函数，独立作用域，避免$变量全局污染
(function($){
    $.fn.changeFont = function(option) {
        this.css('fontSize', option.fontSize + 'px')
        // 链式调用，只需返回当前 jQuery 对象
        return this
    }
})(jQuery)

// 调用
$('p').changeFont({
    fontSize: '15'
})
```