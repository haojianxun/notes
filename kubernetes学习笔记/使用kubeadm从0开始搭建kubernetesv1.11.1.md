# 使用kubeadm从0开始搭建kubernetes:v1.11.1

## 准备阶段

### 前期准备

准备至少2台机器 (我的是CentOS7 )

| ip              | hostname | role       |
| --------------- | -------- | ---------- |
| 192.168.200.139 | master   | 主节点     |
| 192.168.200.140 | node01   | 集群从节点 |



配置hosts文件

```
vim /etc/hosts

192.168.200.139	master
192.168.200.140	node01
```



设置2台机器的hostname

```
hostnamectl set-hostname master   //在master上设置
hostnamectl set-hostname node01   //在node01上设置
```



关闭防火墙(在master和node01上执行)

```
vim /etc/selinux/config
SELINUX=disabled

systemctl stop firewalld && systemctl disable firewalld
setenforce 0
iptables -F

vim /etc/fstab
注释掉swap那一行

swapoff -a

配置各节点系统内核参数使流过网桥的流量也进入iptables/netfilter框架中，在/etc/sysctl.conf中添加以下配置

net.bridge.bridge-nf-call-iptables = 1

net.bridge.bridge-nf-call-ip6tables = 1


sysctl -p
```



### 设置yum源(在master和node01上都执行)

```
#换成阿里云镜像源

mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

#使用阿里云docker镜像
cd /etc/yum.repo.d
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo


#设置国内加速的kubernetes源

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

```



安装docker和kubeadm

```
yum install -y docker-ce kubelet kubeadm
systemctl enable docker && systemctl start docker
systemctl enable kubelet && systemctl start kubelet
```

*注意一定要安装docker-ce 而且要定义好docker的repo源 , 如果只是简单的把CentOS的源替换成了阿里源  ,在执行`yum install -y docker`的时候 下载的docker不是我们要的 ,用命令查看`docker version`会发现docker的版本是1.13的  安装的时候要安装docker-ce `yum install -y docker-ce` 安装完成查看版本的话 显示的是"18.06.0-ce"* 

### 设置docker加速器

```
#方法1   清华大学的docker加速,感觉速度比阿里云的快
vim /etc/docker/daemon.json

请在该配置文件中加入（没有该文件的话，请先建一个）：

{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}

#方法2   阿里云的加速
访问阿里云的加速网址
https://cr.console.aliyun.com/cn-hangzhou/mirrors
之后登陆自己的账号
点击左边的镜像加速器 就可以看到自己的私有加速地址
修改daemon配置文件/etc/docker/daemon.json来使用加速器
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://gekysrd8.mirror.aliyuncs.com"]
}
EOF


systemctl daemon-reload
systemctl restart docker

```



查看docker是否启动

```
systemctl status docker
```



编辑kubelet文件

在kubeadm初始化的时候可能会出现swap错误  v1.11.1其实已经不用再关swap了 以前的版本可能需要关闭 但是现在不需要了

```
vim /etc/sysconfig/kubelet

KUBELET_EXTRA_ARGS="--fail-swap-on=false"
KUBE_PROXY_MODE=ipvs     //启用ipvs  这种service模式这个只有在v1.11.1才能 v1.11.1以下的都不支持
```





### 提前准备拉去所需镜像

众所周知,因为被墙的关系所以kubeadm下载镜像的时候会出问题,这个就是比较头疼的了,

使用`kubeadm init`之后会出现以下情况:

