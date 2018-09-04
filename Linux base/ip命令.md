# ip命令

## 语法

```
ip [ OPTIONS ] OBJECT { COMMAND | help }
```

 OBJECT := { link | addr | route }

OBJECT可以简写,只要可以唯一标识即可

## ip link

```
ip link set eth0 down  //eth0网卡down
ip link set etho name ens12222  //修改eth0名字
ip link show  //查看网卡和ip等
ip link set eth0 multicast on  //打开组播功能
ip link set eth0 netns myset //ns为namespace  把eth0网卡移到myset这个网络空间去
ip netns list //列出网络空间名称
ip netns add myset  //增加一个名为myset的网络空间名称
ip netns exec myset ip link show //在myset这个网络空间中执行ip link show这个命令
ip netns del myset //删除myset这个网络空间


```



## ip address

```
ip address add 192.168.100.12/24 dev eth0  //给eth0增加ip地址,不过增加的地址ifconfig识别不出来,可以给他增加一个label NAME 

```



