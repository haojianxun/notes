# varnish

[TOC]

## 程序架构：

Manager进程
Cacher进程，包含多种类型的线程：

- accept, worker, expiry, ... 

shared memory log：

- 统计数据：计数器；
- 日志区域：日志记录；

varnishlog, varnishncsa, varnishstat... 

配置接口：VCL

- Varnish Configuration Language, 
- vcl complier --> c complier --> shared object


## varnish程序环境

/etc/varnish/varnish.params (配置varnish服务进程的工作特性，例如监听的地址和端口，缓存机制)

/etc/varnish/default.vcl：(配置各Child/Cache线程的缓存工作属性)；

### 主程序

/usr/sbin/varnishd



### CLI interface:

​	/usr/bin/varnishadm

### Shared Memory Log交互工具

​	/usr/bin/varnishhist
​	/usr/bin/varnishlog
​	/usr/bin/varnishncsa
​	/usr/bin/varnishstat
​	/usr/bin/varnishtop

### 测试工具程序

​	usr/bin/varnishtest

### VCL配置文件重载程序

/usr/sbin/varnish_reload_vcl

### Systemd Unit File

/usr/lib/systemd/system/varnish.service

/usr/lib/systemd/system/varnishlog.service

/usr/lib/systemd/system/varnishncsa.service

## varnish的缓存存储机制

-s [name=]type[,options]

- malloc[,size]

  内存存储，[,size]用于定义空间大小；重启后所有缓存项失效

- file[,path[,size[,granularity]]]

  文件存储，黑盒；重启后所有缓存项失效

- persistent,path,size

  文件存储，黑盒；重启后所有缓存项有效；尚属实验阶段

## varnish程序的选项

程序选项：/etc/varnish/varnish.params文件

