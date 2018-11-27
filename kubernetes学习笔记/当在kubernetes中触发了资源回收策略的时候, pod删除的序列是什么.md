# 当在kubernetes中触发了资源回收策略的时候, pod删除的序列是什么

当Eviction阈值达到的时候 , 会进入**MemoryPressure**或者**DiskPressure**状态 , 从而避免新pod调度上来

发生的时候, 删除的pod的顺序是:

- 属于**BestEffort**类别的pod
- **Burstable**类型的 , 并且超过requests量的pod
- **Guaranteed**类型的 , 并且当pod超过limits设置的值 , 或者触发了MemoryPressure或者DiskPressure状态的pod