# kubeadm init可能出现的问题

先去看kubeadm的帮助信息

```
kubeadm --help    //这样会列出许多帮助信息
```

要想使用flannel网络,在初始化的时候就先指定网络

```
kubeadm init --kubernetes-version=v1.11.1 --pod-network-cidr=10.244.0.0/16
```





如果出现[ERROR Swap]

```
vim /etc/sysconfig/kubelet

KUBELET_EXTRA_ARGS="--fail-swap-on=false"

编辑完成之后在kubeadm init后面加个参数

kubeadm init --kubernetes-version=v1.11.1 --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors Swap
```



