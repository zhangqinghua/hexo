---
title: Swagger 教程

categories:
- 后端开发
- Swagger

date: 2021-03-16
---
## 创建多个分组
```java

@Bean
public Docket test2() {
   return docket().paths(PathSelectors.ant("/admin/**"))
                  .build()
                  .groupName("2.平台端");
}

@Bean
public Docket test3() {
   return docket().paths(PathSelectors.ant("/merchant/**"))
                  .build()
                  .groupName("3.商户端");
}

@Bean
public Docket test4() {
   return docket().paths(PathSelectors.ant("/store/**"))
                  .build()
                  .groupName("4.门店端");
}

private ApiSelectorBuilder docket() {
   return new Docket(DocumentationType.SWAGGER_2)
            .host(swagger2CoreConfig.host())
            .directModelSubstitute(LocalDate.class, String.class)
            .directModelSubstitute(LocalTime.class, String.class)
            .directModelSubstitute(LocalDateTime.class, String.class)
            .directModelSubstitute(ZonedDateTime.class, String.class)
            .apiInfo(swagger2CoreConfig.apiInfo())
            .select()
            // 扫描路径
            .apis(Swagger2CoreConfig.basePackage("com.icebartech,com.easybyte"));
}
```

## 时间格式优化
Swagger UI 的页面中，请求的数据类型会被序列化成字符串，显示在 Model Schema 中。但是，Java8 中的 `LocalDateTime` 类型会被序列化成很复杂的字符串，如下：
```java
 @ApiModelProperty(value = "时间段（HH:mm）（开始）", example = "09:30")
private LocalTime startTime;
```

显示效果：

```json
{
   "startTime": {
      "hour": 0,
      "minute": 0,
      "nano": 0,
      "second": 0
   }
}
```

解决的办法其实很简单，在 Swagger 的配置中，添加 `directModelSubstitute` 方法的代码：

```java
@Configuration
@EnableSwagger2
public class Swagger2Config {
 
	@Bean
	public Docket createRestApi() {
		return new Docket(DocumentationType.SWAGGER_2)
				.directModelSubstitute(LocalDateTime.class, Date.class)
				.directModelSubstitute(LocalDate.class, String.class)
				.directModelSubstitute(LocalTime.class, String.class)
				.directModelSubstitute(ZonedDateTime.class, String.class)
				.apiInfo(apiInfo()).select()
				.apis(RequestHandlerSelectors.basePackage("com.abcd.restful")).paths(PathSelectors.any()).build();
	}
 
	private ApiInfo apiInfo() {
		return new ApiInfoBuilder().title("Platform API").contact("abcd").version("1.0").build();
	}
}
```

`directModelSubstitute` 方法顾名思义就是在序列化的时候用一个类型代替一个类型。

上面的例子，`LocalDateTime` 类型用 `Date` 类型替代，`LocalDate` 类型直接用 `String` 类型替代，这样就避免的 Swagger 原生的序列化方法把 `LocalDateTime` 序列化的很复杂。效果如下：

```json
{
   "finishTime": "18:30"
}
```

## 限制对象无限嵌套
https://blog.csdn.net/henuboy/article/details/80822032

## 扫描多个包路径
需要重写 `basePackage` 方法。

```java
private ApiSelectorBuilder docket() {
   return new Docket(DocumentationType.SWAGGER_2)
            .host(swagger2CoreConfig.host())
            .directModelSubstitute(LocalDate.class, String.class)
            .directModelSubstitute(LocalTime.class, String.class)
            .directModelSubstitute(LocalDateTime.class, String.class)
            .directModelSubstitute(ZonedDateTime.class, String.class)
            .apiInfo(swagger2CoreConfig.apiInfo())
            .select()
            // 扫描路径
            .apis(Swagger2CoreConfig.basePackage("com.icebartech,com.easybyte"));
}
   
@SuppressWarnings("Guava")
public static Predicate<RequestHandler> basePackage(final String basePackage) {
   return input -> declaringClass(input).transform(handlerPackage(basePackage)).or(true);
}

@SuppressWarnings("Guava")
private static Function<Class<?>, Boolean> handlerPackage(final String basePackage) {
   return input -> {
      // 循环判断匹配
      for (String strPackage : basePackage.split(",")) {
            boolean isMatch = input.getPackage().getName().startsWith(strPackage);
            if (isMatch) {
               return true;
            }
      }
      return false;
   };
}

@SuppressWarnings({"Guava", "deprecation"})
private static Optional<? extends Class<?>> declaringClass(RequestHandler input) {
   return Optional.fromNullable(input.declaringClass());
}
```

