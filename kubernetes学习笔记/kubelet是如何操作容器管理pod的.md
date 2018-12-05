---
title: kubelet是如何操作容器管理pod的
---

`kubelet`是kubernetes中重要的一个组件 , 它运行在每个node上 

## kubelet工作原理

![](https://ws1.sinaimg.cn/large/6450e885gy1fxvvkrtr10j21110qin3w.jpg)



kubelet的核心便是**SyncLoop**

触发SyncLoop的条件:

- pod更新事件
- pod生命周期变化
- kubelet本身设置的执行周期
- 定时清理事件



### SyncLoop循环

kubelet在启动的时候, 首先第一件事情会做的就是先去设置listers , 就是注册它所关心的informer

之后会依次启动其他的:

- diskSpaceManage
- oomWatcher
- initNetworkPlugin
- chooseRuntime(dockershim , remote)
- NewGenericPLEG
- NewContainerGC
- AddPodAdmitHandler
- Eviction

同时它还管理其它的小循环 , 这些小循环便是一下Manger循环 , 比如上图的NodeStatus , NetworkStatus等等 , 这些manger是完成kubelet的某项具体功能  , 比如nodestatus就是收集node信息 , 之后通过heartbeat发送到API Server



在kubernetes集群中 , pod调度完成之后 , pod会被选到某个node上 , 这个时候就会触发kubelet在控制循环里面注册的handler , 就是上图中出现的**HandlePods** 部分 , 这个时候kubelet会在自己内存中检查这个pod的状态 , 到底是更新操作还是新建pod操作 , 从而触发这个handler里面的对应事件 , 

如何触发这个handler里面对应是事件呢

如上图 , kubelet会调用一个叫`Pod Update Worker` 的goroutine来单独处理这个pod

 如果这个pod是add的话 , 就会:

- 生成pod状态
- 检查volume状态
- 通过CRI(容器运行时接口)来启动一个容器

### CRI(容器运行时接口)

容器运行时插件（Container Runtime Interface，简称 CRI）是 Kubernetes v1.5 引入的容器运行时接口，它将 Kubelet 与容器运行时解耦，将原来完全面向 Pod 级别的内部接口拆分成面向 Sandbox 和 Container 的 gRPC 接口，并将镜像管理和容器管理分离到不同的服务。

Kubelet 作为 CRI 的客户端，而容器运行时则需要实现 CRI 的服务端（即 gRPC server，通常称为 CRI shim）。容器运行时在启动 gRPC server 时需要监听在本地的 Unix Socket （Windows 使用 tcp 格式）。

简单来说就是实现了统一的接口, 各种容器都能接入这个接口 , 从而实现统一的容器管理 , kubel把相关信息发送给CRI  , CRI转换这种信息 , 让下面的各种不同容器引擎能够创建出容器

目前基于 CRI 容器引擎已经比较丰富了，包括

- Docker: 核心代码依然保留在 kubelet 内部（[pkg/kubelet/dockershim](https://github.com/kubernetes/kubernetes/tree/master/pkg/kubelet/dockershim)），是最稳定和特性支持最好的运行时
- OCI 容器运行时：
  - 社区有两个实现
    - [Containerd](https://github.com/containerd/cri)，支持 kubernetes v1.7+
    - [CRI-O](https://github.com/kubernetes-incubator/cri-o)，支持 Kubernetes v1.6+
  - 支持的 OCI 容器引擎包括
    - [runc](https://github.com/opencontainers/runc)：OCI 标准容器引擎
    - [gVisor](https://github.com/google/gvisor)：谷歌开源的基于用户空间内核的沙箱容器引擎
    - [Clear Containers](https://github.com/clearcontainers/runtime)：Intel 开源的基于虚拟化的容器引擎
    - [Kata Containers](https://github.com/kata-containers/runtime)：基于虚拟化的容器引擎，由 Clear Containers 和 runV 合并而来
- [PouchContainer](https://github.com/alibaba/pouch)：阿里巴巴开源的胖容器引擎
- [Frakti](https://github.com/kubernetes/frakti)：支持 Kubernetes v1.6+，提供基于 hypervisor 和 docker 的混合运行时，适用于运行非可信应用，如多租户和 NFV 等场景
- [Rktlet](https://github.com/kubernetes-incubator/rktlet)：支持 [rkt](https://github.com/rkt/rkt) 容器引擎（rknetes 代码已在 v1.10 中弃用）
- [Virtlet](https://github.com/Mirantis/virtlet)：Mirantis 开源的虚拟机容器引擎，直接管理 libvirt 虚拟机，镜像须是 qcow2 格式
- [Infranetes](https://github.com/apporbit/infranetes)：直接管理 IaaS 平台虚拟机，如 GCE、AWS 等

