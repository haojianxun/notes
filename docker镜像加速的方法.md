# docker镜像加速的方法

方法1

```
vim /etc/docker/daemon.json

请在该配置文件中加入（没有该文件的话，请先建一个）：

{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```



method 2

```
vim /lib/systemd/system/docker.service

在ExecStart下加入如下选项:
--registry-mirror=MIRROR-ADDRESS
```



method 3

```
指向私有仓库 harbor
vim /etc/docker/daemon.json

{
    "insecure-registries":["192.168.200.139:5000"]
}

systemctl restart docker
```