> -a address[:port][,address[:port][...]，默认为6081端口； 
> -T address[:port]，默认为6082端口；
> -s [name=]type[,options]，定义缓存存储机制；
> -u user
> -g group
> -f config：VCL配置文件；
> -F：运行于前台

运行时参数：/etc/varnish/varnish.params文件

```
DAEMON_OPTS="-p thread_pool_min=5 -p thread_pool_max=500 -p thread_pool_timeout=300"  #这个可以在配置文件中开启

-p param=value：设定运行参数及其值； 可重复使用多次
-r param[,param...]: 设定指定的参数为只读状态
```

## varnishadm

````
varnishadm -S /etc/varnish/secret -T [ADDRESS:]PORT

可以输入help来获取帮助
help [<command>]
ping [<timestamp>]
auth <response>
quit
banner
status
start
stop
vcl.load <configname> <filename>
vcl.inline <configname> <quoted_VCLstring>
vcl.use <configname>
vcl.discard <configname>
vcl.list
param.show [-l] [<param>]
param.set <param> <value>
panic.show
panic.clear
storage.list
vcl.show [-v] <configname>
backend.list [<backend_expression>]
backend.set_health <backend_expression> <state>
ban <field> <operator> <arg> [&& <field> <oper> <arg>]...
ban.list
````

配置文件相关：

	vcl.list 
	vcl.load：装载，加载并编译；
	vcl.use：激活；
	vcl.discard：删除；
	vcl.show [-v] <configname>：查看指定的配置文件的详细信息；

### 运行时参数

	param.show -l：显示列表；
	param.show <PARAM>
	param.set <PARAM> <VALUE>

### 缓存存储

	storage.list
### 后端服务器

	backend.list



## vcl

Varnish使用区域配置语言，这种语言叫做“VCL”（varnish configuration language），在执行vcl时，varnish就把VCL转换成二进制代码

VCL从C语言继承了很多东西，它读起来非常像C或者Perl。

代码块是使用大括号分割，语句结束使用分号

VCL有多个状态引擎，状态之间存在相关性，但状态引擎彼此间互相隔离；每个状态引擎可使用return(x)指明关联至哪个下一级引擎；每个状态引擎对应于vcl文件中的一个配置段，即为subroutine

vcl_hash --> return(hit) --> vcl_hit

###权限控制列表 Access control lists (ACLs)

ACL声明创建和初始化一个权限控制列表，通常用来匹配客户端地址：

```
acl local {
  "localhost";         // myself
  "192.0.2.0"/24;      // and everyone on the local network
  ! "192.0.2.23";      // except for the dialin router
}

```

如果您的ACL指定了一个varnish无法解析的地址，那么它将与它比较的任何地址来进行匹配。因此，如果它前面有一个否定标记，那么他会拒绝任何和它相关的地址，这些可能是你不能预料的。如果该条目被括号括起来的话，这个将会被简单的忽略。

匹配ip地址的时候，会简单的使用匹配运算符：

```
if (client.ip ~ local) {
  return (pipe);
}

```

###运算符 Operators

下面是一些可以在VCL中使用的运算符：

- `=` 赋值运算符
- `==` 比较运算符
- `~` 匹配。可以使用正则表达式或者ACLs。
- `!` 否定运算符
- `&&` 逻辑与
- `||` 逻辑或

###子程序 Subroutines

子程序可以增加代码的可读性和可重用性：

```
sub pipe_if_local {
  if (client.ip ~ local) {
    return (pipe);
  }
}
```

## Client Side

	vcl_recv, vcl_pass, vcl_hit, vcl_miss, vcl_pipe, vcl_purge, vcl_synth, vcl_deliver
### `vcl_recv` 

`vcl_recv`子程序可以通过调用`return`来结束，参数可以是下列关键词：

- `hash` 继续处理对象以备缓存。将请求转到vcl_hash
- `pass` 进入pass模式,处理转到`vcl_pass`。
- `pipe` 进入pipe模式，处理转到`vcl_pipe`
- `synth(status code, reason)` 带着synth的参数resp.status和resp.reason转到`vcl_synth`处理。
- `purge` 清楚对象。控制权将会通过`vcl_hash`转到`vcl_purge`。

### `vcl_pipe`

进入pipe模式时调用，在这种模式下，请求将会被传递到后端，后续的数据不管是从客户端还是后端来的都会以不变的方式传送，直到连接关闭为止。也就是说，varnish只会做简单的TCP代理，来回的传输数据。在这种模式下的连接，在vcl_pipe之后不会有任何的vcl子程序会被调用。

vcl_pipe子程序可以通过调用`return()`来结束，通过以下关键字：

- `pipe` 使用pipe模式进行处理
- `synth(status code, reason)` 带着synth的参数resp.status和resp.reason转到`vcl_synth`处理。

### `vcl_pass`

进入pass模式时调用。在这种模式下，请求会直接传递到后端主机，但是后端主机的响应会直接返回给客户端，而不会进行缓存。同一客户端的大量请求会按照规则处理。

vcl_pass子程序可以通过调用`return()`来结束，通过以下关键字：

- `fetch` 继续使用pass模式，发起后端请求
- `restart` 重新启动事务。增加了重新启动计数器。如果重启的次数超过了`max_restarts`的设置，就会抛出一个错误。
- `synth(status code, reason)` 带着synth的参数resp.status和resp.reason转到`vcl_synth`处理。

### `vcl_hit`

当在缓存中成功查到的时候调用。读到的数据可能是已经过期的：如果有宽限时间或者保留时间的话，它可以是一个0或者负的ttl。

vcl_hit子程序可以通过调用`return()`来结束，通过以下关键字：

- `deliver` 传递对象。如果它已经过期了，那么将会触发重新从后端获取数据动作。
- `miss` 同步刷新缓存的对象，尽管缓存已经命中。并将会转给`vcl_miss`进行处理。
- `pass` 切换到pass模式。控制将会转到`vcl_pass`进行处理。
- `restart` 重新启动事务。增加了重新启动计数器。如果重启的次数超过了`max_restarts`的设置，就会抛出一个错误。
- `synth(status code, reason)` 带着synth的参数resp.status和resp.reason转到`vcl_synth`处理。
- `fetch`（已废弃） 和`miss`是一样的。将会在将来的版本中去除。

### `vcl_miss`

当在缓存找不到请求内容或者使用`vcl_hit`返回fetch时调用该方法。

该函数主要用于判断是否要从后端服务器来获取数据，从哪一个后端获取内容。

vcl_miss子程序可以通过调用`return()`来结束，通过以下关键字：

- `fetch` 从后端获取请求对象，并转给`vcl_backend_fetch`进行处理。
- `pass` 切换到pass模式，并转给`vcl_pass`进行处理。
- `restart` 重新启动事务。增加了重新启动计数器。如果重启的次数超过了`max_restarts`的设置，就会抛出一个错误。
- `synth(status code, reason)` 带着synth的参数resp.status和resp.reason转到`vcl_synth`处理。

### `vcl_hash`

在`vcl_recv`为请求创建哈希值之后被调用，这将会使用一个key去在varnish中查找这个对象。

vcl_hash子程序可以通过调用`return(lookup)`来结束：

- `lookup` 从缓存中查找对象。当是来了一个purge的话会将控制传递到`vcl_purge`然后返回给`vcl_recv`。否则控制权传递至`vcl_miss`, `vcl_hit` 或者 `vcl_pass`。

### `vcl_purge`

pruge操作执行后调用此函数，所有他的变种将被回避。

vcl_purge子程序可以通过调用`return()`来结束，通过以下关键字：

- `restart` 重新启动事务。增加了重新启动计数器。如果重启的次数超过了`max_restarts`的设置，就会抛出一个错误。
- `synth(status code, reason)` 带着synth的参数resp.status和resp.reason转到`vcl_synth`处理。

### `vcl_deliver`

将在缓存中找到请求的内容发送给客户端前调用此方法，当然`vcl_synth`除外。

vcl_deliver子程序可以通过调用`return()`来结束，通过以下关键字：

- `deliver` 将对象传递给客户端。
- `restart` 重新启动事务。增加了重新启动计数器。如果重启的次数超过了`max_restarts`的设置，就会抛出一个错误。
- `synth(status code, reason)` 带着synth的参数resp.status和resp.reason转到`vcl_synth`处理。

### `vcl_synth`

主要被用于返回组合对象。这个组合对象会在VCL里面进行合成，而不是直接从后端拉取。它的内容可以使用`synthetic()`函数来进行构造。

vcl_synth定义的对象不会存入缓存，相反`vcl_backend_error`定义的对象倒是可以被存入缓存。

这个子程序可以通过调用`return()`来结束，通过以下关键字：

- `deliver` 直接把`vcl_synth`定义的对象返回给客户端，而不调用`vcl_deliver`。
- `synth(status code, reason)` 带着synth的参数resp.status和resp.reason转到`vcl_synth`处理。

## 后端

### `vcl_backend_fetch`

在向后端主机发送请求前，调用此函数，可修改发往后端的请求。

vcl_backend_fetch子程序可以通过调用`return()`来结束，通过以下关键字：

- `fetch` 从后端拉取数据
- `abandon` 放弃后端请求，除非是后端拉取命令。请求会带着预设为503的`resp.status`参数，并转到`vcl_synth`进行处理。

### `vcl_backend_response`

当从后端服务器成功取到响应头之后调用。

对于304响应，varnish核心代码会在调用`vcl_backend_response`之前修改`beresp`：

- 如果gzip的状态发生改变，`Content-Encoding`没有被设置。
- 在304响应中不存在从现有对象中复制的响应头。如果在缓存中存在，那么`Content-Length`会被复制，否则就丢弃。
- 状态设置为200.

`beresp.was_304`标识标识这个条件的响应处理已经发生。

注意： 后端请求时独立的客户端条件请求，所以客户端可能受到304响应，不管后端是否是有条件的。

vcl_backend_response子程序可以通过调用`return()`来结束，通过以下关键字：

- `deliver` 对于304响应，会创建一个最新的缓存对象。否则会从后端拉取对象内容，并将结果返回给所有在等待的客户端请求，可能是并行（流）。
- `abandon` 放弃后端请求。除非后端请求是背景取指令，在客户端会带着被设置为503的`resp.status`参数传递给`vcl_synth`处理。
- `retry` 重试后端事物。增加重试计数器，如果重试次数大于 *max_retries* 设置的值，将会转给vcl_backend_error进行处理。

### `vcl_backend_error`

如果为我们从后端服务拉取数据失败或者重试次数超出的时候被调用。

在VCL中合成的组合对象，其内容可能使用synthetic()函数进行合成。

vcl_backend_error子程序可以通过调用`return()`来结束，通过以下关键字：

- `deliver` 传递错误页面
- `retry` 重试后端事物。增加重试计数器，如果重试次数大于 *max_retries* 设置的值，在客户端上面的`vcl_synth`将会返回resp.status=503。

## vcl.load / vcl.discard

### `vcl_init`

在任何请求通过它之前，当VCL被加载时被调用。通常用来初始化VMODs。

vcl_init子程序可以通过调用`return()`来结束，通过以下关键字：

- `ok` 正常返回，继续加载VCL
- `fail` 中止加载VCL

### `vcl_fini`

只有在所有请求都从VCL退出之后，当VCL被丢弃之后调用。经常被用来清理VMODs。

vcl_fini子程序可以通过调用`return()`来结束，通过以下关键字：

- `ok` 正常返回，丢弃VCL

## vcl的语法格式

- VCL files start with vcl 4.0;
- //, # and /* foo */ for comments;
- Subroutines are declared with the sub keyword; 例如sub vcl_recv { ...}；
- No loops, state-limited variables（受限于引擎的内建变量）
- Terminating statements with a keyword for next action as argument of the return() function, i.e.: return(action)；用于实现状态引擎转换；
- Domain-specific

### 三类主要语法

```
sub subroutine {
	...
}
			
if CONDITION {
	...
} else {	
	...
}
			
return(), hash_data()
```

### VCL Built-in Functions and Keywords

	函数：
		regsub(str, regex, sub)
		regsuball(str, regex, sub)
		ban(boolean expression)
		hash_data(input)
		synthetic(str)
				
	Keywords:
		call subroutine， return(action)，new，set，unset 
举例：obj.hits

	if (obj.hits>0) {
			set resp.http.X-Cache = "HIT via " + server.ip;
	} else {
			set resp.http.X-Cache = "MISS via " + server.ip;
	}
### 变量类型

## `req`

请求对象。当varnish接收到请求的时候，请求对象被创建并填充。在`vcl_recv`你做的大部分工作都是在req对象上。

req.http.*

req.http.User-Agent, req.http.Referer, ...

```
req.http.Cookie：客户端的请求报文中Cookie首部的值
req.http.User-Agent ~ "chrome"
```

## `bereq`

后端请求对象。varnish在发送到后端服务器之前构造bereq，它是基于req对象创建的。

bereq.http.*

```
bereq.http.HEADERS
bereq.request：请求方法；
bereq.url：请求的url；
bereq.proto：请求的协议版本；
bereq.backend：指明要调用的后端主机；
```

## `beresp`

由BE主机响应给varnish的响应报文相关
beresp.http.*

```
beresp.http.HEADERS
beresp.status：响应的状态码；
beresp.backend.name：BE主机的主机名；
beresp.ttl：BE主机响应的内容的余下的可缓存时长；
```

## `resp`

由varnish响应给client相关

```
reresp.proto：协议版本；
```

## `obj`

这是指存在缓存中的对象，只读

	obj.*
	obj.hits：此对象从缓存中命中的次数；
	obj.ttl：对象的ttl值
其中常用的变量

```
server.*
	server.ip
	server.hostname
client.*
	client.ip
```

用户自定义：

	set 
	unset
示例1：强制对某类资源的请求不检查缓存：

	vcl_recv {
		if (req.url ~ "(?i)^/(login|admin)") {  #其中(?i)的意思是忽略大小写 ~表示的是后面是正则表达式
			return(pass);
		}
	}
示例2：对于特定类型的资源，例如公开的图片等，取消其私有标识，并强行设定其可以由varnish缓存的时长； 

	if (beresp.http.cache-control !~ "s-maxage") {
		if (bereq.url ~ "(?i)\.(jpg|jpeg|png|gif|css|js)$") {
			unset beresp.http.Set-Cookie;
			set beresp.ttl = 3600s;
		}
	}
缓存对象的修剪：purge, ban 

	(1) 能执行purge操作
	sub vcl_purge {
		return (synth(200,"Purged"));
	}
				
	(2) 何时执行purge操作
	sub vcl_recv {
		if (req.method == "PURGE") {
			return(purge);
		}
		...
	}
添加此类请求的访问控制法则：

	acl purgers {
		"127.0.0.0"/8;
		"10.1.0.0"/16;
	}
				
	sub vcl_recv {
		if (req.method == "PURGE") {
			if (!client.ip ~ purgers) {
				return(synth(405,"Purging not allowed for " + client.ip));
			}
			return(purge);
		}
			...
	}
定义多个后端主机

```
backend default {
	.host = "172.16.0.2";
	.port = "80";
}

backend paasrv {
	.host = "172.16.0.3";
	.port = "80";
}

sub vcl_recv {
	if (req.url ~"(?i)\.php$") {
		set req.backend_hint = appsrv;
	} else {
		set req.backend_hint = default;
	}
	...
}

```

Director

```
import director;

backend server1 {
	.host = 
	.port =
}

backend server2 {
	.host =
	.port =
}

sub vcl_init {
	new GROUP_NAME = directors.round_robin();
	GROUP_NAME.add_backend(server1);
	GROUP_NAME.add_backend(server2);
}

sub vcl_recv {
	set req.backend_hint = GROUP_NAME.backend();
}

```

### 后端主机健康状态检测

```
backend BE_NAME {
	.host = 
	.port = 
	.probe = {
		.url= 
		.timeout=
		.interval=
		.window=
		.threshhold=
	}
}
```

> .probe 定义健康状态检测方法
>
> ​	.url 检测的时候请求url 默认/
>
> ​	.request 发出的具体请求:
>
> ​		.request=
>
> ​			"GET /.healthtest.html HTTP/1.1"
>
> ​			"Host:www.baidu.com"
>
> ​			"Connection:close"
>
> ​	.window 基于最近多少次检测判定健康状态
>
> ​	.threshold :最近.window中定义这么次检测中至少有.threshold定义 次数的成功的
>
> ​	.interval 检测频度
>
> ​	.timeout 超时时长
>
> ​	.expected_response 期望的响应码 默认是200

健康状态检测配置方法

```
probe check {
	.url="/.healthcheck.html";
	.window = 5;
	.threshold=4;
	.interval=2s
	.timeout=1s
}

backend default {
	.host="10.1.0.1";
	.port="80";
	.probe="check";
}

backend appsrv {
	.host="10.1.0.2";
	.port="80";
	.probe="check";
}
```

## 线程模型

线程模型

- cache-worker
- cache-main
- ban lurker
- acceptor
- epoll/kqueue

线程相关参数

在线程池里,一个请求由一个线程来处理

worker的最大数决定了varnish的并发响应能力

`thread_pools` :最好小于等于cpu核心数量

`thread_poll_max` :每个线程的最大线程数

`thread_pool_min` :the minimum number of worker threads in each pool 最大空闲数

最大并发连接数 thread_pools * thread_pool_max 

`thread_pool_timeout`：Thread idle threshold.  Threads in excess of thread_pool_min, which have been idle for at least this long, will be destroyed

`thread_pool_add_delay` ：Wait at least this long after creating a thread.

`thread_pool_destroy_delay`：Wait this long after destroying a thread.

设置方法

vcl.param

param.set

永久生效方法

varnish.params

DEAMON_OPTS="-p PARAM1=VALUE -p PARAM2=VALUE"

## varnish日志

1、`varnishstat`  - Varnish Cache statistics

-1
-1 -f FILED_NAME 
-l：可用于-f选项指定的字段名称列表；
​				
MAIN.cache_hit 
MAIN.cache_miss
​				

```
varnishstat -1 -f MAIN.cache_hit -f MAIN.cache_miss
```

2、`varnishtop`  - Varnish log entry ranking
-1     Instead of a continously updated display, print the statistics once and exit.
-i taglist，可以同时使用多个-i选项，也可以一个选项跟上多个标签；
-I <[taglist:]regex>
-x taglist：排除列表
-X  <[taglist:]regex>
​				
3、`varnishlog`  - Display Varnish logs
​				
4、 `varnishncsa`  - Display Varnish logs in Apache / NCSA combined log format
​			
内建函数：
hash_data()：指明哈希计算的数据；减少差异，以提升命中率；
regsub(str,regex,sub)：把str中被regex第一次匹配到字符串替换为sub；主要用于URL Rewrite
regsuball(str,regex,sub)：把str中被regex每一次匹配到字符串均替换为sub；
return()：
ban(expression) 
ban_url(regex)：Bans所有的其URL可以被此处的regex匹配到的缓存对象；
synth(status,"STRING")：purge操作；