---
title: mongoose 知识点梳理
tag: 
	- mongoose
---
<!-- markdownlint-disable MD010 -->

> 参考: [mongoose 入门](https://www.cnblogs.com/zhongweiv/p/mongoose.html)

mongoose 作为 mongodb 建模工具(建议使用 4.10.8 版本)，**使用前请确保安装 mongodb**

<!-- more -->

## 连接

```js
var mongoose = require('mongoose')
    url = "mongodb://localhost:27017/mongoosesample"
mongoose.connect(url)
// mongoose.connect(url, {useMongoClient: true})  4.11 版本

// 连接成功
mongoose.connection.on('connected', function() {
    console.log("Mongoose connection open to " + url)
})

// 连接异常
mongoose.connection.on('error', function(err) {
    console.log("Mongoose connection error: " + err)
})

// 连接断开
mongoose.connection.on('disconnected', function() {
    console.log("Mongoose connection disconnected")
})
```

## Schema

Schema 可理解为**文档结构的定义**，Schema 不具备操作数据库的能力， model 代表数据库的文档，具有数据库操作能力

**定义一个 Schema 就是指定字段名和类型**
- **定义 Schema：new mongoose.Schema({}) 传入配置对象**
- Schema Types 内置类型包括 String Number Boolean|Bold Array Buffer Date Mixed ObjectId|Oid
- Schema 如果包含其他对象，则将这些对象包含在 meta 属性里面

```js
var mongoose = require('mongoose')
    url = "mongodb://localhost:27017/mongoosesample"
mongoose.connect(url)

// 定义 Schema
var userSchema = new mongoose.Schema({
    // 设置字段名和类型 简单的方式使用键值对
    // 对于属性的多个参数值设置 { type: String|Number..., ... }
    name: String,
    username: {
        type: String,
        required: true,
        unique: true,
        index: true
    },
    // 设置 username 唯一 unique: true, 因此插入多条相同数据只会保留一条
    // 设置 index 是建立索引，默认主键索引是 ObjectId, 属性名是 _id
    password: {
        type: String,
        required: true
    },
    admin : Boolean,
    location: String,
    age: Number,
    // 若包含其他对象，可将其包含在 meta 属性中
    meta: {
        age: Number,
        website: String
    },
    created_at: {
        type: Date,
        default: Date.now()
    },
    update_at: Date
})
```

- Schema 可添加**公共方法，给 Model 实例化后调动**
- schemaName.**methods**.fnName = function() {// ...}

```js
// ...
userSchema.methods.capitalizeName = function() {
    this.name = this.name.toUpperCapital()
    return this.name
}
```

- Schema 可添加**静态方法，给 Model 调动**
- schemaName.**static**.fnName = function() {// ...}

```js
// ...
userSchema.static.log = function() {
    console.log('log')
}
```

- Schema 可调用 pre 中间件
- schemaName.pre(event, function(next){next()}）

```js
// pre 中间件 如下定义在每次操作之前做其他处理
userSchema.pre('save', function(next) {
    var currentDate = new Date()
    this.update_at = currentDate
    if (!this.create_at) {
        this.create_at = currentDate
    }
    next()
})
```

## Model（定义好了 Schema 后要生成 Model）

Model 是由 Schema 生成的模型，可以操作数据库

- 生成 Model **mongoose.model('模型名', Schema 名)**

```js
// 使用 mongoose.model(模型名, Schema 名) 生成 Model
var User = mongoose.model('User', userSchema)
// 导出 Model
module.exports = User

// 实例化 new ModelName({}) 传入参数实例化
new User({
    name: ...
    // ...
})
```

## 数据库操作 创建

- **利用前面定义的 Model(User) 先 new 一个实例，再调用内置的 save 方法**

内置的 save 方法 给 Model 实例调用

```js
// create a new user
var newUser = new User({
    name: 'jack',
    username: 'user1'
})

// save the user
newUser.save(function(err) {
    if (err) throw err
    console.log('User created!')
})
```

- 或者**调用 Model 的 create 方法**

```js
User.create({
    name: 'jack',
    username: 'user1'
}, function(err, user) {
    if (err) throw err
    console.log('User created!')
})
```

## 数据路操作 删除

- **Model.remove(conditions, [callback])**

```js
User.remove({'username': 'jack'}, function(err, user) {
    if (err) {
        console.log('Error: ' + err)
    } else {
        console.log('user: ' + user)
    }
})
```

常用的方法还有

- **Model.findByIdAndRemove(id, [options], [callback])** 根据 **_id** 删除
- **Model.findOneAndRemove(conditions, [options], [callback])** 根据条件找到第一个删除

## 数据库操作 更新

- **Model.update(conditions, update, [options], [callback])**

```js
User.update({'username': 'jack'}, {'password': 'zzz'}, function(err, user) {
    if (err) {
        console.log('Error: ' + err)
    } else {
        console.log('user: ' + user)
    }
})
```

常用的方法还有

- **Model.findByIdAndUpdate(id, [update],[options], [callback])** 根据 **_id** 更新
- **Model.findOneAndUpdate([conditions], [update], [options], [callback])** 根据条件找到第一个并更新

## 数据库操作 查询

### 直接查询： 查询时候带有回调函数

- **Model.find(conditions, [fields], [options], [callback])**
- **Model.findOne(conditions, [fields], [options], [callback])**

```js
User.find({'username': 'jack'}, 'some select', function(err, user) {
    if (err) {
        console.log('Error: ' + err)
    } else {
        console.log('user: ' + user)
    }
})
```

### 链式查询：**查询时不带回调函数，最后需要将 callback 传入 exec 方法调用**

```js
// 因为 query 操作使用返回自身，因此可以链式调用
var query = User.findOne({'username': 'jack'})
    query.select('some select')
    query.exec(callback)

// 如分页查询
var pageSize = 5 // 一页多少条
var currentPage = 1 // 当前第几页
var sort = { 'create_at': -1 } // 排序（按照创建时间倒叙）
var condition = {} // 条件
var skipnum = (currentPage - 1) * pageSize // 跳过数

User.find(condition).skip(skipnum).limit(pageSize).sort(sort).exec(function(err, users) {
    if (err) {
        console.log('Error: ' + err)
    } else {
        console.log('users: ' + users)
    }
})
```

### 查询全部（查询条件为空，则返回全部文档）

```js
app.get('/all', function(req, res) {
    User.find({}, function(err, users) {
        if (err) {
            res.send({message: 'error'})
            return
        }
        res.send({message: 'done', data: users})
    })
})
```

### 查询第二个参数可以设置过滤

- **$or** 或关系
- **$nor** 或关系取反
- **$gt** 大于
- **$gt** 大于
- **$gte** 大于等于
- **$lt** 小于
- **$lte** 小于等于
- **$ne** 不等于
- **$in** 在多个值范围内
- **$nin** 不在多个值范围内
- **$all** 匹配数组中的多个值
- **$regex** 正则，用于模糊查询
- **$size** 匹配数组大小
- **$maxDistance** 范围查询，距离（基于LBS）
- **$mod** 取模运算
- **$near** 邻域计算，查询附近的位置（基于LBS）
- **$exists** 字段是否存在
- **$elemMatch** 匹配数组内的元素
- **$within** 范围查询（基于LBS）
- **$box** 范围查询，矩形范围（基于LBS）
- **$center** 范围查询，圆形范围（基于LBS）
- **$centerSphere** 范围查询，球形范围（基于LBS）
- **$slice** 查询字段集合中的元素（比如从第几个之后，第 N 到第 M 个元素）

**User.find({"age": {"$gte": 21, "$lte": 56}}, callback)** 查询年龄大于等于 21 而且小于等于 65 岁
**User.find({"username": {"$regex": /m/i}}, callback)** 正则查询用户命中包含 m 的用户

其他常用方法

- **Model.findOne([conditions], [options], [callback])** 根据条件找到第一条记录
- **Model.findById(id, [fields], [options], [callback])** 根据 _id 查找

```js
var id = "56f261fb448779caa359cb73"
User.findById(id, function(err, user) {
    if (err) {
        console.log('Error: ' + err)
    } else {
        console.log('users: ' + users)
    }
})
```

### 数量查询

- **Model.cunt(conditions, [callback])**

```js
User.count({}, function(err, count) {
    if (err) {
        console.log('Error: ' + err)
    } else {
        console.log('users: ' + users)
    }
})
```

### 去重

- **Model.distict(field, [conditions], [callback])**

## mongooose 和 promise

以上数据库操作或链式调用的回调除了作为最后一个参数传入，还可以最终在 exec(callback)

**可以指定第三方 promise 库如 bluebird**

- **mongoose.Promise = require('bluebird') // 将 mongoose 已经不建议的内置的 promise 设为 bluebird**

- **Model.find().exec() 链式操作返回的就是一个 promise 对象**

```js
User.find({
    name: 'jack'
}).exec()
    .then(function(user) {
        if (users.length <=0) {
            throw new Error('no such user')
        }
        console.log('user:' + user)
        res.send({
            message: 'done',
            data: user
        })
    }).catch(function(err) {
        console.log('err:' + err)
        res.send({
            message: 'error'
        })
    })
```