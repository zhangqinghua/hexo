---
title: Elasticsearch 初步检索

categories:
- 后端开发
- Elasticsearch 教程

date: 2021-04-18 00:97:99
---
## _cat
#### 查看 ES 健康信息

```bash
$ curl http://localhost:9200/_cat/health
1618762998 16:23:18 docker-cluster green 1 1 3 3 0 0 0 0 - 100.0%
```

#### 查看节点信息
```bash
$ curl http://localhost:9200/_cat/nodes  
172.17.0.5 66 96 1 0.12 0.03 0.01 dilm * 28e6018cc390
```

#### 查看主节点

```bash
$ curl http://localhost:9200/_cat/master
iqIV_kIATDiLy46oUu-U7w 172.17.0.5 172.17.0.5 28e6018cc390
```

#### 查看所有索引
使用 `_cat/indices` 查询所有索引，类似 MySQL 的 `show databases;`

```bash
$ curl http://localhost:9200/_cat/indices
green open .kibana_task_manager_1   z_w2e978RwyfHT8yffAraQ 1 0 2 0 38.2kb 38.2kb
green open .apm-agent-configuration EgVoY1rRQ8e0aIEPRmri-w 1 0 0 0   283b   283b
green open .kibana_1                ZvfrNF4PQGKmCuE8Hl86sA 1 0 8 4 16.5kb 16.5kb
```

## 保存文档
保存一个数据，保存在哪个索引的哪个类型下，指定用哪个唯一标识。

#### POST 保存
POST 新增，如果不指定 id，会自动生成id。指定 id 则会修改这个数据，并新增版本号。
1. 不带 Id，新增操作；
1. 带 Id，且存在此 Id，修改操作；
1. 带 Id，且不存在此 Id，新增操作；

```bash
$ POST http://localhost:9200/customer/external
{
	"name":"John Doe"
}
```

响应数据：

```json
{
	"_index": "customer",            # 对应 MySQL 的数据库   
	"_type": "external",             # 对应 MySQL 的数据表
	"_id": "nQW3G3kBh2g-jSNW1mPf",   # 对应 MySQL 的记录，自动生成的 Id
	"_version": 1,                   # 版本号是1
	"result": "created",             # 是保存操作
	"_shards": {
		"total": 2,
		"successful": 1,
		"failed": 0
	},
	"_seq_no": 2,
	"_primary_term": 1
}
```

多次发送此请求，均是 created 操作。

```json
{
	"_index": "customer",
	"_type": "external",
	"_id": "ngW6G3kBh2g-jSNWY2NB",  # 第二次发送此请求，Id 变了，表示是一个新的数据
	"_version": 1,
	"result": "created",
	"_shards": {
		"total": 2,
		"successful": 1,
		"failed": 0
	},
	"_seq_no": 3,
	"_primary_term": 1
}
```

如果带上 Id，此 Id 不存在，则是新增操作，此 Id 存在，则是 updated 操作。

```bash
$ POST http://localhost:9200/customer/external/ngW6G3kBh2g-jSNWY2NB
{
	"name":"John Doe"
}
```

响应数据：

```json
{
	"_index": "customer",
	"_type": "external",
	"_id": "ngW6G3kBh2g-jSNWY2NB",
	"_version": 2,
	"result": "updated",
	"_shards": {
		"total": 2,
		"successful": 1,
		"failed": 0
	},
	"_seq_no": 4,
	"_primary_term": 1
}
```
#### PUT  保存
第一次发送，是新增操作，后续请求，是更新操作。

```bash
# 对应 MySQL 的数据库、表、记录。
$ PUT http://localhost:9200/customer/external/1
{
	"name":"John Doe"
}
```

响应的元数据：

```json
{
	"_index": "customer",
	"_type": "external",
	"_id": "1",
	"_version": 1,       # 版本号，第一次操作，版本号是1
	"result": "created", # 操作类型，第一次操作，是保存，后面的操作，是更新
	"_shards": {         # 分片信息
		"total": 2,
		"successful": 1,
		"failed": 0
	},
	"_seq_no": 0,
	"_primary_term": 1
}
```

第二次发送响应：

```json
{
	"_index": "customer",
	"_type": "external",
	"_id": "1",
	"_version": 2,       # 第二次操作，版本号变成2
	"result": "updated", # 第二次操作，变成更新
	"_shards": {
		"total": 2,
		"successful": 1,
		"failed": 0
	},
	"_seq_no": 1,
	"_primary_term": 1
}
```
PUT 操作必须带 Id，这是跟 POST 最大的区别。

```bash
$ PUT http://localhost:9200/customer/external
{
	"name":"John Doe"
}
```

响应数据：

```json
{
	"error": "Incorrect HTTP method for uri [/customer/external] and method [PUT], allowed: [POST]",
	"status": 405
}
```

## 查询文档
Elasticsearch 使用 GET 请求来查询文档。

```bash
$ GET http://localhost:9200/customer/external/1
```

响应数据：

