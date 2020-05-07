原文地址：https://cloud.spring.io/spring-cloud-netflix/multi/multi_pr01.html  

**2.1.2.BUILD-SNAPSHOT**

该项目通过自动配置并绑定到Spring Environment和其他Spring编程模型习惯用法，为Spring Boot应用程序提供了Netflix OSS集成。使用一些简单的批注，您可以快速启用和配置应用程序内部的通用模式，并使用经过测试的Netflix组件构建大型分布式系统。提供的模式包括服务发现（Eureka），断路器（Hystrix），智能路由（Zuul）和客户端负载平衡（Ribbon）。

# 1服务发现：Eureka客户

服务发现是基于微服务的体系结构的主要宗旨之一。尝试手动配置每个客户端或某种形式的约定可能很困难并且很脆弱。Eureka是Netflix Service Discovery服务器和客户端。可以将服务器配置和部署为高可用性，每个服务器将有关已注册服务的状态复制到其他服务器。

## 1.1. 如何包含 Eureka 客户端

要将Eureka Client包含在您的项目中，请使用组ID为org.springframework.cloud的启动程序以及工件ID为spring-cloud-starter-netflix-eureka-client的工件。请参阅Spring Cloud Project页面以获取有关使用当前Spring Cloud Release Train设置构建系统的详细信息

## 1.2使用Enreka进行注册

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

## 1.3使用Eureka服务器进行身份验证

如果其中一个eureka.client.serviceUrl.defaultZone URL中嵌入了凭据（curl样式，如下所示：user：password @ localhost：8761 / eureka），则会将HTTP基本身份验证自动添加到您的eureka客户端。对于更复杂的需求，您可以创建类型为DiscoveryClientOptionalArgs的@Bean并将ClientFilter实例注入其中，所有这些都应用于从客户端到服务器的调用。

由于Eureka的限制，无法支持每服务器的基本身份验证凭据，因此仅使用找到的第一组凭据

## 1.4状态页和健康指示器

Eureka实例的状态页面和运行状况指示器分别默认为/ info和/ health，这是Spring Boot Actuator应用程序中有用端点的默认位置。如果您使用非默认上下文路径或servlet路径（例如server.servletPath = / custom），则即使对于Actuator应用程序，也需要更改它们。下面的示例显示两个设置的默认值：

```yaml
eureka:
  instance:
    statusPageUrlPath: ${server.servletPath}/info
    healthCheckUrlPath: ${server.servletPath}/health
```

这些链接显示在客户端使用的元数据中，并在某些情况下用于确定是否将请求发送到您的应用程序，因此，如果请求准确，将很有帮助。

在Dalston中，还需要在更改该管理上下文路径时设置状态和运行状况检查URL。从Edgware开始就删除了此要求。

## 1.5注册安全的应用程序

如果您的应用程序希望通过HTTPS进行联系，则可以在EurekaInstanceConfig中设置两个标志：

- `eureka.instance.[nonSecurePortEnabled]=[false]`
- `eureka.instance.[securePortEnabled]=[true]`

这样做使Eureka发布实例信息，该实例信息显示出对安全通信的明确偏好。对于以这种方式配置的服务，Spring Cloud DiscoveryClient始终返回以https开头的URI。同样，以这种方式配置服务时，Eureka（本机）实例信息具有安全的运行状况检查URL。

由于Eureka在内部工作的方式，它仍然会为状态和主页发布非安全URL，除非您也明确地覆盖了这些URL。您可以使用占位符来配置eureka实例URL，如以下示例所示：

**application.yml.**

```yaml
eureka:
  instance:
    statusPageUrl: https://${eureka.hostname}/info
    healthCheckUrl: https://${eureka.hostname}/health
    homePageUrl: https://${eureka.hostname}/
```

（请注意，$ {eureka.hostname}是本机占位符，仅在更高版本的Eureka中可用。您也可以使用Spring占位符来实现相同的功能-例如，通过使用$ {eureka.instance.hostName}。）

