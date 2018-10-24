# 如何快速给某个用户授权sudo

sudo授权的命令是`visudo`   里面有各种授权组 其中有一行

```
%wheel ALL=(ALL) ALL
```

所以想让某个用户快速获得`sudo`授权  , 就可以让他加入这个组

```
usermod -aG wheel wang
```



如果用户sudo的时候不想让他输入root密码 怎么办

在`visudo` 的时候  看到的配置文件中有一行

```
%wheel ALL=(ALL) NOPASSWD: ALL  
```

这行默认是注释掉的  , 我们可以启用 , 这样就不用担心sudo的时候输入root密码了