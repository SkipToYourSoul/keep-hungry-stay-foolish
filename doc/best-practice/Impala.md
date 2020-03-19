# 介绍

[Apache Impala](https://impala.incubator.apache.org/)是一款介于MySQL 与Hive之间的实时交互式OLAP产品 。MySQL支持GB级别数据的秒级查询，如果数据量过大，需要分库分表。Hive支持TB~PB级数据，批处理模式，查询响应一般在分钟级别。

Impala支持比MySQL更大的数据量（TB级别），兼容Hive metadata和HiveQL，可以直接查询Hive数据，速度比Hive快10倍以上。

Impala具有以下特点：

- 速度快：Impala提供基于Hadoop的低延时、高并发的分析查询（不同于Apache Hive的批处理框架）；
- 可扩展：Impala计算能力能够随结点数水平扩展；而MySQL在达到TB之后遇到瓶颈；
- 易接入：对于Hive用户，Impala使用相同的Metadata，兼容HiveQL，迁移成本低；
- 易使用：支持SQL，提供Shell、Hue、JDBC多种接入形式，易于和BI工具集成；
- 高安全：Impala使用Kerberos认证且通过Sentry和Hadoop安全机制集成，能确保用户访问授权的数据。

# 最佳实践

#### JOIN顺序大表在前，小表在后

Impala JOIN时有多种策略，默认采用`Broadcast JOIN`，JOIN右侧的表会分发到各个结点，左侧的表则会顺序遍历和内存中右侧表做JOIN。因此右侧表的大小决定了JOIN消耗的内存。

注意：Impala会自动根据STATS信息优化JOIN顺序。建议SQL中也遵循大表在前，小表在后的原则，以避免STATS不准确时JOIN消耗大量内存。但由于统计信息缺失等原因，Impala自动调整的JOIN顺序可能不合理，可以用`STRAIGHT_JOIN`关键字强制指定按SQL顺序执行JOIN。

```sql
// large_table大小在5G，huge_table大小在300G，但是大表根据条件过滤后的大小只有几十MB。Impala根据统计信息自动优化了JOIN顺序(可以从Profile结果看出)，将tmp在前，large_table在后，执行时间为3分钟
// 使用STRAIGHT_JOIN关键字后，Impala不再调整JOIN顺序，执行时间降低为1秒
SELECT STRAIGHT_JOIN tmp.a, COUNT(*)
FROM large_table r INNER JOIN
(SELECT a FROM huge_table WHERE dt = '20170801') tmp
on r.a = tmp.a
GROUP BY 1
```

#### 采用Parquet格式+Snappy压缩

建议采用Parquet+Snappy，仅当表特别大（TB级别）时，使用Parquet+GZ。不要使用Text格式，Text比Parquet要慢一倍。

指导标准：

- 对于大表和追求性能的表，为了效率和扩展性，使用Parquet文件格式；
- 为了方便的导入原始数据，使用Text格式的表，而不是RCFile或者SequenceFile，并在后续处理过程中将其转成Parquet格式。
- Snappy压缩解压的CPU开销很小，并能节省可观的存储空间。当你能够选择压缩方式时，尽量使用Snappy压缩。

另外，Impala建议PARQUET文件大小小于HDFS BLOCK大小（256 MB），因每个文件被一个结点处理，若文件大于BLOCK SIZE，则文件会分布在多个BLOCK，无法保证处理结点能本地读取所有数据。

#### 字段类型建议

- 尽量使用数字而不是字符串：使用数字类型能减少Parquet格式的存储空间，查询时的内存消耗
- STRING vs VARCHAR vs CHAR：推荐使用STRING。STRING相比VARCHAR更高效，与CHAR相比，CHAR会为不足长度的字符串补空格，容易引发语义问题。

#### 使用EXPLAIN和SUMMARY验证执行计划

在执行一个消耗大量资源的查询前，使用`EXPLAIN`语句来查看Impala计划如何并行、分发工作。如果发现效率低的环节，可以采取改变文件格式、Partition表、计算统计信息、添加查询HINTS等方式进行调优。在一个查询执行完毕之后，可以执行`SUMMARY`命令，来查看实际执行过程中性能相关的指标。

#### 语法细节建议

- 查询使用LIMIT：terminal输出会是瓶颈，如果需要全部的数据，将其写入到临时表
- 禁用SELECT *：使用`SELECT *`语法，查询会涉及整个文件（涉及所有列），可能会产生10-30倍非必要的网络、内存、CPU消耗，且查询速度也会大幅下降。
- 避免JOIN条件中使用OR而用UNION：参考：https://stackoverflow.com/questions/5901791/is-having-an-or-in-an-inner-join-condition-a-bad-idea
- 合理使用NOT IN：NOT IN后面适合接规模较小的数据集，如果数据集较大，则需使用LEFT JOIN替代，做相关优化。
- 合理使用UNION DISTINCT和UNION ALL：并集和全集

#### 应用建议

##### Impala应用支持失败重试

Impala服务应考虑加失败重试机制。因Impala采用的是`FAIL FAST`策略，和Hive不同。

其思想是每一次查询执行时间很短，执行期间恰好遇到结点失败的概率也低，且重试代价低；若为了失败的查询能够继续进行而保存中间结果，反而会拖慢正常查询的速度；

Hive因重试代价大，会保存中间结果，即使有结点失败查询也会完成，更为稳定。

> Currently, Impala doesn't support fault tolerance within a query. If a node fails in the middle of processing, the whole query has to be re-run. On the other hand, it will run these small queries often 10-50x faster than Hive. So, even if a node fails and it has to start over from the start, the total runtime will often beat Hive significantly. So, Impala has a big advantage in queries where the runtime is short enough that node failures during the query are unlikely. Assuming a MTBF of 1 year, you can assume that most queries which use less than a few node-months of time are unlikely to see a crash. For example, on a 100 node cluster, a query that runs for 10 minutes is only 1000 node-minutes and thus highly unlikely to experience a fault.

##### JDBC连接捕获异常后需关闭连接

若无关闭连接的逻辑，当Query异常后，连接会一直存在，占据队列的资源。

错误示例：

```java
public void main(){
  GenericQueryResult result = query(sql);
  closeConnection();
}

public GenericQueryResult query(String sql) throws SQLException {
  try {
    // 一些执行QUERY的逻辑
    return new GenericQueryResult(result, columns);
  } catch (SQLException e) {
    result.clear();
    throw e;
  } finally {
    // 一些清理的逻辑
  }
}
```

上述示例中query函数在遇到Exception时会抛出异常，但是外部调用query时没有捕获异常，导致closeConnection()未执行。

修复方法：调用query时捕获异常

##### 设置查询超时，避免预期之外的慢查询

通过JDBC查询Impala时，可以设置超时时间，避免意外情况导致的超长查询一直消耗队列的资源。

```java
// 设置查询超时（秒），当查询超过设置值时，抛出java.sql.SQLTimeoutException
stmt.setQueryTimeout(60);
```