---
title: js 数组方法
tag: 
	- 数组
---

js 提供了很多数组操作方法，有些会影响原来的数组，有些则不会，而是返回操作后的新数组

## 新增：影响原数组

**Array.prototype.push() 从尾部新增元素，返回数组长度**
**Array.prototype.unshift() 从头部新增元素, 返回数组长度**

```javascript
let add = ['a', 'b', 'c', 'd', 'e']
add.push('f') // 6 ['a', 'b', 'c', 'd', 'e', 'f']
add.unshift('z') // 7 ['z', 'a', 'b', 'c', 'd', 'e' 'f']
```

<!--more-->

**Array.prototype.splice(index, 0, value) 也可以新增元素，index为开始新增的下标，value为新增的元素，可以多个元素, 返回一个空数组**

```javascript
let add = ['a', 'b', 'c', 'd', 'e']
const add1 = add.splice(0, 0, 3, 4) 
console.log(add) // [3, 4, 'a', 'b', 'c', 'd', 'e']
console.log(add1) // []
```

## 新增：不影响原数组

**Array.prototype.concat() 多个数组或元素合并，返回一个新数组**

concat 方法传入参数数组会把数组元素取出合并到新数组，但是数组中的嵌套数组不会再取出

```javascript
const arr1 = ['a', 'b', 'c']
const arr2 = arr1.concat('d') // ['a', 'b', 'c', 'd']
console.log(arr1) // ['a', 'b', 'c']

console.log(arr1.concat('d', [1, 2], [3, [4, 5]])) // ['a', 'b', 'c', 'd', 1, 2, 3, [4, 5]]
```

**es6 展开运算符 ...**

```javascript
const arr1 = ['a', 'b', 'c']
const arr2 = [...arr1, 'd'] // ['a', 'b', 'c', 'd'] 
console.log(arr1); // ['a', 'b', 'c']
```

## 移除：影响原数组

**Array.prototype.pop() 从尾部移除元素,返回被移除的元素**
**Array.prototype.shift() 从头部移除元素,返回被移除的元素**

```javascript
let del = ['a', 'b', 'c', 'd', 'e']
const del1 = del.pop()
console.log(del) // ['a', 'b', 'c', 'd']
console.log(del1) // 'e'
const del2 = del.shift()
console.log(del) // ['b', 'c', 'd']
console.log(del2) // 'a'
```

**Array.prototype.splice(index, number) 也可以移除元素，同样会返回被移除的元素，index为开始移除的下标，number为移除元素的个数**

```javascript
let del = ['a', 'b', 'c', 'd', 'e']
const del1 = del.splice(0, 3)
console.log(del) // ['d', 'e']
console.log(del1) // ['a', 'b', 'c']
```

## 移除：不影响原数组

**Array.prototype.filter() 基于原数组返回一个满足匹配条件的新数组**

```javascript
let del = ['a', 'b', 'c', 'd', 'e']
const del1 = del.filter(d => d !== 'e') 
// const del1 = del.filter(d => { return d !== 'e' })
console.log(del) // ['a', 'b', 'c', 'd', 'e']
console.log(del1) // ['a', 'b', 'c', 'd']
```

**Array.prototype.slice() 返回原数组的一个副本**
Array.prototype.slice(start, end) start 为开始小标，end为结束下标（不包括），参数为负数，值等于数组长度加上参数值

```javascript
let del = ['a', 'b', 'c', 'd', 'e']
const del1 = del.slice()
const del2 = del.slice(1, 2)
console.log(del) // ['a', 'b', 'c', 'd', 'e']
console.log(del1) // ['a', 'b', 'c', 'd', 'e']
console.log(del2) // ['b']

const del3 = del.slice(-3, -1)
console.log(del) // ['a', 'b', 'c', 'd', 'e']
console.log(del3) // ['c', 'd']
```

**Array.prototype.find() 返回数组中找到的第一个匹配元素，未找到返回undefined，可传入比较函数, 不影响原数组；es6 数组的新方法**

```javascript
let arr = ['a', 'b', 'd', 'c', 'e']
const item = arr.find(a => {
    return a > 'a'
})
console.log(item) // 'b'
console.log(arr) // ['a', 'b', 'c', 'd', 'e']
```

## 替换： 影响原数组

**Array.prototype.splice(index, number, value) 也可以替换元素，同样会返回被移除的元素，index为开始替换的下标，number为替换元素的个数，value为替换的元素，可为多个, 同样会返回被替换的元素**

