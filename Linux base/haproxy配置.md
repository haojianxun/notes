

haproxy配置

配置段：

	global：全局配置段
			进程及安全配置相关的参数
			性能调整相关参数
			Debug参数
	proxies：代理配置段
			defaults：为frontend, listen, backend提供默认配置；
			fronted：前端，相当于nginx, server {}
			backend：后端，相当于nginx, upstream {}
			listen：同时拥前端和后端
			
	简单的配置示例：
	frontend web  #web的意思是这个前端叫什么名字,每个前端都有一个名字,自己定义,为的是可以方便调用
		bind *:80 #绑定在80端口
		default_backend     websrvs #默认的后端主机名字叫什么,也就是下面backend的名字
	backend websrvs #backend的名称的什么,自定义
		balance roundrobin #采用哪种调度算法
		server srv1 172.16.100.6:80 check #server的名字叫srv1 地址是172.16.100.6:80 check的意思的健康状态检测
		server srv2 172.16.100.7:80 check
## 配置详解:

### 打开记录日志功能

```Bash
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
这段的意思是要加日志信息


具体如下:
1. vim /etc/rsyslog.cfg  #打开rsyslog的配置
local2.*   			/var/log/haproxy.log #添加haproxy日志的具体位置
$ ModLoad imudp 
$ UDPServerRun 514 #rsyslog现在默认情况下，需要在514端口监听UDP，所以可以把/etc/rsyslog.conf如下的注释去掉
systemctl restart rsyslog.service #重启,使rsyslog刚刚改的配置生效
```

### bind

```bash
frontend  main *:5000
    acl url_static       path_beg       -i /static /images /javascript /stylesheets
    acl url_static       path_end       -i .jpg .gif .png .css .js

    use_backend static          if url_static
    default_backend             app
    
上面是haproxy.cfg的配置内容
frontend main *:5000
#上面的main 指的是frontend的名字 ,*:5000:监听的是所有地址的5000端口,类似于bind关键字
#比如frontend man *:5000就等同于
#frontend main 
#bind *:5000

#关于bind的用法可以在官方文档中去看
#bind：Define one or several listening addresses and/or ports in a frontend.
#bind [<address>]:<port_range> [, ...] [param*] #这是bind的语法

#bind示例					
#listen http_proxy
#	bind :80,:443 #端口可以给多个
#	bind 10.0.0.1:10080,10.0.0.1:10443 #监听地址和端口可以有多个
#	bind /var/run/ssl-frontend.sock user root mode 600 accept-proxy

												
						

default_backend <backend>
			设定默认的backend，用于frontend中；		
default-server [param*]
			为backend中的各server设定默认选项；
		
							
				
							
	
						

							

						


```

### balance调度算法

```
balance：后端服务器组内的服务器调度算法
	balance <algorithm> [ <arguments> ]
	balance url_param <param> [check_post]
	
balance#主要用在后端服务器是设定上,是调整服务器调度算法的
<algorithm>:
roundrobin : #轮询.就是把每个请求依次平均调度的每个服务器上面.这个算法是动态的,什么是动态的呢,就是支				持haproxy在运行的时候可以动态的调整而不用重启就能立即生效,支持慢启动,每个后端最多支持				4095个server
static-rr:#静态轮询,不支持权重在运行的时候调整,调整了也不会立即生效,但它比rr的好处就是后端主机没有限			制.
leastconn:#推荐使用在具有较长会话的场景中，例如MySQL、LDAP等
first：#根据服务器在列表中的位置，自上而下进行调度；前面服务器的连接数达到上限，新请求才会分配给下一台		服务；
							
source：源地址hash；
		除权取余法：
		一致性哈希：
							
uri：#对URI的左半部分做hash计算，并由服务器总权重相除以后派发至某挑出的服务器；同一个uri请求总是发往同一个服务器,挑选服务器的算法就由hash-type来决定			This algorithm hashes either the left part of the URI (before the question mark) or the whole URI (if the "whole" parameter 	is present) and divides the hash value by the total weight of the running servers

uri的格式:
<scheme>://<user>:<password>@<host>:<port>/<path>;<params>?<query>#<frag>
		#左半部分：/<path>;<params>
		#整个uri：/<path>;<params>?<query>#<frag>
		
hash-type：哈希算法
		hash-type <method> <function> <modifier>
		map-based：除权取余法，哈希数据结构是静态的数组；
		consistent：一致性哈希，哈希数据结构是一个树；
							
		<function> is the hash function to be used : 哈希函数
			sdbm
			djb2
			wt6
									
url_param：对用户请求的uri听<params>部分中的参数的值作hash计算，并由服务器总权重相除以后派发至某挑出的服务器；通常用于追踪用户，以确保来自同一个用户的请求始终发往同一个Backend Server；
						
hdr(<name>)：对于每个http请求，此处由<name>指定的http首部将会被取出做hash计算； 并由服务器总权重相除以后派发至某挑出的服务器；没有有效值的会被轮询调度； 
hdr(Cookie)						
	rdp-cookie
	rdp-cookie(<name>)
```

### server

