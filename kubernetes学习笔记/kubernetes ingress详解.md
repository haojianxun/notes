# kubernetes ingress详解

NodePort的访问路径是

**NodePort: client----->NodeIP:NodePort----->ClusterIP:ServicePort---->PodIP:containerPort**

在访问`NodeIP:NodePort` 的时候 , 要加一个负载均衡



可以将ingress controller运行为DaemonSet , 也就是每个节点运行这样一个副本 , 如果我们有3千个这样的节点的话 , 可以只挑出3个节点 , 将他们打上污点 , 别的pod不能运行在这节点上 , 只运行ingress controller