# docker中什么是**rootfs（根文件系统）**

一个最常见的 rootfs，或者说容器镜像，会包括如下所示的一些目录和文件，比如 /bin，/etc，/proc 等等：

```
ls /
bin dev etc home lib lib64 mnt opt proc root run sbin sys tmp usr var
```

Docker 在镜像的设计中，引入了层（layer）的概念。也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs。  用到的技术就是联合文件系统（Union File System） 



Union File System 也叫 UnionFS ，最主要的功能是将多个不同位置的目录联合挂载（union mount）到同一个目录下