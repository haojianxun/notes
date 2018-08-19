# 解决kubeadm安装k8s的时候images不能下载的问题--方法2

首先安装docker

```
wge thttps://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  //使用阿里云的docker镜像

yum repolist 

yum install -y docker-ce

```



编辑docker相关文件

```
1. vim /usr/lib/systemd/system/docker.service

在[Service]中添加如下信息
Environment="HTTPS_PROXY=http://www.ik8s.io:10080"
Environment="NO_PROXY=127.0.0.0/8,192.168.0.0/16"    

#这里的192.168.0.0/16是自己本地的网络,替换成自己的本地网络地址   


2. 编辑完成之后执行如下命令:
systemctl daemon-reload
systemctl start docker

可以用docker info查看具体信息
```

查看系统信息

```
cat /proc/sys/net/bridge/bridge-nf-call-ip6tables    //确保是1
cat /proc/sys/net/bridge/bridge-nf-call-iptables    //确保是1
```

