# kubernetes中部署prometheus

前提: 部署了metrics-server



用于测试环境的简单部署

```
git clone https://github.com/iKubernetes/k8s-prom.git
```

```
cd k8s-prom
kubectl apply -f namespace.yaml
```

```
cd prometheus
kubectl apply-f ./
```

```
cd ../node_exporter
kubectl apply -f ./
```



之后就可以访问了 端口是30090



把prometheus收集到的数据可以让自定义的api访问

```
cd k8s-prom/kube-state-metrics/
kubectl apply -f ./
```



```
cd k8s-prom/k8s-prometheus-adapter/
cat custom-metrics-apiserver-deployment.yaml   //查看最好的servicename是什么 , 记住 ,待会用
//之后要是发生部署image错误的话  可以去DirectXMan12的github上获取新的 地址是: https://github.com/DirectXMan12/k8s-prometheus-adapter/tree/master/deploy/manifests , 

获取其中的custom-metrics-config-map.yaml   讲namespace该成prom  之后应用一下

获取custom-metrics-apiserver-deployment.yaml文件   获取之后可以将其中是namespace改成我们自己的namespace  我们自定义的namespace是prom

之后应用

//如果还是出错  也可以将其都下载下来 将其中的namespace都该成prom  之后应用  

```

创建一个证书

```
cd /etc/kubernetes/pki

(umask 077;openssl genrsa -out serving.key 2048)

openssl req -new -key serving.key -out serving.csr -subj "/CN=serving"

openssl x509 -req -in serving.csr -CA ./ca.crt -CAkey ./ca.key -CAcreateserial -out serving.crt -days 3650
```

```
kubectl create secret generic cm-adapter-serving-certs --from-file=serving.crt=./serving.crt --from-file=serving.key=./serving.key -n prom
```

```
cd k8s-prom/k8s-prometheus-adapter/
kubectl aooly -f ./
```



之后查看是否部署成功

```
kubectl api-versions   //确保列出的群组中有  custom.metrics.k8s.io/v1beta1   说明我们新的api已经可以用了
```

查看资源数据

```
curl http://localhost:8080/apis/custom.metrics.k8s.io/v1beta1/     这个8080是之前开的代理 命令是kubectl proxy --port:8080
```



部署一个grafana

我们用一个现成的yaml文件来部署 ,  先下载之后对其进行修改

```
wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/grafana.yaml       
```

修改这个yaml , 以符合我们现在的情况

vim grafana.yaml 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: monitoring-grafana
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      task: monitoring
      k8s-app: grafana
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: grafana
    spec:
      containers:
      - name: grafana
        image: k8s.gcr.io/heapster-grafana-amd64:v5.0.4
        ports:
        - containerPort: 3000
          protocol: TCP
          
........
只是贴了前半段 ,因为我们修改的只有前面的内容而已 , 修改的内容也是之前一样 是为了适应这个v1.11.1  把apiVersion修改了  该成现在v1.11.1用的apps/v1   在spec下增加selector等内容   如果想外部访问  可以在最后是service服务下 增加 type:NodePort

之后把namespace都改成prom 

注释掉spec.containers.env 下的 - name: INFLUXDB_HOST   value: monitoring-influxdb 

```

查看其可以访问的端口

```
kubectl get svc -n prom
```



修改grafana的数据源

```
kubectl get svc -n prom   //查看prometheus的服务名称

所以我们就将数据源的url填成: http://prometheus.prom.svc:9090  方式是proxy

dashboard的模板可以去官网去下载 , 有很多的
```

