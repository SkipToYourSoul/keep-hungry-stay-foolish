# 介绍

RocketMQ 是阿里巴巴开源的一款分布式消息中间件。具有高性能低延时抗堆积可扩展等特点。

作为一款分布式得消息中间件，RocketMQ具有以下特性：

- 高吞吐低延时
- 抗海量消息堆积
- 简单的横向扩容
- 顺序消息
- 定时消息
- 服务端的消息过滤
- 消费组消费
- 多种消息查询方式

# 最佳实践

## Producer

1. 生产消息时善用Tag。发送消息时使用一个Topic，给消息打上不同的Tag来划分，那么消费者订阅时可以选择自己感兴趣的Tag来消费。过滤的行为发生于Broker，不会造成流量浪费。

2. 敏感业务请设置Key。给消息加上一个Key，可以方便后续排查问题。若出现消息丢失的问题，通过Key可以去Broker上查找消息内容以及被哪个Consumer消费。Key的设置请尽量保持唯一，提高查询效率。

3. 重要业务对于发送成功或失败打印对应日志，记录Key，ID等可以方便查询。

4. send方法没有抛异常代表消息已经发送给了Broker。但是发送成功也有多种状态：

   1. SEND_OK

      消息发送成功

   2. FLUSH_DISK_TIMEOUT

      消息发送成功，等待刷盘超时，此时若服务器宕机才会丢消息

   3. FLUSH_SLAVE_TIMEOUT

      消息发送成功，同步到Slave超时，此时若服务器宕机才会丢消息

   4. SLAVE_NOT_AVAILABLE

      Slave不可用，此时若服务器宕机，消息不可被消费

5. 重要业务发送失败后，务必自行重试。重试多次失败则应该记录消息，稍后再触发重发。

6. send方法内部支持重试，重试逻辑如下：

   1. 重试次数：2次
   2. 每次失败会轮询下一个Broker发送（默认不开启）
   3. send的总耗时不超过sendMsgTimeout设置的超时时间，默认3秒。

## Consumer

1. 非幂等操作请自行去重。RocketMQ存在部分场景会有少量重复消费的情况。如果业务对于重复消费无法容忍，需要在消费时借助第三方缓存来记录消息是否被消费过。建议可以通过MessageID作为Unique key。
2. 根据Topic配置的ConsumeQueue的个数来控制消费并行度。RocketMQ的消费并行是通过分配ConsumeQueue到Consumer实现的，因此Consumer的个数不可以超过集群内该Topic的ConsumeQueue的总数。如果有很多Consumer，请提前告知运维人员为该Topic分配足够多的ConsumeQueue。
3. 消费失败时，请返回`ConsumeConcurrentlyStatus.RECONSUME_LATER`。这么一来，这一批消息会被送回Broker的延时队列，一定时间后被重新消费。
4. QAE环境时消费者需要设置一个唯一值的instanceName，例如： `consumer.setInstanceName(System.currentTimeMillis()+"-"+new Random().nextInt(1000));`

#### 消息丢失的case和对应的解决方案

https://blog.csdn.net/LO_YUN/article/details/103949317