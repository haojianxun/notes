# ssh-keygen AND ssh-copy-id用法

关于ansible上要用的ssh认证

## ssh-keygen

ssh 公钥认证是ssh认证的方式之一。通过公钥认证可实现ssh免密码登陆，git的ssh方式也是通过公钥进行认证的。

在用户目录的home目录下，有一个`.ssh`的目录，和当前用户ssh配置认证相关的文件，几乎都在这个目录下。

`ssh-keygen` 可用来生成ssh公钥认证所需的公钥和私钥文件

### 生成的文件名和文件位置

使用 `ssh-kengen` 会在~/.ssh/目录下生成两个文件，不指定文件名和密钥类型的时候，默认生成的两个文件是：

- `id_rsa`
- `id_rsa.pub`

第一个是私钥文件，第二个是公钥文件。

生成ssh key的时候，可以通过 `-f` 选项指定生成文件的文件名，如下:

```
[huqiu@101 .ssh]$ ssh-keygen -f test   -C "test key"
                             ~~文件名   ~~~~ 备注
```

### 语法和选项

```
ssh-keygen(选项)
-b：指定密钥长度； 
-e：读取openssh的私钥或者公钥文件； 
-C：添加注释； 
-f：指定用来保存密钥的文件名； 
-i：读取未加密的ssh-v2兼容的私钥/公钥文件，然后在标准输出设备上显示openssh兼容的私钥/公钥； 
-l：显示公钥文件的指纹数据； 
-N：提供一个新密语； 
-P：提供（旧）密语； 
-q：静默模式； 
-t：指定要创建的密钥类型

```

## ssh-copy-id

ssh-copy-id命令可以把本地主机的公钥复制到远程主机的authorized_keys文件上，ssh-copy-id命令也会给远程主机的用户主目录（home）和~/.ssh, 和~/.ssh/authorized_keys设置合适的权限。

```
ssh-copy-id [-i [identity_file]] [user@]machine

-i：指定公钥文件

例子
1、把本地的ssh公钥文件安装到远程主机对应的账户下：
ssh-copy-id user@server 
ssh-copy-id -i ~/.ssh/id_rsa.pub user@server
```

