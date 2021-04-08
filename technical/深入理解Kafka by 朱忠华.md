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

## 1.4 服务端参数配置
* 配置文件
  * $KAFKA_HOME/config/server.properties

* zookeeper.connect
  * broker要连接的ZooKeeper集群服务器地址，没有默认值，为必填项。
  * 多个节点的地址用逗号隔开
  * 最佳实践是再加一个chroot路径，这样既可以明确指明该路径是Kafka所用的，也可以实现多个Kafka集群复用一套ZooKeeper集群

* listeners
  * 指明broker监听客户端连接的地址列表，即为客户端要连接broker的入口地址列表
  * 格式为protocol1://hostname1:port1, protocol2://hostname2:port2，protocol代表协议类型，其支持的类型有PLAINTEXT，SSL，SASL_SSL等。

* broker.id
  * 用来指定Kafka集群中broker的唯一标识，默认值为-1。如果没有设置，那么Kafka会自动生成一个。

* log.dir和log.dirs
  * Kafka把所有的消息都保存在磁盘上，而这两个参数用来配置Kafka日志文件存放的根目录。

* message.max.bytes
  * 该参数用来指定broker所能接收消息的最大值，默认值为1000012，976.6KB
# 第2章 生产者
## 2.1 客户端开发
* 正常生产逻辑步骤
  * 配置生产者客户端参数及创建相应的生产者实例
  * 构建待发送的消息
  * 发送消息
  * 关闭生产者实例

### 2.1.1 必要的参数设置
* bootstrap.servers
* key.serializer
* value.serializer

### 2.1.2 消息的发送
* ProducerRecord
  * topic, value, partition, timestamp, key, headers
  * 除了topic和value的其他属性可以为null
* 发送消息三种模式
  * 发后即忘(fire-and-forgot)
  * 同步(sync)
  * 异步(async)
* KafkaProducer的send()方法会返回Future\<RecordMetaData>
* RecordMetaData包含消息的一些元数据信息
  * 当前消息的主题
  * 分区号
  * 分区中的偏移量(offset)
  * 时间戳

* 同步发送
  * 使用future.get()
* KafkaProducer异常
  * 可重试异常，若配置了retries参数，在参数次数内成功了，就不会抛出异常
    * NetworkException
    * LeaderNotAvailableException
    * UnknownTopicOrPartitionException
    * NotEnoughReplicasException
    * NotCoordinatorException
  * 不可重试异常
    * RecordTooLargeException
  * 同步发送的方式可靠性高，要么消息发送成功，要么发生异常。但是性能会差很多。
* 异步发送
  * 在send()方法指定一个Callback函数
  * 在同一分区中，回调函数的调用可以保证分区有序
* 回收资源
  * producer.close()
  * 等待所有消息发送完毕关闭producer

### 2.1.3 序列化
* 生产者序列化，消费者反序列化，它们所使用的序列化器需要一一对应。

### 2.1.4 分区器
* 消息在通过send()方法发往broker的过程中，可能需要经过拦截器(Interceptor)、序列化器(Serializer)和分区器(Partitioner)的一系列作用之后才能被发送。
* 如果ProducerRecord中指定了partition字段，就不需要分区器的作用。反之需要用分区器来根据key来计算partition的值

### 2.1.5 生产者拦截器
* 可以用来在消息发送前做一些准备工作，比如按照规则过滤不符合要求的消息、修改消息的内容等，也可以用来在发送回调逻辑前做一些定制化需求，比如统计类工作。

## 2.2 原理分析
### 2.2.1 整体架构
* 整个生产者客户端由两个线程协调运行，分别为主线程和Sender线程
  * 主线程中由KafkaProducer创建消息，经过拦截器、序列化器和分区器的作用之后缓存到消息累加器(RecordAccumulator)中。
  * Sender线程负责从RecordAccumulator获取消息并将其发送到Kafka中
* RecordAccumulator主要用来缓存消息以便Sender线程可以批量发送，进而减少网络传输的资源消耗以提升性能。
* RecordAccumulator大小可由生产者客户端参数buffer.memory配置。默认32KB
* 主线程发过来的消息都会被追加到RecordAccumulator的某个Deque中，Deque中存放的是ProducerBatch, 包含一个以上的ProducerRecord
* Sender从RecordAccumulator中获取缓存的消息后，会进一步将原本\<分区，Deque\<ProducerBatch>>的保存形式转变为\<Node, List\<ProducerBatch>>形式。Node为Kafka集群节点。
* Sender会进一步封装成\<Node, Request>的形式，这样就可以将Request请求发往各个Node了。
* Sender发送之前还会保存到InFlightRequests中，缓存了已经发出去但还没有收到相应的请求。InFlightRequests提供了管理类的方法，例如每个连接最大未响应请求数量。

### 2.2.2 元数据更新
* InFlightRequests还可以获得leastLoadedNode，即所有Node中负载最小的那个。leastLoadedNode可以用在多个场景，例如元数据请求和消费者组播协议的交互。


## 2.3 重要的生产者参数
* acks: 被认为发送成功的，分区中最小接受消息数量，默认为1，为0时不需要等待相应，性能最高。为-1或all时需要所有副本写入消息后，有最强的可靠性
* max.request.size: 消息大小的最大值, 默认1MB
* retries和retry.backoff.ms: 生产者重试次数。
* compression.type: 消息压缩类型，默认不压缩
* connection.max.idel.ms: 关闭闲置连接的时长，默认9分钟
* linger.ms: 生产者发送ProducerBatch之前等待更多消息加入ProducerBatch的时间。
* receive.buffer.bytes: Socket接受消息缓冲区的大小，默认32KB
* sender.buffer.bytes
* request.timeout.ms


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