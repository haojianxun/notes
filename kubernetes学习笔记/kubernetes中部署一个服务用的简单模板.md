#kubernetes中部署一个服务用的简单模板

```
apiVersion: v1
kind: Service
metadata:
  name:
  namespace:
spec:
  selector:
    app:
  ports:
    name:
    targetPort:
    port:
    
    
 ---
 apiVersion: apps/v1
 kind: Deployment
 metadata:
   name:
   namespace:
 spec:
   replicas:
   selector:
     matchLabels:
       app:
   template:
     metadata:
       labels:
         app:
     spec:
       containers:
       - name:
         image:
         ports:
         - name:
           containerPort:

```

