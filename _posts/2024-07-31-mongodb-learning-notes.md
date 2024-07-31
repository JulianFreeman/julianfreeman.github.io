---
title: MongoDB 学习笔记
date: 2024-07-31 12:03:00 -0400
categories: [学习笔记, MongoDB]
tags: [mongodb]
description: 本文记录学习 MongoDB 的笔记，教程来源：Bro Code
---

## 基本概念

document 类似于 SQL 中的一行数据，表现为键值对，实质为 BSON 结构（Binary JSON）。

collection 是一系列 document 的集合，类似于 SQL 中的一个表（table）。

database 是一系列 collection 的集合，类似于 SQL 中的一个数据库（database）。

## mongosh 基本使用

可以单独安装 mongosh，也可以使用 MongoDB Compass 内置的 mongosh。打开 Compass 后左下角就是 mongosh。

### 列举 database

```terminal
test> show dbs
admin   40.00 KiB
config  72.00 KiB
local   80.00 KiB
school  40.00 KiB
```

### 使用 database

```terminal
test> use school
switched to db school
```

如果指定的 database 不存在，会直接创建这个 database。但是如果 database 是空的，用 `show dbs` 不会列举出来，需要填充一些内容才行。

### 创建 collection

```terminal
school> db.createCollection("students")
{ ok: 1 }
```

`cls` 可以清屏。

### 删除 database

```terminal
school> db.dropDatabase()
{ ok: 1, dropped: 'school' }
```

该函数会删除 **当前** database。

### 插入 document

```terminal
school> db.students.insertOne({name: "Spongebob", age: 30, gpa: 3.2})
{
  acknowledged: true,
  insertedId: ObjectId('66aa682fd0a384248e8892e4')
}
```

函数 `insertOne` 接收一个 document 参数。

也可以一次插入多个 document：

```terminal
school> db.students.insertMany([{name: "Patrick", age: 38, gpa: 1.5}, {name: "Sandy", age: 27, gpa: 4.0}, {name: "Gary", age: 18, gpa: 2.5}])
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId('66aa6e5ad0a384248e8892e5'),
    '1': ObjectId('66aa6e5ad0a384248e8892e6'),
    '2': ObjectId('66aa6e5ad0a384248e8892e7')
  }
}
```

函数 `insertMany` 接收一个 document 的列表。

### 查找 document

```terminal
school> db.students.find()
{
  _id: ObjectId('66aa682fd0a384248e8892e4'),
  name: 'Spongebob',
  age: 30,
  gpa: 3.2
}
{
  _id: ObjectId('66aa6e5ad0a384248e8892e5'),
  name: 'Patrick',
  age: 38,
  gpa: 1.5
}
{
  _id: ObjectId('66aa6e5ad0a384248e8892e6'),
  name: 'Sandy',
  age: 27,
  gpa: 4
}
{
  _id: ObjectId('66aa6e5ad0a384248e8892e7'),
  name: 'Gary',
  age: 18,
  gpa: 2.5
}
```

`find` 函数不加参数，会返回 collection 内所有的 document，类似于 `select *`。

## 数据类型

```terminal
school> db.students.insertOne({
  "name": "Larry",
  "age": 32,
  "gpa": 2.8,
  "fullTime": false,
  "registerDate": new Date(),
  "gradutionDate": null,
  "courses": [
    "Biology",
    "Chemistry",
    "Calculus"
  ],
  "address": {
    "street": "123 Fake St.",
    "city": "Bikini Bottom",
    "zip": 12345
  }
})
{
  acknowledged: true,
  insertedId: ObjectId('66aa6ff7d0a384248e8892e8')
}
```

上述囊括了基本的数据类型：字符串，整数，双精度浮点数，布尔值，日期（`new Date()`），空值（`null`），列表，文档（`{}`）。

## mongosh 查找

### 排序

```terminal
school> db.students.find().sort({name: 1})
```

`1` 表示正向排序，`-1` 表示逆向排序。`{name: 1}` 表示按照 `name` 正向排序。

排序后也可以限制输出个数：

```terminal
school> db.students.find().sort({gpa: -1}).limit(2)
```

这里 `find` 返回一个游标对象后，可以链式调用。

跳过固定个数，相当于 `offset`：

```terminal
school> db.students.find().sort({gpa: -1}).limit(2).skip(1)
```

`skip` 和 `limit` 的前后顺序 **不会** 影响结果。MongoDB 总是会先跳过固定个数，再输出固定个数。

### 自定义输出字段

有时候我们不想要一个 document 中的所有字段，而只要其中的某一个或某几个，这个可以通过 `find` 函数的第二个参数指定。

`find` 的第一个参数相当于 `where`，指定 `{}` 表示没有条件，返回全部。第二个参数可以指定返回哪些字段：

