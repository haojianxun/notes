# kubernetes中docker连到私有仓库认证问题

如果docker要连到私有仓库地址的话 , 可以在`pod.spec`下`imagePullSecrets`这个选项 这个secret是由secret创建 , 而且是专有的secret类型 , 用来存放账号密码

目前secret类型分成3类(可以通过`kubectl create secret --help`来查看帮助)

- docker-registery
- generic
- tls  (放证书和私钥)