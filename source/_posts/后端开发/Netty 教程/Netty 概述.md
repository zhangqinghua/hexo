---
title: Netty 概述

categories:
- 后端开发
- Netty 教程

date: 2021-06-20 10:00:09
---
原生 NIO 存在的问题：
1. NIO 的类库和 API 繁杂，使用麻烦：需要熟练掌握 Selector、ServerSocketChannel、SocketChannel、ByteBuffer等。需要具备其他的额外技能：要熟悉 Java 多线程编程，因为 NIO 编程涉及到 Reactor 模式，你必须对多线程和网络编程非常熟悉，才能编写出高质量的 NIO 程序。
1. 开发工作量和难度都非常大：例如客户端面临断连重连、网络闪断、半包读写、失败缓存、网络拥塞和异常流的处理等等。
1. JDK NIO 的 Bug：例如臭名昭著的 Epoll Bug，它会导致 Selector 空轮询，最终导致 CPU100%。直到 JDK1.7 版本该问题仍旧存在，没有被根本解决。

Netty 对 JDK 自带的 NIO 的 API 进行了封装，解决了上述问题：
1. 设计优雅：适用于各种传输类型的统一 API 阻塞和非阻塞 Socket；基于灵活且可扩展的事件模型，可以清晰地分离关注点；高度可定制的线程模型-单线程，一个或多个线程池。
1. 使用方便：详细记录的 Javadoc，用户指南和示例；没有其他依赖项，JDK5（Netty3.x）或 6（Netty4.x）就足够了。
1. 高性能、吞吐量更高：延迟更低；减少资源消耗；最小化不必要的内存复制。
1. 安全：完整的 SSL/TLS 和 StartTLS 支持。
1. 社区活跃、不断更新：社区活跃，版本迭代周期短，发现的 Bug 可以被及时修复，同时，更多的新功能会被加入。

## Netty 版本说明
1. Netty 版本分为 Netty 3.x 和 Netty 4.x、Netty 5.x
1. 因为 Netty 5 出现重大 bug，已经被官网废弃了，目前推荐使用的是 Netty 4.x的稳定版本
1. 目前在官网可下载的版本 Netty 3.x、Netty 4.0.x 和 Netty 4.1.x
1. 在本套课程中，我们讲解 Netty4.1.x 版本
1. Netty 下载地址：https://bintray.com/netty/downloads/netty/

## Netty 高性能架构设计
