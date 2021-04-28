---
title: Maven 手册

categories:
- 后端开发
- Maven 手册

date: 2021-04-27
---

## 指定 Java 版本
在创建新的 Maven module 时, Maven 会默认用 1.5 的版本编译，导致编译出错。这时可以在 `pom.xml` 里面加上以下配置指定 Java 版本。

```xml
<properties>
   <encoding>UTF-8</encoding>
   <java.version>1.8</java.version>
   <maven.compiler.source>1.8</maven.compiler.source>
   <maven.compiler.target>1.8</maven.compiler.target>
   <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```