如果您的应用程序在代理之后运行，并且SSL终止在代理中（例如，如果您在Cloud Foundry或其他平台中作为服务运行），则需要确保代理被“转发”的标头被拦截和处理通过应用程序。如果嵌入在Spring Boot应用程序中的Tomcat容器具有针对'X-Forwarded-\ *`标头的显式配置，则会自动发生。应用程序提供的指向自身的链接错误（错误的主机，端口或协议）表明此配置错误。

## 1.6Eureka’s 健康检查

默认情况下，Eureka使用客户端心跳来确定客户端是否启动。除非另有说明，否则按照Spring Boot Actuator的规定，Discovery Client不会传播应用程序的当前运行状况检查状态。因此，在成功注册后，Eureka始终宣布该应用程序处于“启动”状态。可以通过启用Eureka运行状况检查来更改此行为，这会导致应用程序状态传播到Eureka。结果，所有其他应用程序都不会将流量发送到处于“ UP”状态以外的其他状态的应用程序。以下示例显示如何为客户端启用运行状况检查：

**application.yml.** 

```yaml
eureka:
  client:
    healthcheck:
      enabled: true
```

eureka.client.healthcheck.enabled = true只能在application.yml中设置。在bootstrap.yml中设置该值会导致不良的副作用，例如在Eureka中以UNKNOWN状态注册。

如果您需要对运行状况检查进行更多控制，请考虑实现自己的com.netflix.appinfo.HealthCheckHandler。

## 1.7实例和客户端的Eureka元数据

值得花费一些时间来了解Eureka元数据的工作原理，因此您可以在平台上合理使用它。有用于信息的标准元数据，例如主机名，IP地址，端口号，状态页和运行状况检查。这些都发布在服务注册表中，并由客户端用于以直接方式联系服务。可以在eureka.instance.metadataMap中将其他元数据添加到实例注册中，并且可以在远程客户端中访问此元数据。通常，除非使客户端知道元数据的含义，否则其他元数据不会更改客户端的行为。在本文档后面将介绍几种特殊情况，其中Spring Cloud已经为元数据映射分配了含义。

### 1.7.1在Cloud Foundry上使用Eureka

Cloud Foundry具有全局路由器，因此同一应用程序的所有实例都具有相同的主机名（其他具有类似架构的PaaS解决方案具有相同的排列）。这不一定是使用Eureka的障碍。但是，如果您使用路由器（建议或什至是强制性的，具体取决于平台的设置方式），则需要显式设置主机名和端口号（安全或不安全），以便它们使用路由器。您可能还希望使用实例元数据，以便可以区分客户端上的实例（例如，在自定义负载平衡器中）。默认情况下，eureka.instance.instanceId为vcap.application.instance_id，如以下示例所示：

**application.yml.** 

```yaml
eureka:
  instance:
    hostname: ${vcap.application.uris[0]}
    nonSecurePort: 80
```

根据在Cloud Foundry实例中设置安全规则的方式，您也许可以注册并使用主机VM的IP地址进行直接的服务到服务的调用。Pivotal Web服务（PWS）尚不提供此功能。

### 1.7.2在AWS上使用Eureka

如果计划将应用程序部署到AWS云，则必须将Eureka实例配置为可感知AWS。您可以通过如下自定义EurekaInstanceConfigBean来实现：

```java
@Bean
@Profile("!default")
public EurekaInstanceConfigBean eurekaInstanceConfig(InetUtils inetUtils) {
  EurekaInstanceConfigBean b = new EurekaInstanceConfigBean(inetUtils);
  AmazonInfo info = AmazonInfo.Builder.newBuilder().autoBuild("eureka");
  b.setDataCenterInfo(info);
  return b;
}
```

### 1.7.3更改Eureka实例ID

一个普通的Netflix Eureka实例注册的ID等于其主机名（即，每个主机仅提供一项服务）。Spring Cloud Eureka提供了明智的默认值，其定义如下：

${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}}

一个示例是myhost：myappname：8080。

通过使用Spring Cloud，您可以通过在eureka.instance.instanceId中提供唯一的标识符来覆盖此值，如以下示例所示：

