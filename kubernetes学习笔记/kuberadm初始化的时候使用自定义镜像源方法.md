# kuberadm初始化的时候使用自定义镜像源方法

##第一种方法

可以添加环境变量 , 推荐使用阿里云的源

```
export KUBE_REPO_PREFIX="registry-vpc.cn-beijing.aliyuncs.com/bbt_k8s"
export KUBE_ETCD_IMAGE="registry-vpc.cn-beijing.aliyuncs.com/bbt_k8s/etcd-amd64:3.0.17"
```



还有就是pod的基础镜像pause是从出谷歌的gcr里面拉去的 ,我们修改kubelet的启动参数,让他变成从我们指定的镜像仓库拉去

修改`/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` 文件

```
[Service]
Environment="KUBELET_EXTRA_ARGS=--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/muxi/pause-amd64:3.0"
```

## 第二种方法

使用配置文件  在kubeadm初始化的时候指定`--config` 文件来启动指定参数



```yaml
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.12.1
api:
  advertiseAddress: 172.20.0.71
  bindPort: 6443
  controlPlaneEndpoint: ""
imageRepository: k8s.gcr.io
kubeProxy:
  config:
    mode: "ipvs"
    ipvs:
      ExcludeCIDRs: null
      minSyncPeriod: 0s
      scheduler: ""
      syncPeriod: 30s    
kubeletConfiguration:
  baseConfig:
    cgroupDriver: cgroupfs
    clusterDNS:
    - 10.96.0.10
    clusterDomain: cluster.local
    failSwapOn: false
    resolvConf: /etc/resolv.conf
    staticPodPath: /etc/kubernetes/manifests
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
```



或者是就简单的就行 , 比如:

```yaml
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
api:
  advertiseAddress: "10.20.79.10"
networking:
  podSubnet: "10.244.0.0/16"
kubernetesVersion: "v1.11.1"
imageRepository: "registry.cn-hangzhou.aliyuncs.com/google_containers"
```





里面的具体参数换成自己的



不过在v1.11以后官方是推荐使用`v1alpha3`了  , 具体情况看官网 , 具体地址:https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file



使用 [kubeadm config migrate](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-config/) 命令从`v1alpha2` 迁移到`v1alpha3`   

`v1alpha3` 具体语法参见官网 , 具体地址是:https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1alpha3