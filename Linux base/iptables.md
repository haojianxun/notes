# IPTABLES

netfilter 功能

- 防火:包过滤

  input  forward output


- NAT network addiress translation 

  ​

- mangle  拆解报文 做出修改 而后重新封装

- raw 关闭nat表上启用的连接追踪机制

iptables

- prerouting   : raw  mangle  nat

- input:     mangle  filter

- forward        filter  mangle

- output       raw  mangle  nat  filter

- postrouting    mangle  nat  

  > 允许用户自定义规则链 他们需要手动关联到指定的钩子
  >
  > 优先级  raw-mangle-nat-filter
  >
  > 一个功能  一个表  称为table

iptables规则组成

-   匹配条件

    ​	网络层首部:SourceIP ,DestionIP

    ​	传输层首部  SourceIP DestionIP

-   跳转目标 target

                          内建的处理机制: accept drop  reject snat dnat masquerane mark log...

                          	用户自定义链:

-   添加要考虑的因素

    - 实现的功能  用于判定将规则添加到哪个表
    - 报文的流向和流经位置  用于判断将规则添加到那个链
    - 报文的流向  判断规则中的源和目标
    - 匹配条件 用于编写正确的匹配规则
      1. 同类规则  匹配范围小放前面
      2. 专用某种不同规则  匹配到的可能性较多的放前面;同类别的可以用自定义链存放
      3. 通用的规则放到前面

netfilter 位于内核中TCP/IP中协议报文处理框架

iptables生成规则,可保存于文件中  

​	CentOS 5/6

```
iptables -t filter -F
service iptalbes save
```

CentOS 7  firewalld.firewall-cmd firewall-config

```
systemctl disable firewalld.service #在7上可以关掉这个firewall功能
```

iptables命令

- 链管理

  -N ,--new-chain chain  新建一个自定义链

  -X,--delete-chain  删除用户自定义的引用计数为0的链

  -F ,--flush [chain]  清空指定规则链  没有指定则全部清除

  -E ,--rename-chain old-chain new-chain  重命名链

  -Z ,--zero 置零计数器

  > packets 被本规则所匹配到的所有报文个数
  >
  > bytes 本本规则上匹配到的所有报文大小之和

  -P ,--policy  chain target  修改默认策略

- 规则管理

  -A ,--append chain rule-specification 追加新规则到指定链的尾部

  -I,--insert chain rule-specification  插入新规则到指定链的位置

  -R ,--replace chain rule-specification 替换指定规则为新规则

  -D,--delete chain rule-specification 更加规则本身删除规则

  -D,--delete rulenum 根据规则编号删除规则

- 规则显示

  -L,--list [chain] 列出规则

  ​	-v ,--verbose 详细  -vv

  ​	-n,--numeric 数字显示主机和端口号  不会反解

  ​	-x,--exact 显示计数器的精确值

  ​	--line-numbers 列出规则时,显示链上的相应编号

  -S,--list-rules 显示指定链的所有规则


