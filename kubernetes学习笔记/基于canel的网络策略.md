# 基于canel的网络策略

### Installing Calico for policy and flannel for networking

If your cluster has RBAC enabled, issue the following command to configure the roles and bindings that Calico requires.

```
kubectl apply -f \
https://docs.projectcalico.org/v3.2/getting-started/kubernetes/installation/hosted/canal/rbac.yaml
```



Issue the following command to install Calico.

```
kubectl apply -f \
https://docs.projectcalico.org/v3.2/getting-started/kubernetes/installation/hosted/canal/canal.yaml
```



查看网络怎么定义

```
kubectl explain networkplocy
```

---



#### 实验前提

先创建2个名称空间

```
kubectl create namespace dev
kubectl create namespace prod
```

---



## ingress进站规则

### 创建一条ingress规则

vim ingress-def.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPlicy
metadata:
  name: deny-all-ingress
spec:
 podSelector: {}
 plicyTypes:
 - Ingress
```

```
kubectl apply -f ingress-def.yaml -n dev   //在dev名称空间下生效
```

#### 查看刚刚创建的规则

```
kubectl get netpol -n dev
```

#### 创建一个测试用的pod

```
vim pod-dev-test.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-dev-test
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
```

```
kubectl apply -f pod-dev-test.yaml -n dev
```

##### 查看这个pod并访问

```
kubectl get pods -n dev -o wide   //找到这个pod的IP并用curl访问 , 发现并不能访问, 因为刚刚定义的ingress规则就是拒绝所有访问
```

##### 同样的把这个pod部署在名称空间为`prod` 下, 就不会有这种情况

```
kubectl apply -f pod-dev-test.yaml -n prod
```

##### 查看这个pod并访问

```
kubectl get pods -n prod -o wide   //找到这个pod的IP并用curl访问 , 可以访问, 因为我们没有在这个空间下定义ingress拒绝所有的规则
```

---

### **更改ingress规则**

vim ingress-def.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPlicy
metadata:
  name: allow-all-ingress
spec:
 podSelector: {}
 ingress:
 - {}
 plicyTypes:
 - Ingress
```

```
kubectl apply -f ingress-def.yaml -n dev //将刚刚修改完的规则放在dev下生效
```



#### **查看这个pod并访问**

```
kubectl get pods -n dev -o wide   //找到这个pod的IP并用curl访问 , 这回发现就可以访问了
```



### 控制特定网段和端口的ingress进站访问

先把之前定义的pod先打标签

```
kubectl label pods pod-dev-test app=myapp -n dev
```



vim allow-netpol-demo.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-myapp-ingress
spec:
  podSelector:
    matchLabels:
      app: myapp
  ingress:
  - from:
    - ipBlock:
        cidr: 10.244.0.0/16
        except:
        - 10.244.1.2/32
    ports:
    - protocol: TCP
      port: 80
```

```
kubectl apply -f allow-netpol-demo.yaml -n dev
```

查看pod并访问

```
kubectl get pods -n dev -o wide  //查看pod地址并访问 , 发现除了10.244.1.2访问不了, 其他都可以 , 而且 端口也只能访问80 , 其他端口不能访问
```



**综上:**

**ingress规则如果没有显示的定于出来 , 那么就是默认拒绝所有  在policytypes中没有规定的策略 那么就是接受所有**

对于名称空间

   拒绝所有出入站

   放行出入站为本名称空间的所有pod

对于跨名称空间来说 再单独定义



## 定义egress规则

玩法同ingress相同