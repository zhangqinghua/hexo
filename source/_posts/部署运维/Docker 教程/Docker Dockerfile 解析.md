---
title: Docker Dockerfile 解析

categories:
- 部署运维
- Docker 教程

date: 2020-05-30
---
是什么、有什么、怎么做、案例、小总结。

## Dockerfile 简介
Dockerfile 是用来构建 Docker 镜像的构建文件，是由一系列命令和参数构成的脚本。

## Dockerfile 构建过程
Dockerfile 基础内容知识：

1. 每条保留字指令都必须为大写字母且后面要跟随至少一个参数；

1. 指令按照从上到下顺序执行；

1. 每条指令都会创建一个新的镜像层，并对镜像进行提交；

Docker 制定 Dockerfile 的大致流程：
1. Docker 从基础镜像运行一个容器；

1. 执行一条指令并对容器作出修改；

1. 执行类似 `docker commit` 的操作提交一个新的镜像层；

1. Docker 再基于刚提交的镜像运行一个新容器；

1. 执行 Dockerfile 中的下一条指令直到所有指令都执行完毕；

从应用软件的角度来看，Dockerfile、Docker 镜像与 Docker 容器分别代表软件的三个不通阶段：

1. Dockerfile 是软件的原材料；

1. Docker 镜像是软件的交付品；

1. Docker 容器则可以认为是软件的运行态；

其中，Dockerfile 是的面向开发的，Docker 镜像成为交付标准，Docker 容器则涉及部署与运维。三者缺一不可，合力充当 Docker 体系的基石：

1. Dockerfile
   Dockerfile 定义了进程需要的一切东西。

   Dockerfile 涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链库、操作系统的发行版、服务进程的内核进程（当应用进程需要和系统服务和内容进程打交道时需要考虑如何设计 namspace 的权限控制）等等。

1. Docker 镜像
   在用 Dockerfile 定义一个文件后，`docker build` 时会产生一个 Docker 镜像。当运行 Docker 镜像时，会真正开始提供服务。

1. Docker 容器
   Docker 容器是直接提供服务的。

