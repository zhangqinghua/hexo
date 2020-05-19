---
title: Chapter 1.3 基础 Redis 实现分布式锁

categories:
- 分布式

date: 2020-05-18 00:00:13
---
本篇文章主要介绍基于 Redis 如何实现分布式锁。

## setnx + expire 命令
Redis 的 **setnx** 命令 **setnx key value** 将 key 设置为 value，当键不存在时，才能成功，若键存在，什么也不做，成功返回 1，失败返回 0。**setnx** 实际上就是 SET IF NOT EXISTS 的缩写。

因为分布式锁还需要超时机制，所以我们利用 **expire** 命令来设置，所以利用 **setnx** + **expire** 命令的核心代码如下：

```java
public boolean tryLock(String key, String requset, int timeout) {
    Long result = jedis.setnx(key, requset);
    // result = 1时，设置成功，否则设置失败
    if (result == 1L) {
        return jedis.expire(key, timeout) == 1L;
    } else {
        return false;
    }
}
```

实际上上面的步骤是有问题的，**setnx** 和 **expire** 是分开的两步操作，不具有原子性，如果执行完第一条指令应用异常或者重启了，锁将无法过期。

一种改善方案就是使用 Lua 脚本来保证原子性（包含 **setnx** 和 **expire** 两条指令）。

## 使用 Lua 脚本
使用 Lua 脚本，包含 **setnx** 和 **expire** 两条指令：

```java
public boolean tryLock_with_lua(String key, String UniqueId, int seconds) {
    String lua_scripts = "if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then" +
            "redis.call('expire',KEYS[1],ARGV[2]) return 1 else return 0 end";
    List<String> keys = new ArrayList<>();
    List<String> values = new ArrayList<>();
    keys.add(key);
    values.add(UniqueId);
    values.add(String.valueOf(seconds));
    Object result = jedis.eval(lua_scripts, keys, values);
    //判断是否成功
    return result.equals(1L);
}
```

## set key value 命令
Redis 在 2.6.12 版本开始，为 **set** 命令增加一系列选项：

```bash
SET key value [EX seconds] [PX milliseconds][NX|XX]
```

- EX: 设定过期时间，单位为秒

- PX: 设定过期时间，单位为毫秒

- NX: 仅当 key 不存在时设置值

- XX: 仅当 key 存在时设置值

**set** 命令的 NX 选项，就等同于 **setnx** 命令，代码过程如下：

```java
public boolean tryLock_with_set(String key, String UniqueId, int seconds) {
    return "OK".equals(jedis.set(key, UniqueId, "NX", "EX", seconds));
}
```

value 必须要具有唯一性，我们可以用 UUID 来做，设置随机字符串保证唯一性，至于为什么要保证唯一性？假如 value 不是随机字符串，而是一个固定值，那么就可能存在下面的问题：
1. 客户端 1 获取锁成功
1. 客户端 1 在某个操作上阻塞了太长时间
1. 设置的 key 过期了，锁自动释放了
1. 客户端 2 获取到了对应同一个资源的锁
1. 客户端 1 从阻塞中恢复过来，因为 value 值一样，所以执行释放锁操作时就会释放掉客户端 2 持有的锁，这样就会造成问题

所以通常来说，在释放锁时，我们需要对value进行验证。也就是说我们在获取锁的时候需要设置一个 value，不能直接用 **del key** 这种粗暴的方式，因为直接 **del key** 任何客户端都可以进行解锁了，所以解锁时，我们需要判断锁是否是自己的，基于 value 值来判断，代码如下：

```java
public boolean releaseLock_with_lua(String key,String value) {
    String luaScript = "if redis.call('get',KEYS[1]) == ARGV[1] then " +
            "return redis.call('del',KEYS[1]) else return 0 end";
    return jedis.eval(luaScript, 
                      Collections.singletonList(key), 
                      Collections.singletonList(value)).equals(1L);
}
```

这里使用Lua脚本的方式，尽量保证原子性。

使用 **set key value** 命令看上去很 OK，实际上在 Redis 集群的时候也会出现问题，比如说 A 客户端在 Redis 的 master 节点上拿到了锁，但是这个加锁的 key 还没有同步到 slave 节点，master 故障，发生故障转移，一个 slave 节点升级为 master 节点，B 客户端也可以获取同个 key 的锁，但客户端 A 也已经拿到锁了，这就导致多个客户端都拿到锁。

所以针对Redis集群这种情况，还有其他方案

