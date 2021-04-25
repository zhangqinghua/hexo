---
title: Redis 常用命令

categories:
- 数据存储
- Redis 教程

tags:
- redis

date: 2020-05-28
---
## 

## 数据类型命令

#### string
```
# 1. 添加值
set key value

# 2. 取值
get key

# 3. 批量操作
mset name zhangsan age 18
mget name age

# 4. 自增自减一
incr age
decr age

# 5. 自增自减指定数量
incrby age 2
decrby age 2

# 6. 设置值的同时，指定生存空间
setex age second 10

# 7. 设置值，如果当前 key 不存在的话
setnx age 19

# 8. 在 key 对应的 value 后面追加内存
append name lisi

# 9. 查看 value 字符串的长度
strlen name
```



#### hash
```
# 1. 存储数据
hset person name zhangsan
hset person age  18

# 2. 获取数据
hget person name

# 3. 批量操作
hmset person name zhangsan person age 18
hmget person name person age

# 4. 自增指定数量
hincrby person age 1

# 5. 设置值，如果 key field 不存在
hsetnx person age 18

# 6. 检查 field 是否存在
hexists person age

# 7. 删除 key 下面的一个或多个 field
hdel pseron name age

# 8. 获取当前hash结构中的全部field和值
hgetall person

# 9. 获取当前hash结构中的全部field
hkeys person

# 10. 获取当前hash结构中的全部值
hvals person

# 11. 获取当前hash结构中的field数量
hlen person
```

#### list
```
# 1. 存储数据（从左侧插入数据，从右侧插入数据）
lpush names zhangsan lisi wangwu
rpush names zhangsan lisi wangwu

# 2. 存储数据（如果key不存在，什么事都不做。如果key存在，但是不是lsit结构，什么都不做）
lpushx names zhangsan lisi wangwu
rpsuhx names zhangsan lisi wangwu

# 3. 修改数据（在存储数据时，指定好你的索引位置，覆盖之前的索引位置的数据，index超过整个列表的长度，也会失败）
lset names 1 lisi2

# 4. 弹栈方式获取数据（左侧弹出数据，右侧弹出数据）
lpop names
rpop names

# 5. 获取整个列表的长度
llen names

# 6. 获取指定索引位置的数据
lindex names 1

# 7. 获取指定范围的数据（从 0 开始，-1 表示最后一个，-2 表示倒数第二个）
lrange name 0 3

# 8. 删除列表中的数据（删除指定数量的value，count > 0 从左侧向右删除。count < 0 从右侧向左侧删除。count == 0 删除所有数据）
lrem  names  3 value
lrem  names -3 value
lrem  names  0 value

# 9. 保留列表中的数据（保留指定索引范围内的数据，超过此范围的数据将被删除）
ltrim names 0 3

# 10. 将一个列表中最后的一个数据，插入到另外一个列表的头部位置
rpoplpush name1s name2s
```

#### set
```
# 1. 存储数据
sadd names zhangsan lisi wangwu

# 2. 获取数据（获取全部数据）
smembers names

# 3. 获取数据（随机获取 n 个数据的同时，移除数据）
spop names 3

# 4. 交集（取多个set集合交集）
sinter name1s name2s

# 5. 并集（获取全部集合中的数据）
sunion name1s name2s

# 6. 差集（获取多个集合中不一样的数据）
sdiff name1s name2s

# 7. 删除数据
srem names zhangsan

# 8. 查看当前set集合中是否包含这个值
sismember names zhangsan
```

#### zset
```
# 1. 添加数据（score必须是数值，member不允许重复）
zadd names 10 zhangsan 11 lisi

# 2. 修改member的分数（如果member是存在于key中的，正常增加分数。如果不存在，这个命令相当于zadd）
zincrby names 5 zhangsan

# 3. 查看指定的member的分数
zscore names zhangsan

# 4. 获取zset中的数据的数量
zcard names

# 5. 根据score的范围查询member数量
zcount names 10 99

# 6. 删除zset中的成员
zrem names zhangsan lisi

# 7. 根据分数从小到打排序，获取指定范围内的数据（withscores 如果添加这个参数，那么会返回member对应的分数）
zrange names 0 10
zrange names 0 10 withscores

# 8. 根据分数从打到小排序，获取指定范围内的数据（withscores 如果添加这个参数，那么会返回member对应的分数）
zrevrage names 0 10 
zrevrage names 0 10 withscores

# 9. 根据分数的范围取获取member（添加limt，就和MySQL中的limit一样，括号表示 > 0 的意识，不包含0）
zrangebyscore names 0 10 
zrangebyscore names 0 10 withscores
zrangebyscore names 0 10 withscores limit 0 10
zrangebyscore names (0 10 withscores limit 0 10

# 9. 根据最小的分数、最大的分数取获取member（添加limt，就和MySQL中的limit一样）
zrangebyscore names +inf -inf withscores limit 0 10
```
![](https://www.ofcoder.com/images/middleware/redis_zset_1.png)

## key 的常用命令
```
# 1. 查看 Redis 中的全部 key（pattern：*、xxx*、*xxxx）
key *
key name*
key *name

# 2. 查看某一个key是否存在（1 存在 0 不存在）
exists name

# 3. 删除key
del name
del name1 name2

# 4. 设置key的生存时间（秒、毫秒）
expire  name 10
pexpire name milliseconds 10000

# 5. 设置key的截止时间（秒、毫秒）（时间戳）
expireat  name 1010100101
pexpireat name 1010100101000

# 6. 查看key的剩余生存时间（秒、毫秒）（-2 key不存在 -1 key 没有设置生存时间）
ttl  name
pttl name

# 7. 移除key的生存时间（1 移除生存 0 key不存在或没有设置生存时间）
persist name
```

## 持久化命令
```bash
127.0.0.1:6379> save
OK
```