```bash
server <name> <address>[:[port]] [param*]
		定义后端主机的各服务器及其选项；
						
server <name> <address>[:port] [settings ...]
	default-server [settings ...]
						
	<name>：服务器在haproxy上的内部名称；出现在日志及警告信息；
	<address>：服务器地址，支持使用主机名；
	[:[port]]：端口映射；省略时，表示同bind中绑定的端口；
	[param*]：参数
	maxconn <maxconn>：当前server的最大并发连接数；
	backlog <backlog>：当前server的连接数达到上限后的后援队列长度；
	backup：设定当前server为备用服务器；
	check：对当前server做健康状态检测；
	addr ：检测时使用的IP地址；
	port ：针对此端口进行检测；
	inter <delay>：连续两次检测之间的时间间隔，默认为2000ms; 
	rise <count>：连续多少次检测结果为“成功”才标记服务器为可用；默认为2；
	fall <count>：连续多少次检测结果为“失败”才标记服务器为不可用；默认为3；
									
注意：httpchk，"smtpchk", "mysql-check", "pgsql-check" and "ssl-hello-chk" 用于定义应用层检测方法；

cookie <value>：为当前server指定其cookie值，用于实现基于cookie的会话黏性；
disabled：标记为不可用；
redir <prefix>：将发往此server的所有GET和HEAD类的请求重定向至指定的URL；
weight <weight>：权重，默认为1
```

### 统计接口stats

```
统计接口启用相关的参数：
stats enable
启用统计页；基于默认的参数启用stats page；
	- stats uri   : /haproxy?stats 
	- stats realm : "HAProxy Statistics"
	- stats auth  : no authentication
	- stats scope : no restriction
stats auth <user>:<passwd>
	认证时的账号和密码，可使用多次；					
stats realm <realm>
	认证时的realm；要是出现空白字符等要加转义字符 \
							
stats uri <prefix>
	自定义stats page uri
							
stats refresh <delay>
	设定自动刷新时间间隔；
							
stats admin { if | unless } <cond>
	启用stats page中的管理功能
							
配置示例：
listen stats
	bind :9099
	stats enable
	stats realm HAPorxy\ Stats\ Page
	stats auth admin:admin
	stats admin if TRUE		
```

### 最大连接--maxconn

```
maxconn <conns>：为指定的frontend定义其最大并发连接数；默认为2000；
	Fix the maximum number of concurrent connections on a frontend.
```

### mode

```
mode { tcp|http|health }
	定义haproxy的工作模式；
	tcp：基于layer4实现代理；可代理mysql, pgsql, ssh, ssl等协议；
	http：仅当代理的协议为http时使用；
	health：工作为健康状态检查的响应模式，当连接请求到达时回应“OK”后即断开连接；
						
示例：
listen ssh
	bind :22022
	balance leastconn
	mode tcp
	server sshsrv1 172.16.100.6:22 check
	server sshsrv2 172.16.100.7:22 check
```

### cookie

```
cookie <name> [ rewrite | insert | prefix ] [ indirect ] [ nocache ]  [ postonly ] [ preserve ] [ httponly ] [ secure ]  [ domain <domain> ]* [ maxidle <idle> ] [ maxlife <life> ]
<name>：is the name of the cookie which will be monitored, modified or inserted in order to bring persistence.
		rewirte：重写；
		insert：插入；
		prefix：前缀；
							
基于cookie的session sticky的实现：
		backend websrvs
		cookie WEBSRV insert nocache indirect
		server srv1 172.16.100.6:80 weight 2 check rise 1 fall 2 maxconn 3000 cookie srv1
		server srv2 172.16.100.7:80 weight 1 check rise 1 fall 2 maxconn 3000 cookie srv2
```

### 向后端发送真实ip地址--option forwardfor

```
option forwardfor [ except <network> ] [ header <name> ] [ if-none ]
	Enable insertion of the X-Forwarded-For header to requests sent to servers
						
在由haproxy发往后端主机的请求报文中添加“X-Forwarded-For”首部，其值前端客户端的地址；用于向后端主发送真实的客户端IP；
	[ except <network> ]：请求报请来自此处指定的网络时不予添加此首部；
	[ header <name> ]：使用自定义的首部名称，而非“X-Forwarded-For”；
```

### 自定义错误页面errorfile

```
errorfile <code> <file> #自定义错误页面
	Return a file contents instead of errors generated by HAProxy
						
<code>：is the HTTP status code. Currently, HAProxy is capable of  generating codes 200, 400, 403, 408, 500, 502, 503, and 504.
<file>：designates a file containing the full HTTP response.
						
示例：
errorfile 400 /etc/haproxy/errorfiles/400badreq.http
errorfile 408 /dev/null  # workaround Chrome pre-connect bug
errorfile 403 /etc/haproxy/errorfiles/403forbid.http
errorfile 503 /etc/haproxy/errorfiles/503sorry.http	
							
errorloc <code> <url> #当发生错误页面的时候,可以转到其他的也没
errorloc302 <code> <url>
					
errorfile 403 http://www.magedu.com/error_pages/403.html
```

### 报文处理

