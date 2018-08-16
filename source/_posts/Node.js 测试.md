---
title: Node.js 单元测试
tag: 
	- Node.js
	- 单元测试
---

## BDD vs TDD

### BDD 行为驱动开发(偏向自然语言)

先写需求功能点描述，这种描述是客户、产品经理、开发、测试都能看懂的，且基于文本的；实质是让开发的产品符合客户需求

### TDD 测试驱动开发

先写测试，再写实现代码，并不断进行迭代重构，主要是给开发人员，让开发的程序没有出错

<!--more-->

## Node.js assert 断言模块

Node.js `assert` 模块提供了简单的断言测试功能，**但是只能提示出错信息，正确不会提示**

- **assert.ok(value[, message]) / assert(value[, message])**
  如果 value 值为 true，则什么也不会发生；否则抛出一个 message 的错误

- **assert.equal(actual, expected[, message])**
  判断实际值 `actual` 与期望徝 `expected` 是否相等(==)，如果不相等，则抛出一个 message 的错误。

- **assert.notEqual(actual, expected[, message])**
...

```js
// assert.js
module.exports = {
  add: (...args) => {
    return args.reduce((pre, cur) => {
      return pre + cur
    })
  }
}

// index.js
const assert = require('assert')
const { add, mul } = require('../assert')

assert.equal(add(2, 3), 5)
assert.ok(add(2, 3) === 5, 'equal')
```

## chai

`chai` 是一个类似 Node.js `assert` 模块的断言库，但是提供更多 api

### assert

`chai` 的 TDD 风格的 `assert` 类似 Node.js 的 `assert` 模块, **同样会只能提示出错信息**

- **assert.typeOf(1, 'number')**
- **assert.equal(1, 1)**
- **assert.lengthOf('str', 3)**
- **assert.property(tea, 'flavors')**
...

```js
// assert.js
module.exports = {
  add: (...args) => {
    return args.reduce((pre, cur) => {
      return pre + cur
    })
  }
}

// index.js
const { assert } = require('chai')
const { add, mul } = require('../assert')

assert.equal(add(2, 3), 5)
```

### should

`chai` 提供了 BDD 风格的 `should`，**使用前需要先调用 should()**

- **foo.should.be.a('string')**
- **foo.should.equal('bar')**
- **foo.should.have.lengthOf(3)**
- **tea.should.have.property('flavors').with.lengthOf(3)**
...

```js
// assert.js
module.exports = {
  add: (...args) => {
    return args.reduce((pre, cur) => {
      return pre + cur
    })
  }
}

// index.js
const { should } = require('chai')
const { add, mul } = require('../assert')
should()

 add(2, 3).should.equal(5)
```

### expect

相比 `chai` 的 `should`， `chai` 的 BDD 风格的 `expect` 更偏向自然语言

- **expect(foo).to.be.a('string')**
- **expect(foo).to.equal('bar')**
- **expect(foo).to.have.lengthOf(3)**
- **expect(tea).to.have.property('flavors').with.lengthOf(3)**
...

```js
// assert.js
module.exports = {
  add: (...args) => {
    return args.reduce((pre, cur) => {
      return pre + cur
    })
  }
}

// index.js
const { expect } = require('chai')
const { add, mul } = require('../assert')

expect(add(2,3)).to.equal(5)
```

## mocha

`mocha` 是一个可以运行在浏览器端和 Node.js 上的**测试框架**, 支持任何一个可以抛出错误的断言库, **可以提示正确信息**

### 安装运行

```bash
# 安装
npm i mocha -g
npm i mocha -D

# 运行
mocha xxx/xxx
```

### api

- **describe(message, callback)**
  `describe` 方法的第一个参数会被输出在控制台，作为一个用例集合的描述，第二个参数是用例集合定义函数
  **可以无限嵌套**

- **it(message, callback)**
  `it` 方法的第一个参数会输出在控制台上，作为一个用例的描述，第二个参数是函数，用来编写测试用例，用断言模块来判断结果正确
  **可以无限嵌套**

