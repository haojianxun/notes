# kubernetes中docker连到私有仓库认证问题

如果docker要连到私有仓库地址的话 , 可以在`pod.spec`下`imagePullSecrets`这个选项 这个secret是由secret创建 , 而且是专有的secret类型 , 用来存放账号密码

目前secret类型分成3类(可以通过`kubectl create secret --help`来查看帮助)

- docker-registery
- generic 
- tls  (放证书和私钥)

我们可以把这个secret放到serviceaccount上   因为创建serviceaccount的时候可以指定`iamge pull secret`  , 

```
kubectl describe sa mysa
```

```
Name:                mysa
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   mysa-token-xrbjz
Tokens:              mysa-token-xrbjz
Events:              <none>

```

可以看到创建的serviceaccount的时候 是可以指定`Image pull secrets`   , 所以把认证私有仓库的账号密码放在这个secret上 , 这个secret是附在serviceaccount上的 , 所以再由pod带着`serviceAccountName`去认证私有仓库的密码 ,  这样做就防止了账号密码的泄漏风险

