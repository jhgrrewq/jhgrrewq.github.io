---
title: js 事件
tag: 
	- js
---

## DOM 级别事件（DOM0 DOM2 IE）

### HTML 事件处理程序

```html
<script>
function showMsg() {
  console.log('Clicked')
}
</script>
<input type="button" value="Click me" onclick="showMsg()"/>
```

<!-- more -->

这里在 html 代码中定义一个 onclick 属性触发 `showMsg()`。**HTML 事件处理程序的最大缺点是 html 和 js 强耦合**，一旦需要修改函数名就得修改两个地方, 优点是不需要操作 dom 来完成事件的绑定

### DOM0 级事件处理程序

> DOM0 级事件就是将一个函数赋值给一个事件处理属性

使用 dom element 上的 **on + eventType 属性**

```js
var btn = document.getElementById('btn')
btn.onclick = function() {
  console.log('CLicked')
}
```

**DOM0 级事件处理程序的缺点在于一个处理程序无法同时绑定多个处理函数**，同时绑定过个处理函数只有最后一个优先，之前的会被覆盖

## DOM2 级事件处理程序（不支持 IE8 以下）

DOM2 级事件**允许一个处理程序天机多个处理函数**。其定义的 `addEventListener` `removeEventListener` 方法，分别用来绑定和解绑事件。方法包含 3 个参数：**绑定的事件名称、处理函数、是否需要在捕获时执行事件处理函数**

```js
function showMsg() {
  console.log('clicked')
}
function showText(){
  console.log('hello')
}

var btn = document.getElementById('btn')
btn.addEventListener('click', showMsg, false) // 在冒泡阶段执行
btn.addEventListener('click', showText, false) // 在冒泡阶段执行
btn.removeEventListener('click', showText, false)
```

## IE 事件处理程序

IE 实现了和 DOM2 中类似的方法（**因为 IE8 以下不支持 DOM2，并且只支持事件冒泡**）`attachEvent` `detachEvent`, 方法传入两个参数：**on + 绑定的事件名称、处理函数**

```js
function showMsg() {
  console.log('clicked')
}
function showText(){
  console.log('hello')
}

var btn = document.getElementById('btn')
btn.attachEvent('onclick', showMsg)
btn.addEvent('onclick', showText) // 注意首先看到 hello 才看到 clicked
btn.removeEvent('onclick', showText)
```

IE 事件处理和 DOM2 事件处理不同

- **IE 事件处理程序不是以添加的顺序执行，而是相反的顺序**
- IE 事件处理 `attachEvent` `detachEvent` 方法第一个参数 绑定的事件处理属性名称(**包含 on**)
- IE 事件处理只支持事件冒泡 `attachEvent` `detachEvent` 方法不含第三个参数

## 事件流

事件流描述的是从页面中接受事件的顺序。 IE 的事件流是事件冒泡流， Netscape Communicator 的事件流是事件捕获流

### 事件冒泡

> 事件冒泡就是从事件触发的目标元素向上传播

IE 的事件流就是事件冒泡（IE 8 以下只支持事件冒泡）

### 事件捕获

> 和事件冒泡相反，事件捕获是从上而下，事件的目标源最后才接收到事件

### DOM2 级事件流

DOM2 级事件流包括 3 个阶段：**事件捕获阶段、处于目标事件阶段和事件冒泡阶段**

`addEventListener` 方法第三个参数设置 true，表示是在事件捕获阶段触发；设置为 false，表示在事件冒泡阶段触发

## 事件对象

标准浏览器在事件处理程序中被调用时 事件对象 会通过参数传递给处理程序，IE8 及以下浏览器汇总事件对象通过全局 `window.event` 访问

| | DOM0/DOM2 | IE |
| :------: | :------: | :------: |
| 事件对象 | event | window.event |
| 事件源 | event.target | window.event.srcElement |
| 正在处理事件源 | event.curerntTarget | this |
| 阻止默认行为 | event.preventDefault() | window.event.returnValue = false |
| 停止冒泡 | event.stopPropagation() | window.event.cancelBubbel = false |

### type 属性

type 属性可以获取事件发生的类型

- `event.type`

### 鼠标事件属性

- `event.clientX` 获取鼠标基于浏览器窗口的 X 轴坐标
- `event.clientY` 获取鼠标基于浏览器窗口的 Y 轴坐标
- `event.pageX` 获取鼠标基于文档页面的 X 轴坐标（可能含有滚动）
- `event.pageY` 获取鼠标基于文档页面的 Y 轴坐标（可能含有滚动）
- `event.screenX` 获取鼠标相对屏幕的 X 轴坐标
- `event.screenY` 获取鼠标相对屏幕的 Y 轴坐标
- `event.which` 获取指定事件上哪个键盘键或鼠标按钮被按下

### 键盘事件属性

- `event.keyCode` 获取按下键的键码值
- `event.ctrlKey` 获取是否按下 ctrl
- `event.shiftKey` 获取是否按下 shift
- `event.altKey` 获取是否按下 alt
- `event.metaKey` 获取是否按下 meta


## 跨浏览器的事件对象

```js
var EventUtil = {
  addHandler: function(element, type, handler) {
    if (event.addEventListener)
      event.addEventListener(type, handler, false)
    else if (element.attachEvent)
      event.attachEvent('on' + type, handler)
    else
      element['on' + type] = handler
  },
  removeHandler: function(element, type, handler) {
    if (event.removeEventListener)
      event.removeEventListener(type, handler, false)
    else if (element.detachEvent)
      event.detachEvent('on' + type, handler)
    else
      element['on' + type] = null
  },
  getEvent: function(event) {
    return event ? event : window.event
  },
  getTarget: function(event) {
    return event.target || window.event.srcElement
  },
  stopPropagation: function(event) {
    if (event.stopPropagation)
      event.stopPropagation()
    else
      window.event.cancelBubble = true
  },
  preventDefault: function(event){
    if (event.preventDefault) {
      event.preventDefault()
    else
      window.event.returnValue = false
  },
}
```

