# DNS AND BIND

[TOC]

# bind

​	主配置文件 /etc/named.conf

​	解析库文件 /var/named

> 必须要有根区域解析库文件 named.ca
>
> 还应该有2个区域解析库文件 localhost和127.0.0.1的正反向解析库:
> ​	正向的解析库 named.localhost
>
> ​	反向的解析库 named.loopback

**rndc :remote name domain contoller**

​	953/tcp 监听在127.0.0.1地址 仅允许本地使用

bind的程序安装完成之后,启动:

​	CentOS 6 :`service named start` 

​	CentOS 7 `systemctl start named.service` 

主配置文件格式

​	全局配置段

​		options{....}

​	日志配置段:

​		logging{...}

​	区域配置段

​		zone{...}

> 每个配置文件段配置要以分号结尾

学习的时候建议关闭dnssec

dessec-enable no

dessec-validation no

dessec-lookaside no

检查配置文件语法错误

`named-checkconf`  [/etc/named.conf]

测试工具

dig host,nslookup

## dig命令

​	dig [-t RR_TYPE]name [@service]\[query options\]

​	测试dns系统

> +[no]trace 跟踪解析过程
>
> +[no]recurse 进行递归解析
>
> -x 反向解析

​	dig -t axfr DOMAIN [@service] 模拟完全区域传送

rndc命令 named服务控制命令

rndc status

rndc fiush

## bind配置

### 配置解析一个正向区域

1. #### 定义区域

   在主配置文件中/etc/named.conf   /etc/named.rfc1912.zones

   > RFC :Request For Comments 征求意见稿,是由互联网工程任务组（IETF）发布的一系列备忘录。文件收集了有关互联网相关信息，以及UNIX和互联网社区的软件文件，以编号排定.1996年3月，清华大学提交的适应不同国家和地区中文编码的汉字统一传输标准被IETF通过为RFC 1922，成为中国大陆第一个被认可为RFC文件的提交协议

   ```
   zone "ZONE_NAME" IN {
     	type {master|slave|hint|forward};
     	file "ZONE_NAME.zone";
   };
   ```

   ​

2. #### 建立区域数据文件,在/var/named/xxx建立,好了以后修改权限和属组 

   chgrp named /var/named/xxx.zone

   chmod o= /var/named/xxxx.zone

   最后检查语法错误 `named-checkzone ZONE_NAME ZONE_FILE`

   ​				`named-checkconf`

3. #### 让服务器重载配置文件和区域数据文件

   `rndc reload`   或者 `systemctl reload named.service`


### 配置解析一个反向区域

1. #### 定义区域 

   ```
   zone "ZONE_NAME" IN {
     	type {master|slave|hint|forward};
     	file "ZONE_NAME.zone";
   };
   ```

   注意 反向区域的名字

   ​      反写网段地址.in-addr.arpa

   ​       比如 100.16.172.in-addr.arpa

2. ## 定义区域解析库文件(主要记录是PTR)

   示例:

   ```
   $TTL 3600
   $ORIGN 100.16.172.in-addr.arpa.
   @	IN 		SOA		ns1.magedu.com.(
   						2016090811
   						1H
   						10M
   						3D
   						12H)
       IN      NS       ns1.magedu.com
    67 IN      PTR      ns1.magedu.com
    68 IN      PTR      mx1.magedu.com

   ```

   ​

   ​

## 主从服务器配置

注意 从带服务器试试区域级别的概念

#### 配置一个从区域

#### 	 on slave

1. 定义区域

   zone "ZONE_NAME" IN {

   ​	type slave;

   ​	file "salve/ZONE_NAME.zone";

   ​	masters { MASTER_IP; };

   };

2. 重载配置

   rndc reload

#### on master

1. ### 在区域数据文件中加入从服务器NS记录 在正向区域解析文件中加入A记录 A记录的地址是从服务器地址

   主要时间要同步

   ntpdate命令

### **子域授权**

正向解析区域授权子域的方法

ops.magedu.com. 		IN 		NS		ns1.magedu.com

ops.magedu.com.		IN  		NS		ns2.magedu.com

ns1.ops.magedu.com.		IN 		A		IPADDR

ns2.ops.magedu.com		IN		A		IPADDR

定义转发

注意 被转发的服务器必须允许为当前服务器做递归

1. 区域转发 仅转发对某特定区域的解析请求

   zone "ZONE_NAME"   IN {

   ​	type forward;

   ​	forward {frist|only};

   ​	forwarders { SERVER_IP; };

   };

## acl 访问控制列表

示例

acl mynet {

​	172.16.0.0/16;

​	127.0.0.0/8;

};

**bind有4个内置acl**

​	none 没有一个主机

​	any 任意主机

​	local 本机

​	localhost 本机所在ip所属网络

**访问控制列表**

​	allow-query {};  允许查询的主机 白名单

​	allow-transfer {}; 允许向哪些主机做区域传送 默认为向所有主机  为了安全应该为从服务器

​	allow-recursion {}; 允许那些主机向当前服务器发起递归查询请求

​	allow-update {};ddns 允许动态更新区域数据库文件中的内容





