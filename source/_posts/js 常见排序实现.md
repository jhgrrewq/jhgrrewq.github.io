---
title: js 常见排序实现
tag: 
	- js
---

> 参考: [十大经典排序算法总结（JavaScript描述）
](https://juejin.im/post/57dcd394a22b9d00610c5ec8)

## 算法时间复杂度(一个算法执行耗费的时间)

### 常见的算法时间复杂度

`O(1)＜O(log2n)＜O(n)＜O(nlog2n)＜O(n^2)＜O(n^3)＜…＜O(2^n)＜O(n!)`

### 求解步骤

- **用 O 记号表示算法的时间性能**

将基本语句执行次数的数量级放入大 O 记号中

- **找出算法中的基本语句**

算法中**执行次数最多**的那条语句就是基本语句。如果算法中包含嵌套的循环，则基本语句通常是**最内层的循环体**，如果算法中包含并列的循环，则将并列循环的时间复杂度相加

- **计算基本语句的执行次数的数量级**

**只需计算基本语句执行次数的数量级**，这就意味着只要保证基本语句执行次数的函数中的**最高次幂**正确即可，可以忽略所有低次幂和最高次幂的系数

<!-- more -->

### 简单的程序分析法则

- 对于一些简单的**输入输出语句或赋值语句**,近似认为需要`O(1)`时间  表示基本语句的执行次数是一个常数，一般来说，只要算法中**不存在循环语句，其时间复杂度就是`O(1)`**
- 对于**顺序结构**,需要依次执行一系列语句所用的时间可采用大 O 下 **"求和法则"** 求和法则:是指若算法的2个部分时间复杂度分别为 `T1(n)=O(f(n))` 和 `T2(n)=O(g(n))`,则 `T1(n)+T2(n)=O(max(f(n),g(n)))`。特别地,若 `T1(m)=O(f(m))`, `T2(n)=O(g(n))`,则 `T1(m)+T2(n)=O(f(m)+g(n))`
- 对于**选择结构**,如 if 语句,它的主要时间耗费是在执行 then 字句或 else 字句所用的时间,需注意的是检验条件也需要 `O(1)` 时间
- 对于**循环结构**,循环语句的运行时间主要体现在**多次迭代中执行循环体**以及检验循环条件的时间耗费,一般可用大 O 下 **"乘法法则"**: 是指若算法的2个部分时间复杂度分别为 `T1(n)=O(f(n))` 和 `T2(n)=O(g(n))`, 则 `T1*T2=O(f(n)*g(n))`
- 对于复杂的算法,可分成几个容易估算的部分,然后利用求和法则和乘法法则技术整个算法的时间复杂度
- 还有以下2个运算法则:(1) 若 `g(n)=O(f(n))`,则 `O(f(n))+O(g(n))=O(f(n))`；(2) `O(Cf(n)) = O(f(n))`,其中 `C` 是一个正常数。

## 冒泡排序 （ 平均 O(n^2)）

### 原理

- 重复走访要排序的数列，一次比较两个元素，如果顺序错误就将这两个元素进行交换，直到没有需要交换的元素

```js
function bubbleSort(arr) {
  let len = arr.length
  for (let i = 0; i < len; i++) {
    // 外层表示轮次
    for (let j = 0; j < len - 1 - i; j++) {
      // 本轮次两两比较
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]] // es6 解构赋值
        // var temp = arr[j + 1]
        // arr[j + 1] = arr[j]
        // arr[j] = temp
      }
    }
  }
  return arr
}
var arr=[3,44,38,5,47,15,36,26,27,2,46,4,19,50,48];
console.log(bubbleSort(arr)) // [2, 3, 4, 5, 15, 19, 26, 27, 36, 38, 44, 46, 47, 48, 50]
```

改进

- 设置标志位 pos 用于记录每趟排序最后一次排序交换的位置，由于 pos 之后的元素已经交换到位，因此下一趟排序只要扫描到 pos 位置

```js
function bubbleSort2(arr) {
  let i = arr.length - 1
  while(i > 0) {
    // 每趟初始标志位 为 0，无记录交换
    let pos = 0
    for (let j = 0; j < i; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]]
        pos = j // 记录交换位置
      }
    }
    i = pos // 为下一趟排序准备
  }
  return arr
}
var arr=[3,44,38,5,47,15,36,26,27,2,46,4,19,50,48];
console.log(bubbleSort2(arr)) // [2, 3, 4, 5, 15, 19, 26, 27, 36, 38, 44, 46, 47, 48, 50]
```

## 选择排序 (总是 O(n^2))

### 原理

- 序列分为有序区和无序区，每次从剩下的无序的序列中找到最大（小）的元素，放在无序序列的起始位置

```js
function selectionSort(arr) {
  let len = arr.length
  let minIndex
  for (let i = 0; i < len - 1; i++) {
    // 无序
    minIndex = i // 设置默认无序序列的最小值为该趟第一个元素
    for (let j = i + 1; j < len; j++) {
      if (arr[j] < arr[minIndex]) { // 寻找最小值
        // 记录最小值索引
        minIndex = j
      }
    }
    [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]]
  }
  return arr
}
var arr=[3,44,38,5,47,15,36,26,27,2,46,4,19,50,48];
console.log(selectionSort(arr)) // [2, 3, 4, 5, 15, 19, 26, 27, 36, 38, 44, 46, 47, 48, 50]
```

## 插入排序（平均 O(n^2)）

### 原理

- 类似摸牌时整理牌，第一个元素认为是已经排好序的，取下一个元素，在已经排好序的序列中从后向前扫描，直到找到已经排序的元素小于或者等于新元素的位置，将新元素插入到该位置

```js
function insertionSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    // 默认第一个元素是已经排序号的，轮次从第二个元素开始
    // arr[i] 作为进行排序的元素，和之前已经排序元素从后面向前一一比较
    let key = arr[i]
    // j 为已经排好序的最后一个元素的下标
    let j = i - 1
    while(j >= 0 && arr[j] > key) {
      // 该元素向后挪，新元素继续和前一位元素比较
      arr[j + 1] = arr[j]
      j--
    } 
    // 无序排序，直接将 key 放入已经排好序的序列
    arr[j + 1] = key
  }
  return arr
}
var arr=[3,44,38,5,47,15,36,26,27,2,46,4,19,50,48];
console.log(insertionSort(arr)) // [2, 3, 4, 5, 15, 19, 26, 27, 36, 38, 44, 46, 47, 48, 50]
```

## 归并排序

### 原理

- 对长度为 n 的序列分为两个长度为 n/2 的子序列，对这两个子序列分别再用归并排序，再将排序号的子序列合并成一个最终的排序序列

```js
function mergeSort(arr) { // 自上而下递归
  var len = arr.length
  if (len < 2) return arr
  // 将长度为 n 的序列分为两个子序列
  var mid = Math.floor(len / 2),
      left = arr.slice(0, mid),
      right = arr.slice(mid)
  // 将两个子序列分别递归调用归并排序，最后进行合并
  return merge(mergeSort(left), mergeSort(right))
}
function merge(left, right) {
  var result = []
  while(left.length && right.length) {
    // 分别取出序列中的第一个元素比较然后放入数组
    if (left[0] <= right[0]) {
      result.push(left.shift())
    } else {
      result.push(right.shift())
    }
  }
  while(left.length)
    result.push(left.shift())
  while(right.length)
    result.push(right.shift())
  return result
}
var arr=[3,44,38,5,47,15,36,26,27,2,46,4,19,50,48];
console.log(mergeSort(arr))
```

## 快速排序

### 原理

- 从序列中提出一个元素作为基准，重新排列将所有比基准元素小的放在基准前面，大的放在基准后面，相同的元素可以放在任意一边

```js
function quickSort(arr) {
  if (arr.length <= 1) return arr
  var pivotIndex = Math.floor(arr.length / 2)
  // 将基准元素从原数组删除取出，注意 splice 返回删除元素为数组
  var pivot = arr.splice(pivotIndex, 1)[0]
  var left = []
  var right = []
  // 小于基准的放到左边，大于基准的放到右边
  for (var i = 0; i < arr.length; i++) {
    if (arr[i] < pivot) {
      left.push(arr[i])
    } else {
      right.push(arr[i])
    }
  }
  // 递归处理并合并
  return [...quickSort(left), pivot, ...quickSort(right)]
}
var arr=[3,44,38,5,47,15,36,26,27,2,46,4,19,50,48];
console.log(quickSort(arr)) //[2, 3, 4, 5, 15, 19, 26, 27, 36, 38, 44, 46, 47, 48, 50]

```