---
title: flow
tag: 
  - flow
  - vue
---

> 参考: [Vue 2.0 为什么选用 Flow 进行静态代码检查而不是直接使用 TypeScript](https://www.zhihu.com/question/46397274)、[flow 官网](https://flow.org/)、[flow 的使用](https://zhuanlan.zhihu.com/p/24649359)、[认识 Flow](https://ustbhuangyi.github.io/vue-analysis/prepare/flow.html#%E4%B8%BA%E4%BB%80%E4%B9%88%E7%94%A8-flow)

vue 2.0 源码引入 flow 进行静态代码检查。vue 的作者尤雨溪在知乎的一片文章中解释了使用 flow 而不是 ts 的原因，flow 对于已有的 ES2015 代码的迁入/迁出成本都非常低:

>- 可以一个文件一个文件地迁移，不需要一竿子全弄
- Babel 和 ESLint 都有对应的 Flow 插件以支持语法，可以完全沿用现有的构建配置
- 更贴近 ES 规范。除了 Flow 的类型声明之外，其他都是标准的 ES。万一不想用 Flow 了，用 babel-plugin-transform-flow-strip-types 转一下，就得到符合规范的 ES
- 在需要的地方保留 ES 的灵活性，且对于生成的代码尺寸有更好的控制力 (rollup / 自定义 babel 插件）

<!-- more -->


## 安装

### 安装依赖

项目中将 flow-bin 模块安装并添加到 package.json 的 开发环境依赖

```js
npm i --D flow-bin
```

安装完成后在 package.json 中加入脚本

```js
"scripts": {
  "flow":"flow check"
}
```

### 生成 .flowconfig 并配置

安装依赖后在需要进行检查的目录（一般是项目根目录）运行命令 `flow init`, 会生成 .flowconfig 文件，这个文件会告诉 flow 从这个目录开始检测

.flowconfig 的具体配置可以看[官网介绍](https://flow.org/en/docs/config/), 其中有几个主要字段:

- `[ignore]` 添加忽略检查的目录和文件
- `[include]` 添加需要检查的目录和文件，一般不需要填
- `[libs]` 添加用户自定义的 库定义文件，当 flow 检查时会使用这些目录或文件中的定义进行检查

```js
// vue-router 中的 .flowconfig
[ignore]
.*/node_modules/.*
.*/test/.*
.*/dist/.*
.*/examples/.*
.*/vue/.*

[include]

[libs]
flow

[options]
#unsafe.enable_getters_and_setters=true
suppress_comment= \\(.\\|\n\\)*\\$flow-disable-line
```

## 使用

需要 flow 检查的 js 文件开头需要添加 @flow 注释, 相当于告诉 flow 这个文件需要进行类型检查

```js
// @flow
/* @flow */
```

### flow 的工作方式

通常类型检查分成两种方式

- **类型推断** 通过变量的上下文来推断出变量类型，然后根据这些推断来检查类型
- **类型注释** 事先注释好期待的类型， flow 会基于这些注释来判断

### 类型推断

```js
// @flow
function test(a) {
  return a - 5
}

test('a')
let a = '1'
```

执行 `npm run flow`，会发现 test() 函数中的类型转换出错被标记出来

```js
Cannot perform arithmetic operation because string [1] is not a number.
    1│ /* @flow */
    2│ function test(a) {
    3│   return a - 5
    4│ }
    5│
[1] 6│ test('a')
```

### 类型注释

上面代码中 `let a = '1'` 中的 a 的变量类型可以指定，告诉 flow 这个变量的类型

```js
let a: number = '1'
```

执行 `npm run flow`，会发现 a 变量类型出错被标记出来

```js
Cannot assign '1' to a because string [1] is incompatible with number [2].
         4│ }
         5│
         6│ test(5)
 [2][1]  7│ let a: number = '1'
```

### 自定义类型

flow 还允许我们自定义类型进行检查

- 在一些单独文件中对这些类型进行定义

```js
// flow/declarations.js
declare type MyData = {
  id: number,
  text: string
}
```

- .flowconfig 中 `[libs]` 字段添加自定义类型的 目录和文件

```js
[libs]
flow
```

- 可以这样在 js 文件中使用

```js
var data: MyData = {
  id: 3,
  text: 'data'
}
```

### flow 常见的类型注释

- `数组` 数组类型注释的格式是 `Array<T>`, `T` 表示数组中每一项的数据类型

```js
/*@flow*/
// 这里表示 arr 是每一项都是数字的数组
var arr: Array<number> = [1, 2, 3]
arr.push('Hello')
```

- `类和对象` 类的类型注释如下，可以对类自身的属性做类型检查，也可以对构造函数的参数进行类型检查，**属性中多个类型用 `|` 间隔，表示该属性可以是多个类型中的任意一个**

```js
/*@flow*/
class Bar {
  x: string;           // x 是字符串
  y: string | number;  // y 可以是字符串或者数字
  z: boolean;

  constructor(x: string, y: string | number) {
    this.x = x
    this.y = y
    this.z = false
  }
}

var bar: Bar = new Bar('hello', 4)
var obj: { a: string, b: number, c: Array<string>, d: Bar } = {
  a: 'hello',
  b: 11,
  c: ['hello', 'world'],
  d: new Bar('hello', 3)
}
```

- `null` 若任意类型 `T` 可为 `null` 或者 `undefined`，只需写成类似 `?T` 格式

```js
/*@flow*/
// foo 可为字符串，也可以为 null
var foo: ?string = null
```

## flow 代码的剥离

对于将 flow 代码从正常 js 代码中剥离，有两种方式:

- 使用 flow-remove-types（简单粗暴）

```js
npm install -g flow-remove-types
flow-remove-types src/ --out-dir build/
// flow-remove-types src/ -d build/
```

- 需要用到 babel-preset-flow 并配置 .babelrc

```js
npm install --save-dev babel-cli babel-preset-flow

// .babelrc
{
  "presets": ["flow"]
}
```

```js
./node_modules/.bin/babel src/ --out-dir build/
// ./node_modules/.bin/babel src/ -d build/
```