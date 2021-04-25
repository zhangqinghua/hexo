---
title: SpringCloud OpenFeign

categories:
- 后端开发
- SpringCloud

date: 2021-02-28 00:00:01
---
Feign 是一个声明式 WebService 客户端。使用 Feign 能让便携 WebService 客户端更加简单。

它的使用方法是定一个一个服务接口然后在上面添加注解。Feign 也支持可拔插式的编码器和解码器。SpringCloud 对 Feign 进行了封装，使其支持了 Spring MVC 标准注解和 `HttpMessageConverters`。Feign 可以与 Eureke 和 Ribbon 组合使用以支持负载均衡。

## Feign 能干什么
Feign 旨在使编写 Java Http 客户端变得更加容易。

前面在使用 Ribbon + RestTemplate 时，利用 RestTemplate 对 Http 请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。

所以，Feign 在此基础上做了进一步封装，由它来帮助我们定义和实现依赖服务接口的定义。在 Feign 的实现下，我们只需创建一个接口并使用注解的方式来配置它（以前是 Dao 接口上面标注 Mapper 注解，现在是一个微服务接口上标注一个 Feign 注解），即可完成对服务提供方的接口绑定，简化了使用 Spring Cloud Ribbon 时，自动封装服务调用客户端的开发量。

## Feign 集成了 Ribbon
利用 Ribbon 维护了 Payment 的服务列表信息，并且通过轮询实现了客户端和负载均衡。而与 Ribbon 不同的是，通过 Feign 只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用。

## Feign 和 OpenFeign 的区别
Feign 是 SpringCloud 组件中的一个轻量级 RESTful 的 HTTP 服务客户端。Feign 内置了 Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign 的使用方法是：使用 Feign 的注解定义接口，调用这个接口，就可以调用服务注册中心的服务。

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

OpenFeign 是 Spring Cloud 在 Feign 的基础上支持了 SpringMVC 的注解，如 @RequestMapping 等等。OpenFeign 的 @FeignClient 可以解析 SpringMVC 的@RequestMapping 注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其它服务。

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

## OpenFeign 使用步骤

#### 基本环境
1. 注册中心
   已启动了一个 Eureka 注册中心，地址为本机 8080 端口。当然，如果是其他注册中心也可以。
1. 服务注册
   已经注册了两个服务名为 ORDER_PROVIDE 和 ORDER_COMSUMER 的服务，地址分别为本机 8081 和 8082 端口。
1. 提供服务
    服务提供者 ORDER_PROVIDE 提供 `getUserById` 和 `sayHi` 的接口。

```java
@RestController
@RequestMapping("/user")
public class UserAdminController {

   @Autowired
   IUserAdminService userAdminService;

   @Value("${server.port}")
   private String serverPort;

   @GetMapping("/getUserById/{userId}")
   public BaseResponse<UserVO> getUserById(@PathVariable Integer userId){ 
      return BaseResponse.success(userVO);
   }

   @GetMapping("/sayHi")
   public String sayHi(){
   String number= UUID.randomUUID().toString();
      return "service,port:"+serverPort+",number:"+ number;
   }
}
```

#### 添加依赖
给消费者服务 ORDER_COMSUMER 添加 OpenFeign 的依赖：

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### 配置 YML


#### 创建生产者和消费者服务，将其注册到注册中心



## OpenFeign 超时控制

## OpenFeign 日志打印
