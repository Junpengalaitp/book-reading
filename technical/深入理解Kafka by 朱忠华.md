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
* connection.max.idle.ms: 关闭闲置连接的时长，默认9分钟
* linger.ms: 生产者发送ProducerBatch之前等待更多消息加入ProducerBatch的时间。
* receive.buffer.bytes: Socket接受消息缓冲区的大小，默认32KB
* sender.buffer.bytes
* request.timeout.ms


# 第3章 消费者
* 应用程序可以通过KafkaConsumer订阅主题，并从主题中拉取消息。

## 3.1 消费者和消费组
* 消费者负责订阅主题，每个消费者都有一个对应的消费组。当消息发布到主题后，只会被投递给它的每个消费组中的一个消费者。
* 消费者和消费组模型可以让整体的消费能力具备横向伸缩性。
* Kafka的P2P和Pub/Sub实现
  * P2P: 所有消费者在一个消费组里，那么每条消息只会被一个消费者消费
  * Pub/Sub: 每个消费者都在不同的消费组里，每条消息会被广播到所有消费组，也就是每个消费者。
* 消费组是一个逻辑上的概念，它将旗下的消费者归为一类，每个消费者只隶属于一个消费组。通过group.id来配置，默认为空字符串。

## 3.2 客户端开发
* 消费逻辑的步骤
  * 配置消费者客户端参数及创建相应的消费者实例
  * 订阅主题
  * 拉取消息并消费
  * 提交消费位移
  * 关闭消费者实例

### 3.2.1 必要的参数配置
* bootstrap.servers
* group.id
* key/value.serializer

### 3.2.2 订阅主题与分区
* 一个消费者可以订阅一个或多个主题，多次订阅以最后一次为准，不是并集。
* 订阅多个主题可以用集合或者正则
* 消费者可以订阅某些主题的特定分区，使用assign()方法。使用partitionFor()方法可以用来查询指定主题的元数据信息。
* 取消订阅使用unsubscribe()方法，或者在subscribe方法和assign方法传入空集合。
* 订阅状态
  * AUTO_TOPICS: 通过subscribe集合订阅
  * AUTO_PATTERN: 通过subscribe正则订阅
  * USER_ASSIGNED: 通过assign订阅
  * NONE: 无订阅
* 通过subscribe()订阅具有消费者自动再均衡功能，多个消费者的情况下可以根据分区分配策略来自动分配各个消费者与分区的关系。通过assign()方法订阅没有这个功能。

### 3.2.3 反序列化
### 3.2.4 消息消费
* Kafka中的消息消费是一个不断轮询的过程，消费者会不断地调用poll()方法，返回所订阅主题上的一组消息。
* poll()方法的返回值 类型是ConsumerRecords, 它用来表示一次拉取操作所获得的消息集，内部包含了若干ConsumerRecord, 它提供了一个iterator()方法来循环遍历消息集内部的消息。


### 3.2.5 位移提交 
* 消费者使用 offset 来表示消费到分区中某个消息所在的位置
* 在每次调用 poll()方法时，它返回的是还没有被消费过的消息集（当然这个前提是消息己经存储在 Kafka 了，并且暂不考虑异常情况的发生） 要做到这一点，就需要记录上次消费时的消费位移，并且这个消费位移必须做持久化保存，而不是单单保存在内存中，否则消费者重启之后就无法知晓之前的消费位移。
* 当有新的消费者加入时，那么必然会有再均衡的动作 对于同分区而言，它可能在再均衡动作之后分配给新的消费者 如果不持久化保存消费位移，那么这个新的消费者也无法知晓之前的消费位移。
* 消费位移存储在 Kafka 内部的主题 consumer offsets 这里把将消费位移存储起来（持久化）的动作称为“提交’ ，消费者在消费完消息之后需要执行消费位移的提交。
* 在Kafka中默认的消费位移提交方式是自动提交，由参数enable.auto.commit 配置，默认值为 true 。当然这个默认的自动提交不是每消费1条消息就提交1次，而是定期提交，这个定期的周期时间由客户端参数 auto.commit.interval.ms配置，默认值为5秒，此参数生效的前提是 enable auto.commit 参数为true 。在代码清单3-1 中并没有展示出这两个参数，说明使用的正是默认值。
* Kafka消费的编程逻辑中位移提交是一大难点，自动提交消费位移虽然简单，但是会带来重复消费或消息丢失问题。
* 手动提交，分为同步和异步。

### 3.2.6 控制或关闭消费
* KafkaConsumer 提供了对消费速度进行控制的方法，在有些应用场景下我们可能需要暂停某些分区的消费而先消费其他分区，当达到一定条件时再恢复这些分区的消费。
  * pause()方法
  * resume()方法
  * paused()方法：返回被暂停的分区集合

### 3.2.7 指定位移消费
* Kafka消费者找不到offset记录时，就会根据消费者客户端参数auto.offset.reset的配置来决定从何处开始进行消费，默认值为latest
* 使用seek()方法从特定的offset处开始拉取消息。

### 3.2.8 再均衡
* 分区的所属权从一个消费者转移到另一个消费者的行为，提供消费组高可用和高扩展性的保障。
* ConsumerRebalanceListener是个接口， 包含2个方法
  * void onPartitionsRevoked(Collection\<TopicPartition> partitions)
    * 再均衡开始之前和消费组停止读取消息之后被调用，可以通过这个回调方法来处理消费位移的提交，来避免一些不必要的重复消费现象发生
  * void onPartitionsAssigned(Collection\<TopicPartition> partitions)
    * 这个方法会在重新分配分区之后和消费者开始读取消费之前被调用。

### 3.2.9 消费者拦截器
* KafkaConsumer会在poll()方法返回之前调用拦截器的onConsume()方法来对消息进行相应的定制化操作，比如修改返回的消息内容、按照某种规则过滤消息（可能会减少poll()方法返回的消息的个数）。 如果onConsume()方法中抛出异常， 那么会被捕获并记录到日志中， 但是异常不会再向上传递。
### 3.2.10 多线程实现
* KafkaProducer是线程安全的， 然而KafkaConsumer却是非线程安全的。 KafkaConsumer中定义了一个acquire()方法， 用来检测当前是否只有一个线程在操作， 若有其他线程正在操作则会抛出ConcurrentModificationException异常。
* 线程封闭，为每个线程实例化一个KafkaConsumer对象。
* 多个消费线程同时消费同一个分区， 这个通过assign()、seek()等方法实现， 这样可以打破原有的消费线程的个数不能超过分区数的限制， 进一步提高了消费的能力。 不过这种实现方式对于位移提交和顺序控制的处理就会变得非常复杂， 实际应用中使用得极少，不推荐。
* 处理消息模块改成多线程的实现方式，相比第一种实现方式而言， 除了横向扩展的能力， 还可以减少TCP连接对系统资源的消耗， 不过缺点就是对于消息的顺序处理就比较困难。

### 3.2.11 重要的消费者参数


# 第4章 主题和分区

# 第5章 日志存储

# 第6章 深入服务端

# 第7章 深入客户端

# 第8章 可靠性探究

# 第9章 Kafka应用

# 第10章 Kafka监控

# 第11章 高级应用

# 第12章 Kafka与Spark集成