# kubernetes存储卷

## emptyDir类型

```
apiVersion: v1
kind: pod
metadata:
  name: pod-demo
  namespace: default
  labels:
    app: myapp
    tier: frontend
  annotations:
    test.com/created-by: "cluster admin"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html/
  - name: busybox
    image:busybox:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: html
      mountPath: /data
    command:
    - "/bin/sh"
    - "-c"
    - "while true; do echo $(data) >> /data/index.html; sleep 2; done"
  volumes:
  - name: html
    emptyDir:{}
    
```



## hostPath类型

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-vol-hostpath
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  volumes:
  - name: html
    hostPath:
     path: /data/pod/volumel
     type: DirectoryOrCreate
```

之后可以

```
mkdir -pv /data/pod/volumel

vim /data/pod/volumel/index.html

hello,node01   //可以在各个节点上创建
```



## NFS类型

```
stor01作为存储节点
yum install -y nfs-utils

mkdir -p /data/volumes

vim /etc/exports

/data/volumes 192.168.0.0/16(rw,no_root_squash)

systemctl start nfs

ss -tnl



在node02上
yum install -y nfs-utils

```

编辑一个yaml文件

vim pod-vol-nfs.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-vol-nfs
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  volumes:
  - name: html
    nfs:
     path: /data/volumes
     server: stor01.test.com
```

之后应用这个yaml

```
kubectl apply -f pod-vol-nfs.yaml
```

我们回到stor01上

```
vim /data/volumes/index.html

nfs stor01
```

之后访问这个pod

```
kubectl get pods -o wide

之后找到ip地址 即可访问
```



## PV和PVC类型

```
mkdir -p /data/volumes/v{1,2,3,4,5}
```

vim /etc/exports

```
/data/volumesv1	192.168.0.0/16(rw,no_root_squash)
/data/volumesv2 192.168.0.0/16(rw,no_root_squash)
/data/volumesv3 192.168.0.0/16(rw,no_root_squash)
/data/volumesv4 192.168.0.0/16(rw,no_root_squash)
/data/volumesv5 192.168.0.0/16(rw,no_root_squash)
```

```
exportfs -arv

showmount -e
```



## 定义一个NFS类型的PV

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv001
  labels:
    name: pv001
spec:
  nfs:
    path: /data/volumes/v1
    server: stor01.test.com
  accessModes: ["ReadWriteMany","ReadyWriteOnce"]
  capacity:
    storage: 2Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv002
  labels:
    name: pv002
spec:
  nfs:
    path: /data/volumes/v2
    server: stor01.test.com
  accessModes: ["ReadyWriteOnce"]
  capacity:
    storage: 2Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv003
  labels:
    name: pv003
spec:
  nfs:
    path: /data/volumes/v3
    server: stor01.test.com
  accessModes: ["ReadWriteMany","ReadyWriteOnce"]
  capacity:
    storage: 10Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv004
  labels:
    name: pv004
spec:
  nfs:
    path: /data/volumes/v4
    server: stor01.test.com
  accessModes: ["ReadWriteMany","ReadyWriteOnce"]
  capacity:
    storage: 10Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv005
  labels:
    name: pv005
spec:
  nfs:
    path: /data/volumes/v5
    server: stor01.test.com
  accessModes: ["ReadWriteMany","ReadyWriteOnce"]
  capacity:
    storage: 20Gi
```

创建一个PVC

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
  namespace: default
spec:
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 6Gi   
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-vol-nfs
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  volumes:
  - name: html
    persistentVolumeClaim:
      name: mypvc
```

