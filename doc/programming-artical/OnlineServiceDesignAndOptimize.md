# 概述

如何根据需求设计一个能扛住一定量QPS且接口延时稳定（高可用）的服务，其中需要注意很多细节。以真实项目中的服务设计为例，探讨一下当前的高并发低延时服务的设计以及更多需要改进的地方。

# 需求背景

提供用户数据的RPC/HTTP服务。业务方通过用户的ID，获取当前业务被授权的用户数据。

用户量级10亿+，pb压缩后T级别存储，服务需要满足的几个点包括：

1. 50w+的峰值QPS
2. P99<20ms的接口延时
3. 分钟级延时的用户数据
4. 用户数据权限控制

接口设计大致如下，这里以设备的ID为例，给定其中的一个接口定义。

```java
public interface OnlineService {
    /**
     * 通过设备ID获取用户标签
     * @param deviceId 设备ID，即idfa、imei、iqid
     * @param idType PC, MOBILE or TV
     * @param auth 授权认证
     * @return tag_field, value: tagInfo
     */
    Map<String, Tags> getSingleDeviceTags(String deviceId, DeviceIdType idType, String auth);
}
```

# 框架设计

参考大数据的经典[Lambda框架](https://www.cnblogs.com/cciejh/p/lambda-architecture.html)的分层设计，将离线数据和实时数据分开处理，大致框架图如下图。

![service-design](/img/online-service-design.png)

我们将系统分成了Batch层、Speed层和Server层。

Batch层即离线批量数据层，每天定期将HBase中的用户数据导入Hikv。Hikv和Couchbase作为数据存储介质。

Speed层即实时数据层，接收来自用户实时事件总线（我们的一个内部服务）数据，加工成用户标签，提供用户的实时标签数据。

Server层主要做数据的合并逻辑，包括数据的封装和权限的筛选，将数据结果返回给到用户。

## 技术选型

### DB的选择

离线数据使用HBase存储无疑是最合适的，HBase的列存储以及批量读写的性质，适合用户标签数据。

数据缓存选择了Hikv和Couchbase，Hikv是基于SSD的KV数据存储解决方案，特点是容量大，但性能相对Couchbase会弱一些。Couchbase是内存数据库解决方案，特点是延时低，但容量相对有限制。

我们的用户数据量达到10亿级别，存储量在T级别，公司内部独立的Redis或Couchbase集群都无法支撑如此大的容量，因此这里使用了两级缓存的设计，用Hikv做底层缓存，Couchbase作为上层缓存。

### 服务接口选择

我们同时提供RPC和HTTP的接口。RPC是基于Apache Dubbo实现，Dubbo无论是在功能上或是性能上，无疑都是很优秀的RPC服务框架。

## 设计细节

### 用户数据的取舍

考虑到在线服务主要是用于线上的用户定向以及一些推荐场景中的用户召回，因此我们舍弃了一部分用户数据，只保留了月活用户的特定标签数据。

这样的做法是为了在让数据充分服务业务的条件下，降低存储压力。

### 缓存的使用细节

高并发的服务中，常见的缓存问题包括：缓存击穿、缓存穿透、缓存雪崩的现象。这里把CB当成缓存，Hikv当成传统数据库，来探讨一下系统设计中如何避免这些问题。

先简单描述一下几种case：

**缓存穿透：**缓存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求，如发起为id为“-1”的数据或id为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。

常用的解决方案是接口层的校验和空数据设置。

在这里，我们会做用户ID和用户权限的校验；在都没有数据的情况下，主动构造一条空数据写入缓存中，设置较短的TTL，避免这样的ID重复调用。

**缓存击穿：**缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。

常用的解决方案是热点数据永不过期或加上互斥锁。

在这里，我们不太用担心这个问题，因为线上用户请求可以认为是均匀的，因此不会大量的同时过期，另外Hikv的性能也是相对OK的，高并发读写Hikv是可以支持的，加锁反而会影响效率。有一个可能的场景是，流量高峰到来之前，进行缓存预热。

**缓存雪崩：**缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

常用的解决办法是设置缓存过期时间随机。即数据进缓存时，加上一个随机的时间戳。

在这里我们两层缓存时间使用的是异步的数据Set，若CB没查到数据，而Hikv有，则直接从Hikv中取到数据返回。

### 服务部署细节

服务基于Docker容器化部署，可灵活的伸缩扩容。

服务使用多机房、多数据中心的形式部署，借助Dubbo的负载均衡以及内部扩展的同区域路由的功能，满足线上不同机房的业务调用。

### 系统保护

面对大QPS的情况（如过年抢红包），有如下建议。

服务端：接入Sentinel保障微服务的稳定性。在某个服务可能过多地占用资源，我们需要可配置方式地进行限制某个服务的调用数，并且在服务发生故障时，不级联影响到其他核心功能的正常使用，对这类服务进行快速失败或者是触发备用降级功能。

业务调用方：做好及时熔断和服务降级。

### 系统监控

服务接入接口的监控、调用链路监控、日志监控。

## 性能瓶颈

在目前框架的设计下，Speed层是存在比较明显的性能瓶颈的。实时数据的更新必然造成数据的频繁读写。

目前的设计，在上游的“实时事件总线服务”中，会对数据流进行一次清洗和聚合，保证数据的有效性，也可以削弱数据流量。另外，实时的标签仅限于时间敏感性高且业务效益大的标签，如时间敏感标签，包括观影时间，或是行为敏感标签，如银行卡的支付绑定行为。

在数据流量稳定且可控的情况下，存储的选择可以考虑redis，redis的数据结构会让数据的操作变得更简单高效，目前独立的新客数据服务也是这么做的。

# 问题反思

### Q：缓存问题上为啥不用布隆过滤器？

布隆过滤器解析：[跳转](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485878&idx=2&sn=631a246d525f963459ff9262e11a0dd2)

布隆过滤器可以帮助我们过滤无效的用户ID，减少打到数据库上的请求。考虑在10亿级用户的场景中，布隆过滤器的初始化是比较不灵活的。且我们在防缓存穿透时的做法，数据库是能正常应付的。综合考虑，内部数据接口的调用，id可以认为是规范的，因此我们没有选择使用布隆过滤器。

### Q：为什么不用Redis

redis是好东西，选用主要取决于公司的技术栈和基础设施。