# docker网络

ovs sdn

## closed container

只有lo接口

## bridge

docker0 是个交换机 给他地址就可以也成为网桥 

查看docker网络

```
docker network inspect bridge
```



查看网桥

```
brctl show
```

docker网络的创建的时候  一半的网卡在docker上 一般的网卡在docker0上

查看网络

```
ip link show 
```

就可以看到这些网络  @后面的字符就代表另外一半网卡  那半个网卡是在docker容器中

进入docker中就可以看到另外一个网卡 使用命令`ifconfig` 查看

进入docker中  `ping` docker0网桥的话 可以看到是能ping通的

docker0桥创建的网络默认是net桥  每次创建的时候会分配一个iptables规则, 使用命令`iptables -t nat vnL` 可以看到有个`POSTROUTING`链上有个规则 : 从任何地址进来的 到达任何地址, 只要不经过docker0桥 就进行地址伪装`MASRUERADE`  相当于snat

## 联盟式网络 joined container

2个容器共享ipc等网络组件 

## host

让docker共享宿主机的网络名称空间

## none

有的docker不需要网络 比如处理数据的 他们出来完就直接把处理好的结果放到外部的存储卷上了 所以就不需要网络