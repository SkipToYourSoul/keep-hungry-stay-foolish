# 介绍

[Apache Kafka](http://kafka.apache.org/) 是分布式发布-订阅消息系统。与传统消息系统相比，有以下优势：

- 它被设计为一个分布式系统，易于向外扩展；
- 它同时为发布和订阅提供高吞吐量；
- 它支持多订阅者，当失败时能自动平衡消费者；
- 它将消息持久化到磁盘，因此可用于批量消费，例如ETL，以及实时应用程序。

#### 什么场景适合选用Kafka

主要是从业务的角度出发，生产端与消费端进行合理的配置来适用不同的业务场景。Kafka不太适用到实时线上业务中。由于Kafka很难避免数据丢失以及不进行消息重复性检查，所以Kafka适合对消费重复数据有幂等性以及异常情况下数据丢失的场景，当然正常情况下不会有数据的丢失。如果不接受以上场景可以考虑ActiveMQ，RocketMQ。同时，kafka 一般用于日志收集的离线系统，对于实时性要求高的业务可以考虑ActiveMQ或者使用kafka服务时，使用流式计算进行及时消费。

# 最佳实践

### 生产端

#### **消息发送失败的重试应该使用Producer本身的重试机制**

当批量发送数据时，用户程序拿不到哪些数据发送失败，因此只能对整个批次的数据重发。为了减少重复的数据量应该使用Kafka自身的重试机制，Producer只会重新发送失败的Partition对应的数据。相关配置项message.send.max.retries，默认为3，retry.backoff.ms默认为100ms，为了保证数据的不丢失可以适当提高message.send.max.retries的值以及增大每次尝试重新发送消息的时间间隔。

#### **保证数据均匀的分布在Topic的Partition中，设置key不为null, 且不相同**

数据的不均衡对生产端和消费端的性能都会有较大影响，生产数据时应该保证数据的均匀分布。保证数据均匀分布可以选以下两种方式之一：1. 为每条消息提供一个唯一的主键，如一个自增Long型值；2. 为每条消息设置指定Partition的Key，采取自定义的partitioner 策略。

#### **批量发送数据**

Kafka运行时很少有大量读磁盘的操作，主要是定期批量写磁盘操作，因此操作磁盘很高效。建议使用批量发送，提高生产的吞吐。批处理是一种常用的用于提高I/O性能的方式。对Kafka而言，批处理既减少了网络传输的Overhead，又提高了写磁盘的效率。而对于异步发送而言，无论是使用哪个send方法，实现上都不会立即将消息发送给Broker，而是先存到内部的队列中，直到消息条数达到阈值或者达到指定的Timeout才真正的将消息发送出去，从而实现了消息的批量发送。由于每次网络传输，除了传输消息本身以外，还要传输非常多的网络协议本身的一些内容（称为Overhead），所以将多条消息合并到一起传输，可有效减少网络传输的Overhead，进而提高了传输效率。

#### **同步发送模式下，配置request.required.acks参数项**

- ***0\*** : 表示Producer不关心Broker给的Response，只要不在写入时出现网络错误，都认为写入成功不会触发重试机制，所以可能出现数据丢失。
- ***1\*** : 表示Producer只关心Paritition的leader是否写入成功，如果Broker给的Response中显示某partition的leader写入失败，则重试机制导致Producer会重新发送这个partition的所有数据，如果这个paritition的部分数据写入成功，就会造成数据重复。
- ***-1\*** : 表示一条数据写入大部分的ISR才算写入成功，基本上与min.insync.replica 组合使用，当min.insync.replica=2时，需要2个ISR同步消息成功才不会抛异常，否则，当ISR中的副本数量没有达到2个，则会抛出异常

#### **数据可靠性**

集群中出现Broker down掉可能会丢失数据，通过将request.required.acks设为-1，min.insync.replicas=n,在集群中的kafka broker 服务不可用数量为replica.factor(默认为3)-n 时，可以保证已写入的数据不丢失。min.insync.replicas的默认值为1，是一个topic级别的参数。在高容错模式下，推荐设置min.insync.replicas为 replica.factor/2+1。例如，目前线上每个partition有3个副本，当min.insync.replicas=2时，可以容忍集群中一个kafka broker服务不可用。

#### **控制单条消息体大小，勿超限**

目前线上默认的kafka可接收的最大消息是1000012 Bytes（包含key值与value值）, 其中，真正的消息体即净荷内容（value值）是小于该数值。如果单条消息超过最大设置的消息字节，会发送失败，会抛出“MessageSizeTooLarge” 这样的异常。该消息会被直接丢弃。如果需要更改最大消息的设置，需要同时在服务端以及消费端同时配置： 例如支持最大消息为3MB，需要做以下更改：

-  服务端配置更改： message.max.bytes=3145728 replica.fetch.max.bytes=4194304
-  消费端需要配置：fetch.message.max.bytes=4194304，默认是1024*1024

否则，会抛出”InvalidMessageSizeException“的异常。

### 消费端

#### **设置唯一group.id**

建议消费不同的topic使用不同的consumer group, 否则，如果同一个consumer group中的任意一个消费者下线时，会触发整个consumer group 内所有的consumer rebalance, 在整个rebalance过程中，消费者是不再进行消费消息，rebalance完成后才开始正常消费，会有一个空窗期。而在rebalance完成后，会有可能发生消息重复消费或者错失消费一些消息。主要是与在rebalance时，消费者提交offset相关。

#### 丢包和重发问题

##### 丢包问题

所谓丢包一般是指发送方发送的数据未到达接收方. 常见的丢包可能发生在发送端, 网络,接收端。

解决方案：

1. 对kafka进行限速，平滑流量
2. 启用重试机制，重试间隔时间设置长一些
3. Kafka设置acks=all，即需要相应的所有处于ISR的分区都确认收到该消息后，才算发送成功。

##### 重发问题

当消费者重新分配partition的时候，可能出现从头开始消费的情况，导致重发问题。当消费者消费的速度很慢的时候，可能在一个session周期内还未完成，导致心跳机制检测报告出问题。

> 底层根本原因：已经消费了数据，但是offset没提交。
> 配置问题：设置了offset自动提交
> 重复消费最常见的情况：re-balance问题,通常会遇到消费的数据，处理很耗时，导致超过了Kafka的session timeout时间（0.10.x版本默认是30秒），那么就会re-balance重平衡，此时有一定几率offset没提交，会导致重平衡后重复消费。

保证不丢失消息：

> 生产者（ack=all 代表至少成功发送一次)
> 消费者 （offset手动提交，业务逻辑成功处理后，提交offset）