## 微服务聚合文档
```java
@Slf4j
@Primary
@Component
@ApiIgnore
@RestController
public class CustomSwaggerResourcesProvider implements SwaggerResourcesProvider {

    @Autowired
    private SiteProperties siteProperties;
    @Autowired
    private SwaggerProperties swaggerProperties;

    @GetMapping("/custom/api-docs")
    public JSONObject test(HttpServletRequest request, String group) throws IOException {
        // 当前域名 https://beesgo.chinabeego.com/api
        String baseUrl = request.getRequestURL().toString().replace("/custom/api-docs", "");
        // String baseUrl = "http://localhost:8081/api";

        // 获取网关（本地）模块的swagger数据（只保留头部描述）
        JSONObject localApiDocs = JSONObject.fromObject(Request.Get(baseUrl + "/v2/api-docs?group=Last.所有接口")
                                                               .execute()
                                                               .returnContent()
                                                               .toString());

        // 移除掉basePath，由各自的path定义
        localApiDocs.put("basePath", "");

        // 获取其它模块的swagger数据
        List<JSONObject> routesApiDocs = new ArrayList<>();

        swaggerProperties.getServices().parallelStream().forEach(servername -> {
            String url = "http://localhost:8081" + "/" + servername + "/api/v2/api-docs?group=" + group;
            System.out.println(url);
            try {
                String ret = Request.Get(url).execute().returnContent().toString();
                JSONObject object = JSONObject.fromObject(ret);
                routesApiDocs.add(object);
            } catch (IOException e) {
                log.warn("Swagger整合微服务「{}」失败，接口「{}」访问失败", servername, url);
            } catch (Exception e) {
                e.printStackTrace();
            }
        });

        // 整合其它模块的path数据
        JSONObject paths = new JSONObject();
        for (JSONObject route : routesApiDocs) {
            // 因为本地的basePath已经移除掉，需要这里定义basePath，才能转发到真正的微服务的路径上去
            if (!route.containsKey("paths")) continue;
            if (!route.containsKey("basePath")) continue;

            // 使用 /api 来替换真实路径，有网关兜底。
            // String basePath = route.getString("basePath");
            String basePath = "/api";
            JSONObject routePaths = route.getJSONObject("paths");
            @SuppressWarnings("unchecked")
            Iterator<String> keys = routePaths.keys();
            while (keys.hasNext()) {
                String key = keys.next();
                paths.put(basePath + key, routePaths.getJSONObject(key));
            }
        }
        localApiDocs.put("paths", paths);

        // 整合其它模块的definitions数据（入参，返回的描述）
        JSONObject definitions = new JSONObject();
        for (JSONObject route : routesApiDocs) {
            if (route.containsKey("definitions")) {
                JSONObject routeDefinitions = route.getJSONObject("definitions");
                @SuppressWarnings("unchecked")
                Iterator<String> keys = routeDefinitions.keys();
                while (keys.hasNext()) {
                    String key = keys.next();
                    definitions.put(key, routeDefinitions.getJSONObject(key));
                }
            }
        }
        localApiDocs.put("definitions", definitions);

        // 整合其它模块的tags数据，使得swagger根据tags对path进行排序
        List<JSONObject> tags = new ArrayList<>();
        for (JSONObject route : routesApiDocs) {
            if (!route.containsKey("tags"))
                continue;
            for (int i = 0; i < route.getJSONArray("tags").size(); i++) {
                tags.add(route.getJSONArray("tags").getJSONObject(i));
            }
        }
        tags.sort(Comparator.comparing(l -> l.getString("name")));
        localApiDocs.put("tags", tags);
        return localApiDocs;
    }

    /**
     * 这里要将多个swagger文档整合在一起
     * 1. swagger的头部信息取自http://localhost:8081/api/v2/api-docs
     * 2. swagger的各个服务取自ttp://localhost:8081/api/order/v2/api-docs
     * 3. 创建一个新文档，将上面2个整合在一起
     */
    @Override
    public List<SwaggerResource> get() {
        List<SwaggerResource> resources = new ArrayList<>();
        for (String group : swaggerProperties.getGroups()) {
            SwaggerResource swaggerResource = new SwaggerResource();
            swaggerResource.setName(group);
            // 这里使用自定义的api-docs地址
            swaggerResource.setLocation("/custom/api-docs?group=" + group);
            swaggerResource.setSwaggerVersion("1.0");
            resources.add(swaggerResource);
        }
        return resources;
    }
}
```

配置文件：

```yml
# Swagger文档数据
swagger:
  # 接口分组
  groups:
    - 2.平台端
    - 3.商户端
    - 4.门店端
    - 5.小程序
    - 6.通用接口
  # 聚合文档，涉及到的服务
  services:
    - easybyte-log
    - easybyte-base
    - easybyte-auth
    - easybyte-merchant
    - easybyte-store
    - easybyte-product
    - easybyte-flow
    - easybyte-media
    - easybyte-consumer
    - easybyte-stats
    - easybyte-member
    - easybyte-market
    - easybyte-template
```