# kubernetes ConfigMap

## 用命令行的方式创建一个configmap

```
kubectl create configmap nginx-config --from-literal=nginx_port=80 --from-literal=server_name=myapp.test.com
```

查看刚刚创建的configmap

```
kubectl get cm
```

## 通过文件制作ConfigMap

创建一个文件

vim www.cofig

```
server {
    server_name myapp.test.com;
    listen 80;
    root /data/web/html/;
}
```

使用命令创建

```
kubectl create configmap nginx-www --from-file=./www.conf
```

启动一个容器来使用这个configmap

vim pod-cm.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-1
  namespace: default
  labels:
    app: myapp
    tier: frontend
  annotations:
    test.com/created-by: "cluster admin"
spec:
  containers:
  - name:  myapp
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80
    env:
    - name: NGINX_SERVER_PORT
      valueFrom:
        configMapKeyRef:
          name: nginx-cofig
          key: nginx_port
    - name: NGINX_SERVER_PORT
      valueFrom:
        configMapKeyRef:
          name: nginx-cofig
          key: server_name
```



也可以直接编辑

```
kubectl edit cm nginx-config
```

直接编辑configmap  这个值在env也不会改变   用env这种方式只在pod启动的时候有效, 启动的时候直接注入到环境变量里面 , 是pod创建的时候获取  ,如果用存储卷这种方法 , 当edit configmap的时候 可以实时生效

下面是个例子

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-2
  namespace: default
  labels:
    app: myapp
    tier: frontend
  annotations:
    test.com/created-by: "cluster admin"
spec:
  containers:
  - name:  myapp
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80
    volumeMounts:
    - name: nginx-config
      mountPath: /etc/nginx/conf.d
      readOnly: true
  volumes:
    name: nginxconfig
    configMap:
      name: nginx-www
```

可以进入这个pod中 编辑/data/web/html/  创建一个index.html文件  之后修改/etc/hosts文件即可访问





## 通过注入secret到pod的环境变量

```
kubectl create secret generic mysql-root-passwd --from-literal=passwd=123456
```

查看刚刚创建的secret

```
kubectl get secret
kubectl describe secret mysql_root_passwd
```

编写一个例子,来挂载刚刚创建的secret:

vim pod-secret.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret
  namespace: default
  labels:
    app: myapp
    tier: frontend
  annotations:
    test.com/created-by: "cluster admin"
spec:
  containers:
  - name:  myapp
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80
    env:
    - name: MYSQL_ROOT_PASSWD
      valueFrom:
         secretKeyRef:
           name: mysql-root-passwd
           key: passwd
```

