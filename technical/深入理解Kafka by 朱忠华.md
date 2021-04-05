# 第1章 初识Kafka
* 由Linkedin采用Scala开发的一个多分区、多副本且基于ZooKeeper协调的分布式消息系统。
* Kafka目前已被定位为一个分布式流处理平台，开源分布式处理系统如Cloudera, Storm, Spark, Flink等都支持与Kafka集成。
* Kafka三大角色
  * 消息系统
  * 存储系统
  * 流式处理平台

## 1.1 基本概念
* 一个典型的Kafka体系架构包括若干Producer, Broker, Consumer, 以及一个ZooKeeper集群。
* Producer将消息发送到Broker, Broker负责将收到的消息存储到磁盘中，Consumer负责从Broker订阅并消费消息。
* Broker: 服务代理节点，可以简单地看作一个独立的Kafka服务节点或Kafka服务实例。
* Topic: 消息以主题为单位进行归类，生产者负责将消息发送到特定主题，消费者负责订阅主题并进行消费。
* Partition
  * 一个逻辑上的概念，还可以细分为多个分区，一个分区只属于单个主题，分区也叫做Topic-Partition。
  * 同一主题下的各分区包含不同的消息，通过offset保证消息在分区内的有序性
  * 分区可以在不同的服务器上(broker)，也就是说一个主题可以横跨多个broker
* Replica
  * 通过增加副本数量提升容灾能力。
  * 同一分区的不同副本中保存的是相同的消息，一主多从的形式，leader负责读写，follower只负责与leader同步。
* Kafka消费端也具备一定的容灾能力。Consumer用拉模式从服务端拉取消息，并且保存消费的具体位置。
* AR(Assigned Replicas)
  * 所有副本的统称
  * 所有与leader副本保持一定程度同步的副本组成ISR(In-Sync Replicas)，是AR的一个子集。
  * 与leader同步滞后的副本组成OSR(Out-of-Sync Replicas)。
  * AR = ISR + OSR, 理想情况为OSR = 0， AR = ISR

# 第2章 生产者


# 第3章 消费者

# 第4章 主题和分区

# 第5章 日志存储

# 第6章 深入服务端

# 第7章 深入客户端

# 第8章 可靠性探究

# 第9章 Kafka应用

# 第10章 Kafka监控

# 第11章 高级应用

# 第12章 Kafka与Spark集成