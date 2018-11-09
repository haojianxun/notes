# 在kubernetes中如何给pod自动填充某些字段

在kubernetes的v1.11 版本 , 有个 PodPreset（Pod 预设置）的功能

在PodPreset 对象中，凡是想在Pod 里追加的字段，都可以预先定义好 , 比如:

vim podpreset.yaml

```yaml
apiVersion: settings.k8s.io/v1alpha1
kind: PodPreset
metadata:
  name: allow-database
spec:
  selector:
    matchLabels:
      role: frontend
  env:
    - name: DB_PORT
      value: "6379"
  volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
    - name: cache-volume
      emptyDir: {}
```

在这个 PodPreset 的定义中，首先是一个 selector。这就意味着后面这些追加的定义，只会作用于 selector 所定义的、带有“role: frontend”标签的 Pod 对象，这就可以防止“误伤”。

先创建这个yaml文件

```bash
$ kubectl apply -f podpreset.yaml
```



之后要说开发人员向创建一个pod , 他就可以填些简单的字段 , 比如:

vim website.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: website
  labels:
    app: website
    role: frontend
spec:
  containers:
    - name: website
      image: nginx
      ports:
        - containerPort: 80
```

```bash
$ kubectl apply -f website.yaml
```



运行起来以后 可以看到这个pod的yaml文件已经和之前创建的podpreset.yaml文件合并了

```bash
$ kubectl get pod website -o yaml
apiVersion: v1
kind: Pod
metadata:
  name: website
  labels:
    app: website
    role: frontend
  annotations:
    podpreset.admission.kubernetes.io/podpreset-allow-database: "resource version"
spec:
  containers:
    - name: website
      image: nginx
      volumeMounts:
        - mountPath: /cache
          name: cache-volume
      ports:
        - containerPort: 80
      env:
        - name: DB_PORT
          value: "6379"
  volumes:
    - name: cache-volume
      emptyDir: {}
```

