# docker image分层构建 联合挂载

docker采用分层构建 底层为bootfs , 在它之上为rootfs

- bootfs 用于系统引导文件系统 , 包括bootloader和kernel , 容器启动完成之后会被卸载以节约内存
- rootfs 在bootfs之上,表现为docker的根文件系统
  - 传统模式是 启动的时候, 内核挂载rootfs时会首先挂载为只读模式 , 完整性自检完成后将其重新挂载为读写模式
  - docker中 rootfs由内核挂载为只读模式 , 而后通过联合挂载技术额外挂载一个可写层

aufs

用于为Linux文件系统实现联合挂载

在ubuntu系统下 docker默认用的是aufs  在CentOS7上用的是devicemapper