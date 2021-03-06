# Kafka和Pulsar对比

在寻找替代方案时，我们了解了 Apache Pulsar 并决定对其进行评估。由于 Apache Kafka 和 Apache Pulsar 使用类似的消息概念，因此我们看到 **所有的 Kafka 用例可以使用 Pulsar 实现，其方式与使用与 Kafka 完全相同。兼容是促使我们切换到 Pulsar 的原因之一。**

我们还注意到了 Pulsar 在架构设计上与 Kafka 的一些重要差异。一个关键的区别是 **存储和计算的分离** - Pulsar Broker 是无状态的，与存储相互分离；而在 Kafka 的数据直接存储在 Broker 上。这是架构设计差异上的一个例子，它允许 Pulsar 能够实现一些在 Kafka 上做很困难或不可能的事情。这其中的例子包括：

- **主题可扩展性**：我们需要拥有超过 100,000 个主题（不考虑增长），这不仅有助于我们管理应用程序处理的不同类型的数据，还允许个别客户使用自定义应用程序连接到系统中的数据。Pulsar 的架构可以轻松处理数百万个主题。
- **性能**：由于 Pulsar 的分层架构，以及 IO 隔离的特性，读取和写入使用不同的物理存储。因此，读取的峰值根本不会影响写入性能，反之亦然。Pulsar 还支持非持久性主题，允许非常高的吞吐量，完全不需要持久性的主题，这对于实时应用程序非常有用。
- **消息队列**： Pulsar 提供了统一的消息模型，不仅支持类似 Kafka 的消费模式，也支持消息队里的消费模式。在不需要考虑有序性的应用场景中，Pulsar 可以直接当消息队列进行使用。Pulsar 在订阅（Subscription）级别而不是主题级别执行此操作，因为你可以在同一个主题中同时有按序消费的消费者和不按序消费的消费者，这对于很多场景是非常有价值。另一个场景，如果新的消费者需要从头开始读取一个主题里面的所有消息，那么对于 Kafka 来说，你将被迫要么牺牲吞吐量，要么重新平衡分区，或者要么牺牲有序性。而使用 Pulsar，您只需添加新订阅，Pulsar 就会将消息扇出到新增的消费者，以增加新消费者的吞吐量。
- **操作更简单**：使用 Apache Kafka，任何容量扩展都需要重新平衡分区，同时还需要将被平衡的分区重新拷贝到新添加的 Broker 上。使用 Pulsar，我们可以轻松添加和删除节点，而无需重新平衡整个集群。此外，使用 Pulsar，你永远不必担心一个分区是否会超过 Broker 的物理磁盘空间；但是在 Kafka 中，一个分区的容量不能超过一台 Broker 的物理磁盘空间。
- **无限的数据保留期**：我们的一些客户甚至需要在几个月后访问他们的文档。我们希望能够将数据保存在 Pulsar 中，而不会删除它，并在以后需要时使用它。这样我们不必重新从客户或者政府部门导入数据，我们也不必担心丢失消息。当我们需要使用新的一套系统来执行一个新的业务流程时，我们不需要访问数据库，我们可以简单地将文档从消息总线中拉取出并为新的业务流程重新处理它们即可。