## [kubernetes]Operator SDK开发简介

官方的github项目地址: https://github.com/operator-framework/operator-sdk

你可以基于Go/Ansible开发Opearator 。基于Go的典型工作流程如下：

1. 使用SDK的CLI，创建一个新的Operator项目
2. 添加CRD，并定义一个新的资源API
3. 定义监控、调和（reconcile，通过适当的操作让资源接近期望状态）资源的控制器
4. 使用SDK、controller-runtime API来开发调和逻辑
5. 使用SDK CLI来构建、生成Operator的部署清单文件

##前提条件

- [dep](https://golang.github.io/dep/docs/installation.html) version v0.5.0+.
- [git](https://git-scm.com/downloads)
- [go](https://golang.org/dl/) version v1.10+.
- [docker](https://docs.docker.com/install/) version 17.03+.
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) version v1.11.0+.
- Access to a kubernetes v.1.11.0+ cluster.





## 安装SDK CLI

```shell
mkdir -p $GOPATH/src/github.com/operator-framework
cd $GOPATH/src/github.com/operator-framework
git clone https://github.com/operator-framework/operator-sdk
cd operator-sdk
git checkout master
make dep
make install 
```



## 快速开始

###创建项目

```shell
# Create an app-operator project that defines the App CR.
# The command like this: cd $GOPATH/src/github.com/<your-github-repo>/
$ mkdir -p $GOPATH/src/github.com/example-inc/

# Create a new app-operator project
$ cd $GOPATH/src/github.com/example-inc/
# The command like this : operator-sdk new <operator-project-name> --api-version=<your-api-group>/<version> --kind=<custom-resource-kind>
$ operator-sdk new app-operator


$ cd app-operator
```

上述脚本执行后，会自动调用dep init创建项目，调用dep ensure下载依赖，并初始化Git仓库

- operator-project-name：创建的项目的名称
- your-api-group：Kubernetes 自定义 API 的组名，一般用域名
- version：Kubernetes 自定义资源的 API 版本
- custom-resource-kind：CRD 的名称

### 添加API

```shell
$ operator-sdk add api --api-version=app.example.com/v1alpha1 --kind=AppService
```

上述命令完成后，会在pkg/apis目录下自动生成一系列文件，类似于code-generator

### 添加控制器

```shell
$ operator-sdk add controller --api-version=app.example.com/v1alpha1 --kind=AppService
```

上述命令完成后，会在pkg/controller目录生成控制器源码

### 构建Operator镜像

```shell
# Build and push the app-operator image to a public registry such as quay.io
$ operator-sdk build quay.io/example/app-operator
$ docker push quay.io/example/app-operator
```

注意需要修改deploy/operator.yaml中的REPLACE_IMAGE为实际镜像名：

```shell
sed -i 's|REPLACE_IMAGE|quay.io/example/app-operator|g' deploy/operator.yaml
```

### 在本地运行

在开发期间，一般在本地运行Operator，以便快速测试和调试

```
# 通过环境变量设置Operator名称
export OPERATOR_NAME=app-operator
# 启动Operator，会自动使用$HOME/.kube/config。你可以用--kubeconfig=指定kubeconfig
operator-sdk up local --namespace=default
```

### 部署

```shell
# Setup Service Account
$ kubectl create -f deploy/service_account.yaml
# Setup RBAC
$ kubectl create -f deploy/role.yaml
$ kubectl create -f deploy/role_binding.yaml
# Setup the CRD
$ kubectl create -f deploy/crds/app_v1alpha1_appservice_crd.yaml
# Deploy the app-operator
$ kubectl create -f deploy/operator.yaml

# Create an AppService CR
# The default controller will watch for AppService objects and create a pod for each CR
$ kubectl create -f deploy/crds/app_v1alpha1_appservice_cr.yaml

# Verify that a pod is created
$ kubectl get pod -l app=example-appservice
NAME                     READY     STATUS    RESTARTS   AGE
example-appservice-pod   1/1       Running   0          1m

# Cleanup
$ kubectl delete -f deploy/crds/app_v1alpha1_appservice_cr.yaml
$ kubectl delete -f deploy/operator.yaml
$ kubectl delete -f deploy/role.yaml
$ kubectl delete -f deploy/role_binding.yaml
$ kubectl delete -f deploy/service_account.yaml
$ kubectl delete -f deploy/crds/app_v1alpha1_appservice_crd.yaml
```

开发阶段完成后，建议使用Helm Chart打包封装

###项目录结构

基于operator-sdk生成的Operator项目，其目录结构如下

| 目录/文件      | 说明                                                         |
| -------------- | :----------------------------------------------------------- |
| cmd            | manager/main.go为主函数，它：                                                                                                1. 初始化一个新的Manager                                                                                                            2. 注册所有pkg/apis下定义的CR                                                                                                    3. 启动所有pkg/controllers下定义的控制器 |
| pkg/apis       | 包含CRD的API，开发人员需要编辑pkg/apis/GROUP/VERSION/KIND_types.go文件，为资源添加必要的字段 |
| pkg/controller | 包含控制器的实现，开发人员需要编辑pkg/controller/KIND/KIND_controller.go，实现KIND资源的Reconcile逻辑 |
| build          | 包含Dockerfile，以及构建Operator所需的脚本                   |
| deploy         | YAML形式的K8S资源定义文件，用于注册CRD、创建RABC、Deployment等资源 |
| Gopkg.*        | Go Dep的清单文件                                             |
| vendor         | 为了满足项目import需要的外部依赖的副本                       |

### Manager

operator的主要程序是由 `cmd/manager/main.go` 初始化的 , 并且运行一个[Manager](https://godoc.org/github.com/kubernetes-sigs/controller-runtime/pkg/manager#Manager).

管理器将自动为 `pkg/apis/...`下定义的所有自定义资源注册方案，并在 `pkg/controller/...`.下运行所有控制器

Manager可以限制所有控制器可以监控的命名空间

```go
mgr, err := manager.New(cfg, manager.Options{Namespace: namespace})
```

默认情况下监控Operator所在的命名空间，要监控所有命名空间，可以

```go
mgr, err := manager.New(cfg, manager.Options{Namespace: ""})
```

### 增加一个新的CRD

增加一个新的Custom Resource Definition(CRD) API , 名称叫做Memcached , APIVersion 是`cache.example.com/v1apha1`   , Kind 是`Memcached`.

```shell
$ operator-sdk add api --api-version=cache.example.com/v1alpha1 --kind=Memcached
```

这将在`pkg/apis/cache/v1alpha1/...`.下搭建Memcached资源API

