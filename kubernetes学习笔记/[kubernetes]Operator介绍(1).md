## [kubernetes]Operator介绍(1)

Operator Framework是由coreos研发 , 他的github地址是https://github.com/operator-framework

官网: https://coreos.com/operators/

是一个开源工具箱，用于更高效的、自动化的、支持扩容的管理K8S Native应用程序

目前尚处于alpha阶段，但是由[controller-runtime](https://github.com/kubernetes-sigs/controller-runtime)提供的核心API已经比较稳定

Operator是打包、部署、管理K8S应用程序（这里指既部署在K8S上，也通过K8S API或kubectl管理的应用程序）的方法。从概念上说，Operator将运维人员的运维知识编码到软件中，使之更容易打包、和客户分享。Operator比运维人员的人工判断要敏捷的多，它可以观测集群/应用的当前状态并在若干毫秒之内作出合理的运维决定。

##组件

###**Operator Framework由以下三部分组成**：

- Operator SDK：对开发者屏蔽K8S API的复杂性，简化Operator/Controller的开发

- Operator Lifecycle Management：管理应用的安装、升级，管理K8S集群中运行的所有Operator（以及关联的Service）的生命周期

- Operator Metering：提供Operator运行状况的监控指标

###**SDK**

Operator SDK包含了构建、测试、打包Operator的工具。基于controller-runtime库，它提供了：

- 高层次的API和抽象，让你能编写更加直白的运维逻辑

- 用于快速开始一个新项目的代码生成器和脚手架

- 覆盖一些通用Operator用例的模式，不需要重复造轮子

### Lifecycle Manager

是运行在集群中的各种Operator的总控中心，它可以控制哪些Operator可以在哪些命名空间中运行，哪些用户可以和Operator进行交互。Lifecycle Manager也负责Operator及其关联资源的整体上的生命周期管理 —— 例如触发Operator及其关联资源的更新

对于简单的无状态应用，无需编写代码，直接使用通用Operator（例如Helm Operator）即可。对于复杂的有状态应用，自定义Operator的价值更为突显，“云风格的”特性可以嵌入到Operator代码中，实现自动化的扩容、更新、备份，以提升用户体验

### Metering

在未来，Operator Framework将提供度量应用程序指标的能力





