# kubernetes资源指标metrics-server

资源指标: metrics-server

自定义指标: prometheus 



新一代架构

	核心指标流水线: 由kuberlet , metrics-server以及由API server提供的zpi组成, cpu累计使用率 , 内存实时使用率 , pod的资源占用率个容器的磁盘占用率
	
	监控流水线: 用于从系统收集各种指标并提供终端用户 存储系统和HPA 它们包涵核心指标和许多非核心指标 非核心指标不能被k8s所解析



以后用户访问api群组 , 是通过kube-aggegator来访问 , 这个kube-aggegator同时聚合了由kubernetes自己提供的api群组和metrics-server提供的群组



部署metrics-server

*kubernetes-incubato这种类似于开发版* 

```
git clone https://github.com/kubernetes-incubator/metrics-server.git
```

```
cd metrics-server
kubectl create -f deploy/1.8+/
```

查看是否部署好了

```
kubectl get svc -n kube-system
```





我们可以用kubernetes官方提供的yaml来部署

以后部署addon可以直接去github去找 addon的地址是:https://github.com/kubernetes/kubernetes/tree/master/cluster/addons



metrics-server的github地址: https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/metrics-server



先下载文件

```shell
mkdir metrics-server
cd metrics-server
```



```shell
for file in auth-delegator.yaml auth-reader.yaml,metrics-apiservice.yaml metrics-server-deployment.yaml metrics-server-service.yaml resource-reader.yaml; do wget https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/metrics-server/$file;done
```

修改 metrics-server-deployment.yaml其中的内容 

在`spec.containers.command`下修改 修改为

```yaml
spec:
  priorityClassName: system-cluster-critical
  serviceAccountName: metrics-server
  containers:
    - name: metrics-server
      image: k8s.gcr.io/metrics-server-amd64:v0.3.0
      command:
        - /metrics-server
        - --source=kubernetes.summary_api:https://kubernetes.default?kubeletHttps=true&kubeletPort=10250&insecure=true
```

其次修改resource-reader.yaml

修改其中的`rules.resource`  ,增加一行`- nodes/stats`    修改之后是这样的

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:metrics-server
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  - nodes/stats
  - namespaces
  verbs:
  - get
  - list
  - watch
```



```
kubectl apply -f ./
```



查看刚刚部署的metrics-server

```
kubectl proxy --port:8080  //打开一个反代接口
curl http://localhost:8080/api/metrics.k8s.io/v1beta1
curl http://localhost:8080/api/metrics.k8s.io/v1beta1/nodes
```

稍等片刻 ,等数据收集完就可以用`top` 命令查看

```
kubectl top nodes
kubectl top pods -n kube-system
```

