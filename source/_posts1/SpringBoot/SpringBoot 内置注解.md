---
title: SpringBoot 内置注解

categoies:
- SpringBoot

date: 2018-05-22
---

SpringBoot 提供了许多注解，可以很方便的进行动态识别（也可以说是代码自动化执行）。

- `@Controller` 
    进行控制器的配置注解，这个注解所在的类就是控制器类。

    ![@Controller](001.png)

- `@EnableAutoConfiguration`
    表示开启自动配置处理。如果不加这个注解就不是SpringBoot程序。

    ![@EnableAutoConfiguration](002.png)

- `@RequestMapping("/")`
    表示访问的映射路径，此时的路径为`/`,访问地址：`localhost:8080`。

    ![@RequestMapping](003.png)

- `@ResponseBody`
    在Restful架构之中，改注解表示直接将返回的数据以字符串或JSON的形式获得。

    ![@ResponseBody](004.png)


## `@SpringBootApplication`

假设我们有一个项目

![项目结构](001.png)

入口类`Application`代码：

```java
@SpringBootApplication
public class Application {
    @Bean
    public Runnable createRunnable(){
        return () -> System.out.println("spring boot is running");
    }

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Application.class,args);
        context.getBean(Runnable.class).run();
        System.out.println(context.getBean(User.class));
        Map map = (Map) context.getBean("createMap");
        int age = (int) map.get("age");
        System.out.println("age=="+age);

    }
}
```



首先我们分析的就是入口类Application的启动注解@SpringBootApplication，进入源码：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
....
```

- `@Controller`和`@RestController`
    告诉Spring以字符串的形式渲染结果，并直接返回给调用者。

- `@RequestMapping`
    提供路由信息，负责URL到Controller中的具体函数的映射。

- `@EnableAutoConfiguration`
    Spring Boot自动配置（auto-configuration）：尝试根据你添加的jar依赖自动配置你的Spring应用。例如，如果你的classpath下存在HSQLDB，并且你没有手动配置任何数据库连接beans，那么我们将自动配置一个内存型（in-memory）数据库”。你可以将`@EnableAutoConfiguration`或者`@SpringBootApplication`注解添加到一个`@Configuration`类上来选择自动配置。如果发现应用了你不想要的特定自动配置类，你可以使用`@EnableAutoConfiguration`注解的排除属性来禁用它们。

- `@Configuration`
    相当于传统的xml配置文件，如果有些第三方库需要用到xml文件，建议仍然通过`@Configuration`类作为项目的配置主类——可以使用`@ImportResource`注解加载xml配置文件。

- `@ComponentScan`
    表示将该类自动发现扫描组件。个人理解相当于，如果扫描到有`@Component`、`@Controller`、`@Service`等这些注解的类，并注册为Bean，可以自动收集所有的Spring组件，包括`@Configuration`类。
    我们经常使用`@ComponentScan`注解搜索beans，并结合`@Autowired`注解导入。
    如果没有配置的话，Spring Boot会扫描启动类所在包下以及子包下的使用了`@Service`，`@Repository`等注解的类。

- `@SpringBootApplication`

- `@ConfigurationProperties`

- `@EnableConfigurationProperties`

- `@Component`
    泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注，Spring 注解`@Component`等效于`@Service`，`@Controller`，`@Repository`。

- `@Bean`
    此注解主要被用在方法上，来显式声明要用生成的类。用`@Configuration`注解该类，等价于XML中配置beans，用`@Bean`标注方法等价于XML中配置bean。
    现在项目上，本工程中的类，一般都使用`@Component`来生成bean。在把通过web service取得的类，生成Bean时，使用`@Bean`和getter方法来生成bean。

- `@Profiles`
    Spring Profiles提供了一种隔离应用程序配置的方式，并让这些配置只能在特定的环境下生效。任何`@Component`或`@Configuration`都能被`@Profile`标记，从而限制加载它的时机。
    ```java
    @Configuration
    @Profile("production")
    public class ProductionConfiguration {
    // ...
    }
    ```
    以正常的Spring方式，你可以使用一个`spring.profiles.active`的Environment属性来指定哪个配置生效。你可以使用平常的任何方式来指定该属性，例如，可以将它包含到你的`application.properties`中：
    `spring.profiles.active=dev,hsqldb`

- `@Autowired`
    自动导入依赖的bean。

- `@Service`
    一般用于修饰service层的组件。

- `@Repository`
    使用`@Repository`注解可以确保DAO或者repositories提供异常转译，这个注解修饰的DAO或者repositories类会被`@ComponetScan`发现并配置，同时也不需要为它们提供XML配置项。

- `@Value`
    注入Spring boot `application.properties`配置的属性的值。
    ```java
    @Value(value = "#{message}") 
    private String message;
    ```

- `@Inject`
    等价于默认的`@Autowired`，只是没有required属性。

## SpringMVC 注解

- `@RequestMapping`
    `@RequestMapping`是一个用来处理请求地址映射的注解，表示该控制器处理所有`/path`的URL请求。可用于类或方法上。 
    用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。
    该注解有六个属性： 
    1. `params`：指定request中必须包含某些参数值是，才让该方法处理。 
    1. `headers`：指定request中必须包含某些指定的header值，才能让该方法处理请求。 
    1. `value`：指定请求的实际地址，指定的地址可以是URI Template 模式。
    1. `method`：指定请求的method类型， `GET`、`POST`、`PUT`、`DELETE`等。
    1. `consumes`：指定处理请求的提交内容类型（Content-Type），如`application/json`，`text/html`。  
    1. `produces`：指定返回的内容类型，仅当Request请求头中的(Accept)类型中包含该指定类型才返回。

- `@RequestParam`
    用在方法的参数前面。 

- `@PathVariable`
    路径变量，参数与大括号里的名字一样要相同。
    ```java
    RequestMapping("user/get/mac/{macAddress}") 
    public String getByMacAddress(@PathVariable String macAddress){ 
        //do something; 
    } 
    ```

## 全局异常处理

- `@ControllerAdvice`
- `@ExceptionHandler`

## JAP 注解

- `@Entity`
    表明这是一个实体类。

- `@MappedSuperClass`
    用在确定是父类的entity上。父类的属性子类可以继承。

- `@NoRepositoryBean`
    一般用作父类的repository，有这个注解，spring不会去实例化该repository。

- `@Id`
    表示该属性为主键。

- `@GeneratedValue`
    表示主键生成策略

- `@SequenceGeneretor`

- `@Column`
    如果字段名与列名相同，则可以省略。

- `@Transient`
    表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性。

- `@JsonIgnore`
    作用是json序列化时将Java bean中的一些属性忽略掉,序列化和反序列化都受影响。

- `@JoinColumn`
    一对一：本表中指向另一个表的外键。一对多：另一个表指向本表的外键。

- `@OneToOne、@OneToMany、@ManyToOne`
    对应hibernate配置文件中的一对一，一对多，多对一。