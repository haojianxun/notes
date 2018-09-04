# 一个nginx-ingress部署示例

### 先下载nginx-ingress

```
for file in configmap.yaml default-backend.yaml namespace.yaml rbac.yaml tcp-services-configmap.yaml udp-services-configmap.yaml with-rbac.yaml;do wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/$file; done
```

### 部署一个NodePort

官方给出的做法是:

```kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml
```



不过我们可以先下载下来:

```
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml
```

修改其中的端口:
`vim service-nodeport.yaml` 

原来的文件是这样的:

```
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
  - name: https
    port: 443
    targetPort: 443
    protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx
```

修改后是这样的:

```
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort:30080
    protocol: TCP
  - name: https
    port: 443
    targetPort: 443
    nodePort:30443
    protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx
```



### 由于网络原因 也可以手动提前下载

```
docker pull haojianxun/defaultbackend:1.4

docker tag haojianxun/defaultbackend:1.4 gcr.io/google_containers/defaultbackend:1.4

docker rmi haojianxun/defaultbackend:1.4

docker pull haojianxun/nginx-ingress-controller:0.18.0

docker tag haojianxun/nginx-ingress-controller:0.18.0 quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.18.0

docker rmi haojianxun/nginx-ingress-controller:0.18.0
```



### 应用这些yaml

```
kubectl apply -f ./
```

### 查看这些部署的pods

```
kubectl get pods -n ingress-nginx
```

具体部署的语法,可以查看帮助

```
kubectl explain ingress
```

### 先部署一组pod和service

vim myapp-deploy.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: default
spec:
  selector:
    app: myapp
    release: canary
  ports:
  - name: http
    targetPort: 80
    port: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      release: canary
  template:
    metadata:
      labels:
        app: myapp
        release: canary
    spec:
      containers:
      - name: myapp
        image: ikubernetes/myapp:v2
        ports:
        - name: http
          containerPort: 80

```





### 可以部署一个应用用ingress发布出去

vim ingress-myapp.yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-myapp
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: myapp.test.com
    http:
      paths:
      - path:
        backend:
          serviceName: myapp
          servicePort: 80
```

应用这个写好的yaml

```
kubectl apply -f ingress-myapp.yaml
```

查看这个创建好的ingress

```
kubectl get ingerss
```

查看具体的信息

```
kubectl describe ingress ingerss-myapp
```

验证

查看ingress的相关pod

```
kubectl get pods -n ingress-nginx
```

进pod去查看

```
kubectl exec -n ingress-nginx -it INGRESS-NGINX-NAME -- /bin/sh

进去之后查看文件
cat nginx.conf

即可看到已经自动把规则写好了
```

### 访问pod

修改本地host , (*在windows环境下用浏览器测试的话, 修改C:\Windows\System32\drivers\etc\hosts ,  要是在Linux下用浏览器测试 , 则修改/etc/hosts*)   把所在pod集群中的ip和刚刚填写的host对应起来 , 也就是myapp.test.com



之后在浏览器中输入

myapp.test.com:30080

即可访问, 查看效果







### 附录

ingress官方介绍

具体的官方文档地址为https://kubernetes.github.io/ingress-nginx/deploy/#generic-deployment