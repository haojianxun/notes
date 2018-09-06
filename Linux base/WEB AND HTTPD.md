# WEB AND HTTPD

[TOC]

## web网络基础

web是当初CERN的Tim BernersLee博士提出的,最初的设想是借助文档之间的相互关联形成超文本(HypeText),连成相互参阅的www(world wide web,万维网),现在有3种www技术:

1. 把SGML(Standard Generalized Markup Language,标准通用标记语言)作为页面文本标记语言的HTML
2. HTTP 作为文档传递协议
3. URL 作为文档所在地

## http协议发展

- HTTP/9.0

  http在1990年发布,不过没有作为标准建立

- HTTP/1.0

  1996年5月作为标准发布,记载在RFC1945中

  [**RFC1945-Hypertext Transfer Protocol--HTTP/1.0**](http://www.ietf.org/rfc/rfc1945.txt)

- HTTP/1.1

  1997年1月发布,当初的标准是RFC2068.现在改成RFC2616

  [**RFC2616-Hypertext Transfer Protocol--HTTP/1.1**](http://www.ietf.org/rfc/rfc2616.txt)

- HTTP/2

  由互联网工程任务组（IETF）的Hypertext Transfer Protocol Bis（httpbis）工作小组进行开发。该组织于2014年12月将HTTP/2标准提议递交至IESG进行讨论，于2015年2月17日被批准。 HTTP/2标准于2015年5月以RFC 7540正式发表.

  Google Chrome、Mozilla Firefox、Microsoft Edge和Opera已支持HTTP/2，并默认启用。
  Internet Explorer自IE 11开始支持HTTP/2，但仅限于Windows 10 Beta，并预设激活。

  [**RFC7540-Hypertext Transfer Protocol--HTTP/2**](http://www.ietf.org/rfc/rfc7540.txt)

## 概念

### TCP/IP协议层

> 1. 应用层 
>
>    比如DNS和FTP服务就在这个层,提供应用服务通信获得
>
> 2. 传输层
>
>    对上层的应用层提供网络连接中两台计算机之间的数据传输,有个著名的协议TCP和UDP协议
>
> 3. 网络层
>
>    用来处理网络上流动的数据包,数据包是传输的最小单位,该层规定了走什么样的路径到对方计算机,它的作用就是选择一条传输线路
>
> 4. 链路层
>
>    处理连接网络的硬件部分,包括光线,网卡,硬件驱动器等等

### ARP

> Address Resolution Protocol 地址解析协议 .所谓地址解析（address resolution）就是主机在发送帧前将目标IP地址转换成目标MAC地址的过程.当发送主机和目的主机不在同一个局域网中时，即便知道目的主机的MAC地址，两者也不能直接通信，必须经过路由转发才可以。所以此时，发送主机通过ARP协议获得的将不是目的主机的真实MAC地址，而是一台可以通往局域网外的路由器的MAC地址。于是此后发送主机发往目的主机的所有帧，都将发往该路由器，通过它向外发送。这种情况称为ARP代理（ARP Proxy）

### tcp三次握手

> tcp把数据发送出去以后一定会确认数据是否到达. 握手过程用tcp的标志(flag)--SYN((synchronize)  (美: ['sɪŋkrə.naɪz]   英: ['sɪŋkrənaɪz]  同步处理))和ACK(acknowledgement)(美: [ək'nɒlɪdʒmənt]  英: [ək'nɒlɪdʒmənt] 释义:对事实、现实、存在的）承认；谢礼,确认).
>
> 发送端先发一个SYN标志的数据包给对方,接收端收到以后回传一个SYN/ACK的标志数据包用来表示传达确认到达,最后发送端回传一个ACK标志的数据包,代表握手结束,若是握手过程某个阶段莫名其妙断掉,tcp还会以相同的顺序发送相同的数据包给对方.

### URI

> URI Uniform Resource Identifier的缩写   RFC对这3个缩写单词定义如下
>
> - Uniform 
>
>   规定统一的格式方便处理多钟不同类型的资源,而不用根据上下文环境来判断
>
> - Resource
>
>   资源的定义是"可标识的任何东西",资源可以是单一的 也可以是集合
>
> - Identifier
>
>   表示可标识的对象,也叫标识符
>
> URI就是由某个协议方案表示资源的定位标识符.协议方案指的是访问资源所使用的协议类型名称,比如著名的http方案,标准的URI协议有30种左右,它们属于ICANN的IANA(Internet Assigned Numbers Authority 互联网号码分配局)管理发布.
>
> [IANA-Uniform Resource Identifier SCHEMES](http://www.iana.org/assignments/uri-schemes)
>
> URI用字符标识某一互联网资源,而URL表示资源的地点,URL是URI的子集

### 请求报文

> 请求报文首部
>
> 比如 GET  /index.html HTTP/1.1
>
> ​	Host:baidu.com
>
> 起始行的get表示请求服务器的类型 成为方法 随后的字段/index.html指明了请求的资源对象 叫做请求URI(request-URI) 随后的HTTP/1.1指明了版本号,Host被叫做请求首部字段
>
> 响应报文
>
> ![捕获](C:\Users\Administrator\Desktop\捕获.PNG)

### cookie

> http协议是不保存状态的协议(stateless),对发送过的请求和响应不保存,不做持久化处理,所以引入cookie机制,为了不让服务器保存这些状态成为负担,http引入cookie,cookie会根据服务器端发送的响应报文内的一个叫做Set-Cookie的首部字段信息,通知客户端保存cookie,当下次客户端发送请求的时候,客户端会自动在请求报文中加入cookie发送出去,服务器接收到则对比上次生成cookie是像谁发送的记录,得到之前的状态

### 传输的方法

> method 
>
> #### GET 
>
> 用来获取资源,指定的资源经过服务器解析后返回响应内容,如果请求是文本,则直接返回,如果是其他就通过CGI(Common Gateway Interface 通用网关接口)那样的程序,返回执行过的输出结果
>
> 比如:
>
> 请求GET /index.html  HTTP/1.1
>
> ​	Host:www.example.com
>
> 响应 返回index.html的页面资源
>
> #### POST 
>
> 用来传输实体的主体,它用来告诉服务器信息,比如提交的表格
>
> #### PUT
>
> put用来上传传输文件
>
> #### HEAD
>
> head用来获取报文首部,用于确认URI的有效性和资源更新的日期时间
>
> #### DELETE
>
> delete用来删除指定的资源,但是http/1.1的delete不使用验证机制,一般网站不用
>
> #### OPTIONS
>
> options用来查询服务器对请求URI指定的资源支持的方法
>
> #### TRACE 
>
> trace追踪路径,让web服务器将之前的请求通信环回给客户端的方法,一般不会用容易引发XST攻击
>
> ### CONNECT 
>
> 用隧道协议连接服务器,在和服务器通信的时候建立隧道,主要使用SSL/TLS协议加密经过网络隧道传输

### 持久连接

> 持久连接的特点是 只要任意一端没有明确提出断开连接,则保持tcp连接 

### 管线化

> 持久连接使得管线化(pipelining)成为可能.以前发生请求需要收到响应才能发送下一个请求,管线化则是不用等待回应叶可以直接发送下一个请求

### HTTP报文

> 大致分为报文首部和报文主体 两者由最初出现的空行(CR(Carrisge Return 回车符 16进制 0x0d)+LF(Line Feed 换行符 ,16进制 0x0a))来划分
>
> 报文(message)
>
> 是http通信中基本单位,由8位组字节流(octet sequence 其中octet为8个比特)组成,通过http通信传输
>
> 实体(entity)
>
> 作为请求或者响应的有效载荷数据被传输 内容是由实体首部和实体主体组成

### 分块传输编码

> 在http通信中,请求的编码实体资源尚未全部传输完成之前,浏览器无法显示请求页面.在传输大数据的时候把数据分隔成多块,让浏览器逐步显示页面,这种把实体主体分块的功能叫做分块传输编码(chunked transfer coding) 分割物叫做块(chunk)
>
> 

响应码

> 1xx  接收的请求正在处理
>
> 2xx 请求正常处理完毕  成功定向码
>
> 3xx redirection 重定向状态码  需要进行附加操作以完成请求
>
> 4xx client error 客户端错误状态码  服务器无法处理请求
>
> 5xx service error 服务器错误状态码 服务器处理请求出错