# 在kubernetes中如何添加CRD(Custom Resource Definition)(1)

CustomResourceDefinition（CRD）是 v1.7 + 新增的无需改变代码就可以扩展 Kubernetes API 的机制，用来管理自定义对象。它实际上是 ThirdPartyResources（TPR） 的升级版本，而 TPR 已经在 v1.8 中删除

**一些使用场景：**

- 提供/管理外部数据存储/数据库(例如 CloudSQL/RDS 实例)
- 对k8s基础资源进行更高层次的抽象(比如定义一个etcd集群)

其实crd在很多k8s周边开源项目中有使用，比如ingress-controller和众多的operator。

官方的参考文档地址:https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/

更多参考文章:https://blog.openshift.com/kubernetes-deep-dive-code-generation-customresources/

官方给出的项目地址:https://github.com/kubernetes/code-generator

官方文档给出的样例是

```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: crontabs.stable.example.com
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: stable.example.com
  # version name to use for REST API: /apis/<group>/<version>
  version: v1
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: crontabs
    # singular name to be used as an alias on the CLI and for display
    singular: crontab
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: CronTab
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
    - ct
```



我们参考官方样例  ,  我们以添加network为例

欲达到这个效果

```yaml
apiVersion: samplecrd.k8s.io/v1
kind: Network
metadata:
  name: example-network
spec:
  cidr: "192.168.0.0/16"
  gateway: "192.168.0.1"
```

应用这个yaml文件之后会创建一个network的资源



##例子1: 自定义network资源类型

###创建CRD

```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  # CRD名称必须是: <plural>.<group>格式
  name: networks.samplecrd.k8s.io
spec:
  # 决定了REST API路径：/apis/<group>/<version>
  group: samplecrd.k8s.io
  version: v1
  names:
    # 驼峰式大小写的，正式的资源类型
    kind: Network
    # 决定了REST API路径： /apis/<group>/<version>/<plural>
    plural: networks
  scope: Namespaced

```



在这个自定义资源中 , 我们指定了资源的组是`samplecrd.k8s.io`  , 版本是`v1`  , 他是scrop是namespace , 意思是这个资源是namespace的对象 , 类似于pod



###定义其中的`cidr`和`gateway` 

```bash
$ tree $GOPATH/src/github.com/<your-name>/k8s-controller-custom-resource
.
├── controller.go
├── crd
│   └── network.yaml
├── example
│   └── example-network.yaml
├── main.go
└── pkg
    └── apis
        └── samplecrd
            ├── register.go
            └── v1
                ├── doc.go
                ├── register.go
                └── types.go
```

其中，pkg/apis/samplecrd 就是 API 组的名字，v1 是版本，而 v1 下面的 types.go 文件里，则定义了 Network 对象的完整描述

**在 pkg/apis/samplecrd 目录下创建了一个 register.go 文件，用来放置后面要用到的全局变量**。这个文件的内容如下所示：

```go
package samplecrd
 
const (
 GroupName = "samplecrd.k8s.io"
 Version   = "v1"
)
```

**在 pkg/apis/samplecrd 目录下添加一个 doc.go 文件（Golang 的文档源文件）**。这个文件里的内容如下所示：

```go
// +k8s:deepcopy-gen=package
 
// +groupName=samplecrd.k8s.io
package v1
```

在这个文件中，你会看到 +<tag_name>[=value] 格式的注释，这就是 Kubernetes 进行代码生成要用的 Annotation 风格的注释。

其中，+k8s:deepcopy-gen=package 意思是，请为整个 v1 包里的所有类型定义自动生成 DeepCopy 方法；而`+groupName=samplecrd.k8s.io`，则定义了这个包对应的 API 组的名字。

可以看到，这些定义在 doc.go 文件的注释，起到的是全局的代码生成控制的作用，所以也被称为 Global Tags。

