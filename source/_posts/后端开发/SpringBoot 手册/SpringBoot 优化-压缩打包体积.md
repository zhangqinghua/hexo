---
title: SpringBoot 优化-压缩打包体积
categories:
- 后端开发
- SpringBoot 手册

date: 2021-05-23
---
SpringBoot 的 Web 应用一般都添加了 `spring-boot-maven-plugin` 插件，打出来的 jar 包内置了所有的依赖, 放在 `BOOT-INF/lib `目录, 所以体积很大。

```xml
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-maven-plugin</artifactId>  
        </plugin>  
    </plugins>  
</build>  
```

这时候我们可以将第三方依赖包独立出来，在程序运行时候再使用它们。

## 导出所有依赖到外面
使用 Maven 可以很方便的导出项目依赖的 jar 包，直接使用命令就可以进行导出：
```bash
# /lib 是指要导出的目录
zhangqinghua$ dependency:copy-dependencies -DoutputDirectory=/lib
```
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210523175956.png)

导出结果：

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210523180223.png)

## 将内置的依赖排除掉
修改 `spring-boot-maven-plugin` 的参数, 使其将内置的 jar 包排除掉，使用如下:

```xml
<build>
   <finalName>easybyte-auth</finalName>
   <plugins>
      <!--直接打成可运行的jar包-->
      <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
               ...
               <layout>ZIP</layout>
               <!-- 保留指定的依赖，其它的会排除出去 -->
               <includes>
                  <include>
                        <groupId>com.easybyte</groupId>
                        <artifactId>easybyte-api</artifactId>
                  </include>
               </includes>
            </configuration>
      </plugin>
   </plugins>
</build> 
```

`includes` 是指包含哪些项目的 jar 包，因为我项目结构的问题，我必须将 easybyte-api 这个项目打包进去。假如你的项目中没有自己项目依赖可以写成如下格式，表示不包含任何 jar 包。

```xml
...
<includes>
    <include>
        <groupId>nothing</groupId>            
        <artifactId>nothing</artifactId>
    </include>
</includes>
...
```

另外还可以使用 `excludes` 命令排除指定的依赖。

```xml
<build>
   <finalName>easybyte-auth</finalName>
   <plugins>
      <!--直接打成可运行的jar包-->
      <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
               ...
               <layout>ZIP</layout>
               <!-- 排除指定的依赖，其它的会保留下来 -->
               <excludes>
                  <exclude>
                        <groupId>com.easybyte</groupId>
                        <artifactId>easybyte-api</artifactId>
                  </exclude>
               </excludes>
            </configuration>
      </plugin>
   </plugins>
</build> 
```

> layout 是为了 SpringBoot 简化后能够加载第三方 jar 包目录，如果没加入这句话，在待会儿启动时，会报错。

这时候执行打包命令，编译出来的应用包就很小了：

```bash
zhangqinghua$ mvn clean package
...

zhangqinghua$ ls -l -h easybyte-auth/target
total 2360
drwxr-xr-x  4 zhangqinghua  staff   128B May 23 17:49 classes
-rw-r--r--  1 zhangqinghua  staff   1.1M May 23 17:49 easybyte-auth.jar
-rw-r--r--  1 zhangqinghua  staff    62K May 23 17:49 easybyte-auth.jar.original
drwxr-xr-x  3 zhangqinghua  staff    96B May 23 17:49 generated-sources
drwxr-xr-x  3 zhangqinghua  staff    96B May 23 17:49 maven-archiver
```

## 运行时外部加载依赖
此时的 jar 包并不能直接运行，因为没有这些第三方依赖。

```bash
zhangqinghua$ Java -jar easybyte-auth/target/easybyte-auth.jar
Exception in thread "main" java.lang.NoClassDefFoundError: org.springframework.security.crypto.password.PasswordEncoder
        at java.lang.J9VMInternals.prepareClassImpl(Native Method)
        at java.lang.J9VMInternals.prepare(J9VMInternals.java:303)
...
```

现在需要改成如下格式：

```bash
# /path/lib 是存放第三方依赖的目录
zhangqinghua$ Java –Dloader.path=/path/lib -jar easybyte-auth/target/easybyte-auth.jar
...
```

## 总结
经测试，经过上面的两个步骤，笔者的应用从 70MB 缩小为 1.3MB，极大地缩小了体积。

既缩小了体积，便于传输，又很容易地控制依赖jar的版本，做到全公司统一，共享同一套依赖集合。

特别地，如果要讲应用部署到 Docker 中，需要修改 dockerfile，将依赖目录挂载到 Docker 镜像中。修改应用的启动命令，添加 `loader.path` 参数，指向挂载进来的依赖目录。不建议将依赖 `ADD` 到 Docker 镜像，那样的话 Docker 镜像会很大。