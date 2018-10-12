# 使用ssh进行git登陆和推送

查看git的远程地址是否是ssh形式的

```
git remote -v
```

不是的话 就换车ssh形式的

```
git remote rm origin
git remote add origin git@github.com:haojianxun/mirror-grc.io.git
```

## 生成密钥

```
ssh-keygen -t rsa -p '' -C "haojinaxun@gmail.com"
```

确保 ssh-agent 是可用的。ssh-agent是一种控制用来保存公钥身份验证所使用的私钥的程序，其实ssh-agent就是一个密钥管理器，运行ssh-agent以后，使用ssh-add将私钥交给ssh-agent保管，其他程序需要身份验证的时候可以将验证申请交给ssh-agent来完成整个认证过程。

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```



## 将公钥放到github上去

```
cat ~/.ssh/id_rsa.pub   //复制这个公钥
```

之后登陆github , 点击右上角账号设置 , 找到`settings` 设置项 , 之后找到`SSH and GPG keys`  将公钥添加进去



## 测试

```
ssh -T git@github.com
```

你将会看到：

```
    The authenticity of host 'github.com (207.97.227.239)' can't be established.
    RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
    Are you sure you want to continue connecting (yes/no)?
```

选择 `yes`

## 修改`.git`文件夹下`config`中的`url`

进入到项目目录中

之后

```
cd .git
vim config
```

将其中的`url`的形式改成ssh形式的

修改后是这样的

```
[remote "origin"]
        url = git@github.com:haojianxun/mirror-grc.io.git
        fetch = +refs/heads/*:refs/remotes/origin/*
```



之后就可以进行正常的推送了



如果脚本中要用到git  一般来说都是设置成ssh形式 , 要是不这样设置的话 , git登陆还得用交互式的 很麻烦