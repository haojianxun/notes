# kubernetes怎么修改与设置触发资源回收策略的默认值

目前kubernetes的Qos的设置策略有三种:

- Guaranteed

  就是pod中的container同时设置了requests和limits , 并且requests和limits的值相同

- Burstable

  当pod不满足Guaranteed的条件, 但是至少有一个container设置了requests (requests和limits不相同)

- BestEffort

  既没有设置requests , 也没有设置limits 



设置了Qos策略的话 , 当主机资源不够的时候 , 就会触发Eviction(资源回收) ,  那主机 资源不够的标准是什么呢 , 在kubernetes会有默认的Eviction阈值

默认的Eviction阈值是:

```
memory.available<100Mi
nodefs.available<10%
nodefs.inodesFree<5%
imagefs.available<15%
```



在kubelet里是可以配置的

```
kubelet --eviction-hard=imagefs.available<10%,memory.available<500Mi,nodefs.available<5%,nodefs.inodesFree<5% --eviction-soft=imagefs.available<30%,nodefs.available<10% --eviction-soft-grace-period=imagefs.available=2m,nodefs.available=2m --eviction-max-pod-grace-period=600

```

其中Eviction会有2种模式

- Hard模式

  即选项`--eviction-hard`  , 在hard模式下 , 一旦达到阈值会立即触发资源回收

- soft模式

  即选项`--eviction-soft`  , soft模式下 , 一旦达到阈值 , 会有一段时间的缓冲 , 一般是2分钟  , 即某个值达到阈值了 之后的2分钟才会触发资源回收