# kubernetes初探(1)

### 描述节点的信息

```
kubectl describe node node01   //描述node01节点上的信息
```



### 创建一个应用

```
kubectl run nginx-deploy --iamge=nginx --dry-run=true 
//表示创建了一个名为nginx-deploy的deployment  镜像用的nginx的最新镜像 干跑模式为真,表示这条命令不会真的执行

kubectl run nginx-deploy --iamge=nginx
//去掉干跑模式 就会创建
```



### 查看创建的应用

```
kubectl get pods   //查看创建的pods
NAME                            READY     STATUS    RESTARTS   AGE
nginx-deploy-59c86578c8-pzkpg   1/1       Running   0          24s
```



### 查看创建的deployment

```
kubectl get deployment
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1         1         1            1           56s
```



### 查看创建pod的具体信息

```
kubectl get pods -o wide  //可以看到具体的pod的ip地址和分配的到了哪个node上
NAME                            READY     STATUS    RESTARTS   AGE       IP           NODE
nginx-deploy-59c86578c8-pzkpg   1/1       Running   0          4m        10.244.1.2   localhost.localdomain
```



### 可以访问这个nginx

```
curl 10.244.1.2   //可以看到welcome to nginx的信息  在节点上使用ifconfig可以看到有个cni的网络桥 这个就是kubernetes集群用于内部访问的地址
```



### 创建一个service

如果这个nginx被删除 他还是会被创建出一个的, 但是ip地址已经变了  如果只是通过ip地址访问 显然不能访问了 , 所以创建出了一个service 通过service的地址来访问nginx ,  这样就算是nginx消失和创建 nginx的ip地址再怎么变 也能访问

可以使用 kubectl expose -h  来查看用法

注意:expose中  target port指的是是pod的端口  而--port指的端口是service的端口

```
kubectl expose deployment nginx-deploy --name=ngix --port=80 --target-port=80 --protocol=TCP
```



### 查看创建的service

```
#kubectl get svc

NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   1d
ngix         ClusterIP   10.107.162.167   <none>        80/TCP    37s
```



### 访问service

```
curl 10.107.162.167   //可以通过service的ip或者名称来访问

//这样的访问是通过coredns来解析的
//查看coredns的地址,就会看到dns的地址
kubectl get svc -n kube-system 

NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP   1d
```



### 动态扩展应用

扩展名为nginx-deploy的deployment的数量为5个

```
kubectl scale --replicas=5 deployment ngnix-deploy  


kubectl get deployment  //查看扩展完的nginx-deploy
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   5         5         5            5           1h
```

当然也可以缩减:

```
kubectl scale --replicas=3 deployment ngnix-deploy  
```

扩展名为nginx-deploy的deployment的数量变成为3个



### 升级pod中的应用

```
kubectl set image deployment nginx-deploy nginx-deploy=haojianxun/test:v2
```

这个镜像自己指定,一般都是在docker hub上的

上述命令中第一个nginx-deploy的意思是  有一个名为nginx-deploy的deployment控制器

第二个nginx-deploy的意思的  在这个控制器下 容器的名称是nginx-deploy
要想查看一个控制器下的容器名称 可以用 kubectl get pods来先看pod , 之后 kubectl describe这个容器  这个时候就会看到容器的名称了

### 查看升级pod的状态

```
kubectl rollout status deployment nginx-deploy
```

### 回滚pod

```
kubectl rollout undo deployment NAME
```

### 外部访问pod

查看当前service

```
kubectl get svc
```

修改相应修改的svc ,以nginx为例

```
kubectl edit svc nginx

//修改其中的type  默认的type为ClusterIP  修改为:  type:NodePort
```

查看修改后的service

```
kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        1d
nginx         NodePort    10.107.162.167   <none>       80:31252/TCP   1h
```

可以看到port多了一个端口 是31252 

之后在外部浏览器输入当前集群中某个的ip:31252 即可访问

结果如下图所示:

![](C:\Users\Administrator\Desktop\pic-user-blog\外部访问pod.png)

