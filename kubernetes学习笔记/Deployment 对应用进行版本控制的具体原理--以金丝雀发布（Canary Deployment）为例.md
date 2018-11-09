# Deployment 对应用进行版本控制的具体原理--以金丝雀发布（Canary Deployment）为例

在所有 API 对象的 Metadata 里，都有一个字段叫作 ownerReference，用于保存当前这个 API 对象的拥有者（Owner）的信息

对于一个 Deployment 所管理的 Pod，它的 ownerReference 是谁？

答案就是：ReplicaSet

**Deployment 控制 ReplicaSet（版本），ReplicaSet 控制 Pod（副本数）**



举例说明:

首先，我们来创建这个 nginx-deployment：

```bash
$ kubectl create -f nginx-deployment.yaml --record
```

查看一下这个 Deployment 所控制的 ReplicaSet：

```
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-3167673210   3         3         3       20s
```

在用户提交了一个 Deployment 对象后，Deployment Controller 就会立即创建一个 Pod 副本个数为 3 的 ReplicaSet。这个 ReplicaSet 的名字，则是由 Deployment 的名字和一个随机字符串共同组成。

这个随机字符串叫作 pod-template-hash，在我们这个例子里就是：3167673210。ReplicaSet 会把这个随机字符串加在它所控制的所有 Pod 的标签里，从而保证这些 Pod 不会与集群里的其他 Pod 混淆

下面使用` kubectl edit`来升级这个pod的镜像版本

```
kubectl edit deployment/nginx-deployment  //将其中的image字段的版本修改一下, 改成高于当前的
```

kubectl edit 是把 API 对象的内容下载到了本地文件，修改完成后再提交上去

保存退出会触发滚动更新

查看 Deployment 的 Events，看到这个“滚动更新”的流程

```
kubectl describe deployment nginx-deployment
```

会看到其中的event事件里面有这样的信息

```
Scaled up replica set nginx-deployment-1764197365 to 1
Scaled down replica set nginx-deployment-3167673210 to 2
```

可以看到是创建出了2个`ReplicaSet`  , 之后他们再交替更新这个image的版本 , Deployment Controller 会确保在任何时间窗口内，只有指定比例的新 Pod 被创建出来。这两个比例的值都是可以配置的，默认都是 DESIRED 值的 25%  , 具体的设置可以在创建Deployment 对象的yaml文件中指定, 相关字段是`RollingUpdateStrategy` 

那如果要回滚呢?

执行命令

```bash
$ kubectl rollout undo deployment/nginx-deployment
deployment.extensions/nginx-deploymen
```

Deployment 的控制器，让这个旧 ReplicaSet（hash=1764197365）再次“扩展”成 3 个 Pod，而让新的 ReplicaSet（hash=2156724341）重新“收缩”到 0 个 Pod。

如果想回到更早之前的版本 , 可以执行命令

```
 kubectl rollout history
```

要查看更早的 可以在创建这个 Deployment 的时候，指定了–record 参数



如何控制这些“历史”ReplicaSet 的数量呢？

很简单，Deployment 对象有一个字段，叫作 `spec.revisionHistoryLimit`，就是 Kubernetes 为 Deployment 保留的“历史版本”个数。所以，如果把它设置为 0，你就再也不能做回滚操作了。