# docker相关命令--pull , commit ,save,load

## docker pull

```
docker pull <registry>:[:<port>]/[<namespace>/]<name>:<tag>
```

**registry**: 默认是dockerhub  也可以用其他的仓库地址 , 但是一般忽略协议(https://) , 比如grc.io , quary.io

一般 分成3类:

- sponsor registry: 第三方的registry 供客户和docker社区使用
- mirror registry: 第三方registry 只让客户使用 , 比如阿里云
- vendor registry: 由发布docker镜像的供应商提供的registry
- private registry: 通过设有防火墙和安全层的私有实体提供的registry

	

**namespace**:

| namespace        | examples(<namespace>/<name>)        |
| ---------------- | ----------------------------------- |
| organization     | redhat/kubernetes , google/         |
| login(user name) | alics/application , bob/application |
| role             | devel/database , test/database      |

## docker容器制作

- dockerfile
- docker commit(基于docker容器制作)
- autobulit  dockerhub上自动构建

基于容器制作镜像

### docker commit

```
docker commit [OPTIONS]CONTAINER[REPOSITORY[:TAG]]
```

#### Options

| Name, shorthand  | Default | Description                                                  |
| ---------------- | ------- | ------------------------------------------------------------ |
| `--author , -a`  |         | Author (e.g., “John Hannibal Smith [hannibal@a-team.com](mailto:hannibal@a-team.com)”) |
| `--change , -c`  |         | Apply Dockerfile instruction to the created image            |
| `--message , -m` |         | Commit message                                               |
| `--pause , -p`   | `true`  | Pause container during commit                                |

#### Examples

##### Commit a container

```
$ docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS              NAMES
c3f279d17e0a        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            desperate_dubinsky
197387f1b436        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            focused_hamilton

$ docker commit c3f279d17e0a  svendowideit/testimage:version3

f5283438590d

$ docker images

REPOSITORY                        TAG                 ID                  CREATED             SIZE
svendowideit/testimage            version3            f5283438590d        16 seconds ago      335.7 MB
```

##### Commit a container with new configurations

```
$ docker ps

CONTAINER ID       IMAGE               COMMAND             CREATED             STATUS              PORTS              NAMES
c3f279d17e0a        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            desperate_dubinsky
197387f1b436        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            focused_hamilton

$ docker inspect -f "{{ .Config.Env }}" c3f279d17e0a

[HOME=/ PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin]

$ docker commit --change "ENV DEBUG true" c3f279d17e0a  svendowideit/testimage:version3

f5283438590d

$ docker inspect -f "{{ .Config.Env }}" f5283438590d

[HOME=/ PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin DEBUG=true]
```

##### Commit a container with new `CMD` and `EXPOSE` instructions

```
$ docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS              NAMES
c3f279d17e0a        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            desperate_dubinsky
197387f1b436        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            focused_hamilton

$ docker commit --change='CMD ["apachectl", "-DFOREGROUND"]' -c "EXPOSE 80" c3f279d17e0a  svendowideit/testimage:version4

f5283438590d

$ docker run -d svendowideit/testimage:version4

89373736e2e7f00bc149bd783073ac43d0507da250e999f3f1036e0db60817c0

$ docker ps

CONTAINER ID        IMAGE               COMMAND                 CREATED             STATUS              PORTS              NAMES
89373736e2e7        testimage:version4  "apachectl -DFOREGROU"  3 seconds ago       Up 2 seconds        80/tcp             distracted_fermat
c3f279d17e0a        ubuntu:12.04        /bin/bash               7 days ago          Up 25 hours                            desperate_dubinsky
197387f1b436        ubuntu:12.04        /bin/bash               7 days ago          Up 25 hours                     
```



## 容器的导入和导出

### docker save

#### Usage

```none
docker save [OPTIONS] IMAGE [IMAGE...]
```

#### Options

| Name, shorthand | Default | Description                        |
| --------------- | ------- | ---------------------------------- |
| `--output , -o` |         | Write to a file, instead of STDOUT |

#### Examples

##### Create a backup that can then be used with `docker load`.

```
$ docker save busybox > busybox.tar

$ ls -sh busybox.tar

2.7M busybox.tar

$ docker save --output busybox.tar busybox

$ ls -sh busybox.tar

2.7M busybox.tar

$ docker save -o fedora-all.tar fedora

$ docker save -o fedora-latest.tar fedora:latest
```



### docker load

#### Usage

```none
docker load [OPTIONS]
```

#### Options

| Name, shorthand | Default | Description                                  |
| --------------- | ------- | -------------------------------------------- |
| `--input , -i`  |         | Read from tar archive file, instead of STDIN |
| `--quiet , -q`  |         | Suppress the load output                     |

#### Examples

```
$ docker image ls

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

$ docker load < busybox.tar.gz

Loaded image: busybox:latest
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              769b9341d937        7 weeks ago         2.489 MB

$ docker load --input fedora.tar

Loaded image: fedora:rawhide

Loaded image: fedora:20

$ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              769b9341d937        7 weeks ago         2.489 MB
fedora              rawhide             0d20aec6529d        7 weeks ago         387 MB
fedora              20                  58394af37342        7 weeks ago         385.5 MB
fedora              heisenbug           58394af37342        7 weeks ago         385.5 MB
fedora              latest              58394af37342        7 weeks ago         385.5 MB
```