**application.yml.** 

```yaml
eureka:
  instance:
    instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

通过前面示例中显示的元数据和在本地主机上部署的多个服务实例，在其中插入随机值以使实例唯一。在Cloud Foundry中，vcap.application.instance_id是在Spring Boot应用程序中自动填充的，因此不需要随机值。

## 1.8使用EurekaClient

一旦拥有作为发现客户端的应用程序，就可以使用它从Eureka服务器发现服务实例。一种方法是使用本机com.netflix.discovery.EurekaClient（与Spring Cloud DiscoveryClient相对），如以下示例所示：

```java
@Autowired
private EurekaClient discoveryClient;

public String serviceUrl() {
    InstanceInfo instance = discoveryClient.getNextServerFromEureka("STORES", false);
    return instance.getHomePageUrl();
}
```

不要在@PostConstruct方法或@Scheduled方法（或尚未启动ApplicationContext的任何地方）中使用EurekaClient。它是在SmartLifecycle（阶段= 0）中初始化的，因此最早可以依靠它的状态是在另一个具有较高阶段的SmartLifecycle中。

### 1.8.1没有Jersey的EurekaClient

默认情况下，EurekaClient使用Jersey进行HTTP通信。如果希望避免来自Jersey的依赖关系，可以将其从依赖关系中排除。Spring Cloud基于Spring RestTemplate自动配置传输客户端。以下示例显示排除了Jersey：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    <exclusions>
        <exclusion>
            <groupId>com.sun.jersey</groupId>
            <artifactId>jersey-client</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.sun.jersey</groupId>
            <artifactId>jersey-core</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.sun.jersey.contribs</groupId>
            <artifactId>jersey-apache-client4</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## 1.9本地Netflix EurekaClient的替代产品

您无需使用原始的Netflix EurekaClient。而且，通常在某种包装器后面使用它会更方便。Spring Cloud通过逻辑Eureka服务标识符（VIP）而非物理URL支持Feign（REST客户端构建器）和Spring RestTemplate。要使用固定的物理服务器列表配置Ribbon，可以将<client> .ribbon.listOfServers设置为以逗号分隔的物理地址（或主机名）列表，其中<client>是客户端的ID。

您还可以使用org.springframework.cloud.client.discovery.DiscoveryClient，它为发现客户端提供一个简单的API（非Netflix专用），如以下示例所示：

```java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
    List<ServiceInstance> list = discoveryClient.getInstances("STORES");
    if (list != null && list.size() > 0 ) {
        return list.get(0).getUri();
    }
    return null;
}
```

## 1.10为什么注册服务这么慢？

成为实例还涉及到注册表的定期心跳（通过客户端的serviceUrl），默认持续时间为30秒。直到实例，服务器和客户端在其本地缓存中都具有相同的元数据后，客户端才能发现该服务（因此可能需要3个心跳）。您可以通过设置eureka.instance.leaseRenewalIntervalInSeconds来更改周期。将其设置为小于30的值可以加快使客户端连接到其他服务的过程。在生产中，最好使用默认值，因为服务器中的内部计算对租约续订期进行了假设。

## 1.11区域

如果您已将Eureka客户端部署到多个区域，则您可能希望这些客户端在同一区域内使用服务，然后再尝试其他区域中的服务。要进行设置，您需要正确配置Eureka客户端。

首先，您需要确保已将Eureka服务器部署到每个区域，并且它们彼此对等。有关更多信息，请参见区域和区域部分。

接下来，您需要告诉Eureka服务位于哪个区域。您可以通过使用metadataMap属性来实现。例如，如果将服务1部署到区域1和区域2，则需要在服务1中设置以下Eureka属性：

**Service 1 in Zone 1**

```properties
eureka.instance.metadataMap.zone = zone1
eureka.client.preferSameZoneEureka = true
```

**Service 1 in Zone 2**

```properties
eureka.instance.metadataMap.zone = zone2
eureka.client.preferSameZoneEureka = true
```