```terminal
school> db.students.find({}, {name: true})
[
  { _id: ObjectId('66aa682fd0a384248e8892e4'), name: 'Spongebob' },
  { _id: ObjectId('66aa6e5ad0a384248e8892e5'), name: 'Patrick' },
  { _id: ObjectId('66aa6e5ad0a384248e8892e6'), name: 'Sandy' },
  { _id: ObjectId('66aa6e5ad0a384248e8892e7'), name: 'Gary' },
  { _id: ObjectId('66aa6ff7d0a384248e8892e8'), name: 'Larry' }
]
```

默认情况下 `_id` 一定会输出，但是我们可以显式指定不要它：

```terminal
school> db.students.find({}, {name: true, _id: false})
[
  { name: 'Spongebob' },
  { name: 'Patrick' },
  { name: 'Sandy' },
  { name: 'Gary' },
  { name: 'Larry' }
]
```

### 条件查询

`find` 的第一个参数相当于 `where`，简单的相等查询可以像如下：

```terminal
school> db.students.find({gpa: 4.0})
[
  {
    _id: ObjectId('66aa6e5ad0a384248e8892e6'),
    name: 'Sandy',
    age: 27,
    gpa: 4
  }
]
```

但是复杂一些的比如大于或者小于等，需要用到特殊的操作符，比如：

```terminal
school> db.students.find({age: {$gt: 20, $lt: 31}})
[
  {
    _id: ObjectId('66aa682fd0a384248e8892e4'),
    name: 'Spongebob',
    age: 30,
    gpa: 3.2
  },
  {
    _id: ObjectId('66aa6e5ad0a384248e8892e6'),
    name: 'Sandy',
    age: 27,
    gpa: 4
  }
]
```

`age: {$gt: 20, $lt: 31}` 表示 `age` 这个字段大于 20 但是小于 31 的 document。

> 更多操作符查阅 [官方文档](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/query/)。
{: .prompt-info}

### 更新

```terminal
school> db.students.updateOne({name: "Spongebob"}, {$set: {fullTime: true}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
school> db.students.find({name: "Spongebob"})
[
  {
    _id: ObjectId('66aa682fd0a384248e8892e4'),
    name: 'Spongebob',
    age: 30,
    gpa: 3.2,
    fullTime: true
  }
]
```

函数 `updateOne` 的第一个参数是条件，相当于 `where`，语法跟 `find` 的第一个参数一样。第二个参数是更新操作，其中常用的就是 `$set` 和 `$unset`。

顾名思义，前者给 document 设置或更新字段，后者给 document 删除字段。

> 其他更新操作符见 [官方文档](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/update/)。
{: .prompt-info}

`db.students.updateOne({name: "Spongebob"}, ...)` 表示的是更新 `students` 这个 collection 中，`name` 为 `Spongebob` 的 **第一个** document。如果第一个参数指定为 `{}`，则表示更新该 collection 中的第一个 document。

这里的 *第一个* 应该是 document 的插入顺序。

还有一个函数 `updateMany` 可以一次更新多个文档，比如 `db.students.updateMany({name: "Spongebob"}, {$set: {fullTime: true}})` 表示更新 `students` 这个 collection 中 `name` 为 `Spongebob` 的 **所有** document。如果第一个参数指定为 `{}`，则表示更新该 collection 中的所有 document。

> 所以如果只是想用 `updateOne` 更新一个文档，条件那里最好用 `_id`，因为 `_id` 在同一个 collection 中一定是唯一的。
{: .prompt-tip}

如果移除一个字段，可以如下：

```terminal
school> db.students.updateOne({name: "Spongebob"}, {$unset: {fullTime: ""}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
school> db.students.find({name: "Spongebob"})
[
  {
    _id: ObjectId('66aa682fd0a384248e8892e4'),
    name: 'Spongebob',
    age: 30,
    gpa: 3.2
  }
]
```

如上，删除一个字段，形如 `$unset: {fullTime: ""}` 即可。

### 删除

删除也分 `deleteOne` 和 `deleteMany`，两者都接收一个参数，也就是 `where` 条件，前者删除找到的第一个 document，后者删除找到的所有 document。

```terminal
school> db.students.deleteOne({name: "Larry"})
{ acknowledged: true, deletedCount: 1 }
```

```terminal
school> db.students.deleteMany({gpa: {$lt: 3.0}})
{ acknowledged: true, deletedCount: 2 }
```

## mongosh 索引

创建索引可以加快查找速度，但是会减慢插入和更新速度。

### 创建索引

```terminal
school> db.students.createIndex({name: 1})
name_1
```
上述表示创建 `name` 字段的索引，按照升序（不过对于单字段来说，顺序不重要），创建的索引名称为 `name_1`。

### 查看索引

```terminal
school> db.students.getIndexes()
[
  { v: 2, key: { _id: 1 }, name: '_id_' },
  { v: 2, key: { name: 1 }, name: 'name_1' }
]
```

### 删除索引

```terminal
school> db.students.dropIndex("name_1")
{ nIndexesWas: 2, ok: 1 }
```
