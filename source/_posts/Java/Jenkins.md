---
title: Jenkins

categories:
- Java

date: 2018-03-27 05:00:00
---

Jenkins是一个开源软件项目，是基于Java开发的一种持续集成工具，用于监控持续重复的工作，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能。

## 使用

1. 下载
    `sudo wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war` 下载到指定目录。

1. 启动
    `sudo java -jar jenkins.war --httpPort=8080` 启动

1. 访问
    `http://localhost:8080`

1. 配置Maven
    首次登录需要配置Maven路径。在`系统管理 > Global Tool Configuration > Maven`中新增一个Maven。取消`自动安装`选项，然后填写Maven目录。

    ![配置Maven路径](001.png)

1. 构建maven项目

    ![新建一个Maven项目](002.png)

1. 配置源码管理源

    ![配置源码地址](003.jpg)

1. 构建触发器
    构建触发器指定了触发一次构建的条件。推荐使用最简单的配置“Poll SCM”，它的意思是，定时检查版本库，发现有新的提交就触发构建。这种方式对git、SVN等所有版本管理系统都是通用的。

    ![构建触发器](004.jpg)

1. 构建
    在`Build`中，默认的Root POM是`pom.xml`。如果`pom.xml`不在根目录下，就填入子目录，例如：`wxapi/pom.xml`。
    在`Goals and options`中，填入需要执行的mvn命令：`clean package`，Jenkins将执行如下命令：
    `mvn clean package`
    特殊参数也在这里填写，如`-DskipTests=true clean package`。

保存后，就可以执行自动化构建了。

点击一个构建任务，可以在Console Output中看到控制台详细输出，便于出错排查：

![控制台详细输出](005.jpg)

## Github自动构建

1. Build when a change is pushed to GitHub 打勾
1. GitHub repository setting里面， webhooks & services