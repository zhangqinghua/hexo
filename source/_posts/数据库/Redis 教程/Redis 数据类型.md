---
title: Redis 数据类型

categories:
- Redis 教程

tags:
- redis

date: 2020-05-28
---
## 常用的 5 种数据结构
key-string;
一个 key 对应一个值。

key-hash
一个 key 对应一个 Map。
一般用来存储一个对象，比如一个 person。 

key-list
一个key对应一个列表
使用list实现队列和栈结构。

key-set
一个key对应一个集合

存储交集、差集和并集的操作。

key-zset
一个key对应一个有序的集合。

排行榜、积分存储等操作。


![](https://th.bing.com/th/id/Rba80f21985a0afb426d5cb7a7e8bed8f?rik=%2biFt%2bXzyYrrCvQ&riu=http%3a%2f%2fwww.runoob.com%2fwp-content%2fuploads%2f2018%2f05%2fredis-data-structure-types.jpeg&ehk=QTIn%2bCgaY1enG9YBtkYWpyvQ2uNS4Z9wdkdqqVPy2fQ%3d&risl=&pid=ImgRaw)

## 另外 3 种数据结构
HyperLogLog
用来计算近似值的。

GEO
地址位置，经纬度

BIT
一般存储的也是一个字符串，存储的是一个 byte[]。