![](http://pe9685fps.bkt.clouddn.com/18-9-2/30068876.jpg)



或者是这样的

![](http://pe9685fps.bkt.clouddn.com/18-9-2/35457829.jpg)



可以看到发现以下情况:

1. 使用的版本是v1.11.1
2. 拉取镜像的各个版本也是v1.11.1 所以 当前kubernetes的版本和拉取镜像的版本要一致对应,高于或者低于当前kubernetes版本都不行
3. 拉取镜像的地址是k8s.grc.io,其实就是在GCE上拉取的,如果没有外网是访问不了的

所以以上问题的出现都是由于不能访问外网,导致docker拉取镜像失败 , 所以我们可以先把需要的docker先拉取下来 , 这样kubeadm就可以正常运行了

kubernetes安装部署麻烦的情况是众所周知 , 所以已经有人开始做这样的事情了 , 就是做一个google的镜像站 , docker下载镜像的默认的地方是[docker hub](https://hub.docker.com)   在docker hub上做的做好的是[mirrorgooglecontainers](https://hub.docker.com/u/mirrorgooglecontainers/)  里面需要的google容器镜像几乎都有,不过有些最新的还没有更新,待会我会和大家来讲解自己手动来获取最新的google容器镜像

运行`kubeadm init`之后 不能获取的镜像已经在报错中列出来了 ,分别是:

```
failed to pull image [k8s.gcr.io/kube-apiserver-amd64:v1.11.1]: exit status 1
failed to pull image [k8s.gcr.io/kube-controller-manager-amd64:v1.11.1]: exit status 1
failed to pull image [k8s.gcr.io/kube-scheduler-amd64:v1.11.1]: exit status 1
failed to pull image [k8s.gcr.io/kube-proxy-amd64:v1.11.1]: exit status 1
failed to pull image [k8s.gcr.io/pause:3.1]: exit status 1
failed to pull image [k8s.gcr.io/etcd-amd64:3.2.18]: exit status 1
failed to pull image [k8s.gcr.io/coredns:1.1.3]: exit status 1

```



#### 在master提前下好镜像

下一步我们就要来获取这些镜像 在master上编写个脚本如下:

```
vim get-k8s-containers.sh   //编写脚本从dockerhub上获取这些镜像

#!/bin/bash
docker pull coredns/coredns:1.1.3
docker tag coredns/coredns:1.1.3 k8s.gcr.io/coredns:1.1.3
docker rmi coredns/coredns:1.1.3
images=(kube-proxy-amd64:v1.11.1 kube-scheduler-amd64:v1.11.1 kube-controller-manager-amd64:v1.11.1 kube-apiserver-amd64:v1.11.1
etcd-amd64:3.2.18 pause-amd64:3.1 kubernetes-dashboard-amd64:v1.8.3 k8s-dns-sidecar-amd64:1.14.8 k8s-dns-kube-dns-amd64:1.14.8
k8s-dns-dnsmasq-nanny-amd64:1.14.8 pause:3.1)
for imageName in ${images[@]} ; do
  docker pull mirrorgooglecontainers/$imageName
  docker tag mirrorgooglecontainers/$imageName k8s.gcr.io/$imageName
  docker rmi mirrorgooglecontainers/$imageName
done
```

之后给脚本加权限

```
chmod +x get-k8s-containers.sh
```

之后运行脚本

```
./get-k8s-containers.sh
```





#### 在node01上提前下载好镜像

,编写脚本如下:

```
vim get-k8s-node-images.sh

#!/bin/bash
docker pull haojianxun/flannel:v0.10.0-amd64
docker tag haojianxun/flannel:v0.10.0-amd64 quay.io/coreos/flannel:v0.10.0-amd64
docker rmi haojianxun/flannel:v0.10.0-amd64
images=(kube-proxy-amd64:v1.11.1 pause:3.1 etcd-amd64:3.2.18)
for imageName in ${images[@]} ; do
  docker pull mirrorgooglecontainers/$imageName
  docker tag mirrorgooglecontainers/$imageName k8s.gcr.io/$imageName
  docker rmi mirrorgooglecontainers/$imageName
done


chmod +x get-k8s-node-images.sh    //脚本加上执行权限
./get-k8s-node-images.sh //运行脚本
```



### 初始化kubeadm

```
kubeadm init --kubernetes-version=v1.11.1 --pod-network-cidr=10.244.0.0/16   //如果要使用flannel网络插件,可以在初始化的时候就指定flannel网络地址

kubeadm init --kubernetes-version=v1.11.1      //不指定其他网络插件具体地址 
```

稍等几分钟就可以看到

![](http://pe9685fps.bkt.clouddn.com/18-9-2/47931854.jpg)





node01加入机器

![](http://pe9685fps.bkt.clouddn.com/18-9-2/35285354.jpg)





列出集群状态

```
kubectl get nodes  //在主节点上查看
```

![](http://pe9685fps.bkt.clouddn.com/18-9-2/60612948.jpg)



可以看到status是NotReady状态 原因是没有安装cni网络插件

### 安装网络插件

方法1:安装flannel插件

```
flannel官方的github地址是:
https://github.com/coreos/flannel

对于Kubernetes v1.7以上的版本 都可以直接运行如下命令:

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

之后运行命令 查看images是否running起来了
kubelet get pods -n kube-system -o wide

查看当前命名空间
kubelet get ns

查看集群是否起来了
kubelet get nodes   //显示ready状态就是起来了



注意:
如果要开启flannel的directrouting功能的话 可以先wget下这个yaml文件 之后再修改配置
操作如下:
1.先下载yaml文件

wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml


2.编辑刚刚下载好的文件

vim https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

找到如下片段:
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
    
    
再原来基础上加上directrouting功能 ,修改完成之后应该是这样的:
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan",
        "Directrouting": true
      }
    }
    
之后保存退出
之后再应用:
https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

这样以后各个节点通信的话 就是通过物理网卡直接通信了 可以用ip route show 命令查看规则 , 在节点上用tcpdump -i ens32 -nn icmp 抓包看
```



方法2:安装weave网络插件

```
我们 选择weave  当然在初始化的时候没有指定特殊网络才行

kubectl apply -f https://git.io/weave-kube-1.6

之后运行kubectl get nodes即可看到是ready状态
```



稍等片刻 下载好之后  在master上执行如下命令, 可以查看是否集群已经起来了

```
kubectl get pod -n kube-system -o wide    //都是running就是正常的
kubelet get nodes  //可以看到master和node02都是ready状态
```



## 可能出现问题和解决方法

### 无法拉去k8s.gcr.io相关的镜像

#### 方法1:http代理

最简单的解决方法是设置http代理

docker官方给出的设置http代理方法如下:

[docker http_proxy](https://docs.docker.com/config/daemon/systemd/#configure-where-the-docker-daemon-listens-for-connections)



设置http代理的好处就是一劳永逸 , 不用再哭哈哈的编写脚本 ,提前拉去镜像 , 也不用再把墙外的google镜像拉到国内再下载 之后打tag

设置http代理之后就可以直接运行kubeadm了 其他一切都不需要额外设置

我们可以这样来设置http代理

```
1. vim /usr/lib/systemd/system/docker.service

在[Service]中添加如下信息
Environment="HTTP_PROXY=http:xxxxx:80"
Environment="NO_PROXY=127.0.0.0/8,192.168.0.0/16"    

#这里的192.168.0.0/16是自己本地的网络,替换成自己的本地网络地址
#NO_PROXY的意思就是要走本地网络的就不需要来代理了,直接走本地网络即可


2. 编辑完成之后执行如下命令:
systemctl daemon-reload
systemctl start docker
```

可以用docker info查看具体信息 看看http_proxy有没有设置好, 当然如果是有公司的未批嗯的话最好,没有的话 可以google一个 由于敏感 这样就不放出来了 ,http代理网上一大堆 可以直接用搜索引擎找一个



下面列出几个比较好用的http代理网站

http://cn-proxy.com/archives/218

https://www.kuaidaili.com/free/intr/

http://www.66ip.cn/

http://www.xicidaili.com/

http://www.coobobo.com/



#### 方法2:使用中转站来拉去镜像

1. 一般来说 ,由于网络被墙 , 许多人开始做这个工作了 , 就是把google的相关镜像拉去到dockerhub上  , 之后我们在dockerhub上来下载所需的镜像 , 在安装特定的版本的时候会要求特定的镜像 , 这个时候可以去[mirrorgooglecontainers](https://hub.docker.com/u/mirrorgooglecontainers/) 找 , 如果版本没有更新 ,可以试试以下方法:

   首先 能科学上网 , 可以在自己浏览器的插件市场上下载一个 ,比如"谷歌访问助手" 等等 , 垃圾的也行 , 只要能访问外网就成 , 不要求网速.

   在执行`kubeadm init`的时候对于没有拉取下来的镜像,可以直接到[k8s.gcr.io/google_containers](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL) 下载

   ![](http://pe9685fps.bkt.clouddn.com/18-9-2/94357091.jpg)

2. 启动控制台

    

![](http://pe9685fps.bkt.clouddn.com/18-9-2/23538517.jpg)

3. 以kube-apiserver-amd64为例子 ,搜索kube-apiserver-amd64

   ![](http://pe9685fps.bkt.clouddn.com/18-9-2/92677484.jpg)





   点击kube-apiserver-amd64  我们安装v1.11.1的 点击标记为v1.11.1的镜像

   ![](http://pe9685fps.bkt.clouddn.com/18-9-2/52251691.jpg)

4. 运行拉取命令

   ![](http://pe9685fps.bkt.clouddn.com/18-9-2/99511944.jpg)

5. 中转镜像 , 把镜像拉取出来传到dockerhub上自己的仓库  之后拉取push上的镜像

   

![](http://pe9685fps.bkt.clouddn.com/18-9-2/57435961.jpg)

6. docker获取刚刚上传的镜像并打上k8s.gcr.io/的前缀tag

   ```
   docker pull haojianxun/kube-apiserver-amd64:v1.11.1
   docker images  //列出刚刚下载的镜像
   docker tag kube-apiserver-amd64:v1.11.1 k8s.gcr.io/kube-apiserver-amd64:v1.11.1
   ```


最后镜像下载完成



### kubeadm init可能出现的报错

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





