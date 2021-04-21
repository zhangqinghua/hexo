---
title: SpringCloud Gateway

categories:
- SpringCloud
date: 2021-02-17 00:00:01
---

## 附：常见问题
**The following method did not exist: org/springframework/web/reactive/socket/client/ReactorNettyWebSocketClient.setHandlePing(Z)V**
场景：SpringBoot 2.2.3 SpringCloud Hoxton.SR3 启动 gateway 服务报错。
原因：版本不兼容？官网上写了是 SR5以下对应 SpringBoot 2.2.X。
解决：使用最新版 2.2.13 和 SR10。

**predicates 修改了，不生效，总是跳转到 阿里云首页**
场景：predicates -Path修改不生效。
场景：切换不同浏览器能生效。
场景：www.baidu.com 不成功。CDNS、阿里云成功。
原因：未知。
解决：未知。