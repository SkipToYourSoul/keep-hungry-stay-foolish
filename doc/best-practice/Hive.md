# 介绍

Hive是一个建立在Hadoop上的支持SQL语法的数据仓库，具有很好的扩展性，擅长对大数据的存储和分析计算。用户输入的SQL语句，会被转换成一个或多个MapReduce jobs，并提交给Hadoop执行，最后返回查询结果给用户。

Hive适用于

- 海量数据；
- 批处理
- append-only data

不适用于

- OLTP
- 实时查询
- 行级别的数据更新

Hive底层的执行引擎除了MapReduce之外，还有Hive on Tez、 Hive on Spark，新型计算框架的支持是为了让 Hive查询相应更加快速。

# 最佳实践

## Map/Reduce任务task数量调整

MapReduce是Hive的默认引擎，Map/Reduce任务的task数量对任务的效率有很大的影响。

通常而言，Map数量与输入文件的数量一致，而Reduce数量则由用户配置或根据一定的规则决定。

### Map数量调整

MapReduce根据输入文件决定Mapper数目，每个Mapper处理一个输入分片。过少的Mapper会降低作业的并行度，过多的Mapper会增加集群任务调度的压力。

#### 调整split size来控制Map数量

控制作业分片的大小可以控制Mapper数量，前提是输入数据文件是可合并拆分（splittable）的。

split size的计算公式为：

```sql
split_size = max(mapred.min.split.size, min(mapred.max.split.size, dfs.block.size))
```

在处理大文件时，可以调整mapred.min.split.size为块大小整数倍，使每个Mapper可以处理多个块，这样可以减少启动的Mapper数量，但是如果一个分片中块数过多也会影响Mapper数据的本地化。

```sql
hive> SET mapred.min.split.size=1073741824; (每个map最少处理1GB数据)
```

#### 合并Map输入来减少Map数量

默认Mapper是不会合并输入的，可以通过以下参数设置合并Mapper输入，该设置可以把小文件（小于块大小的文件）打包到一个分片中：

```sql
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
set mapred.min.split.size=128000000;                --(每个map至少处理128M数据)
set mapred.min.split.size.per.node=128000000;       --(每个map至少处理128M数据)
set mapred.min.split.size.per.rack=128000000;       --(每个map至少处理128M数据)
```

**如果小文件很多，合并map输入会造成map运行缓慢，根源上解决问题必须减少上游生成的文件数量。**

### Reduce数量调整

Hive是这样确定Reduce数的：

1. ORDER BY（全局排序）或者COUNT的Job，Reducer个数始终为1；
2. 若指定`mapred.reduce.tasks`参数，则用该参数值；`mapred.reduce.tasks`默认值为-1，表示自动计算；
3. 若未指定`mapred.reduce.tasks`，Hive会自动计算reduce个数，基于以下两个配置：
   - `hive.exec.reducers.bytes.per.reducer`：每个reduce任务处理的数据量，默认为1G
   - `hive.exec.reducers.max`：每个任务最大的reduce数，集群默认为50

计算reducer数的公式如下：

```
reduce_number = min(input_size / hive.exec.reducers.bytes.per.reducer, hive.exec.reducers.max)
```

示例1：如果一个任务的输入总大小不超过1G,那么只会有一个reduce任务；

示例2：输入数据大小为9G+，默认情况下产生10个reduce；

一些经验如下：

- 建议通过调整`hive.exec.reducers.bytes.per.reducer`和`hive.exec.reducers.max`来控制reduce数量 因为数据量往往会随时间变化；
- 不建议采用固定的reduce数量，若reduce过多则会导致资源浪费，过少则会导致OOM异常；此外对于包含多个stage的查询，所有stage的reduce个数相同，这一般不是用户想要的；
- 启动/初始化reduce会消耗时间和资源，如果reduce数量过多会造成资源浪费；
- 一般情况下每个reduce都会生成一个输出文件，reduce数量过多会造成文件过多；

Reducer设置的目标为：

- 输出文件大小接近blocksize（256M），不要太小，最好也不要超过；
- Reduce执行时间在可接受范围内；

## 参数设置建议

#### hive.exec.reducers.max

可以通过任务最终生成的文件大小来计算合理的hive.exec.reducers.max，公式为：

```sql
hive.exec.reducers.max = min(2000, 最终文件大小 / 256M + 1)
```

例如一个每日例行任务，天分区的最终文件大小为100GB，那么合理的参数为

```sql
hive.exec.reducers.max = min(2000,  100GB / 256M + 1) = min(2000, 401) = 401
```

#### mapreduce.map.failures.maxpercent

允许map失败比例，某些任务中，对输入数据中不可控的脏数据，可以设置这个参数保证任务成功运行。

一般情况下不推荐。

#### hive.exec.dynamic.partition / hive.exec.dynamic.partition.mode

动态分区相关设置。第一个参数设为true则表示使用动态分区，第二个参数设为nonstrict则表示允许所有分区都是动态的。

动态分区插入是一个很方便的功能，用户无需指定具体的分区，数据可以自动分配到应该属于的分区，但是这种方式存在创建大量文件的问题。

动态分区一般有以下几个文件数相关的参数，**不建议修改默认值**！

一般而言，若任务超出了这些默认值，说明使用不合理，请修改业务逻辑：减少动态分区数，拆分任务等等

```sql
SET hive.exec.max.dynamic.partitions.pernode;（ 默认100，每一个mapper或者reducer能够创建的最大分区数目，如果超过了将会报错）
SET hive.exec.max.dynamic.partitions;（默认1000，一个query的map或者reducer能够创建的最大分区数目）
SET hive.exec.max.created.files;（默认100000，一个query所有mapper和reducer能够创建的最大文件个数，一个动态分区下通常会产生多个文件）
```

#### hive.auto.convert.join

该参数指定两个表join时默认自动会转换为map join，小表的size由**hive.mapjoin.smalltable.filesize**参数决定，默认为10M。

两个大表关联时，需要设置该参数为false，避免进行map join引起OOM。

进一步的，可以设置SMB Join。SMB 存在的目的主要是为了解决大表与大表间的 Join 问题，分桶其实就是把大表化成了“小表”，然后 Map-Side Join 解决之，这是典型的分而治之的思想。

```shell
set hive.enforce.bucketing=true;
set hive.enforce.sorting=true;
```

表优化数据目标：相同数据尽量聚集在一起。