# yum安装的时候使用gpgkey

```
在repo中启用gpgkey

gpgcheck=1
gpgkey=xxxxx.gpg
enables=1


当然也可以直接把gpgkey下载下来
wget xxxxx.gpg
rpm --import xxx.gpg
```

