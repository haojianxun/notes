# redis文档

[TOC]

REDIS: REmote DIctionary Server

## 环境搭建

- 下载

  ```
   wget http://download.redis.io/releases/redis-3.0.3.tar.gz
   git clone https://github.com/antirez/redis.git
  ```

- 编译安装

  ```
   cd redis-3.0.3
   make
   sudo make install 
  ```

- 启动停止

  ```
   redis-server redis.conf
    redis-cli -p 6379 shutdown / kill pid
  ```

- 配置文件

  ```
  - daemonize yes
  – port 6379
  – logfile /home/redis/data_6379/redis.log
  – pidfile /home/redis/data_6379/redis.pid
  – maxmemory 2gb
  – dbfilename dump.rdb
  – dir /home/redis/data_6379/
  – appendonly yes
  – appendfsync everysec
  ```

## 数据类型

-  REDIS_STRING : 字符串
-  REDIS_LIST : 列表
-  REDIS_SET : 集合
- REDIS_ZSET : 有序集合
- REDIS_HASH : 哈希表

采用type命令来查看数据类型

### key

1. key是字符串类型
2. 不是binary safe
3. 建议用":"来分隔,固定部分最好在后面

> 比如 apple:fru ,  banna:fru  orange:fru 

#### key命令

```
EXISTS
DEL
KEYS
EXPIRE / EXPIREAT / TTL / PERSIST
PEXPIRE / PEXPIREAT / PTTL
RENAME / RENAMENX
DUMP / RESTORE
MOVE / MIGRATE
```

### string

1. 最常用的数据类型
2. 是binary safe   //可以存储图片,音频 , 视频等
3. value最大上限512M , 建议不要超过1M

#### string命令

```
GET / SET / SETEX / SETNX
MGET / MSET / MSETNX
APPEND
INCR / DECR / INCRBY / DECRBY / INCRBYFLOAT
GETBIT / SETBIT / BITCOUNT / BITOP
STRLEN
```

### list

- 是一个双端链表
- 按插入顺序进行排序
- list 的最大长度是2^32 – 1
- 很多操作的时间复杂度都是O(1)

#### list命令

```
LPUSH / LPOP / LPUSHX / RPUSH / RPOP
BLPOP / BRPOP
LINSERT / LSET / LREM
LRANGE / LLEN
RPOPLPUSH / BRPOPLPUSH
```

### set

- set 是 string 类型的无序集合
- Redis 中的 set 不允许有重复的成员
- 可以做集合和集合之间的操作
-  集合的最大长度是2^32 - 1

#### set命令

```
SADD / SREM / SPOP / SMOVE
SISMEMBER / SMEMBERS / SRANDMEMBER
SCARD
SDIFF / SDIFFSTORE
SINTER / SINTERSTORE
SUNION / SUNIONSTORE
```

### sorted set

-  sorted set 是 string 类型的有序集合
- 不允许有重复的成员，不同成员可以有相同的score
- 集合中每一个成员都会关联一个double类型的score用于排序
- 可以做集合和集合之间的操作
- 集合的最大长度是2^32 - 1

#### sorted set命令

```
ZADD / ZREM
ZCOUNT / ZCARD
ZRANK / ZINCRBY / ZSCORE
```

案例:

 各种系统中排名的存储

- 金钱排名
- 经验排名
- 等级排名

### hash

- Hash 类型是一个 string 类型的 field 和 value 的映射表
- 底层通过 ziplist 或是 hash table 实现
- 具体采用何种数据结构实现是由 field 的数量和 value的长度决定
- Field 和 value 对的数量不超过 2^32-1

#### hash命令

```
HSET / HGET / HMSET / HMGET / HDEL
HSETNX
HKEYS / HVALS / HGETALL
HINCRBY / HINCRBYFLOAT
HLEN
```

### hyperloglog

- 主要用于大数量集合的基数（cardinality）统计
- 是不准确统计
- 存储开销特别少（12KB）

#### heperloglog命令

```
PFADD
PFCOUNT
PFMERGE
```

## 事务

- 一般事务都具有 ACID 属性
- Redis 的事务是不完整的事务
-  只能保证一个事务中的命令连续执行
- 事务中间命令出错，并不能回滚
- 事务执行中服务器挂了，也不能回滚

### 事务命令

```
 WATCH counter1 counter2
MULTI
INCR counter1
INCE counter2
EXEC
DISCARD
UNWATCH
```

## 脚本编程

- 从 Redis2.6 开始支持 Lua 脚本编程
- 可以用这种方式实现一些简单事务
- 类似于 RDBMS 中的存储过程
- 节省流量

 一个例子
用 Lua 脚本编程实现一个简单的转账的事务：
bob给 smith 转账了 20 块钱

```
EVAL "local a = redis.call('GET', 'bob:account');

local b = redis.call('GET', 'smith:account'); if not a

or not b then return 1 end; a = a - 20; b = b + 20; if

not redis.call('MSET', 'bob:account', a,

'smith:account', b) then return 2 end; return 0;" 0

```

上面的脚本存在问题
–因为一个交易，smith 需要给 bob 转 30 块钱
 修改之后的脚本

```
EVAL "local a = redis.call('GET', KEYS[1]); local b =

redis.call('GET', KEYS[2]); if not a or not b then

return 1 end; if not redis.call('MSET', KEYS[1], a -

ARGV[1], KEYS[2], b + ARGV[1]) then return 2 end;

return 0;" 2 smith:account bob:account 30

```



上面转账的例子没有考虑到账户余额不足的情况
脚本经过 sha 编码之后的名字还是不容易记住，如何解决
脚本加载到缓存之后，如果 Redis 重启之后，脚本丢失，如何解决
文本持久化
Redis aof 持久化
为避免脚本中出现死循环，Redis 通过下面的配置
实现脚本执行超时控制
`lua-time-limit` 

