---
title: Maven

categories:
- VCS

tags:
- Maven

date: 2018-12-14
---

Maven 是 Apache 软件基金会组织维护的一款自动化构建工具，专注服务于 Java 平台的项目构建和依赖管理。Maven 这个单词的本意是：专家，内行。读音是['meɪv(ə)n]或['mevn]。

Maven能帮助我们处理一下问题：
1. 添加第三方 jar 包
1. 管理 jar 包之间的依赖关系
1. 将项目拆分成多个工程模块

<!-- more -->

## 什么是构建
构建并不是创建，创建一个工程并不等于构建一个项目。要了解构建的含义我们应该由浅入深的从以下三个层面来看：
1. 纯 Java 代码
    Java 是一门编译型语言，.java 扩展名的源文件需要编译成 .class 扩展名的字节码文件才能够执行。所以编写任何 Java 代码想要执行的话就必须经过编译得到对应的 .class 文件。
1. Web 工程
    当我们需要通过浏览器访问 Java 程序时就必须将包含 Java 程序的 Web 工程编译的结果“拿”到服务器上的指定目录下，并启动服务器才行。这个“拿”的过程我们叫部署。
    我们可以将未编译的 Web 工程比喻为一只生的鸡，编译好的 Web 工程是一只煮熟的鸡，编译部署的过程就是将鸡炖熟。
    Web 工程和其编译结果的目录结构对比见下图：
    ![](000.png)
1. 实际项目
    在实际项目中整合第三方框架，Web 工程中除了 Java 程序和 JSP 页面、图片等静态资源之外，还包括第三方框架的 jar 包以及各种各样的配置文件。所有这些资源都必须按照正确的目录结构部署到服务器上，项目才可以运行。

所以综上所述：构建就是以我们编写的 Java 代码、框架配置文件、国际化等其他资源文件、JSP 页面和图片等静态资源作为“原材料”，去“生产”出一个可以运行的项目的过程。

### 构建过程的几个主要环节
1. 清理：删除以前的编译结果，为重新编译做好准备。
1. 编译：将 Java 源程序编译为字节码文件。
1. 测试：针对项目中的关键点进行测试，确保项目在迭代开发过程中关键点的正确性。
1. 报告：在每一次测试后以标准的格式记录和展示测试结果。
1. 打包：将一个包含诸多文件的工程封装为一个压缩文件用于安装或部署。Java 工程对应 jar 包，Web 工程对应 war 包。
1. 安装：在 Maven 环境下特指将打包的结果——jar 包或 war 包安装到本地仓库中。
1. 部署：将打包的结果部署到远程仓库或将 war 包部署到服务器上运行。

### 自动化构建
一般程序员处理 Bug 流程是这样子的：
![](001.png)

从中我们发现，程序员的很大一部分时间花在了“编译、打包、部署、测试”这些程式化的工作上面，而真正需要由“人”的智慧实现的分析问题和编码却只占了很少一部分。
![](002.png)

能否将这些程式化的工作交给机器自动完成呢？——当然可以！这就是自动化构建。
![](003.png)

此时 Maven 的意义就体现出来了，它可以自动的从构建过程的起点一直执行到终点：
![](004.png)

### Maven 核心概念
Maven 能够实现自动化构建是和它的内部原理分不开的，这里我们从 Maven 的九个核心概念入手，看看 Maven 是如何实现自动化构建的：
1. POM
1. 约定的目录结构
1. 坐标
1. 依赖管理
1. 仓库管理
1. 生命周期
1. 插件和目标
1. 继承
1. 聚合

Maven 的核心程序中仅仅定义了抽象的生命周期，而具体的操作则是由 Maven 的插件来完成的。可是 Maven 的插件并不包含在 Maven 的核心程序中，在首次使用时需要联网下载。

下载得到的插件会被保存到本地仓库中。本地仓库默认的位置是：~\.m2\repository。

## 约定的目录结构
约定的目录结构对于 Maven 实现自动化构建而言是必不可少的一环，就拿自动编译来说，Maven 必须能找到 Java 源文件，下一步才能编译，而编译之后也必须有一个准确的位置保持编译得到的字节码文件。

我们在开发中如果需要让第三方工具或框架知道我们自己创建的资源在哪，那么基本上就是两种方式：
1. 通过配置的形式明确告诉它
1. 基于第三方工具或框架的约定

Maven 对工程目录结构的要求就属于后面的一种。

现在 JavaEE 开发领域普遍认同一个观点：约定 > 配置 > 编码。意思就是能用配置解决的问题就不编码，能基于约定的就不进行配置。而 Maven 正是因为指定了特定文件保存的目录才能够对我们的 Java 工程进行自动化构建。

## POM
Project Object Model，即项目对象模型，将 Java 工程的相关信息封装为对象作为便于操作和管理的模型，是 Maven 工程的核心配置。可以说学习 Maven 就是学习 pom.xml 文件中的配置。

## 坐标
使用如下三个向量在 Maven 的仓库中唯一的确定一个 Maven 工程。

