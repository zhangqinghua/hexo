---
title: Maven 构建

categories:
- 后端开发
- Maven 手册

date: 2021-04-27
---
#### 打包应用

#### 构建应用到服务器

#### 构建应用到 DockerHub
要将一个应用发布到 Docker 需要经过一系列步骤：
1. 编译项目，打包成为 jar 文件；
2. 构建 Docker 镜像；
3. 上传到 DockerHub；

使用 Maven 的 docker-maven-plugin 插件，我们就可以一键构建发布 Docker 应用了。

## 发布依赖
#### 发布依赖到本地仓库
要发布依赖到本地仓库，只需要执行命令 `mvn install` 即可。

#### 发布依赖到远程仓库
如果我们需要打包依赖到远程仓库供别人使用，需要配置 `distributionManagement` 属性。

```xml
<distributionManagement>  
    <repository>    
        <id>服务id</id>    
        <name>GitHub 账号名称 Apache Maven Packages</name>
        <url>https://maven.pkg.github.com/Github账号名称/用于保存的仓库名称</url>  
    </repository>
</distributionManagement>
```



