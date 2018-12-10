---
title: Redis

categories:
- Java

date: 2018-03-27 01:00:00
---

Redis笔记

# 特点/优点

1. 足够简单和稳定
2. 支持丰富的数据结构
3. 内存存储读写性能优秀
4. 提供持久化支持
5. 支持部分事物
6. 集群

# 安装

1. 下载
2. 解压
3. `make`，需要安装gcc，`make distclean` 清理上一次make文件

# 启动/关闭

- `redis-server &` 后台启动
- `redis-cli shutdown` 关闭
- `kill pid / kill -9 pid` kill杀掉比较粗暴

# 客户端

1. 命令行客户端`redis-cli -h 12.0.0.1 -p 6379`
2. 远程客户端 要配置下，重启，在window运行界面程序
3. 编程客户端 Jedis, Jar包，让Java访问

# 基本使用

1. `select 1` 使用1号库，默认16个库，默认使用0号库
1. `flushall` 删除所有数据
1. `flushdb` 删除当前库数据
1. `config get *` 获取所有key
1. `exit/quit` 退出
1. `dbsize` 查看当前库key的数量
1. `info redis` 服务器统计信息

# 数据类型

Redis 支持6种数据类型。

## string

字符串

- `get set`
- `incr decr` 数字+-1，字符串无效，不存在则设置为0再操作。
- `setex` set expire 缩写，设置过期时间（秒）
- `setnx` set is not exists 如果不存在再set
- `getset` 设置key的值为value，并返回旧值

## hash

适合存储对象 key (key1 value1 key2 value2)

- `hset k1 id 100`
- `hmset k2 id 100 name zhangsan age 20`
- `hdel k2 age` 删除key的某一个字段
- `hkeys k2`  查出k2所有的key
- `kvals k2` 查出k2所有value

## list

列表，按照插入顺序排序，可以插入在列表的头部(左边)或尾部(右边)，允许重复

- `lpush rpush` 插入一个或多个值在列表左边/右边
- `lrange k1 0 -1` 取指定区间中的所有元素
- `loop k1 roop k1` 弹出 从左边获取一个元素，并将元素删除
- `lindex k1 3` 获取列表第3个元素
- `len` 获取列表key的长度
- `lrem k1 1 4` 删除1-4之间的以实

## set

类似list，不允许重复，无序

- `sadd k1 1 2 3 4`
- `smembers k1 = 1 2 3 4`
- `scard k1` 获取集合里的元素个数 
- `srem k1 5` 删除集合里面一个或多个元素

## zset

有序集合，不允许重复

- `zadd k1 100 beijing 90 shanghai 80 shenzhen`
- `zrange k1 0 -1` 获取集合指定区间内的值，按照score从小到大排序
- `zrank k1 shanghai == 0` 获取排名 
- `zrevrank k1 shanghai == 1` 从大到小排名

# key命令

1. `keys * keys kk*` 列出所有key
2. `exists k1` 是否存在key
3. `move k1 5` 移动k1到5库
4. `expire key seconds` 设置key的过期时间
5. `ttl k1` 查看还有多少秒过期
6. `type key` 查看key所存储的类型
7. `del k1` 删除key

# Java使用Redis

1. maven jedis依赖
    ```xml
    <dependency>
        <groupId>org.redisson</groupId>
        <artifactId>redisson</artifactId>
        <version>1.0.2</version>
    </dependency>
    ```

2. 连接Redis服务器
    ```java
    //连接本地的 Redis 服务
    Jedis jedis = new Jedis("localhost");
    System.out.println("连接成功");
    //查看服务是否运行
    System.out.println("服务正在运行: "+jedis.ping());
    ```

3. 示例
    ```Java
    //连接本地的 Redis 服务
    Jedis jedis = new Jedis("localhost");
    System.out.println("连接成功");
    //设置 redis 字符串数据
    jedis.set("runoobkey", "www.runoob.com");
    // 获取存储的数据并输出
    System.out.println("redis 存储的字符串为: "+ jedis.get("runoobkey"));
    ```

# 发布和订阅

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。消息队列一般用RabbitMQ

![示意图](001.png)

## 流程

