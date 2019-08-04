---
title: js 工具函数
tag: 
	- js
---

> 参考: [js:防抖动与节流](https://blog.csdn.net/crystal6918/article/details/62236730)

<!-- markdownlint-disable MD010 -->

## 判断字符串回文

### 字符串变成数组倒序比较

```js
function plalindrome(str) {
	let reStr = str.split('').reverse().join('')
	return reStr === str
}

console.log(plalindrome('asdfdsa')) // true
console.log(plalindrome('asdf')) // false
```

### reduceRight

```js
function plalindrome(str) {
	let arr = str.split('')
	let reStr = arr.reduceRight((prev, cur) => {
		return prev + cur
	})
	return reStr === str
}

console.log(plalindrome('asdfdsa')) // true
console.log(plalindrome('asdf')) // false
```

### 利用栈

将字符串依次入栈，再一次出栈组成新的字符串（逆序的字符串），最后再比较

## 字符串中单词首字母大写

### for / map + replace

```js
function titleCase(str) {
	let convertArr = str.split(' ')
	for (let i = 0; i < convertArr.length; i++) {
		convertArr[i] = convertArr[i].replace(convertArr[i].charAt(0), convertArr[i].charAt(0).toUpperCase())
	}
	return convertArr.join(' ')
}

console.log(titleCase('adfa Asd'))
```

### map + slice

```js
function titleCase(str) {
	return str.split(' ').map(item => {
		return item.charAt(0).toUpperCase() + item.slice(1)
	}).join(' ')
}

console.log(titleCase('adfa Asd'))
```

### 正则匹配替换

```js
// 每次匹配一个单词
function titleCase(str) {
	return str.replace(/\w\S*/g, item => {
		return item.charAt(0).toUpperCase() + item.slice(1)
	})
}

// 或者匹配出最开始出现的首字母和每个单词的首字母
function titleCase(str) {
	return str.replace(/( |^)\w/g, item => {
		return item.toUpperCase()
	})
}

console.log(titleCase('adfa Asd'))
```

## 字符串中长度最长单词输出并输出长度

### for 循环

```js
function longest(str) {
	let arr = str.split(' ')
	let longNum = 0
	let longer = arr[0]

	for (let i = 0; i < arr.length; i++) {
		if (arr[i].length > longNum) {
			longNum = arr[i].length
			longer = arr[i]
		}
	}
	return [longer, longNum]
}

console.log(longest('asd dffsdf sdffffff')) // ["sdffffff", 8]
```

### reduce

```js
function longest(str) {
	let arr = str.split(' ')

	let longer = arr.reduce((prev, cur) => {
		return prev.length > cur.length ? prev : cur
	})
	return [longer, longer.length]
}

console.log(longest('asd dffsdf sdffffff')) // ["sdffffff", 8]
```

### sort

```js
function longest(str) {
	let arr = str.split(' ')

	let longer = arr.sort((a, b) => {
		return a.length - b.length
	}).pop()
	return [longer, longer.length]
}

console.log(longest('asd dffsdf sdffffff')) // ["sdffffff", 8]
```

## 字符串中重复次数最多的字符

```js
function most(str) {
	if (str.length === 1) {
		return [str, str.length]
	}
	let charObj = {}
	for (let i = 0; i < str.length; i++) {
		if (!charObj[str.charAt(i)]) {
			charObj[str.charAt(i)] = 1
		} else {
			charObj[str.charAt(i)] += 1
		}
	}
	let max = 1
	let maxStr = ''
	for (let key in charObj) {
		if (charObj[key] > max) {
			max = charObj[key]
			maxStr = key
		}
	}
	return [maxStr, max]
}

console.log(most('asdfasdrefcadf asdfasd'))
```

## 数组扁平化

```js
var arr = [[1, 2], [3, [4, 5]], 6]
// 扁平化后
// [1, 2, 3, 4, 5, 6]
```

数组扁平化是将嵌套数组变为一维数组，思路就是遍历数组每个元素，当该元素还是数组时继续遍历，直到不为数组时将其 push 一个新数组中

### 递归实现

```js
function flatten(arr) {
	let ret = [] // 最终返回的数组
	arr.forEach(function(item) {
		if (item instanceof Array) {
		// 判断方法还可以有 Array.isArray()
		// Object.prototype.toString.call(obj) === '[object Array]'
			ret = ret.concat(flatten(item)) // 当元素为数组时递归调用
		} else {
			ret.push(item) // 当元素不为数组直接 push
		}
	})
	return ret
}

console.log(flatten([[1, 2], [3, [4, 5]], 6])) // [1, 2, 3, 4, 5, 6]
```

### reduce

reduce 第一个参数是处理函数，第二个参数是传入处理函数的初始值，数组每次处理数组元素都会调用该处理函数，最终返回处理后的值。

处理函数第一个参数 prev 是每轮处理后的返回值，第二参数 cur 为当前处理元素，第三个参数 index 为当前处理元素位置，第四个参数 arr 为当前处理数组

```js
function flatten(arr) {
	return arr.reduce(function(prev, cur) {
		return prev.concat(cur instanceof Array ? flatten(cur) : cur)
	}, [])
}

console.log(flatten([[1, 2], [3, [4, 5]], 6])) // [1, 2, 3, 4, 5, 6]
```

### es6 扩展运算符

es6 扩展运算符只能扁平一层，可循环判断数组中元素是否存在数组

```js
function flatten(arr) {
	while (arr.some(item => Array.isArray(item))) {
		arr = [].concat(...arr)
	}
	return arr
}

console.log(flatten([[1, 2], [3, [4, 5]], 6])) // [1, 2, 3, 4, 5, 6]
```

## 数组去重

```js
var arr = [1, 1, '1', 2]
// 去重后 [1, '1', 2]
```

### 两层循环

设置标志位为 false，当外层循环的数组元素和内层循环的去重数组元素相等，设置标志位为 true。内层循环结束时标志还为 false，说明之前没有重复元素，数组元素可以添加到去重数组中

另一种方法是利用 js 没有块级作用域，循环变量用 var 声明。同样是两层循环，另一种是当外层循环的数组元素和内层循环的去重数组元素相等跳出当前内存恶棍循环。当 j 等于去重数组长度，说明之前没有重复元素，数组元素可以添加到去重数组中

```js
function unique(arr) {
	let ret = []
	for (let i = 0; i < arr.length; i++) {
		let flag = false
		for (let j = 0; j < ret.length; j++) {
			if (ret[j] === arr[i]) {
				flag = true
				break
			}
		}
		if (!flag) {
			ret.push(arr[i])
		}
	}
	return ret
}

function unique(arr) {
	let ret = []
	for (var i = 0; i < arr.length; i++) {
		for (var j = 0; j < ret.length; j++) {
			if (ret[j] === arr[i]) {
				break
			}
		}
		// 这里 j 是 var 声明，在 for 外可以访问到
		// 如果没有重复元素，j 等于 去重数组长度
		if (j === ret.length) {
			ret.push(arr[i])
		}
	}
	return ret
}

console.log(unique([1, 1, '1', 2])) // [1, '1', 2]
```

### 数组 indexOf() 方法

利用数组 indexOf 方法

```js
function unique(arr) {
	let ret = []
	for (let i = 0; i < arr.length; i++) {
		if (ret.indexOf(arr[i]) === -1) {
			ret.push(arr[i])
		}
	}
	return ret
}

console.log(unique([1, 1, '1', 2])) // [1, '1', 2]
```

### filter + indexOf()

利用 indexOf 返回找到的第一个元素下标是否等于自身下标

```js
function unique(arr) {
	return arr.filter((item, index, array) => {
		return index === array.indexOf(item)
	})
}

console.log(unique([1, 1, '1', 2])) // [1, '1', 2]
```

### 两层循环 + 使用 splice

外层循环下标 i 从 0 到倒数第二个元素（没必要到最后一个），内层循环下标 j 从 i + 1 （i之前的保证已经没有重复）到最后一个元素。删除元素后个数少 1，内层循环下标 j 减 1

```js
function unique(arr) {
	for (let i = 0; i < arr.length - 1; i++) {
		for (let j = i + 1; j < arr.length; j++) {
			if (arr[j] === arr[i]) {
				arr.splice(j--, 1)
				// 删除后元素少了 1，j 应当减1
			}
		}
	}
	return arr
}

console.log(unique([1, 1, '1', 2])) // [1, '1', 2]
```

### es6 Set

Set 不包含重复元素，因此可以使用数组初始化 Set，再将 Set 转化为数组

```js
function unique(arr) {
	// Array.from 将类数组转为数组
	// 或者使用 es6 扩展运算符 [...new Set(arr)]
	return Array.from(new Set(arr))
}

console.log(unique([1, 1, '1', 2])) // [1, '1', 2]
```

## 数组乱序

将数组打乱。思路是遍历数组，通过将当前元素和随机位置元素交换

### Math.random() 结合获取随机位置

```js
function shuffle(arr) {
	let i
	let j
	let tmp
	for (i = arr.length - 1; i >= 0; i--) {
		// Math.random() 生成 0-1 随机数
		// 根据 i 生成随机 j
		j = Math.floor(Math.random() * i)
		tmp = arr[i]
		arr[i] = arr[j]
		arr[j] = tmp
	}
	return arr
}

console.log(shuffle([1, 2, 3, 4, 5]))
```

### es6 数组解构赋值简化

```js
function shuffle(arr) {
	for (let i = arr.length - 1; i >= 0; i--) {
		let j = Math.floor(Math.random() * i);
		[arr[i], arr[j]] = [arr[j], arr[i]];
	}
	return arr
}

console.log(shuffle([1, 2, 3, 4, 5]))
```

## 数组求最值

如求最小值，Math.min(arg1, arg2, arg3, ...) 求一系列参数的最小值，同理 Math.max()
对数组（如元素全是数字）求最值，容易想到的方法是使用 Math.min / Math.max

### 简单的循环比较

```js
function maxAndmin(arr) {
	let min = arr[0]
	let max = arr[0]
	for (let i = 1; i < arr.length; i++) {
		if (arr[i] < min) {
			min = arr[i]
		} else if (arr[i] > max) {
			max = arr[i]
		}
	}
	return {
		min: min,
		max: max
	}
}

console.log(maxAndmin([1, 2, 3, 4, 5, 6, 7])) // {min: 1, max: 7}
```

### reduce

```js
function maxAndmin(arr) {
	let min = arr.reduce(function(prev, cur) {
		return prev > cur ? cur : prev
	})
	let max = arr.reduce(function(prev, cur) {
		return prev < cur ? cur : prev
	})
	return {
		min: min,
		max: max
	}
}

console.log(maxAndmin([1, 2, 3, 4, 5, 6, 7])) // {min: 1, max: 7}
```

### sort() 排序后取出头尾元素

sort() 传入比较函数，升序或者降序后取出头尾元素

```js
function maxAndmin(arr) {
	arr.sort(function(a, b) {
		return a - b // 升序
	})
	return {
		min: arr[0],
		max: arr[arr.length - 1]
	}
}

console.log(maxAndmin([1, 2, 3, 4, 5, 6, 7])) // {min: 1, max: 7}
```

### apply

```js
function maxAndmin(arr) {
	let min = Math.min.apply(null, arr)
	let max = Math.max.apply(null, arr)
	return {
		min: min,
		max: max
	}
}

console.log(maxAndmin([1, 2, 3, 4, 5, 6, 7])) // {min: 1, max: 7}
```

### es6 扩展运算符

```js
function maxAndmin(arr) {
	let min = Math.min(...arr)
	let max = Math.max(...arr)
	return {
		min: min,
		max: max
	}
}

console.log(maxAndmin([1, 2, 3, 4, 5, 6, 7])) // {min: 1, max: 7}
```

## 防抖

防抖的原理是触发事件后延迟一段时间后才执行函数，如果这段时间内事件再次被触发，则以新触发的时间为标准，再延迟一段时间事件后再执行

### 简单实现

```js
function debounce(fn, wait) {
	let timer
	return () => {
		let context = this
		let args = argument
		timer && clearrout(timer)
		timer = setTimeout(() => {
			fn.apply(context, args)
		}, wait)
	}
}
```

```js
// 绑定的回调
// 用户 scroll 事件被调用的函数
function fn() {
	console.log('scroll')
}

window.addEventListener('scroll', debounce(fn, 2000))
```

- 为 scroll 事件绑定处理函数，debounce 函数会立即执行，实际上给 scroll 事件绑定的是 debounce 函数内部返回的函数
- 每次事件触发都会清除当前 time 并重新计算超时调用
- 只有当高频事件停止，最后一次事件触发的超时调用才会在指定时间后执行

### 模仿 underscore 实现

如果不希望非要等到事件停止触发才执行，希望立即执行函数，等到停止触发 n 时间后，才可以重新触发，可以增加 immediate 参数来设置是否要立即执行

```js
function debounce(func, wait, immediate) {
	var timeout, result
	var debounced = function () {
		var context = this
		var args = arguments

		if (timeout) clearTimeout(timeout)
		if (immediate) {
			// 如果已经执行过，不在执行
			var callNow = !timeout
			// 每次都重新设置 timer，保证每次执行至少 wait 时间后才可以执行
			timeout = setTimeout(function() {
				timeout = null
			}, wait)
			// 立即执行
			if (callNow) result = func.apply(context, args)	
		} else {
			timeout = setTimeout(function() {
				func.apply(context, args)
			}, wait)
		}
		return result
	}

	debounced.cancel = function() {
		clearTimeout(timeout)
		timeout = null
	}

	return debounced
}
```

## 节流

节流的原理是**允许函数在规定时间内只执行一次**。不同于防抖，**节流函数不管事件触发多频繁，都保证在规定时间内只真正执行一次事件处理函数**。可以使用 时间戳 或 定时器 实现，也可结合两者实现

### 时间戳实现

当高频事件触发，第一次应该会立即执行（给事件绑定函数与真正触发事件的时间间隔如果大于指定时间），而后再怎么触发事件，也是在指定时间后才执行一次。

```js
function throttle(func, wait）{
	let prev = Date.now()

	return function () {
		let context = this
		let args = arguments
		let now = Date.now()
		if (now - prev > wait) {
			fn.apply(contex, args)
			prev = Date.now()
		}
	}
}
```

### 定时器实现

当触发事件时，设置定时器，当再次触发事件，如果定时器存在就不执行，直到指定时间后，定时器执行函数，清空定时器才可以设置下一个定时器

因此当第一次触发事件，不会立即执行函数，而是在指定时间后执行；之后每次触发事件，都会在指定事件后执行一次；最后一次触发事件后，由于定时器延时，可能还会执行一次函数

```js
function throttle(func, wait）{
	let timer = null

	return function () {
		let context = this
		let args = arguments
		if (!timer) {
			timer = setTimeout(() => {
				fn.apply(context,args)
				timer = null
			}, wait)
		}
	}
}
```

### 模仿 underscore 实现

```js
function throttle(func, wait, options) {
	var timeout, context, args, result
	var previous = 0
	if (!options) options = {}

	var later = function() {
		previous = options.leading === false ? 0 : new Date().getTime()
		timeout = null
		func.apply(context, args)
		if (!timeout) context = args = null
	}

	var throttled = function() {
		var now = new Date().getTime()
		if (!preview && options.leading === false) previous = now
		var remaining = wait - (now - previous)
		context = this
		args = arguments
		if (remaining <= 0 || remaining > wait) {
			if (timeout) {
				clearTimeout(timeout)
				timeout = null
			}
			previous = now
			func.apply(context, args)
			if (!timeout) context = args = null
		} else if (!time && options.trailing !== false) {
			timeout = setTimeout(later, remaining)
		}
	}
	return throttled
}
```

## 防抖和节流

- 防抖：**将几次操作合并为一次操作**。原理是维护一个定时器，在规定时间后执行函数。**但在规定时间内触发事件会清空定时器而重新设置定时器重新计时，因此只有最后一次事件触发才会执行**

- 节流：**规定时间只执行一次函数。和防抖不同，不论事件触发多频繁，规定时间内只真正执行一次事件处理函数，防抖则是在最后一次事件后才执行一次函数** 原理是通过判断是否到达一定时间来执行函数，如果没有达到指定时间，使用定时器延后，而到达指定时间后会重新设定定时器

## dom 查找

### document.getElementById()

- 递归

```js
function getElementById(node, id) {
	if (!node) return null
	if (node.id === id) return node
	if (node.childNodes.length > 0) {
		let children = node.childNodes
		for (let i = 0; i < children.length; i++) {
			let found = getElementById(children[i], id)
			if (found) return found
		}
	}
	return null
}

let node = getElementById(document, 'test')
```

- 非递归

```js
function getElementById(node, id) {
	// 遍历所有 node
	while(node) {
		if (node.id === id) return node
		node = nextElement(node)
	}
	return null
}
function nextElement(node) {
	// 首先找第一个子节点
	if (node.children.length) return node.children[0]
	// 没有找紧跟的兄弟节点
	if (node.nextElementSibling) return node.nextElementSibling
	// 没有看父节点是否存在
	while(node.parentNode) {
		// 父节点是否有兄弟节点
		if (node.parentNode.nextElementSibling) return node.parentNode.nextElementSibling
		node = node.parentNode
	}
	return null
}
let node = getElementById(document, 'charset')
```

### 多种选择器