---
title: Swagger2 笔记

tags:
- Java
- Swagger2

categories:
- Java

date: 2018-04-13
---

Swagger2可以轻松的整合到Spring Boot中，并与Spring MVC程序配合组织出强大RESTful API文档。它既可以减少我们创建文档的工作量，同时说明内容又整合入实现代码中，让维护文档和修改代码整合为一体，可以让我们在修改代码逻辑的同时方便的修改文档说明。另外Swagger2也提供了强大的页面测试功能来调试每个RESTful API。

## 安装

1. 在pom.xml中加入Swagger2的依赖
    ```xml
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.2.2</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.2.2</version>
    </dependency>
    ```
1. 创建Swagger2配置类
    ```java
    @Configuration
    @EnableSwagger2
    public class Swagger2 {

        @Bean
        public Docket createRestApi() {
            return new Docket(DocumentationType.SWAGGER_2)
                    .apiInfo(apiInfo())
                    .select()
                    .apis(RequestHandlerSelectors.basePackage("com.didispace.web"))
                    .paths(PathSelectors.any())
                    .build();
        }

        private ApiInfo apiInfo() {
            return new ApiInfoBuilder()
                    .title("Spring Boot中使用Swagger2构建RESTful APIs")
                    .description("更多Spring Boot相关文章请关注：http://blog.didispace.com/")
                    .termsOfServiceUrl("http://blog.didispace.com/")
                    .contact("程序猿DD")
                    .version("1.0")
                    .build();
        }

    }
    ```
1. 添加文档内容
    ```java
    @RestController
    @RequestMapping(value="/users")     // 通过这里配置使下面的映射都在/users下，可去除
    public class UserController {

        static Map<Long, User> users = Collections.synchronizedMap(new HashMap<Long, User>());

        @ApiOperation(value="获取用户列表", notes="")
        @RequestMapping(value={""}, method=RequestMethod.GET)
        public List<User> getUserList() {
            List<User> r = new ArrayList<User>(users.values());
            return r;
        }

        @ApiOperation(value="创建用户", notes="根据User对象创建用户")
        @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
        @RequestMapping(value="", method=RequestMethod.POST)
        public String postUser(@RequestBody User user) {
            users.put(user.getId(), user);
            return "success";
        }

        @ApiOperation(value="获取用户详细信息", notes="根据url的id来获取用户详细信息")
        @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long")
        @RequestMapping(value="/{id}", method=RequestMethod.GET)
        public User getUser(@PathVariable Long id) {
            return users.get(id);
        }

        @ApiOperation(value="更新用户详细信息", notes="根据url的id来指定更新对象，并根据传过来的user信息来更新用户详细信息")
        @ApiImplicitParams({
                @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long"),
                @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
        })
        @RequestMapping(value="/{id}", method=RequestMethod.PUT)
        public String putUser(@PathVariable Long id, @RequestBody User user) {
            User u = users.get(id);
            u.setName(user.getName());
            u.setAge(user.getAge());
            users.put(id, u);
            return "success";
        }

        @ApiOperation(value="删除用户", notes="根据url的id来指定删除对象")
        @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long")
        @RequestMapping(value="/{id}", method=RequestMethod.DELETE)
        public String deleteUser(@PathVariable Long id) {
            users.remove(id);
            return "success";
        }

    }
    ```

## 常用注解

- `@Api()`
    用于类，表示标识这个类是swagger的资源。
    `description` controller描述。
    `tags` controller别名，如果有多个值，会生成多个list。
- `@ApiOperation()`
    用于方法，表示一个http请求的操作。
    `value` 用于方法描述
    `notes` 用于提示内容
- `@ApiImplicitParam()`
    用于方法，表示单独的请求参数。
    `name` 参数名
    `value` 参数描述
    `dataType` 数据类型
    `paramType` 请求类型，`query`表示get请求
    `required` 是否必填
- `@ApiImplicitParams()`
    用于方法，包含多个`@ApiImplicitParam`
- `@ApiIgnore()`
    用于类或者方法上，可以不被swagger显示在页面上。
- `@ApiModel()`
    用于类，表示对类进行说明，用于参数用实体类接收。
    `description` 类描述 
- `@ApiModelProperty()`
    用于方法，字段，表示对model属性的说明或者数据操作更改。
    `value` 字段说明
    `example` 举例说明
    `hidden` 隐藏
- `@ApiIgnore()`
    用于类，方法，方法参数，表示这个方法或者类被忽略。