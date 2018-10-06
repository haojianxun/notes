#kubernetes集群的外部地址和内部地址

查看kubernetes的内部通讯地址

```
kubectl get svc
```

```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   46d
```

这个10.96.0.1就是内部的地址 , 是集群内部pod访问api-server的地址 , 可以查看具体的内容

```
kubectl get svc kubernetes
```

```
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP:                10.96.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         192.168.200.140:6443
Session Affinity:  None
Events:            <none
```

可以看到有个endpoints  这个就是集群的pod地址  , 这个