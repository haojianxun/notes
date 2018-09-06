# pod-demo.yaml

```yaml
apiVersion: v1
kind: Pod
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
    image: ikubernetess/myapp:v1
    ports:
    - name: http
      containerPort: 80
    - name: https
      containerPort: 443
  - name: busybox
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command:
    - "/bin/sh"
    - "-c"
    - "sleep 3600"
  nodeSelector:
    disktype: ssd
  
```