## Redlock 算法
Redis 作者 antirez 基于分布式环境下提出了一种更高级的分布式锁的实现 Redlock。

假设有 5 个独立的 Redis 节点（注意这里的节点可以是 5 个 Redis 单 master 实例，也可以是 5 个 Redis Cluster 集群，但并不是有 5 个主节点的 cluster 集群）：
1. 获取当前 Unix 时间，以毫秒为单位

1. 依次尝试从 5 个实例，使用相同的 key 和具有唯一性的 value（例如 UUID ）获取锁，当向 Redis 请求获取锁时，客户端应该设置一个网络连接和响应超时时间，这个超时时间应用小于锁的失效时间，例如你的锁自动失效时间为 10s，则超时时间应该在 5 ~ 50 毫秒之间，这样可以避免服务器端 Redis 已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务端没有在规定时间内响应，客户端应该尽快尝试去另外一个 Redis 实例请求获取锁

1. 客户端使用当前时间减去开始获取锁时间（步骤 1 记录的时间）就得到获取锁使用的时间，当且仅当从大多数（N / 2 + 1，这里是 3 个节点）的 Redis 节点都取到锁，并且使用的时间小于锁失败时间时，锁才算获取成功

1. 如果取到了锁，key 的真正有效时间等于有效时间减去获取锁所使用的时间（步骤 3 计算的结果）

1. 如果某些原因，获取锁失败（没有在至少 N / 2 + 1 个 Redis 实例取到锁或者取锁时间已经超过了有效时间），客户端应该在所有的 Redis 实例上进行解锁（即便某些 Redis 实例根本就没有加锁成功，防止某些节点获取到锁但是客户端没有得到响应而导致接下来的一段时间不能被重新获取锁）

