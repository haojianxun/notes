# gpg /  ssh

[TOC]

## gpg实现对称加密

对称加密文件file

```bash
gpg -c file
ls file.gpg
```



在另一台主机上解密gpg文件

```
gpg -o file -d file.gpg
```

## 使用gpg实现公钥加密

在主机A上: 使用命令  生成公钥-私钥对

```bash
gpg --gen-key  #在主机A上执行这个命令
```

生成之后可以查看生成的密钥

```bash
ls .gnupg/   #在生成的时候有提示信息的 在用户的家目录下 有个隐藏文件夹 就是这货
```

查看公钥

```bash
gpg --list-keys
```

导出公钥

```bash
gpg -a --export -o /root/FILENAME #-o指定导出的路径,可以自己指定
```

然后拷贝到host B 上

```bash
scp FILENAME [user@]IPADDR:/PATH   #FILENAME指的是导出的公钥,然后指定ip地址和路径
```

然后在host B上导入公钥

```bash
gpg --import FILENAME #FILENAME指的是要导入的公钥文件名
```

在hostB上也生成公钥私钥对

```bash
gpg --gen-key
```

用从hostA 主机导入的公钥，加密hostB 主机的文件file, 生成file.gpg

```
gpg -e -r wangxiaochun file  #-r导入的公钥不是公钥的文件名 是公钥的uid 可以用--list-keys查看
file file.gpg
```

加密之后scp传给host A

host A 收到之后就解密

```bash
gpg -d file.gpg #-d的意思就是解密 但是解密显现的是内容 没有保存下来
gpg -o file -d file.gpg #用-o来保存解密之后的文件
```

  删除公钥和私钥

```bash
gpg --delete-keys filename
gpg --delete-secret-keys filename  #先删除私钥
```

## ssh

 ssh: secure shell, protocol, 22/tcp,  安全的远程登录
 OpenSSH: ssh 协议的开源实现
 dropbear ：另一个开源实现
SSH 协议版本

- v1:  基于CRC-32 做MAC ，不安全；man-in-middle
- v2 ：双方主机协议选择安全的MAC 方式

基于DH 算法做密钥交换，基于RSA 或DSA 实现身份认证
两种方式的用户登录认证：

- 基于password
- 基于key

ssh服务器程序    sshd



## ssh客户端

ssh  配置文件  /etc/ssh/ssh_config

```bash
ssh [user@]host [COMMAND]   #语法
ssh [-l user] host [COMMAND]
-p port ：远程服务器监听的端口
-b: 指定连接的源IP
-v: 调试模式
-C ：压缩方式
-X:  支持x11 转发
-Y ：支持信任x11 转发
ForwardX11Trusted yes
-t: 强制伪tty 分配
ssh -t remoteserver1 ssh remoteserver2
```

> 当用户远程连接ssh 服务器时，会复制ssh 服务器/etc/ssh/ssh_host*key.pub （centos7.0 默认是
> ssh_host_ecdsa_key.pub ）文件中的公钥到客户机的~./ssh/know_hosts 中。下次连接时，会比较两处是否有不同。