```js
// assert.js
module.exports = {
  add: (...args) => {
    return args.reduce((pre, cur) => {
      return pre + cur
    })
  },
  mul: (...args) => {
    return args.reduce((pre, cur) => {
      return pre * cur
    })
  }
}

// index.js
const { add, mul } = require('../assert')
const { assert } = require('chai')

describe('#Math', function() {
  describe('add', function() {
    it('should return 5 when 2 + 3', function() {
      assert(add(2, 3), 5)
    })
    it('should return -1 when 2 + (-3)', function() {
      assert(add(2, -3), -1)
    })
  })

  describe('mul', function() {
    it('should return 6 when 2 * 3', function() {
      assert(mul(2, 3), 6)
    })
  })
})
```

mocha index.js

```bash
#Math
  add
    ✓ should return 5 when 2 + 3
    ✓ should return -1 when 2 + (-3)
  mul
    ✓ should return 6 when 2 * 3

3 passing (6ms)
```

#### 仅执行一个用例集/用例函数

- 在 `describe` 或 `it` 之后添加 **`.only()`**, 可以让 `mocha` 只执行此用例集/用例函数

```js
// index.js
// ...

describe('#Math', function() {
  describe('add', function() {
    it.only('should return 5 when 2 + 3', function() {
      assert(add(2, 3), 5)
    })
    it('should return -1 when 2 + (-3)', function() {
      assert(add(2, -3), -1)
    })
  })

  describe('mul', function() {
    it('should return 6 when 2 * 3', function() {
      assert(mul(2, 3), 6)
    })
  })
})
```

mocha index.js

```bash
 #Math
    add
      ✓ should return 5 when 2 + 3

  1 passing (5ms)
```

#### 跳过用例集/用例函数

- 在 `describe` 或 `it` 之后添加 **`.skip()`**, 可以让 `mocha` 跳过这些用例集/用例函数

```js
// index.js
// ...

describe('#Math', function() {
  describe('add', function() {
    it('should return 5 when 2 + 3', function() {
      assert(add(2, 3), 5)
    })
    it.skip('should return -1 when 2 + (-3)', function() {
      assert(add(2, -3), -1)
    })
  })

  describe('mul', function() {
    it('should return 6 when 2 * 3', function() {
      assert(mul(2, 3), 6)
    })
  })
})
```

mocha index.js

```bash
#Math
    add
      ✓ should return 5 when 2 + 3
      - should return -1 when 2 + (-3)
    mul
      ✓ should return 6 when 2 * 3

  2 passing (6ms)
  1 pending
```

#### 钩子函数

`mocha` 提供四种钩子函数：`before()` `after()` `beforeEach()` `afterEach()`, 这些钩子函数可以让 `mocha` 在用例集/用例函数开始执行前、执行结束后做一些环境准备或环境清理的工作

```js
// index.js
// ...

describe('#Math', function() {
  describe('add', function() {
    before(function() {
      console.log('before all task')
    })
    this.beforeEach(function() {
      console.log('before every task')
    })
    after(function() {
      console.log('after all task')
    })
    this.afterEach(function() {
      console.log('after every task')
    })

    it('should return 5 when 2 + 3', function() {
      assert(add(2, 3), 5)
    })
    it.skip('should return -1 when 2 + (-3)', function() {
      assert(add(2, -3), -1)
    })
  })

  describe('mul', function() {
    it('should return 6 when 2 * 3', function() {
      assert(mul(2, 3), 6)
    })
  })
})
```

mocha index.js

```bash
 #Math
    add
before all task
before every task
      ✓ should return 5 when 2 + 3
after every task
      - should return -1 when 2 + (-3)
after all task
    mul
      ✓ should return 6 when 2 * 3


  2 passing (6ms)
  1 pending
```

#### 异步代码测试

- 用例集/用例函数的参数 传入 `done`，并在异步代码调用结束后执行 `done()`, 相当于通知 `mocha` 可以执行下一个用例
- **`mocha` 进入一个计时器，等待异步函数来完成 - 这时要么调用done()函数，或者超过 2秒 导致超时**

```js
describe('test', function(){
  var foo = false
  beforeEach(function(done){
    setTimeout(function(){
      foo = true
      // complete the async beforeEach
      done()
    }, 1000)
  })
  it('should pass', function(){
    assert.equal(foo, true)
  })
})
```

