CentOS系统管理考试题

配置 yum 仓库
配置 yum 仓库，名称为 epel,yum 的地址为 http://172.16.0.1/fedora-epel/7/x86_64/
将 centos7.2 的光盘所有 rpm 包复制到在本机/myrepo 目录下，并创建 yum 仓库，配置本机为其 yum 客户端

答:刚开始进系统 当前的目录是在自己家目录

```
cd /etc/yum.repos.d
vim epel.repo
[epel]
name=epel
baseurl=http://172.16.0.1/fedora-epel/7/x86_64/
gpgcheck=0

mkdir /myrepo
cd /run/media/root/CentOS 7 x86_64
cp -r Packages /myrepo
cd /etc/yum.repos.d
cp CentOS-Base.repo CentOS-Base.repo.bak
vim CentOS-Base.repo
[base]
name=local yumrepo
baseurl=file:///myrepo
gpgcheck=0

```

2 按从大打小的顺序显示

```
ll --sort=size   ll -S
```

按mtime时间显示文件列表

```
ll -t
```

按atime显示

```
ll -ut
```

只显示隐藏文件

```
l.    ls -d .*
```

只显示目录

```
ls -d */    ls -ap |grep /      ls -aF |grep /
```

3 给root定义别名

```
su 
输入密码
vim .bashrc
alias vimnet='vim /etc/sysconfig/network-scripts/ifcfg-eth0'
HISTTIMEFORMAT=“%F %T“
```

5误删除kernel包后恢复

```
chroot /mnt/sysimage
rpm -vih /mnt/cdrom/Packages/kernel-xxx.rpm --force
grub-install /dev/sda
```

创建账号

```
useradd -u 3000 -g root -s /sbin/nologin lala
useradd haha -g sales
useradd hehe -g sales
passwd haha   hehe

```

9 cat /testdir/file2 |grep -v ^#

