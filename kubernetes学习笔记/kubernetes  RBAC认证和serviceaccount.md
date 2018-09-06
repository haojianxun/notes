# kubernetes  认证和serviceaccount

### 创建一个serviceaccount

```
kubectl create serviceaccount admin
```

**查看这个刚刚创建的sa**

```
kubectl get sa admin
```

可以看到我们刚刚创建的一个serviceaccount , 系统会自动为其生成一个token

**查看这个自动生成的token**

```
kubectl get secret
```

**创建一个示例**

vim pod-sa-demo.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-sa-demo
  namespace: default
  labels:
    app: myapp
    tier: frontend
  annotations:
    test.com/created-by: "cluster admin"
spec:
  containers:
  - name: myapp
    image: ikubernetess/myapp:v1
    ports:
    - name: http
      containerPort: 80
  serviceAccountName: admin
  
```

**应用这个yaml**

```
kubectl apply -f pod-sa-demo.yaml
```

**查看各个创建的pod**

```
kubectl describe pods pod-sa-demo
```



### 创建kubernetes认证的新用户

```
cd /etc/kubernetes/pki
```

```
(umask 077; openssl genrsa -out test.key 2048)
```

```
openssl req -new -key test.key -out test.csr -subj "/CN=test"
```

```
openssl x509 -req -in test.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out test.crt -days 365
```

**查看刚刚创建的证书内容**

```
openssl x509 -in test.crt -text -noout
```



```
kubectl config set-credentials test --client-certificate=./test.crt --client-key=./test.key --embed-certs=true
```

**完成之后查看这个config**

```
kubectl config view
```



```
kubectl config set-context test@kubernetes --cluster=kubernetes --user=test
```

*注意:上述 `test@kubernetes` 中 test是用户名 kubernetes是集群名称* 



**切换到test用户**

```
kubectl config use-context test@kubernetes
```



### 设定一个新集群

```
kubectl config set-cluster --help   //查看帮助 
```



```
kubectl config set-cluster mycluster --kubeconfig=/tmp/test.conf --server="https://192.168.200.140:6443" --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true
```



查看刚刚创建的新机器配置文件

```
kubectl config view --kubeconfig=/tmp/test.conf
```

