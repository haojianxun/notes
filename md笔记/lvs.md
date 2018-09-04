# lvs

[TOC]

## lvs产生背景

电子商务的大力发展,以及网页的巨额流量,爆炸式的访问带来的不堪重负,导致用户进行长时间等待就迫切需要解决这些问题,总结如下

- 可伸缩性（Scalability），当服务的负载增长时，系统能被扩展来满足需求，且不降低服务质量
- 高可用性（Availability），尽管部分硬件和软件会发生故障，整个系统的服务必须是每天24小时每星期7天可用的。
- 可管理性（Manageability），整个系统可能在物理上很大，但应该容易管理。
- 价格有效性（Cost-effectiveness），整个系统实现是经济的、易支付的

 ![Linux虚拟服务框架](http://www.ibm.com/developerworks/cn/linux/cluster/lvs/part1/3.gif)

Linux虚拟服务器框架



LVS其实就是给内核打了一个补丁，让内核支持Layer4层的转发，以做到负载均衡的目的。LVS中一个连接占用128 bytes，因此1G内存的LVS可以处理800万以上的请求。



### lvs的体系结构

lvs主要有3个部分,分别是负载调度器,服务器群,后端存储

- 负载调度器是负责接收前端用户请求,之后转发到后端服务器群处理,该调度器可以使用IP负载均衡技术,基于内容分发等技术,比如著名的keepalived
- 服务器集群可以是一组执行用户请求的服务器,比如web,mail,dns,db等
- 后端存储,后端存储为服务器提供一个共享存储,其实也可以自己单独弄,可是这样麻烦,数据同步就是问题,有时候需要一个共同的数据,更新的时候就得更新3份,还有实时更新,最好是弄自己的单独存储


负载调度器又叫做Director/Load Balancer，服务器池叫做Realserver/Server Pool,后端存储Backend Storage


## ipvs的3种负载均衡技术

1. **Virtual Server via Network Address Translation（VS/NAT）**

   通过网络地址转换，调度器重写请求报文的目标地址，根据预设的调度算法，将请求分派给后端的真实服务器；真实服务器的响应报文通过调度器时，报文的源地址被重写，再返回给客户，完成整个负载调度过程。

2. **Virtual Server via IP Tunneling（VS/TUN）**
   采用NAT技术时，由于请求和响应报文都必须经过调度器地址重写，当客户请求越来越多时，调度器的处理能力将成为瓶颈。为了解决这个问题，调度器把请求报文通过IP隧道转发至真实服务器，而真实服务器将响应直接返回给客户，所以调度器只处理请求报文。由于一般网络服务应答比请求报文大许多，采用VS/TUN技术后，集群系统的最大吞吐量可以提高10倍。

3. **Virtual Server via Direct Routing（VS/DR）**
   VS/DR通过改写请求报文的MAC地址，将请求发送到真实服务器，而真实服务器将响应直接返回给客户。同VS/TUN技术一样，VS/DR技术可极大地提高集群系统的伸缩性。这种方法没有IP隧道的开销，对集群中的真实服务器也没有必须支持IP隧道协议的要求，但是要求调度器与真实服务器都有一块网卡连在同一物理网段上。

## 八种负载调度算法：

分为静态方法和动态方法

- 静态方法

  > RR：roundrobin，轮询；
  >
  > WRR：Weighted RR，加权轮询；
  >
  > SH：Source Hashing，实现session sticy，源IP地址hash；将来自于同一个IP地址的请求始终发往第一次挑中的RS，从而实现会话绑定；
  >
  > DH：Destination Hashing；目标地址哈希，将发往同一个目标地址的请求始终转发至第一次挑中的RS，典型使用场景是正向代理缓存场景中的负载均衡

- 动态方法

  Overhead=

> LC：least connections
>
>  	Overhead=activeconns*256+inactiveconns
>
> WLC：Weighted LC
>
> ​	Overhead=(activeconns*256+inactiveconns)/weight
>
> SED：Shortest Expection Delay
>
> ​	Overhead=(activeconns+1)*256/weight
>
> NQ：Never Queue
>
> LBLC：Locality-Based LC，动态的DH算法；
>
> LBLCR：LBLC with Replication，带复制功能的LBLC



1. **轮叫（Round Robin）**
   调度器通过"轮叫"调度算法将外部请求按顺序轮流分配到集群中的真实服务器上，它均等地对待每一台服务器，而不管服务器上实际的连接数和系统负载。
2. **加权轮叫（Weighted Round Robin）**
   调度器通过"加权轮叫"调度算法根据真实服务器的不同处理能力来调度访问请求。这样可以保证处理能力强的服务器处理更多的访问流量。调度器可以自动问询真实服务器的负载情况，并动态地调整其权值。
3. **最少链接（Least Connections）**
   调度器通过"最少连接"调度算法动态地将网络请求调度到已建立的链接数最少的服务器上。如果集群系统的真实服务器具有相近的系统性能，采用"最小连接"调度算法可以较好地均衡负载。
4. **加权最少链接（Weighted Least Connections）**
   在集群系统中的服务器性能差异较大的情况下，调度器采用"加权最少链接"调度算法优化负载均衡性能，具有较高权值的服务器将承受较大比例的活动连接负载。调度器可以自动问询真实服务器的负载情况，并动态地调整其权值。
5. **基于局部性的最少链接（Locality-Based Least Connections）**
   "基于局部性的最少链接" 调度算法是针对目标IP地址的负载均衡，目前主要用于Cache集群系统。该算法根据请求的目标IP地址找出该目标IP地址最近使用的服务器，若该服务器是可用的且没有超载，将请求发送到该服务器；若服务器不存在，或者该服务器超载且有服务器处于一半的工作负载，则用"最少链接"的原则选出一个可用的服务器，将请求发送到该服务器。
6. **带复制的基于局部性最少链接（Locality-Based Least Connections with Replication）** "带复制的基于局部性最少链接"调度算法也是针对目标IP地址的负载均衡，目前主要用于Cache集群系统。它与LBLC算法的不同之处是它要维护从一个目标IP地址到一组服务器的映射，而LBLC算法维护从一个目标IP地址到一台服务器的映射。该算法根据请求的目标IP地址找出该目标IP地址对应的服务器组，按"最小连接"原则从服务器组中选出一台服务器，若服务器没有超载，将请求发送到该服务器，若服务器超载；则按"最小连接"原则从这个集群中选出一台服务器，将该服务器加入到服务器组中，将请求发送到该服务器。同时，当该服务器组有一段时间没有被修改，将最忙的服务器从服务器组中删除，以降低复制的程度。
7. **目标地址散列（Destination Hashing）**
   "目标地址散列"调度算法根据请求的目标IP地址，作为散列键（Hash Key）从静态分配的散列表找出对应的服务器，若该服务器是可用的且未超载，将请求发送到该服务器，否则返回空。
8. **源地址散列（Source Hashing）**
   "源地址散列"调度算法根据请求的源IP地址，作为散列键（Hash Key）从静态分配的散列表找出对应的服务器，若该服务器是可用的且未超载，将请求发送到该服务器，否则返回空。


## 集群/负载均衡

### 1. 为什么是集群

典型场景:

一个web站点,一开始没有什么人访问,计算机压力不大,后来人渐渐的多了起来,网站吃不消,开始加内存,加硬盘,加RAID,但是再怎么加硬件也是跑不赢越来越多的人访问,而且加硬件成本极高,又受到摩尔定律的限制所以成本高,故加几台机器一起负载才是正道



集群

就是多台机器组成的一个集合,在客户端看来就像是一台机器,可以并发处理提高速度

负载均衡

多台计算机地位相同,单独对外提供服务,通过负载均衡技术把外部的请求按照某种算法分配到不同的机器上

### 集群分类

- 高可用集群High-availability (HA) clusters
- 负载均衡集群Load balancing clusters
- 高性能计算集群High-performance（[HPC](https://zh.wikipedia.org/wiki/%E8%B6%85%E7%BA%A7%E8%AE%A1%E7%AE%97%E6%9C%BA)）clusters
- 网格计算Grid computing

#### 高可用集群

一般是指当集群中有某个节点失效的情况下，其上的任务会自动转移到其他正常的节点上。还指可以将集群中的某节点进行离线维护再上线，该过程并不影响整个集群的运行

A=MTBF/(MTBF+MTTR)  MTBF:平均无故障时间   MTTR:平均修复时间

99%  99.99%  99.999% ....

#### 负载均衡集群

负载均衡集群运行时，一般通过一个或者多个前端负载均衡器，将工作负载分发到后端的一组服务器上，从而达到整个系统的高性能和高可用性。这样的计算机集群有时也被称为服务器群（Server Farm）。一般高可用性集群和负载均衡集群会使用类似的技术，或同时具有高可用性与负载均衡的特点

#### 高性能计算机集群

高性能计算集群采用将计算任务分配到集群的不同计算节点而提高计算能力，因而主要应用在科学计算领域。比较流行的HPC采用Linux操作系统和其它一些免费软件来完成并行运算。这一集群配置通常被称为[Beowulf集群](https://zh.wikipedia.org/wiki/Beowulf%E9%9B%86%E7%BE%A4)。这类集群通常运行特定的程序以发挥HPC cluster的并行能力。这类程序一般应用特定的运行库，比如专为科学计算设计的[MPI](https://zh.wikipedia.org/w/index.php?title=%E6%B6%88%E6%81%AF%E4%BC%A0%E9%80%92%E6%8E%A5%E5%8F%A3MPI&action=edit&redlink=1)库。

HPC集群特别适合于在计算中各计算节点之间发生大量数据通讯的计算作业，比如一个节点的中间结果或影响到其它节点计算结果的情况。

#### 网格计算

网格计算或网格集群是一种与集群计算非常相关的技术。网格与传统集群的主要差别是网格是连接一组相关并不信任的计算机，它的运作更像一个计算公共设施而不是一个独立的计算机。还有，网格通常比集群支持更多不同类型的计算机集合。

网格计算是针对有许多独立作业的工作任务作优化，在计算过程中作业间无需共享数据。网格主要服务于管理在独立执行工作的计算机间的作业分配。资源如存储可以被所有节点共享，但作业的中间结果不会影响在其他网格节点上作业的进展

## LB cluster

LB Cluster的实现：

	硬件：
		F5 Big-IP
		Citrix Netscaler
		A10 A10
	软件：
		lvs：Linux Virtual Server
		nginx
		haproxy
		ats：apache traffic server 
		perlbal
		pound
			
	基于工作的协议层次划分：
		传输层（通用）：（DPORT）
			lvs：
			nginx：（stream）
			haproxy：（mode tcp）
		应用层（专用）：（自定义的请求模型分类）
			proxy sferver：
			http：nginx, httpd, haproxy(mode http), ...
			fastcgi：nginx, httpd, ...
			mysql：mysql-proxy, ...
			...

## ipvs

```
grep -i -C 10 "ipvs" /boot/config-VERSION-RELEASE.x86_64
支持的协议有 TCP， UDP， AH， ESP， AH_ESP,  SCTP

安装工具ipvsadm
程序包ipvsadm
主程序：/usr/sbin/ipvsadm
规则保存工具：/usr/sbin/ipvsadm-save
规则重载工具：/usr/sbin/ipvsadm-restore
配置文件：/etc/sysconfig/ipvsadm-config

```



## ipvsadm

ipvsadm是lvs的管理工具,IPVS(IP Virtual Server)是IP虚拟服务器

查看内核是否支lvs补丁

```
modprobe ip_vs
cat /proc/net/ip_vs
```



#### syntax:

```Bash
管理集群服务:(用大写字母)
ipvsadm -A|E -t|u|f service-address [-s scheduler][-p [timeout]] [-M netmask] [-b sched-flags]
#-A add 添加虚拟服务器
#-E edit 编辑服务器
#-t|u|f //-t tcp-server  //-u udp-server  //-f 经过firewall时候标记的服务
#-s 使用的调度算法 默认是wlc
#-p 持久性连接。这个选项的意思是来自同一个客户的多次请求，将被同一台真实的服务器处理。timeout 的默认值为300 秒。在SSL和FTP时，该选项比较重要。在处理FTP连接时，如果采用的是TUN和DR模式，则该选项必须使用，如果使用的是NAT模式，则该选项不是必须的，但是必须手动的载入ip_vs_ftp模块。
#-M 持久性连接的粒度

ipvsadm -D -t|u|f service-address
#-D delete
ipvsadm -C
#-C 清除所有的虚拟服务器
ipvsadm -R
#-R restore 重新载入备份文件,可以使用重定向
ipvsadm -S [-n]
#-S 保存 save之意 可以使用重定向来保存到特定的文件路径


管理集群上RS:(用小写字母)
ipvsadm -a|e -t|u|f service-address -r server-address [-g|i|m] [-w weight] [-x upper] [-y lower]
#-w weight 权重
#[-g|i|m]  //-g gatway dr类型 //-i ipip 隧道类型 tun类型  //-m masquerade nat模式
#-x 指定服务器连接数的上限阀值。范围0-65535，如果为0则没有上限，如果为该范围内的其他值，则超过该值后，将不会有请求发送到该服务器
#-y 指定重新发送请求到该服务器时的下限值。范围是0-65535，如果是0，则没有设置下限阀值。当服务器释放的连接数小于该下限值时，该服务器将会接收新的连接。如果没有设置下限，但是设置了上限，则当连接释放掉上限的四分之三时，该服务器将会接收新的请求。
ipvsadm -d -t|u|f service-address -r server-address
#-d delete
ipvsadm -L|l [options]
#-L 显示
#[option]
	--numeric, -n：numeric output of addresses and ports #用数字显示地址,不反解
	--exact：expand numbers (display exact values)
	--connection， -c：output of current IPVS connections #当前连接
	--stats：output of statistics information #统计
	--rate ：output of rate information #速率
ipvsadm -Z [-t|u|f service-address]
#-Z 将虚拟服务器计数器清零 zero
ipvsadm --set tcp tcpfin udp #设置超时时间
ipvsadm --start-daemon state [--mcast-interface interface][--syncid syncid]
#--mcast-interface 指定sync daemon发送组播消息的接口，或者是sync backup daemon监听组播的接口
#--start-daemon 启动同步守护进程。可以跟上master或backup，用来说明LVS Router是master还是backup，该功能可以用keepalived来实现
ipvsadm --stop-daemon state
#--stop-daemon 停止同步守护进程
ipvsadm -h
#-h 帮助
=================================

```



ipvs规则备份和恢复


```Bash
备份
ipvsadm -A -t 172.16.0.100:80 -s rr
ipvsadm -a -t 172.16.0.100:80 -r 172.16.0.1:80 -g
ipvsadm -a -t 172.16.0.100:80 -r 172.16.0.2:80 -g
ipvsadm-save -n > /tmp/test.ipvsadm

恢复
ipvsadm-restore < /tmp/test.ipvsadm #ipvsadm-restore等同于ipvsadm -R
ipvsadm -R </tmp/test.ipvsadm
```



## lvs-nat

```Bash
1.设置防火墙
iptables -t nat -A PREROUTING -i eth1 -p tcp -d 192.168.0.253 --dport 80 -j DNAT --to-dest 172.16.10.100:80 #vip是172.16.10.100:80 所有发到本机的地址都转发到vip上
cat /proc/sys/net/ipv4/ip_forward #查看ipv4的转发功能打开了没,如果没有就开启

2.配置ipvsadm规则
ipvsadm -A -t 172.16.10.100:80 -s rr
ipvsadm -a -t 172.16.10.100:80 -r 172.17.10.1:80 -m
ipvsadm -a -t 172.16.10.100:80 -r 172.17.10.2:80 -m

3.增加vip
ifconfig eth0:1 172.16.10.100 netmask 255.255.255.0 up

4.配置RS01和RS02
route add default gw 172.16.10.100

5.测试
客户端访问192.168.0.253,经过防火墙会转化成172.16.10.100
curl http://192.168.0.253
RS01
curl http://192.168.0.253
RS02

同一网段做lvs-nat:
同一网段：指的是客户端和VIP处于同一网段。
正常情况下，客户端和VIP不能处于同一网段。
在同一网段时，LVS会发送ICMP Redirect，需要在LVS端抑制发送重定向。并且RS端需要删除网络路由，使其走LVS转发流量。

1.开启ip_forward
cat /proc/sys/net/ipv4/ip_forward
2.关闭ICMP Redirect
more /proc/sys/net/ipv4/conf/{all,default,eth0}/send_redirects#查看默认值
echo 0 > /proc/sys/net/ipv4/conf/all/send_redirects#关闭重定向
echo 0 > /proc/sys/net/ipv4/conf/default/send_redirects
echo 0 > /proc/sys/net/ipv4/conf/eth0/send_redirects
之后在删除同网段的路由



```

