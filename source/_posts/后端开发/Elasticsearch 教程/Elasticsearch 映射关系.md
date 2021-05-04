---
title: Elasticsearch 映射关系

categories:
- 后端开发
- Elasticsearch 教程

date: 2021-04-18 00:97:95
---

## 字段类型
在 ES 中，字段的类型很关键：
1. 在索引的时候，如果字段第一次出现，会自动识别某个类型；
1. 那么如果一个字段已经存在了，并且设置为某个类型。再来一条数据，字段的数据不与当前的类型相符，就会出现字段冲突的问题。如果发生了冲突，在 2.x 版本会自动拒绝；
1. 如果自动映射无法满足需求，就需要使用者自己来设置映射类型，因此，就需要使用者了解 ES 中的类型；

#### 字段中的索引和存储
`index` 定义字段的分析类型以及检索方式：
1. 如果是 `no`，则无法通过检索查询到该字段；
1. 如果设置为 `analyzed` 则将会通过默认的 standard 分析器进行分析。
1. 如果设置为 `not_analyzed` 则会将整个字段存储为关键词，常用于 Id、汉字短语、邮箱等复杂的字符串；

#### string
字符串类型，es中最常用的类型。比较重要的参数：
1. `index` 分析
   `analyzed`（默认）
   `not_analyzed` 设置为该值可以保证该字段能通过检索查询到
   `no`
1. `store` 存储
   `true` 独立存储
   `false`（默认）不存储，从 `_source` 中解析

## numeric
数值类型，注意 numeric 并不是一个类型，它包括多种类型，比如：`long`、`integer`、`short`、`byte`、`double`、`float`，每种的存储空间都是不一样的，一般默认推荐 `integer` 和 `float`。

重要的参数：
1. `index` 分析
   `not_analyzed`（默认），设置为该值可以保证该字段能通过检索查询到
   `no`
1. `store` 存储
   `true` 独立存储
   `false`（默认）不存储，从 `_source` 中解析

#### date
日期类型，该类型可以接受一些常见的日期表达方式。

重要的参数：
1. `index` 分析
   `not_analyzed`（默认），设置为该值可以保证该字段能通过检索查询到
   `no`
1. `store` 存储
   `true` 独立存储
   `false`（默认）不存储，从 `_source` 中解析
1. `format` 格式化
   `epoch_millis`（默认）
   `strict_date_optional_time`

我们也可以自定义格式化内容，例如：

```json
"date": {
  "type":   "date",
  "format": "yyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
}
```

#### ip
这个类型可以用来标识IPV4的地址。

重要的参数：
1. `index` 分析
   `not_analyzed`（默认），设置为该值可以保证该字段能通过检索查询到
   `no`
1. `store` 存储
   `true` 独立存储
   `false`（默认）不存储，从 `_source` 中解析

#### boolean
布尔类型，所有的类型都可以标识布尔类型：
1. `True`: 所有非 `False` 的都是 `True`
1. `False`: 表示该值的有:`false`、`"false"`、`"off"`、`"no"`、`"0"`、`""`、`0`、`0.0`

重要的参数：
1. `index` 分析
   `not_analyzed`（默认），设置为该值可以保证该字段能通过检索查询到
   `no`
1. `store` 存储
   `true` 独立存储
   `false`（默认）不存储，从 `_source` 中解析

## 映射
`mapping` 是用来定义一个文档（`document`），以及它所包含的属性（`field`）是如何存储和索引的。比如，使用 `mapping` 来定义：
1. 哪些字符串属性应该被看做全文本属性（full text fields）；
1. 哪些属性包含数字，日期或者地理位置；
1. 文档中的所有属性是否都能被索引（`_all` 配置）；
1. 日期的格式；
1. 自定义映射规则来执行动态添加属性。

#### 查看 mapping 信息
```
GET bank/_mapping
```

相应数据：

```json
{
  "bank" : {
    "mappings" : {
      "properties" : {
        "account_number" : {
          "type" : "long"
        },
        "address" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "age" : {
          "type" : "long"
        },
        ...
      }
    }
  }
}
```

#### 创建 mapping 信息
**1. 创建索引并指定映射**
```json
PUT /my-index
{
  "mappings": {
    "properties": {
      "age": {
        "type": "integer"
      }, 
      "email": {
        "type": "keyword"
      }, 
      "name": {
        "type": "text"
      }
    }
  }
}
```

响应数据：

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "my-index"
}
```

**2. 添加新的字段映射**
```json
PUT /my-index/_mapping
{
  "properties": {
    "employee-id": {
      "type": "keyword", 
      "index": false
    }
  }
}
```

响应数据：

```json
{
  "acknowledged" : true
}
```

#### 更新 mapping 信息
对于已经存在的映射字段，我们不能更新。更新必须创建新的索引进行数据迁移。

#### 数据迁移
先创建出一个新的索引的正确映射：

```json
PUT /new-index
...
```

然后使用 `_reindex` 进行数据迁移：

```json
POST _reindex
{
   "source": {
      "index": "my-index",
      "type": ""            # 旧版本 ES 的 type 类型 例如：在 bank/customer 中，cusomter 就是 type
   },
   "dest": {
      "index": "new-index"
   }
}
```