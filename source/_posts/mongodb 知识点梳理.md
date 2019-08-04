---
title: mongodb 知识点梳理
tag: 
	- mongodb
---

<!-- markdownlint-disable MD010 -->

> mongodb 为非关系型数据库，collection（集合）可对应一个关系型数据库的二维表，document（文档）对应关系型数据库的行（或者说一条记录）
> **一个集合里面各个文档的字段不一定相同**
> 一个 database 可以有多个 collection，一个 collection 中又可以有多个 document

<!-- more -->

## 安装 连接

### 通过 homebrew 安装

```js
brew install mongodb
```

### 本地安装

通过 homebrew 安装远程包无法安装，可以将 tar 包下载到本地安装

- 进入 homebrew 所在目录文件 如 /Users/jackzhang/Library/Caches/Homebre 
- open ./ 将下载好的 tar 包放入文件夹中
- 运行 ```brew install mongodb``` 会直接从本地文件进行解压安装

### 启动 **mongod --config [mongod.conf 文件路径]**

```js
mongod --config /usr/local/etc/mongod.conf
```

### 连接 **mongo**

```js
jackdeMacBook-Pro:~ jackzhang$ mongo
MongoDB shell version v3.4.2
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.4.2
Server has startup warnings:
2018-02-11T10:30:25.883+0800 I CONTROL  [initandlisten]
2018-02-11T10:30:25.883+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2018-02-11T10:30:25.883+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2018-02-11T10:30:25.883+0800 I CONTROL  [initandlisten]
2018-02-11T10:30:25.883+0800 I CONTROL  [initandlisten]
2018-02-11T10:30:25.883+0800 I CONTROL  [initandlisten] ** WARNING: soft rlimits too low. Number of files is 256, should be at least 1000
```

## mongodb 基本语法和操作

### 库操作

- 新建数据库 （1）**use 数据库名** （2）执行此库的相关操作（不进行第二部，该库不会被创建）
- 切换数据库 **use 数据库名**
- 查看数据库 **show dbs**
- 新建集合（对应关系型数据库的表）**db.createCollection('集合名')**
- 查看当前数据库下集合 **show collections**
- 删除当前数据库下指定集合 **db.集合名.drop()**
- 删除当前数据库 **db.dropDatabase()**

### 插入

没有直接创建集合，**通过插入操作也能自动创建集合**

**_id 由 monggodb 生成，每个 document（文档，对应关系数据库的行）数据都有，默认是 ObjectId，可在插入时插入这个键的值（mongogdb 支持的数据类型）**

- **db.集合名.insert(data)**

```js
use test // switched to db test
db.tb1.insert({"name": "zhang","age": 17})
// WriteResult({ "nInserted" : 1 })
db.tb1.find()
// { "_id" : ObjectId("5a7ff18a2087cb067f6dd7c7"), "name" : "zhang", "age" : 17 }
```

- **db.集合名.save(data)** 建议使用这种方式

```js
db.tb1.save({"name": "zhang","age": 17, "_id": 1})
// WriteResult({ "nMatched" : 0, "nUpserted" : 1, "nModified" : 0, "_id" : 1 })
db.tb1.find()
// { "_id" : ObjectId("5a7ff18a2087cb067f6dd7c7"), "name" : "zhang", "age" : 17 }
// { "_id" : 1, "name" : "zhang", "age" : 17 }
```

这两种方法都能插入数据，区别在于，**当默认 "_id" 存在时，调用 insert 方法会报错， save 方法则会更新相同 _id 所在的文档的数据**

```js
> db.tb1.insert({ "_id" : 1, "name" : "zhang", "age" : 19 })
// WriteResult({
// 	"nInserted" : 0,
// 	"writeError" : {
// 		"code" : 11000,
// 		"errmsg" : "E11000 duplicate key error collection: test.tb1 index: _id_ dup key:{ : 1.0 }"
// 	}
// })
db.tb1.save({ "_id" : 1, "name" : "zhang", "age" : 19 })
// 报错 WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
db.tb1.find()
// { "_id" : ObjectId("5a7ff18a2087cb067f6dd7c7"), "name" : "zhang", "age" : 17 }
// { "_id" : 1, "name" : "zhang", "age" : 19 }
```

