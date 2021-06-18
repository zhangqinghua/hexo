---
title:  Filebeat 常用操作

categories:
- 后端开发
- Filebeat 教程

date: 2021-05-06
---

#### 传送多行日志
参考：[Beats：使用 Filebeat 传送多行日志](https://elasticstack.blog.csdn.net/article/details/106272704)

堆栈跟踪是引发异常时应用程序处于中间的一系列方法调用。 堆栈跟踪包括遇到错误的相关行以及错误本身。 可以在此处查看 Java 堆栈跟踪的示例：

```
Exception in thread "main" java.lang.NullPointerException
        at com.example.myproject.Book.getTitle(Book.java:16)
        at com.example.myproject.Author.getBookTitles(Author.java:25)
        at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
```

当使用类似 Elastic Stack 的日志记录工具时，如果没有正确的配置，可能很难识别和搜索堆栈跟踪。 使用像 Filebeat 这样的开源轻型日志摄入器发送应用程序日志时，堆栈跟踪的每一行在 Kibana 中都将被视为单个文档。

因此，上面的堆栈跟踪将在 Kibana 中视为四个单独的文档。 这使得在堆栈跟踪中搜索和理解错误和异常变得很困难，因为它们与它们的上下文脱离了共同的事件。 使用 Filebeat 记录应用程序日志时，用户可以通过在 `filebeat.yml` 文件中添加配置选项来避免此问题。

```conf
multiline.pattern: '^[[:space:]]'
multiline.negate: false
multiline.match: after
```

