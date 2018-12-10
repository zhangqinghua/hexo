---
title: SpringBoot

categories:
- Java

date: 2018-03-26 15:00:00
---

Spring 是一个开源框架，是一个 轻量级的 实现IOC 和 AOP的框架。Spring正如其字面意思，是程序员的春天

<!-- more -->

# MVC是什么

1. model 模型，应用程序中处理数据逻辑的部分，Mybatis...
1. view 视图层，应用程序中用于展示的部分，html，jsp，themyleaf
1. controller 控制层，用于处理用户请求交互

# Spring是什么

Spring框架就是一个容器，这个容器最主要的作用就是创建对象，以前我们创建对象时通过`new`关键字，现在不需要这么麻烦了，只需要找到这个容器就可以找到你需要的对象，其实就是一个factory，这个工厂可以提供很多我们需要的对象，我们只需要知道对象对应的名字就行了。

Spring的特性是什么？
控制反转，面对切面和非侵入式。

# Spring都有哪些产品组成

1. Spring core
    核心容器提供 Spring 框架的基本功能。核心容器的主要组件是 BeanFactory，它是工厂模式的实现。BeanFactory 使用控制反转 （IOC） 模式将应用程序的配置和依赖性规范与实际 的应用程序代码分开。

1. Spring AOP
    过配置管理特性，Spring AOP 模块直接将面向方面的编程功能集成到了 Spring 框架中。

1. Spring ORM
    Spring 框架插入了若干个 ORM 框架，从而提供了 ORM 的对象关系工具

1. Spring MVC
    MVC 容纳了大量视图技术，其中包括 JSP、Velocity、Tiles、iText 和 POI。

1. Spring Web
    Web模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。

# 什么是IOC

IOC是一个生产和管理bean的容器就行了，原来需要在调用类中new的东西，现在都是有这个IOC容器进行产生，同时，要是产生的是单例的bean，他还可以给管理bean的生命周期！

主动控制变成被动控制，反客为主的意思。假如说，我需要牙刷，我自己去找，这个是我的主动控制！！假如说有牙刷时刻监测我是否需要牙刷，只要我需要就自动提供给我使用～这就是控制反转。

为什么要用IOC？

1. 依赖注入把代码量降到最低
1. 低侵入性使松散耦合得以实现

什么是控制反转？
在程序中被调用类的选择控制权从调用她的类中移除，转交给第三方裁决，这个第三方指的就是spring的容器。

Spring注入有哪些方式？

1. 构造器注入
1. Set注入
1. 接口注入
1. 静态工厂的方法注入
1. 实例工厂的方法注入

# 什么是AOP

面向切面编程，是一种编程技术，允许程序模块化横向切割关注点，或横切典型的责任划分，如日志和事务管理。Spring AOP使用了Java动态代理机制。

AOP能做什么？
日记，权限拦截，监控

AOP简单实现？
本质上就是运行时重写类方法，实现上就是运行时生成代理类。简单的说就是代理模式。

# Spring底层实现机制是什么

使用Domo4j（解析XML）+Java反射机制（使用反射实例化bean）。

# Spring有哪些注解

- @Controller（标识为控制器bean id）
- @Service（标记为注入为服务层）
- @Resource（按名称注入）
- @Autowired（按类型注入）
- @RequestMapping（表示映射URL路径）

# Spring MVC工作流程

1. 用户发送请求，被`DispatcherServlet`分发器捕获

1. `DispatcherServlet`解析URL，找到对应的`Handler`

1. 提取Request中的模型数据,给`Handler`入参
    - 将请求消息（如json，xml等）转换为一个对象
    - 数据转换
    - 数据格式化
    - 数据验证

1. 业务逻辑处理

1. 返回`ModelAndView`

1. 根据`ModelAndView`找到合适的`ViewResolver`

1. `ViewResolver`结合`ModelAndView`，来渲染视图

1. 将渲染结果返回给客户端

# Spring的优缺点

1. 降低组件之间的耦合度，实现软件各层之间的解耦
1. IOC负责对象的创建管理工作，减轻工作量
1. AOP可以很容易实现日记管理，权限拦截，监控等工作
1. 可以很容易集成JPA，Hibernate，Mybatis

1. 使用了大量的反射机制，自动实例化类，占内存

# Spring 微服务

待完成

# SpringBoot容器启动流程

![SpringBoot容器启动流程](001.png)

# 容器加载时执行特定操作

某些情况下我们需要在Spring Boot容器启动加载完后执行一些操作，此时可以通过实现`ApplicationListener<E extends ApplicationEvent>`接口，并指定相应事件来执行操作，例如启动某些自定义守护线程

`ApplicationContextEvent` 是由 ApplicationContext 引发的事件基类，它有几个实现类：

- `ContextRefreshedEvent`
    ApplicationContext 容器初始化或者刷新时触发该事件,执行一次
- `ContextStartedEvent`
    当使用 ConfigurableApplicationContext 接口的 start() 方法启动 ApplicationContext 容器时触发该事件
- `ContextClosedEvent`
    当使用 ConfigurableApplicationContext 接口的 close() 方法关闭 ApplicationContext 容器时触发该事件
- `ContextStopedEvent`
    当使用 ConfigurableApplicationContext 接口的 stop() 方法停止 ApplicationContext 容器时触发该事件

```java
@Component
class ApplicationStartup implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println("容器初始化或者刷新时触发该事件,执行一次");
    }
}
```

## 常见问题

- 同类转换错误
    热部署插件使用不同的Classloader导致，禁止即可。

- WebSocket无法注入Service对象
    其实不是不能注入，是已经注入了，但是客户端每建立一个链接就会创建一个对象，这个对象没有任何的bean注入操作，解决办法就是springboot启动的时候注入一个static的对象。
    ```java
    private static BaseWebSocket baseWebSocket;

    @Autowired
    protected GameService gameService;
    // 通过@PostConstruct实现初始化bean之前进行的操作
    @PostConstruct
    public void init() {
        baseWebSocket = this;
        // 初使化时将已静态化的Service实例化
        baseWebSocket.gameService = this.gameService;
    }

    // 这样使用
    public void createGame() {
        baseWebSocket.gameService.createGame();
    }
    ```

- 微信小程序json参数解析失败
    需要将`@ResponseBody` 去掉，这样才能接收

- 通配符
    `*` 表示多个任意字符
    `**` 表示可以表示任意多级目录