---
title: Docker Dockerfile

categories:
- 部署运维
- Docker 手册

date: 2021-04-22
---

#### Java 原始项目
```Dockerfile
## java:8-alpine（145） java8:latest （500m）
FROM java:8
​
# 维者信息
MAINTAINER fage
​
RUN mkdir -p /work /work/config /work/libs /work/logs /work/file
​
EXPOSE  8080
​
ADD  empty.jar  /work/web-discovery-test.jar  
​
WORKDIR   /work
​
CMD ["java","-Dloader.path=/work/libs","-jar","/work/web-discovery-test.jar"]
```

完整版本的 JKD 的打包，体积相当大。

```bash
zhangqinghua$ docker images
REPOSITORY           TAG                  IMAGE ID            CREATED             SIZE
easybyte-auth        0.0.4                cc4a8d3083ca        2 minutes ago       718MB
...

zhangqinghua$ docker history cc4a8d3083ca
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
cc4a8d3083ca        15 seconds ago      /bin/sh -c #(nop)  ENTRYPOINT ["java" "-jar"…   0B                  
48f8a69b72c2        16 seconds ago      /bin/sh -c #(nop)  LABEL test=1                 0B                  
6a7e15d29525        7 minutes ago       /bin/sh -c #(nop) ADD file:557bbfeb265c5f399…   74.7MB              
73a2c0bae7c1        7 minutes ago       /bin/sh -c #(nop)  VOLUME [/tmp]                0B                  
853addbcce7c        7 minutes ago       /bin/sh -c #(nop)  EXPOSE 8080                  0B                  
d23bdf5b1b1b        4 years ago         /bin/sh -c /var/lib/dpkg/info/ca-certificate…   419kB               
<missing>           4 years ago         /bin/sh -c set -x  && apt-get update  && apt…   352MB               
<missing>           4 years ago         /bin/sh -c #(nop)  ENV CA_CERTIFICATES_JAVA_…   0B                  
<missing>           4 years ago         /bin/sh -c #(nop)  ENV JAVA_DEBIAN_VERSION=8…   0B                  
<missing>           4 years ago         /bin/sh -c #(nop)  ENV JAVA_VERSION=8u111       0B                  
<missing>           4 years ago         /bin/sh -c #(nop)  ENV JAVA_HOME=/usr/lib/jv…   0B                  
<missing>           4 years ago         /bin/sh -c {   echo '#!/bin/sh';   echo 'set…   87B                 
<missing>           4 years ago         /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B                  
<missing>           4 years ago         /bin/sh -c echo 'deb http://deb.debian.org/d…   55B                 
<missing>           4 years ago         /bin/sh -c apt-get update && apt-get install…   1.29MB              
<missing>           4 years ago         /bin/sh -c apt-get update && apt-get install…   123MB               
<missing>           4 years ago         /bin/sh -c apt-get update && apt-get install…   44.3MB              
<missing>           4 years ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           4 years ago         /bin/sh -c #(nop) ADD file:89ecb642d662ee7ed…   123MB  
```

#### Java 瘦身项目
```Dockerfile
FROM openjdk:8u212-b04-jre-slim
EXPOSE 8080

VOLUME /tmp

# 复制本地的renren-fast.jar文件到容器/目录并改名app.jar
ADD target/easybyte-auth.jar /app.jar

# 可选：修改app.jar的日期
# RUN bash -c 'touch /app.jar'

LABEL test=1

# 运行命令：java -jar /app.jar
ENTRYPOINT ["java", "-jar", "/app.jar", "--server.port=8080"]
```

使用 `openjdk:8u212-b04-jre-slim` 后，体积小一点。

```bash
zhangqinghua$ docker images
REPOSITORY           TAG                  IMAGE ID            CREATED             SIZE
easybyte-auth        0.0.5                9673188d3935        5 minutes ago       258MB
...

zhangqinghua$ docker history 9673188d3935
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9673188d3935        5 minutes ago       /bin/sh -c #(nop)  ENTRYPOINT ["java" "-jar"…   0B                  
cc2224ba30fe        5 minutes ago       /bin/sh -c #(nop)  LABEL test=1                 0B                  
eaedb4d82e63        5 minutes ago       /bin/sh -c #(nop) ADD file:557bbfeb265c5f399…   74.7MB              
12a668a5bd05        5 minutes ago       /bin/sh -c #(nop)  VOLUME [/tmp]                0B                  
8208d6377858        5 minutes ago       /bin/sh -c #(nop)  EXPOSE 8080                  0B                  
57583b88a4ae        22 months ago       /bin/sh -c set -eux;   dpkgArch="$(dpkg --pr…   105MB               
<missing>           22 months ago       /bin/sh -c #(nop)  ENV JAVA_URL_VERSION=8u21…   0B                  
<missing>           22 months ago       /bin/sh -c #(nop)  ENV JAVA_BASE_URL=https:/…   0B                  
<missing>           22 months ago       /bin/sh -c #(nop)  ENV JAVA_VERSION=8u212-b04   0B                  
<missing>           22 months ago       /bin/sh -c { echo '#/bin/sh'; echo 'echo "$J…   27B                 
<missing>           22 months ago       /bin/sh -c #(nop)  ENV PATH=/usr/local/openj…   0B                  
<missing>           22 months ago       /bin/sh -c #(nop)  ENV JAVA_HOME=/usr/local/…   0B                  
<missing>           22 months ago       /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B                  
<missing>           22 months ago       /bin/sh -c set -eux;  apt-get update;  apt-g…   8.79MB              
<missing>           22 months ago       /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
<missing>           22 months ago       /bin/sh -c #(nop) ADD file:71ac26257198ecf6a…   69.2MB  
```