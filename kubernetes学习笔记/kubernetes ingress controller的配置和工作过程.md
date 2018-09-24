# kubernetes ingress controller的配置和工作过程

我们暴露外部服务一般用的是NodePort , 但是nodeport暴露服务有一个问题  , 就是每个node都会占一个端口去暴露, 而且前面没有统一的服务入口 没有负载均衡 ,nodeport是基于4层调度 这也是问题

NodePort的访问路径是

**NodePort: client----->NodeIP:NodePort----->ClusterIP:ServicePort---->PodIP:containerPort**

在访问`NodeIP:NodePort` 的时候 , 要加一个负载均衡

我们可以部署一个基于七层的pod调度器 , 运行在用户空间 , 来接管访问的流量 , 监听在宿主机的特定端口也可以将其部署成daemonset模式 同享宿主机网络 这就是ingress-nginx

ingress-nginx里面有nginx也有ingress controller 它可以根据ingress规则来动态的生成nginx配置文件



将ingress-nginx部署成daemonset模式

可以将ingress controller运行为DaemonSet , 也就是每个节点运行这样一个副本 , 如果我们有3千个这样的节点的话 , 可以只挑出3个节点 , 将他们打上污点 , 别的pod不能运行在这节点上 , 只运行ingress controller



部署ingress-nginx, 外部用户访问内部pod的路径

externalLB------><service> ingress-nginx [NodePort类型]-----><IngressController>ingress-nginx(当然也可以运行成共享节点网络空间 或者是部署成为DaemonSet , 工作在用户空间 , 具有七层协议 , 它成了http/https会话卸载器了)------>根据ingress的规则来选择后端pod(可以根据虚拟主机或者url来路由) 这个后端pod可以根据service来进行



如何部署为DaemonSet

在部署的过程中 , 在应用`with-rbac.yaml`文件的时候 , 可以手动对其进行改造

文件的地址是: https://github.com/kubernetes/ingress-nginx/blob/master/deploy/with-rbac.yaml

修改以下几点:

- 将其中的`kind: Deployment`改成DaemonSet  
- 去掉`spec`下的`replicas: 1`   (因为DaemonSet不需要replicas)
- 在`template.spec`下加个`hostNetwork`     (共享宿主机的网络名称空间)

