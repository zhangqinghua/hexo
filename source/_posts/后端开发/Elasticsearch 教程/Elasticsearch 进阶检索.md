---
title: Elasticsearch 进阶检索

categories:
- 后端开发
- Elasticsearch 教程

date: 2021-04-18 00:97:98
---
## 样本测试数据
Elasticsearch 准备了一份顾客银行账户信息的虚构的 [JSON 文档样本](https://gitee.com/ufo360/picgo2/raw/master/accounts.json)。每个文档都有下列的 schema（模式）:

```json
{
   "account_number": 0,
   "balance": 16623,
   "firstname": "Bradshaw",
   "lastname": "Mckenzie",
   "age": 29,
   "gender": "F",
   "address": "244 Columbus Place", "employer": "Euron",
   "email": "bradshawmckenzie@euron.com", "city": "Hobucken",
   "state": "CO"
}
```

将这份数据导入到 Elasticsearch 中。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210503143906.png)

## Search API
ES 支持两种基本方式检索 :
1. 一个是通过使用 REST Request URI 发送搜索参数（URL + 检索参数）
1. 一个是通过使用 REST Request Body 来发送它们（URL + 请求体）

#### REST Request URI
检索 bank 下所有信息，包括 type 和 docs：

```
GET bank/_search

{
  "took" : 0,                       # 执行搜索的时间（毫秒）
  "timed_out" : false,              # 告诉我们搜索是否超时
  "_shards" : {                     # 告诉我们多少个分片被搜索了，以及统计了成功/失败的搜索分片
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {                        # 搜索结果
    "total" : {
      "value" : 1000,               # 总数
      "relation" : "eq"             # 
    },
    "max_score" : 1.0,              # 最高得分（全文检索用）
    "hits" : [                      # 实际的搜索结果数组（默认为前 10 的文档）
      {
        "_index" : "bank",
        "_type" : "customer",
        "_id" : "1",
        "_score" : 1.0,             # 相关性得分（全文检索用）
        "_source" : {
          "account_number" : 1,
          "balance" : 39225,
          "firstname" : "Amber",
          "lastname" : "Duke",
          "age" : 32,
          "gender" : "M",
          "address" : "880 Holmes Lane",
          "employer" : "Pyrami",
          "email" : "amberduke@pyrami.com",
          "city" : "Brogan",
          "state" : "IL"
        }
      },
      {
        "_index" : "bank",
        "_type" : "customer",
        "_id" : "6",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 6,
          "balance" : 5686,
          "firstname" : "Hattie",
          "lastname" : "Bond",
          "age" : 36,
          "gender" : "M",
          "address" : "671 Bristol Street",
          "employer" : "Netagy",
          "email" : "hattiebond@netagy.com",
          "city" : "Dante",
          "state" : "TN"
        }
      }
      ...
    ]
  }
}
```

带上参数的请求方式：

```
GET bank/_search?q=*&sort=account_number:asc

{
  "took" : 9,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "customer",
        "_id" : "0",
        "_score" : null,
        "_source" : {
          "account_number" : 0,
          "balance" : 16623,
          "firstname" : "Bradshaw",
          "lastname" : "Mckenzie",
          "age" : 29,
          "gender" : "F",
          "address" : "244 Columbus Place",
          "employer" : "Euron",
          "email" : "bradshawmckenzie@euron.com",
          "city" : "Hobucken",
          "state" : "CO"
        },
        "sort" : [                  # 结果的排序 key（键）（没有则按 score 排序）
          0
        ]
      },
      {
        "_index" : "bank",
        "_type" : "customer",
        "_id" : "1",
        "_score" : null,
        "_source" : {
          "account_number" : 1,
          "balance" : 39225,
          "firstname" : "Amber",
          "lastname" : "Duke",
          "age" : 32,
          "gender" : "M",
          "address" : "880 Holmes Lane",
          "employer" : "Pyrami",
          "email" : "amberduke@pyrami.com",
          "city" : "Brogan",
          "state" : "IL"
        },
        "sort" : [
          1
        ]
      }
    ]
  }
}
```

#### REST Request Body
```
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 5,
  "sort": {
    "account_number": {
      "order" : "desc"
    }
  }
}
```

REST Request Body 使用到了 Query DSL。

## Query DSL
Elasticsearch 提供了一个可以执行查询的 JSON 风格的 DSL（domain-specific language 领域特定语言）。这个被称为 Query DSL。该查询语言非常全面，并且刚开始的时候感觉有点复杂，真正学好它的方法是从一些基础的示例开始的。

一个查询语句的典型结构：

```
{
   QUERY_NAME: {
      ARGUMENT: VALUE, 
      ARGUMENT: VALUE,
      ... 
   }
}
```

如果是针对某个字段，那么它的结构如下：

```bash
{
   QUERY_NAME: {
      FIELD_NAME: {
         ARGUMENT: VALUE, 
         ARGUMENT: VALUE,
         ... 
      }
   }
}
```

示例：

```
GET bank/_search
{
  "query": {                        # 定义如何查询
    "match_all": {}                 # 查询类型【代表查询所有的所有】，es 中可以在 query 中组合非常多的查询类型完成复杂查询
  },
  "from": 0,                        # 除了 query 参数之外，我们也可以传递其它的参数以改变查询结果。如 sort，from + size 限定，完成分页功能
  "size": 5,
  "sort": {                         # sort 排序，多字段排序，会在前序字段相等时后续字段内部排序，否则以前序为准
    "account_number": {
      "order" : "desc"
    }
  }
}
```

