# 为什么kubernetes集群上master不会被调度

```
查看master节点信息
kubectl describe node master

可以看到master的tain信息   显示的是NoSchedule
Taints:             node-role.kubernetes.io/master:NoSchedule
```

