# memcached文档

## memcached介绍

Memcached 是国外 社区 网站 LiveJournal  的开发团队开发的 高性能的分布式内
存缓存服务器。一般的使用目的是，通过缓存数据库查询结果，减少数据库访
问次数，以提高动态 Web 应用的速度、提高可扩展性

### 基于 libevent 的事件处理

libevent 是一套跨平台的事件处理接口的封装，能够兼容包括这些操作系统：
Windows/Linux/BSD/Solaris  等操作系统的的事件处理。
包装的接口包括：
poll 、 select(Windows) 、 epoll(Linux) 、 kqueue(BSD) 、 /dev/pool(Solaris)
Memcached  使用 libevent 来进行网络并发连接的处理，能够保持在很大并发情
况下，仍旧能够保持快速的响应能力。

### Memcached 的主要特点

- 基于 C/S 架构， 协议简单
- 基于 libevent 的事件处理
- 自主内存存储处理
- 基于客户端的 Memcached

### 自主的内存存储处理

- 数据 存储方式： Slab Allocation
- 数据 过期 方式： Lazy Expiration + LRU

### 数据存储方式： Slab Allocation

Slab Allocator 的基本原理是按照预先规定的大小，将分配的内存分割成特定长度的块，以完全解决内存碎片问题。
Slab Allocation 的原理相当简单。 将分配的内存分割成各种尺寸的块(chunk)，并把尺寸相同的块分成组(chunk 的集合)





Page ： 分配给 Slab 的内存空间，默认是1MB 。分配给 Slab 之后根据 slab 的大小
切分成 chunk 。

Chunk ： 用于缓存记录的内存空。

Slab Class ： 特定大小的 chunk 的组。

memcached 根据收到的数据的大小，选择最适合数据大小的 slab 。

memcached 中保存着 slab 内空闲 chunk 的列表，根据该列表选择 chunk ，然后将数据缓存于其中。



Slab Alloction  缺点

这个问题就是，由于分配的是特定长度的内存，因此无法有效利用分配的内存。例如，将 100 字节的数据缓存到 128 字节的 chunk 中，剩余的 28 字节就浪费了

### 数据过期方式

- Lazy Expiration

memcached 内部不会监视记录是否过期，而是在 get 时查看记录的时间戳，检查记录是否过期。这种技术被称为 lazy （惰性） expiration 。因此， memcached 不会在过期监视上耗费CPU 时间。

- LRU

memcached 会优先使用已超时的记录的空间，但即使如此，也会发生追加新记录时空间不足的情况，此时就要使用名为  Least Recently Used （ LRU ）机制来分配空间。顾名思义，这是删除 “ 最近最少使用 ” 的记录的机制。因此，当 memcached 的内存空间不足时（无法从 slab class  获取到新的空间时），就从最近未被使用的记录中搜索，并将其空间分配给新的记录。从缓存的实用角度来看，该模型十分理想。

### memcached语法

```
命令：
统计类：stats, stats items, stats slabs, stats sizes
存储类：set, add, replace, append, prepend
命令格式：<command name> <key> <flags> <exptime> <bytes>  <cas unique>
检索类：get, delete, incr/decr
清空：flush_all
			
示例：
telnet> add KEY <flags> <expiretime> <bytes> \r
telnet> VALUE


-p <num> 监听的 TCP 端口 ( ( 缺省 : 11211)
-d 以守护进程方式运行 Memcached
-u <username> 运行 Memcached 的账户，非 root 用户
-m <num> 最大的内存使用,  单位是 MB ， 缺省是  64 MB
-c <num> 软连接数量,  缺省是  1024
-v 输出警告和错误信息
-vv 打印客户端的请求和返回信息
-h 打印帮助信息
-i 打印 memcached 和 libevent
-M：内存耗尽时，不执行LRU清理缓存，而是拒绝存入新的缓存项，直到有多余的空间可用时为止；
-f <factor>：增长因子；默认是1.25；
-t <threads>：启动的用于响应用户请求的线程数
-U <num>：Listen on UDP port <num>, the default is port 11211, 0 is off.


memcached默认没有认证机制，可借用于SASL进行认证；
SASL：Simple Authentication Secure Layer



启动
systemctl start memcached

连接
telnet localhost 11211
```

### Memcached 一些特性和限制

- 在  Memcached  中可以保存的 item 数据量是没有限制的，只有内存足够
- Memcached 单进程最大使用内存为 2G ，要使用更多内存，可以分多个端口开启多个 Memcached 进程
- 最大 30 天的数据过期时间,  设置为永久的也会在这个时间过期，常量 REALTIME_MAXDELTA 60* * 60* * 24* * 30  控制


- 最大键长为 250 字节，大于该长度无法存储，常量 KEY_MAX_LENGTH 250  控制
- 单个 item 最大数据是 1MB ，超过 1MB 数据不予存储，常量 POWER_BLOCK 1048576  进行控制，它是默认的 slab 大小


- 最大同时连接数是 200 ，通过  conn_init() 中的 freetotal  进行控制，最大软连接数是 1024 ，通过settings.maxconns=1024  进行控制
- 跟空间占用相关的参数： settings.factor=1.25, settings.chunk_size=48,  影响 slab
- 调优slab 比如`memcached -f 1.5 -vv` 

