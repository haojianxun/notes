# kubernetes StatefulSet

[TOC]

## **statefulset的特征**

- 稳定唯一的网络标识符
- 稳定持久的存储
- 有序平滑的部署和扩展
- 有序平滑的删除终止
- 有序的滚动更新

pod的名字必须持久有效 , 比如一个pod销毁了 ,  再次重新创建 , 还必须还是原来的名字 , 因为statefulset是根据pod名称有次序的进行更新等操作 , 次序还不能乱 , 比如创建的时候是1-9 那么更新的时候就是9-1 , 那么如何保证持久有效呢 ,得使用headless服务 直达ip地址 还得给pod配置一个唯一的名称, 每个节点得有自己专用的存储 , 使用`volumeClaimTemolates` 来生成一个专有的pvc 之后在绑定一个专有的pv



**statefulset的必要三个组件**

- headless service
- statefulset
- volumeClaimTemplate



**唯一稳定的网络标识**

pod是名字叫做网络标识(hostname) , 由系统生成 , 名字格式为: STATEFULSET-NAME-#   #的取值范围是0-期望的副本数-1 



pod之间互相解析

```
服务间是通过Pod域名来通信而非Pod IP，因为当Pod所在Node发生故障时，Pod会被飘移到其它Node上，Pod IP会发生变化，但是Pod域名不会有变化
pod的dns域名格式为:
pod_name.service_name.ns_name.svc.cluster.local

例如:
进入pod容器内部使用命令
nslookup myapp-0.myapp.default.svc.cluster.local
```

**持久有效的存储**

是指pv , 并且pv不会随着pod的销毁或者缩容而消失

**有序的部署拓展删除更新**

pod都是有序的 , 可以从pod的名字就可以看出 , pod的部署是从0-(N-1) 的顺序 , 删除是从(N-1)-0有序进行 更新也是从(N-1)-0的顺序更新 , 更新也可以指定策略来进行, 次序进行一般是在上一个pod好了之后才进行  , 也可以调整策略`.spec.podManagementPolicy` 

pod排序策略:

- **OrderedReady**：上述的启停顺序，默认设置
- **Parallel**：告诉StatefulSet控制器并行启动或终止所有Pod，并且在启动或终止另一个Pod之前不等待前一个Pod变为Running and Ready或完全终止





## pv回收策略

当删除statefulset里面的pod或者进行缩容的时候 , 并不会主动删除pv , 而且这个pv也不会被其他pvc所绑定 , 当前pv的回收策略(**RECLAIM POLICY**)有三种

- **Retain** 这个是默认的 , 意思是需要手动清理
- **Recycle**，等同于rm -rf /thevolume/*
- **Delete**





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



### **设定升级策略**

升级策略可以通过命令`kubectl explain sts.spec.updateStrategy` 查看

updateStrategy

- rollingUpdate  支持分区更新 , 分区更新默认值是0



通过打补丁的方法更新:

```
kubectl patch sts myapp -p '{"spec":{"updateStrategy":{"rollingUpdate":{"partition":4}}}}'
```

partition: N  分区为N 则表明大于等于N的分区才会被更新



### 升级指定镜像

```
kubectl set image sts/myapp myapp=ikubernetes/myapp:v2
```

查看升级后pod所用的镜像

```
kubectl get pods myapp-4 -o yaml   
```

在输出的yaml格式内容中科院看到所用的镜像