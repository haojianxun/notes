# 部署一个有证书的ingress-nginx

### 生成证书和secret

```
openssl genrsa -out tls.key 2048

openssl req -new -x509 -key tls.key -out tls.crt -subj /C=CN/ST=shanghai/L=shanghai/O=devops/CN=tomcat.test.com

kubectl create secret tls tomcat-ingress-secret --cert=tls.crt --key-tls.key

kubectl get secret

kubectl describe secret tomcat-ingress-secret
```



### 部署一个实例

vim tomcat-deploy.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: tomcat
  namespace: default
spec:
  selector:
    app: tomcat
    release: canary
  ports:
  - name: http
    targetPort: 8080
    port: 8080
  - name: ajp
    targetPort: 8009
    port: 8009



---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deploy
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tomcat
      release: canary
  template:
    metadata:
      labels:
        app: tomcat
        release: canary
    spec:
      containers:
      - name: myapp
        image: tomcat:8.5.32-jre8-alpine
        ports:
        - name: http
          containerPort: 8080
        - name: ajp
          containerPort: 8009

```

### 部署一个有证书的ingress

vim ingress-tomcat-tls.yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-tomcat-tls
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - tomcat.test.com
    secretName: tomcat-ingress-secret
  rules:
  - host: tomcat.test.com
    http:
      paths:
      - path:
        backend:
          serviceName: tomcat
          servicePort: 8080
```

### 应用这个yaml文件

```
kubectl apply -f tomcat-deploy.yaml
kubectl apply -f ingress-tomcat-tls.yaml
```



### 查看这个部署的ingress

```
kubectl get ingress
```

### 访问pod

在windows下测试的话 就编辑本地C:\Windows\System32\drivers\etc\hosts 

把集群节点所在ip加上刚刚的tomcat.test.com

最后在浏览器中输入

https://tomcat.test.com:30443/

效果如下:

![](http://pe9685fps.bkt.clouddn.com/18-9-2/87430346.jpg)