**添加 types.go 文件**。顾名思义，它的作用就是定义一个 Network 类型到底有哪些字段（比如，spec 字段里的内容）。这个文件的主要内容如下所示：

```go
package v1
...
// +genclient
// +genclient:noStatus
// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
 
// Network describes a Network resource
type Network struct {
 // TypeMeta is the metadata for the resource, like kind and apiversion
 metav1.TypeMeta `json:",inline"`
 // ObjectMeta contains the metadata for the particular object, including
 // things like...
 //  - name
 //  - namespace
 //  - self link
 //  - labels
 //  - ... etc ...
 metav1.ObjectMeta `json:"metadata,omitempty"`
 
 Spec networkspec `json:"spec"`
}
// networkspec is the spec for a Network resource
type networkspec struct {
 Cidr    string `json:"cidr"`
 Gateway string `json:"gateway"`
}
 
// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
 
// NetworkList is a list of Network resources
type NetworkList struct {
 metav1.TypeMeta `json:",inline"`
 metav1.ListMeta `json:"metadata"`
 
 Items []Network `json:"items"`
}
```

在上面这部分代码里，你可以看到 Network 类型定义方法跟标准的 Kubernetes 对象一样，都包括了 TypeMeta（API 元数据）和 ObjectMeta（对象元数据）字段。

其中的 Spec 字段，就是需要我们自己定义的部分。所以，在 networkspec 里，我定义了 Cidr 和 Gateway 两个字段。其中，每个字段最后面的部分比如`json:"cidr"`，指的就是这个字段被转换成 JSON 格式之后的名字，也就是 YAML 文件里的字段名字

此外，除了定义 Network 类型，你还需要定义一个 NetworkList 类型，用来描述**一组 Network 对象**应该包括哪些字段。之所以需要这样一个类型，是因为在 Kubernetes 中，获取所有 X 对象的 List() 方法，返回值都是List 类型，而不是 X 类型的数组。这是不一样的。

同样地，在 Network 和 NetworkList 类型上，也有代码生成注释。

其中，+genclient 的意思是：请为下面这个 API 资源类型生成对应的 Client 代码（这个 Client，我马上会讲到）。而 +genclient:noStatus 的意思是：这个 API 资源类型定义里，没有 Status 字段。否则，生成的 Client 就会自动带上 UpdateStatus 方法。

如果你的类型定义包括了 Status 字段的话，就不需要这句 +genclient:noStatus 注释了。比如下面这个例子：

```
// +genclient
 
// Network is a specification for a Network resource
type Network struct {
 metav1.TypeMeta   `json:",inline"`
 metav1.ObjectMeta `json:"metadata,omitempty"`
 
 Spec   NetworkSpec   `json:"spec"`
 Status NetworkStatus `json:"status"`
}
```

需要注意的是，+genclient 只需要写在 Network 类型上，而不用写在 NetworkList 上。因为 NetworkList 只是一个返回值类型，Network 才是“主类型”。

由于我在 Global Tags 里已经定义了为所有类型生成 DeepCopy 方法，所以这里就不需要再显式地加上 +k8s:deepcopy-gen=true 了。当然，这也就意味着你可以用 +k8s:deepcopy-gen=false 来阻止为某些类型生成 DeepCopy。

你可能已经注意到，在这两个类型上面还有一句`+k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object`的注释。它的意思是，请在生成 DeepCopy 的时候，实现 Kubernetes 提供的 runtime.Object 接口。否则，在某些版本的 Kubernetes 里，你的这个类型定义会出现编译错误。这是一个固定的操作，记住即可

**最后，我需要再编写的一个 pkg/apis/samplecrd/v1/register.go 文件**。

在前面对 APIServer 工作原理的讲解中，我已经提到，“registry”的作用就是注册一个类型（Type）给 APIServer。其中，Network 资源类型在服务器端的注册的工作，APIServer 会自动帮我们完成。但与之对应的，我们还需要让客户端也能“知道”Network 资源类型的定义。这就需要我们在项目里添加一个 register.go 文件。它最主要的功能，就是定义了如下所示的 addKnownTypes() 方法：

