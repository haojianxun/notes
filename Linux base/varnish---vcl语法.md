# varnish---vcl语法

# VCL

## varnish配置语言

### 介绍

varnish是一个专门用于描述varnish请求处理和文件缓存策略规则的语言。

当一个新的配置加载后，varnishd管理进程将会将其转换为C代码并编译，然后加载到服务器进程中。

本文档侧重于VCL的语法。对于语法和语义的完整介绍，和示例可以参阅[https://www.varnish-cache.org/docs/](https://www.varnish-cache.org/docs/) 。

从varnish 4.0版本开始，每一个VCL文件都必须在文件开始处说明版本号`VCL 4.0`。

#### 运算符

在VCL中可以使用以下运算符：

- `=` 赋值运算符
- `==` 比较运算符
- `~` 匹配运算符。既可以使用正则表达式，也可以使用ACLs。
- `!` 否定运算符。
- `&&` 逻辑与
- `||` 逻辑或

#### 条件运算

VCL有if和else语句。嵌套的逻辑可以使用elseif来实现（elsif/elif/else if这几个是一样的）。

请注意，VCL中没有任何的循环和迭代器。

### 字符串、布尔值、时间、时间单位和整数

在varnish中有各种数据类型，你可以设置或者取消这些：

```
set req.http.User-Agent = "unknown";
unset req.http.Range;

```

#### 字符串

基本的字符串都是以双引号包括，不得包含换行符。长字符串包括在`{"..."}`中。他们可能会包含单双引号、换行符以及除了NUL(0x00)以外的其他控制字符。

#### 布尔型

布尔型可以是true或者false。

#### 时间

VCL中有时间类型，函数`now`返回一个时间。持续的时间可以被添加到一个时间中，来成为另外一个时间。在字符串中它会返回一个格式化的字符串。

#### 持续时间

持续时间可以使用一个数字加一个单位来限定。数量可以是实数，因此1.5w也是被允许的。

- `ms` 毫秒
- `s` 秒
- `m` 分钟
- `h` 小时
- `d` 天
- `w` 周/星期
- `y` 年

#### 整数

某些字段可能是整数，在字符串语境下，它会返回一个字符串。

#### 实数

VCL中可以使用实数，和整数一样，在字符串的语境下，它会返回字符串。

#### 正则表达式

varnish支持Perl兼容的正则表达式(PCRE)。有关完整的说明可以参见PCRE(3)手册。

要发送一个标识给PCRE引擎，例如做不区分大小写的匹配，括号内添加一个问号。例如：

```
# If host is NOT example dot com..
if (req.http.host !~ "(?i)example.com$") {
    ...
}

```

#### 包含语句

在另外一个文件中包含一个VCL文件可以使用include关键字：

```
include "foo.vcl";

```

#### 导入语句

导入语句主要用来加载varnish模块(VMODs)。

```
import std;
sub vcl_recv {
    std.log("foo");
}

```

#### 注释

单行注释可以使用`//`或者`#`。如果是多行注释的话可以使用`/* block */`。例如：

```
sub vcl_recv {
    // Single line of out-commented VCL.
    # Another way of commenting out a single line.
    /*
        Multi-line block of commented-out VCL.
    */
}

```

#### 后端定义

一个后端声明创建和初始化唯一后端对象。声明使用关键词backend后面跟一个后端名字。实际的声明是在大括号里：

```
backend name {
    .attribute = "value";
}

```

唯一的一个强制必须有的属性是host。属性会从全局参数继承默认值。以下属性都是可用的：

- `host`（必须）

要使用的主机。ip地址或者可以解析为一个ip地址的hostname都是可以的。

- `port`

varnish应该连接的服务器端口

- `host_header`

添加的主机头

- `connect_timeout`

连接超时时间

- `first_byte_timeout`

varnish等待从后端接受第一个字节的超时时间，参考[http://jolestar.com/varnish-timeout/](http://jolestar.com/varnish-timeout/)

- `between_bytes_timeout`

varnish等待后端响应的 idel timeout

- `probe`

连接探针到后端

- `max_connections`

后端服务器最大连接数。如果varnish达到最大值，剩下的就会连接失败。

后端可以使用 *directors* 。请查看 [vmod_directors](https://www.varnish-cache.org/docs/4.1/reference/vmod_directors.generated.html#vmod-directors-3)帮助页面查看更多信息。

#### 探针 Probes

探针会定时查询后端服务器的状态，如果访问失败则标记它并关闭它。探针的定义是这样的：

```
probe name {
    .attribute = "value";
}

```

这里没有必需的选项。以下选项都是可以设置的：

- `url`

要查询的url，默认是“/”

- `request`

指定使用多个字符串来做完整的HTTP请求。.request会在每个字符串后面添加\r\n。如果指定了.request，则优先级高于.url。

- `expected_response`

预期的http响应码，默认是200.

- `timeout`

探针的超时时间，默认是2s

- `interval`

探针执行的频率，默认为5s

- `initial`

当varnish启动时多少在.window中多少polls是最好的。默认的阈值为-1。在这种情况下，后台即使出问题，只要有一个探针返回正常就被认为是健康的。

- `window`

我们有多少最新的数据信息，以确定后端是否健康。默认为8

- `threshold`

多少次返回成功，我们认为是健康的。默认为3.

#### 权限控制列表 ACL

一个权限控制列表声明创建初始化后，一般可以用于匹配客户端地址：

```
acl localnetwork {
    "localhost";    # myself
    "192.0.2.0"/24; # and everyone on the local network
    ! "192.0.2.23"; # except for the dial-in router
}

```

如果一个ACL条目指定的是一个主机名，但是varnish又无法解析的话，它将会匹配和他比较的任何地址。因此，如果它有一个否定标记，它将会拒绝它相对于任何地址，这可能不是预期的。如果该条目是在括号内的，那么它将会被简单的忽略。

为了匹配ACL的ip地址，只需要使用匹配运算符：

```
if (client.ip ~ localnetwork) {
    return (pipe);
}

```

#### VCL对象

一个VCL对象可以使用new关键字来实例化。

```
sub vcl_init {
    new b = directors.round_robin()
    b.add_backend(node1);
}

```

这只有在vcl_init中是被允许的。

### 子程序

子程序可以被用来提升代码的可读性和可重用性。

```
sub pipe_if_local {
    if (client.ip ~ localnetwork) {
        return (pipe);
    }
}

```

在VCL中子程序不带任何的参数，也没有返回值。内置的子程序都是以vcl_开头。

要调用一个子程序，可以使用call关键字，后面跟上子程序的名字。

```
sub vcl_recv {
    call pipe_if_local;
}

```

#### return语句

当使用return(action)语句的话，那么下一步vcl_*子程序将会被执行。该动作指定下一步如何执行。

#### 多个子程序

如果多个子程序的名称有一个是内置定义的，那么他们就可以有顺序的被连接在一起。

当VCL被编译时，varnish内置的VCL将会被隐式的连接起来。

### 变量

在VCL中，你有权限访问某个变量。其中包含请求和响应的处理。哪些变量可用取决于上下文。

#### bereq

- `bereq`

类型：HTTP

可读性： backend

整个后端HTTP请求的数据结构。

- `bereq.backend`

类型： BACKEND

可读： vcl_pipe, backend

可写： vcl_pipe, backend

这是我们获取数据的后端或者director

- `bereq.between_bytes_timeout`

类型： DURATION

可读： backend

可写： backend

这是从后端接收每个字节之间的超时时间。在pipe模式下是不可用的。

- `bereq.connect_timeout`

类型： DURATION

可读： vcl_pipe, backend

可写： vcl_pipe, backend

连接后端等待的超时时间。

- `bereq.first_byte_timeout`

类型： DURATION

可读： backend

可写： backend

varnish等待从后端接受第一个字节的超时时间。在pipe模式下是不可用的。

- `bereq.http.`

类型： HEADER

可读： vcl_pipe, backend

可写： vcl_pipe, backend

相应的HTTP头

- `bereq.method`

类型： STRING

可读： vcl_pipe, backend

可写： vcl_pipe, backend

请求的类型（例如：GET和HEAD）

- `bereq.proto`

类型： STRING

可读： vcl_pipe, backend

可写： vcl_pipe, backend

用于和服务器交互的HTTP版本号

- `bereq.retries`

类型： INT

可读： backend

请求重试的次数

- `bereq.uncacheable`

类型： BOOL

可读： backend

指示该请求是否是不可缓存的。

- `bereq.url`

类型： STRING

可读： vcl_pipe, backend

可写： vcl_pipe, backend

请求的URL

- `bereq.xid`

类型： STRING

可读： backend

请求的唯一ID

#### beresp

- `beresp`

类型： HTTP

可读： vcl_backend_response, vcl_backend_error

整个HTTP后端响应数据。

- `beresp.age`

类型： DURATION

可读： vcl_backend_response, vcl_backend_error

对象的age。

- `beresp.backend`

类型： BACKEND

可读： vcl_backend_response, vcl_backend_error

这是请求的后端服务器。如果bereq.backend被设置成一个director，那么它将会是director选择的服务器。

- `beresp.backend.ip`

类型： IP

可读： vcl_backend_response, vcl_backend_error

这个响应的来源服务器IP

- `beresp.backend.name`

类型： STRING

可读： vcl_backend_response, vcl_backend_error

后端响应的来源后端服务器名字。

- `beresp.do_esi`

类型： BOOL

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

布尔型。默认是false。设置为true将会解析ESI指令对象。如果req.esi是真的才会解析。

- `beresp.do_gunzip`

类型： BOOL

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

布尔型。在将对象存入缓存之前对其解压缩。默认为false。

- `beresp.do_gzip`

类型： BOOL

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

布尔值。在存储之前压缩。默认为false。当http_gzip_support是开启的时候，varnish将会从后端请求道早已经压缩好的内容。

- `beresp.do_stream`

类型： BOOL

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

直接将对象返回给客户端，而不需要将整个对象存入varnish。如果request是pass将不会被存到内存中。

- `beresp.grace`

类型： DURATION

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

Set to a period to enable grace.

- `beresp.http.`

类型： HEADER

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

相应的HTTP头

- `beresp.keep`

类型： DURATION

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

设置一个时间允许有条件的后端请求。

所保持的时间会是这个时间加上ttl。

对象的ttl过期，但是还剩有keep时间，那么将会发出有条件的后端请求来更新他们。

- `beresp.proto`

类型： STRING

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

后端响应的HTTP版本号

- `beresp.reason`

类型： STRING

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

后端服务器返回的http状态信息。

- `beresp.status`

类型： INT

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

服务器返回的http状态码

- `beresp.storage_hint`

类型： STRING

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

提示varnish你要将对象存储到特定的存储后端。

- `beresp.ttl`

类型： DURATION

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

对象剩余的存活时间，单位秒

- `beresp.uncacheable`

类型： BOOL

可读： vcl_backend_response, vcl_backend_error

可写： vcl_backend_response, vcl_backend_error

从bereq.uncacheable继承。设置此变量可以使对象不可缓存，这个可以在缓存中得到一个一次性对象。清除变量不会有效果，并且会记录日志提醒“Ignoring attempt to reset beresp.uncacheable”。

- `beresp.was_304`

类型： BOOL

可读： vcl_backend_response, vcl_backend_error

布尔值。如果这对于后端服务器请求是一个成功的304响应，将会更新现有的缓存对象。

#### client

- `client.identity`

类型： STRING

可读： client

可写： client

客户端身份，用来在client director中加载均衡。

- `client.ip`

类型： IP

可读： client

客户端的IP

#### local

- `local.ip`

类型： IP

可读： client

TCP连接的本地IP地址。

#### now

- `now`

类型： TIME

可读： all

当前的时间，当用在字符串上下文时返回一个格式化的字符串。

#### obj

- `obj.age`

类型： DURATION

可读： vcl_hit

对象已经生存的时间。

- `obj.grace`

类型： DURATION

可读： vcl_hit

该对象剩余的宽限时间，以秒为单位。

- `obj.hits`

类型： INT

可读： vcl_hit, vcl_deliver

缓存命中此对象的计数。如果为0表示未命中。

- `obj.http.`

类型： HEADER

可读： vcl_hit

相应的HTTP头。

- `obj.keep`

类型： DURATION

可读： vcl_hit

该对象剩余保留时间，以秒为单位。

- `obj.proto`

类型： STRING

可读： vcl_hit

当对象被检索时使用的HTTP协议版本。

- `obj.reason`

类型： STRING

可读： vcl_hit

由服务器返回的HTTP状态信息。

- `obj.status`

类型： INT

可读： vcl_hit

服务器返回的HTTP状态。

- `obj.ttl`

类型： DURATION

可读： vcl_hit

该对象的剩余生存时间，以秒为单位

- `obj.uncacheable`

类型： BOOL

可读： vcl_deliver

对象是否是不可缓存的(pass 还是 hit-for-pass)

#### remote

- `remote.ip`

类型： IP

可读： client

TCP连接另一端的IP地址。这个可以是客户端的IP或者代理服务器IP。

#### req

- `req`

类型： HTTP

可读： client

整个HTTP请求的数据结构。

- `req.backend_hint`

类型： BACKEND

可读： client

可写： client

如果我们尝试后去内容，会将bereq.backend设置为它。

- `req.can_gzip`

类型： BOOL

可读： client

客户端是否支持gzip传输编码。

- `req.esi`

类型： BOOL

可读： client

可写： client

布尔值。设置为false可以禁止处理ESI。默认为true。这个变量会在将来的版本中修改，所以尽量不要用它。

- `req.esi_level`

类型： INT

可读： client

目前ESI请求有多少level。

- `req.hash_always_miss`

类型： BOOL

可读： vcl_recv

可写： vcl_recv

对这个请求强制不命中缓存。如果设置为true，varnish将会忽略所有当前存在的对象，并总是从后端拉取内容。

- `req.hash_ignore_busy`

类型： BOOL

可读： vcl_recv

可写： vcl_recv

忽略缓存查找中任何繁忙的对象。你可以想象一下，如果你有两台服务器在查找内容，那么这个地方就可能会出现死锁。

- `req.http.`

类型： HEADER

可读： client

可写： client

相应的HTTP头。

- `req.method`

类型： STRING

可读： client

可写： client

请求的类型（GET或者POST）

- `req.proto`

类型： STRING

可读： client

可写： client

客户端使用的HTTP协议的版本号。

- `req.restarts`

类型： INT

可读： client

这个请求重启的次数。

- `req.ttl`

类型： DURATION

可读： client

可写： client

- `req.url`

类型： STRING

可读： client

可写： client

请求的URL

- `req.xid`

类型： STRING

可读： client

请求的唯一ID。

#### req_top

- `req_top.http.`

类型： HEADER

可读： client

在ESI请求树中顶层请求的HTTP头。在非ESI请求的情况下和req.http.相同。

- `req_top.method`

类型： STRING

可读： client

在ESI请求树中顶层请求的请求方式(例如：GET或者POST。。。)。在非ESI请求的情况下和req.method是一样的。

- `req_top.proto`

类型： STRING

可读： client

在ESI请求树中顶层请求的HTTP版本。在非ESI请求下和req.proto相同。

- `req_top.url`

类型： STRING

可读： client

在ESI请求树中顶层请求的URL。如果是非ESI请求的话和req.url相同。

#### resp

- `resp`

类型： HTTP

可读： vcl_deliver, vcl_synth

整个响应HTTP数据结构。

- `resp.http.`

类型: HEADER

可读: vcl_deliver, vcl_synth

可写: vcl_deliver, vcl_synth

相应的HTTP头数据

- `resp.is_streaming`

类型： BOOL

可读： vcl_deliver, vcl_synth

当响应从后端进行流处理的时候返回true

- `resp.proto`

类型： STRING

可读： vcl_deliver, vcl_synth

可写： vcl_deliver, vcl_synth

响应的HTTP协议版本。

- `resp.reason`

类型： STRING

可读： vcl_deliver, vcl_synth

可写： vcl_deliver, vcl_synth

将要返回的HTTP状态信息。

- `resp.status`

类型： TYPE

可读： vcl_deliver, vcl_synth

可写： vcl_deliver, vcl_synth

将要返回的HTTP状态码。

指定一个HTTP规范代码resp.status将也会设置resp.reason设置为相应的HTTP状态信息。

#### server

- `server.hostname`

类型： STRING

可读： all

服务器的名字。

- `server.identity`

类型： STRING

可读： all

由-i参数设置服务器的身份。如果-i参数不传递到varnishd，server.identity将会被设置为该实例的名称，可以由-n参数指定。

- `server.ip`

类型： IP

可读： client

接收到连接的socket的IP地址。

#### storage

- `storage..free_space`

类型： BYTES

可读： client, backend

存储的空闲空间，仅适用于malloc。

- `storage..used_space`

类型： BYTES

可读： client， backend

存储中已经使用的空间大小，仅使用于malloc。

- `storage..happy`

类型： BOOL

可读： client, backend

以name命名的存储的健康状态。

### 函数 Functions

以下内置的函数都是可用的。

- `ban(expression)`

让所有匹配到ban规则的所有对象。

- `hash_data(input)`

为hash输入增加一个输入。在内置的vcl hash_data()是使用host和url。在vcl_hash中是可用的。

- `rollback()`

恢复req HTTP头到他们的原始状态。这个函数已经被废弃，已经被std.rollback()替代。

- `synthetic(STRING)`

准备一个包含STRING的合成头。可以在vcl_synth和vcl_backend_error中是可用的。

- `regsub(str, regex, sub)`

将str中的内容,能够被regex匹配到的替换成sub,只替换第一个。在sub中，\0（也可以被写为\&）会被替换为整个匹配的字符串。\n会被第二组匹配的内容替代。

- `regsuball(str, regex, sub)`

这个会匹配到字符串中所有的匹配内容。regsuball()可以将str中能够被regex匹配到的字符串统统替换为sub。

### EXAMPLES

示例可以查看在线的文档

### 也可以看看

- [varnishd](https://www.varnish-cache.org/docs/4.1/reference/varnishd.html#varnishd-1)
- [vmod_directors](https://www.varnish-cache.org/docs/4.1/reference/vmod_directors.generated.html#vmod-directors-3)
- [vmod_std](https://www.varnish-cache.org/docs/4.1/reference/vmod_std.generated.html#vmod-std-3)

### 历史

vcl是被Poul-Henning Kamp和Verdens Gang AS，Redpill Linpro 和 Varnish Software合作开发的。本手册页是由Per Buer, Poul-Henning Kamp, Martin Blix Grydeland, Kristian Lyngstøl, Lasse Karstensen 和其他人写的。

### 版权信息

这个文档的开源协议和varnish是一样的，请参考许可证：

- Copyright (c) 2006 Verdens Gang AS
- Copyright (c) 2006-2015 Varnish Software AS

