# docker网络初探

设置网络名称空间

```
ip netns add r1
ip netns add r2

ip netns list //查看各个创建的名称空间
```

```
ip netns exec r1 ifconfig   //进入r1网络名称空间里面查看网络信息
```

创建网络虚拟网卡对

```
ip link add name veth1.1 type veth peer name vethv1.2
```

查看刚刚创建的网卡对

```
ip link show     //@符号前面的是一半网卡名称  后面的是对应的另外一半网卡名称
```

把一半网卡移到另外一个名称空间

```
ip link set dev veth1.2 netns r1
```

查看刚刚移过去的那半个网卡

```
ip netns exec r1 ifconfig -a
```

把这些网络激活

```
ifconfig veth1.1 10.1.0.1/24 up

ip netns exec r1 ifconfig eth0 10.1.0.2/24 up
```

查看这2个网卡的互相通信

```
ping 10.1.0.2
```

把在宿主机的一半网卡移到r2上

```
ip link set dev veth1.1 netns r2
```

激活移到r2上的另外一半网卡

```
ip netns exec r2 ifconfig veth1.1 10.1.0.3 up
```





创建一个网桥

```
docker network create -d bridge --subnet "172.26.0.0/16" --gateway "172.20.0.1" mybr1
```

可以创建docker容器的时候 , `docker run`的时候使用短选项`--network`来指定要用的网桥