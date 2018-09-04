# nginx配置详解

[TOC]

## nginx介绍

nginx的程序架构

- 采用master/worker架构

- 一个master进程,负责加载和分析配置文件、管理worker进程、平滑升级

- 一个或多个worker进程 处理并响应用户请求

- 缓存相关的进程

  cache loader：载入缓存对象

  cache manager：管理缓存对象

### nginx特性

异步、事件驱动和非阻塞

并发请求处理：通过kevent/epoll/select，/dev/poll

文件IO：高级IO sendfile，异步，mmap

nginx高度模块：高度模块化，但其模块早期不支持DSO机制；近期版本支持动态装载和卸载

模块分类:

- 核心模块:core module
- 标准模块:http module等等
- 第三方模块

**nginx的功用**

- 静态的web资源服务器；(图片服务器，或js/css/html/txt等静态资源服务器)
- 结合FastCGI/uwSGI/SCGI等协议反代动态资源请求；
- http/https协议的反向代理；
- imap4/pop3协议的反向代理
- tcp/udp协议的请求转发

## nginx安装

```
1.配置yum源,yum安装
http://nginx.org/packages/centos/7/x86_64/RPMS/
yum -y install nginx


2.编译安装
yum install pcre-devel openssl-devel zlib-devel

useradd -r nginx

./configure --prefix=/usr/local/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_dav_module --with-http_stub_status_module --with-threads --with-file-aio

make && make install
```

# 配置文件

### 配置

配置文件的组成部分:

