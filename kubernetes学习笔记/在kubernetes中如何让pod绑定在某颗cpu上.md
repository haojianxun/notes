# 在kubernetes中如何让pod绑定在某颗cpu上

让pod绑定在cpu上 , 可以大大减少上下文切换的次数 ,容器性能可以得到大幅提升 , 那该如何设置呢

设置的方法很简单

前置条件:

- pod必须是Guaranteed类型的Qos
- pod的CPU资源设置 , requests和limits必须相同

例子:

```yaml
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "2"
      requests:
        memory: "200Mi"
        cpu: "2"

```

这样这个pod会绑定在2颗cpu上 , 至于是那2个 , 是由kubernetes分配的