```
package v1
...
// addKnownTypes adds our types to the API scheme by registering
// Network and NetworkList
func addKnownTypes(scheme *runtime.Scheme) error {
 scheme.AddKnownTypes(
  SchemeGroupVersion,
  &Network{},
  &NetworkList{},
 )
 
 // register the type in the scheme
 metav1.AddToGroupVersion(scheme, SchemeGroupVersion)
 return nil
}
```

有了这个方法，Kubernetes 就能够在后面生成客户端的时候，“知道”Network 以及 NetworkList 类型的定义了。

像上面这种**register.go 文件里的内容其实是非常固定的，你以后可以直接使用我提供的这部分代码做模板，然后把其中的资源类型、GroupName 和 Version 替换成你自己的定义即可**

接下来，我就要使用 Kubernetes 提供的代码生成工具，为上面定义的 Network 资源类型自动生成 clientset、informer 和 lister。其中，clientset 就是操作 Network 对象所需要使用的客户端

这个代码生成工具名叫`k8s.io/code-generator`，使用方法如下所示：

```bash
# 代码生成的工作目录，也就是我们的项目路径
$ ROOT_PACKAGE="github.com/resouer/k8s-controller-custom-resource"
# API Group
$ CUSTOM_RESOURCE_NAME="samplecrd"
# API Version
$ CUSTOM_RESOURCE_VERSION="v1"
 
# 安装 k8s.io/code-generator
$ go get -u k8s.io/code-generator/...
$ cd $GOPATH/src/k8s.io/code-generator
 
# 执行代码自动生成，其中 pkg/client 是生成目标目录，pkg/apis 是类型定义目录
$ ./generate-groups.sh all "$ROOT_PACKAGE/pkg/client" "$ROOT_PACKAGE/pkg/apis" "$CUSTOM_RESOURCE_NAME:$CUSTOM_RESOURCE_VERSION"
```

代码生成工作完成之后，我们再查看一下这个项目的目录结构：

```bash
$ tree
.
├── controller.go
├── crd
│   └── network.yaml
├── example
│   └── example-network.yaml
├── main.go
└── pkg
    ├── apis
    │   └── samplecrd
    │       ├── constants.go
    │       └── v1
    │           ├── doc.go
    │           ├── register.go
    │           ├── types.go
    │           └── zz_generated.deepcopy.go
    └── client
        ├── clientset
        ├── informers
        └── listers
```

其中，pkg/apis/samplecrd/v1 下面的 zz_generated.deepcopy.go 文件，就是自动生成的 DeepCopy 代码文件。

而整个 client 目录，以及下面的三个包（clientset、informers、 listers），都是 Kubernetes 为 Network 类型生成的客户端库，这些库会在后面编写自定义控制器的时候用到。

可以看到，到目前为止的这些工作，其实并不要求你写多少代码，主要考验的是“复制、粘贴、替换”这样的“基本功”。

而有了这些内容，现在你就可以在 Kubernetes 集群里创建一个 Network 类型的 API 对象了。

我们不妨一起来实验一下

**首先**，使用 network.yaml 文件，在 Kubernetes 中创建 Network 对象的 CRD（Custom Resource Definition）：

```shell
$ kubectl apply -f crd/network.yaml
customresourcedefinition.apiextensions.k8s.io/networks.samplecrd.k8s.io created
```

这个操作，就告诉了 Kubernetes，我现在要添加一个自定义的 API 对象。而这个对象的 API 信息，正是 network.yaml 里定义的内容。我们可以通过 kubectl get 命令，查看这个 CRD：

```shell
$ kubectl get crd
NAME                        CREATED AT
networks.samplecrd.k8s.io   2018-09-15T10:57:12Z
```