```bash
  #Math
    test
      ✓ should pass


  1 passing (1s)
```

```js
describe('test', function(){
  var foo = false
  beforeEach(function(done){
    setTimeout(function(){
      foo = true
      // complete the async beforeEach
      done()
    }, 3000)
  })
  it('should pass', function(){
    assert.equal(foo, true)
  })
})
```

```bash
  #Math
    test
      1) "before each" hook for "should pass"

  0 passing (2s)
  1 failing

  1) #Math test "before each" hook for "should pass":
     Error: Timeout of 2000ms exceeded. For async tests and hooks, ensure "done()" is called; if returning a Promise, ensure it resolves.
```

## istanbul

当代码较多以及分支较多时，为保证测试代码覆盖率，需要使用 `istanbul`, 可以结合 `mocha`

主要是判断 **statement** **branches** **function** **lines**

### 安装使用

```bash
# 安装
npm i istanbul -g
npm i istanbul -D

# 运行
istanbul cover xx/xxx
# 将 mocha 进行测试，并通过 istanbul 进行覆盖率测试（_mocha 使得 istanbul 中拿到 mocha 的实例）
istanbul cover _mocha xx/xxx
```

### 基本概念

- **statements** 声明（包括函数、变量等）
- **branches** 分支（包括 if-else 等语句）
- **function** 函数
- **lines** 行数

```js
// assert.js
function min(a, b) {
  return Math.min(a, b)
}

module.exports = {
  add: (...args) => {
    return args.reduce((pre, cur) => {
      return pre + cur
    })
  },
  mul: (...args) => {
    return args.reduce((pre, cur) => {
      return pre * cur
    })
  },
  cover: (a, b) => {
    if (a > b) {
      return a - b
    } else {
      return min(a, b)
    }
  }
}

// index.js
const { add, mul, cover } = require('../assert')
const { assert } = require('chai')

describe('#Math', function() {
  describe('add', function() {
    it('should return 5 when 2 + 3', function() {
      assert(add(2, 3), 5)
    })
  })

  describe('mul', function() {
    it('should return 6 when 2 * 3', function() {
      assert(mul(2, 3), 6)
    })
  })
})
```

istanbul cover _mocha index.js

```bash
  #Math
    add
      ✓ should return 5 when 2 + 3
    mul
      ✓ should return 6 when 2 * 3


  2 passing (7ms)

=============================================================================
Writing coverage object [/Users/jackzhang/workspace/code/test/assert/coverage/coverage.json]
Writing coverage reports at [/Users/jackzhang/workspace/code/test/assert/coverage]
=============================================================================

=============================== Coverage summary ===============================
Statements   : 60% ( 6/10 )
Branches     : 0% ( 0/2 )
Functions    : 0% ( 0/1 )
Lines        : 60% ( 6/10 )
================================================================================
```

```js
// index.js
const { add, mul, cover } = require('../assert')
const { assert } = require('chai')

describe('#Math', function() {
  describe('add', function() {
    it('should return 5 when 2 + 3', function() {
      assert(add(2, 3), 5)
    })
  })

  describe('mul', function() {
    it('should return 6 when 2 * 3', function() {
      assert(mul(2, 3), 6)
    })
  })

  describe('cover', function() {
    it('should return 1 when 3 > 2', function() {
      assert(cover(3, 2), 1)
    })

    it('should return 2 when 2 > 3', function() {
      assert(cover(2, 3), 2)
    })
  })
})
```

istanbul cover _mocha index.js

```bash
  #Math
    add
      ✓ should return 5 when 2 + 3
    mul
      ✓ should return 6 when 2 * 3
    cover
      ✓ should return 1 when 3 > 2
      ✓ should return 2 when 2 > 3


  4 passing (14ms)

=============================================================================
Writing coverage object [/Users/jackzhang/workspace/code/test/assert/coverage/coverage.json]
Writing coverage reports at [/Users/jackzhang/workspace/code/test/assert/coverage]
=============================================================================

=============================== Coverage summary ===============================
Statements   : 100% ( 10/10 )
Branches     : 100% ( 2/2 )
Functions    : 100% ( 1/1 )
Lines        : 100% ( 10/10 )
================================================================================
```