```xml
<!-- 公司或组织的域名倒序+当前项目名称 -->
<groupId>com.atguigu.maven</groupId>
<!-- 当前项目的模块名称 -->
<artifactId>Hello</artifactId>
<!-- 当前模块的版本 -->
<version>0.0.1-SNAPSHOT</version>
```

我们可以将这三个向量连起来作为目录结构到仓库中查询到对应的工程：`com/atguigu/maven/Hello/0.0.1-SNAPSHOT/Hello-0.0.1-SNAPSHOT.jar`。

> 注意：我们自己的 Maven 工程必须执行安装操作才会进入仓库。安装的命令是：`mvn install`。

## 依赖
Maven 中最关键的部分，我们使用 Maven 最主要的就是使用它的依赖管理功能。

### 依赖的目的是什么
当 A jar 包用到了 B jar 包中的某些类时，A 就对 B 产生了依赖，这是概念上的描述。那么如何在项目中以依赖的方式引入一个我们需要的 jar 包呢？

答案非常简单，就是使用 dependency 标签指定被依赖 jar 包的坐标就可以了。

```xml
<dependency>
    <groupId>com.atguigu.maven</groupId>
    <artifactId>Hello</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <scope>compile</scope>
</dependency>
```

### 依赖的范围
大家注意到上面的依赖信息中除了目标 jar 包的坐标还有一个 `scope` 设置，这是依赖的范围。依赖的范围有几个可选值，我们用得到的是：`compile`、`test`、`provided` 三个。

1. 从项目结构角度理解 `compile` 和 `test` 的区别：
    结合具体例子：对于 HelloFriend 来说，Hello 就是服务于主程序的，`junit` 是服务于测试程序的。
    HelloFriend 主程序需要 Hello 是非常明显的，测试程序由于要调用主程序所以也需要 Hello，所以 `compile` 范围依赖对主程序和测试程序都应该有效。
    HelloFriend 的测试程序部分需要 `junit` 也是非常明显的，而主程序是不需要的，所以 `test` 范围依赖仅仅对于主程序有效。
    ![](005.png)
1. 从开发和运行这两个不同阶段理解 `compile` 和 `provided` 的区别：
    ![](006.png)
1. 有效性总结：
    ![](007.png)

### 依赖的传递性
A 依赖 B，B 依赖 C，A 能否使用 C 呢？那要看 B 依赖 C 的范围是不是 `compile`，如果是则可用，否则不可用。

![](008.png)

### 依赖的排除
如果我们在当前工程中引入了一个依赖是 A，而 A 又依赖了 B，那么 Maven 会自动将 A 依赖的 B 引入当前工程，但是个别情况下 B 有可能是一个不稳定版，或对当前工程有不良影响。这时我们可以在引入 A 的时候将 B 排除。

1. 情景举例：
    ![](009.png)
1. 配置方式：
    ```xml
    <dependency>
        <groupId>com.atguigu.maven</groupId>
        <artifactId>HelloFriend</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <type>jar</type>
        <scope>compile</scope>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    ```
1. 排除后的效果：
    ![](010.png)

### 统一管理所依赖 jar 包的版本
对同一个框架的一组 jar 包最好使用相同的版本。为了方便升级框架，可以将 jar 包的版本信息统一提取出来：
1. 统一声明版本号
    ```xml
    <properties>
        <atguigu.spring.version>4.1.1.RELEASE</atguigu.spring.version>
    </properties>
    ```
    其中 `atguigu.spring.version` 部分是自定义标签。
1. 引用前面声明的版本号
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${atguigu.spring.version}</version>
    </dependency>
    ……