```json
{
	"_index": "customer",
	"_type": "external",
	"_id": "1",
	"_version": 2,
	"_seq_no": 1,        # 并发控制字段，每次更新就会 +1，用来做乐观锁
	"_primary_term": 1,  # 同上，主分片重新分配。如重启，就会变化
	"found": true,       # 是否找到了数据
	"_source": {         # 数据的内容
		"name": "John Doe"
	}
}
```

`_seq_no` 和 `_primary_term` 可以用来做并发更新操作，假设我们有 2 个线程要对这条数据进行修改操作。

线程一进行修改：

```bash
$ PUT http://localhost:9200/customer/external/1?if_seq_no=1&if__primary_term=1
{
	"name":"John Doe"
}
```

响应数据：

```json
{
	"_index": "customer",
	"_type": "external",
	"_id": "1",
	"_version": 4,
	"result": "updated",
	"_shards": {
		"total": 2,
		"successful": 1,
		"failed": 0
	},
	"_seq_no": 9,
	"_primary_term": 1
}
```

此时 `_seq_no` 和 `_primary_term` 已经变更了，如果线程二还是持有旧的数据，则会修改失败：

```bash
$ PUT http://localhost:9200/customer/external/1?if_seq_no=1&if__primary_term=1
{
	"name":"John Doe"
}
```

响应数据：

```json
{
	"error": {
		"root_cause": [
			{
				"type": "version_conflict_engine_exception",
				"reason": "[1]: version conflict, required seqNo [8], primary term [1]. current document has seqNo [9] and primary term [1]",
				"index_uuid": "2dqEzaASRGWaJBIpVgMqvg",
				"shard": "0",
				"index": "customer"
			}
		],
		"type": "version_conflict_engine_exception",
		"reason": "[1]: version conflict, required seqNo [8], primary term [1]. current document has seqNo [9] and primary term [1]",
		"index_uuid": "2dqEzaASRGWaJBIpVgMqvg",
		"shard": "0",
		"index": "customer"
	},
	"status": 409
}
```

此时线程二只能重新查询一次数据，然后再更新，跟 CAS 相似。

## 更新文档
前面的 POST 和 PUT 均可以新增和更新文档。但是如果我们希望明确的指定更新文档，则可以使用 _update 请求。

```bash
$ POST http://localhost:9200/customer/external/1/_update
{
   "doc": {             # 嵌套的 doc 属性
      "name":"John Doe"
   }
}
```

响应数据：

```json
{
	"_index": "customer",
	"_type": "external",
	"_id": "1",
	"_version": 4,
	"result": "noop",
	"_shards": {
		"total": 0,
		"successful": 0,
		"failed": 0
	},
	"_seq_no": 9,
	"_primary_term": 1
}
```

重复请求，则返回成功，但是没有更新：

```json
{
	"_index": "customer",
	"_type": "external",
	"_id": "1",
	"_version": 4,          # 不变
	"result": "noop",       # 表示没有操作
	"_shards": {
		"total": 0,
		"successful": 0,
		"failed": 0
	},
	"_seq_no": 9,           # 不变
	"_primary_term": 1      # 不变
}
```

`_update` 请求必须嵌套一层 doc 属性。

```bash
$ POST http://localhost:9200/customer/external/1/_update
{
	"name":"John Doe"
}
```

响应数据：

```json
{
	"error": {
		"root_cause": [
			{
				"type": "version_conflict_engine_exception",
				"reason": "[1]: version conflict, required seqNo [8], primary term [1]. current document has seqNo [9] and primary term [1]",
				"index_uuid": "2dqEzaASRGWaJBIpVgMqvg",
				"shard": "0",
				"index": "customer"
			}
		],
		"type": "version_conflict_engine_exception",
		"reason": "[1]: version conflict, required seqNo [8], primary term [1]. current document has seqNo [9] and primary term [1]",
		"index_uuid": "2dqEzaASRGWaJBIpVgMqvg",
		"shard": "0",
		"index": "customer"
	},
	"status": 409
}
```

## 删除文档
DELETE 请求用来删除文档。

```bash
$ DELETE http://localhost:9200/customer/external/1
```

响应数据：

```json
{
	"_index": "customer",
	"_type": "external",
	"_id": "1",
	"_version": 5,
	"result": "deleted",
	"_shards": {
		"total": 2,
		"successful": 1,
		"failed": 0
	},
	"_seq_no": 10,
	"_primary_term": 1
}
```

再次查找，提示数据不存在：

```bash
$ GET http://localhost:9200/customer/external/1
{
	"_index": "customer",
	"_type": "external",
	"_id": "1",
	"found": false
}
```

我们可以可以来删除整个索引：

