#  corosync和pacemaker中重要概念

HA Cluster 

OpenAIS:Messaging Layer,Resource Allocation Layer,RA

- Messaging Layer

  heartbeat,cman,openais,corosync

- CRM

  haresource,crm,pacemaker,rgmanager

- RA

  LSB,OCF,systemd,STONITH,service

## **Resource Allocation Layer** : 资源管理

**DC**:designated coordinator

**component**:

​	**DC**:

​		CRM , CIB , PE , LRM

​	**非DC**:

​		CRM , CIB ,LRM

**CIB**:Cluster Infornation Base 集群信息库

​	DC接受新配置 并传播给其他的节点

### 资源类型

​	primitive:基本资源 主资源

​		仅能运行一份,仅能运行于单点

​	group:组

​		将resource组合成雨果service;

​		功能:组合,次序

​	clone:克隆

​		同一个资源在集群中科院出现多个副本,可以运行在多个节点

​	multi-state:(master/slave)

​		是克隆类型资源的特殊表现,存在多个副本,副本间存在主从关系;drbd

## quorum

quorum:

- with quorum:votes>total/2

- without quorum:votes <=total/2

  no_quorum_policy

  - stop
  - ignore
  - suicide
  - freeze

### fencing

node level:STONITH (shooting the other node in the head)

stonith device:

- hareware
- software
- meatware

resource level:

​	fc swith

### 特殊场景:two nodes cluster

1. no_quorum_policy

   ignore

2. quorum device

   - ping node
   - quorum disk

   ​

### 资源倾向性:资源的约束关系;

score  从正无穷到负无穷

- location 位置约束 资源对节点的倾向性
- colocation 排列约束 资源与资源在一起的倾向性
- order 顺序约束 定义资源间的依赖关系

## RA 资源代理

**LSB** /etc/rc.d/init.d/*

​	支持start | stop | status | reload | force-reload;

​	注意 一定不能开机启动

**OCF**:Open Cluster Framework 

​	/usr/lib/ocf/resource.d/provider/ 目录下 类似于LSB脚本 但是支持start stop status monitor meta-date等;

**systemd** unit file , /usr/lib/systemd/system/

​	注意 一定要enable

**STONITH** 调用stonith 设备的专用RA



## Messaging Layer

- hearbeat  (v1 , v2  ,v3)
- cman(cluster manager)
- corosync

### Resource  Allocation

 hearbeat v1 :haresources

​	配置接口:haresources配置文件

heartbeat v2 :crm

​	运行方式:在集群上每个节点运行雨果crmd守护进程(5560/tcp0,提供API

​	配置接口:crmsh hb_gui

hearbeat v3 :pacemaker

​	配置接口 :crmsh pcs

​	GUI :hawk(suse) LCMC pacemaker-gui

cman:rgmanager

​	配置接口:cluster.conf system-config-cluster ,conga(riccl/luci) ,cman_tool,ccs_tool ,clustat,....

corosync:pacemaker



​	