### 查询（查询条件写成 json）

- **查询集合中所有数据 db.集合名.find()**
- 按照条件查询（支持多条件）**db.集合名.find(条件)**
- 查询第一条（支持条件）**db.集合名.findOne(条件)**
- 限制数量 **db.集合名.find().limit(数量)**
- 跳过指定数量 **db.集合名.find().skip(数量)**
- **查询数量 db.集合名.find().count()**
- **排序 db.集合名.find().sort({"字段名": 1} ) 1 升序 -1 降序**
- **指定字段返回 db.集合名.find({}, {"字段名": 0} ) 0 不返回 1 返回** 注意，除了 **_id 以外，其他字段要么都是0，要么都是1，要么不写**

```js
for(var i=0;i<3;i++){db.tb1.save({"name": "zhang"+i, "age": i})}
// WriteResult({ "nInserted" : 1 })
db.tb1.find()
// { "_id" : ObjectId("5a7ff18a2087cb067f6dd7c7"), "name" : "zhang", "age" : 17 }
// { "_id" : 1, "name" : "zhang", "age" : 19 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c8"), "name" : "zhang0", "age" : 0 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c9"), "name" : "zhang1", "age" : 1 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7ca"), "name" : "zhang2", "age" : 2 } 
db.tb1.find({"name":"zhang0"})
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c8"), "name" : "zhang0", "age" : 0 }
db.tb1.find({"name":"zhang0", "age":0})
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c8"), "name" : "zhang0", "age" : 0 }
db.tb1.findOne()
// { "_id" : ObjectId("5a7ff18a2087cb067f6dd7c7"), "name" : "zhang", "age" : 17 }
db.tb1.findOne({"name":"zhang0"})
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c8"), "name" : "zhang0", "age" : 0 }
db.tb1.find().limit(2)
// { "_id" : ObjectId("5a7ff18a2087cb067f6dd7c7"), "name" : "zhang", "age" : 17 }
// { "_id" : 1, "name" : "zhang", "age" : 19 }
db.tb1.find().skip(3)
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c9"), "name" : "zhang1", "age" : 1 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7ca"), "name" : "zhang2", "age" : 2 }
db.tb1.find().count()
// 5
db.tb1.find().sort({"age": 1}).limit(3)
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c8"), "name" : "zhang0", "age" : 0 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c9"), "name" : "zhang1", "age" : 1 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7ca"), "name" : "zhang2", "age" : 2 }
db.tb1.find().sort({"age": -1}).limit(3)
// { "_id" : 1, "name" : "zhang", "age" : 19 }
// { "_id" : ObjectId("5a7ff18a2087cb067f6dd7c7"), "name" : "zhang", "age" : 17 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7ca"), "name" : "zhang2", "age" : 2 }
db.tb1.find({},{"_id":0,"age":0,"name":1})
// 报错 投影里除了 _id 以外，要么全部是1，要么全部是0
// Error: error: {
// 	 "ok" : 0,
// 	 "errmsg" : "Projection cannot have a mix of inclusion and exclusion.",
// 	 "code" : 2,
// 	 "codeName" : "BadValue"
// }

db.tb1.find({},{"_id":0,"age":1,"name":1})
// { "name" : "zhang", "age" : 17 }
// { "name" : "zhang", "age" : 19 }
// { "name" : "zhang0", "age" : 0 }
// { "name" : "zhang1", "age" : 1 }
// { "name" : "zhang2", "age" : 2 }
db.tb1.find({},{"name":1})
// { "_id" : ObjectId("5a7ff18a2087cb067f6dd7c7"), "name" : "zhang" }
// { "_id" : 1, "name" : "zhang" }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c8"), "name" : "zhang0" }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c9"), "name" : "zhang1" }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7ca"), "name" : "zhang2" }
```

#### 比较查询

- **大于 $gt**
- **小于 $lt**
- **大于等于 $gte**
- **小于等于 $lte**
- **非等于 $ne**