**然后**，我们就可以创建一个 Network 对象了，这里用到的是 example-network.yaml：

```shell
$ kubectl apply -f example/example-network.yaml 
network.samplecrd.k8s.io/example-network created
```

通过这个操作，你就在 Kubernetes 集群里创建了一个 Network 对象。它的 API 资源路径是`samplecrd.k8s.io/v1/networks`。

这时候，你就可以通过 kubectl get 命令，查看到新创建的 Network 对象：

```bash
$ kubectl get network
NAME              AGE
example-network   8s
```

你还可以通过 kubectl describe 命令，看到这个 Network 对象的细节：

```bash
$ kubectl describe network example-network
Name:         example-network
Namespace:    default
Labels:       <none>
...API Version:  samplecrd.k8s.io/v1
Kind:         Network
Metadata:
  ...
  Generation:          1
  Resource Version:    468239
  ...
Spec:
  Cidr:     192.168.0.0/16
  Gateway:  192.168.0.1
```

当然 ，你也可以编写更多的 YAML 文件来创建更多的 Network 对象，这和创建 Pod、Deployment 的操作，没有任何区别



## 例子2: 自定义CronTab资源类型

### 创建CRD

```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  # CRD名称必须是: <plural>.<group>格式
  name: crontabs.samplecrd.k8s.io
spec:
  # 决定了REST API路径：/apis/<group>/<version>
  group: samplecrd.k8s.io
  # 此CRD支持的版本
  versions:
    - name: v1
      # 每个版本都可以被禁用或启用
      served: true
      # 只有一个版本可以被标记为true，表示以此版本来存储
      storage: true
  # 可选Namespaced或Cluster，表示此资源是命名空间限定的，还是全局的
  scope: Namespaced
  names:
    # 决定了REST API路径： /apis/<group>/<version>/<plural>
    plural: crontabs
    # 在CLI中的别名
    singular: crontab
    # 驼峰式大小写的，正式的资源类型
    kind: CronTab
    # 在CLI中可以使用的短名称
    shortNames:
    - ct
```

执行这个yaml

```shell
kubectl create -f crd.yaml
```

创建之后 , 可以通过API端点/apis/k8s.gmem.cc/v1/namespaces/*/crontabs/来管理自定义资源。

### 子资源

自定义资源可以支持/status和/scale子资源。此特性在1.11版本中处于Beta状态且默认启用。你需要在CRD中进行定义才能启用这些子资源。

scale子资源支持让其他K8S组件（例如HorizontalPodAutoscaler和PodDisruptionBudget控制器）与你的CR进行交互。kubectl scale也可以利用该子资源对CR进行扩容。

status子资源可以让你把资源的规格和状态分开。

#### /status

启用后, 可以在自定义的资源下的/status管理
- 数据对应资源的.status字段
- PUT /status仅仅会修改.status字段，也仅仅对该字段进行验证
- 对资源本身进行PUT/POST/PATCH操作，会忽视.status字段
- 每次修改.spec字段，都导致.metadata.generation ++ 

#### /scale

启用扩容子资源后 , CRD需要指定

- SpecReplicasPath，指定自定义资源中对应Scale.Spec.Replicas的JSON路径。必须值
- StatusReplicasPath，指定自定义资源中对应Scale.Status.Replicas的JSON路径。必须值
- LabelSelectorPath，指定自定义资源中对应Scale.Status.Selector的JSON路径。可选值，和HPA联用则必须设置

比如:

```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
spec:
  subresources:
    # 启用状态子资源
    status: {}
    # 启用扩容子资源
    scale:
      specReplicasPath: .spec.replicas
      statusReplicasPath: .status.replicas
      labelSelectorPath: .status.labelSelector
```



### Categories

用于指定资源所属的类别，例如all

比如:

```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
spec:
  names:
    categories:
    - all
```

通过kubectl get all可以访问到上述CRD的自定义资源