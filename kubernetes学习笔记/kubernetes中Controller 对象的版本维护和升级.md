# kubernetes中Controller 对象的版本维护和升级

在 Kubernetes 项目中，任何你觉得需要记录下来的状态，都可以被用 API 对象的方式实现。当然，“版本”也不例外。

Kubernetes v1.7 之后添加了一个 API 对象，名叫**ControllerRevision**，专门用来记录某种 Controller 对象的版本

 比如:

某个对象(DeamonSet)进行了滚动升级 (假设是部署fluentd-elasticsearch)

对fluentd-elasticsearch进行镜像升级 , 升到v2.2.0

```bash
kubectl set image ds/fluentd-elasticsearch fluentd-elasticsearch=k8s.gcr.io/fluentd-elasticsearch:v2.2.0 --record -n=kube-system
```



使用 kubectl rollout status 命令看到这个“滚动更新”的过程

```
kubectl rollout status ds/fluentd-elasticsearch -n kube-system
```

查看滚动升级历史

```
kubectl rollout history daemonset fluentd-elasticsearch -n kube-system
```



查看这个Controller 对象的对于版本

```
kubectl get controllerrevision -n kube-system -l name=fluentd-elasticsearch
```

使用 kubectl describe 查看这个 ControllerRevision 对象

```
kubectl describe controllerrevision fluentd-elasticsearch-64dc6799c9 -n kube-system
```

尝试将这个 DaemonSet 回滚到 Revision=1 时的状态

```
kubectl rollout undo daemonset fluentd-elasticsearch --to-revision=1 -n kube-system
```



这个 kubectl rollout undo 操作，实际上相当于读取到了 Revision=1 的 ControllerRevision 对象保存的 Data 字段。而这个 Data 字段里保存的信息，就是 Revision=1 时这个 DaemonSet 的完整 API 对象。

所以，现在 DaemonSet Controller 就可以使用这个历史 API 对象，对现有的 DaemonSet 做一次 PATCH 操作（等价于执行一次 kubectl apply -f “旧的 DaemonSet 对象”），从而把这个 DaemonSet“更新”到一个旧版本。

这也是为什么，在执行完这次回滚完成后，你会发现，DaemonSet 的 Revision 并不会从 Revision=2 退回到 1，而是会增加成 Revision=3。这是因为，一个新的 ControllerRevision 被创建了出来