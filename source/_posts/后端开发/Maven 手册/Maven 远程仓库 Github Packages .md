---
title: Maven 远程仓库 GitHub's repo
categories:
- 后端开发
- Maven 手册

date: 2021-04-27
---

## 配置 Token
在 Guthub -> Settings -> Developer settings -> Personal access tokens 创建一个用于上传依赖的 Token。需要勾选 repo 和 user 选项。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210627154807.png)

## 配置 server
找到 Maven 的配置文件 `.m2/settings.xml`，添加以下内容：

```xml
<settings>
   <servers>
      <server>
         <id>github</id>
         <!-- GitHub登录名 -->
         <username>the.patron.saint.of.science@gmail.com</username>
         <!-- GitHub Token -->
         <password>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</password>
      </server>
   </servers>
</settings>
```

## 配置 distributionManagement
在项目的 pom.xml 上添加以下配置。

```xml
<project>
   <distributionManagement>
      <repository>
         <id>maven.repo</id>
         <name>Local Staging Repository</name>
         <url>file://${project.build.directory}/mvn-repo</url>
      </repository>
   </distributionManagement>
</project>
```

它的作用是将依赖安装在本地，以便后面我们用来上传到 GitHub。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210627155324.png)

如果希望把源码也一块提交，可以加上 maven-source-plugin 插件。

```xml
<project>
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-source-plugin</artifactId>
                <version>3.0.1</version>
                <configuration>
                    <attach>true</attach>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                        <phase>compile</phase>
                    </execution>
                </executions>
            </plugin>
         </plugins>
      </build>
</project>
```

## 配置 maven-deploy-plugin
接下来我们需要添加 maven-deploy-plugin 插件用于必定上面依赖（maven.repo）存放的地址。

```xml
<project>
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>2.8.2</version>
                <configuration>
                    <altDeploymentRepository>maven.repo::default::file://${project.build.directory}/mvn-repo</altDeploymentRepository>
                </configuration>
            </plugin>
         </plugins>
      </build>
</project>
```

## 配置 site-maven-plugin
然后再添加 site-maven-plugin 插件，这是核心。需要配置 `server` 就是 Maven settings.xml 里面配置的 `server.id`。

```xml
<project>
    <build>
        <plugins>
            <plugin>
                <groupId>com.github.github</groupId>
                <artifactId>site-maven-plugin</artifactId>
                <version>0.12</version>
                <configuration>
                    <!-- git commit message -->
                    <message>Maven artifacts for ${project.version}</message>
                    <!-- disable webpage processing -->
                    <noJekyll>false</noJekyll>
                    <!-- matches distribution management repository url above -->
                    <outputDirectory>${project.build.directory}/mvn-repo</outputDirectory>
                    <!-- remote branch name -->
                    <branch>refs/heads/master</branch>
                    <includes>
                        <include>**/*</include>
                    </includes>
                    <merge>true</merge>
                    <!--Maven setttings 文件中定义的 server.id-->
                    <server>github</server>
                    <!-- 对应github上创建的仓库名 -->
                    <repositoryName>maven-repo</repositoryName>
                    <!-- github仓库所有者setting中的name或者组织名 -->
                    <repositoryOwner>zhangqinghua</repositoryOwner>
                </configuration>
                <executions>
                    <execution>
                        <phase>deploy</phase>
                        <goals>
                            <goal>site</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
         </plugins>
      </build>
</project>
```

需要注意的是 `repositoryOwner` 是 GitHub.Settings.name。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210627160110.png)

如果我们需要发布到 Organizations 上，则 `repositoryOwner`  改成组织名。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210627160221.png)

## 运行 mvn clean deploy

上面配置好之前，就可以运行发布了。

```bash
zhangqinghua$ mvn clean deploy
....
```

在 GitHub 的项目页面上可以看到已经多了一个 mvn-repo 的分支了。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210627150603.png)

里面可以看多上传了不同的项目。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210627151426.png)

不同的版本：

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210627151531.png)

## 使用
要使用 GitHub 上的依赖，只需要在项目 pom.xml 上添加仓库地址即可。

```xml
<project>
   <repositories>
      <repository>
         <id>maven-repo</id>
         <!-- 格式是 https://raw.githubusercontent.com/[github 用户名]/[github 仓库名]/[分支名]/repository -->
         <url>https://raw.githubusercontent.com/rxliuli/maven-repository-example/mvn-repo/repository</url>
      </repository>
   </repositories>
</project>>
```

就像其他 Maven 仓库一样，我们知道 groupId, artifactId 与 version，自然可以直接使用。

```xml
<project>
   <dependencies>
      <dependency>
         <groupId>com.rxliuli</groupId>
         <artifactId>maven-repository-example</artifactId>
         <version>1.0-SNAPSHOT</version>
      </dependency>
   </dependencies>
</project>>
```

> 这种使用 GitHub 部署的 Maven 仓库，在 Maven 中央仓库中并不能搜索到的哦

## 常见问题
#### Error creating blob: Not Found (404)
场景：`mvn deploy` 发布时报错，卡住了几个小时

```
Failed to execute goal com.github.github:site-maven-plugin:0.12:site (default) on project demo2: Error creating blob: Not Found (404)
```

原因：Maven settings.xml 里面的 `servers.server.password` 要使用 token，不要使用 GitHub 登陆密码。

原因：`repositoryOwner` 名称错误，`repositoryOwner` 是指 Githuh.Settings.Name 的，不是 GitHub 的登录名。

解决：参考上面。

#### Error retrieving user info: Not Found (404)
场景：`mvn deploy` 发布时报错。

```
Failed to execute goal com.github.github:site-maven-plugin:0.12:site (default) on project demo: Error retrieving user info: Not Found (404)
```

原因：GitHub Token 要需要授权用户信息，没有勾选就抱这个错误。

解决：参考上面。

#### Git Repository is empty. (409)
场景：`mvn deploy` 发布时报错。

```
Failed to execute goal com.github.github:site-maven-plugin:0.12:site (default) on project demo: Error creating blob: Git Repository is empty. (409)
```

原因：仓库为空。

解决：加个 README.md 即可。