1. 开启4个redis客户端，1个发布者，3个订阅者
2. 让订阅者订阅某个频道
    `subscribe channel`
3. 让发布者向频道发布消息
    `publish channel "Welcome to redis"`
4. 观察消息的发布和订阅情况

## Java实现

```java
public class RedisPublisher{ 
    public static void main(String[] args) {
        // 链接到redis
        Jedis jedis = new Jedis("192.168.157.128", 6379);
        // 向redis消息频道发布一条消息
        jedis.publish("channel", "HelloWorld");
        jedis.close();
    }
}
```

```Java
public class RedisSubscriber JedisPubSub{
    /**
     * 重写接收消息的方法
     */
    @Override
    publicvoid onMessage(String channel, String message) {
        System.out.println("onMessage: channel["+channel+"], message["+message+"]");
    }

    public static void main(String[] args) {
        // 链接到redis
        Jedis jedis = new Jedis("192.168.157.128", 6379);
        // 创建一个JedisPubSub对象
        RedisSubscriber rb = new RedisSubscriber();
        // 从Redis消息频道订阅消息，此程序会一直运行
        jedis.subscribe(rb, "channel");
    }
}
```

# 事物

事物是指一系列操作步骤，要么全部执行完成，要么全部不执行。
![事物](002.png)

redis中的事物是一组命令的集合，至少是2个或以上的命令，redis事物保证这些命令在执行中不会被其它操作打断。

1. 正常情况
    ![正常情况](003.png)
1. 异常情况
    ![异常情况](004.png)
1. 例外情况
    ![例外情况](005.png)
1. 放弃情况
    ![放弃情况](006.png)
1. 复杂情况
    ![复杂情况](007.png)

# 持久化

Redis数据存储在内存中，内存是瞬时的，如果linux死机或重启，又或者Redis崩溃或重启，所有的内存数据将丢失。为解决这个问题，Redis提供2种机制对数据进行持久化存储，便于发生事故后能迅速恢复。

## RDB方式

![RDB方式](008.png)

## AOF方式

![RDB方式](009.png)
![RDB方式](010.png)

# 集群

![集群](011.png)

## 如何实现

![集群](012.png)

## 容灾处理

1. 冷处理
    当Master服务出现故障，需要手动将Slave中的一个提升为Master，剩下的Slave挂载至新的Master。
    - `slaveof no one` 将一台Slave服务器提升至Master
    - `slaveof 127.0.0.1` 将Slave挂载至新的Master
2. 高可用哨兵
    Sentine哨兵是Redis官方提供的高可用方案，可以用它来监控多个Redis服务的允许情况。

# 安全

- 设置密码
- 绑定IP
- 命令禁止或重命名
- 修改默认端口号

# SpringBoot 整合

1. 引入依赖
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-redis</artifactId>
    </dependency>
    ```
2. 参数配置
    ```properties
    # REDIS (RedisProperties)
    # Redis数据库索引（默认为0）
    spring.redis.database=0
    # Redis服务器地址
    spring.redis.host=localhost
    # Redis服务器连接端口
    spring.redis.port=6379
    # Redis服务器连接密码（默认为空）
    spring.redis.password=
    # 连接池最大连接数（使用负值表示没有限制）
    spring.redis.pool.max-active=8
    # 连接池最大阻塞等待时间（使用负值表示没有限制）
    spring.redis.pool.max-wait=-1
    # 连接池中的最大空闲连接
    spring.redis.pool.max-idle=8
    # 连接池中的最小空闲连接
    spring.redis.pool.min-idle=0
    # 连接超时时间（毫秒）
    spring.redis.timeout=0
    ```
3. 测试访问
    ```java
    @RunWith(SpringJUnit4ClassRunner.class)
    @SpringApplicationConfiguration(Application.class)
    public class ApplicationTests {

        @Autowired
        private StringRedisTemplate stringRedisTemplate;

        @Test
        public void test() throws Exception {

            // 保存字符串
            stringRedisTemplate.opsForValue().set("aaa", "111");
            Assert.assertEquals("111", stringRedisTemplate.opsForValue().get("aaa"));
        }
    }
    ```