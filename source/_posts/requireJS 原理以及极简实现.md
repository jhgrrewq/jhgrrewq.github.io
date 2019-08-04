---
title: requireJS 原理以及极简实现
tag: 
	- requireJS
---

> 依次加载 require 的模块， 然后检测 script 的 onloade 事件，判断所有的模块加载完毕后，执行 require 的 callback。如果 require 的只有一个参数而不是数组，就加载完成 return 的模块

<!-- more -->

./tinyRequire.js

```javascript
(function(global, factory) {
    typeof exports === 'object' && typeof module !== 'undefined' ? module.exports === factory() : typeof define === 'function' && typeof define.amd !== 'undefined' ? define(factory) : global.require = factory()
})(this, function() {
    //标记已经加载成功的个数
    var REQ_TOTAL = 0;
    //模块导出
    window.exports = {};
    //记录各个模块的顺序
    var exp_arr = [];

    //require 真正实现
    function require(arr, callback) {
        var req_list;

        arr instanceof Array ? req_list = arr : req_list = [arr];
        
        //模块逐个加载
        for(var i=0;i<req_list.length;i++) {
            var req_item = req_list[i];

            var $script = createScript(req_item, i);

            var $node = document.querySelector('head');

            (function($script) {
                // 检测script 的onload事件
                $script.onload = function() {
                    REQ_TOTAL++;
                    var script_index = $script.getAttribute('index');

                    exp_arr[script_index] = exports;

                    window.exports = {};

                    //所有链接加载成功后，执行callback
                    if(REQ_TOTAL == req_list.length) {
                        callback && callback.call(exports, ...exp_arr);
                        // 或者用 callback && callback.apply(exports, exp_arr);
                    }
                }
                $node.appendChild($script);
            })($script);
        }
    }

    //创建一个script标签
    function createScript(src, index) {
        var $script = document.createElement('script');
        $script.setAttribute('src', src);
        $script.setAttribute('index', index);
        return $script;
    }

    return require
})
```

./test1.js

```javascript
// 定义简单的test1.js
exports.test1 = {
    title: 'test1',
    info: 'this is a simple test1',
    print: function() {
        console.log('title: ' + this.title + this.info)
    }
}
```

./test2.js

```javascript
// 定义简单的test2.js
exports.test2 = {
    title: 'test2',
    info: 'this is a simple test2',
    print: function() {
        console.log('title: ' + this.title + this.info)
    }
}
```

index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script src="./tinyRequireJs.js"></script>
<script>
    require(["./test/test1.js", "./test/test2.js"], function(ts1, ts2) {
        ts1.test1.print()
        ts2.test1.print()
        console.log(ts1, ts2)
    })
</script>
</body>
</html>
```