## 发布订阅

发布订阅（pub/sub）是一种消息通信方式 , 和消息队列类似
有两个角色执行不同的动作

- 消息发布者(publisher)向某个频道(channal)发布消息
- 消息订阅者(subscriber)订阅某个频道的消息

发布订阅不涉及数据的存储，和 Redis 数据库

其他功能完全隔离

### 常用命令

```
PUBLISH / SUBSCRIBE / UNSUBSCRIBE
PSUBSCRIBE / PUNSUBSCRIBE
PUBSUB
```

## 持久存储

Redis 支持两种持久化方式

- Snapshotting (default)

  ​	dump.rdb

- AOF 方式

  ​	appendonly.aof

### snapshotting

快照是默认的存储方式，这种方式是将内存中的数据以快照的方式写入到二进制文件中

触发条件

- 自动触发

  - 如下配置

    `save 900 1`

    `save 300 10`

    `save 60 10000`

- 手动触发

  - SAVE
  - BGSAVE
  - SLAVEOF

原理

- 触发快照存储之后，Redis 会 fork 出一个子进程
  1. 父进程继续处理 client 的请求
  2. 子进程复制将内存中的数据写到临时文件中
- 父、子进程共享相同的物理页面，当父进程接收到 client 端的请求需要修改数据时，OS 会为要修改的页面创建一个副本
- 子进程写完，替换点旧的快照文件，释放空间，
- 结束进程

 注意事项

- SAVE 命令的处理逻辑和上面不一样

​	阻塞式的——快照期间阻塞 client 请求

- 不要将大部分内存都分配给一个 Redis 实例
- 使用多个 Redis 实例充分利用内存
- 快照的时候非常不推荐使用“SAVE”命令

缺点
做快照需要时间，在做快照的过程中 Redis 做更改的那部分数据如何保证不丢失？

- aof

### AOF

Append-Only File

配置文件开启,默认是关闭

```
appendfsync no
appendfsync everysec
appendfsync always
```

缺点

-  aof 文件较大
-  从 aof 文件恢复数据太久了

如何解决

-  从 aof 文件恢复数据太久了

注意:***BGSAVE 和 BGREWRITEAOF 最好不要同时执行***



 Redis 灾难恢复

- 如果配置了 AOF 方式，那么首选从 AOF 文件恢复
- 如果没有配置 AOF，那么从快照文件（dump.rdb）恢复
- 快照作为全量备份, AOF 做增量备份

## 主从复制

 Redis 主从支持多种模式：M-S，S-M-S，M-S-S，据说支持 DAG！

- 使得读写分离成为可能
- 主从之间异步复制
- 有这样一种模式，为提升写性能，Master 不做持久化，Slave 做持久化

### **如何配置 1M-1S 结构**

1. 启动一个 Redis 实例做为 Master
2. 启动一个 Redis 实例做为 Slave

 在 Slave 实例上执行下面的命令
`SLAVEOF Master_ip Master_port`

实现1主1从结构

### **如何解除**

在 Slave 实例上执行下面的命令
`SLAVEOF NO ONE` 

### 实现原理

1. Slave 通过‘SLAVEOF’命令请求 Master
2. Master 接收到同步请求，会 fork 一个子进程对当前数据库做一个快照,同时将 clients 发过来的写请求数据记录在 slave-buffer 中
3. Master 快照做好之后发给 Slave
4. Slave 将收到的快照，加载到内存中
5. Master 会将 slave-buffer 中记录的新数据发给Slave
6. 经过一段时间之后，完成数据同步

注意事项:

 slave-buffer 大小的设定需要注意

- client-output-buffer-limit slave hard soft seconds
- client-output-buffer-limit slave 256mb 64mb 60

## Redis DBA CMD

```
INFO
DBSIZE
CONFIG SET / CONFIG GET
FLUSHDB / FLUSHALL
MONITOR / SLOWLOG GET / SLOWLOG SET
TIME / PING
CLIENT LIST / CLIENT KILL
SAVE / BGSAVE / BGREWRITESAVE / LASTSAVE
– SHUTDOWN
```

## Client 编程

C: https://github.com/redis/hiredis

C++: https://github.com/jrk/redis-cplusplus-client

Java: https://github.com/xetorthio/jedis

Lua: https://github.com/nrk/redis-lua

Nodejs: https://github.com/fictorial/redis-node-client

Scala: https://github.com/top10/scala-redis-client

Php: https://github.com/nrk/phpiredis



##  Redis-Sentinel

Redis 高可用方案
适用于1主多从的情况，Master 挂掉，Sentinel负责从多个 Slave 中选出一个作为 Master，并将其他的 Slave 连上新的 Master
使用 Raft 选举算法：https://raft.github.io/

## Redis-Cluster

从 Redis3.0 开始支持
无中心，节点之间采用 gossip 协议传播信息
数据如何分片

- 每个节点都有一个唯一标识符作为 node_id
- 将 16384 个槽位平均（也可以不平均，看你高兴）分给N 个 Redis 节点
- key 保存在哪个节点：slot_code = hash(key) mod 16384 ，然后根据上面 slot 的分配情况找到对应的 node_id
- 每个节点都可以接受 client 的请求，会将不在本节点的请求转移到对应的节点上

– C: https://github.com/redis/hiredis
– C++: https://github.com/jrk/redis-cplusplus-client
– Java: https://github.com/xetorthio/jedis
– Lua: https://github.com/nrk/redis-lua
– Nodejs: https://github.com/fictorial/redis-node-client
– Scala: https://github.com/top10/scala-redis-client
– Php: https://github.com/nrk/phpiredis

