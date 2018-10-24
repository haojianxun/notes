# kubernetes中service的规则是由谁实现的

是由kube-proxy实现的 , kube-proxy和api-server通信 , 发现pod或者service有改变就