```
reqadd  <string> [{if | unless} <cond>]
	Add a header at the end of the HTTP request
						
rspadd <string> [{if | unless} <cond>]
	Add a header at the end of the HTTP response
						
rspadd X-Via:\ HAPorxy
						
reqdel  <search> [{if | unless} <cond>]
reqidel <search> [{if | unless} <cond>]  (ignore case)
Delete all headers matching a regular expression in an HTTP request
						
rspdel  <search> [{if | unless} <cond>]
rspidel <search> [{if | unless} <cond>]  (ignore case)
	Delete all headers matching a regular expression in an HTTP response
						
rspidel  Server.*
```

### acl

```
语法:
acl <aclname> <criterion> [flags] [operator] [<value>] ...

<aclname>：ACL names must be formed from upper and lower case letters, 			digits, '-' (dash), '_' (underscore) , '.' (dot) and ':' 				(colon).ACL names are case-sensitive.
<criterion> ：
		dst : ip
		dst_port : integer
		src : ip
		src_port : integer
		例子:		
		acl invalid_src  src  172.16.200.2
		block if invalid_src
		
		
		七层检测:
		base
		ACL derivatives :
  				base     : exact string match
  				base_beg : prefix match
  				base_dir : subdir match
  				base_dom : domain match
  				base_end : suffix match
  				base_len : length match
  				base_reg : regex match
  				base_sub : substring match
  				
  				
  		path : string
				This extracts the request's URL path, which starts 						at the first slash and ends before the question mark 					 (without the host part)./path;<params>
						
				path     : exact string match
				path_beg : prefix match
				path_dir : subdir match
				path_dom : domain match
				path_end : suffix match
				path_len : length match
				path_reg : regex match
				path_sub : substring match	
				例子:
				 acl url_static       path_beg       -i /static /images 				/javascript /stylesheets
				 
    			acl url_static       path_end       -i .jpg .gif .png 	 	
		url : string
				This extracts the request's URL as presented in the 					request. A typical use is with prefetch-capable 						caches, and with portals which need to aggregate 						multiple information from databases and keep them in 					 caches.
					
				url     : exact string match
				url_beg : prefix match
				url_dir : subdir match
				url_dom : domain match
				url_end : suffix match
				url_len : length match
				url_reg : regex match
				url_sub : substring match
  		
		
<value>的类型：
	- boolean
	- integer or integer range
	- IP address / network
	- string (exact, substring, suffix, prefix, subdir, domain)
	- regular expression
	- hex block
	
<flags>
	-i : ignore case during matching of all subsequent patterns.忽略大小写
	-m : use a specific pattern matching method
	-n : forbid the DNS resolutions 拒绝做DNS解析
	-u : force the unique id of the ACL 强制acl的名称唯一
	-- : force end of flags. Useful when a string looks like one of the flags.
	
[operator] 
匹配整数值：eq、ge、gt、le、lt
				
匹配字符串：
	- exact match     (-m str) : the extracted string must exactly match the patterns ;
	- substring match (-m sub) : the patterns are looked up inside the extracted string, and the ACL matches if any of them is found inside ;
	- prefix match    (-m beg) : the patterns are compared with the beginning of the extracted string, and the ACL matches if any of them matches.
	- suffix match    (-m end) : the patterns are compared with the end of the extracted string, and the ACL matches if any of them matches.
	- subdir match    (-m dir) : the patterns are looked up inside the extracted string, delimited with slashes ("/"), and the ACL matches if any of them matches.
	- domain match    (-m dom) : the patterns are looked up inside the extracted string, delimited with dots ("."), and the ACL matches if any of them matches.

req.hdr([<name>[,<occ>]]) : string
	This extracts the last occurrence of header <name> in an HTTP request.
					
	hdr([<name>[,<occ>]])     : exact string match
	hdr_beg([<name>[,<occ>]]) : prefix match
	hdr_dir([<name>[,<occ>]]) : subdir match
	hdr_dom([<name>[,<occ>]]) : domain match
	hdr_end([<name>[,<occ>]]) : suffix match
	hdr_len([<name>[,<occ>]]) : length match
	hdr_reg([<name>[,<occ>]]) : regex match
	hdr_sub([<name>[,<occ>]]) : substring match					
					
示例：
	acl bad_curl hdr_sub(User-Agent) -i curl
	block if bad_curl	
    
    
    
haproxy配置示例:frontend main *:80
		maxconn 6000
		errorloc 503 http://www.baidu.com
		rspadd X-Via:\ HAProxy-1.5
		rspidel server
		acl invalid_src src 172.16.0.1
		acl adminapp path_beg -i /admin
		acl static path_end -i .jpg .gif .png .jpeg .css .js .html
		acl static path_beg -i /imagis /img /stylesheets /javascripts
		use_backend staticsrvs if static
		block if invalid_src adminapp
		default_backend dynamicsrvs
		
	backend staticsrvs
		balance roundrobin
		hash-type consistent
		server web1 172.16.0.2:80 check weight 2 check inter 3000 rise 1 fall 2
	backend dynamicsrvs
		cookie SRV insert indirect nocache
		server web2 172.16.0.3:80 check cookie web2 maxconn 2000
```