> rule-specification=[matches...]\[target\]
>
> qpm -ql iptables   其中看到 大写的扩展so模块就是 target 小写就是matches模块
>
> matches :匹配条件
>
> target :跳转目标
>
> 匹配条件
>
> - 通用匹配(parameters)
>
>   [!] -s, --source address[/mask]\[,...\] 检查此报文的源ip地址
>
>   是否符合此处指定范围
>
>    [!] -d, --destination address[/mask]\[,...\] 检查此报文的目标ip地址
>
>   是否符合此处指定范围
>
>    [!] -p, --protocol protocol 匹配报文中的协议 比如tcp, udp, udplite, icmp, icmpv6,esp, ah, sctp, mh或者使用关键字all
>
> - m,--match match 调用指定的扩展匹配模块来扩展匹配条件检查机制
>
>   扩展匹配条件
>
>   ​	显示扩展:必须用-m指明扩展模块
>
>   ​	隐藏扩展:使用-p选项指明了特定的协议时候,无须-m指出扩展	 模块
>
> 隐式扩展
>
> - -p tcp:可使用tcp扩展模块块的专用选项
>
>   > [!] --source-port,--sport port[:port]匹配报文源端口,可以给出多个端口,但是只能是连续的端口
>   >
>   > [!] --destination-port,--dport port[:port]匹配报文目标端口,可以给出多个端口,但是只能是连续的端口
>   >
>   > [!] --tcp-flags mask comp匹配报文中的tcp协议的标志位,flags are: SYN ACK FIN RST URG PSH ALL NONE
>   >
>   > ​	mask :要检查的flags list ,以逗号分隔
>   >
>   > ​	comp:在mask给定的诸多flags中,其值必须为1的flags列表,余下的必须为0
>   >
>   > ​	比如:--tcp-flags SYN,ACK,FIN,RST SYN
>   >
>   > [!] --syn   和--tcp-flags SYN,ACK,FIN,RST SYN 等同
>
> - -p udp  使用udp协议扩展模块
>
>   > [!] --source-port,--sport port[:port]
>   >
>   > [!] --destination-port,--dport port[:port]
>
> - -p icmp [!] --icmp-type {type[/code]|typename} 
>
>   > 0/0 echo reply
>   >
>   >  8/0 echo request
>
>
>
> 显示扩展:必须使用-m选项来调用扩展模块
>
>
> -   multiport 以离散或者连续的方式多端口匹配条件 最多15个
>
>     > [!] --source-ports,--sports port[,port|,port:port]...
>     >
>     > [!] --destination-ports,--dports port[,port|,port:port]...
>
> -   iprange 以连续地址块的方式来指明多ip地址匹配条件
>
>     > [!] --src-range from[-to]
>     >
>     > [!] --dst-range from[-to]
>
> -   time  匹配时间
>
>     > --timestart hh:mm[:ss]
>     >
>     > --timestop hh:mm[:ss]
>     >
>     > [!] --monthdays day[,day...]
>     >
>     > [!] --weekdays day[,day...]
>     >
>     > --kerneltz 使用内核的时区  系统默认是utc时间 
>
> -   string
>
>     > --algo {bm|kmp}
>     >
>     > --from offset
>     >
>     > --to offset
>     >
>     > [!] --string pattern
>     >
>     > [!] --hex-string pattern
>
> -   connlimit  
>
>         Allows you to restrict the number of parallel connections to a server per client  IP  address
>
>     > --connlimit-upto n
>     >
>     > --connlimit-above n
>
> -   limit
>
>         This module matches at a limited rate using a token bucket filter
>
>     > --limit rate[/second|/minute|/hour|/day]
>     >
>     > --limit rate[/second|/minute|/hour|/day]
>
> -   state
>
>         The  "state"  extension  is a subset of the "conntrack" module
>
>     > [!] --state state   (INVALID, ESTABLISHED, NEW,RELATED or UNTRACKED)
>     >
>     > NEW :新建立的请求
>     >
>     > ESTABLISHED:已经建立的请求
>     >
>     > RELATED :相关联的连接,当前连接是新请求,但是附属与某个已存在的连接
>     >
>     > UNTRACKED :未追踪的 或者在raw中不追踪的
>     >
>     > INVALID:无法识别的连接
>     >
>     > contrack 能追踪到的最大连接数定义在/proc/sys/net/nf_conntrack_max
>     >
>     > 能追踪到的连接 /proc/net/nf_conntrack
>
>     ​
>
>
> 
>
>
> - [!] -i, --in-interface name 限定报文流入的接口流入(only for  packets  entering  the INPUT,  FORWARD  and  PREROUTING  chains)
>
> - [!] -o, --out-interface name 限定报文流入的接口流出(for packets entering  the FORWARD,  OUTPUT  and  POSTROUTING  chains)
>
> - 跳转目标target
>
>   -j targetname [per-target-options]
>
>   ACCEPT 接受
>
>   DROP 丢弃
>
>   REJECT 拒绝

state扩展

内核模块装载

```
modprobe conntrack
modprobe nf_conntrack_ipv4
#nf_conntrack_ftp需要手动装载
```

开放某个端口

```
iptables -I OUTPUT -s 172.16.0.6 -m state --state ESTABLISHED -j ACCEPT
```

处理动作

-j targetname[per-target-options]

保存和载入规则

保存



