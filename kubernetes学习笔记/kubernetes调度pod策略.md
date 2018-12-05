# kubernetes调度pod策略 

pod调度分为3个步骤

1. 预选过程(Predicate): 排除不符合pod要求的节点
2. 优选过程(Priority): 根据 优先级找出最佳匹配的节点
3. 选择(Select)



## 预选阶段策略Predicate

源码具体的地址: https://github.com/kubernetes/kubernetes/blob/master/pkg/scheduler/algorithm/predicates/predicates.go

主要是分为三类

###GeneralPredicates类型

1. GeneralPredicates

   - HostName : 检查pod对象是否定义了`pod.spec.hostname` 
   - PodFitsHostPorts: `pod.spec.containers.ports.hostPort`
   - MatchNodeSelector: `pod.spec.nodeSelector` 
   - PodFitsResources: 检查pod资源能否被节点所满足  查看节点的资源可以用`kubectl describe nodes NODE_NAME` 

###与Volume相关的过滤规则

1. NoDiskConflict: 检查pod依赖的存储卷是否满足需求
2. PodToleratesNodeTaints: 检查pod上`sepc.tolerations` 可容忍的污点是否完全包含节点上的污点
3. PodToleratesNodeNoExecuteTaints: 此策略默认不启用 , 比如pod早调度完之后 , node的污点改了, 变成了pod不能接受的污点 , 这个时候要不要继续在这个node上 , 此策略是 如果出现了不能容忍的污点就调度出去 , 默认的情况是不调度, 即pod调度完成之后 node的污点改成了自己不能接受的 , 也不会再调度出去了
4. MaxEBSVolumeCount
5. MaxGCEPDVolumeCount
6. MaxAzureDiskVolumeCount
7. MaxCSIVolumeCountPred
8. NoVolumeZoneConflict

###与宿主机相关的过滤规则

1. CheckNodeMemoryPressure
2. CheckNodeDiskPressure
3. CheckNodeLabelPresence: 默认不启用
4. CheckNodePIDPressure
5. CheckNodeCondition
6. MatchInterPodAffinity
7. CheckServiceAffinity



## 优选阶段策略Priority

具体地址: https://github.com/kubernetes/kubernetes/tree/master/pkg/scheduler/algorithm/priorities

Priority阶段主要是为过滤后的node节点打分

下述1-7 是默认启用的

1. **least_requested**

   (cpu((capacity-sum(requested))*10/capacity) + memory((capacity-sum(requested))*10/capacity))/2

2. **balanced_resource_allocation**

   cpu和内存资源被占用率相近的胜出

3. **node_prefer_avoid_pods**

   根据节点注释信息`scheduler.alpha.kubernetes.io/preferAvoidPods` 评估优先级  , 如果没有这个注解信息 , 则得分为10 , 权重为10000 , 没有这个注解适合运行, 有这个注解的话 , 则不能运行在这个节点上

4. **taint_toleration**

   讲pod对象的`spec.tolerations` 列表项和node的taints列表项匹配 , 匹配越多 , 得分越低

5. **selector_spreading**

   node上的pod越多 得分越低  (是同pod 同一个标签的pod , 这个策略顾名思义)

6. **interpod_affinity**

   匹配到的匹配项越多 得分越高

7. **node_affinity**

   越亲和 得分越高

8. **most_requested**

   空闲资源越低的 得分越高  同**least_requested**相反   目的是尽可能把一个节点上的资源先用完

9. **node_label**

   有标签就得分 

10. **image_locality**

    node上有满足pod需求的镜像就得分  , 根据镜像体积大小之和来算  得分高胜出 , 而不是根据镜像的数量