---
title: Elasticsearch 复杂检索

categories:
- 后端开发
- Elasticsearch 教程

date: 2021-04-18 00:97:99
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


