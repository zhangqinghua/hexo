---
title: Maven 远程仓库
categories:
- 后端开发
- Maven 手册

date: 2021-04-27
---
当构建一个 Maven 项目时，系统首先检查 pom.xml 文件以确定依赖包的下载位置，执行顺序如下：
1. 从本地资源库中查找并获得依赖包，如果没有，执行第2步。

1. 从 [Maven 默认中央仓库](http://repo1.maven.org/maven2) 中查找并获得依赖包。如果没有，执行第3步。

1. 如果在 pom.xml 中定义了自定义的远程仓库，那么也会在这里的仓库中进行查找并获得依赖包。如果都没有找到，那么 Maven 就会抛出异常。

## 自定义远程仓库
在很多情况下，默认的中央仓库无法满足项目的需求，可能项目需要的构件存在于另外一个远程仓库中，如 Company Maven 仓库。这时，可以在项目 pom.xml 中或 Maven 的 settings.xml 中配置该仓库。





## 远程仓库的配置
```xml
<!-- 配置远程仓库 -->
<repositories>
   <!-- 在repositories元素下，可以使用repository子元素声明一个或者多个远程仓库 -->
   <repository>
      <!-- 仓库声明的唯一id，尤其需要注意的是，Maven自带的中央仓库使用的id为central，如果其他仓库声明也使用该id，就会覆盖中央仓库的配置 -->
      <id>jboss</id>
      <!-- 仓库的名称，让我们直观方便的知道仓库是哪个，暂时没发现其他太大的含义 -->
      <name>JBoss Repository</name>
      <!-- 指向了仓库的地址，一般来说，该地址都基于http协议，Maven用户都可以在浏览器中打开仓库地址浏览构件 -->
      <url>http://repository.jboss.com/maven2/</url>
      <!-- 用来控制Maven对于发布版构件和快照版构件的下载权限。需要注意的是enabled子元素，该例中releases的enabled值为true，表示开启JBoss仓库的发布版本下载支持，而snapshots的enabled值为false，表示关闭JBoss仓库的快照版本的下载支持。根据该配置，Maven只会从JBoss仓库下载发布版的构件，而不会下载快照版的构件。 -->
      <releases>
            <enabled>true</enabled>
            <!-- 对于releases和snapshots来说，除了enabled，它们还包含另外两个子元素updatePolicy和checksumPolicy。 -->
            <!-- 元素updatePolicy用来配置Maven从远处仓库检查更新的频率，默认值是daily，表示Maven每天检查一次。其他可用的值包括：never-从不检查更新；always-每次构建都检查更新；interval：X-每隔X分钟检查一次更新（X为任意整数）。 -->
            <updatePolicy>daily</updatePolicy>
      </releases>

      <snapshots>
            <enabled>false</enabled>
            <!-- 元素checksumPolicy用来配置Maven检查校验和文件的策略。当构建被部署到Maven仓库中时，会同时部署对应的检验和文件。在下载构件的时候，Maven会验证校验和文件，如果校验和验证失败，当checksumPolicy的值为默认的warn时，Maven会在执行构建时输出警告信息，其他可用的值包括：fail-Maven遇到校验和错误就让构建失败；ignore-使Maven完全忽略校验和错误。 -->
            <checksumPolicy>warn</checksumPolicy>
      </snapshots>
      <!-- 元素值default表示仓库的布局是Maven2及Maven3的默认布局，而不是Maven1的布局。基本不会用到Maven1的布局。 -->
      <layout>default</layout>
   </repository>
</repositories>
```

## 远程仓库的认证
大部分公共的远程仓库无须认证就可以直接访问，但我们在平时的开发中往往会架设自己的 Maven 远程仓库，出于安全方面的考虑，我们需要提供认证信息才能访问这样的远程仓库。配置认证信息和配置远程仓库不同，远程仓库可以直接在 pom.xml 中配置，但是认证信息必须配置在 settings.xml 文件中。这是因为 pom 往往是被提交到代码仓库中供所有成员访问的，而 settings.xml 一般只存在于本机。因此，在 settings.xml 中配置认证信息更为安全。

```xml
<settings>
 	<!--配置远程仓库认证信息-->
    <servers>
        <server>
            <id>releases</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
   </servers>
</settings>
```

上面代码我们配置了一个 id 为 releases 的远程仓库认证信息。Maven 使用 settings.xml 文件中的 servers 元素及其子元素 server 配置仓库认证信息。认证用户名为 admin，认证密码为 admin123。这里的关键是 id 元素，settings.xml 中 `server` 元素的 id 必须与 pom.xml 中需要认证的 `repository` 元素的 id 完全一致。正是这个 id 将认证信息与仓库配置联系在了一起。

## 常用的远程仓库
远程仓库一般是国内镜像以及用nexus私有仓库居多。在 pom.xml 配置远程仓库时，顺序也是关键点，是从上往下开始查找的。

在 pom.xml 的 `repositories` 节点上添加远程仓库地址，下面整理了一份比较常用的国内远程仓库地址。

```xml
<!-- 设定远程主仓库，按设定顺序进行查找。 -->
<repositories>

   <!-- 如有Nexus私服, 取消注释并指向正确的服务器地址.
   <repository>
      <id>nexus-repos</id>
      <name>Team Nexus Repository</name>
      <url>http://192.168.11.36:8888/nexus/content/groups/public</url>
   </repository> -->
   
   <repository>
      <id>oschina-repos</id>
      <name>Oschina Releases</name>
      <url>http://maven.oschina.net/content/groups/public</url>
   </repository>
   
   <repository>
      <id>aliyun-repos</id>
      <name>aliyun Releases</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
   </repository>

   <repository>
      <id>java-repos</id>
      <name>Java Repository</name>
      <url>https://maven.java.net/content/repositories/public/</url>
   </repository>
   
   <repository>
      <id>JBoss-repos</id>
      <name>JBoss Repository</name>
      <url>http://repository.jboss.org/nexus/content/groups/public/</url>
   </repository>

   <repository>
      <id>springsource-repos</id>
      <name>SpringSource Repository</name>
      <url>http://repo.spring.io/release/</url>
   </repository>
   
   <repository>
      <id>central-repos</id>
      <name>Central Repository</name>
      <url>http://repo.maven.apache.org/maven2</url>
   </repository>
   
   <repository>
      <id>central-repos2</id>
      <name>Central Repository 2</name>
      <url>http://repo1.maven.org/maven2/</url>
   </repository>
   
   <repository>
      <id>activiti-repos</id>
      <name>Activiti Repository</name>
      <url>https://maven.alfresco.com/nexus/content/groups/public</url>
   </repository>
   
   <repository>
      <id>activiti-repos2</id>
      <name>Activiti Repository 2</name>
      <url>https://app.camunda.com/nexus/content/groups/public</url>
   </repository>
   
   <repository> 
      <id>easonjim-repos</id> 
      <name>EasonJim Repository</name>
      <url>https://raw.github.com/easonjim/repository/master</url>
   </repository>
   
</repositories>
```

## 远程仓库的镜像
如果仓库 X 可以提供仓库 Y 存储的所有内容，那么就可以认为X是Y的一个镜像。换句话说，任何一个可以从仓库 Y 获得的构件，都能够从它的镜像中获取。举个例子，http://maven.oschina.net/content/groups/public/ 是中央仓库 http://repo1.maven.org/maven2/ 在中国的镜像，由于地理位置的因素，该镜像往往能够提供比中央仓库更快的服务。因此，可以配置 Maven 使用该镜像来替代中央仓库。编辑 settings.xml，代码如下：

```xml
<mirrors>
     <mirror>
      <id>maven.oschina.net</id>
      <name>maven mirror in China</name>
      <url>http://maven.oschina.net/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
</mirrors>
```

该例中，`mirrorOf` 的值为 central，表示该配置为中央仓库的镜像，任何对于中央仓库的请求都会转至该镜像，用户也可以使用同样的方法配置其他仓库的镜像。id 表示镜像的唯一标识符，`name` 表示镜像的名称，`url` 表示镜像的地址。

关于镜像的一个更为常见的用法是结合私服。由于私服可以代理任何外部的公共仓库(包括中央仓库)，因此，对于组织内部的 Maven 用户来说，使用一个私服地址就等于使用了所有需要的外部仓库，这可以将配置集中到私服，从而简化 Maven 本身的配置。在这种情况下，任何需要的构件都可以从私服获得，私服就是所有仓库的镜像。这时，可以配置这样的一个镜像：

```xml
<!--配置私服镜像-->
<mirrors> 
    <mirror>  
        <id>nexus</id>  
        <name>internal nexus repository</name>  
        <url>http://183.238.2.182:8081/nexus/content/groups/public/</url>  
        <mirrorOf>*</mirrorOf>  
    </mirror>  
</mirrors>
```

该例中 `<mirrorOf>` 的值为星号，表示该配置是所有 Maven 仓库的镜像，任何对于远程仓库的请求都会被转至 http://183.238.2.182:8081/nexus/content/groups/public/。如果该镜像仓库需要认证，则配置一个 `id` 为 nexus 的认证信息即可。

需要注意的是，由于镜像仓库完全屏蔽了被镜像仓库，当镜像仓库不稳定或者停止服务的时候，Maven 仍将无法访问被镜像仓库，因而将无法下载构件。

下面是一份常用的 Maven 镜像仓库：

```xml
<mirror>    
   <id>repo2</id>    
      <mirrorOf>central</mirrorOf>    
      <name>Human Readable Name for this Mirror.</name>    
      <url>http://repo2.maven.org/maven2/</url>    
   </mirror>

   <mirror>    
      <id>ui</id>    
      <mirrorOf>central</mirrorOf>    
      <name>Human Readable Name for this Mirror.</name>    
      <url>http://uk.maven.org/maven2/</url>    
   </mirror>


   <mirror>    
      <id>ibiblio</id>    
      <mirrorOf>central</mirrorOf>    
      <name>Human Readable Name for this Mirror.</name>    
      <url>http://mirrors.ibiblio.org/pub/mirrors/maven2/</url>    
   </mirror>

   <mirror>    
      <id>jboss-public-repository-group</id>    
      <mirrorOf>central</mirrorOf>    
      <name>JBoss Public Repository Group</name>    
      <url>http://repository.jboss.org/nexus/content/groups/public</url>    
   </mirror>

   <mirror>    
      <id>JBossJBPM</id>   
      <mirrorOf>central</mirrorOf>   
      <name>JBossJBPM Repository</name>   
      <url>https://repository.jboss.org/nexus/content/repositories/releases/</url>  
   </mirror>
</mirror>
```

## 部署到远程仓库
我们使用自己的远程仓库的目的就是在远程仓库中部署我们自己项目的构件以及一些无法从外部仓库直接获取的构件。这样才能在开发时，供其他对团队成员使用。

Maven 除了能对项目进行编译、测试、打包之外，还能将项目生成的构件部署到远程仓库中。首先，需要编辑项目的 pom.xml 文件。配置 `distributionManagement` 元素，代码如下：

```xml
<distributionManagement>
        <repository>
            <id>releases</id>
            <name>public</name>
            <url>http://59.50.95.66:8081/nexus/content/repositories/releases</url>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <name>Snapshots</name>
            <url>http://59.50.95.66:8081/nexus/content/repositories/snapshots</url>
        </snapshotRepository>
</distributionManagement>
```

`distributionManagement` 包含 `repository` 和 `snapshotRepository` 子元素，前者表示发布版本（稳定版本）构件的仓库，后者表示快照版本（开发测试版本）的仓库。这两个元素都需要配置 `id`、`name` 和 `url`，`id` 为远程仓库的唯一标识，`name` 是为了方便人阅读，关键的 `url` 表示该仓库的地址。

往远程仓库部署构件的时候，往往需要认证，配置认证的方式同上。

配置正确后，运行命令 `mvn clean deploy`，Maven 就会将项目构建输出的构件部署到配置对应的远程仓库，如果项目当前的版本是快照版本，则部署到快照版本的仓库地址，否则就部署到发布版本的仓库地址。

## 附加说明
#### 配置阿里云远程仓库
有两种方式可以配置阿里云的远程仓库：
1. 修改 Maven 配置文件 settings.xml；
2. 修改项目的配置文件 pom.xml；

**1. 修改 Maven 配置文件**

找到 Maven 的配置文件 `.m2/settings.xml`，在 `<mirrors></mirrors>` 标签中间插入镜像的配置参数。

```xml
<mirrors>
   <mirror>
      <id>aliyunmaven</id>
      <mirrorOf>central</mirrorOf>
      <name>central库</name>
      <url>https://maven.aliyun.com/repository/central</url>
   </mirror>
</mirrors>
```

**2. 修改项目的 pom.xml 文件**

打开你的项目，找到 pom.xml 文件，在 `<repositories></repositories>` 中添加 `<repository></repository`> 仓库，格式如下。仓库的地址同第一种方法中的仓库地址。

```xml
<repositories>
        <repository>
            <id>maven-ali</id>
            <url>https://maven.aliyun.com/repository/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
                <checksumPolicy>fail</checksumPolicy>
            </snapshots>
        </repository>
</repositories>
```

#### 配置阿里云的企业私有仓库

#### 配置私服的 Nexut 私有仓库