如查询 tb1 年龄大于 3 的文档数据
db.tb1.find({"age": {"$gt": 3}})
如查询 tb1 年龄小于 3 的文档数据
db.tb1.find({"age": {"$lt": 3}})
如查询 tb1 年龄大于 1 小于 3 的文档数据
db.tb1.find({"age": {"$gt": 1, "$lt": 3}})
如查询 tb1 年龄不等于 0 的文档数据
db.tb1.find({"age": {"$ne": 0}})

```js
 db.tb1.find({"age": {"$gt": 3}})
// { "_id" : ObjectId("5a7ff18a2087cb067f6dd7c7"), "name" : "zhang", "age" : 17 }
// { "_id" : 1, "name" : "zhang", "age" : 19 }
db.tb1.find({"age": {"$lt": 3}})
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c8"), "name" : "zhang0", "age" : 0 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c9"), "name" : "zhang1", "age" : 1 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7ca"), "name" : "zhang2", "age" : 2 }
db.tb1.find({"age": {"$gt": 1, "$lt": 3}})
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7ca"), "name" : "zhang2", "age" : 2 }
db.tb1.find({"age": {"$ne": 0}})
// { "_id" : ObjectId("5a7ff18a2087cb067f6dd7c7"), "name" : "zhang", "age" : 17 }
// { "_id" : 1, "name" : "zhang", "age" : 19 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c9"), "name" : "zhang1", "age" : 1 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7ca"), "name" : "zhang2", "age" : 2 }
```

#### 或 查询（$or）

**多个条件写成数组**

如查询 tb1 年龄为 0 或者 名字为 zhang1 的文档数据
db.tb1.find({"$or": [{"name": "zhang1"}, {"age": 0}]})

```js
db.tb1.find({"$or": [{"name": "zhang1"}, {"age": 0}]})
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c8"), "name" : "zhang0", "age" : 0 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c9"), "name" : "zhang1", "age" : 1 }
```

#### 包含和不包含（$in $nin）

**多个条件写成数组**

如查询 tb1 年龄包含 0 17 的文档数据
db.tb1.find({"age": {"$in": [0, 17]}})

```js
db.tb1.find({"age": {"$in": [0, 17]}})
// { "_id" : ObjectId("5a7ff18a2087cb067f6dd7c7"), "name" : "zhang", "age" : 17 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c8"), "name" : "zhang0", "age" : 0 }
db.tb1.find({"age": {"$nin": [0, 17]}})
// { "_id" : 1, "name" : "zhang", "age" : 19 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c9"), "name" : "zhang1", "age" : 1 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7ca"), "name" : "zhang2", "age" : 2 }
```

### 修改 （ _id 存在的情况下调用 save 方法为修改操作）

- **db.集合名.update({'条件名称': '字段值'}, {$set: {'要修改的字段名': '修改后的字段值'}})**

```js
db.tb1.find({"name":"zhang2"})
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7ca"), "name" : "zhang2", "age" : 2 }
db.tb1.update({"name":"zhang2"},{"$set":{"age": 10}})
// WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
db.tb1.find({"name":"zhang2"})
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7ca"), "name" : "zhang2", "age" : 10 }
```

### 删除

- **db.集合名.remove(条件)**

```js
db.tb1.find()
// { "_id" : ObjectId("5a7ff18a2087cb067f6dd7c7"), "name" : "zhang", "age" : 17 }
// { "_id" : 1, "name" : "zhang", "age" : 19 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c8"), "name" : "zhang0", "age" : 0 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c9"), "name" : "zhang1", "age" : 1 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7ca"), "name" : "zhang2", "age" : 10 }
db.tb1.remove({"age":{"$gt": 5}})
// WriteResult({ "nRemoved" : 3 })
db.tb1.find()
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c8"), "name" : "zhang0", "age" : 0 }
// { "_id" : ObjectId("5a7ff9202087cb067f6dd7c9"), "name" : "zhang1", "age" : 1 }
```

## 用户名 密码

早期使用 `db.addUser()`

```js
db.addUser('root', '12345')
```

新版 api `db.createUser({user: '', pwd: '', roles: []})`

```js
db.createUser({user: 'root', pwd: '12345', roles: []})
db.createUser({user: 'root', pwd: '12345', roles: [{role: '', db: ''}]})
...
```