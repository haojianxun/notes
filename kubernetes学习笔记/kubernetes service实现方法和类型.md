# kubernetes service实现方法和类型

kube-proxy一直watch着api-service的变动 , 如果监听到了api-service关于service的任何变动 , kube-proxy就调度给后端



## service实现方式

- userspace:  

  请求------->serviceIP(iptables)----->kube-proxy(kube-proxy是运行在用户空间的)---->serviceIP(iptables)----->backend(后端pod)

- iptables

  请求------>iptables-------->pod

- ipvs

  请求------->iptables------->pod



## service类型

- `ClusterIP`：通过集群的内部 IP 暴露服务，选择该值，服务只能够在集群内部可以访问，这也是默认的 `ServiceType`。
- `NodePort`：通过每个 Node 上的 IP 和静态端口（`NodePort`）暴露服务。`NodePort` 服务会路由到 `ClusterIP` 服务，这个 `ClusterIP` 服务会自动创建。通过请求 `<NodeIP>:<NodePort>`，可以从集群的外部访问一个 `NodePort` 服务。
- `LoadBalancer`：使用云提供商的负载局衡器，可以向外部暴露服务。外部的负载均衡器可以路由到 `NodePort` 服务和 `ClusterIP` 服务。
- `ExternalName`：通过返回 `CNAME` 和它的值，可以将服务映射到 `externalName` 字段的内容（例如， `foo.bar.example.com`）。 没有任何类型代理被创建，这只有 Kubernetes 1.7 或更高版本的 `kube-dns` 才支持

---



### 创建一个nodeport

首先`spec.type`要定义为`NodePort`  ,  之后`spec.ports` 增加`nodePort` 对象  , 如果不指定`nodePort` 的话, 则 ,系统会默认分配一个随机端口(默认范围: 30000-32767)



### service的负载均衡

默认service的负载均衡是轮询 , 可以改变`spec.sessionAffinity` 来改变策略  `sessionAffinity`默认是`None`  改成`ClientIP`  这样就会变成去\同一个请求调度到同一个后端pod上去



### headless service

有时不需要或不想要负载均衡，以及单独的 Service IP。 遇到这种情况，可以通过指定 Cluster IP（`spec.clusterIP`）的值为 `"None"` 来创建 `Headless` Service

对这类 `Service` 并不会分配 Cluster IP，kube-proxy 不会处理它们，而且平台也不会为它们进行负载均衡和路由。 DNS 如何实现自动配置，依赖于 `Service` 是否定义了 selector。

- 配置 Selector

  对定义了 selector 的 Headless Service，Endpoint 控制器在 API 中创建了 `Endpoints` 记录，并且修改 DNS 配置返回 A 记录（地址），通过这个地址直接到达 `Service` 的后端 `Pod` 上。

- 不配置 Selector

  对没有定义 selector 的 Headless Service，Endpoint 控制器不会创建 `Endpoints` 记录。 然而 DNS 系统会查找和配置，无论是：

  - ExternalName类型 Service 的 CNAME 记录
    - 记录：与 Service 共享一个名称的任何 `Endpoints`，以及所有其它类型

例子:

vim myapp-headless-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-headless-svc
spec:
  selector:
    app: myapp
    release: canary
  clusterIP: None
  ports:
  - port: 80
    targetPort: 80
```

```
kubectl apply -f myapp-headless-svc.yaml
```

查看集群dns服务地址

```
kubectl get svc -n kube-system   //查看kube-dns的clusterIP是多少  我的是10.96.0.10
```

解析刚刚的headless service

```
dig -t A myapp-headless-svc.default.svc.cluster.local. @10.96.0.10
```

解析出来的时候会看到这些ip就是那些pod的ip地址  ( 这些pod的ip地址可以用命令`kubectl get pods -o wide -l app=myapp` 来查看 )