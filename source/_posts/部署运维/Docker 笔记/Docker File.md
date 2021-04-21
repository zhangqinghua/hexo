---
title: Docker Dockerfile

categories:
- Docker 笔记

date: 2020-05-30
---

Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

## 示例
##### 使用 Docker 部署 Spring Boot

```Dockerfile
# 表示使用 Jdk8 环境 为基础镜像，如果镜像不是本地的会从 DockerHub 进行下载
FROM openjdk:8-jdk-alpine

# 指向了一个/tmp的目录，由于 Spring Boot 使用内置的Tomcat容器，Tomcat 默认使用/tmp作为工作目录。这个命令的效果是：在宿主机的/var/lib/docker目录下创建一个临时文件并把它链接到容器中的/tmp目录
VOLUME /tmp

# 拷贝文件并且重命名
ADD spring-boot-docker-1.0.jar app.jar

# 为了缩短 Tomcat 的启动时间，添加java.security.egd的系统属性指向/dev/urandom作为 ENTRYPOINT
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```