# **涉及概念**

## **网域名称系统（Domain Name System，缩写：DNS）**

> 是互联网的一项服务。它作为将域名和IP地址相互映射的一个分布式数据库，能够使人更方便地访问互联网。DNS使用TCP和UDP端口53

DNS的记录类型有以下几种:

| type  | MEANING                                  |
| :---- | :--------------------------------------- |
| A     | Address  记录ipv4地址                        |
| AAAA  | ipv6地址                                   |
|       |                                          |
| CNAME | the canonical name for an alias 记录别名     |
|       |                                          |
| NS    | an authoritative name server 名称服务器:    委托DNS区域（DNS zone）使用已提供的权威域名服务器 |
|       |                                          |
| SOA   | marks the start of a zone of authority  权威记录的起始: 指定有关DNS区域的权威性信息，包含主要名称服务器、域名管理员的电邮地址、域名的流水式编号、和几个有关刷新区域的定时器。 |
|       |                                          |
| MX    | mail exchange 电邮交互记录: 引导域名到该域名的邮件传输代理（MTA, Message Transfer Agents）列表。 |
|       |                                          |
| PTR   | a domain name pointer 指针记录 :引导至一个规范名称（Canonical Name）。与 CNAME 记录不同，DNS“不会”进行进程，只会传回名称。最常用来运行反向 DNS 查找，其他用途包括引作 DNS-SD |
|       |                                          |
| NSEC  | 下一代安全记录,DNSSEC 的一部分 — 用来验证一个未存在的服务器，使用与 NXT（已过时）记录的格式 |

更多的信息查看:

[IANA DNS Parameters registry. [2008-05-25]..](http://www.iana.org/assignments/dns-parameters)

## **根域名服务器（root name server）**

> 是互联网域名解析系统（DNS）中最高级别的域名服务器，负责返回顶级域名的权威域名服务器的地址。截至2014年10月，全球有504台根服务器，被编号为A到M共13个标号

## **DNS解析过程**

![86c708cd47ebe3f677e6cda2db6491411b22bf9a13758-TfWRUQ](C:\Users\Administrator\Desktop\86c708cd47ebe3f677e6cda2db6491411b22bf9a13758-TfWRUQ.png)

举一个例子，google.com作为一个域名就和IP地址64.233.189.104相对应。DNS在我们直接调用网站的名字以后就会将像google.com一样便于人类使用的名字转化成像64.233.189.104一样便于机器识别的IP地址。

DNS查询有两种方式：**递归**和**迭代**。

以查询zh.wikipedia.org为例：
客户端发送查询报文"query zh.wikipedia.org"至DNS服务器，DNS服务器首先检查自身缓存，如果存在记录则直接返回结果。
如果记录老化或不存在，则

1. DNS服务器向根域名服务器发送查询报文"query zh.wikipedia.org"，根域名服务器返回.org域的权威域名服务器地址，这一级首先会返回的是顶级域名的权威域名服务器。
2. DNS服务器向.org域的权威域名服务器发送查询报文"query zh.wikipedia.org"，得到.wikipedia.org域的权威域名服务器地址。
3. DNS服务器向.wikipedia.org域的权威域名服务器发送查询报文"query zh.wikipedia.org"，得到主机zh的A记录，存入自身缓存并返回给客户端。

## BIND

> BIND（Berkeley Internet Name Daemon）是现今互联网上最常使用的DNS服务器软件[3]，使用BIND作为服务器软件的DNS服务器约占所有DNS服务器的九成。BIND现在由互联网系统协会（Internet Systems Consortium）负责开发与维护。

## **ICANN **

> The Internet Corporation for Assigned Names and Numbers  互联网名称与数字地址分配机构，负责在全球范围内对 互联网通用 顶级域名（gTLD ）以及国家和地区顶级域名（ccTLD ）系统的管理、以及根服务器系统的管理

## DNSSEC

> 域名系统安全扩展（Domain Name System Security Extensions)是Internet工程任务组 （IETF）的对确保由域名系统 （DNS）中提供的关于互联网协议 （IP）网络使用特定类型的信息规格套件。它是对DNS提供给DNS客户端（解析器）的DNS数据来源进行认证，并验证不存在性和校验数据完整性验证，但不提供或机密性和可用性。

FQDN

> 完全限定域名（Fully qualified domain name），又译为完全资格域名、完整领域名称，又称为绝对领域名称（absolute domain name）、 绝对域名，网域名称的一种，能指定其在域名系统 (DNS) 树状图下的一个确实位置。一个完全资格域名会包含所有域名级别，包括 顶级域名 和 根域名。完整网域名称这个名称的由来，是因为它没有模糊空间，只能用一种方式来解析。完整网域名称是因应互联网上需要一个统一识别方式而出现，在1980年代后期快速成长。
>
> 完整网域名称由主机名称与母网域名称两部分所组成，例如有一部服务器的本地主机名为myhost，而其母域名为example.com，那指向该服务器的完整网域名称就是myhost.example.com。虽然世界上可能有很多服务器的本地主机名是myhost，但myhost.example.com是唯一的，因此完整网域名称能识别该特定服务器