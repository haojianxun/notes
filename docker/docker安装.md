# docker安装

下面是基于CentOS



##卸载旧版本

旧版本的 Docker 称为  docker  或者  docker-engine  ，使用以下命令卸载旧版本：

```
$ sudo yum remove docker \
           docker-common \
           docker-selinux \
           docker-engine
```



##使用 yum 源 安装

执行以下命令安装依赖包：

```
$ sudo yum install -y yum-utils \
          device-mapper-persistent-data \
          lvm2
```

鉴于国内网络问题，强烈建议使用国内源，下面先介绍国内源的使用。



###国内源

执行下面的命令添加  yum  软件源：

```
$ sudo yum-config-manager \
     --add-repo \
     https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

> 以上命令会添加稳定版本的 Docker CE yum 源。从 Docker 17.06 开始，edge test 版本的 yum 源也会包含稳定版本的 Docker CE。



###官方源

```
$ sudo yum-config-manager \
      --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo
```


如果需要最新版本的 Docker CE 请使用以下命令：

```
$ sudo yum-config-manager --enable docker-ce-edge
$ sudo yum-config-manager --enable docker-ce-test
```



###安装 Docker CE

更新  yum  软件源缓存，并安装  docker-ce  。

```
$ sudo yum makecache fast
$ sudo yum install docker-ce
```



####使用脚本自动安装

在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，CentOS
系统上可以使用这套脚本安装：

```
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker CE 的 edge 版本安
装在系统中。

#### 启动 Docker CE

```
$ sudo systemctl enable docker
$ sudo systemctl start docker
```



###建立 docker 用户组

默认情况下， docker  命令会使用 Unix socket 与 Docker 引擎通讯。而只有  root  用户和
docker  组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统
上不会直接使用  root  用户。因此，更好地做法是将需要使用  docker  的用户加入  docker
用户组。

建立  `docker`   组：

```
$ sudo groupadd docker
```

将当前用户加入  `docker`   组：

```
$ sudo usermod -aG docker $USER
```



### 镜像加速

鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，强烈建议安装 Docker 之后配置 国内镜
像加速。



### 添加内核参数

默认配置下，如果在 CentOS 使用 Docker CE 看到下面的这些警告信息：

```
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```


请添加内核配置参数以启用这些功能。

```
$ sudo tee -a /etc/sysctl.conf <<-EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

然后重新加载  `sysctl.conf `  即可

```
$ sudo sysctl -p
```

