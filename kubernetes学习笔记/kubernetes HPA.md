# kubernetes HPA

查看帮助

```
kubectl explain hpa
```



先创建一个有资源限制的pod

```
kubectl run myapp --image=ikubernetes/myapp:v1 --replicas=1 --requests='cpu=50m,memory=256Mi' --limits='cpu=50m,memory=256Mi' --labels='app=myapp' --expose --port=80
```

创建一个HPA

```
kubectl autoscale --help  //查看帮助
```

```
kubectl autoscale deployment myapp --min=1 --max=8 --cpu-percent=60
```

```
kubectl patch svc myapp -p '{"spec":{"type":"NodePort"}}'
```

```
kubectl get svc  //查看这个服务端口
```

```
yum install -y httpd-tools
ab -c 1000 -n 500000 http://IP:PORT/index.html
```



查看效果

```
kubectl  get pods
kubectl describe hpa
```





vim myapp-hpa-demo.yaml

```yaml
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: mapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 55
  - type: Resource
    resource:
      name: memory
      targetAverageValue: 50Mi
```

应用这些文件

```
kubectl apply -f myapp-hpa-demo.yaml
```

压测

```
yum install -y httpd-tools
ab -c 1000 -n 500000 http://IP:PORT/index.html
```



查看效果

```
kubectl  get pods
kubectl describe hpa
```

可以看到会自动扩展



比如其他的指标

```
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: mapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metricName: http_requests
      targetAverageValue: 800m
```

