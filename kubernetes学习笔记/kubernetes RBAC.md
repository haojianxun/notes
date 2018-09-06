

# kubernetes RBAC

> RBAC: Role Based Access Control 
>
> Role Based Access Control is comprised of four layers:
>
> 1. `ClusterRole` - permissions assigned to a role that apply to an entire cluster
> 2. `ClusterRoleBinding` - binding a ClusterRole to a specific account
> 3. `Role` - permissions assigned to a role that apply to a specific namespace
> 4. `RoleBinding` - binding a Role to a specific account
>
> In order for RBAC to be applied to an nginx-ingress-controller, that controller should be assigned to a `ServiceAccount`. That `ServiceAccount` should be bound to the `Role`s and `ClusterRole`s defined for the nginx-ingress-controller.



**资源访问**

object URL

- `/apis/<GROUP>/<VERSION>/namespace/<NAMESPACE_NAME>/<KIND>[/OBJECT_ID]` 

HTTP requsets verb

- `get`  , `post`  , `put`  , `delete` 

API requetes verb

- `get`  ,`list`  , `create`  , `update`  , `patch`  , `watch`  , `proxy`  , `redirect`  , `delete` `deletecollection` 



### Role

- operations
- objects

角色的访问控制, 要定义以上2个 即: 有什么权限 , 要访问什么资源  默认都是拒绝的 , 所以, 我们在定义权限的时候, 定义的内容是`允许权限`  , 没有明确定义出来的都是拒绝 



### Rolebinding

- user account OR serviceaccount
- role

角色的绑定是要先有一个角色 , 之后把这个`user account OR serviceaccount` 绑定在`Role` 上  , 而这个

`Role`是之前就定义好的 , 上面规定了这个角色有哪些`权限`  ,  这样这个账号通过角色的绑定间接获得了权限

注意: 同时`Rolebinding` 是基于某个名称空间的  , `Role`的权限只是基于某个名称空间的



示例

---

## 创建一个role

```
kubectl create role pods-reader --verb=get,list,watch --resource=pods
```

也可以创建出了定义role的模板文件, 用于以后的定义

```
kubectl create role pods-reader --verb=get,list,watch --resource=pods --dry-run -o yaml > role-demo.yaml
```



vim role-demo.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: pods-reader
  namespace: default
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
```

查看刚刚创建的role

```
kubectl describe role pods-reader
```



### 创建一个rolebinding

```
kubectl create rolebinding test-read-pods --role=pods-reader --user=test
```

查看刚刚创建的rolebinding

```
kubectl describe rolebinding test-read-pods
```

切换用户查看效果

```
kubectl config use-context test@kubernetes

kubectl get pods   //发现这个用户可以查看当前用户空间的pod了
kubectl get pods -n kube-system  //查看kube-system空间的pod , 发现就不行 ,因为没有授权
```

### 删除一个rolebinding

```
kubectl delete rolebinding test-read-pods
```



## 创建一个ClusterRole

vim cluserrole-demo.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
```

```
kubectl apply -f clusterrole-demo.yaml
```

创建一个clusterrolebinding

```
kubectl create clusterrolebinding test-read-all-pods --clusterrole=cluster-reader --user=test
```



## 

## 用`ClusterRoleBinding` 来绑定一个`ClusterRole`  

```
kubectl create clusterrolebinding test-read-all-pods --clusterrole=cluster-reader --user=test --dry-run -o yaml > clusterrolebinding-demo.yaml
```

```yaml
vim clusterrolebinding-demo.yaml

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: test-read-all-pods
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-reader
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: test
```

```
kubectl apply -f clusterrolebinding-demo.yaml
```

查看刚刚创建的clusterrolebinding

```
kubectl describe clusterrolebinding test-read-all-pods
```

查看效果

```
kubectl config use-context test@kubernetes    //切换到test用户

kubectl get pods   //发现可以查看
kubectl get pods -n kube-system  //发现也可以查看,之前rolebinding是不可以查看的,现在可以了
```

查看集群中所有的clusterrole

```
kubectl get clusterrole
```



## 用`RoleBinding` 来绑定一个`ClusterRole` 

先把刚刚创建好的`ClusterRoleBinding` 删除了

```
kubectl delete clusterrolebinding test-read-all-pods
```

创建一个rolebinding

```
kubectl create rolebinding test-read-pods --clusterrole=cluster-reader --user=test
```

当然也可以用文件来创建

```
kubectl create rolebinding test-read-pods --clusterrole=cluster-reader --user=test --dry-run -o yaml > rolebinding-clusterrole.yaml
```



vim  rolebinding-clusterrole.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: test-read-pods
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-reader
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: test
```

```
kubectl apply -f rolebinding-clusterrole.yaml
```

查看效果

```
kubectl config use-context test@kubernetes  //切换到test用户

kubectl get pods   //发现可以查看
kubectl get pods -n ingress-nginx  
//发现不能查看 , 因为是rolebinding , 所以clusterrole的权限被降级了, rolebinding绑定的role只能在某个名称空间下拥有指定的权限 ,不能查看全部 ,虽然绑定的是clusterrole
```