- 主配置文件：nginx.conf

  include conf.d/*.conf

- fastcgi， uwsgi，scgi等协议相关的配置文件

- mime.types：支持的mime类型



主配置文件的配置指令：

```
directive value [value2 ...];

注意：
	(1) 指令必须以分号结尾；
	(2) 支持使用配置变量；
		内建变量：由Nginx模块引入，可直接引用；
		自定义变量：由用户使用set命令定义；
			set variable_name value;
			引用变量：$variable_name;
			

```

## 主配置文件结构

```
main block：主配置段，也即全局配置段；
	event {
	...
}：事件驱动相关的配置；
http {
	...
}：http/https 协议相关的配置段；
mail {
	...
}
stream {
	...
}
			
http协议相关的配置结构
http {
	...
	...：各server的公共配置
	server {
			...
	}：每个server用于定义一个虚拟主机；
	server {
		...
		server_name
		root
		alias
		location [OPERATOR] URL {
		...
		if CONDITION {
				...
					}
		}
	}
}
```

### main配置段常见的配置指令

分类:

- 正常运行必备的配置
- 优化性能相关的配置
- 用于调试及定位问题相关的配置
- 事件驱动相关的配置

#### 正常 运行必备的配置

##### `user` 

```
Syntax:	user user [group];
Default:	user nobody nobody;
Context:	main
						
Defines user and group credentials used by worker processes. If group is omitted, a group whose name equals that of user is used
```

##### `pid` 

```
pid /PATH/TO/PID_FILE;
指定存储nginx主进程进程号码的文件路径；
```

##### `include`

```
include file | mask;
指明包含进来的其它配置文件片断；
```

##### `load_module`

```
load_module file;
指明要装载的动态模块；
```

#### 性能优化相关的配置

##### `worker_processes` 

```
worker_processes number | auto;
worker进程的数量；通常应该为当前主机的cpu的物理核心数
```

##### `worker_cpu_affinity` 

```
worker_cpu_affinity cpumask ...;
worker_cpu_affinity auto [cpumask];
CPU MASK：
	00000001：0号CPU
	00000010：1号CPU
```

##### `worker_priority` 

```
worker_priority number;
指定worker进程的nice值，设定worker进程优先级；[-20,20]
```

##### `worker_rlimit_nofile` 

```
worker_rlimit_nofile number;
worker进程所能够打开的文件数量上限；
```



#### 调试、定位问题

##### `daemon` 

```
daemon on|off;
是否以守护进程方式运行Nignx；
```

##### `master_process`

```
master_process on|off;
是否以master/worker模型运行nginx；默认为on；
```

##### `error_log` 

```
error_log file [level];
```



#### 事件驱动相关的配置

```
events {
	...
}
					
1、worker_connections number;
	每个worker进程所能够打开的最大并发连接数数量；					
	worker_processes * worker_connections
						
2、use method;
	指明并发连接请求的处理方法；
							
	use epoll;
							
3、accept_mutex on | off;
处理新的连接请求的方法；on意味着由各worker轮流处理新请求，Off意味着每个新请求的到达都会通知所有的worker进程；
```

### 与套接字相关的配置

##### `server`  { ... }

```
server {
	listen address[:PORT]|PORT;
	server_name SERVER_NAME;
	root /PATH/TO/DOCUMENT_ROOT;
}
```

##### `listen` 

```
listen PORT|address[:port]|unix:/PATH/TO/SOCKET_FILE
listen address[:port] [default_server] [ssl] [http2 | spdy]  [backlog=number] [rcvbuf=size] [sndbuf=size]

default_server：设定为默认虚拟主机；
ssl：限制仅能够通过ssl连接提供服务；
backlog=number：后援队列长度；
rcvbuf=size：接收缓冲区大小；
sndbuf=size：发送缓冲区大小；
```

##### `server_name` 	

```
server_name name ...;
指明虚拟主机的主机名称；后可跟多个由空白字符分隔的字符串；
支持*通配任意长度的任意字符；server_name *.magedu.com  www.magedu.*
支持~起始的字符做正则表达式模式匹配；server_name ~^www\d+\.magedu\.com$
							
匹配机制：
(1) 首先是字符串精确匹配;
(2) 左侧*通配符；
(3) 右侧*通配符；
(4) 正则表达式；
```

##### `tcp_nodelay` 

```
tcp_nodelay on | off;
在keepalived模式下的连接是否启用TCP_NODELAY选项
```

##### `sendfile` 

```
sendfile on | off;
是否启用sendfile功能；
```

### 定义路径相关的配置

##### `root path` 

```
root path; 
设置web资源路径映射；用于指明用户请求的url所对应的本地文件系统上的文档所在目录路径；可用的位置：http, server, location, if in location
```

##### `location` 

```
location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
						
在一个server中location配置段可存在多个，用于实现从uri到文件系统的路径映射；ngnix会根据用户请求的URI来检查定义的所有location，并找出一个最佳匹配，而后应用其配置；
						
=：对URI做精确匹配；例如, http://www.magedu.com/, http://www.magedu.com/index.html
location = / {
	...
}
~：对URI做正则表达式模式匹配，区分字符大小写；
~*：对URI做正则表达式模式匹配，不区分字符大小写；
^~：对URI的左半部分做匹配检查，不区分字符大小写；
不带符号：匹配起始于此uri的所有的url；
						
匹配优先级：=, ^~, ～/～*，不带符号；
						
root /vhosts/www/htdocs/
http://www.magedu.com/index.html --> /vhosts/www/htdocs/index.html
							
server {
	root  /vhosts/www/htdocs/						
	location /admin/ {
		root /webapps/app1/data/
	}
}
```

##### `alias path` 

```
alias path;
定义路径别名，文档映射的另一种机制；仅能用于location上下文；
						
注意：location中使用root指令和alias指令的意义不同；
	(a) root，给定的路径对应于location中的/uri/左侧的/；
	(b) alias，给定的路径对应于location中的/uri/右侧的/
```

##### `index`

```
index file ...;
默认资源；http, server, location
```

##### `error_page` 

```
error_page code ... [=[response]] uri;
Defines the URI that will be shown for the specified errors
```

##### `try_files` 

```
try_files file ... uri;
```

### 定义客户端请求的相关配置

##### `keepalive_timeout` 

```
keepalive_timeout timeout [header_timeout];
设定保持连接的超时时长，0表示禁止长连接；默认为75s；
```

##### `keepalive_requests` 

```
keepalive_requests number;
在一次长连接上所允许请求的资源的最大数量，默认为100;
```

##### `keepalive_disable` 

```
keepalive_disable none | browser ...;
对哪种浏览器禁用长连接；
```

##### `send_timeout` 

```
send_timeout time;
向客户端发送响应报文的超时时长，此处，是指两次写操作之间的间隔时长
```

##### `client_body_buffer_size` 

```
client_body_buffer_size size;
用于接收客户端请求报文的body部分的缓冲区大小；默认为16k；超出此大小时，其将被暂存到磁盘上的由client_body_temp_path指令所定义的位置
```

##### `client_body_temp_path` 

```
lient_body_temp_path path [level1 [level2 [level3]]];
设定用于存储客户端请求报文的body部分的临时存储路径及子目录结构和数量；					
16进制的数字；							
client_body_temp_path path  /var/tmp/client_body  1 2 2
```

### 对客户端进行限制的相关配置

##### `limit_rate` 

```
limit_rate rate;
限制响应给客户端的传输速率，单位是bytes/second，0表示无限制
```

##### `limit_except` 

```
limit_except method ... { ... }
限制对指定的请求方法之外的其它方法的使用客户端；
						
limit_except GET {
	allow 192.168.1.0/24;
	deny  all;
}
```

### 文件操作优化的配置

##### `aio` 

```
aio on | off | threads[=pool];
是否启用aio功能；
```

##### `directio` 

```
directio size | off;
在Linux主机启用O_DIRECT标记，此处意味文件大于等于给定的大小时使用，例如directio 4m;
```

##### `open_file_cache` 

```
open_file_cache off;
open_file_cache max=N [inactive=time];
nginx可以缓存以下三种信息：
		(1) 文件的描述符、文件大小和最近一次的修改时间；
		(2) 打开的目录结构；
		(3) 没有找到的或者没有权限访问的文件的相关信息；
							
max=N：可缓存的缓存项上限；达到上限后会使用LRU算法实现缓存管理；
							
inactive=time：缓存项的非活动时长，在此处指定的时长内未被命中的或命中的次数少于open_file_cache_min_uses指令所指定的次数的缓存项即为非活动项；
```

##### `open_file_cache_valid` 

```
open_file_cache_valid time;
缓存项有效性的检查频率；默认为60s; 
```

##### `open_file_cache_min_uses` 

```
open_file_cache_min_uses number;
在open_file_cache指令的inactive参数指定的时长内，至少应该被命中多少次方可被归类为活动项；
```

##### `open_file_cache_errors` 

```
open_file_cache_errors on | off;
是否缓存查找时发生错误的文件一类的信息；
```

### 各种模块

#### ngx_http_access_module模块

```
ngx_http_access_module模块：
实现基于ip的访问控制功能
					
allow address | CIDR | unix: | all;
deny address | CIDR | unix: | all;
					
http, server, location, limit_except
```

#### ngx_http_auth_basic_module模块

```
ngx_http_auth_basic_module模块
实现基于用户的访问控制，使用basic机制进行用户认证；
					
auth_basic string | off;
auth_basic_user_file file;
					
location /admin/ {
	alias /webapps/app1/data/;
	auth_basic "Admin Area";
	auth_basic_user_file /etc/nginx/.ngxpasswd;
}
						
注意：htpasswd命令由httpd-tools所提供；
```

### ngx_http_stub_status_module模块

```
用于输出nginx的基本状态信息；
					
Active connections: 291 
server accepts handled requests
	16630948 16630948 31070465 
Reading: 6 Writing: 179 Waiting: 106 	
					
Active connections: 活动状态的连接数；
accepts：已经接受的客户端请求的总数；
handled：已经处理完成的客户端请求的总数；
requests：客户端发来的总的请求数；
Reading：处于读取客户端请求报文首部的连接的连接数；
Writing：处于向客户端发送响应报文过程中的连接数；
Waiting：处于等待客户端发出请求的空闲连接数；
```

### stub_status

```
配置示例：
location  /basic_status {
		stub_status;
}
```

### ngx_http_log_module模块

```
he ngx_http_log_module module writes request logs in the specified format.
```

##### `log_format`

```
log_format name string ...;
string可以使用nginx核心模块及其它模块内嵌的变量；
```

##### `access_log` 

```
access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
access_log off;
						
访问日志文件路径，格式及相关的缓冲的配置；
buffer=size
flush=time
```

##### `open_log_file_cache`

```
open_log_file_cache max=N [inactive=time] [min_uses=N] [valid=time];
open_log_file_cache off;
缓存各日志文件相关的元数据信息；
							
max：缓存的最大文件描述符数量；
min_uses：在inactive指定的时长内访问大于等于此值方可被当作活动项；
inactive：非活动时长；
valid：验正缓存中各缓存项是否为活动项的时间间隔；
```

### ngx_http_gzip_module

##### `gzip` 

```
gzip on | off;
Enables or disables gzipping of responses.
```

##### `gzip_comp_level` 

```
gzip_comp_level level;
Sets a gzip compression level of a response. Acceptable values are in the range from 1 to 9.
```

##### `gzip_disable` 

```
gzip_disable regex ...;
Disables gzipping of responses for requests with “User-Agent” header fields matching any of the specified regular expressions.
```

##### `gzip_min_length` 

```
gzip_min_length length;
启用压缩功能的响应报文大小阈值；
```

##### `gzip_buffers` 

```
gzip_buffers number size;
支持实现压缩功能时为其配置的缓冲区数量及每个缓存区的大小；
```

##### `gzip_proxied` 

```
gzip_proxied off | expired | no-cache | no-store | private | no_last_modified | no_etag | auth | any ...;
nginx作为代理服务器接收到从被代理服务器发送的响应报文后，在何种条件下启用压缩功能的；
off：对代理的请求不启用
no-cache, no-store，private：表示从被代理服务器收到的响应报文首部的Cache-Control的值为此三者中任何一个，则启用压缩功能
```

##### `gzip_types` 

```
gzip_types mime-type ...;
压缩过滤器，仅对此处设定的MIME类型的内容启用压缩功能
```

##### 示例

```
示例：
gzip  on;
gzip_comp_level 6;
gzip_min_length 64;
gzip_proxied any;
gzip_types text/xml text/css  application/javascript;
```

### ngx_http_ssl_module模块

##### `ssl ` 

```
ssl on | off;
Enables the HTTPS protocol for the given virtual server.
```

##### `ssl_certificate` 

```
ssl_certificate file;
当前虚拟主机使用PEM格式的证书文件；
```

##### `ssl_certificate_key` 

```
ssl_certificate_key file;
当前虚拟主机上与其证书匹配的私钥文件
```

##### `ssl_protocols` 

```
ssl_protocols [SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2];
支持ssl协议版本，默认为后三个；
```

##### `ssl_session_cache` 

```
ssl_session_cache off | none | [builtin[:size]] [shared:name:size];
builtin[:size]：使用OpenSSL内建的缓存，此缓存为每worker进程私有；						
[shared:name:size]：在各worker之间使用一个共享的缓存
```

##### `ssl_session_timeout` 

```
ssl_session_timeout time;
客户端一侧的连接可以复用ssl session cache中缓存 的ssl参数的有效时长；
```

##### 配置示例

```
server {
	listen 443 ssl;
	server_name www.magedu.com;
	root /vhosts/ssl/htdocs;
	ssl on;
	ssl_certificate /etc/nginx/ssl/nginx.crt;
	ssl_certificate_key /etc/nginx/ssl/nginx.key;
	ssl_session_cache shared:sslcache:20m;
}
```

### ngx_http_rewrite_module模块

将用户请求的URI基于regex所描述的模式进行检查，而后完成替换

##### `rewrite` 

```
rewrite regex replacement [flag]
将用户请求的URI基于regex所描述的模式进行检查，匹配到时将其替换为replacement指定的新的URI；
				
注意：如果在同一级配置块中存在多个rewrite规则，那么会自下而下逐个检查；被某条件规则替换完成后，会重新一轮的替换检查，因此，隐含有循环机制；[flag]所表示的标志位用于控制此循环机制；
				
如果replacement是以http://或https://开头，则替换结果会直接以重向返回给客户端；
301：永久重定向；
					
[flag]：
last：重写完成后停止对当前URI在当前location中后续的其它重写操作，而后对新的URI启动新一轮重写检查；提前重启新一轮循环； 
break：重写完成后停止对当前URI在当前location中后续的其它重写操作，而后直接跳转至重写规则配置块之后的其它配置；结束循环；
redirect：重写完成后以临时重定向方式直接返回重写后生成的新URI给客户端，由客户端重新发起请求；不能以http://或https://开头；
permanent:重写完成后以永久重定向方式直接返回重写后生成的新URI给客户端，由客户端重新发起请求
```

##### `return` 

```
return
return code [text];
return code URL;
return URL;
				
Stops processing and returns the specified code to a client.
```

##### `rewrite_log` 

```
rewrite_log on | off;
是否开启重写日志；
```

##### `if (condition) { ... }` 

```
if (condition) { ... }
引入一个新的配置上下文 ；条件满足时，执行配置块中的配置指令；server, location；
				
condition：
比较操作符：
==
!=
~：模式匹配，区分字符大小写；
~*：模式匹配，不区分字符大小写；
!~：模式不匹配，区分字符大小写；
!~*：模式不匹配，不区分字符大小写；
文件及目录存在性判断：
	-e, !-e
	-f, !-f
	-d, !-d
	-x, !-x
```

##### `set` 

```
set $variable value;
用户自定义变量 ；
```

### ngx_http_referer_module模块

*The ngx_http_referer_module module is used to block access to a site for requests with invalid values in the “Referer” header field*

##### `valid_referers` 

```
valid_referers none | blocked | server_names | string ...;
定义referer首部的合法可用值；
					
none：请求报文首部没有referer首部；
blocked：请求报文的referer首部没有值；
server_names：参数，其可以有值作为主机名或主机名模式；
arbitrary_string：直接字符串，但可使用*作通配符；
regular expression：被指定的正则表达式模式匹配到的字符串；要使用~打头，例如 ~.*\.magedu\.com；
						
配置示例：
valid_referers none block server_names *.magedu.com *.mageedu.com magedu.* mageedu.* ~\.magedu\.;
					
if($invalid_referer) {
	return 403;
}
```

### ngx_http_proxy_module模块

*The ngx_http_proxy_module module allows passing requests to another server*

##### `proxy_pass` 

```
proxy_pass URL;
Context:	location, if in location, limit_except
				
注意：proxy_pass后面的路径不带uri时，其会将location的uri传递给后端主机；
					
server {
	...
	server_name HOSTNAME;
	location /uri/ {
			proxy http://hos[:port];
	}
	...
}
					
http://HOSTNAME/uri --> http://host/uri 
					
proxy_pass后面的路径是一个uri时，其会将location的uri替换为proxy_pass的uri；
					
server {
	...
	server_name HOSTNAME;
	location /uri/ {
		proxy http://host/new_uri/;
	}
	...
}
					
http://HOSTNAME/uri/ --> http://host/new_uri/
					
如果location定义其uri时使用了正则表达式的模式，则proxy_pass之后必须不能使用uri; 用户请求时传递的uri将直接附加代理到的服务的之后；
				
server {
	...
	server_name HOSTNAME;
	location ~|~* /uri/ {
		proxy http://host;
	}
	...
}
					
http://HOSTNAME/uri/ --> http://host/uri/；
```

##### `proxy_set_header` 

```
proxy_set_header field value;
设定发往后端主机的请求报文的请求首部的值；Context:	http, server, location
				
proxy_set_header X-Real-IP  $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

##### `proxy_cache_path` 

```
proxy_cache_path
定义可用于proxy功能的缓存；Context:	http			
				
proxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
```

##### `proxy_cache` 

```
proxy_cache zone | off;
指明要调用的缓存，或关闭缓存机制；Context:	http, server, location
```

##### `proxy_cache_key`

```
proxy_cache_key string;
缓存中用于“键”的内容；
默认值：proxy_cache_key $scheme$proxy_host$request_uri;
```

##### `proxy_cache_valid` 

```
proxy_cache_valid [code ...] time;
定义对特定响应码的响应内容的缓存时长；
				
定义在http{...}中；
proxy_cache_path /var/cache/nginx/proxy_cache levels=1:1:1 keys_zone=pxycache:20m max_size=1g;
				
定义在需要调用缓存功能的配置段，例如server{...}；
proxy_cache pxycache;
proxy_cache_key $request_uri;
proxy_cache_valid 200 302 301 1h;
proxy_cache_valid any 1m;
```

##### `proxy_cache_use_stale` 

```
proxy_cache_use_stale error | timeout | invalid_header | updating | http_500 | http_502 | http_503 | http_504 | http_403 | http_404 | off ...;

Determines in which cases a stale cached response can be used when an error occurs during communication with the proxied server.
```

##### `proxy_cache_methods` 

```
proxy_cache_methods GET | HEAD | POST ...;
If the client request method is listed in this directive then the response will be cached. “GET” and “HEAD” methods are always added to the list, though it is recommended to specify them explicitly.
```

##### `proxy_hide_header` 

```
proxy_hide_header field;
By default, nginx does not pass the header fields “Date”, “Server”, “X-Pad”, and “X-Accel-...” from the response of a proxied server to a client. The proxy_hide_header directive sets additional fields that will not be passed.
```

`proxy_connect_timeout` 

```
proxy_connect_timeout time;
Defines a timeout for establishing a connection with a proxied server. It should be noted that this timeout cannot usually exceed 75 seconds.
				
默认为60s；
```

### ngx_http_headers_module模块

*The ngx_http_headers_module module allows adding the “Expires” and “Cache-Control” header fields, and arbitrary fields, to a response header*

向由代理服务器响应给客户端的响应报文添加自定义首部，或修改指定首部的值

`add_header` 

```
add_header name value [always];
添加自定义首部；
				
add_header X-Via  $server_addr;
add_header X-Accel $server_name;
```

##### `expires` 

```
expires [modified] time;
expires epoch | max | off;
				
用于定义Expire或Cache-Control首部的值；								
```

### ngx_http_fastcgi_module模块

*The ngx_http_fastcgi_module module allows passing requests to a FastCGI server.*

##### `fastcgi_pass` 

```
fastcgi_pass address;
address为fastcgi server的地址；	location, if in location
```

##### `fastcgi_index` 

```
fastcgi_index name;
fastcgi默认的主页资源; 
```

##### `fastcgi_param parameter`

```
fastcgi_param parameter value [if_not_empty];
Sets a parameter that should be passed to the FastCGI server. The value can contain text, variables, and their combination.
```

##### 配置示例

```
配置示例1：
前提：配置好fpm server和mariadb-server服务；
location ~* \.php$ {
	root           /usr/share/nginx/html;
	fastcgi_pass   127.0.0.1:9000;
	fastcgi_index  index.php;
	fastcgi_param  SCRIPT_FILENAME  /usr/share/nginx/html$fastcgi_script_name;
	include        fastcgi_params;
}
					
配置示例2：通过/pm_status和/ping来获取fpm server状态信息；
location ~* ^/(pm_status|ping)$ {
	include        fastcgi_params;
	fastcgi_pass 127.0.0.1:9000;
	fastcgi_param  SCRIPT_FILENAME  $fastcgi_script_name;
}							
```

##### `fastcgi_cache_path` 

```
fastcgi_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
			
定义fastcgi的缓存；缓存位置为磁盘上的文件系统，由path所指定路径来定义；
				
levels=levels：缓存目录的层级数量，以及每一级的目录数量；levels=ONE:TWO:THREE
	leves=1:2:2
keys_zone=name:size
	k/v映射的内存空间的名称及大小
inactive=time
	非活动时长
max_size=size
	磁盘上用于缓存数据的缓存空间上限
```

##### `fastcgi_cache` 

```
fastcgi_cache zone | off;
调用指定的缓存空间来缓存数据；http, server, location
```

##### `fastcgi_cache_key` 

```
fastcgi_cache_key string;
定义用作缓存项的key的字符串；
```

##### `fastcgi_cache_methods`

```
fastcgi_cache_methods GET | HEAD | POST ...;
为哪些请求方法使用缓存；
```

##### `fastcgi_cache_min_uses` 

```
fastcgi_cache_min_uses number;
缓存空间中的缓存项在inactive定义的非活动时间内至少要被访问到此处所指定的次数方可被认作活动项；
```

##### `fastcgi_cache_valid`

```
fastcgi_cache_valid [code ...] time;
不同的响应码各自的缓存时长；

示例：
http {
	...
	fastcgi_cache_path /var/cache/nginx/fastcgi_cache levels=1:2:1 keys_zone=fcgi:20m inactive=120s;
	...
	server {
		...
		location ~* \.php$ {
				...
				fastcgi_cache fcgi;
				fastcgi_cache_key $request_uri;
				fastcgi_cache_valid 200 302 10m;
				fastcgi_cache_valid 301 1h;
				fastcgi_cache_valid any 1m;	
				...
		}
		...
	}
	...
}
```

##### `fastcgi_keep_conn` 

```
fastcgi_keep_conn on | off;
By default, a FastCGI server will close a connection right after sending the response. However, when this directive is set to the value on, nginx will instruct a FastCGI server to keep connections open.
```

### ngx_http_upstream_module模块

##### `upstream` 

```
upstream name { ... }
定义后端服务器组，会引入一个新的上下文；Context: http
			
upstream httpdsrvs {
	server ...
	server...
	...
}
```

`server` 

```
server address [parameters];
在upstream上下文中server成员，以及相关的参数；Context:	upstream

address的表示格式：
	unix:/PATH/TO/SOME_SOCK_FILE
	IP[:PORT]
	HOSTNAME[:PORT]
				
parameters：
	weight=number
		权重，默认为1；
	max_fails=number
		失败尝试最大次数；超出此处指定的次数时，server将被标记为不可用；
	fail_timeout=time
		设置将服务器标记为不可用状态的超时时长；
	max_conns
		当前的服务器的最大并发连接数；
	backup
		将服务器标记为“备用”，即所有服务器均不可用时此服务器才启用；
	down
		标记为“不可用”；
```

##### `least_conn` 

```
least_conn;
最少连接调度算法，当server拥有不同的权重时其为wlc;
```

##### `ip_hash`

```
ip_hash;
源地址hash调度方法；
```

##### `hash key` 

```
hash key [consistent];
基于指定的key的hash表来实现对请求的调度，此处的key可以直接文本、变量或二者的组合；
			
作用：将请求分类，同一类请求将发往同一个upstream server；
			
If the consistent parameter is specified the ketama consistent hashing method will be used instead.
				
示例：
hash $request_uri consistent;
hash $remote_addr;
```

##### `keepalive connections`

为每个worker进程保留的空闲的长连接数量

### ngx_stream_core_module模块

模拟反代基于tcp或udp的服务连接，即工作于传输层的反代或调度器

##### `stream { ... }` 

```
stream { ... }
定义stream相关的服务；Context:main
			
stream {
	upstream sshsrvs {
	server 192.168.22.2:22; 
	server 192.168.22.3:22; 
	least_conn;
	}

	server {
		listen 10.1.0.6:22022;
		proxy_pass sshsrvs;
	}
}	
```

##### `listen` 

```
listen address:port [ssl] [udp] [proxy_protocol] [backlog=number] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];
```