去重问题：消息可以使用唯一id标识

> 保证不重复消费：落表（主键或者唯一索引的方式，避免重复数据）
> 业务逻辑处理（选择唯一主键存储到Redis或者mongdb中，先查询是否存在，若存在则不处理；若不存在，先插入Redis或Mongdb,再进行业务逻辑处理）

#### **消费线程数小于或者等于partition**

一个Consumer Group下消费一个topic的的所有消费线程数应该小于等于这个topic的partition数量。

#### **Consumer Group 的Consumer Lag**

反映消费滞后生产的情况。在kafka monitor中，可以看到某个topic 下的所有partition的消费情况，consume offset 代表消费partition的offset位置，如果发现滞后逐渐变多，而consume offset还是在移动，说明消费速度赶不上生产速度，这时侯需要提高consumer的并行度，即需要增加partition数量；

如果consume offset在一段时间内没有移动，可能出现以下几种情况：

(1) 没有可用的消息；可以查看一下是否logSize也没有发生变化，即没有消息生产到集群上，这时候消费 者处于阻塞的状态；

(2) 下一条可用消息大于你指定的最大fetch.message.max.bytes,该值默认为1MB，如果消息过大，大于1MB，会无法fetch到该消息，导致该消费停滞；

(3) 客户端代码停止从迭代器中拉消息; 可能是消费消息的应用程序代码以某种原因死掉了，因此导致消费者线程被杀死。建议使用try/catch字句来捕捉消费逻辑中所有抛出的异常。

(4) 消费者在平衡失败,会看到ConsumerRebalanceFailedException：这是由于两个消费者想拥有同一个topic分区而引起冲突导致的，日志会告诉你冲突的原因（查找”conflict in “），需要客户配置满足这个条件：rebalance.max.retries * rebalance.backoff.ms > zookeeper.session.timeout.ms