## 事件代理/事件委托

对 '事件处理程序过多' 的问题的解决方案就是 事件委托，**事件委托利用了事件冒泡**，只指定一个事件处理程序，就可以管理某一类型的所有事件；通常也用于给不存在的（未来可能会存在）的元素绑定事件

## jQuery 中事件绑定

jQuery 提供了很多事件绑定的 api，核心都是调用了 jQuery 底层事件的 `jQuery.event.add` 方法。其实现也是上面提到的 `addEventListener` `attachEvent`

### bind

jQuery 早期版本 `bind()` 只能为**存在**的匹配元素的特定事件绑事件处理函数

jQuery 1.7 版本不再建议使用，用 `on()` 代替, jQuery 3.0 版本已经废弃，用 on() 代替

`$(selector).bind(event, data, function)`

- `event` （必需）添加到元素的一个或多个事件; 可用空格分隔添加多事件;可用大括号灵活定义多事件

- `data` (可选) 传递的参数

- `function` (必需) 事件绑定的处理函数

```js
// a 标签必须是实际存在的 dom
$("a").bind("click", data, function(){alert("ok");});

// 空格分隔添加多个事件
$(selector).bind("dbclick click", data, function(){alert("ok");});

// 使用大括号为每个事件单独绑定函数
$(selector).bind({event1: function, event2: function});
```

`$(selector).unbind(event, [function])` 从每个匹配的元素中删除一个或多个绑定的事件处理函数

### live

jQuery 早期版本 `live()` 利用**事件冒泡可以将当前或未来的匹配元素添加一个或多个事件处理程序**，实现事件委托(**内部实现是把事件都绑定到 document 上**)

jQuery 1.7 版本不再建议使用，用 `on()` 代替，对旧版本用户优先使用 `delegate()`

`$(selector).live(event, data, function)`

- `event` （必需）添加到元素的一个或多个事件; 可用空格分隔添加多事件;可用大括号灵活定义多事件

- `data` (可选) 传递的参数

- `function` (必需) 事件绑定的处理函数

```js
$("ul").live("click", function(){alert("ok");});

// 空格分隔添加多个事件
$("ul").live("dbclick click", data, function(){alert("ok");});

// 使用大括号为每个事件单独绑定函数
$("ul").live({"click": function, "dbclick": function});
```

### delegate

1.4 版本引入的 `delegate()` 同样利用**事件冒泡可以将当前或未来的匹配元素添加一个或多个事件处理程序**，实现事件委托。只不过 `live()` 是通过 `document` 元素委派，`delegate()` 可以是任意元素

`$(selector).delegate(childSelector, event, data, function)`

- `childSelector` （必需）需要添加事件处理程序的元素，一般为 selector 的子元素

- `event` （必需）添加到元素的一个或多个事件; 可用空格分隔添加多事件;可用大括号灵活定义多事件

- `data` (可选) 传递的参数

- `function` (必需) 事件绑定的处理函数

```js
$("ul").delegate("li", "click", function(){alert("ok");});

// 空格分隔添加多个事件
$("ul").delegate("li", "dbclick click", data, function(){alert("ok");});

// 使用大括号为每个事件单独绑定函数
$("ul").delegate("li", {"click": function, "dbclick": function});
```

### on

1.7 版本引入的 `on()` 同样利用**事件冒泡可以将当前或未来的匹配元素添加一个或多个事件处理程序**，实现事件委托。**和 `delegate()` 不同的是前两个参数的顺序**

`$(selector).on(event, childSelector, data, function)`

- `event` （必需）添加到元素的一个或多个事件; 可用空格分隔添加多事件;可用大括号灵活定义多事件

- `childSelector` （必需）需要添加事件处理程序的元素，一般为 selector 的子元素

- `data` (可选) 传递的参数

- `function` (必需) 事件绑定的处理函数

```js
$("ul").on("click", "li", function(){alert("ok");});

// 空格分隔添加多个事件
$("ul").on("dbclick click", "li", data, function(){alert("ok");});

// 使用大括号为每个事件单独绑定函数
$("ul").on({"click": function, "dbclick": function}, "li");
```

`$(selector).off(event, [childSelector], [function])` 从每个匹配的元素中删除一个或多个绑定的事件处理函数

### delegate 简单实现

```js
function delegate(parent, event, child, fn) {
  if (typeof parent === 'string') {
    parentEle = document.querySelector(parent)
    !parent && alert('parent not found')
  }
  if (typeof child !== 'string') {
    alert('child not string')
  }
  if (typeof fn !== 'function') {
    alert('fn not function')
  }

  function handler(event) {
    var evt = event ? event : window.event
    var target = evt.target || evt.srcElement
    var currentTarget = event ? event.currentTarget : this
    if (target.id === child || target.className.indexOf(child) !== -1) fn.call(target)
  }

  function addEvent(element, type, handler) {
    if (event.addEventListener)
      event.addEventListener(type, handler, false)
    else if (element.attachEvent)
      event.attachEvent('on' + type, handler)
    else
      element['on' + type] = handler
  }

  addEvent(parent, event, handler)
}
```

## 自定义事件与发布/订阅设计模式

自定义事件是设计模式中 发布/订阅 的一种实现