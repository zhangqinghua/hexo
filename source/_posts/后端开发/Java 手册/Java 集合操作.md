---
title: Java 集合操作

categories:
- 后端开发
- Java 手册

date: 2021-05-07
---

## 排序
#### 简单类型排序
#### 对象类型排序
```java
businessTimes.sort((a, b) -> a.getFinishTime().isAfter(b.getFinishTime()) ? 1 : 0);

Collections.sort(businessTimes, (a, b) -> a.getFinishTime().isAfter(b.getFinishTime()) ? 1 : 0);
```