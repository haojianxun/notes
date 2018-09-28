# docker存储卷

写时复制

docker镜像由多个只读层叠加而成 , 启动容器时,docker会加载只读镜像层并在镜像栈顶部添加一个读写层

如果运行中的容器修改了一个已经存在的文件 , 那该文件将会从下面的只读层复制到读写层,该文件的只读版本仍然存在,只是被读写层中该文件的副本所隐藏 ,此即`写时复制(COW)`机制 



## docker挂载

docker-managerd volume

```
docker run -it -name NAME -v /data busybox  

挂载了一个由docker管理的存储卷  挂载到了docker内部的/data下  对应于宿主机来说 他的目录是在/var/lib/docker/volume/MOUNTS_NAME/_data   可以使用命令docker imspect DOCKER_NAME 查看 , 在Mounts一段有具体说明

使用命令
docker inspect -f {{.Mounts}} busybox  //查看busybox挂载的卷,卷标识符 存储目录
```

bind-mount volume

```
docker run -it -name NAME -v HOSTDIR:CONTAINERDIR busybox

在启动busybox的时候 挂载一个存储卷 

使用命令
docker inspect -f {{.Mounts}} busybox  //查看busybox挂载的卷,卷标识符 存储目录
```

## 共享存储

多个容器卷使用同一个主机目录

```
docker run -it -name c1 -v /data/docker/volume:/data busybox 
docker run -it -name c2 -v /data/docker/volume:/data busybox 
```

复制使用其他容器的卷

```
docker run -it -name c1 -v /data/docker/volume:/data busybox 
docker run -name nginx --network container:c1 --volumes-from c1 busybox
```





