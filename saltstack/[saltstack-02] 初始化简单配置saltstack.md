# [saltstack-02] 初始化简单配置saltstack

## MASTER的主要配置

### 网络接口

默认: `0.0.0.0` (所有的网络接口都可访问)

绑定本地接口

```
interface: 192.168.200.132
```

## MINION的主要配置

### `MASTER`

缺省:`salt`

主机名或者master的IPV4地址。

缺省:`salt`

```
master: 192.168.200.132    //这个master的地址填上salt-master的ip地址
```



---

## 为SALT开启防火墙配置

在Salt minions端没有设置防火墙的必要。下面的配置只是涉及master。

```
firewall-cmd --permanent --zone=<zone> --add-port=4505-4506/tcp  


//如果没有区域可以去掉--zone
firewall-cmd --permanent  --add-port=4505-4506/tcp
```



### MASTER端白名单

```
# Allow Minions from these networks
iptables -I INPUT -s 10.1.2.0/24 -p tcp -m multiport --dports 4505,4506 -j ACCEPT
iptables -I INPUT -s 10.1.3.0/24 -p tcp -m multiport --dports 4505,4506 -j ACCEPT
# Allow Salt to communicate with Master on the loopback interface
iptables -A INPUT -i lo -p tcp -m multiport --dports 4505,4506 -j ACCEPT
# Reject everything else
iptables -A INPUT -p tcp -m multiport --dports 4505,4506 -j REJECT
```

### MINION开启iptables规则

```
iptables -A INPUT -m state --state new -m tcp -p tcp --dport 4505 -j ACCEPT
iptables -A INPUT -m state --state new -m tcp -p tcp --dport 4506 -j ACCEPT
```



---



## KEY管理

Salt在Master和Minion之间的通讯采用AES加密. 这就确保了发送给minions的命令不会被篡改， Master和Minion之间的通讯认证通过信任的已接受的key进行管理.

在发送给Minion之前，需要确保minion的key已经被Master所接受. 运行 [`salt-key`](https://docs.saltstack.cn/ref/configuration/index.html#id1)命令将列出Salt Master已知的所有keys

```
[root@node01 salt]# salt-key -L
Accepted Keys:
node01
Denied Keys:
Unaccepted Keys:
node02
Rejected Keys:
```

使用`salt-key -A` 来接受key以使Mionions可以被Master管控

```
[root@node01 salt]# salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
node02
Proceed? [n/Y] 
```

输入`y` 接受

```
[root@node01 salt]# salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
node02
Proceed? [n/Y] y
Key for minion node02 accepted.
```

当然可以使用`salt-key -a` 来使用通配符来选择接受 , 具体命令可以使用帮助查看`salt-key -h`

---



## 测试是否成功

```
salt '*' test.ping  
```

如果成功则显示`true`   ,我在本机测试如下:

```
[root@node01 salt]# salt \* test.ping
node01:
    True
node02:
    True
```



## 可能出现的报错

如果minion一直识别不了 , 表现为执行`salt-key -L`  ,命令输出中 `Unaccepted Keys` 的没有minion的, 可以去查看日志

查看minion报错日志

```
tail /var/log/salt/minion
```

如果出现的报错是这样的

```
[ERROR   ][4076] Error while bringing up minion for multi-master
```

则就是防火墙和iptables没有搞好 , 可以去添加防火墙和iptables规则 , 在学习的过程中 , 如果嫌麻烦可以直接关掉防火墙

```
systemctl stop firewalld
```

