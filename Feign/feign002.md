# Ribbon

该模块包括一个feign目标和客户端适配器，以利用功能区。

## Conventions-约定

这种集成依赖于Feign Target.url（）像https：// myAppProd一样进行编码，其中myAppProd是功能区客户端或负载平衡器名称，并且设置了myAppProd.ribbon.listOfServers配置。

### RibbonClient

添加RibbonClient会覆盖Feign客户端的URL解析，并添加Ribbon提供的智能路由和弹性功能。

#### Usage-用法

代替

```java
MyService api = Feign.builder().target(MyService.class, "https://myAppProd-1234567890.us-east-1.elb.amazonaws.com");
```

做

```java
MyService api = Feign.builder().client(new RibbonClient()).target(MyService.class, "https://myAppProd");
```

### LoadBalancingTarget

使用或扩展LoadBalancingTarget将通过功能区启用动态URL发现，包括增加服务器请求计数。

代替

```java
MyService api = Feign.builder().target(MyService.class, "https://myAppProd-1234567890.us-east-1.elb.amazonaws.com");
```

做

```java
MyService api = Feign.builder().target(LoadBalancingTarget.create(MyService.class, "https://myAppProd"));
```