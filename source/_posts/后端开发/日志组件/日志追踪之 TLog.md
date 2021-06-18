---
title: 日志追踪之 TLog

categories:
- 后端开发
- 日志组件

date: 2021-06-18
---

## 快速接入
#### 配置依赖
```yml
<dependency>
  <groupId>com.yomahub</groupId>
  <artifactId>tlog-all-spring-boot-starter</artifactId>
  <version>1.0.0</version>
</dependency>
```

#### 框架适配
只需要在你的启动类中加入一行代码，即可以自动进行探测你项目所使用的Log框架，并进行增强。目前支持 log4j，log4j2，logback 三大日志框架。

```java
@SpringBootApplication
public class Runner {

    static {AspectLogEnhance.enhance();}//进行日志增强，自动判断日志框架

    public static void main(String[] args) {
        SpringApplication.run(Runner.class, args);
    }
}
```

> AspectLogEnhance.enhance() 需要放在启动类里面，从而第一个加载启动。

#### 查看效果
启动项目时，如果有打印以下字样，就表示接入成功了：

```
locakback同步日志增强成功
...
```

打印日志效果：

```
2021-06-18 14:38:48.212 ERROR 20038 --- [nio-8087-exec-1] c.a.druid.pool.DruidAbstractDataSource   : <0.1><8717751251595072> discard long time none received connection. , jdbcUrl : jdbc:mysql://47.119.139.41:3306/easybyte_log?useSSL=false&serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&autoReconnect=true&useCompression=true&zeroDateTimeBehavior=CONVERT_TO_NULL, jdbcUrl : jdbc:mysql://47.119.139.41:3306/easybyte_log?useSSL=false&serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&autoReconnect=true&useCompression=true&zeroDateTimeBehavior=CONVERT_TO_NULL, lastPacketReceivedIdleMillis : 319672
2021-06-18 14:38:48.217 ERROR 20038 --- [nio-8087-exec-1] c.a.druid.pool.DruidAbstractDataSource   : <0.1><8717751251595072> discard long time none received connection. , jdbcUrl : jdbc:mysql://47.119.139.41:3306/easybyte_log?useSSL=false&serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&autoReconnect=true&useCompression=true&zeroDateTimeBehavior=CONVERT_TO_NULL, jdbcUrl : jdbc:mysql://47.119.139.41:3306/easybyte_log?useSSL=false&serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&autoReconnect=true&useCompression=true&zeroDateTimeBehavior=CONVERT_TO_NULL, lastPacketReceivedIdleMillis : 319730
```

## 附：Scheduled 注解支持
默认情况下是不支持 `Scheduled` 注解了，如果我们希望在这些注解上应用 TLog，只需要手工编码：

```java
@Slf4j
@Component
public class AfterSaleScheduled {

    private static final TLogRPCHandler tLogRPCHandler = new TLogRPCHandler();

    @Scheduled(cron = "0 0/1 * * * ?")
    public void test1() {
        tLogRPCHandler.processProviderSide(new TLogLabelBean());

        // 这后面的日志都会被 TLog 托管
        log.info("=================AfterSaleScheduled======================");
    }
}
```

## 常见问题
#### 日志增强失败，Cannot enhance @Configuration bean definition 'feignConfig' since 。。。 
场景：接入日志增强失败，提示：

```
2021-06-18 15:04:29.897  INFO 20670 --- [           main] o.s.c.a.ConfigurationClassPostProcessor  : Cannot enhance @Configuration bean definition 'feignConfig' since its singleton instance has been created too early. The typical cause is a non-static @Bean method with a BeanDefinitionRegistryPostProcessor return type: Consider declaring such methods as 'static'.
locakback同步日志增强失败
log4j日志增强失败
```

原因：TLog 需要在其它日志组件前先加载。

解决：`AspectLogEnhance.enhance()` 定义在启动类里。
