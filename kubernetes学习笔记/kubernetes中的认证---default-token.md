# kubernetes中的认证---default-token

在一个pod中 , pod可以通过一个token来认证 pod中各个容器

先查看pod

```
kubectl get pod
```

我们随便挑一个pod来看

```
kubectl describe pod myapp-deploy-67f6f6b4dc-zsr5j
```

可以看到 , 我们在创建的时候并没有指定存储卷 , 但是他自己就有了一个

```
Volumes:
  default-token-vwdjg:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-vwdjg
    Optional:    false
```

这个默认的存储卷是default-token  , 这个pod在任何一个名称空间都存在 , 他通过挂载一个secret资源来存储 , 让一个pod中的其他容器来进行认证