---
title: js 深拷贝
tag: 
	- 深拷贝
---

## 概念

- 深拷贝和浅拷贝

对于基本数值类型，浅拷贝就是对值的复制；对于引用类型，js中存储对象都是存储地址，浅拷贝(变量赋值)都是指向同一个内存区域，深拷贝则是另外开辟一个内存区域

- 为什么需要深拷贝

一个简单的使用是vue父组件通过props传值给子组件，而有时子组件需要修改父组件的值。由于vue不建议直接修改父组件的值，对传入引用类型的值，往往需要拿到一个引用类型的副本。

<!--more-->

## 实现

> 对于非引用类型，直接复制；对于引用类型还要继续遍历，递归拷贝


```javascript
function deepClone(value) {
    var copy

    if (value == null || typepf value !== 'object') {
        // 直接返回赋值
        return value
    }

    // 处理 时间类型对象
    if (value instanceof Date) {
        copy = new Date()
        copy.setTime(value.getTime())
        return copy
    }

    // 处理 数组
    if (value instanceof Array) {
        copy = []

        for(var i = 0; i < value.length; i++) {
            // 数组元素可能还是引用类型，递归调用
            copy[i] = deepClone(value[i])
        }

        return copy
    }

    // 处理对象
    if (value instanceof Object) {
        copy = {}

        for(var key in value) {
            if(value.hasOwnProperty(key)) {
                // 对象的属性是自己的属性，而不会继承的属性
                // 对象的属性可能还是引用类型，递归调用
                copy[key] = deepClone(value[key])
            }
        }

        return copy
    }

    throw new Error('Unable to copy value! Its type isn't supported')
}
```

这里判断数据类型用instanceof，更好的判断方法是用Object.prototype.toString,可封装一个判断数据类型的方法

```javascript
function type(obj) {
    var toString = Object.prototype.toString
    var map = {
        '[object Boolean]' : 'boolean', 
        '[object Number]'  : 'number', 
        '[object String]'  : 'string', 
        '[object Function]' : 'function', 
        '[object Array]'  : 'array', 
        '[object Date]'   : 'date', 
        '[object RegExp]'  : 'regExp', 
        '[object Undefined]': 'undefined',
        '[object Null]'   : 'null', 
        '[object Object]'  : 'object'
    }
    return map[toString.call(obj)]
}
```

## 其他方法实现

### Object.assign({}, obj1, ...)

只是实现浅拷贝

### JSON.parse(JSON.stringify(obj))

深拷贝数据类型只支持基本数值类型，序列化时函数和原型成员会被忽略

```javascript
var obj = {
    num: 1,
    fn: function() {console.log(this.num)}
}

// JSON.stringify 就已经把function给过滤掉了
JSON.stringify(obj) // {num: 1}

```

### lodash

浅拷贝

```javascript
_.clone(obj)
```

深拷贝

```javascript
_.cloneDeep(obj)
```

### jquery $.extend() 或者 $.fn.extend()

浅拷贝

```javascript
$.extend({}, obj1, ...)
```

深拷贝

```javascript
$.extend(true, {}, obj1, ...)
```

源码

```javascript
$.extend(target, obj1, ....)

jQuery.extend = jQuery.fn.extend = function() {
    var src, copyIsArray, copy, name, options, clone,
        target = arguments[0] || {}
        i = 1
        length = arguments.length 
        deep = false // 默认浅拷贝

        // 处理 拷贝情况
        if (typeof target === 'boolean') {
            // 第一个参数是布尔值，目标对象从第二个参数开始
            deep = target

            target = arguments[1] || {}
            i++
        }

        // 处理 target 是字符串或者其他的情况
        if (typeof target !== 'object' && !jQuery.isFunction(target)) {
            target = {}
        }

        // 只有一个参数则是扩展jQuery对象
        if (i === length) {
            target = this // 目标对象是jQuery对象
            i--
        }

        for(;i < lenght; i++){
            // 只处理 非null或undefined数据
            if ((options = arguments[i]) != null) {
                // 扩展基本对象
                for（name in options）{
                    src = target[name]
                    copy = options[name]

                    // 防止死循环
                    if (target === copy) {
                        continue
                    }

                    if (deep && copy && (jQuey.isPlainObject(copy) || copyIsArray = jQuey.isArray(copy)) ) {
                        // 拷贝对象子属性是数组
                        // 目标对象子属性需要也同样要是数组
                        if (copyIsArray) {
                            copyIsArray = false
                            clone = src && jQuey.isArray(src) ? src : []
                        } else {
                            // 拷贝对象子属性还是对象
                            // 目标对象子属性需要也同样要是对象
                            clone = src && jQuey.isPlainObject(src) ? src : {}
                        }

                        // 递归深拷贝
                        target[name] = jQuery.extend(deep, clone, copy)
                    } else if (copy !== undefined) {
                        // 不为undeined的基本数据类型直接赋值
                        target[name] = copy
                    }
                }
            }
        }

        // 返回修改后的target
        return target
}

```
