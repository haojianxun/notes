# kubernetes StatefulSet

[TOC]

**statefulset的特征**

- 稳定唯一的网络标识符
- 稳定持久的存储
- 有序平滑的部署和扩展
- 有序平滑的删除终止
- 有序的滚动更新



**statefulset是必要三个组件**

- headless service
- statefulset
- volumeClaimTemplate



pod之间互相解析

```
pod_name.service_name.ns_name.svc.cluster.local

例如:
myapp-0.myapp.default.svc.cluster.local
```



## 实验开始

---

先创建pv , 可以使用之前的pv-demo.yaml文件

之后 创建一个statefulset



vim stateful-demo.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: myapp-pod
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: myapp
spec:
  serviceName: myapp
  replicas: 3
  selector:
    matchLabels:
      app: myapp-pod
  template:
    metadata:
      labels:
        app: myapp-pod
    spec:
      containers:
        name: myapp
        image: ikubernetes/myapp:v1
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: myappdata
          mountPath: /usr/share/nginx/html
  volumeClaimTemolates:
    metadata:
      name: myappdata
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "gluster-dynamic"
      resources:
        requests:
          storage: 2Gi
```



### 扩展

```
kubectl scale sts myapp --replicas=5
kubectl patch sts myapp -p "{"spec":{"repplicas":5}}"
```

### 升级

查看帮助

```
kubectl explain sts.spc..updateStrategy.rollingUpdate
```



**设定升级策略**

```
kubectl patch sts myapp -p '{"spec":{"updateStrategy":{"rollingUpdate":{"partition":4}}}}'
```

partition: N  分区为N 则表明大于等于N的分区才会被更新



升级指定镜像

```
kubectl set image sts/myapp myapp=ikubernetes/myapp:v2
```

查看升级后pod所用的镜像

```
kubectl get pods myapp-4 -o yaml   
```

在输出的yaml格式内容中科院看到所用的镜像