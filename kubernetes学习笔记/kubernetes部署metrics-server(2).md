# kubernetes部署metrics-server

官方部署文件地址: https://github.com/kubernetes-incubator/metrics-server/tree/master/deploy/1.8%2B

下载所有yaml到目录`metrics-server`，修改`metrics-server-deployment.yaml`：

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: metrics-server
  namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    k8s-app: metrics-server
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      name: metrics-server
      labels:
        k8s-app: metrics-server
    spec:
      serviceAccountName: metrics-server
      volumes:
      # mount in tmp so we can safely use from-scratch images and/or read-only containers
      - name: tmp-dir
        emptyDir: {}
      containers:
      - name: metrics-server
        image: rancher/metrics-server-amd64:v0.3.1
        imagePullPolicy: Always
        command:
        - /metrics-server
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
        volumeMounts:
        - name: tmp-dir
          mountPath: /tmp
```

执行部署命令：

```
kubectl apply -f metrics-server/
```

查看监控数据：

```
root@master01:~# kubectl top nodes
NAME       CPU(cores)   CPU%      MEMORY(bytes)  MEMORY%   
master01   465m         26%       295Mi          18%       
master02   408m         23%       229Mi          13%       
master03   440m         25%       221Mi          17%       
node01     376m         10%       1047Mi         13%       
node02     196m         5%        976Mi          10%       
node03     206m         5%        907Mi          12%
```