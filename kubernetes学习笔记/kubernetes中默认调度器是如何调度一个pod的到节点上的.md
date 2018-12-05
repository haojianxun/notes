# kubernetes中默认调度器的调度流程是怎么进行的

kubernetes默认的调度器要调度一个pod到一个节点上 , 它的步骤是这样的:

1. 预选过程(**Predicate**): 排除不符合pod要求的节点 , 选出符合pod调度的node列表
2. 优选过程(**Priority**): 在符合pod的node列表中 , 给每个node打分(1-10分) 
3. 选择(**Select**)   选出得分最高的那个node作为最终的调度结果





## 调度器的核心

![](https://ws1.sinaimg.cn/large/6450e885gy1fxozu3eh37j219b0r943n.jpg)

调度器的核心其实是2个独立的循环

### 循环一: Informer Path

有许多informer会启动起来, 监听etcd中的pod , node , server等相关api对象 , 比如一个刚刚创建出来的node , 会有一个对应的node informer来监听 , 将pod添加进调度队列

调度队列是一个PriorityQueue  , 当集群信息发生变化的时候 , 调度器会对他进行特殊的操作

### 循环二: Scheduling Path

从一个调度队列中拿出一个pod来 , 之后再调用Predicates从scheduler cache里拿出一个可用的node列表来 , 之后调用Priorities来对这个node列表中的node打分, 得分最高的node为调度的结果 

调度完成之后 , 就需要将pod里面的`nodeName`字段修改为调度出来node的字段 (这个步骤在kubernetes里叫**Bind**)

为了不在调度路径里远程访问APIServer , kubernetes的默认调度在Bind阶段 , 只会更新scheduler cache里的pod和node的信息 , 再异步的推送到etcd里 , 这种基于乐观 假设的API对象更新方式 在kubernetes里称为**Assume**

assume之后 , 会创建一个goroutine来异步的向apiserver发起更新请求 , 完成真正的bind操作









