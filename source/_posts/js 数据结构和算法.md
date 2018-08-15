---
title: js 数据结构和算法
tag: 
	- 数据结构
---

> 参考: [javaScript的数据结构与算法（一）——栈和队列](https://github.com/wengjq/Blog/issues/4)、[javaScript的数据结构与算法（二）——链表](https://github.com/wengjq/Blog/issues/4)、[javaScript的数据结构与算法（一）——栈和队列](https://github.com/wengjq/Blog/issues/5)、[javaScript的数据结构与算法（三）——集合 数据结构与算法](https://github.com/wengjq/Blog/issues/6)、[javaScript的数据结构与算法（四）——字典和散列表](https://github.com/wengjq/Blog/issues/7)、[javaScript的数据结构与算法（五）——树](https://github.com/wengjq/Blog/issues/8)、[javaScript的数据结构与算法（六）——图](https://github.com/wengjq/Blog/issues/9)

<!-- more -->

## 栈 stack

栈是一种遵循 **后进先出(LIFO)** 原则的 **有序集合**。新添加和待删除的元素保存在栈尾

### 栈的创建

栈的构造函数对数组的一些方法进行封装

- 内部使用 `数组` 保存栈数据
- `push(element)` 添加一个或多个元素到栈顶
- `pop()` 移除栈顶元素，同时返回被移除元素
- `peek()` 返回栈顶元素，但不对栈顶元素做出任何修改
- `isEmpty()` 检查栈内是否有元素，有返回 true，否则返回 false
- `clear()` 清除栈内元素
- `size()` 返回栈的元素个数
- `print()` 打印栈的元素

```js
class Stack {
  constructor() {
    this.items = []
  }
  push() {
    this.items.push(...arguments)
  }
  pop() {
    return this.items.pop()
  }
  peek() {
    return this.items[this.items.length - 1]
  }
  isEmpty() {
    return this.items.length === 0
  }
  clear() {
    // this.items.length = 0
    this.items = []
  }
  size() {
    return this.items.length
  }
  print() {
    console.log(this.items.toString())
  }
}
```

### 栈的应用

判断字符串回文

```js
function isPalindrome(str) {
  let stack = new Stack()
  for (let i = 0; i < str.length; i++) {
    stack.push(str[i])
  }
  let newStr = ''
  while(stack.size() > 0) {
    newStr += stack.pop()
  }
  return str === newStr
}
```

十进制转其他进制 (除进制数取余，逆序)

```js
function multiBase(num, base) {
  let s = new Stack()
  do {
    s.push(num % base)
    num = Math.floor(num /= base)
  } while (num > 0)
  let str = ''
  while(s.size() > 0) {
    str += s.pop()
  }
  return str
}
```

## 队列 queue

队列是一种遵循 **先进先出(LILO)** 原则的 **有序集合**。队列从尾部添加新元素，从顶部移除元素，最先添加的元素必须排在队列的末尾

### 队列的创建

队列的构造函数对数组的一个方法进行封装

- 内部使用 `数组` 保存队列数据
- `enqueue(element)` 向队列尾部添加一个或多个元素
- `dequeue()` 移除队列的第一个元素，同时返回被移除元素
- `front()` 返回队列的第一个元素（最先被添加也是最先被移除的），队列不做任何改动
- `isEmpty()` 检查队列内是否有元素，有返回 true，否则返回 false
- `clear()` 清除队列内元素
- `size()` 返回队列的长度
- `print()` 打印队列的元素

```js
class Queue {
  constructor() {
    this.items = []
  }
  enqueue() {
    this.items.push(...arguments)
  }
  dequeue() {
    return this.items.shift()
  }
  front() {
    return this.items[0]
  }
  isEmpty() {
    return this.items.length === 0
  }
  clear() {
    // this.items.length = 0
    this.items = []
  }
  size() {
    return this.items.length
  }
  print() {
    console.log(this.items.toString())
  }
}
```

## 链表 LinkedList

在内存中，栈和队列如果需要对内部某个元素进行移除或者添加需要对整个数据进行操作，链表每个元素都由元素本身的数据和指向下一个元素的指针构成，因此**添加或者移除某个元素不需要对链表整体进行操作，只需要改变相关元素的指针指向**

### 链表的创建

- 内部需要一种数据结构保存链表数据

```js
// Node 类表示要添加的元素，有 2 个属性，一个是 element 表示添加到链表的具体值，一个是 next, 表示链表中指向下一个元素的指针
function Node(element) {
  this.element = element
  this.next = null
}
```

- `append(element)` 向链表尾部添加一个新元素
- `insert(position, element)` 向链表特定位置插入元素
- `remove(element)` 从链表中移除一项
- `indexOf(element)` 返回链表中某元素的索引，如果没有返回 -1
- `removeAt(position)` 从特定位置移除一项
- `isEmpty()` 判断链表是否为空，若为空返回 true，否则为 false
- `size()` 返回链表包含的元素个数
- `toString()` 重写继承 Object 类的 toString() 方法

```js
// Node 类声明, 两个属性，element 表示添加到链表的具体指，next 表示指向链表中下一个元素的指针
function Node(element) {
  this.element = element
  this.next = null
}

function LinkedList() {
  // 初始化链表长度
  this.length = 0
  // 初始化第一个元素
  this.head = null
}

LinkedList.prototype.append = function(element) {
  // 初始化添加的 Node 实例
  let node = new Node(element), 
      current

  if (!this.head) {
    // 说明是插入第一个元素
    this.head = node
  } else {
    current = this.head
    // 循环链表找到最后一项，循环结束 current 指向链表的最后一个元素
    while(current.next) {
      current = current.next
    }
    // 找到最后一个元素，将其 next 属性指向新元素 node
    current.next = node
  }
  // 更新链表长度
  this.length++
}

LinkedList.prototype.insert = function(pos, element) {
  // 检查是否越界，超过链表长度或小于 0 
  if (pos > 0 || pos <= this.length) {
    let node = new Node(element),
        current = this.head,
        previous,
        index = 0
    if (pos === 0) {
      // 在第一个位置添加
      node.next = current
      this.head = node
    } else {
      // 循环链表，找到正确位置，循环完毕，previous，current 分别是被添加元素的前一个和后一个元素
      while(pos > index) {
        previous = current
        current = current.next
        index++
      }
      node.next = current
      previous.next = node
    }
    // 更新链表长度
    this.length++
    return true
  } else {
    return false
  }
}

LinkedList.prototype.removeAt = function(pos) {
  // 检查是否越界，超过链表长度或小于 0 
  if (pos > 0 || pos <= this.length) {
    let current = this.head,
        previous,
        index = 0
    if (pos === 0) {
      // 移除第一个元素 
      this.head = current.next
    } else {
      // 循环链表，找到正确位置，循环完毕，previous，current 分别是被添加元素的前一个和后一个元素
      while(pos > index) {
        previous = current
        current = current.next
        index++
      }
      previous.next = current.next
    }
    // 更新链表长度
    this.length--
    // 返回移除元素的数据
    return current.element
  } else {
    return null
  }
}

LinkedList.prototype.indexOf = function(element) {
  let current = this.head,
      index = 0
  // 循环找到链表的位置
  while(current) {
    if (current.element === element) {
      return index
    }
    index++
    current = current.next
  }
  return -1
}

LinkedList.prototype.remove = function(element) {
  // 调用已经声明的 indexOf 和 removeAt 方法
  let index = this.indexOf(element)
  return this.removeAt(index)
}

LinkedList.prototype.isEmpty = function() {
  return this.length === 0
}

LinkedList.prototype.size = function() {
  return this.length
}

LinkedList.prototype.getHead = function() {
  return this.head
}

LinkedList.prototype.toString = function() {
  let current = this.head,
      string = ''
  while(current) {
    string += current.element + (current.next ? ',' : '') 
    current = current.next
  }
  return string
}

LinkedList.prototype.print = function() {
  console.log(this.toString())
}
```

### 双向链表

双向链表比起单项链表就是 Node 类多了一个 prev 属性，也就是每个 node 不仅有一个指向后面元素的指针，还有一个指向前面的指针

### 循环链表

循环链表就是整个链表变成一个环。单向链表最后一个元素的 next 属性 为 null，现在将它指向第一个元素（head）就变成单向循环链表；双向链表最后一个元素的 next 属性为 null，现在将它指向第一个元素（head）就变成双向循环链表

## 集合 set

栈、队列和链表都是线性有序结构，集合是具有某种特定性质的具体或者抽象的对象汇成的集体，这些对象成为集合的元素

集合具有：

- `确定性` 给定一个结合，任给一个元素，该元素或者属于或者不属于这个集合
- `互异性` 一个集合中，任意两个元素被认为是不相同的，每个元素只能出现一次
- `无序性` 一个集合中元素之间是无序的。当然集合上可以定义序关系，定义序关系后，元素之间可以按照序关系排列

集合常见的几种操作：

- 并集
- 交集
- 差集

### 集合的创建

**集合是以 [值，值] 形式存储数据，就是对 js 对象的属性方法进行封装，只不过键值都是一样的值**

- 内部使用 `对象` 保存集合数据
- `add(value)` 向集合添加一个新的项
- `delete(value)` 从集合中移除一个值
- `has(value)` 如果值在集合中，返回 true，否则返回 false
- `clear()` 清除集合中所有项
- `size()` 返回集合所包含的元素数量
- `values()` 返回集合所包含的值的数组
- `union(setName)` 并集，返回两个集合所有元素的集合（元素不重复）
- `intersection(setName)` 交集，返回两个集合共有元素的集合
- `difference(setName)` 差集，返回包含所有存在本集合而不存在 setName 集合的元素的新集合
- `subset(setName)` 子集，验证 setName 是否是本集合的子集

```js
// 集合是以 [值，值] 形式存储数据，内部实现使用 对象 存储集合数据
function Set() {
  this.items = {}
}

Set.prototype.add = function(value) {
  if (!this.has(value)) {
    this.items[value] = value
    return true
  }
  return false
}

Set.prototype.delete = function(value) {
  if (this.has(value)) {
    delete this.items[value]
    return true
  }
  return false
}

Set.prototype.has = function(value) {
  // 判断是否是自身属性而不是原型链上属性
  return this.items.hasOwnProperty(value)
}

Set.prototype.clear = function() {
  this.items = {}
}

Set.prototype.getItems = function() {
  return this.items
}

Set.prototype.size = function() {
  // 属性个数
  return Object.keys(this.items).length
}

Set.prototype.sizeLegacy = function() {
  // 浏览器兼容 size
  let count = 0
  for(let key in this.items) {
    if (this.items.hasOwnProperty(key)) {
      count++
    }
  }
  return count
}

Set.prototype.values = function() {
  // 取出值作为一个数组
  let values = []
  let i = 0, keys = Object.keys[this.items]
  for(;i < keys.length; i++) {
    values.push(this.items[i])
  }
  return values
}

Set.prototype.valuesLegacy = function() {
  // 浏览器兼容 values
  let values = []
  for(let key in this.items) {
    if (this.items.hasOwnProperty(key)) {
      values.push(this.items[key])
    }
  }
  return values
}

// 并集
Set.prototype.union = function(otherSet) {
  let unionSet = new Set()
  // 取出所有值
  let values = this.values
  for (let i = 0; i < values.length; i++) {
    unionSet.add(values[i])
  }
  // 取出传入 set 的值
  values = otherSet.values()
  for (let i = 0; i < values.length; i++) {
    unionSet.add(values[i])
  }
  return unionSet
}

// 交集
Set.prototype.intersection = function(otherSet) {
  let intersectionSet = new Set()
  // 取出所有值
  let values = this.values
  for (let i = 0; i < values.length; i++) {
    if (otherSet.has(values[i])) {
      intersectionSet.add(values[i])
    }
  }
  return intersectionSet
}

// 差集
Set.prototype.difference = function(otherSet) {
  let differenceSet = new Set()
  // 取出所有值
  let values = this.values
  for (let i = 0; i < values.length; i++) {
    if (!otherSet.has(values[i])) {
      differenceSet.add(values[i])
    }
  }
  return differenceSet
}

// 子集
Set.prototype.subset = function(otherSet) {
  if (this.size() > otherSet.size()) {
    return false
  } else {
    let values = this.values
    for (let i = 0; i < values.length; i++) {
      if (!otherSet.has(values[i])) {
        return false
      }
    }
    return true
  }
}
```

## 字典（映射） dictionary

字典和集合很相似，**集合以 [值，值] 形式存储元素，字典以 [键，值] 形式存储元素**

### 字典的创建

**字典是以 [键，值] 形式存储数据，就是对 js 对象的属性和方法的封装**

- 内部使用 `对象` 保存字典数据
- `set(key, value)` 向字典添加一个键值对
- `remove(key)` 从字典中移除一个特定 key 的键值对
- `has(key)` 如果 key 在字典中，返回 true，否则返回 false
- `get(key)` 如果 key 在字典中，返回 key 对应值，否则返回 undefined
- `clear()` 清除字典中所有项
- `size()` 返回字典所包含的键值对数量
- `values()` 返回字典所有键值对中值的数组
- `each(fn)` 传入回调，遍历字典执行回调

```js
// 字典是以 [键，值] 形式存储数据，内部实现使用 对象 存储集合数据
function Dictionary() {
  this.items = {}
}

Dictionary.prototype.set = function(key, value) {
  // 本身对象上的 key 就是唯一的
  this.items[key] = value
}

Dictionary.prototype.remove = function(key) {
  if (this.has(key)) {
    delete this.items[key]
    return true
  }
  return false
}

Dictionary.prototype.has = function(key) {
  // 判断是否是自身属性而不是原型链上属性
  return this.items.hasOwnProperty(key)
}

Dictionary.prototype.get = function(key) {
  return this.has(key) ? this.items[key] : undefined
}

Dictionary.prototype.clear = function() {
  this.items = {}
}

Dictionary.prototype.getItems = function() {
  return this.items
}

Dictionary.prototype.size = function() {
  // 属性个数
  return Object.keys(this.items).length
}

Dictionary.prototype.sizeLegacy = function() {
  // 浏览器兼容 size
  let count = 0
  for(let key in this.items) {
    if (this.items.hasOwnProperty(key)) {
      count++
    }
  }
  return count
}

Dictionary.prototype.keys = function() {
  // 输出 key 数组
  return Object.keys(this.items)
}

Dictionary.prototype.keysLegacy = function() {
  // 浏览器兼容 keys
  let keys = []
  for(let key in this.items) {
    if (this.has(key)) {
      keys.push(key)
    }
  }
  return keys
}

Dictionary.prototype.values = function() {
  // 取出值作为一个数组
  let values = []
  let i = 0, keys = Object.keys[this.items]
  for(;i < keys.length; i++) {
    values.push(this.items[i])
  }
  return values
}

Dictionary.prototype.valuesLegacy = function() {
  // 浏览器兼容 values
  let values = []
  for(let key in this.items) {
    if (this.has(key)) {
      values.push(this.items[key])
    }
  }
  return values
}

// each
Dictionary.prototype.each = function(fn) {
  for (let key in this.items) {
    if (this.has(key)) {
      fn(key, this.items[key])
    }
  }
}
```

## 散列表 HashTable

散列表是 Dictionary 类的一种散列实现方式。在之前的数据结构中如果要获取一个值，通常需要遍历整个数据结构。如果使用散列函数，就知道值的具体位置，因此能快速检索到该值。**散列函数的作用是给定一个键值，然后返回值在表中的位置**

**散列函数可能存在将两个键映射为同一个值的可能（碰撞），简单的办法是对映射出的值向前查找**

### 散列表的创建

- 内部使用 `数组` 保存数据
- `hashCode(key)` 传入键，通过散列函数计算键在表中的位置
- `remove(key)` 将散列表 key 对应值设为 undefined
- `put(key, value)` 根据 key 通过散列函数计算键在表中位置，进行映射，如果映射位置被占用，继续尝试下一个位置
- `get(key)` 返回 key 在散列表中对应值

```js
// 散列表基于数组进行设计，使用散列函数，给定一个键值，返回值在表中的位置，能快速检索到该值
// 散列函数将键映射为一个数字（作为唯一的数组索引，是数字的范围是 0-散列表的长度）
// 散列函数可能存在将两个键映射为同一个值的可能，成为碰撞
// 散列表中数组的大小常见的限制是：数组长度应该是一个质数
function HashTable() {
  this.table = []
}

// 散列函数1：使用 ASCII 码值相加并对一个质数取余作为键在表中的位置
function loseHashCode(key, prime = 37) {
  let hash = 0
  for(let i = 0; i < key.length; i++) {
    hash += key.charCodeAt(i)
  }
  return hash % prime
}

// 散列函数2：霍纳算法优化的散列函数，仍然先计算字符串中每个字符的 ASCII 码值，不过求和前每次要乘以一个质数
// 大多数算法书建议使用较小的质数，如 31，37
function betterHashCode(key, prime = 37) {
  let hash = 0
  for(let i = 0; i < key.length; i++) {
    hash = hash * prime + key.charCodeAt(i)
  }
  if (hash < 0) {
    hash += this.table.length - 1
  }
  return parseInt(hash)
}

HashTable.prototype.hashCode = function(key) {
  // 散列函数 给定一个键值，返回值在表中的位置
  // return loseHashCode(key)
  return betterHashCode(key)
}

HashTable.prototype.get = function(key) {
  return this.table[this.hashCode(key)]
}

// 根据所给的 key 散列函数计算出在表中的位置，然后进行映射
// 为避免碰撞，如果映射位置 pos 已经被占用，尝试 pos + 1
HashTable.prototype.put = function(key, value) {
  let pos = this.hashCode(key)
  if (this.table[pos] === undefined) {
    this.table[pos] = value
  } else {
    let index = ++props
    while(this.table[index] !== undefined) {
      index++
    }
    this.table[index] = value
  }
}

HashTable.prototype.remove = function(key) {
  this.table[this.hashCode(key)] = undefined
}

HashTable.prototype.print = function() {
  for (let i = 0; i < this.table.lenght; i++) {
    if (this.table[i] !== undefined) {
      console.log(i + ': ' + this.table[i])
    }
  }
}
```

## 树

树是一种一对多的非线性结构

选择树而非别的数据结构：二叉树上查找较快（链表查找较慢）、二叉树添加/删除较快（数组添加/删除较慢）

- 树中的数据元素叫做节点
- 每个节点有 0 个或多个子节点
- 没有父节点的节点称为根节点
- 每一个非根节点有且只有一个父节点

- `节点的度` 一个节点含有的子树的个数
- `树的度` 一棵树中最大的节点的度
- `叶节点或终端节点` 度为 0 的节点
- `分支节点或非终端节点` 度不为 0 的节点
- `兄弟节点` 具有相同父节点的节点互为兄弟节点
- `层次` 从根节点开始定义，根节点为第 1 层，以此类推
- `节点的祖先` 从根节点到该节点所经分支上的所有节点
- `子孙` 以某节点为根的子树中的任一节点

### 二叉树 （每个节点最多有两个子树的树结构）

二叉树由根节点、左子树和右子树三部分构成

#### 二叉树分类

- `满二叉树` 一棵树深度为 k，并且有 2^k - 1 各节点（除了叶子节点，其他节点都有两个子节点）
- `完全二叉树` 深度为 k, 有 n 个节点的二叉树，当且仅当每个节点都和深度为 k 的满二叉树中，序号为 1 到 n 的节点对应时，称为完全二叉树
- `平衡二叉树/二叉搜索树/二叉排序树` 它是一颗空树或它的左右两个子树的高度差绝对值不超过 1，并且左右两个子树都是一颗平衡二叉树

### 二叉树的遍历

- `先序遍历` 首先访问根节点，然后遍历左子树，最后遍历右子树（**根-左-右**）
- `中序遍历` 首先访问左子树，然后访问根节点，最后遍历右子树（**左-根-右**）
- `后序遍历` 首先遍历左子树，然后遍历右子树，最后访问根节点（**左-右-根**）

### 二叉搜索树的创建

二叉搜索树或者是一颗空树，或者是具有下列性质的二叉树：

- 若它的左子树不为空，**则左子树上所有的节点的值均小于它的根节点的值**
- 若它的右子树不为空，**则右子树上所有的节点的值均大于它的根节点的值**
- 它的左右子树也为二叉搜索树

```js
function Node(key) {
  this.key = key
  this.left = null
  this.right = null
}
```

在双向链表中，每个节点有两个指针，一个指向上一个节点，一个指向下一个节点。类似链表，二叉树同样有两个指针，一个指向左孩子，一个指向右孩子

- `insert(key)` 向树中插入一个新的键(节点)
- `search(key)` 在树中查找一个键，如果节点存在，返回true，否则返回 false
- `inOrder` 通过中序遍历方式遍历所有节点
- `preOrder` 通过先序遍历方式遍历所有的节点
- `postOrder` 通过后序遍历的方式遍历所有的节点
- `min` 返回树中的最小值
- `max` 返回树中的最大值
- `remove(key)` 从树中移除某个键

```js
// 二叉搜索树 BinarySearchTree
function Node(key) {
  this.key = key
  this.left = null
  this.right = null
}

function BST() {
  // 初始化根节点
  this.root = null
}

// 插入节点
function insertNode(node, newNode) {
  // 判断两个节点的大小，根据二叉搜索树的特点，左子树上所有节点的值都小于它根节点，右子树上所有节点的值都大于它的根节点
  if (node.key > newNode) {
    // 放在左子树
    if (node.left === null) {
      node.left = newNode
    } else {
      // 递归调用
      insertNode(node.left, newNode)
    }
  } else {
    if (node.right === null) {
      node.right = newNode
    } else {
      // 递归调用
      insertNode(node.right, newNode)
    }
  }
}

// 搜索该 key 是在存在
function searchNode(node, key) {
  if (node === null) {
    return false
  }
  if (key < node.key) {
    return searchNode(node.left, key)
  } else if (key > node.key) {
    return searchNode(node.right, key)
  } else {
    return true
  }
}

// 查找 key 最小节点, 返回 key
function minNode(node) {
  // 从左子树中找
  if (node) {
    while(node && node.left !== null) {
      node = node.left
    }
    return node.key
  }
  return null
}

function findMinNode(node) {
  // 从左子树中找
  if (node) {
    while(node && node.left !== null) {
      node = node.left
    }
    return node
  }
  return null
}

// 查找 key 最大节点
function maxNode(node) {
  // 从右子树中找
  if (node) {
    while(node && node.right !== null) {
      node = node.right
    }
    return node.key
  }
  return null
}

// 中序遍历
function inOrderNode(node, cb) {
  if (node !== null) {
    inOrderNode(node.left, cb)
    cb(node.key)
    inOrderNode(node.right, cb)
  }
}

// 先序遍历
function preOrderNode(node, cb) {
  if (node !== null) {
    cb(node.key)
    preOrderNode(node.left, cb)
    preOrderNode(node.right, cb)
  }
}

// 后序遍历
function postOrderNode(node, cb) {
  if (node !== null) {
    postOrderNode(node.left, cb)
    postOrderNode(node.right, cb)
    cb(node.key)
  }
}

// 从树中删除 key
// 判断当前节点是否包含该 key，包含则删除节点，不包含则比较当前节点 key 和待删除 key，待删除 key 小于当前节点上的 key，则要移到当前节点的左子节点继续比较；如果删除的 key 大于当前节点上的 key ，则移至当前节点的右子节点继续比较
// 如果待删除节点是叶子节点，将父节点指向它的指针设为 null
// 如果待删除节点是只有一个孩子的节点, 将父节点指向它的指针指向它的子节点 
// 如果待删除节点是两个孩子的节点, 找到删除节点右子树上最小值节点，
function removeNode(node, key) {
  if (node === null) {
    return null
  }
  if (key < node.key) {
    // 从左子树查找，注意
    node.left = removeNode(node.left, key)
    return node
  } else if (key > node.key) {
    node.right = removeNode(node.right, key)
    return node
  } else {
    // 考虑三种特殊情况
    // 1 叶子节点：将父节点指向它的指针设为 null
    // 2 只有一个孩子的节点：将父节点指向它的指针指向它的子节点
    // 3 两个孩子的节点：找到删除节点右子树上最小值节点，
    // case 1
    if (node.left === null && node.right === null) {
      // 直接将节点置空
      return null
    }
    // case 2
    if (node.left === null) {
      // 没有左子树
      return node.right
    } else if (node.right === null) {
      // 没有右子树
      return node.left
    }
    // case 3
    const rightMin = findMinNode(node.right)
    node.key = rightMin.key
    node.right = removeNode(node.right, rightMin.key)
    return node
  }
}

BST.prototype.insert = function(key) {
  let node = new Node(key)

  // 判断是否是第一个节点，如果是作为根节点保存，否则调用 insertNode 方法
  !this.root ? this.root = node : insertNode(this.root, node)
}

BST.prototype.getRoot = function() {
  return this.root
}

BST.prototype.search = function(key) {
  return searchNode(this.root, key)
}

BST.prototype.min = function(key) {
  return minNode(this.root)
}

BST.prototype.max = function(key) {
  return maxNode(this.root)
}

BST.prototype.remove = function(key) {
  // 将根节点转化，注意赋值左边作为调用 removeNode 方法的 根节点
  this.root = removeNode(this.root, key)
}

BST.prototype.inOrder = function(cb) {
  inOrderNode(this.root, cb)
}

BST.prototype.preOrder = function(cb) {
  preOrderNode(this.root, cb)
}

BST.prototype.postOrder = function(cb) {
  postOrderNode(this.root, cb)
}
```

## 算法

### 基于 LocalStorage 设计一个 1M 的缓存系统，需要实现缓存淘汰机制

- 存储的每个对象添加两个属性：过期时间和存储时间
- 利用一个属性保存系统中目前所占空间大小，每次存储都改变该属性。当属性值大于 1M，需要对系统中的数据按照时间先后排序，删除一定量的数据
- 每次取数据，要判断数据是否过期，过期就删除

```js
class Store {
  constructor() {
    let store = localStorage.getItem('cache')
    if (!store) {
      store = {
        maxSize: 1024 * 1024,
        size: 0
      }
      this.store = store
    } else {
      this.store = JSON.parse(store)
    }
  }
  set(key, value, expire) {
    this.store[key] = {
      date: Date.now(),
      expire,
      value
    }
    let size = this.sizeOf(JSON.stringify(this.store[key]))
    if (this.store.size + size > this.store.maxSize) {
      console.log('over maxSize！！！')
      let keys = Object.keys(this.store)
      keys = keys.sort((a, b) => {
        return this.store[b].date - this.store[a].date
      })

      while(this.store.size + size > this.store.maxSize) {
        let index = keys[keys.length -1]
        this.store.size -= this.sizeOf(JSON.stringify(this.store[index]))
      }
    }
    this.store.size += size
    localStorage.setItem('cache', JSON.stringify(this.store))
  }
  get(key) {
    let d = this.store[key]
    if (!d) {
      console.log(`${key} is not exist!`)
      return
    }
    if (d.expire > Date.now()) {
      console.log(`${key} is outDated!`)
      let size = this.sizeOf(JSON.stringify(this.store[key]))
      delete this.store[key]
      this.store.size -= size
      localStorage.setItem('cache', JSON.stringify(this.store))
    } else {
      return d.value
    }
  }
  sizeOf(str, charset) {
    let total = 0, len, i, charcode, charset = charset ? charset.toLowerCase() : ''
    if (charset === 'utf-16' || charset === 'utf16') {
      for (i = 0, len = str.length; i < len; i++) {
        charcode = str.charCodeAt(i)
        if (charcode <= 0xffff) {
          total += 2
        } else {
          total += 4
        }
      }
    } else {
      for (i = 0, len = str.length; i < len; i++) {
        charcode = str.charCodeAt(i)
        if (charcode <= 0x007f) {
          total += 1
        } else if (charcode <= 0x07ff) {
          total += 2
        } if (charcode <= 0xffff) {
          total += 3;
        } else {
          total += 4;
        }
      }
    }
    return total
  }
}
```