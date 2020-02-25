# 重点知识点归纳

大数据 + 数据库 + 中间件 + 微服务 + 编程语言

## 大数据

### Hadoop

Hadoop是一个由Apache基金会所开发的分布式系统基础架构。Hadoop的框架最核心的设计就是：HDFS和MapReduce。HDFS为海量的数据提供了存储，而MapReduce则为海量的数据提供了计算。

HDFS + MapReduce

### HBase

HBase是Google Bigtable的开源版本，是建立在HDFS之上，提供高可靠性、高性能、列存储、可伸缩、实时读写的数据库。它属于列式（Column-Oriented）NoSQL数据库，目前仅支持单行事务，没有成熟的二级索引方案，主要用来存储非结构化和半结构化的松散数据。

http://doc.gitlab.qiyi.domain/hbase/

### Impala

Apache Impala是一款介于MySQL 与Hive之间的实时交互式OLAP产品 。是由Cloudera开源的基于Hadoop的分析引擎。它在提升Hadoop上SQL查询性能的同时保持了相似的用户体验。

http://doc.gitlab.qiyi.domain/impala/

### Hive

Hive是一个建立在Hadoop上的支持SQL语法的数据仓库，具有很好的扩展性，擅长对大数据的存储和分析计算。用户输入的SQL语句，会被转换成一个或多个MapReduce jobs，并提交给Hadoop执行，最后返回查询结果给用户。

http://doc.gitlab.qiyi.domain/hive/

## 数据库

### Mysql

MySQL 是最流行的关系型数据库管理系统之一，其所使用的 SQL 语言是用于访问数据库的最常用标准化语言。

http://doc.gitlab.qiyi.domain/mysql/

### Redis

Redis是一个速度非常快的非关系数据库（non-relational database），它可以存储键（key）与5种不同类型的值（value）之间的映射（mapping），可以将存储在内存的键值对数据持久化到硬盘，可以使用复制特性来扩展读性能，还可以使用客户端分片来扩展写性能。

http://doc.gitlab.qiyi.domain/redis/jian-jie.html

### CouchBase

Couchbase是由CouchOne(创办人包括CouchDB的设计者)和Membase(由memcached的主要开发人员建立)两家公司在2011年初合并而来。Membase公司有一个名为Membase的产品，它是个键/值、持久化、可伸缩的解决方案，使用了memcached wire协议和SqlLite嵌入式存储引擎。CouchOne支持的CouchDB是个文档数据库，提供了端到端的复制方法，这对于移动与分布在不同位置的数据中心来说是很有用的。Couchbase是基于Membase与CouchDB开发的一款新产品，综合了两者的优点。

http://doc.gitlab.qiyi.domain/couchbase/

### 中间件

#### Kafka

Apache Kafka 是分布式发布-订阅消息系统。它最初由LinkedIn公司开发，之后成为Apache项目的一部分。Kafka是一种快速、可扩展的、设计内在就是分布式的，分区的和可复制的提交日志服务。

http://doc.gitlab.qiyi.domain/kafka/articles/introduce/kafkaIntroduce.html

#### RocketMQ

RocketMQ 是阿里巴巴开源的一款分布式消息中间件。具有高性能低延时抗堆积可扩展等特点。

http://doc.gitlab.qiyi.domain/rocketmq/

### 微服务

#### [Dubbo](http://dubbo.apache.org/zh-cn/index.html)

Apache Dubbo是一款高性能Java RPC框架。它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。

http://doc.gitlab.qiyi.domain/dubbo/