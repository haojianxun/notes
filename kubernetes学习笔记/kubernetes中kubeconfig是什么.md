# kubernetes中kubeconfig是什么?

kubeconfig是kubernetes集群中各个组件(api-server客户端)连入api-server的时候 , 使用的认证格式的客户端配置文件



查看kubeconfig配置文件

```
kubectl config view
```

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: https://192.168.200.140:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

可以看到里面有`clusters`   这个就表示集群列表 , `users`表示 用户列表 ,  还有上下文列表

context表示  我们用哪个账号访问哪个集群