![](https://graph.baidu.com/thumb/v4/3968054224,2923627959.jpg)

## Dockerfile 体系结构
Dockerfile 常用的保留字指令：
1. `FROM`
   基础镜像，当前镜像是基于哪个镜像的。
1. `MAINTAINER`
   镜像维护者的姓名和邮箱地址。
1. `RUN`
   容器构建的时候运行运行的命令。
1. `EXPOSE`
   当前容器对外暴露出的端口。
1. `WORKDIR`
   指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点。
1. `ENV`
   用来在构建镜像过程中设置环境变量。
1. `ADD`
   将宿主机目录下的文件拷贝进镜像且自动处理 URL 和解压 tar 压缩包。
1. `COPY`
   将宿主机目录下的文件拷贝进镜像。
1. `VOLUME`
   容器数据卷，用于数据保存和持久化工作。
1. `CMD`
   指定一个容器启动时要运行的命令。
   
   Dockerfile 中可以有多个 `CMD` 指令，但只有最后一个生效。且 `CMD` 会被 `docker run` 之后的参数替换。
1. `ENTRYPOINT`  
   和 `CMD` 类似，唯一的区别是，`CMD` 会被 `docker run` 之后的参数替换，而 `ENTRYPOINT` 则是追加形式。

1. `ONEBUILD`
   当构建一个带继承的 Dockerfile 时，被继承的 `Dockerfile` 会触发 `onbuild` 命令。

## Dockerfile 总结

```Dockerfile
# base 镜像。
# Docker Hub 中 99% 的镜像都是通过在 base 镜像中安装和配置需要的软件构建出来的。
FROM scrath

ADD cento-7-docker.tar.xz /

LABEL 
```

## 附：构建自定义 Centos
为什么需要自定义 Centos 镜像？
1. 初始 Centos 默认路径是 `/`。
1. 初始 Centos 默认不支持 vim。
1. 初始 Centos 默认不支持 ifconfig；

自定义 Centos 目的是使得我们自己的镜像具备以下功能：
1. 默认路径是 `/tmp`；
1. 支持 vim。
1. 支持 ifconfig；

编写 Dockerfile 文件：

```Dockerfile
# 1. 基于 centos 构建
FROM centos

# 2. 配置作者信息
MAINTAINER zhangqinghua<the.patron.saint.of.science@gmail.com>

# 3. 定义环境变量：默认工作路径
ENV path /tmp

# 4. 安装 vim 和 ifconfig
RUN yum install -y vim && yum install -y net-tools

# 5. 暴露出 80 端口，这里没什么用（没有 Tomcat 或 Nginx 服务）
EXPOSE 80

# 6. 设置工作路径
WORKDIR $path

# 7. 进入打开 bash
CMD /bin/bash
```

构建 Dockerfile 生成镜像：

```bash
# docker build -f 文件名 -t 镜像名:TAG .
zhangqinghua$ docker build -f centos_dockerfile -t mycentos:1.03 .
Sending build context to Docker daemon  2.048kB
Step 1/8 : FROM daocloud.io/centos:latest
...
Removing intermediate container b823edba81f4
 ---> 6c0da28e0bde
Successfully built 6c0da28e0bde
Successfully tagged mycentos:1.03

zhangqinghua$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
mycentos                    1.03                6c0da28e0bde        50 seconds ago      291MB
daocloud.io/library/mysql   latest              c8562eaf9d81        3 weeks ago         546MB
daocloud.io/centos          latest              300e315adb2f        2 months ago        209MB
```

运行镜像并进入容器：

```bash
zhangqinghua$ docker run -it mycentos:1.03
# 已经进行容器内部了
[root@7a724202b2eb tmp]# pwd
/tmp
# 退出并停止容器
[root@7a724202b2eb tmp]# exit 
exit
zhangqinghua$ 
```

列出镜像的变更历史：

```bash
zhangqinghua$ docker history 3bf3cfd1ea86
IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
3bf3cfd1ea86        About a minute ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B                  
07a9dda7087a        About a minute ago   /bin/sh -c #(nop) WORKDIR /tmp                  0B                  
db9117cd3073        About a minute ago   /bin/sh -c #(nop)  EXPOSE 80                    0B                  
2c87c6827d2b        About a minute ago   /bin/sh -c yum install -y vim && yum install…   58.9MB              
58fd6b93f93a        2 minutes ago        /bin/sh -c #(nop)  ENV path=/tmp                0B                  
485ed4774bf3        2 minutes ago        /bin/sh -c #(nop)  MAINTAINER zhangqinghua<t…   0B                  
300e315adb2f        2 months ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           2 months ago         /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B                  
<missing>           2 months ago         /bin/sh -c #(nop) ADD file:bd7a2aed6ede423b7…   209MB   
```

## 附：构建自定义 Centos，使用 ENTRYPOINT 指令
## 腹：构建自定义 Tomcat
Dockerfile 文件的内容如下： 

```bash
FROM ccc7a11d65b1     （这串数字是我已经创建好一个ubuntu镜像的镜像id，在这里作为tomcat的基础镜像
MAINTAINER hmk
ENV REFRESHED_AT 2018-03-10  （这个环境变量用来表名该镜像模板的最后更新时间）

#切换镜像目录，进入/usr目录
WORKDIR /usr
#在/usr/下创建jdk目录,用来存放jdk文件
RUN mkdir jdk
#在/usr/下创建tomcat目录，用来存放tomcat
RUN mkdir tomcat

#将宿主机的jdk目录下的文件拷至镜像的/usr/jdk目录下
ADD jdk1.8.0_131 /usr/jdk/
#将宿主机的tomcat目录下的文件拷至镜像的/usr/tomcat目录下
ADD apache-tomcat-7.0.81 /usr/tomcat/

#设置环境变量
ENV JAVA_HOME=/usr/jdk
ENV JRE_HOME=$JAVA_HOME/jre
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH=/sbin:$JAVA_HOME/bin:$PATH

#公开端口
EXPOSE 8080
#设置启动命令
ENTRYPOINT ["/usr/tomcat/bin/catalina.sh","run"]
```

构建镜像：

```bash
zhangqinghua$ docker build -t jamtur01/tomcat .
```

通过创建好的镜像，启动一个容器：

```bash
zhangqinghua$ docker run -d -p 8080:8080 --name hmk_tomcat jamtur01/tomcat:latest
```

使用 `-v` 参数将 war 包挂载至容器内的 `tomcat/webapps` 目录：

```bash
zhangqinghua$ docker run -d -p 8080:8080 
                         -v /HMK/helloword/webapps/HelloWorld.war:/usr/tomcat/webapps/HelloWorld.war 
                         --name hmk_tomcat jamtur01/tomcat
```

## 附：常见错误
**构建时出现：Error response from daemon: No build stage in current context**
原因：`FROM` 指令没有放在最前面。
解决：`FROM` 指令放在最前面。