## travis-ci 持续集成

> Travis CI 是在软件开发领域中的一个在线的，分布式的持续集成服务，用来构建及测试在 GitHub 托管的代码

为了尽早发现错误，防止分支大幅度偏离主干，对于频繁将代码提交远程仓库，可以进行持续集成。在提交代码时让 travis-ci 跑一些任务

### 使用步骤

- travis-ci 官网对 github 项目开启
- 项目本地编写 `.travis.yml` 配置文件
- 提交代码到远程

### .travis.yml 配置

项目仓库中的 `.travis.yml` 文件，一旦代码仓库中有新的 commit，`travis` 会找到这个配置文件并执行里面的命令

- **language** 指定了默认运行环境
- **install** 安装，一般用于执行安装项目依赖命令
- **before_script** 指定运行的脚本命令之前的操作
- **script** 指定要运行的脚本，`script: true` 表示不执行任何脚本，状态直接设为成功
- **after_script** 指定运行的脚本命令之后的操作
- **branch** 指定分支，只有指定的分支提交时才会运行脚本

```bash
language: node_js
# nodejs版本
node_js:
    - '6'

# Travis-CI Caching
cache:
  directories:
    - node_modules


# S: Build Lifecycle
install:
  - npm install

before_script:

# 无其他依赖项所以执行npm run build 构建就行了
script:
  - npm run build

after_script:
  - cd ./dist
  - git init
  - git config user.name "${U_NAME}"
  - git config user.email "${U_EMAIL}"
  - git add .
  - git commit -m "Update tools"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:${P_BRANCH}
# E: Build LifeCycle

#指定分支，只有指定的分支提交时才会运行脚本
branches:
  only:
    - master
env:
 global:
```

### 应用：github pages 博客持续集成

笔者之前已经通过 github pages 搭建了 hexo 主题的博客，部署需要执行以下命令

```bash
# 清空 public 文件夹
hexo clean

# hexo generate 将文章编译到 public 文件夹
hexo g

# hexo deploy 将 public 内容部署到 jhgrrewq.github.io 仓库
hexo d
```

jhgrrewq.github.io 仓库下 master 分支是 public 文件夹内容，也就是博客的静态内容，为了使用 travis 持续集成，需要将博客源代码提交到一个新分支如 `hexo`

#### 步骤

- 在 travis 官网启动应用
- 在 travis 对应项目（笔者的是 jhgrrewq.github.io）下进入 `setting`
- 在 `Environment Variables` 选项下添加 [`github token`](https://github.com/settings/tokens)

![](http://ony85apla.bkt.clouddn.com/18-8-15/90889324.jpg)

#### .travis.yml 配置文件

```bash
language: node_js
node_js: stable

addons: # Travis CI建议加的，自动更新api
  apt:
    update: true

cache:
  directories:
  - node_modules # 缓存 node_modules

install:
- npm install # 初次安装，在CI环境中，执行安装npm依赖

# before_script:

script:
- hexo clean
- hexo g # 执行 hexo generate，把文章编译到 public

after_success: # 执行 script 成功后，进入到 public
- cd ./public
- git init
- git config user.name "jhgrrewq"
- git config user.email "52792521@163.com"
- git add .
- git commit -m "Update site"
- git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master
- git push "https://qwerrghj:${CO_TOKEN}@${CO_REF}" master:master
# 分别推送到 github coding.net 需要使用到 token 登录

branches:
  only:
  - hexo # CI 只针对分支 hexo

env:
  global: # 全局变量，上面的提交到github的命令有用到
  - GH_REF: github.com/jhgrrewq/jhgrrewq.github.io.git
  - CO_REF: git.coding.net/qwerrghj/qwerrghj.coding.me.git
  - secure:
# secure是自动生成的，执行`travis encrypt 'GH_TOKEN=${your_github_personal_access_token}' --add`
```

编写好 .travis.yml 文件后，以后分支下文件修改提交到远程，会自动持续集成

#### 常见问题

- 本地页面正常，推送后页面空白

一般 next 主题文件缺少导致，查看远程仓库分支下 themes/next 一般是空的。只需要将 themes/next 全部内容添加进 git 版本依赖即可