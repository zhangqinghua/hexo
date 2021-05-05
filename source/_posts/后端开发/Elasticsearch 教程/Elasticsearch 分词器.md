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

#### config 分析
在 ik 的 config 目录下，有几个文件。
1. IKAnalyzer.cfg.xml：用来配置自定义词库
1. main.dic：ik原生内置的中文词库，总共有27万多条，只要是这些单词，都会被分在一起
1. quantifier.dic：存放了一些单位相关的词
1. suffix.dic：存放了一些后缀
1. surname.dic：中国的姓氏
1. stopword.dic：英文停用词

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210505101256.png)

其中最常用的两个：
1. main.dic：包含了原生的中文词语，会按照这个里面的词语去分词,只要是这些单词，都会被分在一起
1. stopword.dic：包含了英文的停用词 ( 停用词 stop word ,比如 a 、the 、and、 at 、but 等 . 通常像停用词，会在分词的时候，直接被干掉，不会建立在倒排索引中 )

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

## 自定义词库
有一些特殊的流行词，一般不会在 ik 的原生词典 main.dic 里。

举个例子，比如2019年很火的 “盘他”，就无法正确解析：
```
GET _analyze
{
  "text": ["盘他","杠精","脱粉"],
  "analyzer": "ik_max_word"
}
```

响应数据：

```json
{
  "tokens": [
    {
      "token": "盘",
      "start_offset": 0,
      "end_offset": 1,
      "type": "CN_CHAR",
      "position": 0
    },
    {
      "token": "他",
      "start_offset": 1,
      "end_offset": 2,
      "type": "CN_CHAR",
      "position": 1
    },
    ...
  ]
}
```

可以看到使用ik的 `ik_max_word` 分词器还是将每个汉字作为一个 `term`，这个时候去使用这些词语去搜索，效果肯定不是很理想。

#### 配置本地词库
**1. 新建自定义分词库**

在 config 目录下新建一个 custom 文件夹，然后新建一个文件：artisan.dic。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210505114424.png)

将希望不分词的词语放到该文件中，比如：

```
盘他
杠精
脱粉
```

**2. 添加到 ik 的配置文件中**

打开 `IKAnalyzer.cfg.xml` 文件，在 `ext_ditc` 节点添加自定义的扩展字典。当然 ik 本身提供的 `extra_main.dic` 词语更加丰富。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210505114644.png)

**3. 重启 ES，查看分词**
```json
GET _analyze
{
  "text": ["盘他","杠精","脱粉"],
  "analyzer": "ik_max_word"
}
```

响应数据：

```json
{
  "tokens": [
    {
      "token": "盘他",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "盘",
      "start_offset": 0,
      "end_offset": 1,
      "type": "CN_WORD",
      "position": 1
    },
    {
      "token": "他",
      "start_offset": 1,
      "end_offset": 2,
      "type": "CN_CHAR",
      "position": 2
    },
    {
      "token": "杠精",
      "start_offset": 3,
      "end_offset": 5,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "脱粉",
      "start_offset": 6,
      "end_offset": 8,
      "type": "CN_WORD",
      "position": 4
    }
  ]
}
```

可以看到，和未添加自定义词典相比，已经可以按照自己指定的规则进行分词了。

#### 配置远程词库
部署一个web服务器，提供一个http接口，通过modified和tag两个http响应头，来提供词语的热更新。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210505120557.png)

#### 停用自定义词库
比如了、的、啥、么，我们可能并不想去建立索引，让人家搜索。

**1. 新建自定义停用词词典**

我们在新建的目录 `custom` 下新建一个文件：`artisan_stopword.dic`，添加停用词。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210505115414.png)

**2. 添加到 ik 的配置文件中**

在 `ext_stopwords` 节点 添加自定义的停用词扩展字典，ik 本身提供的 `extra_stopword.dic` 这里我们也添加进去吧。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210505115457.png)

**3. 重启 ES，查看停用词**

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210505115942.png)

## 总结
不同的分词器，分词有明显的区别，所以以后定义一个索引不能再使用默认的 `mapping` 了，要手工建立 `mapping`, 因为要选择分词器。