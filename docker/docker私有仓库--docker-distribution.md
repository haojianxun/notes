# docker私有仓库--docker-distribution

安装docker-distribution

```
yum install -y docker-distribution
```

配置文件路径

```
cd /etc/docker-distribution/registry
vim config.yml
```

启动服务

```
systemctl start docker-distribution
```





当我们推的时候 , 走的是https协议 而这里的是http

所以我们可以在我们要push的主机上的/etc/docker/daemon.json增加一项

```json
{
    "insecure-registries":["REGISTRY:5000"]
}
```

