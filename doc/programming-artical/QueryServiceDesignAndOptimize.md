# 概述

用户数据的查询服务在各DMP系统或用户画像系统里是很常用的，与用户交互的系统中，用户能忍受的交互延时极限也就是秒级别。

查询服务提供的数据能力主要包括：

1. 人群数量预估，比如运营场景下，我需要知道性别为男、常驻一线城市、商业兴趣是汽车的人群数量以辅助决策
2. 数据详情分布，比如以上场景，我需要知道这个条件下的用户的会员状态情况分布详情
3. 人群圈定，比如推送系统需要快速圈定符合某个条件的人群ID，对其进行消息推送

在这里探讨一下查询服务的设计以及更多可以改进的地方。

# 设计

举一个真实场景中画像系统的应用界面的例子。

![query](/img/query-service-1.jpeg)

系统提供多样性的圈选条件，圈选后可以查看人群数量、人群画像，进行人群推送等。

## 技术挑战

从查询服务提供的数据能力看，很直接的能联想到日常的查询query。在小数据量的情况下，用关系型数据库是很好解决的。在大数量的场景下，设计一个可以快速响应的用户数据查询服务，主要挑战有以下几点。

### 查询条件多样

用户圈选的人群条件是自由组合的，包括各种条件组合逻辑的搭配，条件的判断等。这就要求系统有识别条件并且转换条件为查询的能力。

进一步的，优化查询是很有必要的。

### 数据结构复杂

用户标签数据的结构，是一个多元的结构体（信息、权重等），而不是一个简单的字符串，这就需要系统在进行查询构建时下更大的功夫。

若是单维度的标签信息，可以考虑选用TiDb这样的方案，当然也可以考虑将单标签数据转换成json格式存储，但这样无论是存储成本或是解析成本，都是很大的。

### 系统吞吐量

系统的吞吐量与选用的数据方案挂钩，在系统设计时，需要考虑用线程池来控制当前执行的任务数。

## 技术选型

下图是一个常用的数据库决策树。考虑到数据量大和维度多的现状，我们选用Impala来实现。

![decision_tree](/img/decision_tree.png)

最初的实现方案中，是基于es实现的数据索引，使用es实现优点是索引后查询速度快，缺点是查询不够灵活，且数据量增大后构建索引的时间过长，加上es集群无法有力支持，因此就更新了数据方案。

# 实现

具体的实现主要分为以下几个步骤。

### 查询转换

查询转换置顶了一套统一的查询表达标准。主要是完成系统界面层级到查询表述层级的转换。

这里的查询，并不是真正的数据库查询，而是一套通用的查询表达标准，用其作为一层缓冲，这样底层更换数据库方案时，直接基于这套标准来构建对应的数据库查询即可。

```json
// 查询转换示例
// 单条件
ConditionQuery1 = "
{
  "range":{
    "birth_year" : {
       "gt": 1,
       "lte" : 5
     }
  }
}
"
// 条件组合
{
  "and": [CondtionQuery1, ConditionQuery2]
}
```

### 查询构建

查询构建描述的是如何将统一的查询表达标准构建成数据库能识别的查询，具体需要看底层数据表和数据结构的设计。

在查询构建时，我们将查询条件描述为一个简单的树关系。

```java
 *          RootConditionGroup(ConditionGroupList without RelationOp)
 *                  |
 *              ----------------------
 *              |                     |
 *     SubConditionGroup    LeafConditionGroup(ConditionList)
 *              |
 *      ---------------------
 *      |                   |
 * LeafConditionGroup   LeafConditionGroup
```

基于这个关系的一些规则，可以帮助我们梳理条件之间搭配的合理性，以及指导后续的查询优化。

一个被构建出来的简单的query如下所示：

```sql
SELECT count(t_0.entity_id)
FROM main_table t_0,
     t_0.sex_col.item t_1
WHERE t_1.value >1 and t_1.value <= 5
```

### 查询优化

查询优化描述的是针对我们使用的具体数据解决方案，做对应的优化策略。

例如，impala查询中，我们需要避免JOIN条件中使用OR，而是用UNION来代替，参考：https://stackoverflow.com/questions/5901791/is-having-an-or-in-an-inner-join-condition-a-bad-idea

这就需要我们在根据查询条件生成查询时，查询构造器去自动的进行优化。

```sql
// 改写前
SELECT b.id AS id, a.sim AS sim, a.id1 AS id1, a.id2 AS id2
FROM film_similarity a INNER JOIN film_id b
WHERE a.id1 = b.id OR a.id2 = b.id

// 改写后
SELECT b.id AS id, a.sim AS sim, a.id1 AS id1, a.id2 AS id2
FROM film_similarity a INNER JOIN film_id b
WHERE a.id1 = b.id
UNION
SELECT b.id AS id, a.sim AS sim, a.id1 AS id1, a.id2 AS id2
FROM film_similarity a INNER JOIN film_id b
WHERE a.id2 = b.id
```

在进行优化时，还需要注意各条件之间的拆分和合并，其实就是or或and条件的交换律和结合律的情况，类似我们的加减乘除法的操作。

上述例子中，若表a有X条（十亿量级）数据，表b有Y条（万量级）数据，根据分析，JOIN结果最多有2X条记录。但是因为在JOIN CONDITION中使用了OR语法，IMPALA无法做一些优化（如Hash Join等），其做法是先产生所有可能的组合，再根据JOIN CODITION进行过滤，即会产生X*Y条（十万亿）中间结果，最后过滤出2X条。在产生巨大无比的中间结果这一步非常耗时。

使用UNION改写后，Impala进行应用优化，产生的中间结果只有2X条。

查询优化的方案有且不止一种，具体都需要根据实际中的使用场景和技术方案来定制。

# 思考

### Q：是否有更合适的数据使用方案

ClickHouse是一个可以调研替代的使用方案，其性能在该数据场景下会表现得更优异，且支持增量的更新。

### Q：数据的时效性

诚然，impala是需要批量数据更新的，数据延时只能做到T+1。若考虑数据延时性更低的场景，则需要构建speed层，进行更快速的数据更新。可以选用TiDb作为speed层的存储介质。