响应数据：

```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "customer",
        "_id" : "999",
        "_score" : null,
        "_source" : {
          "account_number" : 999,
          "balance" : 6087,
          "firstname" : "Dorothy",
          "lastname" : "Barron",
          "age" : 22,
          "gender" : "F",
          "address" : "499 Laurel Avenue",
          "employer" : "Xurban",
          "email" : "dorothybarron@xurban.com",
          "city" : "Belvoir",
          "state" : "CA"
        },
        "sort" : [
          999
        ]
      },
      {
        "_index" : "bank",
        "_type" : "customer",
        "_id" : "998",
        "_score" : null,
        "_source" : {
          "account_number" : 998,
          "balance" : 16869,
          "firstname" : "Letha",
          "lastname" : "Baker",
          "age" : 40,
          "gender" : "F",
          "address" : "206 Llama Court",
          "employer" : "Dognosis",
          "email" : "lethabaker@dognosis.com",
          "city" : "Dunlo",
          "state" : "WV"
        },
        "sort" : [
          998
        ]
      },
      ...
    ]
  }
}
```

#### 匹配查询「match」
`match` 能够对字段进行匹配查询。

**1. 基本类型（非字符串），精确匹配**

查询出 `account_number` = 20 的数据：

```
GET bank/_search
{
  "query": {
    "match": {
      "account_number": "20"
    }
  }
}
```

**2. 字符串，全文检索**

`match` 当搜索字符串类型的时候，会进行全文检索，并且每条记录有相关性得分。

查询出 `address` 中包含 `mill` 单词的所有记录：
```
GET bank/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  }
}
```

**3. 字符串，多个单词（分词 + 全文检索）**

查询出 `address` 中包含 `mill` 或者 `road` 或者 `mill road` 的所有记录，并给出相关性得分：

```
GET bank/_search
{
  "query": {
    "match": {
      "address": "mill road"
    }
  }
}
```

**4. 短语匹配「match_phrase」**

将需要匹配的值当成一个整体单词（不分词）进行检索。

查出 `address` 中包含` mill road` 的所有记录，并给出相关性得分：

```
GET bank/_search
{
  "query": {
    "match_phrase": {
      "address": "mill road"
    }
  }
}
```

**5. 多字段匹配「multi_match」**

查询 `state` 或者 `address` 包含 `mill` 的记录。

```
GET bank/_search
{
  "query": {
    "multi_match": {
      "query": "mill",
      "fields": ["state", "address"]
    }
  }
}
```

**6. term**

和 `match` 一样。匹配某个属性的值。全文检索字段用 `match`，其他非 `text` 字段匹配用 `term`。

```
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "age": {
              "value": "28"
            }
          }
        }, 
        {
          "match": {
            "address": "990 Mill Road"
          }
        }
      ]
    }
  }
}
```

#### 复合查询「bool」
`bool` 用来做复合查询，它可以合并任何其它查询语句，包括复合语句。了解这一点是很重要的，这就意味着复合语句之间可以互相嵌套，可以表达非常复杂的逻辑。

|事件|描述|
| :- |
|must|子句（查询）必须出现在匹配的文档中，并将有助于得分。|
|must_not|子句（查询）不能出现在匹配的文档中。|
|should|子句（查询）应该出现在匹配的文档中，并将有助于得分。|
|filter|子句（查询）必须出现在匹配的文档中，并将无助于得分。|

**1. must：必须达到列举的所有条件**

```
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "mill"
          }
        }, 
        {
          "match": {
            "gender": "M"
          }
        }
      ]
    }
  }
}
```

**2. must_not：必须不是指定的情况**

`address` 包含 `mill`，并且 `gender` 是 `M`，如果 `address` 里面有 `lane` 最好不过，但是 `email` 必须不包含 `baluba.com`。

```
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "mill"
          }
        }, 
        {
          "match": {
            "gender": "M"
          }
        }
      ], 
      "should": [
        {
          "match": {
            "address": "lane"
          }
        }
      ], 
      "must_not": [
        {
          "match": {
            "email": "baluba.com"
          }
        }
      ]
    }
  }
}
```

**3. should：应该达到列举的条件，如果达到会增加相关文档的评分**
`should` 并不会改变查询的结果。如果 query 中只有 `should` 且只有一种匹配规则，那么 `should` 的条件就会被作为默认匹配条件而去改变查询结果。

```
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "mill"
          }
        }, 
        {
          "match": {
            "gender": "M"
          }
        }
      ], 
      "should": [
        {
          "match": {
            "address": "lane"
          }
        }
      ]
    }
  }
}
```

**4. filter：结果过滤**
并不是所有的查询都需要产生分数，特别是那些仅用于 “filtering”（过滤）的文档。为了不计算分数 Elasticsearch 会自动检查场景并且优化查询的执行。

```
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "mill"
          }
        }
      ], 
      "filter": {
        "range": {
          "balance": {
            "gte": 10000, 
            "lte": 20000
          }
        }
      }
    }
  }
}
```