---
title: SpringBoot 技巧
categories:
- 后端开发
- SpringBoot 手册

date: 2021-06-14
---

#### 依赖初始化
场景：现在有一个静态类里面引用了一个 SpringBoot 对象注入，然后在使用这个静态类的时候对象还没有初始化。

```java
@Component
public class GlobalContext {

    public static GlobalContext instance;

    @PostConstruct
    public void init() {
        System.out.println("--------------------------GlobalContext--------------------------------");
        instance = this;
    }

    public static void setLcnConnection(String groupId, LcnConnectionProxy proxy) {
        instance._setLcnConnection(groupId, proxy);
    }

    public static LcnConnectionProxy getLcnConnection(String groupId) {
        return instance._getLcnConnection(groupId);
    }

    @Autowired
    private AttachmentCache attachmentCache;

    public void _setLcnConnection(String groupId, LcnConnectionProxy proxy) {
        // ..
    }

    public LcnConnectionProxy _getLcnConnection(String groupId) {
        // ..
    }
}
```

期望：在使用静态的时候里面依赖的那个 SpringBoot 对象已经初始化完成，最好是懒加载形式的。

解决：

#### 延迟依赖注入
场景：当我们使用 `@Service` 之类的注解修饰一个对象的时候，SpringBoot 在启动时会自动初始化它们。

期望：当我们真正使用这个对象的时候，才进行初始化，提高启动速度。

解决：使用 `@ComponentScan` 注解的 `lazyInit` 属性。

参考：[Spring Boot-延迟依赖注入](https://blog.csdn.net/weixin_38106322/article/details/107850813)