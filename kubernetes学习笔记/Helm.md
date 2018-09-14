## Helm

concept

chart: 一个helm程序包

repository: chart仓库 , http/https服务器

release: chart部署在目标集群上的一个实例

chart------->config-------->release



程序架构

helm: 客户端 , 管理本地chart仓库  和 tiller服务器交互 实例安装 查询 卸载等操作

tiller: 服务端 , 接收helm发来的chart和config 合并生成release



项目的github地址: https://github.com/helm/helm

官网: https://helm.sh

helm官方仓库地址:https://hub.kubeapps.com/



下载安装包

```
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.11.0-rc.2-linux-amd64.tar.gz

tar xvzf 

mv helm /usr/bin/

helm --help  //查看帮助
```



RBAC部署文件示例

https://github.com/helm/helm/blob/master/docs/rbac.md

### Example: Service account with cluster-admin role

In `rbac-config.yaml`:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

*Note: The cluster-admin role is created by default in a Kubernetes cluster, so you don't have to define it explicitly.*

```
$ kubectl create -f rbac-config.yaml
serviceaccount "tiller" created
clusterrolebinding "tiller" created
$ helm init --service-account tiller
```



helm用法

```
helm search jenkins   //搜索jenkins
helm inspect stable/jenkins   //查看stable/jenkins的具体配置信息
helm install --name mem1  stable/memcached
helm delete mem1  //删除这个名为mem1的release
```

helm命令

release管理:

- install
- delete
- upgrade/rollback

- list



chart管理

- create
- fetch
- get
- inspect
- package
- verify



下载好的东西在当前目录的`.helm` 下