```bash
# 查询所有索引，存在 customer
$ curl http://localhost:9200/_cat/indices 
green  open .kibana_task_manager_1       GfxC_x2NQIGGkUEnhlFugA 1 0     2 0 31.5kb 31.5kb
green  open kibana_sample_data_ecommerce 6IRUVk8qRKCO4p8DD5r0tw 1 0  4675 0  4.7mb  4.7mb
green  open .apm-agent-configuration     Su0kz7ljSSW6GkIoUtddug 1 0     0 0   283b   283b
green  open kibana_sample_data_logs      7DeKSAJ3T3KFhm48WEE14A 1 0 14074 0 11.7mb 11.7mb
green  open kibana_sample_data_flights   d_Dw3tLzSWSEqSaL7ziGbw 1 0 13059 0  6.3mb  6.3mb
green  open .kibana_1                    TYaPkgSGRyGhWdg657t4Cg 1 0   158 6    1mb    1mb
yellow open customer                     OUgakvzFRHyZPp0ZmuKlQw 1 1     1 0   230b   230b

# 删除 customer 索引
$ DELETE http://localhost:9200/customer
{
	"acknowledged": true
}

# 查询所有索引，不存在 customer
$ curl http://localhost:9200/_cat/indices 
green  open .kibana_task_manager_1       GfxC_x2NQIGGkUEnhlFugA 1 0     2 0 31.5kb 31.5kb
green  open kibana_sample_data_ecommerce 6IRUVk8qRKCO4p8DD5r0tw 1 0  4675 0  4.7mb  4.7mb
green  open .apm-agent-configuration     Su0kz7ljSSW6GkIoUtddug 1 0     0 0   283b   283b
green  open kibana_sample_data_logs      7DeKSAJ3T3KFhm48WEE14A 1 0 14074 0 11.7mb 11.7mb
green  open kibana_sample_data_flights   d_Dw3tLzSWSEqSaL7ziGbw 1 0 13059 0  6.3mb  6.3mb
green  open .kibana_1                    TYaPkgSGRyGhWdg657t4Cg 1 0   158 6    1mb    1mb
```

> Elasticsearch 没有提供删除 type（类型）的操作。

## 批量 API
Elasticserch 使用 `_bulk` 来做批量操作，语法如下：

```bash
$ POST http://localhost:9200/customer/external/_bulk

{action: {metadata}} \n
{request body} \n
{action: {metadata}} \n
{request body} \n
```

#### 批量新增
批量新增，不指定 Id。

```bash
POST /customer/external/_bulk
{"index":{}}
{"name":"123"}
{"index":{}}
{"name":"456"}

{
  "took" : 8,                             # 花费了8ms
  "errors" : false,                       # 没有发生任何错误
  "items" : [
    {
      "index" : {
        "_index" : "customer",
        "_type" : "external",
        "_id" : "oAXvG3kBh2g-jSNWkmM_",   # 第一条记录Id
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 3,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "customer",
        "_type" : "external",
        "_id" : "oQXvG3kBh2g-jSNWkmM_",   # 第二条记录Id
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 4,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}

GET /customer/external/oAXvG3kBh2g-jSNWkmM_

{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "oAXvG3kBh2g-jSNWkmM_",
  "_version" : 1,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "123"
  }
}
```
批量新增，指定 Id。

```bash
POST /customer/external/_bulk
{"index":{"_id": "1"}}
{"name":"123"}
{"index":{"_id": "2"}}
{"name":"456"}

{
  "took" : 99,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "customer",
        "_type" : "external",
        "_id" : "1",
        "_version" : 3,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 5,
        "_primary_term" : 1,
        "status" : 200
      }
    },
    {
      "index" : {
        "_index" : "customer",
        "_type" : "external",
        "_id" : "2",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 6,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}

GET /customer/external/2

{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "2",
  "_version" : 1,
  "_seq_no" : 6,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "456"
  }
}
```

#### 批量修改
```bash
POST /customer/external/_bulk
# 修改操作，重试3次。
{"update" : {"_index": "website", "_type": "blog", "_id": "123","_retry_on_confict": 3}}	
# 修改内容
{"doc"	 : {"title": "My update blog post"}}		
# 修改操作，重试3次。
{"update" : {"_index": "website", "_type": "blog", "_id": "123","_retry_on_confict": 3}}	
# 修改内容
{"doc"	 : {"title": "My update blog post"}}	
```

#### 批量删除
```bash
POST /customer/external/_bulk
{"delete" : "_index", "website", "_type": "blog", "_id": "123"}		
{"delete" : "_index", "website", "_type": "blog", "_id": "123"}		
```

#### 混合操作
```bash
POST /customer/external/_bulk
# 删除操作
{"delete" : "_index", "website", "_type": "blog", "_id": "123"}									
# 创建操作，和 index 类似
{"create" : {"_index", "website", "_type": "blog", "_id": "123"}}									
{"title"  : "My First blog post"}																			
# 创建操作
{"index"  : {"_index", "website", "_type": "blog"}}				
{"title"  : "My Second blog post"}										
# 修改操作，重试3次。
{"update" : {"_index": "website", "_type": "blog", "_id": "123","_retry_on_confict": 3}}	
# 修改内容
{"doc"	 : {"title": "My update blog post"}}							
```

