# 介绍

 Couchbase是一个集群化的NoSQL内存数据库系统，具有如下特性：

- Key/Value
- 文档（json)
- Bucket (Couchbase、Memcached)
- View（可从数据中提取特定的字段信息）
- N1QL（二级索引，可搜索和查询文档内容）

# 最佳实践

#### Couchbase bucket内存占用评估

假如原始数据key、value大小累加为x，由于couchbase类型的bucket，每个item都有一份replica，那么该申请的bucket大小至少为 x*2/0.75 = 2.67x

当bucket使用内存达到分配内存的75%时，由于内存不足，couchbase会通过lru算法将部分数据从内存中踢出，只存储在磁盘上，下一次读取这部分数据时，再从磁盘取出并加载到内存。从磁盘取数据会使couchbase的读取性能降低

当bucket使用内存达到或接近分配内存的85%时，bucket可能会出现写不进数据的情况，同时集群读取性能受到较大影响

#### Java序列化方式优化建议

Java序列化速度较慢，序列化后对象占用字节数较大，对于性能要求较高的场景需要慎重使用。用户在选择序列化方式时可从减少频繁的序列化内存操作、降低每次序列化内存的使用大小、减轻jvm压力等方面来考虑，推荐使用thrift、pb以及其他常用的rpc架构的序列化方式

a)Java序列化速度较慢，序列化后对象占用字节数较大，对于性能要求较高的场景需要慎重使用。用户在选择序列化方式时可从减少频繁的序列化内存操作、降低每次序列化内存的使用大小、减轻jvm压力等方面来考虑，推荐使用thrift、pb以及其他常用的rpc架构的序列化方式

如果已经在使用Java序列化且不方便调整序列化方式，考虑到Java反序列化会频繁调用String.intern()和Class.forName()方法，可以尝试调优jvm参数-XX:PredictedLoadedClassCount=80009（具体设置多大根据场景） -XX:StringTableSize=240011（具体设置多大根据场景），这些方法可能会是cpu瓶颈

参考：https://docs.oracle.com/javase/7/docs/api/java/lang/String.html#intern

https://docs.oracle.com/javase/8/docs/api/java/lang/String.html#intern--

#### Couchbase（同Membase）和Memcached两种类型的bucket有什么区别

1. Couchbase类型的bucket内存大小可以动态调整，数据支持持久化，可以设置replica，可以建立XDCR同步，支持数据rebalance。允许集群中有一个节点failover出集群，数据不丢失，不影响客户端访问。
2. Memcached类型的bucket内存是预先分配好的一块，大小不支持动态调整，数据不进行持久化，不能设置replica，不能建立XDCR同步。集群failover一个节点即损失该节点上的数据。
3. 推荐使用Couchbase类型的bucket，可以尽量避免数据丢失，但如果存储数据不是很重要，且数据更新频率非常高，可以尝试使用Memcached类型的bucket，由于数据不进行持久化，Memcached带给集群的压力较小。

#### 遇到超时应该怎么排查

1. 造成超时的原因可能有很多，集群、客户端甚至网络的异常都会造成超时。
2. 如果连接同一集群的多个客户端中的一个客户端超时，则较大可能为客户端异常。另外，客户端可以适当增加读取和写入的timeout。 
3. 合理输出客户端日志，从日志排查问题。

#### 批量操作

https://developer.couchbase.com/documentation/server/3.x/developer/java-2.0/documents-bulk.html