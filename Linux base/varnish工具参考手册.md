# varnish工具参考手册

# varnish处理状态

## 介绍

varnish处理的客户端和服务端的请求都有相应的状态。每当进入一个状态的时候，一个C函数就会被调用，从而在这个阶段调用合适的varnish函数去处理请求或者响应。对于大多数的状态，也可能会调用一些VCL子程序。

+

一般情况下，核心代码的目的都是调用相应的子程序之前准备好默认值，这些对于大部分常见的情况都是可以应付的了的。如果必要的话，这些都可以在VCL中进行修改。

下列的图表提供了各种处理状态，以及他们的代表的内容和很多相关函数的概览。他们代表了core/VMOD开发人员和管理人员之间的妥协，旨在作为一些衍生作品的参考依据。

## 客户端

![客户端](https://jefferywang.gitbooks.io/varnish_4_1_doc_zh/content/images/cache_req_fsm.svg)

## 后端

![后端服务器](https://jefferywang.gitbooks.io/varnish_4_1_doc_zh/content/images/cache_fetch.svg)

# varnishadm

## 控制一个正在运行的varnish实例

### 概要

varnishadm [-n ident] [-t timeout] [-S secretfile] [-T [address]:port] [command [...]]

### 描述

该varnishadm可以用来建立一个cli连接连接varnishd，使用-n名称或者-T和-S参数。如果提供了-n参数，密钥文件和address:port会在共享内存中被查找。如果没有提供的话，那么varnishadm将会查找一个实例而不需要指定一个名字。

如果给出一个命令，命令和参数会通过CLI连接发送出去，并且结果会返回给stdout。

如果没有命令参数提供，varnishadm会通过cli socket和stdin/stdout来传递命令和输出。

### 参数

- `-n ident` 通过这个名字连接varnishd
- `-S secretfile` 指定认证的密钥文件。这个需要和提供给varnishd的-S参数一致。只有它可以读取该文件的内容，并验证这个CLI连接。
- `-t timeout` 操作的超时时间。
- `-T ` 连接指定地址和端口的管理接口。

实际上CLI接口的语法和操作符在[varnish-cli](https://www.varnish-cache.org/docs/4.1/reference/varnish-cli.html#varnish-cli-7)帮助页面有介绍。参数可以查看[varnishd](https://www.varnish-cache.org/docs/4.1/reference/varnishd.html#varnishd-1)帮助页面。

此外， 命令的描述可以通过help命令来获取，参数的详细描述可以通过param.show获取。

### 退出

如果给出一个命令，varnishadm中如果命令执行成功会返回0，否则为非零。

### 示例

有时候，你可以这样使用varnishadm：

```
varnishadm -T localhost:999 -S /var/db/secret vcl.use foo
echo vcl.use foo | varnishadm -T localhost:999 -S /var/db/secret
echo vcl.use foo | ssh vhost varnishadm -T localhost:999 -S /var/db/secret
```

# varnishd

### 描述

varnish守护进程可以接受来自客户端的HTTP请求，把他们传递到后端服务器并缓存然后返回，以更好的满足后续同一地址的请求。

### 参数

- -a <address[:port][,PROTO]>

监听指定地址和端口的客户端请求。这个地址可以是服务器名称(例如：localhost)，一个IPV4点号分隔的地址（例如127.0.0.1）或者是方括号括起来的IPV6地址（例如[::1]）。如果地址没有指定的话，varnishd将会监听所有可用的IPV4和IPV6接口。如果未指定端口号，那么80端口将会被使用。另外监听套接字的协议类型可以通过PROTO来设置，有效的协议有HTTP/1(默认),PROXY。多个监听地址可以通过使用多个-a参数来指定。

- -b <host[:port]>

指定后端服务器的主机。如果端口没有指定，那么默认是8080.

- -c

打印被编译为C语言的VCL代码并退出。指定编译的VCL文件可以使用-f参数。

- -d

启用调试模式

- -F

不要fork进程，直接在前台运行。

- -f config

使用指定的VCL配置文件来取代内置的默认内容。可以查看VCL相关章节来了解VCL的语法。当没有指定-f也没有指定-b参数时，varnishd将无法启动工作进程。

- -h <type[,options]>

指定的哈希算法。可以查看[散列算法参数](https://www.varnish-cache.org/docs/4.1/reference/varnishd.html#hash-algorithm-options)来查看支持的算法列表。

- -i identity

指定varnish服务器的特性，这个可以使用server.identity在VCL中进行访问。

- -j <jail[,jailoptions]>

指定jailing技术的使用。

- -l <vsl[,vsm]>

指定shmlog文件的大小。vsl是VSL记录的空间[80M]，vsm是状态计数器的空间[1M]。像K和M这样的后缀，可以使用到G。默认是81M。

- -M <address:port>

连接到这个端口，并提供命令行界面。可以把它看成是一个反向的shell。当运行的时候使用-M参数的话，如果没有后端服务器定义子进程将不能够正常启动。

- -n name

指定此实例的名称。除其他事项外，这个名字一般都是用来构建varnish保存临时文件和持久状态目录名称时使用。如果在指定名称时使用正斜杠开头，那么会被解释为使用目录绝对路径。

- -P file

将PID写入指定的文件。

- -p <param=value>

通过设置参数来指定参数的值，这个选项可以多次使用，以指定多个参数。

- -r <param[,param...]>

列出只读参数列表。这个可以给系统管理员一种方法去限制varnish cli可以做什么事情。考虑让cc_command, vcc_allow_inline_c和vmod_dir这些参数只读，因为这些参数可能被用于从cli修改权限。

- -S file

包含了用于认证管理端口权限的密钥文件的地址。如果没有提供新的密钥，那么它会从系统的PRNG获取。

- -s <[name=]type[,options]>

用于指定存储的后端。这个选项可以多次使用，以指定多个存储文件。名称可以参考日志、VCL、统计等。

- -T <address[:port]>

提供一个在指定地址和端口的管理接口。

- -t TTL

指定缓存对象默认的生存时间。这是用于指定default_ttl运行时参数的快捷方式。

- -V

展示版本号并退出。

- -W waiter

指定waiter type。

#### hash算法参数

下列的hash算法是可用的：

- -h critbit

varnish 2.1和以后版本的默认算法。除非你知道你要做什么，否则不要修改这个。

- -h simple_list

一个简单的双向链表，一般不建议用于生产环境。

- -h

一个标准的hash表。

#### 后端存储参数

下列后端存储类型是被允许的：

- -s

malloc是基于内存的后端存储。

- -s

在磁盘文件上存储后端数据。这个文件将会使用mmap进行访问。path是必须的，如果路径指向一个目录的话，临时文件将会在这个目录中创建，并取消链接。如果path指向一个不存在的文件，那么文件会被创建。如果省略大小，并且path指向一个尺寸大于0已经存在的文件，那么将会使用现在文件的大小来充当此参数。如果没有，则报错。granularity设置分配块的尺寸。默认为系统页面大小或者文件系统块大小，以两者较大者为准。

- -s

持久存储。varnish将会存储对象到一个文件中。持久存储有很多问题，所以很可能在将来的版本中移除。

#### Jail参数

varnish的jail是一个泛化的在各种平台特定的方法来降低varnish进程的权限。他们有特定的选项，下面都是可用的：

- -j solaris
- -j

默认情况下，在所有其他平台上面，varnishd被root用户启动或者是varnish用户。随着unix jail技术的激活，varnish将切换到一个子进程的用户，并修改主进程可能有效的UID。可选的用户参数是替代用户，默认是varnish。

- -j none

不得已的选择：varnish将使用启动时的权限来运行所有进程。

#### 管理接口

如果指定了-T参数，varnishd将会提供在指定的地址和端口上面的命令行管理接口。连接到命令行管理接口的推荐方式是通过varnishadm。

可用的命令都在varnish中有所介绍。

### RUN TIME参数

#### RUN TIME 参数信号

Runtime参数都有缩写，以避免一遍一遍的输入重复的文本。标识的含义如下：

- experimental

我们对这个参数的good/bad/optimal没有可靠的信息。

- delayed

这个参数可以动态改变，但是不会立即生效。

- restart

工作进程必须停止并重启这个参数才会生效。

- reload

VCL程序必须重载这个参数才会生效。

- experimental

我们真的不知道这个参数。

- wizard

不要碰，除非你知道你要做什么。

- only_root

如果运行是通过root用户才工作。

#### 在32为系统上的默认值

要知道，在32位系统上面，某些默认值相对于一下列出的值可以节省空间：

- workspace_client: 16k
- thread_pool_workspace: 16k
- http_resp_size: 8k
- http_req_size: 12k
- gzip_stack_buffer: 4k
- thread_pool_stack: 64k

#### 参数列表

本文的参数你都可以在param.show命令查看到的列表是差不多的：

##### accept_filter

单位： BOOL

默认值： 开启

标识： must_restart

启用内核过滤器（如果内核可用的话）

##### acceptor_sleep_decay

默认： 0.9

最小值： 0

最大值： 1

标识： experimental

这个参数可以减少每个成功接受的睡眠时间。(0.9就是减少10%)

##### acceptor_sleep_incr

单位： 秒

默认： 0.000

最小： 0.000

最大： 1.000

标识： experimental

此参数可以控制睡眠时间长短，每次失败的时候都可以接受新的连接。

##### acceptor_sleep_max

单位： 秒

默认： 0.050

最小： 0.000

最大： 10.000

标识： experimental

此参数可以控制多久之后接受新的连接。

##### auto_restart

单位： BOOL

默认： 开启

如果进程挂掉的话就重启子进程或者工作进程。

##### backend_idle_timeout

单位： 秒

默认： 60.000

最小： 1.000

我们关闭未使用的后端连接之前的超时时间。

##### ban_dups

单位： BOOL

+

默认： 开启

## varnishadm

### 介绍

varnish具有命令行接口(cli)，这个可以控制和修改大部分的操作参数以及varnish的配置信息，而不需要打断varnish的运行。 CLI可以用于进行下列任务：

- 配置configuration

你可以在CLI中上传、修改和删除vcl文件。

- 参数 parameters

你可以通过CLI检查并修改各个参数。各个参数在varnishd帮助页面都有介绍。

- bans

bans是用于过滤陈旧的内容。当你发出一个ban之后，varnish将不会从缓存中返回任何被禁掉的对象，但是可以重新从后端服务器拉取内容。

- 流程管理

你可以通过CLI停止或者开启缓存子进程。你也可以检索最新的跟踪堆栈，如果子进程崩溃掉的话。

如果使用-T, -M或者-d参数调用varnishd也是被允许的。在调试模式下(-d)，CLI将会在前台，如果使用-T参数，你可以通过varnishadm或者telnet连接它。-M参数varnishd将会连接回监听服务。具体可以查看varnishd了解详情。

### 语法

长命令在这里可以使用sh风格语法。这里的文档格式是：

```
<< word
     here document
word

```

当使用当前的文档风格的话，单词长度是不受限制的。当使用回车命令的话，就会受到varnish的cli_buffer参数限制的最大长度。

### 命令

- help [<command>]

展示命令帮助。

- ping [<timestamp>]

保持连接。

- auth <response>

认证

- quit

关闭连接

- banner

打印欢迎内容

- status

检查varnish缓存处理状态。

- start

开启varnish缓存进程。

- stop

关闭varnish缓存进程。

- vcl.load <configname> <filename> [auto|cold|warm]

编译和加载提供的vcl文件

- vcl.inline <configname> <quoted_VCLstring> [auto|cold|warm]

编译并加载vcl数据。

- vcl.use <configname>

切换到该命名下的配置文件

- vcl.discard <configname>

卸载该命名的配置（如果可能的话）

- vcl.list

列出所有加载的配置

- vcl.show [-v] <configname>

显示指定配置的源代码。

- vcl.state <configname> <state>

强制修改指定配置文件的状态。可以是auto, warm, cold等值。

- param.show [-l] [<param>]

显示参数及其值。

- param.set <param> <value>

设置参数值

- panic.show

如果有的话，返回最后的panic。

- panic.clear [-z]

清除最后的panic，如果有的话。-z将会清除所有相关的varnishstat计数器。

- storage.list

存储设备列表

- backend.list [-p] [<backend_expression>]

后端服务器列表

- backend.set_health <backend_expression> <state>

设置某个后端服务器的状态。值可以是auto,healthy或者sick。

- ban <field> <operator> <arg> [&& <field> <oper> <arg> ...]

当条件匹配时，标记所有的过期内容。

- ban.list

活动ban的列表。输出格式为：

1. ban发布时间
2. 引用计数
3. c代表bans完成(被新的ban替代)或者-
4. 如果`lucker`调试是开启的话：R代表请求属性或者-， O代表对象属性或者-。
5. ban规范

### 后端表达式

后端表达式可以是一个后端名字或者是一个由名字、ip地址、端口组成的“name(IP:port)”形式的名字。所有的字段都是可选的。如果没有找到确切匹配的后端，将会基于提供的名字、IP、端口做部分匹配。

示例：

```
backend.list def*
backend.set_health default sick
backend.set_health def* healthy
backend.set_health * auto

```

### Ban表达式

一个ban表达式可以包含一个或者多个条件。一个条件是由一个字段、一个运算符和一个参数组成。条件可以使用“&&”来进行逻辑与的连接。

一个字段可以是任何VCL变量，例如req.url, req.http.host, obj.http.set-cookie等等。

运算符可以是“==”直接比较，“~”的正则表达式匹配，以及“>”和“<”来做大小比较。同时也可以在前面加上“!”来做否定操作。

参数值可以是一个带引号的字符串，或者一个正则表达式，或者一个整数，整数还可以有“KB”， “MB”, “GB”或者“TB”等相关领域的单位。

### VCL热度

一个VCL程序需要经过几个与不同命令相关联的状态：它可以被加载、使用和卸载。你可以加载几个VCL程序，然后在任何时候从一个切换到另外一个。同一时间只会有一个VCL程序是活动状态的，但是上一个活动状态的VCL仍然是有效的，知道它所有的处理都已经结束。

随着时间推移，如果你经常刷新你的VCL并保持之前的版本活动，消耗的资源就会增加，这是你无法逃避的。然而，大多数时候，你应该只需要一个活动状态的VCL即可，并保持旧的VCLs的情况下，你需要可以回滚到之前的版本。

VCL热度可以让你减少非活动VCLs的足迹。一旦VCL热度下降的话，varnish就会释放所有的资源。你可以手动设置VCL的热度，让varnish自动处理。

### 脚本

如果你打算写一个脚本来通过VCLI与varnishd进行交互的话，include/cli.h包含了相关的很多魔法数。

一个特别神奇的数字你需要知道，那就是状态码和长度字段行都是13个字符长，这里包含了NL字符。

可以供你参考的源文件是lib/libvarnish/cli_common.h，这里包含了用来读写CLI响应的函数。

### -S/PSK认证是如何工作的？

如果给varnishd提供了-S secret-file参数的话，所有的网络CLI连接都必须进行认证，通过这个文件内容。

在认证命令发起并且文件内容没有被varnishd缓存时会读取这个文件的内容。

使用unix文件权限控制访问该文件。

认证会话看起来是这样的：

```
critter phk> telnet localhost 1234
Trying ::1...
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
107 59
ixslvvxrgkjptxmcgnnsdxsvdmvfympg

Authentication required.

auth 455ce847f0073c7ab3b1465f74507b75d3dc064c1e7de3b71e00de9092fdc89a
200 193
-----------------------------
Varnish HTTP accelerator CLI.
-----------------------------
Type 'help' for command list.
Type 'quit' to close CLI session.
Type 'start' to launch worker process.

```

CLI的107状态表示身份验证是必须的。响应文本的前32个字符是"ixsl...mpg"， 这个是为每一个cli连接随机生成的，并且会在每一次107状态时改变。

然后必须使用计算认证符“455c...c89a”来进行验证。

验证器通过SHA256函数来对下面的字节序列进行计算：

- 质询字符串，指“ixsl...mpg”
- 换行符(0x0a)
- 密钥文件内容
- 质询字符串
- 换行符（0x0a）
- dumping the resulting digest in lower-case hex

在上面的例子中，密钥文件包含foon和thus：

```
critter phk> cat > _
ixslvvxrgkjptxmcgnnsdxsvdmvfympg
foo
ixslvvxrgkjptxmcgnnsdxsvdmvfympg
^D
critter phk> hexdump -C _
00000000  69 78 73 6c 76 76 78 72  67 6b 6a 70 74 78 6d 63  |ixslvvxrgkjptxmc|
00000010  67 6e 6e 73 64 78 73 76  64 6d 76 66 79 6d 70 67  |gnnsdxsvdmvfympg|
00000020  0a 66 6f 6f 0a 69 78 73  6c 76 76 78 72 67 6b 6a  |.foo.ixslvvxrgkj|
00000030  70 74 78 6d 63 67 6e 6e  73 64 78 73 76 64 6d 76  |ptxmcgnnsdxsvdmv|
00000040  66 79 6d 70 67 0a                                 |fympg.|
00000046
critter phk> sha256 _
SHA256 (_) = 455ce847f0073c7ab3b1465f74507b75d3dc064c1e7de3b71e00de9092fdc89a
critter phk> openssl dgst -sha256 < _
455ce847f0073c7ab3b1465f74507b75d3dc064c1e7de3b71e00de9092fdc89a

```

## 示例

简单的例子： 所有请求的req.url匹配上字符串/news的时候就从缓存中限制服务：

```
req.url == "/news"

```

示例：禁止所有服务主机为example.com或者www.example.com，并从后台 收到Set-Cookie的头包含有"USERID=1663".

```
req.http.host ~ "^(?i)(www\.)example.com$" && obj.http.set-cookie ~ "USERID=1663"
```

# varnishhist

## 概要

```
varnishhist [-C] [-d] [-g <request|vxid>] [-h] [-L limit] [-n name] [-N filename] [-p period] [-P <size|responsetime|tag:field_num:min:max>] [-q query] [-r filename] [-t <seconds|off>] [-T seconds] [-V]

```

## 描述

varnishhist工具可以读取varnishd共享内存日志，并呈现出一个实时更新的展示最后N个请求分布情况的直方图。N的值和垂直刻度会显示在左上角，水平可读是对数形式的。命中的会显示|符号，未命中的会显示#号。

下列选项可供选择：

- `-c`

所有的正则表达式和字符串是否匹配

- `-d`

在日志头部开启日志记录，而不是尾部

- `-g `

日志分组。默认是按照vxid。

- `-h`

打印程序使用情况并退出。

- `-L limit`

在旧的事务处理完成之前保持的未完成事务上限。此设置会在运行查询上保持一个内存使用上限，默认为1000事务。

- `-n name`

指定varnishd实例名称以获取日志。如果未指定-n，则使用主机名。

- `-N filename`

指定一个旧的VSM实例的文件名。当使用此选项时，abandonment检查是被禁用的。

- `-p period`

指定屏幕刷新的频率，默认为1秒1次，并且可以在运行的时候通过按1-9来修改。

- `-P `

Either specify "size" or "responsetime" profile or create a new one. Define the tag we'll look for, and the field number of the value we are interested in. min and max are the boundaries of the graph (these are power of tens).

- `-q query`

指定使用的VSL查询。

- `-r filename`

从这个文件中以二进制形式读取日志。这个文件可以通过varnishlog -w filename来创建。

- `-t `

初始化VSM连接返回错误之前的超时时间。如果设置了VSM连接将会每隔一段时间就重试一次。如果设置为0，连接将会只尝试一次，如果失败就立即失效。如果设置为off，连接将不会失败，而是允许工具启动并等待varnish实例出现。默认为5妙。

- `-T seconds`

设置事务超时时间。这个会定义开始标记和结束标记之间经过的秒数。如果超时，将会记录错误，事务强制完成。默认是120秒。

- `-V`

打印版本信息并退出。

# varnishlog

## 展示varnish log

### 概要

```
varnishlog [-a] [-A] [-b] [-c] [-C] [-d] [-D] [-g <session|request|vxid|raw>] [-h] [-i taglist] [-I <[taglist:]regex>] [-k num] [-L limit] [-n name] [-N filename] [-P file] [-q query] [-r filename] [-t <seconds|off>] [-T seconds] [-v] [-V] [-w filename] [-x taglist] [-X <[taglist:]regex>]

```

### 参数

下面的参数都是可用的：

- `-a`

当使用-w参数将输出写入一个文件的时候，追加到文件末尾，而不是覆盖它。

- `-A`

当使用-w参数将输出写入一个文件的时候，输出数据。

- `-b`

只显示transactions和来自后台日志记录

- `-c`

只显示transactions和来自客户端通信日志记录

- `-C`

所有正则表达式和字符串是否匹配

- `-d`

开始处理头部日志而不是尾部记录。

- `-D`

守护进程模式

- `-g `

日志记录分组，默认是按照vxid。

- `-h`

打印程序使用情况并退出。

- `-i taglist`

输出包含这些标签的日志记录。标签列表以逗号分隔。

- `-I <[taglist:]regex>`

输出包含匹配正则表达式的标签的日志记录。如果标记列表不存在，则适用于任何标记。

- `-k num`

在退出之前处理这个数量匹配到的日志记录。

- `-L limit`

设置限制未完成事务数量保持的上限。发生这种情况时，会记录警告。此设置会保持运行查询的内存使用情况。默认为1000

- `-n name`

指定varnish实例名字，用来获取日志。如果没有指定-n参数，那么会使用host name。

- `-N filename`

指定一个旧的VSM实例的文件名。当使用此选项时，abandonment检查是被禁用的。

- `-P file`

指定记录进程PID的文件。

- `-q query`

指定使用的VSL查询。

- `-r filename`

从这个文件中以二进制形式读取日志。这个文件可以通过varnishlog -w filename来创建。

- `-t `

初始化VSM连接返回错误之前的超时时间。如果设置了VSM连接将会每隔一段时间就重试一次。如果设置为0，连接将会只尝试一次，如果失败就立即失效。如果设置为off，连接将不会失败，而是允许工具启动并等待varnish实例出现。默认为5妙。

- `-T seconds`

设置事务超时时间。这个会定义开始标记和结束标记之间经过的秒数。如果超时，将会记录错误，事务强制完成。默认是120秒。

- `-v`

为每一行的VXID记录集打印使用详细输出，如果没有这个参数，VXID将仅仅给出这个记录的头部。

- `-V`

打印版本信息并退出。

- `-w filename`

重定向输出到文件。这个会将其覆盖除非使用了-a参数。如果在守护进程模式下程序收到SIGHUP，那么文件会被重新打开，并允许旧数据回转。该文件可以被varnishlog或者其他工具使用-r选项读取，除非指定了-A选项。在守护进程模式下运行时，这个选项是必需的。

- `-x taglist`

输出时排除这些标签的日志记录。标签列表以逗号分隔，多个可以使用-x选项提供。

- `-X <[taglist:]regex>`

通过正则表达式匹配来排除标签，不输出匹配标签列表和正则表达式的日志记录。如果标记列表不存在，则适用于任何标记。

### 信号

- SIGHUP

反转日志文件

- SIGUSR1

刷新任何未完成的事务。

# varnishncsa

## 在Apache中展示varnish日志/NCSA组合日志格式化

### 概要

```
varnishncsa [-a] [-b] [-c] [-C] [-d] [-D] [-F format] [-f formatfile] [-g <request|vxid>] [-h] [-n name] [-N filename] [-P file] [-q query] [-r filename] [-t <seconds|off>] [-V] [-w filename]

```

### 描述

varnishncsa可以读取varnishd共享内存日志，并将它们按照apache/NCSA组合的格式进行格式化。

产生的每个日志行都是基于共享内存日志中单个请求类型收集而来的。该请求会扫描相关部分，以输出日志行。如果要筛选产生的日志行，可以使用查询语言来做适当的处理。非请求的都会被忽略。

下面的参数都是可用的：

- `-a`

当将输出写入文件时，追加内容而不是覆盖它。

- `-b`

记录后端请求。如果没有指定-c参数，则仅仅会记录后端请求日志。

- `-c`

记录客户端请求，这个是默认的。如果指定了-b参数，则-c参数也会记录客户端请求。

- `-C`

所有正则表达式和字符串是否匹配

- `-d`

开始处理头部日志而不是尾部记录。

- `-D`

守护进程模式

- `-F format`

设置日志输出格式化字符串

- `-f formatfile`

从一个文件读取输出格式化字符串。将会从指定文件中读取一行，并使用这一行来进行格式化

- `-g `

日志记录分组，默认是按照vxid。

- `-h`

打印程序使用状况，并退出

- `-n name`

指定varnish实例名字，用来获取日志。如果没有指定-n参数，那么会使用host name。

- `-N filename`

指定一个旧的VSM实例的文件名。当使用此选项时，abandonment检查是被禁用的。

- `-P file`

指定记录进程PID的文件。

- `-q query`

指定要使用的VSL查询字符串

- `-r filename`

从这个文件中以二进制形式读取日志。这个文件可以通过varnishlog -w filename来创建。

- `-t `

初始化VSM连接返回错误之前的超时时间。如果设置了VSM连接将会每隔一段时间就重试一次。如果设置为0，连接将会只尝试一次，如果失败就立即失效。如果设置为off，连接将不会失败，而是允许工具启动并等待varnish实例出现。默认为5妙。

- `-V`

打印版本信息并退出。

- `-w filename`

重定向输出到文件。这个会将其覆盖除非使用了-a参数。如果在守护进程模式下程序收到SIGHUP，那么文件会被重新打开，并允许旧数据回转。该文件可以被varnishlog或者其他工具使用-r选项读取，除非指定了-A选项。在守护进程模式下运行时，这个选项是必需的。

### 模式

默认的模式是“client mode”（客户端模式）。在这种模式下，日志将会和web服务器类似。客户端模式可以通过使用-c参数来使用。

如果指定了-b参数，varnishncsa将会使用"backend mode"(后端模式)。在这种模式下，通过varnish生成的后端请求都会被记录。除非指定了-c参数，客户端请求接收到之后也会被忽略。

当运行varnishncsa同时使用客户端模式和后端模式，强烈建议你包含格式化说明符`%{Varnish:side}`来区分是后端请求还是客户端请求。

在管道中（例如在vcl中的return(pipe)）产生的客户端请求，将不会再后端模式中被记录。这是因为varnish不产生请求，而是以为的在两个方向中传递字节。然而，在正常模式下，可以通过`%{Varnish:handling}x`来看到这种情况。

在后台模式下，一些字段在格式化字符串中有不同的意义。最需要注意的是，字节计数格式化(%b, %I, %O)会认为是客户端。

当然也有可能是使用了两个varnishncsa的实例，一个使用后端模式，一个使用客户端模式，记录到两个不同的文件。

### 格式化

指定使用的日志格式。如果没有指定格式，则使用默认的日志格式。

默认的日志格式为：

```
%h %l %u %t "%r" %s %b "%{Referer}i" "%{User-agent}i"

```

支持\n和\t转义字符。

支持的格式化方式如下：

- `%b`

在客户端模式下，以字节为单位统计响应的大小，不包含HTTP头。在后端模式下，是从后端接收到的字节数，不包含http头。在CLF格式中，当没有数据传输时时一个"-"而不是0.

- `%D`

在客户端模式下，是服务器请求的时间，以微妙为单位。在后端模式下是从请求发送到全部被接收的时间。

- `%H`

请求的协议。默认是HTTP/1.0。

- `%h`

远程主机。如果没有默认是“-”。在后端模式下是后端服务器的IP地址。

- `%I`

在客户端模式下，是从客户单接收的总字节数。在后端模式下，是发送到后端的总字节数。

- `%{X}i`

请求头X的内容。

- `%l`

远程登录名字（总是“-”）

- `%m`

请求方式，如果不知道的话默认是“-”

- `%{X}o`

响应头X的内容。

- `%O`

在客户端模式下，是发送到客户端的总字节数。在后端模式下，是从后端接收到的总字节数。

- `%q`

查询字符串。如果没有，则是空字符串。

- `%r`

请求的第一行，是从其他字段组合而成，因此它可能不是逐字的。

- `%s`

发送到客户端的状态。在后端模式下，是从后端接收到的状态。

- `%t`

在客户端模式下，是当请求接收到的时间，格式为HTTP date/time格式时间。在后端模式下，则是请求发送的时间。

- `%{X}t`

在客户端模式下，是请求收到的时间。在后端模式下，是请求发送的时间。时间的格式和strftime相同。

- `%T`

在客户端模式下，是服务请求话费的时间，单位秒。在后端模式下，则是从请求发送到被接收花费的时间。

- `%U`

不包含查询字符串的url。如果没有则为“-”

- `%u`

认证的远程用户

- `%{X}x`

扩展变量，支持的变量有：

1. `Varnish:time_firstbyte`：从请求处理开始到第一字节发送到客户端的时间。对于后端模式，是从请求发起到后端接收到整个头部的时间。
2. `Varnish:hitmiss`：请求是否命中缓存。pipe和pass都会被认为未命中。
3. `Varnish:handling`：请求如何被处理，无论请求缓存是hit、miss、pass、pipe、synth。
4. `Varnish:side`：后端或者客户端。两个中的某一个值，“b”或者“c”(不带引号)，这取决于发出请求的地方。在纯后端模式或者客户端模式，这个是不变的。
5. `Varnish:vxid`：varnish处理中的VXID
6. `VCL_Log:key`：在VCL中通过std.log("key:value")来设置输出值。
7. `VSL:tag or VSL:tag[field]`：给定标签的varnishlog项的值。如果字段指定了，则仅仅展示指定的部分。当key没见过时则默认为“-”。若一个key在同一事务中出现多次，则只会使用第一次出现的。例如%{VSL:Begin[2]}x将会打印开始标记的第二个字段，它是父事务的VXID。

### 信号

- SIGHUP

反转日志

- SIGUSR1

刷新所有未完成的事务