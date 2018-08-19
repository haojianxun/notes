# kubernetes中查看pod内部的dns解析

查看当前coredns解析的地址是什么

```
kubectl get pod -n kube-system -o wide  //查看当前运行的pod 其中dns的地址是10.96.0.10


kubectl get svc -n kube-system   //查看dns解析的地址  
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP   1d
```



先创建一个pod

```
kubectl run test --image=busybox --restart=Never -it  //创建一个名为test的deployment , 使用-it来进行交互
```

查看/etc/resolv.conf

```
/ # cat /etc/resolv.conf 
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local localdomain
options ndots:5
```

可以看到地址解析的名称就是kube-dns的地址, 搜索域也变成default.svc.cluster.local  其中default表示的是当前pod所属名称空间的名字

解析pod地址

我当前主机上已经有一个名为nginx的service

```
#dig -t A nginx.svc.cluster.local @10.96.0.10
```



