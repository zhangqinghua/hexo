---
title: MongoDB 常用操作

categories:
- MongoDB

date: 2020-05-22
---

## 查询文档
```js
// query：可选，使用查询操作符指定查询条件
// projection：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）
// collection：表名
db.collection.find(query, projection)
```


```js
// pretty() 方法以格式化的方式来显示所有文档
db.collection.find().pretty()
```


```js
db.col.find().pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
````

如果你熟悉常规的 SQL 数据，通过下表可以更好的理解 MongoDB 的条件语句查询：

|操作|范例|SQL类似语句|
|:--|:--|:--|
|等于|db.col.find({"by":"菜鸟教程"}).pretty()|where by = '菜鸟教程'|
|小于|db.col.find({"likes":{$lt:50}}).pretty()|where likes < 50|
|小于或等于|db.col.find({"likes":{$lte:50}}).pretty()|where likes <= 50|
|大于|db.col.find({"likes":{$gt:50}}).pretty()|where likes > 50|
|大于或等于|db.col.find({"likes":{$gte:50}}).pretty()|where likes >= 50|
|不等于|db.col.find({"likes":{$ne:50}}).pretty()|	where likes != 50|