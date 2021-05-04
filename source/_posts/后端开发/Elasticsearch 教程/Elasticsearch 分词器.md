---
title: Elasticsearch 分词器

categories:
- 后端开发
- Elasticsearch 教程

date: 2021-04-18 00:97:94
---
tokenizer（分词器）接收一个字符流，将之分割为独立的 tokens（词元，通常是独立的单词），然后输出 tokens 流。

例如，whitespace tokenizer 遇到空白字符时分割文本，它会将文本 "Quick brown fox!" 分割为 [Quick, brown, fox!]。

tokenizer 还负责记录各个 term（词条）的顺序或 position（位置）（用于 phrase（短语）和 word proximity（词近邻查询）），以及 term（词条）所代表的原始 word（单词）的 start（起始）和 end（结束）的 character offsets（字符偏移量）（用于高亮显示搜索的内容）。

Elasticsearch 提供了很多内置的分词器，可以用来构建 custom analyzers。


## standard tokenizer
standard tokenizer 是 Elasticsearch 提供的默认分词器，它可以将一段英文文本按照空格分词，并去掉末尾符号。

```json
POST _analyze
{
  "analyzer": "standard",  # 使用 standard tokenizer
  "text": "hello world!"   # 要分词的文本
}
```

响应数据：

```json
{
  "tokens" : [
    {
      "token" : "hello",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "world",
      "start_offset" : 6,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 1
    }
  ]
}
```

但是 standard tokenizer 无法处理中文分词，它会把每一个中文当作一个分词：

```json
POST _analyze
{
  "analyzer": "standard",
  "text": "分词器接收一个字符"
}
```

响应数据：

```json
{
  "tokens" : [
    {
      "token" : "分",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "词",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "器",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    ...
  ]
}
```

## ik 分词器
ik 分词器对中文具有良好支持。要安装 ik 分词器，我们只需要到 https://github.com/medcl/elasticsearch-analysis-ik/releases 下载跟 Elasticsearch 对应的版本，然后解压到 Elasticsearch 的 plugins 目录下即可。

判断是否安装成功：
```bash
# 进入 Elasticsearch 容器后，执行命令
[root@28e6018cc390 elasticsearch]# bin/elasticsearch-plugin list
ik-7.4.2
```

注意：MacOS 系统要移除掉 .DS_Store 目录，否则会抛异常 `Exception in thread "main" java.nio.file.FileSystemException: /usr/share/elasticsearch/plugins/.DS_Store/plugin-descriptor.properties: Not a directory`。

```bash
$ ls -a
.DS_Store	ik-7.4.2

$ rm -rf .DS_Store 
```

最后，重启一下 Elasticsearch。

ik 提供了 `ik_smart` 和 `ik_max_word` 两种分词器。

#### 测试 ik_smart 分词

```json
PUT /my-index/_mapping
POST _analyze
{
  "analyzer": "ik_smart",
  "text": "我是中国人"
}
```

响应数据：

```json
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "中国人",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 2
    }
  ]
}
```

#### 测试 ik_max_word 分词
```json
POST _analyze
{
  "analyzer": "ik_max_word",
  "text": "我是中国人"
}
```

响应数据：

```json
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "中国人",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "中国",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "国人",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 4
    }
  ]
}
```
