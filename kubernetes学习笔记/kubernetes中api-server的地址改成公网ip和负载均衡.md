# kubernetes中api-server的地址改成公网ip和负载均衡

通常情况下，我们的集群都是内网集群，只暴露一个公网ip，我希望通过本地访问这个集群，这时候只能通过公网ip访问。 这时候应该怎么配置呢？我们一步一步的思考，刚才我们说了，Kubeadm在初始化的时候会为管理员生成一个 Kubeconfig文件，把它下载下来 是不是就可以？事实证明这样不行， 因为这个集群是内网集群，Kubeconfig文件 中APIServer的地址是内网ip。所以无法访问到。那么我把内网ip改成 APIServer公网ip是不是就可以了 呢？经过实验发现也是不可以的，会报 出认证无法通过的错误。为什么认证无法通过？这要回顾我们刚刚讲过的服务端认证的流程，刚才说一个客户端想跟APIServer访问 时，要拿服务器证书，进行解析，去看APIServer都有哪些别名，再把客户端访问APIServer时所采用的ip 或者域名和别名相比较，看是否已经涵盖在别名里面了。如果涵盖进去，我就认为这个server是可认证的。由于我们在这个场景部署出来的集群Kubeadm生成 APIServer证书不会把公网ip写到证书里，所以 导致用公网ip访问不通过验证。解决方案很简单，把公网ip签到证书里面就可以，所以这个yaml文件也给我们提供了这样一个选项，叫apiServerCertSANs这个选项，只要把公网IP写到这里，再启动这个集群的时候，这个证书就可以有这个公网ip，我就可以使用刚才我说的流程，把文件下载下来把APIServer的地址改成公网ip，然后就可以访问了。这是我工作当中非常常见的需求，这样会让你的开发工作更加方便一些。



### Api-Server负载均衡

配置负载均衡器对kube-apiserver进行负载均衡，可采用DNS轮询解析或者Haproxy（Nginx）反向代理实现负载均衡。

本文采用DNS轮询解析实现简单的负载均衡，在Dns01,Dns02节点上部署DNS。

1、修改`/etc/hosts`文件，添加域名解析

```
172.16.2.1 api.me
172.16.2.2 api.me
172.16.2.3 api.me
```

2、docker-compose部署dnsmasq服务：

```yaml
version: "3"
services:
  dnsmasq:
    image: cloudnil/dnsmasq:2.76
    command: -q --log-facility=- --all-servers
    network_mode: "host"
    cap_add:
    - NET_ADMIN
    restart: always
    stdin_open: true
    tty: true
```

3、除了部署dnsmasq服务的其他所有节点上(包括Master和Node)，配置DNS

```
cat <<EOF >/etc/resolvconf/resolv.conf.d/base
nameserver 172.16.2.251
nameserver 172.16.2.252
EOF
```

记得重启解析服务`resolvconf`：

```
/etc/init.d/resolvconf restart
```



### 安装master节点

kubeadm配置文件kubeadm-config.yml：

```yaml
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.2
imageRepository: cloudnil
apiServerCertSANs:
- api.me
api:
  controlPlaneEndpoint: api.me:6443
etcd:
  external:
    endpoints:
    - http://172.16.2.1:2379
    - http://172.16.2.2:2379
    - http://172.16.2.3:2379
networking:
  podSubnet: "10.68.0.0/16"
kubeletConfiguration:
  baseConfig:
    systemReserved: 
      cpu: "0.25"
      memory: "128Mi"
kubeProxy:
  config:
    ipvs:
      minSyncPeriod: 1s
      scheduler: rr
      syncPeriod: 10s
    mode: ipvs
```

master01初始化指令：

```
kubeadm init --config kubeadm-config.yml
```

> PS：`token`是使用指令`kubeadm token generate`生成的，执行过程如有异常，用命令`kubeadm reset`初始化后重试，生成的`token`有效时间为24小时，超过24小时后需要重新使用命令`kubeadm token create`创建新的`token`。

复制`/etc/kubernetes/pki`下的以下文件到Master02和Master03对应目录，`.ssh/cloudnil.pem`是方便节点之间访问的证书，可以使用`ssh-keygen`生成，具体使用不详细阐述，网络上文章很多，也可以直接使用账号密码。

```
USER=root
CONTROL_PLANE_IPS="172.16.2.2 172.16.2.3"
for host in ${CONTROL_PLANE_IPS}; do
    scp -i .ssh/cloudnil.pem /etc/kubernetes/pki/ca.crt "${USER}"@$host:/etc/kubernetes/pki
    scp -i .ssh/cloudnil.pem /etc/kubernetes/pki/ca.key "${USER}"@$host:/etc/kubernetes/pki
    scp -i .ssh/cloudnil.pem /etc/kubernetes/pki/sa.key "${USER}"@$host:/etc/kubernetes/pki
    scp -i .ssh/cloudnil.pem /etc/kubernetes/pki/sa.pub "${USER}"@$host:/etc/kubernetes/pki
    scp -i .ssh/cloudnil.pem /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:/etc/kubernetes/pki
    scp -i .ssh/cloudnil.pem /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:/etc/kubernetes/pki
done
```

在master02，master03上分别用同样配置文件`kubeadm-config.yml`的执行：

```
kubeadm init --config kubeadm-config.yml
```