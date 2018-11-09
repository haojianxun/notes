# 在kubernetes中**如何在指定的 Node 上创建新 Pod 呢**



 ## 使用nodeAffinity字段

Kubernetes 项目里，在`Pod API` 对象中的`nodeSelector`字段可以创建 , 不过 , `nodeSelector` 其实已经是一个将要被废弃的字段了。因为，现在有了一个新的、功能更完善的字段可以代替它，即：`nodeAffinity`

我来举个例子：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: metadata.name
            operator: In
            values:
            - node-geektime
```

在这个 Pod 里，我声明了一个 `spec.affinity `字段，然后定义了一个 `nodeAffinity`。其中，`spec.affinity` 字段，是 Pod 里跟调度相关的一个字段。关于它的完整内容，我会在讲解调度策略的时候再详细阐述。

而在这里，我定义的` nodeAffinity` 的含义是：

1. `requiredDuringSchedulingIgnoredDuringExecution`：它的意思是说，这个 `nodeAffinity` 必须在每次调度的时候予以考虑。同时，这也意味着你可以设置在某些情况下不考虑这个 nodeAffinity；
2. 这个 Pod，将来只允许运行在“`metadata.name`”是“node-geektime”的节点上。

在这里，你应该注意到 `nodeAffinity` 的定义，可以支持更加丰富的语法，比如 `operator: In`（即：部分匹配；如果你定义 `operator: Equal`，就是完全匹配），这也正是 `nodeAffinity` 会取代 `nodeSelector `的原因之一。



## 使用Taint

可以在部署的时候给node打上taint , 之后我们在容忍它

比如:

```yaml
...
template:
    metadata:
      labels:
        name: network-plugin-agent
    spec:
      tolerations:
      - key: node.kubernetes.io/network-unavailable
        operator: Exists
        effect: NoSchedule
```