具体实现参考： [Redlock：Redis分布式锁最牛逼的实现](https://mp.weixin.qq.com/s?__biz=MzU5ODUwNzY1Nw==&mid=2247484155&idx=1&sn=0c73f45f2f641ba0bf4399f57170ac9b&scene=21#wechat_redirect)

## Redisson 实现
对于 Java 用户而言，我们经常使用 Jedis，Jedis 是 Redis 的 Java 客户端，除了 Jedis 之外，Redisson 也是 Java 的客户端，Jedis 是阻塞式 I/O，而 Redisson 底层使用 Netty 可以实现非阻塞 I/O，该客户端封装了锁的，继承了 J.U.C 的 **Lock** 接口，所以我们可以像使用 **ReentrantLock** 一样使用 Redisson，具体使用过程如下。

首先加入POM依赖：
```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.10.6</version>
</dependency>
```

使用 Redisson，代码如下（与使用 **ReentrantLock** 类似）：
```java
// 1. 配置文件
Config config = new Config();
config.useSingleServer()
        .setAddress("redis://127.0.0.1:6379")
        .setPassword(RedisConfig.PASSWORD)
        .setDatabase(0);
//2. 构造RedissonClient
RedissonClient redissonClient = Redisson.create(config);

//3. 设置锁定资源名称
RLock lock = redissonClient.getLock("redlock");
lock.lock();
try {
    System.out.println("获取锁成功，实现业务逻辑");
    Thread.sleep(10000);
} catch (InterruptedException e) {
    e.printStackTrace();
} finally {
    lock.unlock();
}
```

## 一个简易的分布式锁
下面利用 SpringBoot + Jedis + AOP 的组合来实现一个简易的分布式锁。

#### 自定义注解
自定义一个注解，被注解的方法会执行获取分布式锁的逻辑。

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface RedisLock {
    /**
     * 业务键
     *
     * @return
     */
    String key();
    /**
     * 锁的过期秒数,默认是5秒
     *
     * @return
     */
    int expire() default 5;

    /**
     * 尝试加锁，最多等待时间
     *
     * @return
     */
    long waitTime() default Long.MIN_VALUE;
    /**
     * 锁的超时时间单位
     *
     * @return
     */
    TimeUnit timeUnit() default TimeUnit.SECONDS;
}
```

#### AOP 拦截器实现
在AOP中我们去执行获取分布式锁和释放分布式锁的逻辑。

```java
@Aspect
@Component
public class LockMethodAspect {
    
    @Autowired
    private JedisUtil jedisUtil;
    @Autowired
    private RedisLockHelper redisLockHelper;
  
    private Logger logger = LoggerFactory.getLogger(LockMethodAspect.class);

    @Around("@annotation(com.redis.lock.annotation.RedisLock)")
    public Object around(ProceedingJoinPoint joinPoint) {
        Jedis jedis = jedisUtil.getJedis();
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();

        RedisLock redisLock = method.getAnnotation(RedisLock.class);
        String value = UUID.randomUUID().toString();
        String key = redisLock.key();
        try {
            final boolean islock = redisLockHelper.lock(jedis,
                                                        key, 
                                                        value, 
                                                        redisLock.expire(), 
                                                        redisLock.timeUnit());
            logger.info("isLock : {}",islock);
            if (!islock) {
                logger.error("获取锁失败");
                throw new RuntimeException("获取锁失败");
            }
            try {
                return joinPoint.proceed();
            } catch (Throwable throwable) {
                throw new RuntimeException("系统异常");
            }
        }  finally {
            logger.info("释放锁");
            redisLockHelper.unlock(jedis,key, value);
            jedis.close();
        }
    }
}
```

#### Redis 实现分布式锁核心类
```java
@Component
public class RedisLockHelper {
    private long sleepTime = 100;
    /**
     * 直接使用setnx + expire方式获取分布式锁
     * 非原子性
     *
     * @param key
     * @param value
     * @param timeout
     * @return
     */
    public boolean lock_setnx(Jedis jedis,String key, String value, int timeout) {
        Long result = jedis.setnx(key, value);
        // result = 1时，设置成功，否则设置失败
        if (result == 1L) {
            return jedis.expire(key, timeout) == 1L;
        } else {
            return false;
        }
    }

    /**
     * 使用Lua脚本，脚本中使用setnex+expire命令进行加锁操作
     *
     * @param jedis
     * @param key
     * @param UniqueId
     * @param seconds
     * @return
     */
    public boolean Lock_with_lua(Jedis jedis,String key, String UniqueId, int seconds) {
        String lua_scripts = "if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then" +
                "redis.call('expire',KEYS[1],ARGV[2]) return 1 else return 0 end";
        List<String> keys = new ArrayList<>();
        List<String> values = new ArrayList<>();
        keys.add(key);
        values.add(UniqueId);
        values.add(String.valueOf(seconds));
        Object result = jedis.eval(lua_scripts, keys, values);
        //判断是否成功
        return result.equals(1L);
    }

    /**
     * 在Redis的2.6.12及以后中,使用 set key value [NX] [EX] 命令
     *
     * @param key
     * @param value
     * @param timeout
     * @return
     */
    public boolean lock(Jedis jedis,String key, String value, int timeout, TimeUnit timeUnit) {
        long seconds = timeUnit.toSeconds(timeout);
        return "OK".equals(jedis.set(key, value, "NX", "EX", seconds));
    }

    /**
     * 自定义获取锁的超时时间
     *
     * @param jedis
     * @param key
     * @param value
     * @param timeout
     * @param waitTime
     * @param timeUnit
     * @return
     * @throws InterruptedException
     */
    public boolean lock_with_waitTime(Jedis jedis,
                                      String key, 
                                      String value, 
                                      int timeout, 
                                      long waitTime,
                                      TimeUnit timeUnit) throws InterruptedException {
        long seconds = timeUnit.toSeconds(timeout);
        while (waitTime >= 0) {
            String result = jedis.set(key, value, "nx", "ex", seconds);
            if ("OK".equals(result)) {
                return true;
            }
            waitTime -= sleepTime;
            Thread.sleep(sleepTime);
        }
        return false;
    }
    /**
     * 错误的解锁方法—直接删除key
     *
     * @param key
     */
    public void unlock_with_del(Jedis jedis,String key) {
        jedis.del(key);
    }

    /**
     * 使用Lua脚本进行解锁操纵，解锁的时候验证value值
     *
     * @param jedis
     * @param key
     * @param value
     * @return
     */
    public boolean unlock(Jedis jedis,String key,String value) {
        String luaScript = "if redis.call('get',KEYS[1]) == ARGV[1] then " +
                "return redis.call('del',KEYS[1]) else return 0 end";
        return jedis.eval(luaScript, 
                          Collections.singletonList(key), 
                          Collections.singletonList(value)).equals(1L);
    }
}
```

#### TestController 测试
定义一个 **TestController** 来测试我们实现的分布式锁。

```java
@RestController
public class TestController {

    @RedisLock(key = "redis_lock")
    @GetMapping("/index")
    public String index() {
        return "index";
    }
}
```