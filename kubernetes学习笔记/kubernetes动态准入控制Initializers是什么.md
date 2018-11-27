# kubernetes动态准入控制Initializers是什么

在kubernetes中 , 当一个 Pod 或者任何一个 API 对象被提交给 APISServer 之后，总有一些“初始化”性质的工作需要在它们被处理 , 比如为所有pod加上某些标签 , 在实际中的应用的话, 可以体现在以下方面

- 为即将创建的每个或者某些特定的pod都自动插入一个SideCar容器
- 为即将创建的每个或者某些特定的pod都自动添加一些环境变量或者设置
- 检测secret长度 , 如果不满足就拒绝

## Initializers的工作原理和使用方法

将一个编写好的 Initializer，作为一个 Pod 部署在kubernetes中, 可以用pod或者deployment部署  , 定义如下:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  initializers:
    pending: []
  labels:
    app: envoy-initializer
  name: envoy-initializer
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: envoy-initializer
      name: envoy-initializer
    spec:
      containers:
        - name: envoy-initializer
          image: envoy-initializer:0.0.1
          imagePullPolicy: Always
          args:
            - "-annotation=initializer.kubernetes.io/envoy"
            - "-require-annotation=true"
```

*部署envoy-initializer时，千万要注意设置metadata.initializers.pending为空，防止envoy-initializer的部署被自己stuck了。*



或者

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: envoy-initializer
  name: envoy-initializer
spec:
  containers:
    - name: envoy-initializer
      image: envoy-initializer:0.0.1
      imagePullPolicy: Always
```

然后创建你的`initializerConfiguration`API Object , 可以指定要对什么样的资源进行Initialize操作 , 比如

```yaml
apiVersion: admissionregistration.k8s.io/v1alpha1
kind: InitializerConfiguration
metadata:
  name: envoy-config
initializers:
  // 这个名字必须至少包括两个 "."
  - name: envoy.initializer.kubernetes.io
    rules:
      - apiGroups:
          - "" // 前面说过， "" 就是 core API Group 的意思
        apiVersions:
          - v1
        resources:
          - pods
```

这就是要对所有的pod执行这个Initialize操作

一旦InitializerConfiguration被创建 , 就会对在所有的pod上加上一个metadata信息`metadata.initializers.pending`

```yaml
apiVersion: v1
kind: Pod
metadata:
  initializers:
    pending:
      - name: envoy.initializer.kubernetes.io
  name: myapp-pod
  labels:
    app: myapp
...
```

这个metadata信息是判断这个pod有没有执行过initalizer的依据 , 如果初始化完了之后要把`metadata.initializers.pending`的标志去掉

如果要是想指定某个initializer的话, 可以在部署pod的时候声明

```yaml
apiVersion: v1
kind: Pod
metadata
  annotations:
    "initializer.kubernetes.io/envoy": "true"
    ...
```

这样就会用到我们之前定义的envoy-initializer 了



