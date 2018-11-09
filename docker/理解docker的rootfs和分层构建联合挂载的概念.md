# 理解docker的rootfs和分层构建联合挂载的概念

一个最常见的 rootfs，或者说容器镜像，会包括如下所示的一些目录和文件，比如 /bin，/etc，/proc 等等：

```
ls /
bin dev etc home lib lib64 mnt opt proc root run sbin sys tmp usr var
```

Docker 在镜像的设计中，引入了层（layer）的概念。也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs。  用到的技术就是联合文件系统（Union File System） 



Union File System 也叫 UnionFS ，最主要的功能是将多个不同位置的目录联合挂载（union mount）到同一个目录下



docker项目最著名的UnionFS的实现是AUFS, 它是对 Linux 原生 UnionFS 的重写和改进 , 由于AuFS 未进入 Linux 内核主干的缘故，所以我们只能在 Ubuntu 和 Debian 这些发行版上使用它

UnionFS的其他实现分别有: aufs, device mapper, btrfs, overlayfs, vfs, zfs。aufs是ubuntu 常用的，device mapper 是 centos，btrfs 是 SUSE，overlayfs ubuntu 和 centos 都会使用，现在最新的 docker 版本中默认两个系统都是使用的 overlayfs，vfs 和 zfs 常用在 solaris 系统。 



举例子来说明:

````
docker run -d ubuntu
````

这时 , docker就会去dockerhub上拉去ubuntu的最新镜像

结果如下:

```bash
$ docker run -d ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
473ede7ed136: Pull complete 
c46b5fa4d940: Pull complete 
93ae3df89c92: Pull complete 
6b1eed27cade: Pull complete 
Digest: sha256:29934af957c53004d7fb6340139880d23fb1952505a15d69a03af0d1418878cb
Status: Downloaded newer image for ubuntu:latest
124a67c0309c387ae9453408fe80c3f7ec0f9f9860b2e169d7dc4690d18caf63
```

可以看到出现了几组字母和数字组合的字符串, 后面显示 pull complete , 这些就拉去下来的层

我们可以用命令去查看具体的层

```
docker image inspect ubuntu
```

其中有一段显示:

```
.....

 },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:102645f1cf722254bbfb7135b524db45fbbac400e79e4d54266c000a5f5bc400",
                "sha256:ae1f631f14b7667ca37dca207c631d64947c60d923995cf0d73ceb1b08c406bb",
                "sha256:2146d867acf390370d4d0c7b51951551e0e91fb600b69dbc8922d531b05b12bc",
                "sha256:76c033092e100f56899d7402823c5cb6ce345442b3382d7b240350ef4252187e"
            ]
        },

.....
```



docker会把这几个层统一挂载到`/var/lib/docker/aufs/mnt` 



那这几个层是怎么挂载到这个目录下的呢?

挂载的信息记录在` /sys/fs/aufs `



首先，通过查看 AuFS 的挂载信息，我们可以找到这个目录对应的 AuFS 的内部 ID（也叫：si）

```
cat /proc/mounts| grep aufs
```

然后使用这个 ID，你就可以在 `/sys/fs/aufs `下查看被联合挂载在一起的各个层的信息

可以看到镜像的层都放置在` /var/lib/docker/aufs/diff `目录下，然后被联合挂载在 `/var/lib/docker/aufs/mnt `里面



容器的rootfs分为三个部分:

- 可读层(rw)
- init层(ro+wh)
- 只读层(ro+wh)

**只读层**对于的docker的镜像层, 这部分不可被修改

**可读层**是 rootfs 最上面的一层,它的挂载方式为：rw，即 read write。在没有写入文件之前，这个目录是空的。而一旦在容器里做了写操作，你修改产生的内容就会以增量的方式出现在这个层中。 删除文件的话 ,AuFS 会在可读写层创建一个 whiteout 文件，把只读层里的文件“遮挡”起来。比如，你要删除只读层里一个名叫 foo 的文件，那么这个删除操作实际上是在可读写层创建了一个名叫.wh.foo 的文件。这样，当这两个层被联合挂载之后，foo 文件就会被.wh.foo 文件“遮挡”起来，“消失”了。这个功能，就是“ro+wh”的挂载方式，即只读 +whiteout 的含义

**init层**它是一个以“-init”结尾的层，夹在只读层和读写层之间。Init 层是 Docker 项目单独生成的一个内部层，专门用来存放 /etc/hosts、/etc/resolv.conf 等信息

