# kubernetes资源清单

### 查看帮助

```
kubectl explain pods   //如何定义pod的帮助
kubectl explain pods.metadata  //查看其中matadata字段的定义方法
kubectl explain pods.spec.containers  //查看spec下的containers字段的定义方法
```



### 如何查看一个已经运行的pod的资源清单

1. 先查看pod 

```
kubectl get pods
NAME                            READY     STATUS      RESTARTS   AGE
nginx-deploy-59c86578c8-8x2jc   1/1       Running     0          50m
nginx-deploy-59c86578c8-ml57k   1/1       Running     0          50m
nginx-deploy-59c86578c8-pmbkm   1/1       Running     0          50m
nginx-deploy-59c86578c8-pqh8m   1/1       Running     0          1h
nginx-deploy-59c86578c8-pvh79   1/1       Running     0          50m
test                            0/1       Completed   0          1h
```

2. 选择其中某一个来查看,并且以yaml格式来输出

```
#kubectl get pod nginx-deploy-59c86578c8-ml57k -o yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: 2018-08-19T06:51:28Z
  generateName: nginx-deploy-59c86578c8-
  labels:
    pod-template-hash: "1574213474"
    run: nginx-deploy
  name: nginx-deploy-59c86578c8-ml57k
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: nginx-deploy-59c86578c8
    uid: 5966e9a9-a373-11e8-aca9-000c29c844cd
  resourceVersion: "15686"
  selfLink: /api/v1/namespaces/default/pods/nginx-deploy-59c86578c8-ml57k
  uid: 4c24b125-a37c-11e8-aca9-000c29c844cd
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx-deploy
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-vwdjg
      readOnly: true
  dnsPolicy: ClusterFirst
  nodeName: localhost.localdomain
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-vwdjg
    secret:
      defaultMode: 420
      secretName: default-token-vwdjg
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: 2018-08-19T06:51:28Z
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: 2018-08-19T06:51:37Z
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: null
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: 2018-08-19T06:51:28Z
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://7a424a6941050ecdc61c23dee86e7a339304a028e096414a1cd69608084f9b80
    image: nginx:latest
    imageID: docker-pullable://nginx@sha256:d85914d547a6c92faa39ce7058bd7529baacab7e0cd4255442b04577c4d1f424
    lastState: {}
    name: nginx-deploy
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: 2018-08-19T06:51:36Z
  hostIP: 192.168.200.139
  phase: Running
  podIP: 10.244.1.7
  qosClass: BestEffort
  startTime: 2018-08-19T06:51:28Z
```





apiserver仅接受JSON格式的资源定义, 当我们提供YAML格式的配置清单 , apiserver可以自动将其转换为JSON格式 , 而后在提交执行

### apiVersion

使用命令来查看当前apiserver支持哪些版本

```
#kubectl api-versions
admissionregistration.k8s.io/v1beta1
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1
apiregistration.k8s.io/v1beta1
apps/v1
apps/v1beta1
apps/v1beta2
authentication.k8s.io/v1
authentication.k8s.io/v1beta1
authorization.k8s.io/v1
authorization.k8s.io/v1beta1
autoscaling/v1
autoscaling/v2beta1
batch/v1
batch/v1beta1
certificates.k8s.io/v1beta1
events.k8s.io/v1beta1
extensions/v1beta1
networking.k8s.io/v1
policy/v1beta1
rbac.authorization.k8s.io/v1
rbac.authorization.k8s.io/v1beta1
scheduling.k8s.io/v1beta1
storage.k8s.io/v1
storage.k8s.io/v1beta1
v1
```

可以看到最后一个就是我们经常看到的v1版本 , v1版本不指明的话 一般都是指的是核心组v1  即core组

### kind

资源类别

### metadata

元数据

```
metadata:
	name:NAME //名称必须唯一
	namespace:
	labels:
	annotations
```

### spec

期望状态 disired state

```
spec:
	
```

status

当前状态 current state  此字段由kubernetes集群自己维护 用户不可定义

