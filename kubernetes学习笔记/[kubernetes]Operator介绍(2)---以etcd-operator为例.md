# [kubernetes]Operator介绍(2)---以etcd-operator为例

官方学习文档: https://coreos.com/operators/etcd/docs/0.9.0/user/client_service.html

## 部署Etcd Operator

**先将代码拉下来**

```shell
git clone https://github.com/coreos/etcd-operator
```

**备份数据**

```shell
example/rbac/create_role.sh
```

### Install etcd operator

```shell
kubectl create -f example/deployment.yaml
```

**这时它会创建自定义资源 , 我们可以看下这个自定义资源**

```shell
$ kubectl get customresourcedefinitions
NAME                                    KIND
etcdclusters.etcd.database.coreos.com   CustomResourceDefinition.v1beta1.apiextensions.k8s.io
```

**使用describe 命令看到它的细节**

```shell
kubectl describe crd etcdclusters.etcd.database.coreos.com
```

在上述步骤做完以后, 相当于我们在kubernetes集群中添加一个EtcdCluster的自定义资源类型 , 而Etcd Operator就是这个自定义资源类型对应的自定义控制器



**部署好Etcd Operator之后 , 我们可以启动一个集群了**

```shell
$ kubectl create -f example/example-etcd-cluster.yaml
$ kubectl get services
NAME                          CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
example-etcd-cluster          None           <none>        2380/TCP   1m
example-etcd-cluster-client   10.0.222.115   <none>        2379/TCP   1m
```

**从集群中访问这个服务:**

```shell
$ kubectl run --rm -i --tty fun --image quay.io/coreos/etcd --restart=Never -- /bin/sh
/ # ETCDCTL_API=3 etcdctl --endpoints http://example-etcd-cluster-client:2379 put foo bar
OK
(ctrl-D to exit)
```

如果从与etcd群集不同的命名空间访问此服务，请使用完全限定的域名（FQDN）

`http://<cluster-name>-client.<cluster-namespace>.svc.cluster.local:2379`.

这个 example-etcd-cluster.yaml 文件里描述的，是一个 3 个节点的 Etcd 集 , 我们查看这个pod的话发现创建的就是3个



**集群数据备份**

```shell
kubectl create -f example/etcd-backup-operator/deployment.yaml
```

具体aws的备份看官方文档, 里面列出了很详细的步骤

https://coreos.com/operators/etcd/docs/0.9.0/user/walkthrough/backup-operator.html

## 总结

启动是3个etcd , 是因为在[example-etcd-cluster.yaml](https://raw.githubusercontent.com/coreos/etcd-operator/master/example/example-etcd-cluster.yaml)中定义了3个

Operator 的工作原理，实际上是利用了 Kubernetes 的自定义 API 资源（CRD），来描述我们
想要部署的“有状态应用”；然后在自定义控制器里，根据自定义 API 对象的变化，来完成具体的部署和运维工作

所以Etcd Operator和之前自己写的自定义控制器没有什么两样

