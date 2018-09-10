# kubernetes资源限制

1颗逻辑的cpu  

1= 1000millicores



QoS类型

- Guaranteed   此类拥有最高优先级

  同时设置了CPU和内存的requests和limits , 并且

  limits.cpu=requests.cpu

  limits.memory=requests.memory

- Burstable  此类拥有中等优先级

  至少一个容器设置cpu或者内存资源的requests属性

- BestEffort

  没有任何一个容器设置了requests或limits属性

vim pod-source.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: default
  labels:
    app: myapp
    tier: frontend
spec:
  containers:
  - name: myapp
    image: ikubernetess/myapp:v1
    resources:
      requestes:
        cpu: "200m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "500Mi"
```

