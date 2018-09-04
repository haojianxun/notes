# kubernetes中资源yaml模板创建

在部署各类资源的时候 需要自己填写各种属性 , 其实我们可以创建一个模板,创建完成之后再修改其中的值即可, 这样的方便和准确了许多

创建模板的方法

```
kubectl create SERVICENAME NAME -o yaml --dry-run > SOMENAME.yaml
```

比如创建资源认证的模板

```
kubectl create serviceaccount mysa -o yaml --dry-run > mysa.yaml
```

