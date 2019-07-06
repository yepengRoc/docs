## nameserver

### routerinfomanager路由信息管理

​		rocketmq基于订阅发布机制，一个topic拥有多个消息队列，一个broker为每一个主题默认创建4个读队列4个写队列。多个broker组成一个集群，bokername有相同的多台broker组成master-slaver架构，brokerid为0代表master,大于0代表slave.brokerliveinfo中的lastUpdateTimeStamp存储上次broker心跳包的时间

|      |
| ---- |
|      |

