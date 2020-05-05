老文档地址：https://cloud.spring.io/spring-cloud-netflix/multi/multi_pr01.html

新文档地址：https://cloud.spring.io/spring-cloud-netflix/reference/html/#service-discovery-eureka-clients

# Spring Cloud Netflix

**3.0.0.BUILD-SNAPSHOT**

该项目通过自动配置并绑定到Spring Environment和其他Spring编程模型习惯用法，为Spring Boot应用程序提供了Netflix OSS集成。使用一些简单的批注，您可以快速启用和配置应用程序内部的通用模式，并使用经过测试的Netflix组件构建大型分布式系统。提供的模式包括服务发现（Eureka），断路器（Hystrix），智能路由（Zuul）和客户端负载平衡（Ribbon）。

## 1服务发现：Eureka客户

服务发现是基于微服务的体系结构的主要宗旨之一。尝试手动配置每个客户端或某种形式的约定可能很困难并且很脆弱。Eureka是Netflix Service Discovery服务器和客户端。可以将服务器配置和部署为高可用性，每个服务器将有关已注册服务的状态复制到其他服务器。

### 1.1. 如何包含 Eureka 客户端

要将Eureka Client包含在您的项目中，请使用组ID为org.springframework.cloud的启动程序以及工件ID为spring-cloud-starter-netflix-eureka-client的工件。请参阅Spring Cloud Project页面以获取有关使用当前Spring Cloud Release Train设置构建系统的详细信息

### 1.2使用Enreka进行注册

客户端向Eureka注册时，它将提供有关自身的元数据，例如主机，端口，运行状况指示器URL，主页和其他详细信息。Eureka从属于服务的每个实例接收心跳消息。如果心跳在可配置的时间表上进行故障转移，则通常会将实例从注册表中删除。

以下示例显示了一个最小的Eureka客户端应用程序：

```java
@SpringBootApplication
@RestController
public class Application {

    @RequestMapping("/")
    public String home() {
        return "Hello world";
    }

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```

请注意，前面的示例显示了一个普通的Spring Boot应用程序。通过在类路径上使用spring-cloud-starter-netflix-eureka-client，您的应用程序将自动向Eureka Server注册。需要进行配置才能找到Eureka服务器，如以下示例所示：

**application.yml**

```yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

在前面的示例中，defaultZone是一个魔术字符串后备值，它为任何不表达首选项的客户端提供服务URL（换句话说，这是一个有用的默认值）。

defaultZone属性区分大小写，并且要求使用驼峰式大小写，因为serviceUrl属性是Map <String，String>。因此，defaultZone属性不遵循默认区域的正常Spring Boot 蛇例(蛇壳) 惯例。

默认应用程序名称（即服务ID），虚拟主机和非安全端口（从环境获取）为$ {spring.application.name}，$ {spring.application.name}和$ {server.port}。

在类路径上具有spring-cloud-starter-netflix-eureka-client会使该应用程序同时进入Eureka“实例”（即，它自己注册）和“客户端”（它可以查询注册表以定位其他服务）。实例行为由eureka.instance。*配置键驱动，但是如果确保您的应用程序具有spring.application.name的值（这是Eureka服务ID或VIP的默认值），则默认值很好。

有关可配置选项的更多详细信息，请参见EurekaInstanceConfigBean和EurekaClientConfigBean。

See [EurekaInstanceConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java) and [EurekaClientConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java) for more details on the configurable options.

要禁用Eureka Discovery Client，可以将eureka.client.enabled设置为false。当spring.cloud.discovery.enabled设置为false时，Eureka Discovery Client也将被禁用。

### 1.3使用Eureka服务器进行身份验证

如果其中一个eureka.client.serviceUrl.defaultZone URL中嵌入了凭据（curl样式，如下所示：user：password @ localhost：8761 / eureka），则会将HTTP基本身份验证自动添加到您的eureka客户端。对于更复杂的需求，您可以创建类型为DiscoveryClientOptionalArgs的@Bean并将ClientFilter实例注入其中，所有这些都应用于从客户端到服务器的调用。

由于Eureka的限制，无法支持每服务器的基本身份验证凭据，因此仅使用找到的第一组凭据

### 1.4状态页和健康指示器

