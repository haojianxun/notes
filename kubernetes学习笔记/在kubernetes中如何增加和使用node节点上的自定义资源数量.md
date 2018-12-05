---
title: 在kubernetes中如何增加和使用node节点上的自定义资源数量
category: kubernetes
---

在node节点上 , 节点上各个类型资源可用量 , 体现在node对象的`status`字段中 

```yaml
apiVersion: v1
kind: Node
metadata:
  name: node-1
...
Status:
  Capacity:
   cpu:  2
   memory:  2049008Ki
.....
```

那么如何增加一个自定义类型资源的数量呢 

## 增加方法

```bash
#启动一个代理
kubectl proxy

#执行pacth操作
curl --header "Content-Type: application/json-patch+json" \
--request PATCH \
--data '[{"op": "add", "path":"/status/capacity/nvidia.com/gpu", "value": "1"}]' \
http://localhost:8001/api/v1/nodes/<YOUR-NODE-NAME>/status

```

## 使用方法

在pod中声明

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vector-add
spec:
  restartPolicy: OnFailure
  containers:
    - name: cuda-vector-add
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1

```

在字段`spec.containers.resources.limits` 下增加即可