</dependencies>
```
1. 其他用法
    ```xml
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    ```

### 依赖的原则：解决 jar 包冲突
1. 路径最短者优先
    ![](011.png)

1. 路径相同时先声明者优先
    ![](012.png)


## 仓库
仓库可分为本地仓库和远程仓库两种：
- 本地仓库：为当前本机电脑上的所有 Maven 工程服务。
- 远程仓库：
    - 私服：架设在当前局域网环境下，为当前局域网范围内的所有 Maven 工程服务。
    - 中央仓库：架设在 Internet 上，为全世界所有 Maven 工程服务。
    - 中央仓库的镜像：架设在各个大洲，为中央仓库分担流量。减轻中央仓库的压力，同时更快的响应用户请求。

仓库中的文件包括：
- Maven 的插件
- 我们自己开发的项目的模块
- 第三方框架或工具的 jar 包

> 不管是什么样的 jar 包，在仓库中都是按照坐标生成目录结构，所以可以通过统一的方式查询或依赖。

## 生命周期
Maven 生命周期定义了各个构建环节的执行顺序，有了这个清单，Maven 就可以自动化的执行构建命令了。

Maven 有三套相互独立的生命周期，分别是：
1. **Clean Lifecycle** 在进行真正的构建之前进行一些清理工作。
2. **Default Lifecycle** 构建的核心部分，编译，测试，打包，安装，部署等等。
3. **Site Lifecycle** 生成项目报告，站点，发布站点。

它们是相互独立的，你可以仅仅调用 clean 来清理工作目录，仅仅调用 site 来生成站点。当然你也可以直接运行 `mvn clean install site` 运行所有这三套生命周期。

每套生命周期都由一组阶段组成，我们平时在命令行输入的命令总会对应于一个特定的阶段。比如，运行 `mvn clean`，这个 clean 是 Clean 生命周期的一个阶段。有 Clean 生命周期，也有 `clean` 阶段。

### Clean 生命周期
Clean 生命周期一共包含了三个阶段：
1. `pre-clean` 执行一些需要在 clean 之前完成的工作
1. `clean` 移除所有上一次构建生成的文件
1. `post-clean` 执行一些需要在 clean 之后立刻完成的工作

### Default 生命周期
Default 生命周期是 Maven 生命周期中最重要的一个，绝大部分工作都发生在这个生命周期中。这里，只解释一些比较重要和常用的阶段：
1. `validate`
1. `generate-sources`
1. `process-sources`
1. `generate-resources`
1. `process-resources` 复制并处理资源文件，至目标目录，准备打包。
1. `compile` 编译项目的源代码。
1. `process-classes`
1. `generate-test-sources`
1. `process-test-sources`
1. `generate-test-resources`
1. `process-test-resources` 复制并处理资源文件，至目标测试目录。
1. `test-compile` 编译测试源代码。
1. `process-test-classes`
1. `test` 使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。
1. `prepare-package`
1. `package` 接受编译好的代码，打包成可发布的格式，如 JAR。
1. `pre-integration-test`
1. `integration-test`
1. `post-integration-test`
1. `verify`
1. `install` 将包安装至本地仓库，以让其它项目依赖。
1. `deploy` 将最终的包复制到远程的仓库，以让其它开发人员与项目共享或部署到服务器上运行。

### Site 生命周期
1. `pre-site` 执行一些需要在生成站点文档之前完成的工作
1. `site` 生成项目的站点文档
1. `post-site` 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
1. `site-deploy` 将生成的站点文档部署到特定的服务器上

这里经常用到的是 `site` 阶段和 `site-deploy` 阶段，用以生成和发布 Maven 站点，这可是 Maven 相当强大的功能，Manager 比较喜欢，文档及统计数据自动生成，很好看。

### 生命周期与自动化构建
**运行任何一个阶段的时候，它前面的所有阶段都会被运行。**例如我们运行 `mvn install` 的时候，代码会被编译，测试，打包。这就是 Maven 为什么能够自动执行构建过程的各个环节的原因。此外，Maven 的插件机制是完全依赖 Maven 的生命周期的，因此理解生命周期至关重要。


## 插件和目标
- Maven 的核心仅仅定义了抽象的生命周期，具体的任务都是交由插件完成的
- 每个插件都能实现多个功能，每个功能就是一个插件目标
- Maven 的生命周期与插件目标相互绑定，以完成某个具体的构建任务

例如：compile 就是插件 maven-compiler-plugin 的一个目标；pre-clean 是插件 maven-clean-plugin 的一个目标。

## 继承
由于非 `compile` 范围的依赖信息是不能在“依赖链”中传递的，所以有需要的工程只能单独配置。

**Hello Project**
```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.0</version>
    <scope>test</scope>
</dependency>
```
**HelloFriend Project**
```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.0</version>
    <scope>test</scope>
</dependency>
```

**MakeFriend Project**
```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.0</version>
    <scope>test</scope>
</dependency>
```

此时如果项目需要将各个模块的 `junit` 版本统一为 4.9，那么到各个工程中手动修改无疑是非常不可取的。

使用继承机制就可以将这样的依赖信息统一提取到父工程模块中进行统一管理：
1. 创建父工程
    创建父工程和创建一般的 Java 工程操作一致，唯一需要注意的是：打包方式处要设置为 `pom`。
1. 在父工程中管理依赖
    ```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.9</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    ```
1. 在子工程中引用父工程
    此时如果子工程的 `groupId` 和 `version` 如果和父工程重复则可以删除。
    ```xml
    <parent>
        <groupId>com.atguigu.maven</groupId>
        <artifactId>Parent</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <!-- 指定从当前子工程的pom.xml文件出发，查找父工程的pom.xml的路径 -->
        <relativePath>../Parent/pom.xml</relativePath>
    </parent>
    ```
1. 在子项目中重新指定需要的依赖，删除范围和版本号
    ```xml
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
    </dependencies>
    ```

## 聚合
将多个工程拆分为模块后，需要手动逐个安装到仓库后依赖才能够生效。修改源码后也需要逐个手动进行 clean 操作。而使用了聚合之后就可以批量进行 Maven 工程的安装、清理工作。 

在总的聚合工程中使用 modules/module 标签组合，指定模块工程的相对路径即可。

```xml
<modules>
    <module>../Hello</module>
    <module>../HelloFriend</module>
    <module>../MakeFriends</module>
</modules> 
```