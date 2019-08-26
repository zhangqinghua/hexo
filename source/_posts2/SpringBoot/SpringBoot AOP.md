---
title: SpringBoot AOP

categories:
- SpringBoot

date: 2018-05-22
---

在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程。

我们知道，面向对象的特点是继承、多态和封装。而封装就要求将功能分散到不同的对象中去，这在软件设计中往往称为职责分配。实际上也就是说，让不同的类设计不同的方法。这样代码就分散到一个个的类中去了。这样做的好处是降低了代码的复杂程度，使类可重用。

但是人们也发现，在分散代码的同时，也增加了代码的重复性。什么意思呢？比如说，我们在两个类中，可能都需要在每个方法中做日志。按面向对象的设计方法，我们就必须在两个类的方法中都加入日志的内容。也许他们是完全相同的，但就是因为面向对象的设计让类与类之间无法联系，而不能将这些重复的代码统一起来。

也许有人会说，那好办啊，我们可以将这段代码写在一个独立的类独立的方法里，然后再在这两个类中调用。但是，这样一来，这两个类跟我们上面提到的独立的类就有耦合了，它的改变会影响这两个类。那么，有没有什么办法，能让我们在需要的时候，随意地加入代码呢？**这种在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程。**

一般而言，我们管切入到指定类指定方法的代码片段称为切面，而切入到哪些类、哪些方法则叫切入点。有了AOP，我们就可以把几个类共有的代码，抽取到一个切片中，等到需要时再切入对象中去，从而改变其原有的行为。

## SpringBoot使用AOP

- 引入AOP依赖
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
    ```

- 创建切面类
    ```java
    @Configuration
    @Aspect
    public class WebLogAspect {
        @Pointcut("execution(public * com.didispace.web..*.*(..))")
        public void webLog(){}

        @Before("webLog()")
        public void doBefore(JoinPoint joinPoint) throws Throwable {}

        @AfterReturning(returning = "ret", pointcut = "webLog()")
        public void doAfterReturning(Object ret) throws Throwable {}
    }
    ```

## AOP 注解

- `@Configuration`
    spring-boot配置类
- `@Aspect`
    描述一个切面类，定义切面类的时候需要打上这个注解
- `@Pointcut`
    声明一个切入点，切入点决定了连接点关注的内容，使得我们可以控制通知什么时候执行。

- `@Before`
    前置通知：在某连接点之前执行的通知，但这个通知不能阻止连接点之前的执行流程（除非它抛出一个异常）。
- `AfterReturning`
    后置通知：在某连接点正常完成后执行的通知，通常在一个匹配的方法返回的时候执行。
    可以在后置通知中绑定返回值，如：
    ```java
    @AfterReturning（pointcut="CacheDemoService.findById(..)) and @annotation(Transactional)", returning="retVal"）
    public void doFindByIdCheck（Object retVal） {
        // ...
    }
    ```

## AOP切面中的同步

在`WebLogAspect`切面中，分别通过`doBefore`和`doAfterReturning`两个独立函数实现了切点头部和切点返回后执行的内容，若我们想统计请求的处理时间，就需要在`doBefore`处记录时间，并在`doAfterReturning`处通过当前时间与开始处记录的时间计算得到请求处理的消耗时间。

那么我们是否可以在`WebLogAspect`切面中定义一个成员变量来给`doBefore`和`doAfterReturning`一起访问呢？是否会有同步问题呢？

的确，直接在这里定义基本类型会有同步问题，所以我们可以引入`ThreadLocal`对象，像下面这样进行记录：

```java
@Aspect
@Component
public class WebLogAspect {

    private Logger logger = Logger.getLogger(getClass());

    ThreadLocal<Long> startTime = new ThreadLocal<>();

    @Pointcut("execution(public * com.didispace.web..*.*(..))")
    public void webLog(){}

    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
        startTime.set(System.currentTimeMillis());

        // 省略日志记录内容
    }

    @AfterReturning(returning = "ret", pointcut = "webLog()")
    public void doAfterReturning(Object ret) throws Throwable {
        // 处理完请求，返回内容
        logger.info("RESPONSE : " + ret);
        logger.info("SPEND TIME : " + (System.currentTimeMillis() - startTime.get()));
    }
}
```

## AOP切面的优先级

由于通过AOP实现，程序得到了很好的解耦，但是也会带来一些问题，比如：我们可能会对Web层做多个切面，校验用户，校验头信息等等，这个时候经常会碰到切面的处理顺序问题。

所以，我们需要定义每个切面的优先级，我们需要`@Order(i)`注解来标识切面的优先级。`i`的值越小，优先级越高。

假设我们还有一个切面是`CheckNameAspect`用来校验`name`必须为`didi`，我们为其设置`@Order(10)`，而上文中`WebLogAspect`设置为`@Order(5)`，所以`WebLogAspect`有更高的优先级，

这个时候执行顺序是这样的：
- 在`@Before`中优先执行`@Order(5)`，再执行`@Order(10)`
- 在`@After`和`@AfterReturning`中优先执行`@Order(10)`，再执行`@Order(5)`

所以我们可以这样子总结：
- 在切入点前的操作，按`order`的值由小到大执行
- 在切入点后的操作，按`order`的值由大到小执行
