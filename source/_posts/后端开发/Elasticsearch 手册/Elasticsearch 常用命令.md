---
title: Elasticsearch 常用命令

categories:
- 后端开发
- Elasticsearch 手册

date: 2021-04-18 99:99:99
---
全文搜索属于最常见的需求，开源的 Elasticsearch 是目前全文搜索引擎的首选。它可以快速地储存、搜索和分析海量数据。维基百科、Stack Overflow、Github 都采用它。

Elastic 的底层是开源库 Lucene。但是，你没法直接用 Lucene，必须自己写代码去调用它的接口。Elastic 是 Lucene 的封装，提供了 REST API 的操作接口，开箱即用。