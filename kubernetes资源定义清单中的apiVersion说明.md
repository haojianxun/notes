# kubernetes资源定义清单中的apiVersion说明

### 什么是apiVersion

在kubernetes资源定义的时候 第一个要写的就是apiVersion , 那什么是apiVersion呢

查看当前kubernetes版本支持的apiVersion可以使用下面的命令

```
kubectl api-versions
admissionregistration.k8s.io/v1beta1
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1
apiregistration.k8s.io/v1beta1
apps/v1
apps/v1beta1
apps/v1beta2
authentication.k8s.io/v1
authentication.k8s.io/v1beta1
authorization.k8s.io/v1
authorization.k8s.io/v1beta1
autoscaling/v1
autoscaling/v2beta1
batch/v1
batch/v1beta1
certificates.k8s.io/v1beta1
events.k8s.io/v1beta1
extensions/v1beta1
networking.k8s.io/v1
policy/v1beta1
rbac.authorization.k8s.io/v1
rbac.authorization.k8s.io/v1beta1
scheduling.k8s.io/v1beta1
storage.k8s.io/v1
storage.k8s.io/v1beta1
v1
```

apiVersion定义的语法为:

`apiVersion:group/version`

可以看到上述apiVersion中都有前缀, 唯独最后一个没有 , 同时这也是我们经常用到的v1

v1是省略的写法 省略了core  因为是核心组 所以就省略 只写v1了