# kubernetes dashboard部署

1. ### 部署

   ```
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
   
   //详细情况可以去github上去看  地址是https://github.com/kubernetes/dashboard
   ```

   

2. ### 将Service改成NodePort

   ```
   kubectl patch svc kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}' -n kube-system
   ```

3. ### 认证

   认证的账号必须是ServiceAccount, 被dashboard pod拿来由kubernetes认证

   有以下认证方法:

   **token**:

   - 创建ServiceAccount, 根据管理目标 , 使用rolebinding或者clusterrolebinding绑定至合理的role或者clusterrole
   - 获取到次ServiceAccount的secret , 查看secret的详细信息, 其中就有token

   **kubeconfig**:

   - 创建ServiceAccount, 根据管理目标 , 使用rolebinding或者clusterrolebinding绑定至合理的role或者clusterrole

   - `TOKEN=$(kubectl get secret SERVICEACCOUNT_NAME -0 jsonpath={.data.token} | base64 -d)`

     其中TOKEN是自己定义的一个变量名称, 为了方便管理token而已 , SERVICEACCOUNT_NAME是其中token的那个pod的名称 , 如何获取呢, 运行下面命令`kubectl get secret |awk '/^YOUR_SERVICEACCOUNT/{print $1}'`   比如你创建的secretaccount的名称是def-ns-admin 运行`kubectl get secret |awk '/^def-ns-admin/{print $1'}` 

   - 生成config文件

     ```
     kubectl config set-cluster
     kubectl config set-credentials NAME --token=$TOKEN
     kubectl config set-context
     kubectl config use-context
     ```

     

     

     