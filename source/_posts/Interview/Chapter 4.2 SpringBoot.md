---
title: Chapter 4.2 SpringBoot

categories:
- Interview

date: 2020-04-28 00:00:42
---
#### Spring Boot 中的 starter 到底是什么？

## 概述
#### 什么是 Spring Boot？

#### Spring Boot 有哪些优点？

#### Spring Boot 的核心注解是哪个？它主要由哪几个注解组成的？
启动类上面的注解是 **@SpringBootApplication**，它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：
- **@SpringBootConfiguration**：组合了 **@Configuration** 注解，实现配置文件的功能。
- **@EnableAutoConfiguration**：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： **@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })**。
- **@ComponentScan**：Spring 组件扫描。

## 配置
#### 什么是 Spring JavaConfig？
Spring JavaConfig 是 Spring 社区的产品，它提供了配置 Spring IoC 容器的纯 Java 方法。因此它有助于避免使用 XML 配置。使用 JavaConfig 的优点在于：
- 面向对象的配置。由于配置被定义为 JavaConfig 中的类，因此用户可以充分利用 Java 中的面向对象功能。一个配置类可以继承另一个，重写它的 **@Bean** 方法等。

- 减少或消除 XML 配置。基于依赖注入原则的外化配置的好处已被证明。但是，许多开发人员不希望在 XML 和 Java 之间来回切换。

- 类型安全和重构友好。JavaConfig 提供了一种类型安全的方法来配置 Spring 容器。由于 Java 5.0 对泛型的支持，现在可以按类型而不是按名称检索 bean，不需要任何强制转换或基于字符串的查找。

#### Spring Boot 自动配置原理是什么？
注解 **@EnableAutoConfiguration**, **@Configuration**, **@ConditionalOnClass** 就是自动配置的核心，**@EnableAutoConfiguration** 给容器导入 META-INF/spring.factories 里定义的自动配置类。筛选有效的自动配置类。每一个自动配置类结合对应的 xxxProperties.java 读取配置文件进行自动配置功能

#### 你如何理解 Spring Boot 配置加载顺序？
在 Spring Boot 里面，可以使用以下几种方式来加载配置。

1. properties 文件

1. YAML 文件

1. 系统环境变量

1. 命令行参数

1. 等等……

#### Spring Boot 是否可以使用 XML 配置？
Spring Boot 推荐使用 Java 配置而非 XML 配置，但是 Spring Boot 中也可以使用 XML 配置，通过 **@ImportResource** 注解可以引入一个 XML 配置。

#### Spring Boot 核心配置文件是什么？

#### bootstrap.properties 和 application.properties 有何区别 ?

#### 如何在自定义端口上运行 Spring Boot？
**server.port = 8090**

#### Spring Boot 读取配置的方式？
Spring Boot 可以通过 **@PropertySource**，**@Value**，**@Environment**，**@ConfigurationProperties** 来绑定变量。

## 安全

#### 如何解决跨域问题？
实现 **WebMvcConfigurer** 接口然后重写 **addCorsMappings** 方法解决跨域问题。

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowCredentials(true)
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .maxAge(3600);
    }

}
```

#### 什么是 CSRF 攻击？
CSRF 代表跨站请求伪造。这是一种攻击，迫使最终用户在当前通过身份验证的 Web 应用程序上执行不需要的操作。CSRF 攻击专门针对状态改变请求，而不是数据窃取，因为攻击者无法查看对伪造请求的响应。

## 监视器

## 异常处理
#### 如何使用 Spring Boot 实现异常处理
Spring 提供了一种使用 **ControllerAdvice** 处理异常的非常有用的方法。 我们通过实现一个 **ControlerAdvice** 类，来处理控制器类抛出的所有异常。

## 定时任务
#### Spring Boot 中如何实现定时任务
在 Spring Boot 中使用定时任务主要有两种不同的方式，一个就是使用 Spring 中的 **@Scheduled** 注解，另一个则是使用第三方框架 Quartz。

## Session
#### 微服务中如何实现 session 共享
在微服务中，一个完整的项目被拆分成多个不相同的独立的服务，各个服务独立部署在不同的服务器上，各自的 session 被从物理空间上隔离开了，但是经常，我们需要在不同微服务之间共享 session ，常见的方案就是 Spring Session + Redis 来实现 session 共享。将所有微服务的 session 统一保存在 Redis 上，当各个微服务对 session 有相关的读写操作时，都去操作 Redis 上的 session 。这样就实现了 session 共享，Spring Session 基于 Spring 中的代理过滤器实现，使得 session 的同步操作对开发人员而言是透明的，非常简便。

## 整合第三方项目

## 其他
#### 怎么创建一个 Spring Boot 项目
- 继承 spring-boot-starter-parent 项目
- 导入 spring-boot-dependencies 项目依赖

#### spring-boot-starter-parent 有什么用
新创建一个 Spring Boot 项目，默认都是有 parent 的，这个 parent 就是 spring-boot-starter-parent，它主要有如下作用：
- 定义了 Java 编译版本为 1.8
- 使用 UTF-8 格式编码
- 继承自 spring-boot-dependencies，这个里边定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号
- 执行打包操作的配置
- 自动化的资源过滤
- 自动化的插件配置
- 针对 application.properties 和 application.yml 的资源过滤，包括通过 profile 定义的不同环境的配置文件，例如 application-dev.properties 和 application-dev.yml

#### 您使用了哪些 starter maven 依赖项

#### Spring Boot 项目如何热部署
通过引用 devtools 依赖可以实现热部署。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

这样，当修改一个 Java 类时就会热更新。

注意事项：
- 生产环境 devtools 将被禁用，如 **java -jar** 方式或者自定义的类加载器等都会识别为生产环境。
- 打包应用默认不会包含 devtools，除非你禁用 SpringBoot Maven 插件的 **excludeDevtools** 属性。
- Thymeleaf 无需配置 **spring.thymeleaf.cache: false**，devtools 默认会自动设置。

#### Spring Boot 需要独立的容器运行吗
可以不需要，内置了 Tomcat / Jetty 等容器。

#### 运行 Spring Boot 有哪几种方式
- 打包用命令或者放到容器中运行
- 用 Maven/ Gradle 插件运行
- 直接执行 main 方法运行

#### Spring Boot 打成的 jar 和普通的 jar 有什么区别？
Spring Boot 项目最终打包成的 jar 是可执行 jar，这种 jar 可以直接通过 **java -jar xxx.jar** 命令来运行，这种 jar 不可以作为普通的 jar 被其他项目依赖，即使依赖了也无法使用其中的类。

Spring Boot 的 jar 无法被其他项目依赖，主要还是他和普通 jar 的结构不同。普通的 jar 包，解压后直接就是包名，包里就是我们的代码，而 Spring Boot 打包成的可执行 jar 解压后，在 \BOOT-INF\classes 目录下才是我们的代码，因此无法被直接引用。如果非要引用，可以在 pom.xml 文件中增加配置，将 Spring Boot 项目打包成两个 jar，一个可执行，一个可引用。

[面试题](https://blog.csdn.net/thinkwon/article/details/104397299)