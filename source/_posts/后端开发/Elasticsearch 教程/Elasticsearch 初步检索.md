---
title: Elasticsearch 初步检索

categories:
- 后端开发
- Elasticsearch 教程

date: 2021-04-18 00:97:99
---
## _cat
#### 查看节点信息
```bash
$ curl http://localhost:9200/_cat/nodes  
172.17.0.5 66 96 1 0.12 0.03 0.01 dilm * 28e6018cc390
```

#### 查看 ES 健康信息

```bash
$ curl http://localhost:9200/_cat/health
1618762998 16:23:18 docker-cluster green 1 1 3 3 0 0 0 0 - 100.0%
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
## 索引一个文档（保存）
保存一个数据，保存在哪个索引的哪个类型下，指定用哪个唯一标识

## 查询文档

## 更新文档

## 删除文档/索引

## bulk 批量 API

## 样本测试数据
我准备了一份顾客银行账户信息的虚构的 [JSON 文档样本](https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true)。每个文档都有下列的 schema（模式）:

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

