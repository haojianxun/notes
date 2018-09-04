docker相关概念

docker daemon启动的时候用libcontainer启动  启动的驱动是execdrive  下载相关的数据是由graphdb保存的,下载下来保存的分层结构是保存在/var/lib/docker/graph中



index管理用户账号,访问权限,镜像,和镜像标签等相关

repostory  由具有某个功能镜像的所有相关版本集合

registry 保存docker镜像及镜像层次结构和元数据