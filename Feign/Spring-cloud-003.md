# 3.断路器：Hystrix客户端

Netflix创建了一个名为Hystrix的库，该库实现了断路器模式。在微服务架构中，通常有多个服务调用层，如以下示例所示：

图3.1。微服务图

图



较低级别的服务中的服务故障可能会导致级联故障，直至用户。当对特定服务的调用超过circuitBreaker.requestVolumeThreshold（默认值：20个请求）并且故障百分比大于由metrics.rollingStats.timeInMilliseconds定义的滚动窗口中的circuitBreaker.errorThresholdPercentage（默认值：> 50％）（默认值：10秒），电路断开，无法拨打电话。在错误和断路的情况下，开发人员可以提供后备功能。

图3.2。Hystrix后备可防止级联故障

图

开路可以停止级联故障，并让不堪重负的服务故障时间得以恢复。回退可以是另一个受Hystrix保护的调用，静态数据或合理的空值。可以将回退链接在一起，以便第一个回退进行其他业务调用，而后者又回退到静态数据。

## 3.1如何包括Hystrix

