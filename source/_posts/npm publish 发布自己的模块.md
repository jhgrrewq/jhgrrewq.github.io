---
title: npm publish 发布自己的模块
tag:
	- 
---

在 npm 官网上发布自己的模块首先需要在 https://www.npmjs.com 注册账号

## 1 编写模块

一般遵循 umd / amd / commonJS 风格编写模块

## 2 初始化包描述文件 package.json

使用 npm init 命令初始化包描述文件，要注意两点：

- **包名不能包含大写字母**
- **最好先在 npm 官网搜索一下包名是否已被占用，被占用的包名模块是无法发布的**

```js
npm init
```

<!-- more -->

## 3 上传模块

```js
npm publish
```

## 对模块管理

### 查看当前项目引入了哪些模块

```js
npm ls
```

### 查看模块拥有者

```js
npm owner ls <package_name>
```

### 添加一个发布者

```js
npm owner add <user> <package_name>
```

### 删除一个发布者

```js
npm owner rm <user> <package_name>
```

### 撤销发布包

```js
npm unpublic <package_name> --force
```

- **根据规范，只有发布 24 小时内才允许撤销发布的包**
- **即使撤销，发包时也不能再和被撤销的包的名称和版本重复了（不能名称相同，版本相同，因为这两者构成的唯一标志被占用）**

## 常见问题

### 问题 1

上传模块可能需要验证用户身份, 使用命令 npm adduser 输入用户名密码

```js
npm adduser
```

### 问题 2

```js
npm ERR! you do not have permission to publish "your module name". Are you logged in as the correct user?
```

报错提示没有权限，实是包名被占用。要在 npm 官网搜索没有被占用的包名后修改 package.json 的 name 字段，重新上传

- 为了防止命名的包名和 npm 上的包名重复，可使用作用域，将该包作为你 npm 官网用户名下的**私有模块**，命名规则为 `@` + `npm 官网用户名 / 模块名`
- 最后命令行发布包时 `npm publish --access=public` 将该包变成共有包

### 问题 3

```js
no_perms Private mode enable, only admin can publish this module
```

一般是使用了国内的淘宝镜像源，需要修改会 npm 源

```js
npm config set registry http://registry.npmjs.org
```

返回淘宝镜像源

```js
npm config set registry http://registry.npmjs.taobao.org
```

或者使用 nrm 模块用于切换镜像源

```js
// 全局安装
npm i -g nrm
// 添加 registry 地址
npm add npm http://registry.npmjs.org
npm add taobao https://registry.npm.taobao.org
// 切换 registry 地址
nrm use npm // 使用 npm 源
nrm use taobao // 使用淘宝源
```