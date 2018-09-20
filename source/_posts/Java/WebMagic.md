---
title: WebMagic

tags:
- WebMagic
- 爬虫

categories:
- Java

date: 2017-11-26
---

WebMagic是一款简单灵活的爬虫框架。基于它你可以很容易的编写一个爬虫。

## 页面元素的抽取
1. XPath
	```java
	page.getHtml().xpath("//h1[@class='entry-title public']/strong/a/text()")
	```

2. CSS选择器
	```java
	$('h1.entry-title')
	```

3. 正则表达式
	正则表达式则是一种通用的文本抽取语言。
	```java
	page.addTargetRequests(page.getHtml().links().regex("(https://github\\.com/\\w+/\\w+)").all());
	```
4. JsonPath
	JsonPath是于XPath很类似的一个语言，它用于从Json中快速定位一条内容。

## 链接的发现
一个站点的页面是很多的，一开始我们不可能全部列举出来，于是如何发现后续的链接，是一个爬虫不可缺少的一部分。
```java
page.addTargetRequests(page.getHtml().links().regex("(https://github\\.com/\\w+/\\w+)").all());
```