# kubernetes中的"控制器"模型-以Deployment的实现为例

kubernetes中的控制器就是运行在master上的**kube-controller-manager** , 这个组件，就是一系列控制器的集合

在github上kubernetes项目里(具体地址为https://github.com/kubernetes/kubernetes/tree/master/pkg/controller)pkg/controller 目录下 , 可以看到有许多的子文件夹 , 这些都是一些控制器模型 , 包括我们常用的Deployment等等 , 这每一个子文件夹就是一个控制器 , 负责某个编排功能



控制器的逻辑大概如下:

```go
for {
  实际状态 := 获取集群中对象 X 的实际状态（Actual State）
  期望状态 := 获取集群中对象 X 的期望状态（Desired State）
  if 实际状态 == 期望状态{
    什么都不做
  } else {
    执行编排动作，将实际状态调整为期望状态
  }
}
```

其中实际状态的来源是kubelet 通过心跳汇报的容器状态和节点状态，或者监控系统中保存的应用监控数据，或者控制器主动收集的它自己感兴趣的信息



接下来，以 Deployment 为例，我和你简单描述一下它对控制器模型的实现：

1. Deployment 控制器从 Etcd 中获取到所有携带了标签的 Pod，然后统计它们的数量，这就是实际状态；
2. Deployment 对象的 Replicas 字段的值就是期望状态；
3. Deployment 控制器将两个状态做比较，然后根据比较结果，确定是创建 Pod，还是删除已有的 Pod

可以看到，一个 Kubernetes 对象的主要编排逻辑，实际上是在第三步的“对比”阶段完成的。

这个操作，通常被叫作调谐（Reconcile）。这个调谐的过程，则被称作“Reconcile Loop”（调谐循环）或者“Sync Loop”（同步循环）



在一个`kind: Deployment`的编排yaml文件中 `template 字段` 以上 , 就是控制器对象的定义 , `template 字段`就是被控制对象的定义 