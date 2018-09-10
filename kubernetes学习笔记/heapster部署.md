# heapster部署

==**heapster在1.11版本已经不在支持 heapster   之前的还可以用 之后的版本将考虑新的, 以后的监控方案是Prometheus**==

具体变更说明的地址: https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.11.md#sig-instrumentation-1



>Heapster是容器集群监控和性能分析工具，天然的支持Kubernetes和CoreOS。
>
>Kubernetes有个出名的监控agent—cAdvisor。在每个kubernetes Node上都会运行cAdvisor，它会收集本机以及容器的监控数据(cpu,memory,filesystem,network,uptime)。在较新的版本中，K8S已经将cAdvisor功能集成到kubelet组件中。每个Node节点可以直接进行web访问。
>
>Heapster是一个收集者，Heapster可以收集Node节点上的cAdvisor数据，将每个Node上的cAdvisor的数据进行汇总，还可以按照kubernetes的资源类型来集合资源，比如Pod、Namespace，可以分别获取它们的CPU、内存、网络和磁盘的metric。默认的metric数据聚合时间间隔是1分钟。还可以把数据导入到第三方工具(如InfluxDB)。
>
>Kubernetes原生dashboard的监控图表信息来自heapster。在Horizontal Pod Autoscaling中也用到了Heapster，HPA将Heapster作为Resource Metrics API，向其获取metric



不过也可以试着部署下

## 部署influxdb

```
wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/influxdb.yaml
```



可以直接应用 , 也可以自己修改下

vim influxdb.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: monitoring-influxdb
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      task: monitoring
      k8s-app: influxdb
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: influxdb
    spec:
      containers:
      - name: influxdb
        image: k8s.gcr.io/heapster-influxdb-amd64:v1.5.2
        volumeMounts:
        - mountPath: /data
          name: influxdb-storage
      volumes:
      - name: influxdb-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: monitoring-influxdb
  name: monitoring-influxdb
  namespace: kube-system
spec:
  ports:
  - port: 8086
    targetPort: 8086
  selector:
    k8s-app: influxdb
```



```
kubectl apply -f influxdb.yaml
```





## 部署rbac

```
wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/rbac/heapster-rbac.yaml
```

```
kubectl apply -f heapster-rbac.yaml
```



## 部署heapster

```
wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/heapster.yaml
```



vim heapster.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: heapster
  namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: heapster
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      task: monitoring
      k8s-app: heapster
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: heapster
    spec:
      serviceAccountName: heapster
      containers:
      - name: heapster
        image: k8s.gcr.io/heapster-amd64:v1.5.4
        imagePullPolicy: IfNotPresent
        command:
        - /heapster
        - --source=kubernetes:https://kubernetes.default
        - --sink=influxdb:http://monitoring-influxdb.kube-system.svc:8086
---
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: Heapster
  name: heapster
  namespace: kube-system
spec:
  ports:
  - port: 80
    targetPort: 8082
  tpye: NodePort
  selector:
    k8s-app: heapster
```

说明: 如果要想外部访问的话 可以加上`tpye: NodePort`    这个文件也可以直接应用 , 也可以自己修改下 修改的地方是: apiVersion改成`apiVersion: apps/v1`   在`spec`下增加

```
selector:
    matchLabels:
      task: monitoring
      k8s-app: heapster
```

其余可以不变 , 修改完成应用这个yaml文件

```
kubectl apply -f heapster.yaml
```

查看这个服务

```
kubectl get svc -n kube-system
```



部署grafana

```
wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/grafana.yaml
```

 vim grafana.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: monitoring-grafana
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      task: monitoring
      k8s-app: grafana
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: grafana
    spec:
      containers:
      - name: grafana
        image: k8s.gcr.io/heapster-grafana-amd64:v5.0.4
        ports:
        - containerPort: 3000
          protocol: TCP
          
........
只是贴了前半段 ,因为我们修改的只有前面的内容而已 , 修改的内容也是之前一样 是为了适应这个v1.11.1  把apiVersion修改了  该成现在v1.11.1用的apps/v1   在spec下增加selector等内容   如果想外部访问  可以在最后是service服务下 增加 type:NodePort
 
```

应用这个yaml文件

```
kubectl apply -f grafana.yaml
```

查看这个svc 可以看到用于外部访问的port 如果你修改了之前是service服务 增加了type:NodePort的话

```
kubectl get svc -n kube-system
```

