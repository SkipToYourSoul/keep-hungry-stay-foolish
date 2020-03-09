# 高并发低延时服务

如何根据需求设计一个能扛住一定量QPS且接口延时稳定（高可用）的服务，其中需要注意很多细节。以真实项目中的服务设计为例，探讨一下当前的设计以及更多需要改进的地方。

## 背景

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

## 框架设计

参考大数据的经典[Lambda框架](https://www.cnblogs.com/cciejh/p/lambda-architecture.html)的分层设计，将离线数据和实时数据分开处理，大致框架图如下图。

![service-design](/img/online-service-design.png)

### 技术选型

db的选择

## 问题反思

### Q：缓存问题上为啥不用布隆过滤器？

布隆过滤器解析：[跳转](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485878&idx=2&sn=631a246d525f963459ff9262e11a0dd2)