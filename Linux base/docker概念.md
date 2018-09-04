# docker概念

registry:保存docker镜像和镜像层次结构和元数据

repository:某个功能镜像的所有相关版本构成的集合

index:管理用户的账号,访问权限,镜像和镜像标签等相关

graph:从registry中下载的docker镜像需要保存在本地,此功能由graph完成  路径在/var/lib/docker/graph



## docker相关命令

```
docker pull [OPTIONS] NAME[:TAG|@DIGEST]


Pull an image or a repository from a registry

Options:
  -a, --all-tags                Download all tagged images in the repository
      --disable-content-trust   Skip image verification (default true)
      --help                    Print usage
      
#docker pull的命令后面跟一个容器名称,是从默认的hub.docker.com中下载的 像CentOS和busybox等都是官方的,所以就没有registry/registory 直接就是在docker pull后面跟镜像名称,要是像其他的私人做的镜像就要跟  '仓库名称/镜像名称:标签'  这样的形式
```

docker push

```
docker push 是要把一个镜像推向某个仓库

push之前要先打tag

docker tag IMAGE_ID REGISTORY_HOST:PORT/NAME[:TAG]

之后打好标签
docker push REGISTORY_HOST:PORT/NAME[:TAG]

比如:docker tag efe10ee6727f haojianxun/busybox:lala
docker push haojianxun/busybox:lala

我的hub.docker.com中的仓库 就是haojianxun   其中busybox是我自己registory  lala是打的标签
```

docker启动

```
docker run -it --rm -v -d --name NAME IMAGE_NAME[:TAG] COMMEND

-it的意思就是运行起来的docker要进入交互模式
--rm //容器结束之后就删除
-v //-v就是挂载  挂载的形式有2种,一种是默认 比如:-v /data  这个的意思就是说把宿主机里面的/var/lib/docker/volumes/  ,/var/lib/docker/volumes/就是默认的宿主机路径
还有一个就是自己指定路径挂载
-v /HOST/DIR:/CONTAINER/DIR
/HOST/DIR:宿主机路径
/CONTAINER/DIR:容器中的路径
```

自己私建仓库

```
安装docker-registry程序包
启动服务
	systemctl start docker-registry.service
建议使用nginx反代,使用ssl,基于basic做用户认证

docker配置仓库
配置文件/etc/sysconfig/docker
ADD_REGISIRY='--add-registry 192.168.1.1:5000'
INSECURE_REGISTRY='--insecure-registry 192.168.1.1:5000'
```

