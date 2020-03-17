# 介绍

HBase是Google Bigtable的开源版本，是建立在HDFS之上，提供高可靠性、高性能、列存储、可伸缩、实时读写的数据库。它属于列式（Column-Oriented）NoSQL数据库，目前仅支持单行事务，没有成熟的二级索引方案，主要用来存储非结构化和半结构化的松散数据。

HBase 中的表一般有这样的特点：

- 大：一个表可以有上亿行，上百万列；
- 面向列：面向列(族)的存储和权限控制，列(族)独立检索；
- 稀疏：对于为空(null)的列，并不占用存储空间，因此，表可以设计的非常稀疏；
- 数据多版本：每个单元中的数据可以有多个版本，默认情况下版本号自动分配，是单元插入时的时间戳；

##### HBase的适用场景

- 存储大量的数据且能保证良好的随机访问性能
- 需要很高的写吞吐量，瞬间写入量很大，传统数据库不能支撑或需要很高成本支撑的场景
- 可以进行优雅的数据扩展，动态扩展整个存储系统容量
- 数据格式无限制，支持办结构化和非结构化的数据
- 业务场景简单，不需要全部的关系型数据库特性，例如交叉列、交叉表，事务、连接等

# 最佳实践

## 避免RowKey热点

RowKey设计不合理容易导致热点问题，即所有的访问集中在一个或几个结点之上，导致这些机器过载，性能下降。一些常用的避免热点的方法：

#### 哈希

- 适用场景：1. 无需连续读取；2. RowKey较为复杂
- 具体方法：记原始Key为OriginalKey，则新的Rowkey = Substr(Md5(OriginalKey), 0, 3) + OriginalKey.
- 说明：MD5取4位做前缀用于保证负载均衡，OriginalKey也需要拼接上去，避免冲突

#### Reversing the key

- 适用场景：1. 无需连续读取；2. 固定长度或者数字类型的Rowkey
- 具体方法：将Rowkey倒序
- 说明：最后一位变化最频繁（数字的最低位）被移到开头，效果相当于哈希。和哈希相比，结果相似。优点是：1. 计算倒序代价比计算哈希低；2. 倒序无需额外的存储空间;

#### 取模

- 适用场景：同Reversing the key类似
- 说明：以聊天室为例，Original key为 roomID + msgID，若msgID全局唯一，则采用取模的方法，Rowkey = (roomID % N) + msgID；若msgID在roomID唯一，但在全局不唯一，则rowkey需要roomID的信息，采用Reversing the key的设计。

#### 单调增rowkey / 时间序

应避免使用单调增长的id或者时间戳做为rowkey，否则会出现请求集中在一个Region，然后再转移到下一个Region。若要上传时间序的数据，可以参考OpenTSDB的设计，其rowkey是[metric_type][event_timestamp]，虽然包含了时间戳，但不在最开头；同一时间都更新最新时间戳的值，由于metric_type有很多种，足以用来均衡负载。

## 合理的参数设置

#### 合理设置列族个数

列族个数需要在设计表的时候确定，每个列族下的列数可以动态扩展。建议的列族数是1-3个。
列族的划分主要按照访问模式，例如爬虫系统，对每个URL需要存储网页的历史版本和元信息，搜索时使用元信息，查看内容则访问历史版本，因此可以使用两个列族，分别存储历史版本和元信息。

####  设置TTL

如果业务上数据无需永久保存，可以设置TTL（Time To Live），HBase会自动清理过期的数据。

```shell
# 建表时设置，单位为秒。例子设置f1列族TTL为30天
hbase shell > create 't1', {NAME => 'f1', TTL => 2592000}

# 已存在表修改TTL
hbase shell > alter 't1', {NAME => 'f1', TTL => 2592000}
```

#### 设置VERSION

HBase默认VERSION是1，VERSION是对表的列族进行设置， 可以通过如下命令修改VERSION值

```shell
hbase shell> alter 't1', { NAME => 'f1', VERSIONS => 3 }
```

注意：不建议设置过大的VERSION，除非明确的知道历史版本对你很有用处，较大的VERSION会使得该列族大小成倍增加。

#### 保持HBase的Key简短

HBase是KeyValue数据库，数据单元是Cell，每个Cell都会存储Key(RowKey + Column Family + Column Qualifier + Timestamp)和Value。
因此，请保持RowKey、Column Family和Column Qulifier的简短，过长的名字会使得存储效率降低。尽量使用有意义的缩写，如myVeryImportantAttribute可以用via代替。

## 使用须知

#### 正确设置Scan Caching的行数

- 含义：Caching是Server端一次返回数据的行数，默认值是100，意即一次RPC请求中，Server端获取到100行数据后一次性返回；
- 影响：
  - 若Caching值设置过大，RegionServer需要在内存中存储所有需要返回的结果，可能导致OutOfMemory；
  - 若Caching值设置过大，Server获取这些数据需要很长时间（尤其是设置了过滤器），可能导致OutOfOrderScannerNextException；
  - 若Caching值设置过小，则获取N行数据需要N/Caching次RPC请求，速度慢；
- 设置：通过scan.setCaching()进行设置，接受的配置是行数，不是Byte，建议根据返回的总数据量进行估算，总额是10MB；若一行是10KB，cache设置为1000；若一行是2MB，cache设置为5。

#### 正确设置WriteBuffer大小

大量的Put请求，请关闭autoFlush()，避免每一个Put请求都向Server端发送RPC请求

- 含义：客户端会累积Put请求，当数据累积到Buffer大小时，触发一次RPC请求。
- 风险：若客户端程序崩溃，则Buffer之中的Put请求会丢失！
- 影响：
  - 若设置太大，则会导致OutOfMemory
  - 若设置太小，则触发多次RPC请求；
- 设置：
  - htable.setAutoFlush(false) # 启用Write Buffer，默认不启用
  - htable.setWriteBufferSize(long writeBufferSize) #参数是Byte，建议2MB~5MB，一般不允许超过10MB。若设置为5MB，则是htable.setBufferSize(5*1024*1024)。

#### 正确设置Batch大小

- 含义：当行很大的时候，Batch设置每次next()返回的列数。例如一个表有10000行，设置Caching=10，Batch=100，则每100列被当做一个Result，每次RPC返回10 个Result即1000列；
- 影响：对于很大的行，需要设置合理的Batch大小，以免发生OOME。
- 设置：scan.setBatch()进行设置，接受的是batch的列数。

#### 批量导入使用Bulkload

当大批量导入数据的时候，推荐使用HFileOutFormat（Bulkload），使用单个Client导数据瓶颈在Client，无法充分利用HBase的扩展性。

# 使用陷进

##### HBase时间戳单位是毫秒

HBase时间戳是13位的数字，务必注意！对于设置了TTL的表，如果Put数据时使用10位的时间戳，则写入成功，但无法查询得到！