# kubernetes部署dashboard(nginx-ingress模式访问)

先下载官方的部署文件

```
wget https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

可以将其中的镜像pull地址改成自己的

之后在末尾加上ingress规则(以下内容补在官方yaml部署文件的最后 , 追加信息)

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kube-system
spec:
  rules:
  - host: dashboard.hjx.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kubernetes-dashboard
          servicePort: 443
```

再之后部署Nginx-ingress     这样就能通过访问域名来访问dashboard的了