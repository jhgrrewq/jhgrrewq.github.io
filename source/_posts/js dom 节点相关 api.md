---
title: js dom 节点相关 api
tag: 
	- js
---

> 参考: [原生JS中DOM节点相关API合集](https://microzz.com/2017/04/06/jsdom/)

## 节点属性

```js
node.nodeName // 返回节点名称，只读
node.nodeType // 返回节点类型的常数值，只读
        // 1 代表 Element 元素结点
        // 2 代表属性节点
        // 3 代表 Text 文本节点
        // 8 代表Comment注释节点
        // 9 代表Document节点
        // 11 代表DocumentFragment文档片段节点
node.nodeValue // 返回 Text 或 Comment 节点的文本值，只读
node.textContent // 返回当前节点和后代所有节点的文本内容， 可读写
node.baseURI // 返回当前页面的绝对路径
node.ownerDocument // 返回当前节点所在顶层文档对象 document

node.nextSibling // 返回紧跟在当前节点后面的第一个兄弟节点
node.previousSibling // 返回当前节点前面距离最近的一个兄弟节点
node.parentNode // 返回当前节点的父节点
node.childNodes // 返回当前节点的全部子节点（只读的类数组，可能包含空白节点）
node.firstChild // 返回当前节点的第一个子节点
node.lastChild // 返回当前节点的最后一个子节点
node.children // 返回指定节点的全部 element 子节点（Text 节点或 Comment 节点没有 children 属性）
node.firstElementChild // 返回当前节点的第一个 element 子节点
node.lastElementChild // 返回当前节点的最后一个 element 子节点
```

<!-- more -->

## 操作

```js
node.appendChild(newNode) // 向节点添加最后一个子节点
node.hasChildNodes() // 返回布尔值，表示当前节点是否有子节点
node.cloneNode(true) // 默认为 false（克隆节点），true(克隆节点和其属性以及后代)
node.insertBefore(newNode, oldNode) // 在指定子节点之前插入新的子节点c
node.removeChild(child) // 删除子节点，必须在删除节点的父节点上操作
node.replaceChild(newChild, oldChild) // 替换节点
node.contain(newNode) // 返回一个布尔值，表示参数节点是否是当前节点的后代节点
node.compareDocumentPosition(newNode) // 返回一个 7 比特位的二进制值，表示参数节点和当前节点的关系
node.isEqualNode(newNode) // 返回布尔值，检查两个节点是否相等。两个节点相等就是节点的类型相同、属性相同、子节点相同
node.normalize() //用于清理当前节点内部全部 Text 节点。会去除空的文本节点并将毗邻的文本节点合并为一个

node.remove() // 用于删除当前节点
```

## Document 节点

### Document 节点属性

```js
document.doctype // 返回文档 doctype
document.documentElement // 返回当前文档的根节点
document.defaultView // 返回 document 对象所在的 window 对象
document.body // 返回当前文档的 body 节点
document.head // 返回当前文档的 head 节点
document.activeElement // 返回当前文档中获得焦点的元素

// 节点集合属性
document.links // 返回当前文档的全部 a 元素
document.forms // 返回页面中全部表单元素
document.images // 返回页面中全部图片元素
document.embeds // 返回页面中全部嵌入对象
document.scripts // 返回当前文档中所有脚本
document.styleSheets // 返回当前网页的全部样式表

// 文档信息属性
document.documentURI // 表示当前文档网址
document.URL // 返回当前文档网址
document.domain // 返回当前文档域名
document.lastModified // 返回当前文档最后修改的时间戳
document.location // 返回 location 对象，提供当前文档的 URL 信息
document.referrer // 返回当前文档的访问来源
document.title // 返回当前文档的标题
document.characterSet // 返回渲染当前文档的字符集，如 UTF-8
document.readState // 返回当前文档的状态
document.desigeMode // 控制当前文档是否可编辑，可读写
document.compatMode // 返回浏览器处理文档的模式
document.cookie // 用来操作 cookie
```

### document 节点方法

```js
// 读写方法
document.open() // 新建并打开一个文档
document.write() // 向当前文档写入内容
document.writeIn() // 向当前文档写入内容，尾部添加换行符

// 查找结点
document.querySelector(selectors) // 接受一个 css 选择器作为参数，返回第一个匹配该选择器的元素结点
document.querySelectorAll(selectors) // 接受一个 css 选择器作为参数，返回所有匹配该选择器的元素结点
document.getElementsByTagName(tagName) // 返回指定 html 标签的元素
document.getElementsByClassName(className) // 返回包含所有 class 符合指定条件的元素
document.getElementsByName(name) // 选择拥有 name 属性的 HTML 元素（如 <form> <radio> <img> <frame> <embed> <object>）
document.getElementById(id) // 返回匹配指定 id 属性的元素节点
document.elementFromPoint(x, y) // 返回位于页面指定位置最上层的 element 子节点

// 生成节点
document.createElement(tagName) // 生成 html 元素节点
document.createTextNode(text) // 生成文本节点
document.createAttribute(name) // 申城一个新的属性对象节点，并返回它
document.createDocumentFragment() // 生成一个 DocumentFragment 对象

document.createEvent(event) // 生成一个事件对象，该对象能被 element.dispatchEvent() 方法使用
document.addEventListener(type, listener, capture) // 注册事件
document.removeEventListener(type, listener, capture) // 注销事件
document.dispatchEvent(event) // 触发事件

document.hasFocus() // 返回一个布尔值，表示当前文档中是否有元素被激活
document.adoptNode(externalNode) // 将某个节点从原来所在文档中移除，插入当前文档，并返回插入后的新节点
document.importNode(externalNode, deep) // 件外部文档拷贝指定节点，插入当前文档
```

## Element 节点

### Element 节点属性

```js
// 特性属性
element.attributes // 返回当前元素结点的所有属性节点
element.id // 返回指定元素的 id 属性，可读写
element.tagName // 返回指定元素的大写标签名
element.innerHTML // 返回该元素包含的 html 代码，可读写
element.outerHTML // 返回指定元素结点的所有 html 代码，包括自身和包含的所有子元素，可读写
element.className // 返回当前元素的 class 属性，可读写
element.classList // 返回当前元素节点的所有 class 集合
element.dataset // 返回元素节点中所有 data-* 属性

// 尺寸属性
element.clientWidth // 返回元素结点可见部分的宽度(含 padding，不含 border margin 滚动条)
element.clientHeight // 返回元素结点可见部分的高度(含 padding，不含 border margin 滚动条)
element.clientTop // 返回元素结点距左边框的宽度
element.clientLeft // 返回元素结点距顶部边框的高度
element.offsetWidth // 返回元素的水平宽度（包括 padding border）
element.offsetHeight // 返回元素的垂直高度（包括 padding border）
element.offsetLeft // 返回当前元素左上角相对 element.offsetParent 节点的水平偏移（包括 padding border 滚动条）
element.offsetTop // 返回当前元素左上角相对 element.offsetParent 节点的垂直偏移（包括 padding border 滚动条）
element.scrollWidth // 返回元素结点的总高度（包括看不见的部分 滚动的部分）
element.scrollWidth // 返回元素结点的总宽度（包括看不见的部分 滚动的部分）
element.scrollWidth // 返回元素结点的水平滚动条向右滚动的像素数值，通过设置该属性可以改变元素的滚动位置
element.scrollWidth // 返回元素节点垂直滚动向下滚动的像素数值
element.style // 返回元素节点的行内样式

// 节点相关属性
element.children // 当前元素节点的左右子元素
element.childElementCount // 返回当前元素节点包含的子 html 元素节点的个数
element.firstElementChild // 返回当前节点的第一个 element 子节点
element.lastElementChild // 返回当前节点的最后一个 element 子节点
element.nextElementSibling // 返回当前元素节点的下一个兄弟 HTML 元素节点
element.previousElementSibling // 返回当前元素节点的前一个兄弟 HTML 元素节点
element.offsetParent // 返回当前元素节点最靠近的，并且 css 的 position 属性不等于 static 的父元素
```

### Element 节点方法

```js
// 位置方法
element.getBoundingClientRect() // 返回一个对象，包含 top、left、right、bottom、width、height
    // top 元素(不包括 margin)上边界距离视口原点的距离
    // left 元素(不包括 margin)左外边界距离视口原点的距离
    // right 元素(不包括 margin)右外边界距离视口原点的距离
    // bottom 元素(不包括 margin)下边界距离视口原点的距离
    // width 元素自身宽度（包含 border padding 滚动部分）
    // height 元素自身高度（包含 border padding 滚动部分）

//属性方法
element.getAttribute(name) // 读取指定属性
element.setAttribute(name, value) // 设置指定属性
element.hasAttribute(name) // 返回一个布尔值，表示当前元素结点
element.removeAttribute(name) // 移除指定属性

// 查找方法
element.querySelector()
element.querySelectorAll()
element.querySelectorByTagName()
element.querySelectorByClassName()

// 事件方法
element.addEventListener()
element.removeEventListener()
element.dispatchEvent()
element.attachEvent(oneventName, listener) // ie8 及以下
element.detachEvent(oneventName, listener) // ie8 及以下

element.scrollInteView() // 滚动当前元素，进入浏览器可见区域

element.insertAdjacentHTML(where, htmlString) // 解析 html 字符串，然后将生成的节点插入到 dom 树的指定位置
element.insertAdjacentHTML('beforeBegin', htmlString) // 在该元素前插入
element.insertAdjacentHTML('afterBegin', htmlString) // 在该元素的第一个子元素前插入
element.insertAdjacentHTML('beforeEnd', htmlString) // 在该元素最后一个子元素后面插入
element.insertAdjacentHTML('afterEnd', htmlString) // 在该元素后插入

element.remove() // 将当前元素节点从 dom 中移除
element.focus() // 将当前页面焦点转移到指定元素上
```

## 补充

### document.createDocumentFragment()

- DocumentFragment 是独立的，不属于任何其他文档，其 parentNode 为 null
- 有任意多个子节点，**可通过 `appendChild()` `insertBefore()` 等方法传递一个 DocumentFragment 时，文档片段子节点会从片段移动到文档中，文档片段清空**

```js
var fragment = document.createDocumentFragment()
var el = document.querySelector(selector)
var child = el.firstChild
while(child) {
  fragment.appendChild(child)
  child = el.firstChild
}
```

### document.createAttribute(name)

```js
var node = document.createElement('div')
var attr = document.createAttribute('class')
attr.value = 'newClass'
node.setAttributeNode(attr)
```

- `node.setAttributeNode(node)` 设置或更新当前元素属性为指定的属性节点
- `node.setAttribute(attribute,attributeValue)` 设定具有指定名称的属性值
- `node.getAttributeNode(node)` 返回当前元素属性为指定的属性节点
- `node.getAttribute(attribute)` 返回具有指定名称的属性值

### node.cloneNode 重写（利用 documentFragment）

```js
function cloneNode(node, flag) {
  var fragment = document.createDocumentFragment()
  var child = node.firstChild
  while(child) {
    fragment.appendChild(child)
    child = node.firstChild
  }
  flag && node.appendChild(fragment)
  return node
}
```

![](http://ony85apla.bkt.clouddn.com/18-6-4/97931137.jpg)