# kubernetes高级调度方式

节点亲和性

```
kubectl explain pod.spec.affiniy
```





taint的effect定义对pod排斥效果:

NoSchedule: 仅影响调度过程 , 对现存pod对象不产生影响

NoExecute: 既影响调度过程 也影响现在的pod对象

PreferNoSchedule:  影响调度过程  不过你非要调度也可以 