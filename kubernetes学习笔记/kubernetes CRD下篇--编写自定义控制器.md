# kubernetes CRD下篇--编写自定义控制器

## 自定义控制器的工作流程

![kubernetes的自定义控制器流程.png](https://i.loli.net/2018/11/21/5bf4db97d1448.png)

通过Informer , 从 Kubernetes 的 APIServer 里获取它所关心的对象 , Informer 与 API 对象是一一对应的 , 我们下面的例子的informer就是 Network 对象的 Informer（Network Informer）



而informer通过这个对象资源对象的client的对象来传递参数, 并和APIServer维持连接 , 使用的是一个叫Reflector 包 , client通过**ListAndWatch** 的方法来"获取"和"监听"要获取对象的变化

在 ListAndWatch 机制下，一旦 APIServer 端有新的 Network 实例被创建、删除或者更新，Reflector 都会收到“事件通知”。这时，该事件及它对应的 API 对象这个组合，就被称为增量（Delta），它会被放进一个 Delta FIFO Queue（即：增量先进先出队列）中。

而另一方面，Informe 会不断地从这个 Delta FIFO Queue 里读取（Pop）增量。每拿到一个增量，Informer 就会判断这个增量里的事件类型，然后创建或者更新本地对象的缓存。这个缓存，在 Kubernetes 里一般被叫作 Store。

比如，如果事件类型是 Added（添加对象），那么 Informer 就会通过一个叫作 Indexer 的库把这个增量里的 API 对象保存在本地缓存中，并为它创建索引。相反地，如果增量的事件类型是 Deleted（删除对象），那么 Informer 就会从本地缓存中删除这个对象。

这个**同步本地缓存的工作，是 Informer 的第一个职责，也是它最重要的职责。**

而**Informer 的第二个职责，则是根据这些事件的类型，触发事先注册好的 ResourceEventHandler**。这些 Handler，需要在创建控制器的时候注册给它对应的 Informer



## 例子1

接下来，我们就来编写这个控制器的定义，它的主要内容如下所示：

```go
func NewController(
  kubeclientset kubernetes.Interface,
  networkclientset clientset.Interface,
  networkInformer informers.NetworkInformer) *Controller {
  ...
  controller := &Controller{
    kubeclientset:    kubeclientset,
    networkclientset: networkclientset,
    networksLister:   networkInformer.Lister(),
    networksSynced:   networkInformer.Informer().HasSynced,
    workqueue:        workqueue.NewNamedRateLimitingQueue(...,  "Networks"),
    ...
  }
    networkInformer.Informer().AddEventHandler(cache.ResourceEventHandlerFuncs{
    AddFunc: controller.enqueueNetwork,
    UpdateFunc: func(old, new interface{}) {
      oldNetwork := old.(*samplecrdv1.Network)
      newNetwork := new.(*samplecrdv1.Network)
      if oldNetwork.ResourceVersion == newNetwork.ResourceVersion {
        return
      }
      controller.enqueueNetwork(new)
    },
    DeleteFunc: controller.enqueueNetworkForDelete,
 return controller
}
```

**我前面在 main 函数里创建了两个 client（kubeclientset 和 networkclientset），然后在这段代码里，使用这两个 client 和前面创建的 Informer，初始化了自定义控制器。**

值得注意的是，在这个自定义控制器里，我还设置了一个工作队列（work queue），它正是处于示意图中间位置的 WorkQueue。这个工作队列的作用是，负责同步 Informer 和控制循环之间的数据。

> 实际上，Kubernetes 项目为我们提供了很多个工作队列的实现，你可以根据需要选择合适的库直接使用。

**然后，我为 networkInformer 注册了三个 Handler（AddFunc、UpdateFunc 和 DeleteFunc），分别对应 API 对象的“添加”“更新”和“删除”事件。而具体的处理操作，都是将该事件对应的 API 对象加入到工作队列中。**

需要注意的是，实际入队的并不是 API 对象本身，而是它们的 Key，即：该 API 对象的/。

而我们后面即将编写的控制循环，则会不断地从这个工作队列里拿到这些 Key，然后开始执行真正的控制逻辑。

综合上面的讲述，你现在应该就能明白，**所谓 Informer，其实就是一个带有本地缓存和索引机制的、可以注册 EventHandler 的 client**。它是自定义控制器跟 APIServer 进行数据同步的重要组件。

更具体地说，Informer 通过一种叫作 ListAndWatch 的方法，把 APIServer 中的 API 对象缓存在了本地，并负责更新和维护这个缓存。

其中，ListAndWatch 方法的含义是：首先，通过 APIServer 的 LIST API“获取”所有最新版本的 API 对象；然后，再通过 WATCH API 来“监听”所有这些 API 对象的变化。

而通过监听到的事件变化，Informer 就可以实时地更新本地缓存，并且调用这些事件对应的 EventHandler 了。

此外，在这个过程中，每经过 resyncPeriod 指定的时间，Informer 维护的本地缓存，都会使用最近一次 LIST 返回的结果强制更新一次，从而保证缓存的有效性。在 Kubernetes 中，这个缓存强制更新的操作就叫作：resync。

需要注意的是，这个定时 resync 操作，也会触发 Informer 注册的“更新”事件。但此时，这个“更新”事件对应的 Network 对象实际上并没有发生变化，即：新、旧两个 Network 对象的 ResourceVersion 是一样的。在这种情况下，Informer 就不需要对这个更新事件再做进一步的处理了。

这也是为什么我在上面的 UpdateFunc 方法里，先判断了一下新、旧两个 Network 对象的版本（ResourceVersion）是否发生了变化，然后才开始进行的入队操作。

以上，就是 Kubernetes 中的 Informer 库的工作原理了。

接下来，我们就来到了示意图中最后面的控制循环（Control Loop）部分，也正是我在 main 函数最后调用 controller.Run() 启动的“控制循环”。它的主要内容如下所示：

```go
func (c *Controller) Run(threadiness int, stopCh <-chan struct{}) error {
 ...
  if ok := cache.WaitForCacheSync(stopCh, c.networksSynced); !ok {
    return fmt.Errorf("failed to wait for caches to sync")
  }
  
  ...
  for i := 0; i < threadiness; i++ {
    go wait.Until(c.runWorker, time.Second, stopCh)
  }
  
  ...
  return nil
}
```

可以看到，启动控制循环的逻辑非常简单：

- 首先，等待 Informer 完成一次本地缓存的数据同步操作；
- 然后，直接通过 goroutine 启动一个（或者并发启动多个）“无限循环”的任务。

而这个“无限循环”任务的每一个循环周期，执行的正是我们真正关心的业务逻辑。

所以接下来，我们就来编写这个自定义控制器的业务逻辑，它的主要内容如下所示：

```go
func (c *Controller) runWorker() {
  for c.processNextWorkItem() {
  }
}
 
func (c *Controller) processNextWorkItem() bool {
  obj, shutdown := c.workqueue.Get()
  
  ...
  
  err := func(obj interface{}) error {
    ...
    if err := c.syncHandler(key); err != nil {
     return fmt.Errorf("error syncing '%s': %s", key, err.Error())
    }
    
    c.workqueue.Forget(obj)
    ...
    return nil
  }(obj)
  
  ...
  
  return true
}
 
func (c *Controller) syncHandler(key string) error {
 
  namespace, name, err := cache.SplitMetaNamespaceKey(key)
  ...
  
  network, err := c.networksLister.Networks(namespace).Get(name)
  if err != nil {
    if errors.IsNotFound(err) {
      glog.Warningf("Network does not exist in local cache: %s/%s, will delete it from Neutron ...",
      namespace, name)
      
      glog.Warningf("Network: %s/%s does not exist in local cache, will delete it from Neutron ...",
    namespace, name)
    
     // FIX ME: call Neutron API to delete this network by name.
     //
     // neutron.Delete(namespace, name)
     
     return nil
  }
    ...
    
    return err
  }
  
  glog.Infof("[Neutron] Try to process network: %#v ...", network)
  
  // FIX ME: Do diff().
  //
  // actualNetwork, exists := neutron.Get(namespace, name)
  //
  // if !exists {
  //   neutron.Create(namespace, name)
  // } else if !reflect.DeepEqual(actualNetwork, network) {
  //   neutron.Update(namespace, name)
  // }
  
  return nil
}
```

可以看到，在这个执行周期里（processNextWorkItem），我们**首先**从工作队列里出队（workqueue.Get）了一个成员，也就是一个 Key（Network 对象的：namespace/name）。

**然后**，在 syncHandler 方法中，我使用这个 Key，尝试从 Informer 维护的缓存中拿到了它所对应的 Network 对象。

可以看到，在这里，我使用了 networksLister 来尝试获取这个 Key 对应的 Network 对象。这个操作，其实就是在访问本地缓存的索引。实际上，在 Kubernetes 的源码中，你会经常看到控制器从各种 Lister 里获取对象，比如：podLister、nodeLister 等等，它们使用的都是 Informer 和缓存机制。

而如果控制循环从缓存中拿不到这个对象（即：networkLister 返回了 IsNotFound 错误），那就意味着这个 Network 对象的 Key 是通过前面的“删除”事件添加进工作队列的。所以，尽管队列里有这个 Key，但是对应的 Network 对象已经被删除了。

这时候，我就需要调用 Neutron 的 API，把这个 Key 对应的 Neutron 网络从真实的集群里删除掉。

**而如果能够获取到对应的 Network 对象，我就可以执行控制器模式里的对比“期望状态”和“实际状态”的逻辑了。**

其中，自定义控制器“千辛万苦”拿到的这个 Network 对象，**正是 APIServer 里保存的“期望状态”**，即：用户通过 YAML 文件提交到 APIServer 里的信息。当然，在我们的例子里，它已经被 Informer 缓存在了本地。

**那么，“实际状态”又从哪里来呢？**

当然是来自于实际的集群了。

所以，我们的控制循环需要通过 Neutron API 来查询实际的网络情况。

比如，我可以先通过 Neutron 来查询这个 Network 对象对应的真实网络是否存在。

- 如果不存在，这就是一个典型的“期望状态”与“实际状态”不一致的情形。这时，我就需要使用这个 Network 对象里的信息（比如：CIDR 和 Gateway），调用 Neutron API 来创建真实的网络。
- 如果存在，那么，我就要读取这个真实网络的信息，判断它是否跟 Network 对象里的信息一致，从而决定我是否要通过 Neutron 来更新这个已经存在的真实网络。

这样，我就通过对比“期望状态”和“实际状态”的差异，完成了一次调协（Reconcile）的过程。

至此，一个完整的自定义 API 对象和它所对应的自定义控制器，就编写完毕了。

> 备注：与 Neutron 相关的业务代码并不是本篇文章的重点，所以我仅仅通过注释里的伪代码为你表述了这部分内容。如果你对这些代码感兴趣的话，可以自行完成。最简单的情况，你可以自己编写一个 Neutron Mock，然后输出对应的操作日志。

接下来，我们就一起来把这个项目运行起来，查看一下它的工作情况。

你可以自己编译这个项目，也可以直接使用我编译好的二进制文件（samplecrd-controller）。编译并启动这个项目的具体流程如下所示：

```shell
# Clone repo
$ git clone https://github.com/resouer/k8s-controller-custom-resource$ cd k8s-controller-custom-resource
 
### Skip this part if you don't want to build
# Install dependency
$ go get github.com/tools/godep
$ godep restore
# Build
$ go build -o samplecrd-controller .
 
$ ./samplecrd-controller -kubeconfig=$HOME/.kube/config -alsologtostderr=true
I0915 12:50:29.051349   27159 controller.go:84] Setting up event handlers
I0915 12:50:29.051615   27159 controller.go:113] Starting Network control loop
I0915 12:50:29.051630   27159 controller.go:116] Waiting for informer caches to sync
E0915 12:50:29.066745   27159 reflector.go:134] github.com/resouer/k8s-controller-custom-resource/pkg/client/informers/externalversions/factory.go:117: Failed to list *v1.Network: the server could not find the requested resource (get networks.samplecrd.k8s.io)
...
```

你可以看到，自定义控制器被启动后，一开始会报错。

这是因为，此时 Network 对象的 CRD 还没有被创建出来，所以 Informer 去 APIServer 里“获取”（List）Network 对象时，并不能找到 Network 这个 API 资源类型的定义，即：

```
Failed to list *v1.Network: the server could not find the requested resource (get networks.samplecrd.k8s.io)
```

所以，接下来我就需要创建 Network 对象的 CRD，这个操作在上一篇文章里已经介绍过了。

在另一个 shell 窗口里执行：

```shell
$ kubectl apply -f crd/network.yaml
```

这时候，你就会看到控制器的日志恢复了正常，控制循环启动成功：

```
...
I0915 12:50:29.051630   27159 controller.go:116] Waiting for informer caches to sync
...
I0915 12:52:54.346854   25245 controller.go:121] Starting workers
I0915 12:52:54.346914   25245 controller.go:127] Started workers
```

接下来，我就可以进行 Network 对象的增删改查操作了。

首先，创建一个 Network 对象：

```shell
$ cat example/example-network.yaml 
apiVersion: samplecrd.k8s.io/v1
kind: Network
metadata:
  name: example-network
spec:
  cidr: "192.168.0.0/16"
  gateway: "192.168.0.1"
  
$ kubectl apply -f example/example-network.yaml 
network.samplecrd.k8s.io/example-network created
```

这时候，查看一下控制器的输出：

```
...
I0915 12:50:29.051349   27159 controller.go:84] Setting up event handlers
I0915 12:50:29.051615   27159 controller.go:113] Starting Network control loop
I0915 12:50:29.051630   27159 controller.go:116] Waiting for informer caches to sync
...
I0915 12:52:54.346854   25245 controller.go:121] Starting workers
I0915 12:52:54.346914   25245 controller.go:127] Started workers
I0915 12:53:18.064409   25245 controller.go:229] [Neutron] Try to process network: &v1.Network{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta:v1.ObjectMeta{Name:"example-network", GenerateName:"", Namespace:"default", ... ResourceVersion:"479015", ... Spec:v1.NetworkSpec{Cidr:"192.168.0.0/16", Gateway:"192.168.0.1"}} ...
I0915 12:53:18.064650   25245 controller.go:183] Successfully synced 'default/example-network'
...
```

可以看到，我们上面创建 example-network 的操作，触发了 EventHandler 的“添加”事件，从而被放进了工作队列。

紧接着，控制循环就从队列里拿到了这个对象，并且打印出了正在“处理”这个 Network 对象的日志。

可以看到，这个 Network 的 ResourceVersion，也就是 API 对象的版本号，是 479015，而它的 Spec 字段的内容，跟我提交的 YAML 文件一摸一样，比如，它的 CIDR 网段是：192.168.0.0/16。

这时候，我来修改一下这个 YAML 文件的内容，如下所示：

```shell
$ cat example/example-network.yaml 
apiVersion: samplecrd.k8s.io/v1
kind: Network
metadata:
  name: example-network
spec:
  cidr: "192.168.1.0/16"
  gateway: "192.168.1.1"
```

可以看到，我把这个 YAML 文件里的 CIDR 和 Gateway 字段的修改成了 192.168.1.0/16 网段。

然后，我们执行了 kubectl apply 命令来提交这次更新，如下所示：

```shell
$ kubectl apply -f example/example-network.yaml 
network.samplecrd.k8s.io/example-network configured
```

这时候，我们就可以观察一下控制器的输出：

```
...
I0915 12:53:51.126029   25245 controller.go:229] [Neutron] Try to process network: &v1.Network{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta:v1.ObjectMeta{Name:"example-network", GenerateName:"", Namespace:"default", ...  ResourceVersion:"479062", ... Spec:v1.NetworkSpec{Cidr:"192.168.1.0/16", Gateway:"192.168.1.1"}} ...
I0915 12:53:51.126348   25245 controller.go:183] Successfully synced 'default/example-network'
```

可以看到，这一次，Informer 注册的“更新”事件被触发，更新后的 Network 对象的 Key 被添加到了工作队列之中。

所以，接下来控制循环从工作队列里拿到的 Network 对象，与前一个对象是不同的：它的 ResourceVersion 的值变成了 479062；而 Spec 里的字段，则变成了 192.168.1.0/16 网段。

最后，我再把这个对象删除掉：

```shell
$ kubectl delete -f example/example-network.yaml
```

这一次，在控制器的输出里，我们就可以看到，Informer 注册的“删除”事件被触发，并且控制循环“调用”Neutron API“删除”了真实环境里的网络。这个输出如下所示：

```
W0915 12:54:09.738464   25245 controller.go:212] Network: default/example-network does not exist in local cache, will delete it from Neutron ...
I0915 12:54:09.738832   25245 controller.go:215] [Neutron] Deleting network: default/example-network ...
I0915 12:54:09.738854   25245 controller.go:183] Successfully synced 'default/example-network'
```

以上，就是编写和使用自定义控制器的全部流程了。

实际上，这套流程不仅可以用在自定义 API 资源上，也完全可以用在 Kubernetes 原生的默认 API 对象上。

比如，我们在 main 函数里，除了创建一个 Network Informer 外，还可以初始化一个 Kubernetes 默认 API 对象的 Informer 工厂，比如 Deployment 对象的 Informer。这个具体做法如下所示：

```go
func main() {
  ...
  
  kubeInformerFactory := kubeinformers.NewSharedInformerFactory(kubeClient, time.Second*30)
  
  controller := NewController(kubeClient, exampleClient,
  kubeInformerFactory.Apps().V1().Deployments(),
  networkInformerFactory.Samplecrd().V1().Networks())
  
  go kubeInformerFactory.Start(stopCh)
  ...
}
```

在这段代码中，我们**首先**使用 Kubernetes 的 client（kubeClient）创建了一个工厂；

**然后**，我用跟 Network 类似的处理方法，生成了一个 Deployment Informer；

**接着**，我把 Deployment Informer 传递给了自定义控制器；当然，我也要调用 Start 方法来启动这个 Deployment Informer。

而有了这个 Deployment Informer 后，这个控制器也就持有了所有 Deployment 对象的信息。接下来，它既可以通过 deploymentInformer.Lister() 来获取 Etcd 里的所有 Deployment 对象，也可以为这个 Deployment Informer 注册具体的 Handler 来。

更重要的是，**这就使得在这个自定义控制器里面，我可以通过对自定义 API 对象和默认 API 对象进行协同，从而实现更加复杂的编排功能**。

比如：用户每创建一个新的 Deployment，这个自定义控制器，就可以为它创建一个对应的 Network 供它使用。



## 例子2

控制器主要使用以下client-go组件：

1. Informer/SharedInformer：监控目标K8S资源的变化，并交由ResourceEventHandler处理
2. ResourceEventHandler：通常是将事件发送到工作队列
3. Workqueue ：暂存资源变更事件，由控制循环取出事件并处理

### **Informer**

此组件负责获取对象状态，通常你不会直接向API Server发请求，而是通过client-go提供的编程接口。client-go提供了缓存功能，避免反复从API Server获取数据。

如果仅仅需要关注对象的创建、修改、删除事件，可以使用ListerWatcher接口。该接口可以对特定的资源进行监控（watch）操作：

```go
import "k8s.io/client-go/tools/cache"
// 返回ListWatch结构，它实现了ListerWatcher接口
lw := cache.NewListWatchFromClient(
      client,   // 客户端
      &v1.Pod{}, // 被监控资源类型
      api.NamespaceAll, // 被监控命名空间
      fieldSelector) // 选择器，减少匹配的资源数量
```

有了ListerWatcher你就可以创建Informer了：

```go
store, controller := cache.NewInformer (
    &lw,
    &v1.Pod{},           // 监控的对象类型
    resyncPeriod,        // 如果非0则自动定期relist对象
    cache.ResourceEventHandlerFuncs{} // ResourceEventHandler 事件发送给此对象处理
)
```



实际编程时并不常使用Informer，下文会提到的SharedInformer使用的更多。



### **ListWatcher** 

```go
// ListerWatcher是任何支持对一个资源进行init list，并进行watch的对象
type ListerWatcher interface {
    List(options metav1.ListOptions) (runtime.Object, error)
        // watch能保证对资源进行持续不断的监控
    Watch(options metav1.ListOptions) (watch.Interface, error)
}
```

上文调用的cache.NewListWatchFromClient，已经提供了ListWatcher的实现：

```go
func NewFilteredListWatchFromClient(c Getter, resource string, namespace string, optionsModifier func(options *metav1.ListOptions)) *ListWatch {
    // ListWatch已经实现List方法，并代理给其成员函数listFunc，Watch方法类似
 
        listFunc := func(options metav1.ListOptions) (runtime.Object, error) {
        optionsModifier(&options) // 修改选项的回调
        return c.Get(). // 获得*restclient.Request，此结构允许你以链式调用方式构建对API的请求
            Namespace(namespace). // 限定命名空间
            Resource(resource). // 限定资源类型
            VersionedParams(&options, metav1.ParameterCodec). // 解析并限定资源版本
            Do(). // 执行请求并获得Result
            Get() // 获取Result中的runtime.Object对象
    }
    watchFunc := func(options metav1.ListOptions) (watch.Interface, error) {
        options.Watch = true
        optionsModifier(&options)
        return c.Get().
            Namespace(namespace).
            Resource(resource).
            VersionedParams(&options, metav1.ParameterCodec).
            Watch() // 尝试对请求的API进行监控，返回watch.Interface
    }
    return &ListWatch{ListFunc: listFunc, WatchFunc: watchFunc}
}
```

### ResourceEventHandler

通常在此接口中提供事件处理逻辑

```go
type ResourceEventHandler interface {
        // 当资源第一次加入到Informer的缓存后调用
    OnAdd(obj interface{})
        // 当既有资源被修改时调用。oldObj是资源的上一个状态，newObj则是新状态
        // resync时此方法也被调用，即使对象没有任何变化
    OnUpdate(oldObj, newObj interface{})
        // 当既有资源被删除时调用，obj是对象的最后状态，如果最后状态未知则返回DeletedFinalStateUnknown
    OnDelete(obj interface{})
}
```

### ResyncPeriod

规定每隔多久，控制器遍历缓存中所有对象，并调用OnUpdate。

如果控制器可能错过对象更新事件，或者先前的事件处理回调可能执行失败，则此配置参数很重要。

### SharedInformer

Informer会创建一个私有的缓存，其中包含它自己用到的所有资源。但是，在K8S中有很多控制器在运行，它们关注多种类型的对象。如果基于Informer实现这些控制器，就会有很多重复的缓存数据，增加资源占用。

SharedInformer能够创建一个共享的缓存，在多个控制器之间共享数据。此外，不管下游有多少个消费者，SharedInformer都仅仅对上游服务器建立一个Watch。因此SharedInformer同时降低了客户端的内存占用和服务器的负载。包含很多控制器的 kube-controller-manager使用SharedInformer。

SharedInformer直接提供了接受新增、更新、删除特定资源的钩子。

类似于Informer，cache模块也为SharedInformer提供了工厂函数：

```go
func NewSharedInformer(lw ListerWatcher, objType runtime.Object, resyncPeriod time.Duration) SharedInformer {
    return NewSharedIndexInformer(lw, objType, resyncPeriod, Indexers{})
}
```

### Workqueue

由于SharedInformer是共享的，因此它不能跟踪每个控制器处理事件的进度。控制器必须提供自己的队列和重试（处理）机制。

当资源状态变化后，SharedInformer的ResourceEventHandler在Workqueue中添加一个Key。Key的格式是 资源命名空间/资源名称，资源命名空间是可以省略的。

client-go/util/workqueue.提供了多种工作队列的实现，包括：

1. 延迟队列，延后一段时间再将元素入队，由接口DelayingInterface提供
2. 限速队列，限定单位时间内能够入队的元素量，由接口RateLimitingInterface提供

下面的代码示意了如何创建限速队列：

```go
queue := workqueue.NewRateLimitingQueue(workqueue.DefaultControllerRateLimiter())
```

一个Key在工作队列中的生命周期如下：

1. queue.Add(key)入队
2. queue.Get()获取第一个Key进行处理，如果：
   1. 处理成功，queue.Forget(key)清除掉Key
   2. 处理失败，在到达最大重试次数之前，控制器调用queue.AddRateLimited(key)重新入队
3. queue.Forget(key)仅仅让队列不再跟踪事件的历史。控制器会最终调用queue.Done()彻底删除事件

控制器仅仅（如果自己实现，也应该遵守此准则）在缓存完整同步后，才调用Worker，处理Workqueue，原因是：

1. 直到缓存同步完毕，列出的资源才是精确的
2. 可以让针对单个资源的多次更新合并为一个，避免反复处理中间状态，浪费资源



一个简单控制器的例子

```go
package main
 
import (
    "flag"
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/util/workqueue"
    "k8s.io/sample-controller/pkg/signals"
    "k8s.io/client-go/tools/cache"
    "k8s.io/client-go/tools/clientcmd"
    "github.com/golang/glog"
    "k8s.io/apimachinery/pkg/watch"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/runtime"
    utilruntime "k8s.io/apimachinery/pkg/util/runtime"
    apiv1 "k8s.io/api/core/v1"
    "fmt"
    "k8s.io/apimachinery/pkg/util/wait"
    "time"
)
 
/* 控制器 */
type Controller struct {
    // 此控制器使用的客户端
    clientset kubernetes.Interface
    // 此控制器使用的工作队列
    queue workqueue.RateLimitingInterface
    // 此控制器使用的共享Informer，SharedIndexInformer可以维护缓存中对象的索引
    informer cache.SharedIndexInformer
}
 
/* 主函数 */
var (
    // 参数变量
    masterURL  string
    kubeconfig string
)
// 启动控制器
func (c *Controller) Run(stopCh <-chan struct{}) {
    // 捕获应用程序崩溃并打印日志
    defer utilruntime.HandleCrash()
    // 关闭队列，从而导致Worker结束
    defer c.queue.ShutDown()
 
    glog.Info("启动控制器……")
 
    // 运行Informer
    go c.informer.Run(stopCh)
 
    // 在启动Worker之前，等待缓存同步完成
    if !cache.WaitForCacheSync(stopCh, c.informer.HasSynced) {
        utilruntime.HandleError(fmt.Errorf("同步缓存超时"))
        return
    }
 
    glog.Info("缓存已经同步，准备启动Worker")
    // 循环执行Worker，直到TERM
    wait.Until(c.runWorker, time.Second, stopCh)
}
 
// 启动Worker
func (c *Controller) runWorker() {
    for c.processNextItem() {
    }
}
 
// Worker的逻辑框架
func (c *Controller) processNextItem() bool {
    // 最大重试次数
    maxRetries := 3
 
    // 获取下一个元素，第2个出参提示队列是否已经关闭
    key, quit := c.queue.Get()
    if quit {
        return false
    }
 
    // 总是移除Key
    defer c.queue.Done(key)
 
    // 处理Key
    err := c.processItem(key.(string))
 
    if err == nil {
        // 处理成功，提示队列不再跟踪事件历史
        c.queue.Forget(key)
    } else if c.queue.NumRequeues(key) < maxRetries {
        glog.Errorf("处理%s事件失败，准备重试： %v", key, err)
        c.queue.AddRateLimited(key)
    } else {
        glog.Errorf("处理%s事件失败，放弃： %v", key, err)
        c.queue.Forget(key)
        utilruntime.HandleError(err)
    }
    return true
}
 
// Worker核心逻辑
func (c *Controller) processItem(key string) error {
    glog.Infof("开始处理事件%s", key)
    // 根据Key获取对象
    obj, exists, err := c.informer.GetIndexer().GetByKey(key)
    if err != nil {
        return fmt.Errorf("获取对象%s失败: %v", key, err)
    }
    fmt.Print(obj)
    if !exists {
        // 在这里处理对象删除事件
    } else {
        // 在这里处理对象创建事件
    }
    // 因为不进行Resync，不会有更新事件
    return nil
}
 
func main() {
    // 解析参数，存入上述变量
    flag.Parse()
    cfg, err := clientcmd.BuildConfigFromFlags(masterURL, kubeconfig)
    if err != nil {
        glog.Fatalf("构建kubeconfig失败: %s", err.Error())
    }
    // 创建客户端，Clientset是一系列K8S API的集合
    clientset, err := kubernetes.NewForConfig(cfg)
    if err != nil {
        glog.Fatalf("构建clientset失败: %s", err.Error())
    }
    // 信号处理通道，当进程接收到信号后，此通道可读
    stopCh := signals.SetupSignalHandler()
 
    queue := workqueue.NewRateLimitingQueue(workqueue.DefaultControllerRateLimiter())
 
    informer := cache.NewSharedIndexInformer(
        &cache.ListWatch{
            ListFunc: func(options metav1.ListOptions) (runtime.Object, error) {
                // 仅仅列出所有命名空间的Pod
                return clientset.CoreV1().Pods(metav1.NamespaceAll).List(options)
            },
            WatchFunc: func(options metav1.ListOptions) (watch.Interface, error) {
                return clientset.CoreV1().Pods(metav1.NamespaceAll).Watch(options)
            },
        },
        &apiv1.Pod{},
        0,                // 不进行relist
        cache.Indexers{}, // map[string]IndexFunc
    )
 
    // 添加事件处理回调，仅仅是简单的入队
    informer.AddEventHandler(cache.ResourceEventHandlerFuncs{// 此结构实现ResourceEventHandler
        AddFunc: func(obj interface{}) {
            // 从对象中抽取Key
            key, err := cache.MetaNamespaceKeyFunc(obj)
            if err == nil {
                queue.Add(key)
            }
        },
        DeleteFunc: func(obj interface{}) {
            key, err := cache.DeletionHandlingMetaNamespaceKeyFunc(obj)
            if err == nil {
                queue.Add(key)
            }
        },
    })
 
    // 构建控制器对象
    ctrl := Controller{
        clientset,
        queue,
        informer,
    }
 
    // 启动
    ctrl.Run(stopCh)
}
```

获取对象内容

```go
newDepl = new.(*appsv1.Deployment)
```

判断对象是否有变化

利用ObjectMeta.ResourceVersion，资源有变化后此字段即改变

```go
deploymentInformer.Informer().AddEventHandler(cache.ResourceEventHandlerFuncs{
        // 资源添加到缓存中后调用（不是在集群中创建后）
    AddFunc: controller.handleObject,
        // 既有资源修改后调用。当执行resync后，该回调也被调用，即使资源没有任何变化
    UpdateFunc: func(old, new interface{}) {
        newDepl := new.(*appsv1.Deployment)
        oldDepl := old.(*appsv1.Deployment)
        if newDepl.ResourceVersion == oldDepl.ResourceVersion {
            // 如果资源没有变化则直接返回
            return
        }
        controller.handleObject(new)
    },
        // 当既有资源被删除后调用，入参是资源的最终状态
        // 如果最终状态未知，则入参是DeletedFinalStateUnknown。这种情况的原因可能是watch关闭而错过
        // 资源的删除事件，直到下次relist控制器才意识到资源被删除
    DeleteFunc: controller.handleObject,
})
```

###代码生成

[code-generator](https://github.com/kubernetes/code-generator)是K8S提供的一个代码生成器项目，可以用来：

1. 开发CRD的控制器时，生成版本化的、类型化的客户端代码（clientset），以及Lister、Informer代码
2. 开发API聚合时，在内部和版本化的类型、defaulters、protobuf编解码器、client、informer之间进行转换

K8S本身以及OpenShift也在使用此项目。

code-generator提供的，和CRD有关的生成器包括：

1. deepcopy-gen：为每个T类型生成 func (t* T) DeepCopy() *T方法。API类型都需要实现深拷贝
2. client-gen：为CustomResource API组生成强类型的clientset
3. informer-gen：为CustomResources生成Informer
4. lister-gen：为CustomResources生成Lister，Lister为GET/LIST请求提供只读缓存层

Informer和Lister是构建控制器（或者叫Operetor）的基本要素。使用这4个代码生成器可以创建全功能的、和K8S上游控制器工作机制相同的production-ready的控制器。

code-generator还包含一些其它的代码生成器。例如Conversion-gen负责产生内外部类型的转换函数、Defaulter-gen负责处理字段默认值。

[crd-code-generation](https://github.com/openshift-evangelists/crd-code-generation)是使用代码生成器的一个示例项目，可以作为你的实际项目的起点

####调用代码生成器

[code-generator](https://github.com/kubernetes/code-generator)基于[k8s.io/gengo](https://github.com/kubernetes/gengo)实现，两者共享一部分命令行标记。大部分的生成器支持--input-dirs参数来读取一系列输入包，处理其中的每个类型，然后生成代码：

1. 部分代码生成到输入包所在目录，例如deepcopy-gen生成器。可以使用参数 --output-file-base "zz_generated.deepcopy"来定义输出文件名
2. 其它代码生成到--output-package指定的目录（通常为pkg/client），例如client-gen、informer-gem、lister-gen等生成器

开发CRD时，你可以使用generator-group.sh脚本而不是逐个手工调用生成器。通常可以在项目中编写hack/update-codegen.sh：

```bash
#!/bin/bash
 
set -o errexit
set -o nounset
set -o pipefail
 
SCRIPT_ROOT=$(dirname ${BASH_SOURCE})/..
# 代码生成器包位置
CODEGEN_PKG=${CODEGEN_PKG:-$(cd ${SCRIPT_ROOT}; ls -d -1 ./vendor/k8s.io/code-generator 2>/dev/null || echo ${GOPATH}/src/k8s.io/code-generator)}
 
# generate-groups.sh <generators> <output-package> <apis-package> <groups-versions>
#                    使用哪些生成器，可选值deepcopy,defaulter,client,lister,informer，逗号分隔，all表示全部使用
#                                 输出包的导入路径
#                                                  CR定义所在路径
#                                                                 API组和版本
vendor/k8s.io/code-generator/generate-groups.sh all \
  github.com/openshift-evangelists/crd-code-generation/pkg/client \
  github.com/openshift-evangelists/crd-code-generation/pkg/apis \
  example.com:v1 \
  # 自动生成的源码，头部附加的内容
  --go-header-file ${SCRIPT_ROOT}/hack/custom-boilerplate.go.txt
```

执行上面的脚本后，所有API代码会生成在pkg/apis目录下，clientsets、informers、listers则生成在pkg/client目录下。 

你可以进一步提供hack/update-codegen.sh脚本，用于判断生成的代码是否up-to-date：

```shell
#!/bin/bash
 
set -o errexit
set -o nounset
set -o pipefail
 
# 先调用update-codegen.sh生成一份新代码，然后对比新老代码是否一样
 
SCRIPT_ROOT=$(dirname "${BASH_SOURCE}")/..
DIFFROOT="${SCRIPT_ROOT}/pkg"
TMP_DIFFROOT="${SCRIPT_ROOT}/_tmp/pkg"
_tmp="${SCRIPT_ROOT}/_tmp"
 
cleanup() {
    rm -rf "${_tmp}"
}
trap "cleanup" EXIT SIGINT
 
cleanup
 
mkdir -p "${TMP_DIFFROOT}"
cp -a "${DIFFROOT}"/* "${TMP_DIFFROOT}"
 
"${SCRIPT_ROOT}/hack/update-codegen.sh"
echo "diffing ${DIFFROOT} against freshly generated codegen"
ret=0
diff -Naupr "${DIFFROOT}" "${TMP_DIFFROOT}" || ret=$?
cp -a "${TMP_DIFFROOT}"/* "${DIFFROOT}"
if [[ $ret -eq 0 ]]
then
    echo "${DIFFROOT} up to date."
else
    echo "${DIFFROOT} is out of date. Please run hack/update-codegen.sh"
    exit 1
fi 
```



####控制代码生成

除了通过命令行标记控制代码生成器之外，你还可以在go源码中使用tag来设定一些供生成器使用的属性。这些tag分为两类：

1. 在doc.go的package语句之上提供的全局tag
2. 在需要被处理的类型上提供的局部tag

tag的语法如下：

```go
// +tag-name
// 或者
// +tag-name=value
```

也就是说，tag是以注释的形式存在的。tag的位置很重要，很多tag必须直接位于type或package语句的上一行，另外一些则必须和go语句隔开至少一行空白。

**全局tag**

必须在目标包的doc.go文件中声明，典型路径是 pkg/apis/<apigroup>/<version>/doc.go。 内容示例：

```go
// 为包中任何类型生成深拷贝方法，可以在局部tag覆盖此默认行为
// register关键字现在已经不需要，它的含义是将深拷贝方法注册到scheme中，从1.9开始scheme不再负责runtime.Object的深拷贝
// 你只需要直接调用obj.DeepCopy/DeepCopyObject()即可
// +k8s:deepcopy-gen=package,register
 
// groupName指定API组的全限定名
// 此API组的v1版本，放在同一个包中
// +groupName=example.com
package v1
```

**局部tag**

要么直接声明在类型之前，要么位于类型之前的第二个注释块中。下面的 types.go中声明了CR对应的go类型

```go
// 为当前类型生成客户端
// +genclient
// 提示此类型不基于/status子资源来实现spec-status分离，产生的客户端不具有UpdateStatus方法
// 否则，只要类型具有Status字段，就会生成UpdateStatus方法
// +genclient:noStatus
// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
 
// K8S资源，数据库
type Database struct {
    metav1.TypeMeta   `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty"`
 
    Spec DatabaseSpec `json:"spec"`
}
 
// 数据库的Spec
type DatabaseSpec struct {
    User     string `json:"user"`
    Password string `json:"password"`
    Encoding string `json:"encoding,omitempty"`
}
 
// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
 
// K8S资源，数据库列表
type DatabaseList struct {
    metav1.TypeMeta `json:",inline"`
    metav1.ListMeta `json:"metadata"`
 
    Items []Database `json:"items"`
}
```

内嵌了metav1.TypeMeta通常都是顶级类型，实现runtime.Object，一般需要为它们生成client。

对于集群级别（非命名空间内）的资源，你需要提供

```go
// +genclient:nonNamespaced
```

你还可以控制客户端提供哪些HTTP方法：

```go
// +genclient:noVerbs
// +genclient:onlyVerbs=create,delete
// +genclient:skipVerbs=get,list,create,update,patch,delete,deleteCollection,watch
// 仅仅返回Status而非整个资源
// +genclient:method=Create,verb=create,result=k8s.io/apimachinery/pkg/apis/meta/v1.Status
```