```javascript
let rep = ['a', 'b', 'c', 'd', 'e']
const rep1 = rep.splice(0, 3, 30, 20) 
console.log(rep) // [30, 20, 'd', 'e']
console.log(rep1) // ['a', 'b', 'c']
```

## 替换： 不影响原数组

**Array.prototype.map() 会创建一个新数组，遍历每个元素，根据条件替换转换，不会污染原数据**

```javascript
let rep = ['a', 'b', 'c', 'd', 'e']
const rep1 = rep.map(r => {
    return r + r
})
console.log(rep) // ['a', 'b', 'c', 'd', 'e']
console.log(rep1) // ['aa', 'bb', 'cc', 'dd', 'ee']
```

## 其他方法

**Array.prototype.reverse() 颠倒数组，影响原数组**

```javascript
let arr = ['a', 'b', 'c', 'd', 'e']
arr.reverse()
console.log(arr) // ['e', 'd', 'c', 'b', 'a']
```

**Array.prototype.sort() 数组排序，可以传入比较函数，影响原数组**

Array.prototype.sort() 默认调用每个数组元素的 toString 方法，比较得到的字符串

```javascript
let arr = [0, 2, 1, 10]
arr.sort()
console.log(arr) // [0, 1, 10, 2]

// 升序
arr.sort(function(a, b) {
    return a - b
}) // [0, 1, 2, 10]

// 等同于
arr.sort(function(a, b) {
    if (a > b) {
        // 第一个参数应该位于第二个参数之后，返回正数
        return 1
    } else if (a < b) {
        // 第一个参数应该位于第二个参数之前 返回负数
        return -1
    } else {
        return 0
    }
})
```

**Array.prototype.indexOf() 返回数组中找到的第一个元素的位置，未找到返回-1；IE低版本浏览器不支持**

```javascript
let arr = ['a', 'b', 'd', 'c', 'e']
const index = arr.indexOf('a')
console.log(index) // 0
```

**Array.prototype.findIndex() 返回数组中找到的匹配元素的位置，未找到返回-1，可传入比较函数；es6 数组的新方法**

```javascript
let arr = ['a', 'b', 'd', 'c', 'e']
const index = arr.findIndex(a => {
    return a === 'a'
})
console.log(index) // 0
```

**Array.prototype.some() 返回布尔值，判断数组中是否有元素满足比较函数条件；es6 数组的新方法**

```javascript
let arr = ['a', 'b', 'd', 'c', 'e']
const flag = arr.some(a => {
    return a > 'a'
})
console.log(flag) // true
```

**Array.prototype.every() 返回布尔值，判断数组中是否每个元素都满足比较函数条件；es6 数组的新方法**

```javascript
let arr = ['a', 'b', 'd', 'c', 'e']
const flag = arr.every(a => {
    return a > 'a'
})
console.log(flag) // false
```

**Array.prototype.reduce(function(previousValue, currentValue, index, array){}, [initialValue]) 接受一个函数作为累加器，数组中的每个值（从左到右合并），最后变为一个值, 不影响原数组**

第一个参数是执行数组每个元素的回调callback，包含4个参数：

- previousValue: 上一次调用回调的返回值 或 初始 initialValue

- currentValue: 数组中当前处理的元素

- index: 数组中当前处理元素的下标

- array: 调用reduce方法的数组

第二个参数 initialValue 是首次调用callback的第一个参数，若不传入，回调首次执行的previousValue为数组的第一个元素，否则previous取initialValue

```javascript
// 取最大值（或最小值）
let arr = [23, 45, 2, 30, 5]
let max = arr.reduce(function(pre, cur){
    return pre > cur ? pre : cur
})
console.log(max) // 45

// 数组扁平化
let arr2 = [[1, 2, 3],[6, 7, 8],[9]]
let newArr = arr2.reduce(function(pre, cur){
    return pre.concat(cur)
})
console.log(newArr) // [1, 2, 3, 4, 5, 6, 7, 8, 9]
console.log(arr2) // [[1, 2, 3],[6, 7, 8],[9]]
```

**Array.prototype.reduceRight() 类似Array.prototype.reduce，合并是从右到左**

## vue 数组更新检测

vue 中包含一组观察数组更新的方法，会触发视图更新

- **push()**
- **pop()**
- **shift()**
- **unshift()**
- **spilce()**
- **sort()**
- **reverse()**

注意： **单纯 设数组的length 或 将数组设为 [], 不能触发视图更新**