# Spring Cloud Netflix

https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#configuration-properties

该项目通过自动配置并绑定到Spring Environment和其他Spring编程模型习惯用法，为Spring Boot应用程序提供了Netflix OSS集成。使用一些简单的批注，您可以快速启用和配置应用程序内部的通用模式，并使用经过测试的Netflix组件构建大型分布式系统。提供的模式包括服务发现（Eureka），断路器（Hystrix），智能路由（Zuul）和客户端负载平衡（Ribbon）。

## 1.Service Discovery: Eureka Clients

服务发现是基于微服务的体系结构的主要宗旨之一。尝试手动配置每个客户端或某种形式的约定可能很困难并且很脆弱。Eureka是Netflix Service Discovery服务器和客户端。可以将服务器配置和部署为高可用性，每个服务器将有关已注册服务的状态复制到其他服务器。

### [1.1。如何包括尤里卡客户](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#netflix-eureka-client-starter)

要将Eureka Client包含在您的项目中，请使用组ID为`org.springframework.cloud`和工件ID为的启动器`spring-cloud-starter-netflix-eureka-client`。有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参见[Spring Cloud Project页面](https://projects.spring.io/spring-cloud/)。

### [1.2。在尤里卡注册](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#registering-with-eureka)

当客户端向Eureka注册时，它会提供有关其自身的元数据，例如主机，端口，运行状况指示器URL，主页和其他详细信息。Eureka从属于服务的每个实例接收心跳消息。如果心跳在可配置的时间表上进行故障转移，则通常会将实例从注册表中删除。

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

请注意，前面的示例显示了一个普通的[Spring Boot](https://projects.spring.io/spring-boot/)应用程序。通过使用`spring-cloud-starter-netflix-eureka-client`类路径，您的应用程序将自动向Eureka Server注册。需要进行配置才能找到Eureka服务器，如以下示例所示：

application.yml

```
尤里卡：
  客户：
    serviceUrl：
      defaultZone：http：// localhost：8761 / eureka /
```

在前面的示例中，`defaultZone`是一个魔术字符串回退值，它为任何不表达首选项的客户端提供服务URL（换句话说，这是一个有用的默认值）。

|      | 该`defaultZone`属性区分大小写，并且需要使用驼峰式大小写，因为该`serviceUrl`属性是`Map`。因此，该`defaultZone`属性不遵循的正常Spring Boot蛇案惯例`default-zone`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

默认的应用程序名称（即，服务ID），虚拟主机，和非安全端口（从所拍摄的`Environment`）是`${spring.application.name}`，`${spring.application.name}`和`${server.port}`分别。

拥有`spring-cloud-starter-netflix-eureka-client`类路径会使应用程序同时成为Eureka的“实例”（即，它自己注册）和“客户端”（它可以查询注册表以定位其他服务）。实例行为由`eureka.instance.*`配置键驱动，但是如果确保您的应用程序具有值`spring.application.name`（这是Eureka服务ID或VIP 的默认值），则默认值很好。

有关[可](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java)配置选项的更多详细信息，请参见[EurekaInstanceConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java)和[EurekaClientConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java)。

要禁用Eureka Discovery Client，可以设置`eureka.client.enabled`为`false`。当尤里卡发现客户端也将被禁止`spring.cloud.discovery.enabled`设置为`false`。

### [1.3。使用Eureka服务器进行身份验证](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#authenticating-with-the-eureka-server)

如果其中一个`eureka.client.serviceUrl.defaultZone`URL嵌入了凭据（curl样式，如下所示`user:password@localhost:8761/eureka`），则会将HTTP基本身份验证自动添加到您的eureka客户端。对于更复杂的需求，您可以创建一个`@Bean`类型`DiscoveryClientOptionalArgs`并将其注入`ClientFilter`实例，所有这些都将应用于从客户端到服务器的调用。

|      | 由于Eureka的限制，无法支持每服务器的基本身份验证凭据，因此仅使用找到的第一组凭据。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [1.4。状态页和健康指示器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#status-page-and-health-indicator)

Eureka实例的状态页面和运行状况指示器分别默认为`/info`和`/health`，这是Spring Boot Actuator应用程序中有用端点的默认位置。如果您使用非默认的上下文路径或servlet路径（例如`server.servletPath=/custom`），则即使对于Actuator应用程序，也需要更改它们。下面的示例显示两个设置的默认值：

application.yml

```
尤里卡：
  实例：
    statusPageUrlPath：$ {server.servletPath} / info
    healthCheckUrlPath：$ {server.servletPath} / health
```

这些链接显示在客户端使用的元数据中，并在某些情况下用于确定是否将请求发送到您的应用程序，因此，如果请求准确，将很有帮助。

|      | In Dalston it was also required to set the status and health check URLs when changing that management context path. This requirement was removed beginning in Edgware. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [1.5. Registering a Secure Application](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#registering-a-secure-application)

If your app wants to be contacted over HTTPS, you can set two flags in the `EurekaInstanceConfig`:

- `eureka.instance.[nonSecurePortEnabled]=[false]`
- `eureka.instance.[securePortEnabled]=[true]`

Doing so makes Eureka publish instance information that shows an explicit preference for secure communication. The Spring Cloud `DiscoveryClient` always returns a URI starting with `https` for a service configured this way. Similarly, when a service is configured this way, the Eureka (native) instance information has a secure health check URL.

由于Eureka在内部工作的方式，它仍然会为状态和主页发布非安全URL，除非您也明确地覆盖了这些URL。您可以使用占位符来配置eureka实例URL，如以下示例所示：

application.yml

```
尤里卡：
  实例：
    statusPageUrl：https：// $ {eureka.hostname} / info
    healthCheckUrl：https：// $ {eureka.hostname} / health
    homePageUrl：https：// $ {eureka.hostname} /
```

（请注意，这`${eureka.hostname}`是本机占位符，仅在更高版本的Eureka中可用。您也可以使用Spring占位符来实现相同的目的-例如，通过使用`${eureka.instance.hostName}`。）

|      | 如果您的应用程序在代理后面运行，并且SSL终止在代理中（例如，如果您在Cloud Foundry或其他平台中作为服务运行），则需要确保拦截并处理了代理的“转发”标头通过应用程序。如果嵌入在Spring Boot应用程序中的Tomcat容器具有针对'X-Forwarded-\ *`标头的显式配置，则会自动发生。应用程序提供的指向自身的链接错误（错误的主机，端口或协议）表明此配置错误。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [1.6。尤里卡的健康检查](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#eurekas-health-checks)

默认情况下，Eureka使用客户端心跳来确定客户端是否启动。除非另有说明，否则按照Spring Boot Actuator的规定，Discovery Client不会传播应用程序的当前运行状况检查状态。因此，在成功注册后，Eureka始终宣布该应用程序处于“启动”状态。可以通过启用Eureka运行状况检查来更改此行为，这会导致应用程序状态传播到Eureka。结果，所有其他应用程序都不会将流量发送到处于“ UP”状态以外的其他状态的应用程序。以下示例显示如何为客户端启用运行状况检查：

application.yml

```
尤里卡：
  客户：
    健康检查：
      已启用：true
```

|      | `eureka.client.healthcheck.enabled=true`应该只在中设置`application.yml`。将该值设置为`bootstrap.yml`会产生不良的副作用，例如在Eureka中注册`UNKNOWN`状态。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果您需要对健康检查进行更多控制，请考虑实施自己的`com.netflix.appinfo.HealthCheckHandler`。

### [1.7。实例和客户端的Eureka元数据](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#eureka-metadata-for-instances-and-clients)

值得花费一些时间来了解Eureka元数据的工作原理，因此您可以在平台上以合理的方式使用它。有用于信息的标准元数据，例如主机名，IP地址，端口号，状态页和运行状况检查。这些都发布在服务注册表中，并由客户端用于以直接方式联系服务。可以将其他元数据添加到中的实例注册中`eureka.instance.metadataMap`，并且可以在远程客户端中访问此元数据。通常，除非使客户端知道元数据的含义，否则其他元数据不会更改客户端的行为。在本文档后面将介绍几种特殊情况，其中Spring Cloud已经为元数据映射分配了含义。

#### [1.7.1。在Cloud Foundry上使用Eureka](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#using-eureka-on-cloud-foundry)

Cloud Foundry具有全局路由器，因此同一应用程序的所有实例都具有相同的主机名（其他具有类似架构的PaaS解决方案具有相同的排列）。这不一定是使用尤里卡的障碍。但是，如果您使用路由器（建议或什至是强制性的，取决于平台的设置方式），则需要显式设置主机名和端口号（安全或非安全），以便它们使用路由器。您可能还希望使用实例元数据，以便可以区分客户端上的实例（例如，在自定义负载平衡器中）。默认情况下`eureka.instance.instanceId`为`vcap.application.instance_id`，如以下示例所示：

application.yml

```
尤里卡：
  实例：
    主机名：$ {vcap.application.uris [0]}
    nonSecurePort：80
```

根据在Cloud Foundry实例中设置安全规则的方式，您可能可以注册并使用主机VM的IP地址进行直接的服务到服务的调用。Pivotal Web服务（[PWS](https://run.pivotal.io/)）尚不提供此功能。

#### [1.7.2。在AWS上使用Eureka](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#using-eureka-on-aws)

如果计划将应用程序部署到AWS云，则必须将Eureka实例配置为可感知AWS。您可以通过如下自定义[EurekaInstanceConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java)来实现：

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

#### [1.7.3。更改尤里卡实例ID](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#changing-the-eureka-instance-id)

一个普通的Netflix Eureka实例注册的ID等于其主机名（即，每个主机仅提供一项服务）。Spring Cloud Eureka提供了明智的默认值，其定义如下：

```
${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}}
```

一个例子是`myhost:myappname:8080`。

通过使用Spring Cloud，您可以通过在中提供唯一标识符来覆盖此值`eureka.instance.instanceId`，如以下示例所示：

application.yml

```
尤里卡：
  实例：
    instanceId：$ {spring.application.name}：$ {vcap.application.instance_id：$ {spring.application.instance_id：$ {random.value}}}
```

通过前面示例中显示的元数据和在本地主机上部署的多个服务实例，在其中插入随机值以使实例唯一。在Cloud Foundry中，`vcap.application.instance_id`会在Spring Boot应用程序中自动填充，因此不需要随机值。

### [1.8。使用EurekaClient](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#using-the-eurekaclient)

一旦拥有作为发现客户端的应用程序，就可以使用它从[Eureka Server](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-eureka-server)发现服务实例。一种方法是使用本机`com.netflix.discovery.EurekaClient`（与Spring Cloud相对`DiscoveryClient`），如以下示例所示：

```
@Autowired
私人EurekaClient发现客户端；

公共字符串serviceUrl（）{
    InstanceInfo instance = DiscoveryClient.getNextServerFromEureka（“ STORES”，false）;
    返回instance.getHomePageUrl（）;
}
```

|      | 不要`EurekaClient`在`@PostConstruct`方法或`@Scheduled`方法中（或`ApplicationContext`可能尚未开始的任何地方）使用。它使用`SmartLifecycle`（`phase=0`）进行了初始化，因此最早可以依靠它的是处于`SmartLifecycle`更高阶段的另一个。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### [1.8.1。没有球衣的EurekaClient](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#eurekaclient-without-jersey)

默认情况下，EurekaClient使用Jersey进行HTTP通信。如果希望避免来自Jersey的依赖关系，可以将其从依赖关系中排除。Spring Cloud基于Spring自动配置传输客户端`RestTemplate`。以下示例显示排除了Jersey：

```
<依赖性>
    <groupId> org.springframework.cloud </ groupId>
    <artifactId> spring-cloud-starter-netflix-eureka-client </ artifactId>
    <排除>
        <排除>
            <groupId> com.sun.jersey </ groupId>
            <artifactId>球衣客户端</ artifactId>
        </ exclusion>
        <排除>
            <groupId> com.sun.jersey </ groupId>
            <artifactId>球衣核心</ artifactId>
        </ exclusion>
        <排除>
            <groupId> com.sun.jersey.contribs </ groupId>
            <artifactId> jersey-apache-client4 </ artifactId>
        </ exclusion>
    </ exclusions>
</ dependency>
```

### [1.9。本地Netflix EurekaClient的替代产品](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#alternatives-to-the-native-netflix-eurekaclient)

您无需使用原始Netflix `EurekaClient`。而且，通常在某种包装器后面使用它会更方便。Spring Cloud 通过逻辑Eureka服务标识符（VIP）而非物理URL 支持[Feign](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-feign)（REST客户端构建器）和[Spring`RestTemplate`](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-ribbon)。要使用固定的物理服务器列表配置Ribbon，可以将其设置`.ribbon.listOfServers`为以逗号分隔的物理地址（或主机名）列表，其中``为客户端ID。

您还可以使用`org.springframework.cloud.client.discovery.DiscoveryClient`，为发现客户端提供简单的API（非Netflix专用），如以下示例所示：

```
@Autowired
私有DiscoveryClient DiscoveryClient;

公共字符串serviceUrl（）{
    List <ServiceInstance> list = DiscoveryClient.getInstances（“ STORES”）;
    如果（list！= null && list.size（）> 0）{
        返回list.get（0）.getUri（）;
    }
    返回null;
}
```

### [1.10。为什么注册服务这么慢？](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#why-is-it-so-slow-to-register-a-service)

成为实例还涉及到注册表的定期心跳（通过客户端`serviceUrl`），默认持续时间为30秒。直到实例，服务器和客户端在其本地缓存中都具有相同的元数据后，客户端才能发现该服务（因此可能需要3个心跳）。您可以通过设置更改周期`eureka.instance.leaseRenewalIntervalInSeconds`。将其设置为小于30的值可以加快使客户端连接到其他服务的过程。在生产中，最好使用默认值，因为服务器中的内部计算对租约续订期进行了假设。

### [1.11。区域](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zones)

如果您已将Eureka客户端部署到多个区域，则您可能希望这些客户端在同一区域内使用服务，然后再尝试其他区域中的服务。要进行设置，您需要正确配置Eureka客户端。

首先，您需要确保已将Eureka服务器部署到每个区域，并且它们彼此对等。有关 更多信息，请参见[区域和区域](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-eureka-server-zones-and-regions)部分。

接下来，您需要告诉Eureka您的服务位于哪个区域。您可以使用`metadataMap`属性来实现。例如，如果`service 1`同时部署到`zone 1`和`zone 2`，则需要在中设置以下Eureka属性`service 1`：

**1区服务1**

```
eureka.instance.metadataMap.zone = zone1
eureka.client.preferSameZoneEureka = true
```

**2区服务1**

```
eureka.instance.metadataMap.zone = zone2
eureka.client.preferSameZoneEureka = true
```

### [1.12。刷新尤里卡客户](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#refreshing-eureka-clients)

默认情况下，`EurekaClient`bean是可刷新的，这意味着可以更改和刷新Eureka客户端属性。刷新发生时，客户端将从Eureka服务器中注销，并且可能会在短暂的时间内不提供给定服务的所有实例。消除这种情况的一种方法是禁用刷新Eureka客户端的功能。为此设置`eureka.client.refresh.enable=false`。

### [1.13。将Eureka与Spring Cloud LoadBalancer结合使用](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#using-eureka-with-spring-cloud-loadbalancer)

我们为Spring Cloud LoadBalancer提供支持`ZonePreferenceServiceInstanceListSupplier`。在`zone`从尤里卡实例元数据值（`eureka.instance.metadataMap.zone`）被用于设置的值`spring-clod-loadbalancer-zone`是由区用于过滤服务实例属性。

如果缺少该字段，并且该`spring.cloud.loadbalancer.eureka.approximateZoneFromHostname`标志设置为`true`，则可以使用服务器主机名中的域名作为该区域的代理。

如果没有其他区域数据源，则根据客户端配置（而不是实例配置）进行猜测。我们采用`eureka.client.availabilityZones`，这是从区域名称到区域列表的地图，并为实例自己的区域拉出第一个区域（即`eureka.client.region`，默认为“ us-east-1”，以与本机Netflix兼容） 。

## 2.Service Discovery: Eureka Server

本节介绍如何设置Eureka服务器。

### [2.1。如何包含Eureka服务器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#netflix-eureka-server-starter)

要将Eureka Server包含在您的项目中，请使用组ID为`org.springframework.cloud`和工件ID为的启动器`spring-cloud-starter-netflix-eureka-server`。有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参见[Spring Cloud Project页面](https://projects.spring.io/spring-cloud/)。

|      | 如果您的项目已经使用Thymeleaf作为其模板引擎，则可能无法正确加载Eureka服务器的Freemarker模板。在这种情况下，必须手动配置模板加载器： |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

application.yml

```
弹簧：
  freemarker：
    template-loader-path：classpath：/模板/
    首选文件系统访问：false
```

### [2.2。如何运行尤里卡服务器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-running-eureka-server)

以下示例显示了最小的Eureka服务器：

```java
@SpringBootApplication
@EnableEurekaServer
public class Application {

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```

该服务器具有一个主页，其中包含UI和HTTP API端点，可用于下方的常规Eureka功能`/eureka/*`。

以下链接具有一些Eureka背景知识：[磁通电容器](https://github.com/cfregly/fluxcapacitor/wiki/NetflixOSS-FAQ#eureka-service-discovery-load-balancer)和[google小组讨论](https://groups.google.com/forum/?fromgroups#!topic/eureka_netflix/g3p2r7gHnN0)。

|      | 由于Gradle的依赖性解析规则以及缺少父bom功能，具体取决于`spring-cloud-starter-netflix-eureka-server`可能导致应用程序启动失败。要解决此问题，请添加Spring Boot Gradle插件并按如下所示导入Spring Cloud Starter父Bom：build.gradle`buildscript {  dependencies {    classpath("org.springframework.boot:spring-boot-gradle-plugin:{spring-boot-docs-version}")  } } apply plugin: "spring-boot" dependencyManagement {  imports {    mavenBom "org.springframework.cloud:spring-cloud-dependencies:{spring-cloud-version}"  } }` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [2.3。高可用性，区域和区域](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-eureka-server-zones-and-regions)

Eureka服务器没有后端存储，但是注册表中的所有服务实例都必须发送心跳信号以使其注册保持最新（因此可以在内存中完成）。客户端还具有Eureka注册的内存缓存（因此，对于每个对服务的请求，它们都不必进入注册表）。

默认情况下，每个Eureka服务器也是Eureka客户端，并且需要（至少一个）服务URL来定位对等方。如果您不提供该服务，则该服务将运行并运行，但是它将使您的日志充满关于无法向对等方注册的噪音。

另请参阅[以下](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-ribbon)有关客户端对区域和区域[的功能区支持的详细信息](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-ribbon)。

### [2.4。独立模式](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-eureka-server-standalone-mode)

只要有某种监视器或灵活的运行时（例如Cloud Foundry），这两个缓存（客户端和服务器）以及心跳的组合就可以使独立的Eureka服务器对故障具有相当的恢复能力。在独立模式下，您可能希望关闭客户端行为，以使其不会继续尝试并无法到达其对等对象。下面的示例演示如何关闭客户端行为：

application.yml（独立Eureka服务器）

```
服务器：
  端口：8761

尤里卡：
  实例：
    主机名：localhost
  客户：
    registerWithEureka：否
    fetchRegistry：否
    serviceUrl：
      defaultZone：http：// $ {eureka.instance.hostname}：$ {server.port} / eureka /
```

请注意，`serviceUrl`指向与本地实例相同的主机。

### [2.5。同行意识](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-eureka-server-peer-awareness)

通过运行多个实例并要求它们相互注册，可以使Eureka更具弹性并可以使用。实际上，这是默认行为，因此要使其正常工作，您需要做的就是`serviceUrl`向对等方添加一个有效值，如以下示例所示：

application.yml（两个对等的Eureka服务器）

```
---
弹簧：
  个人资料：peer1
尤里卡：
  实例：
    主机名：peer1
  客户：
    serviceUrl：
      defaultZone：https：// peer2 / eureka /

---
弹簧：
  个人资料：peer2
尤里卡：
  实例：
    主机名：peer2
  客户：
    serviceUrl：
      defaultZone：https：// peer1 / eureka /
```

在前面的示例中，我们有一个YAML文件，该文件可以通过在不同的Spring配置文件中运行，从而在两个主机（`peer1`和`peer2`）上运行同一服务器。您可以使用此配置通过操作`/etc/hosts`解析主机名来测试单个主机上的对等感知（在生产环境中这样做没有太大价值）。实际上，`eureka.instance.hostname`如果您在知道其主机名的计算机上运行，则不需要（默认情况下，通过使用进行查找`java.net.InetAddress`）。

您可以将多个对等方添加到系统，并且只要它们都通过至少一个边缘相互连接，它们就可以在彼此之间同步注册。如果对等方在物理上是分开的（在一个数据中心内部或在多个数据中心之间），则该系统原则上可以承受“裂脑”型故障。您可以将多个对等方添加到系统中，并且只要它们都直接相互连接，它们就可以在彼此之间同步注册。

application.yml（三个对等的Eureka服务器）

```
尤里卡：
  客户：
    serviceUrl：
      defaultZone：https：// peer1 / eureka /，http：// peer2 / eureka /，http：// peer3 / eureka /

---
弹簧：
  个人资料：peer1
尤里卡：
  实例：
    主机名：peer1

---
弹簧：
  个人资料：peer2
尤里卡：
  实例：
    主机名：peer2

---
弹簧：
  个人资料：peer3
尤里卡：
  实例：
    主机名：peer3
```

### [2.6。何时首选IP地址](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-eureka-server-prefer-ip-address)

在某些情况下，Eureka最好公布服务的IP地址而不是主机名。设置`eureka.instance.preferIpAddress`为`true`，当应用程序向eureka注册时，它将使用其IP地址而不是其主机名。

|      | 如果Java无法确定主机名，则IP地址将发送到Eureka。设置主机名的唯一明确方法是通过设置`eureka.instance.hostname`属性。您可以在运行时使用环境变量（例如，）设置主机名`eureka.instance.hostname=${HOST_NAME}`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [2.7。保护Eureka服务器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#securing-the-eureka-server)

您只需通过将Spring Security添加到服务器的类路径中，就可以保护Eureka服务器`spring-boot-starter-security`。默认情况下，当Spring Security位于类路径上时，它将要求在每次向应用程序发送请求时都发送有效的CSRF令牌。Eureka客户通常不会拥有有效的跨站点请求伪造（CSRF）令牌，您需要为`/eureka/**`端点禁用此要求。例如：

```java
@EnableWebSecurity
class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().ignoringAntMatchers("/eureka/**");
        super.configure(http);
    }
}
```

有关CSRF的更多信息，请参见[Spring Security文档](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#csrf)。

可以在Spring Cloud Samples存储[库中](https://github.com/spring-cloud-samples/eureka/tree/Eureka-With-Security)找到演示版的Eureka Server 。

### [2.8。使用Eureka Server和客户端启动器禁用功能区](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#disabling-ribbon-with-eureka-server-and-client-starters)

`spring-cloud-starter-netflix-eureka-server`并`spring-cloud-starter-netflix-eureka-client`附带一个`spring-cloud-starter-netflix-ribbon`。由于Ribbon负载均衡器现在处于维护模式，我们建议改用Eureka启动程序中也包括的Spring Cloud LoadBalancer。

为此，您可以将`spring.cloud.loadbalancer.ribbon.enabled`property 的值设置为`false`。

然后，您还可以在构建文件中从Eureka启动程序中排除功能区相关的依赖项，如下所示：

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.cloud</groupId>
                    <artifactId>spring-cloud-starter-ribbon</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>com.netflix.ribbon</groupId>
                    <artifactId>ribbon-eureka</artifactId>
                </exclusion>
            </exclusions>
</dependency>
```

### [2.9。JDK 11支持](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#jdk-11-support)

在JDK 11中删除了Eureka服务器依赖的JAXB模块。如果打算在运行Eureka服务器时使用JDK 11，则必须在POM或Gradle文件中包括这些依赖项。

```xml
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
</dependency>
```

## 3.Circuit Breaker: Spring Cloud Circuit Breaker With Hystrix

断路器：Spring Cloud Hystrix断路器

### [3.1。禁用Spring Cloud Hystrix断路器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#disabling-spring-cloud-circuit-breaker-hystrix)

您可以通过设置`spring.cloud.circuitbreaker.hystrix.enabled` 为来禁用自动配置`false`。

### [3.2。配置Hystrix断路器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#configuring-hystrix-circuit-breakers)

#### [3.2.1。默认配置](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#default-configuration)

要为所有断路器提供默认配置，请创建一个`Customize`通过`HystrixCircuitBreakerFactory`或传递的bean `ReactiveHystrixCircuitBreakerFactory`。该`configureDefault`方法可用于提供默认配置。

```java
@Bean
public Customizer<HystrixCircuitBreakerFactory> defaultConfig() {
    return factory -> factory.configureDefault(id -> HystrixCommand.Setter
            .withGroupKey(HystrixCommandGroupKey.Factory.asKey(id))
            .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
            .withExecutionTimeoutInMilliseconds(4000)));
}
```

##### [Reactive例子](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#reactive-example)

```java
@Bean
public Customizer<ReactiveHystrixCircuitBreakerFactory> defaultConfig() {
    return factory -> factory.configureDefault(id -> HystrixObservableCommand.Setter
            .withGroupKey(HystrixCommandGroupKey.Factory.asKey(id))
            .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
                    .withExecutionTimeoutInMilliseconds(4000)));
}
```

#### [3.2.2。特定断路器配置](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#specific-circuit-breaker-configuration)

与提供默认配置类似，您可以创建一个`Customize`传递给它的bean `HystrixCircuitBreakerFactory`

```java
@Bean
public Customizer<HystrixCircuitBreakerFactory> customizer() {
    return factory -> factory.configure(builder -> builder.commandProperties(
                    HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(2000)), "foo", "bar");
}
```

##### [Reactive例子](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#reactive-example-2)

```java
@Bean
public Customizer<ReactiveHystrixCircuitBreakerFactory> customizer() {
    return factory -> factory.configure(builder -> builder.commandProperties(
                    HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(2000)), "foo", "bar");
}
```

## 4.Circuit Breaker: Hystrix Clients

Netflix创建了一个名为[Hystrix](https://github.com/Netflix/Hystrix)的库，该库实现了[断路器模式](https://martinfowler.com/bliki/CircuitBreaker.html)。在微服务架构中，通常有多个服务调用层，如以下示例所示：

![Hystrix](https://raw.githubusercontent.com/spring-cloud/spring-cloud-netflix/master/docs/src/main/asciidoc/images/Hystrix.png)

图1.微服务图

较低级别的服务中的服务故障可能会导致级联故障，直至用户。当在由（默认值：10秒）定义的滚动窗口中，对特定服务的调用超过`circuitBreaker.requestVolumeThreshold`（默认值：20个请求）并且失败百分比大于`circuitBreaker.errorThresholdPercentage`（默认值：> 50％）时`metrics.rollingStats.timeInMilliseconds`，电路将断开并且不会进行调用。在错误和断路的情况下，开发人员可以提供备用功能。

![HystrixFallback](https://raw.githubusercontent.com/spring-cloud/spring-cloud-netflix/master/docs/src/main/asciidoc/images/HystrixFallback.png)

图2. Hystrix后备可防止级联故障

开路可停止级联故障，并让不堪重负的服务时间得以恢复。回退可以是另一个受Hystrix保护的调用，静态数据或合理的空值。可以将回退链接在一起，以便第一个回退进行其他业务调用，然后回退到静态数据。

### [4.1。如何包括Hystrix](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#how-to-include-hystrix)

要将Hystrix包含在您的项目中，请使用组ID为`org.springframework.cloud` 和工件ID为的启动器`spring-cloud-starter-netflix-hystrix`。有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参见[Spring Cloud Project页面](https://projects.spring.io/spring-cloud/)。

以下示例显示了带有Hystrix断路器的最小型Eureka服务器：

```
@SpringBootApplication
@EnableCircuitBreaker
公共类申请{

    公共静态void main（String [] args）{
        新的SpringApplicationBuilder（Application.class）.web（true）.run（args）;
    }

}

@零件
公共类StoreIntegration {

    @HystrixCommand（fallbackMethod =“ defaultStores”）
    public Object getStores（Map <String，Object>参数）{
        //做可能失败的事情
    }

    public Object defaultStores（Map <String，Object>参数）{
        返回/ *一些有用的* /;
    }
}
```

这`@HystrixCommand`是由一个名为[“ javanica”](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-javanica)的Netflix contrib库提供的。Spring Cloud会自动将带有该批注的Spring bean包装在连接到Hystrix断路器的代理中。断路器计算何时断开和闭合电路，以及在发生故障时应采取的措施。

配置，`@HystrixCommand`您可以将`commandProperties` 属性与`@HystrixProperty`注释列表一起使用。有关 更多详细信息，请参见 [此处](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-javanica#configuration)。有关 可用属性的详细信息，请参见[Hystrix Wiki](https://github.com/Netflix/Hystrix/wiki/Configuration)。

### [4.2。传播安全上下文或使用Spring Scope](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#netflix-hystrix-starter)

如果您希望某些线程本地上下文传播到中`@HystrixCommand`，则默认声明不起作用，因为它在线程池中执行命令（如果超时）。您可以通过配置或要求注释使用不同的“隔离策略”，将Hystrix切换为使用与调用方相同的线程或直接在注释中使用。下面的示例演示了如何在注释中设置线程：

```java
@HystrixCommand(fallbackMethod = "stubMyService",
    commandProperties = {
      @HystrixProperty(name="execution.isolation.strategy", value="SEMAPHORE")
    }
)
...
```

如果您使用`@SessionScope`或，则同样适用`@RequestScope`。如果遇到运行时异常，提示它找不到范围内的上下文，则需要使用同一线程。

您还可以选择将`hystrix.shareSecurityContext`属性设置为`true`。这样做会自动配置一个Hystrix并发策略插件挂钩，以将其`SecurityContext`从您的主线程转移到Hystrix命令使用的线程。Hystrix不允许注册多个Hystrix并发策略，因此可以通过将自己声明`HystrixConcurrencyStrategy`为Spring Bean 来使用扩展机制。Spring Cloud在Spring上下文中寻找您的实现，并将其包装在自己的插件中。

### [4.3。健康指标](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#health-indicator)

连接的断路器的状态也`/health`显示在调用应用程序的端点中，如以下示例所示：

```json
{
    "hystrix": {
        "openCircuitBreakers": [
            "StoreIntegration::getStoresByLocationLink"
        ],
        "status": "CIRCUIT_OPEN"
    },
    "status": "UP"
}
```

### [4.4。Hystrix指标流](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#hystrix-metrics-stream)

为了使猬度量流，包括在依赖关系`spring-boot-starter-actuator`和集 `management.endpoints.web.exposure.include: hystrix.stream`。这样做将`/actuator/hystrix.stream`as作为管理端点公开，如以下示例所示：

```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
```

## 5.Circuit Breaker: Hystrix Dashboard

Hystrix的主要优点之一是它收集的有关每个HystrixCommand的一组度量。Hystrix仪表盘以有效的方式显示每个断路器的运行状况。

![Hystrix](https://raw.githubusercontent.com/spring-cloud/spring-cloud-netflix/master/docs/src/main/asciidoc/images/Hystrix.png)

图3. Hystrix仪表板

## 6. Hystrix Timeouts And Ribbon Clients

使用用于包装Ribbon客户端的Hystrix命令时，您要确保将Hystrix超时配置为比配置的Ribbon超时更长，包括可能进行的任何重试。例如，如果您的功能区连接超时为一秒，并且功能区客户端可能重试该请求三次，则您的Hystrix超时应该稍微超过三秒钟。

### [6.1。如何包括Hystrix仪表板](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#netflix-hystrix-dashboard-starter)

要将Hystrix仪表板包括在您的项目中，请使用组ID为`org.springframework.cloud`和工件ID为的启动器`spring-cloud-starter-netflix-hystrix-dashboard`。有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参见[Spring Cloud Project页面](https://projects.spring.io/spring-cloud/)。

要运行Hystrix仪表板，请使用注释您的Spring Boot主类`@EnableHystrixDashboard`。然后访问`/hystrix`并将仪表板指向`/hystrix.stream`Hystrix客户端应用程序中单个实例的端点。

|      | 连接到`/hystrix.stream`使用HTTPS 的端点时，JVM必须信任服务器使用的证书。如果证书不受信任，则必须将证书导入JVM，以使Hystrix仪表板成功连接到流端点。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [6.2。涡轮](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#turbine)

就系统的整体运行状况而言，查看单个实例的Hystrix数据不是很有用。[Turbine](https://github.com/Netflix/Turbine)是一种将所有相关`/hystrix.stream`端点聚合到一起以`/turbine.stream`用于Hystrix仪表板的应用程序。个别实例通过Eureka定位。运行Turbine需要使用注释对您的主类进行`@EnableTurbine`注释（例如，通过使用spring-cloud-starter-netflix-turbine设置类路径）。[Turbine 1 Wiki中](https://github.com/Netflix/Turbine/wiki/Configuration-(1.x))记录的所有配置属性均适用。唯一的区别是`turbine.instanceUrlSuffix`不需要端口，因为除非，否则端口会自动处理`turbine.instanceInsertPort=false`。

|      | 默认情况下，Turbine `/hystrix.stream`通过在Eureka中查找其实例`hostName`和`port`条目，然后追加`/hystrix.stream`到它，来在已注册实例上查找端点。如果实例的元数据包含`management.port`，则使用它代替端点的`port`值`/hystrix.stream`。默认情况下，调用的元数据条目`management.port`等于`management.port`配置属性。可以使用以下配置覆盖它： |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```
尤里卡：
  实例：
    元数据映射：
      management.port：$ {management.port：8081}
```

该`turbine.appConfig`配置关键是Eureka serviceIds的列表，涡轮用来查找实例。然后在Hystrix仪表板中使用该涡轮机流，其URL类似于以下内容：

```
my.turbine.server:8080/turbine.stream?cluster=CLUSTERNAME
```

如果名称为，则可以省略cluster参数`default`。该`cluster`参数必须项匹配`turbine.aggregator.clusterConfig`。从Eureka返回的值是大写的。因此，如果有一个`customers`在Eureka进行了注册的应用程序，则以下示例可用：

```
涡轮：
  聚合器：
    clusterConfig：客户
  appConfig：客户
```

如果您需要定制Turbine应该使用哪些集群名称（因为您不想在`turbine.aggregator.clusterConfig`配置中存储集群名称 ），请提供type的bean `TurbineClustersProvider`。

所述`clusterName`可以通过SPEL表达被定制`turbine.clusterNameExpression`以根作为实例`InstanceInfo`。默认值为`appName`，这意味着Eureka `serviceId`成为群集密钥（即`InstanceInfo`for客户的`appName`of为`CUSTOMERS`）。一个不同的示例是`turbine.clusterNameExpression=aSGName`，它从AWS ASG名称获取集群名称。以下清单显示了另一个示例：

```
涡轮：
  聚合器：
    clusterConfig：SYSTEM，USER
  appConfig：客户，商店，用户界面，管理员
  clusterNameExpression：元数据['集群']
```

在前面的示例中，来自四个服务的集群名称从它们的元数据映射中拉出，并且期望具有包含`SYSTEM`和的值`USER`。

要对所有应用程序使用“默认”群集，您需要一个字符串文字表达式（如果在YAML中，也要使用单引号和双引号进行转义）：

```
涡轮：
  appConfig：客户，商店
  clusterNameExpression：“'默认'”
```

Spring Cloud提供了一个`spring-cloud-starter-netflix-turbine`具有运行Turbine服务器所需的所有依赖项。要添加Turbine，请创建一个Spring Boot应用程序并使用对其进行注释`@EnableTurbine`。

|      | 默认情况下，Spring Cloud允许Turbine使用主机和端口来允许每个主机，每个集群多个进程。如果你想内置式水轮机本地Netflix的行为*不*允许每个主机上的多个过程，每簇（关键实例ID是主机名），集`turbine.combineHostPort=false`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### [6.2.1。集群端点](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#clusters-endpoint)

在某些情况下，其他应用程序了解在Turbine中配置了哪些custers可能会很有用。为此，您可以使用`/clusters`端点，该端点将返回所有已配置集群的JSON数组。

GET /集群

```json
[
  {
    "name": "RACES",
    "link": "http://localhost:8383/turbine.stream?cluster=RACES"
  },
  {
    "name": "WEB",
    "link": "http://localhost:8383/turbine.stream?cluster=WEB"
  }
]
```

可以通过设置`turbine.endpoints.clusters.enabled`为禁用此端点`false`。

### [6.3。涡轮流](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#turbine-stream)

在某些环境中（例如在PaaS设置中），从所有分布式Hystrix命令中提取指标的经典Turbine模型不起作用。在这种情况下，您可能想让Hystrix命令将度量标准推送到Turbine。Spring Cloud通过消息传递实现了这一点。要在客户端上执行此操作，请向`spring-cloud-netflix-hystrix-stream`和`spring-cloud-starter-stream-*`选择一个依赖项。请参阅[Spring Cloud Stream文档](https://docs.spring.io/spring-cloud-stream/docs/current/reference/htmlsingle/)以获取有关代理以及如何配置客户端凭据的详细信息。对于本地代理，它应该开箱即用。

在服务器端，创建一个Spring Boot应用程序并使用对其进行注释`@EnableTurbineStream`。Turbine Stream服务器需要使用Spring Webflux，因此`spring-boot-starter-webflux`需要包含在您的项目中。`spring-boot-starter-webflux`添加`spring-cloud-starter-netflix-turbine-stream`到您的应用程序时默认包括。

然后，您可以将Hystrix仪表板指向Turbine Stream Server，而不是单个Hystrix流。如果Turbine Stream在myhost的端口8989上运行，则将其放入`myhost:8989`Hystrix仪表板的流输入字段中。电路的前缀是各自的`serviceId`，后跟点（`.`），然后是电路名称。

Spring Cloud提供了一个`spring-cloud-starter-netflix-turbine-stream`具有运行Turbine Stream服务器所需的所有依赖项。然后，您可以添加所选的Stream活页夹-例如`spring-cloud-starter-stream-rabbit`。

Turbine Stream Server也支持该`cluster`参数。与Turbine服务器不同，Turbine Stream使用eureka serviceIds作为群集名称，并且这些名称不可配置。

如果Turbine Stream服务器在端口8989上运行，并且您的环境中`my.turbine.server`有两个eureka serviceId ，则以下URL将在您的Turbine Stream服务器上可用。空的群集名称将提供Turbine Stream服务器接收的所有指标。`customers``products``default`

```
https：//my.turbine.sever：8989 / turbine.stream？cluster = customers
https：//my.turbine.sever：8989 / turbine.stream？cluster = products
https：//my.turbine.sever：8989 / turbine.stream？cluster = default
https：//my.turbine.sever：8989 / turbine.stream
```

因此，您可以将eureka serviceIds用作Turbine仪表板（或任何兼容的仪表板）的群集名称。你并不需要配置像任何性能`turbine.appConfig`，`turbine.clusterNameExpression`并`turbine.aggregator.clusterConfig`为您的涡轮机流服务器。

|      | Turbine Stream服务器使用Spring Cloud Stream从配置的输入通道收集所有指标。这意味着它不会从每个实例主动收集Hystrix指标。它仅可以提供每个实例已经收集到输入通道中的度量。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 7.Client Side Load Balancer: Ribbon

Ribbon是客户端负载平衡器，可让您对HTTP和TCP客户端的行为进行大量控制。Feign已经使用了Ribbon，因此，如果使用`@FeignClient`，则此部分也适用。

Ribbon中的中心概念是指定客户端的概念。每个负载平衡器都是组件的一部分，这些组件可以一起工作以按需联系远程服务器，并且该组件具有一个名称，您可以将其命名为应用程序开发人员（例如，通过使用`@FeignClient`批注）。根据需要，Spring Cloud `ApplicationContext`通过使用为每个命名客户端 创建一个新的集合`RibbonClientConfiguration`。这包含（除其他事项外）an `ILoadBalancer`，a `RestClient`和a `ServerListFilter`。

### [7.1。如何包括功能区](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#netflix-ribbon-starter)

要将Ribbon包含在项目中，请使用组ID为`org.springframework.cloud`和工件ID为的启动器`spring-cloud-starter-netflix-ribbon`。有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参见[Spring Cloud Project页面](https://projects.spring.io/spring-cloud/)。

### [7.2。自定义功能区客户端](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#customizing-the-ribbon-client)

您可以使用中的外部属性来配置Ribbon客户端的某些位`.ribbon.*`，这与本地使用Netflix API相似，不同之处在于可以使用Spring Boot配置文件。可以将本机选项检查为[`CommonClientConfigKey`](https://github.com/Netflix/ribbon/blob/master/ribbon-core/src/main/java/com/netflix/client/config/CommonClientConfigKey.java)（功能区核心的一部分）中的静态字段。

Spring Cloud还允许您通过使用声明其他配置（位于之上`RibbonClientConfiguration`）来完全控制客户端`@RibbonClient`，如以下示例所示：

```java
@Configuration
@RibbonClient(name = "custom", configuration = CustomConfiguration.class)
public class TestConfiguration {
}
```

在这种情况下，客户端由in中的组件`RibbonClientConfiguration`以及in中的组件组成`CustomConfiguration`（其中后者通常会覆盖前者）。

|      | 该`CustomConfiguration`班必须是`@Configuration`一流的，但照顾，这是不是在`@ComponentScan`主应用程序上下文。否则，将由所有共享`@RibbonClients`。如果您使用`@ComponentScan`（或`@SpringBootApplication`），则需要采取措施避免将其包括在内（例如，您可以将其放在单独的，不重叠的程序包中，或在中指定要扫描的程序包`@ComponentScan`）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

下表显示了Spring Cloud Netflix默认为Ribbon提供的bean：

|       豆类型        |           豆名            |             班级名称             |
| :-----------------: | :-----------------------: | :------------------------------: |
|   `IClientConfig`   |   `ribbonClientConfig`    |    `DefaultClientConfigImpl`     |
|       `IRule`       |       `ribbonRule`        |       `ZoneAvoidanceRule`        |
|       `IPing`       |       `ribbonPing`        |           `DummyPing`            |
|    `ServerList`     |    `ribbonServerList`     |  `ConfigurationBasedServerList`  |
| `ServerListFilter`  | `ribbonServerListFilter`  | `ZonePreferenceServerListFilter` |
|   `ILoadBalancer`   |   `ribbonLoadBalancer`    |     `ZoneAwareLoadBalancer`      |
| `ServerListUpdater` | `ribbonServerListUpdater` |    `PollingServerListUpdater`    |

创建这些类型之一的bean并将其放置在`@RibbonClient`配置中（例如`FooConfiguration`上述配置），可以覆盖所描述的每个bean，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
protected static class FooConfiguration {

    @Bean
    public ZonePreferenceServerListFilter serverListFilter() {
        ZonePreferenceServerListFilter filter = new ZonePreferenceServerListFilter();
        filter.setZone("myTestZone");
        return filter;
    }

    @Bean
    public IPing ribbonPing() {
        return new PingUrl();
    }

}
```

前面示例中的include语句替换`NoOpPing`为`PingUrl`并提供了一个custom `serverListFilter`。

### [7.3。为所有功能区客户端自定义默认值](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#customizing-the-default-for-all-ribbon-clients)

通过使用`@RibbonClients`注释并注册默认配置，可以为所有Ribbon客户提供默认配置，如以下示例所示：

```java
@RibbonClients(defaultConfiguration = DefaultRibbonConfig.class)
public class RibbonClientDefaultConfigurationTestsConfig {

    public static class BazServiceList extends ConfigurationBasedServerList {

        public BazServiceList(IClientConfig config) {
            super.initWithNiwsConfig(config);
        }

    }

}

@Configuration(proxyBeanMethods = false)
class DefaultRibbonConfig {

    @Bean
    public IRule ribbonRule() {
        return new BestAvailableRule();
    }

    @Bean
    public IPing ribbonPing() {
        return new PingUrl();
    }

    @Bean
    public ServerList<Server> ribbonServerList(IClientConfig config) {
        return new RibbonClientDefaultConfigurationTestsConfig.BazServiceList(config);
    }

    @Bean
    public ServerListSubsetFilter serverListFilter() {
        ServerListSubsetFilter filter = new ServerListSubsetFilter();
        return filter;
    }

}
```

### [7.4。通过设置属性来自定义功能区客户端](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#customizing-the-ribbon-client-by-setting-properties)

从1.2.0版开始，Spring Cloud Netflix现在通过将属性设置为与[Ribbon文档](https://github.com/Netflix/ribbon/wiki/Working-with-load-balancers#components-of-load-balancer)兼容来支持自定义Ribbon客户端。

这使您可以在启动时在不同环境中更改行为。

以下列表显示了受支持的属性>：

- `.ribbon.NFLoadBalancerClassName`：应实施 `ILoadBalancer`
- `.ribbon.NFLoadBalancerRuleClassName`：应实施 `IRule`
- `.ribbon.NFLoadBalancerPingClassName`：应实施 `IPing`
- `.ribbon.NIWSServerListClassName`：应实施 `ServerList`
- `.ribbon.NIWSServerListFilterClassName`：应实施 `ServerListFilter`

|      | 这些属性中定义的类优先于使用`@RibbonClient(configuration=MyRibbonConfig.class)`和由Spring Cloud Netflix提供的默认值定义的bean 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要`IRule`为名为的服务名称`users`设置，您可以设置以下属性：

application.yml

```
用户：
  带：
    NIWSServerListClassName：com.netflix.loadbalancer.ConfigurationBasedServerList
    NFLoadBalancerRuleClassName：com.netflix.loadbalancer.WeightedResponseTimeRule
```

有关[功能区](https://github.com/Netflix/ribbon/wiki/Working-with-load-balancers)提供的实现，请参见[功能区文档](https://github.com/Netflix/ribbon/wiki/Working-with-load-balancers)。

### [7.5。将功能区与Eureka一起使用](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#using-ribbon-with-eureka)

当Eureka与Ribbon结合使用时（也就是说，两者都在类路径上），`ribbonServerList`会被扩展名覆盖，该扩展名`DiscoveryEnabledNIWSServerList`会填充Eureka中的服务器列表。它还将替换为`IPing`接口`NIWSDiscoveryPing`，该接口委托Eureka确定服务器是否启动。该`ServerList`由默认安装的是`DomainExtractingServerList`。其目的是不使用AWS AMI元数据（这就是Netflix所依赖的）使元数据可用于负载均衡器。默认情况下，服务器列表是用实例区域元数据中提供的“区域”信息构造的（因此，在远程客户端上是set `eureka.instance.metadataMap.zone`）。如果缺少，并且`approximateZoneFromHostname`如果设置了flag，它可以使用服务器主机名中的域名作为区域的代理。一旦区域信息可用，就可以在中使用它`ServerListFilter`。默认情况下，它用于在与客户端相同的区域中定位服务器，因为默认值为a `ZonePreferenceServerListFilter`。默认情况下，以与远程实例相同的方式（即，通过`eureka.instance.metadataMap.zone`）确定客户端的区域。

|      | 设置客户端区域的传统“ archaius”方法是通过名为“ @zone”的配置属性。如果可用，Spring Cloud会优先使用所有其他设置（请注意，密钥必须在YAML配置中用引号引起来）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果没有其他区域数据源，则根据客户端配置（而不是实例配置）进行猜测。我们采用`eureka.client.availabilityZones`，这是从区域名称到区域列表的地图，并为实例自己的区域拉出第一个区域（即`eureka.client.region`，默认为“ us-east-1”，以与本机Netflix兼容） 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [7.6。示例：如何在没有尤里卡的情况下使用色带](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-ribbon-without-eureka)

Eureka是一种抽象发现远程服务器的便捷方法，因此您不必在客户端中对它们的URL进行硬编码。但是，如果您不想使用Eureka，Ribbon和Feign也可以使用。假设您`@RibbonClient`为“ stores” 声明了a ，并且Eureka未被使用（甚至不在类路径上）。功能区客户端默认为已配置的服务器列表。您可以提供以下配置：

application.yml

```
店铺：
  带：
    listOfServers：example.com，google.com
```

### [7.7。示例：禁用功能区中的尤里卡使用](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#example-disable-eureka-use-in-ribbon)

将该`ribbon.eureka.enabled`属性设置为`false`显式禁用在功能区中使用Eureka，如以下示例所示：

application.yml

```
带：
  尤里卡：
   启用：false
```

### [7.8。直接使用功能区API](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#using-the-ribbon-api-directly)

您也可以`LoadBalancerClient`直接使用direct，如以下示例所示：

```java
public class MyClass {
    @Autowired
    private LoadBalancerClient loadBalancer;

    public void doStuff() {
        ServiceInstance instance = loadBalancer.choose("stores");
        URI storesUri = URI.create(String.format("https://%s:%s", instance.getHost(), instance.getPort()));
        // ... do something with the URI
    }
}
```

### [7.9。缓存功能区配置](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#ribbon-child-context-eager-load)

每个命名为Ribbon的客户端都有一个Spring Cloud维护的相应子应用程序上下文。该应用程序上下文在对命名客户端的第一个请求上延迟加载。通过指定功能区客户端的名称，可以更改此延迟加载行为，以代替在启动时急于加载这些子应用程序上下文，如以下示例所示：

application.yml

```
带：
  急切负载：
    已启用：true
    客户端：client1，client2，client3
```

### [7.10。如何配置Hystrix线程池](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#how-to-configure-hystrix-thread-pools)

如果更改`zuul.ribbonIsolationStrategy`为`THREAD`，则将Hystrix的线程隔离策略用于所有路由。在这种情况下，`HystrixThreadPoolKey`设置`RibbonCommand`为默认值。这意味着所有路由的HystrixCommands在同一个Hystrix线程池中执行。可以使用以下配置更改此行为：

application.yml

```
祖尔：
  threadPool：
    useSeparateThreadPools：true
```

前面的示例导致在Hystrix线程池中为每个路由执行HystrixCommands。

在这种情况下，默认值`HystrixThreadPoolKey`与每个路由的服务ID相同。要向添加前缀`HystrixThreadPoolKey`，请设置`zuul.threadPool.threadPoolKeyPrefix`为要添加的值，如以下示例所示：

application.yml

```
祖尔：
  threadPool：
    useSeparateThreadPools：true
    threadPoolKeyPrefix：zuulgw
```

### [7.11。如何提供功能区的密钥`IRule`](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#how-to-provdie-a-key-to-ribbon)

如果您需要提供自己的`IRule`实现来处理特殊的路由要求（例如“ canary”测试），请将一些信息传递给的`choose`方法`IRule`。

com.netflix.loadbalancer.IRule.java

```
公共接口IRule {
    公共服务器选择（对象键）；
         ：
```

您可以提供一些信息，供您的`IRule`实现用于选择目标服务器，如以下示例所示：

```
RequestContext.getCurrentContext（）
              .set（FilterConstants.LOAD_BALANCER_KEY，“ canary-test”）;
```

如果您`RequestContext`使用键将任何对象放入`FilterConstants.LOAD_BALANCER_KEY`，则会将其传递给实现的`choose`方法`IRule`。前面示例中显示的代码必须在执行之前`RibbonRoutingFilter`执行。Zuul的前置过滤器是最好的选择。您可以通过`RequestContext`in前置过滤器访问HTTP标头和查询参数，因此可以使用它来确定`LOAD_BALANCER_KEY`传递到Ribbon的参数。如果不使用`LOAD_BALANCER_KEY`in 放置任何值，则将`RequestContext`null传递为`choose`方法的参数。

## [8.外部配置：Archaius](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#external-configuration-archaius)

[Archaius](https://github.com/Netflix/archaius)是Netflix客户端配置库。它是所有Netflix OSS组件用于配置的库。Archaius是[Apache Commons Configuration](https://commons.apache.org/proper/commons-configuration)项目的扩展。它允许通过轮询源以进行更改或通过将源将更改推送到客户端来更新配置。Archaius使用Dynamic <Type> Property类作为属性的句柄，如以下示例所示：

Archaius示例

```java
class ArchaiusTest {
    DynamicStringProperty myprop = DynamicPropertyFactory
            .getInstance()
            .getStringProperty("my.prop");

    void doSomething() {
        OtherClass.someMethod(myprop.get());
    }
}
```

Archaius具有自己的一组配置文件和加载优先级。Spring应用程序通常不应该直接使用Archaius，但是仍然需要本地配置Netflix工具。Spring Cloud具有一个Spring Environment Bridge，以便Archaius可以从Spring Environment中读取属性。该桥允许Spring Boot项目使用常规配置工具链，同时允许它们按记录的方式配置Netflix工具（大部分情况下）。

## [9.路由器和过滤器：Zuul](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#router-and-filter-zuul)

路由是微服务架构不可或缺的一部分。例如，`/`可能被映射到您的Web应用程序，`/api/users`被映射到用户服务以及`/api/shop`被映射到商店服务。 [Zuul](https://github.com/Netflix/zuul)是Netflix提供的基于JVM的路由器和服务器端负载平衡器。

[Netflix将Zuul](https://www.slideshare.net/MikeyCohen1/edge-architecture-ieee-international-conference-on-cloud-engineering-32240146/27)用于以下[用途](https://www.slideshare.net/MikeyCohen1/edge-architecture-ieee-international-conference-on-cloud-engineering-32240146/27)：

- 认证方式
- 见解
- 压力测试
- 金丝雀测试
- 动态路由
- 服务迁移
- 减载
- 安全
- 静态响应处理
- 主动/主动流量管理

Zuul的规则引擎使规则和过滤器基本上可以用任何JVM语言编写，并具有对Java和Groovy的内置支持。

|      | 配置属性`zuul.max.host.connections`已被替换为两个新属性，`zuul.host.maxTotalConnections`和`zuul.host.maxPerRouteConnections`，分别默认为200和20。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | `ExecutionIsolationStrategy`所有路由 的默认Hystrix隔离模式（）为`SEMAPHORE`。 `zuul.ribbonIsolationStrategy`可以更改`THREAD`为首选该隔离模式。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [9.1。如何包含Zuul](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#netflix-zuul-starter)

要将Zuul包含在您的项目中，请使用组ID为`org.springframework.cloud`和工件ID为的启动器`spring-cloud-starter-netflix-zuul`。有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参见[Spring Cloud Project页面](https://projects.spring.io/spring-cloud/)。

### [9.2。嵌入式Zuul反向代理](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#netflix-zuul-reverse-proxy)

Spring Cloud创建了一个嵌入式Zuul代理，以简化UI应用程序要对一个或多个后端服务进行代理调用的常见用例的开发。此功能对于用户界面代理所需的后端服务很有用，从而避免了为所有后端独立管理CORS和身份验证问题的需求。

要启用它，请使用注释Spring Boot主类`@EnableZuulProxy`。这样做会导致将本地呼叫转发到适当的服务。按照惯例，ID为的服务`users`从位于的代理`/users`（去掉前缀）接收请求。代理使用功能区来定位要通过发现转发到的实例。所有请求均在[hystrix命令](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#hystrix-fallbacks-for-routes)中执行，因此失败会显示在Hystrix指标中。一旦电路断开，代理就不会尝试与服务联系。

|      | Zuul启动程序不包括发现客户端，因此，对于基于服务ID的路由，您还需要在类路径上提供其中之一（Eureka是一种选择）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要跳过自动添加服务的步骤，请设置`zuul.ignored-services`为服务ID模式列表。如果服务与被忽略但已包含在显式配置的路由映射中的模式匹配，则将其忽略，如以下示例所示：

application.yml

```yaml
 zuul:
  ignoredServices: '*'
  routes:
    users: /myusers/**
```

在上面的例子中，所有的服务都将被忽略，**除了**为`users`。

要增加或更改代理路由，可以添加外部配置，如下所示：

application.yml

```yaml
 zuul:
  routes:
    users: /myusers/**
```

前面的示例意味着HTTP调用要`/myusers`转发到`users`服务（例如`/myusers/101`转发到`/101`）。

要对路由进行更细粒度的控制，可以分别指定路径和serviceId，如下所示：

application.yml

```yaml
 zuul:
  routes:
    users:
      path: /myusers/**
      serviceId: users_service
```

前面的示例意味着HTTP调用将`/myusers`转发到该`users_service`服务。路由必须具有`path`可以指定为蚂蚁风格模式的，因此`/myusers/*`只能匹配一个级别，但可以`/myusers/**`分层匹配。

后端的位置可以指定为`serviceId`（用于发现服务）或`url`（物理位置），如以下示例所示：

application.yml

```yaml
 zuul:
  routes:
    users:
      path: /myusers/**
      url: https://example.com/users_service
```

这些简单的url-routes不会以的形式执行`HystrixCommand`，也不会使用Ribbon负载均衡多个URL。为了实现这些目标，您可以指定一个`serviceId`带有静态服务器列表的，如下所示：

application.yml

```yaml
zuul:
  routes:
    echo:
      path: /myusers/**
      serviceId: myusers-service
      stripPrefix: true

hystrix:
  command:
    myusers-service:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: ...

myusers-service:
  ribbon:
    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
    listOfServers: https://example1.com,http://example2.com
    ConnectTimeout: 1000
    ReadTimeout: 3000
    MaxTotalHttpConnections: 500
    MaxConnectionsPerHost: 100
```

另一种方法是指定服务路由并配置Ribbon客户端`serviceId`（这样做需要在Ribbon中禁用Eureka支持- [有关更多信息](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-ribbon-without-eureka)，请参见[上文](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-ribbon-without-eureka)），如以下示例所示：

application.yml

```yaml
zuul:
  routes:
    users:
      path: /myusers/**
      serviceId: users

ribbon:
  eureka:
    enabled: false

users:
  ribbon:
    listOfServers: example.com,google.com
```

您可以使用来提供`serviceId`和之间的约定`regexmapper`。它使用正则表达式命名组从中提取变量`serviceId`并将其注入到路由模式中，如以下示例所示：

ApplicationConfiguration.java

```java
@Bean
public PatternServiceRouteMapper serviceRouteMapper() {
    return new PatternServiceRouteMapper(
        "(?<name>^.+)-(?<version>v.+$)",
        "${version}/${name}");
}
```

上面的示例指的是`serviceId`的`myusers-v1`被映射到路线`/v1/myusers/**`。可以接受任何正则表达式，但所有命名组都必须同时存在于`servicePattern`和中`routePattern`。如果`servicePattern`与不匹配`serviceId`，则使用默认行为。在前面的示例中，`serviceId`of `myusers`映射到“ / myusers / **”路由（未检测到版本）。默认情况下，此功能是禁用的，仅适用于发现的服务。

要为所有映射添加前缀，请设置`zuul.prefix`一个值，例如`/api`。默认情况下，在转发请求之前，将从请求中剥离代理前缀（您可以使用来关闭此行为`zuul.stripPrefix=false`）。您还可以关闭从单个路由中剥离特定于服务的前缀，如以下示例所示：

application.yml

```yaml
 zuul:
  routes:
    users:
      path: /myusers/**
      stripPrefix: false
```

|      | `zuul.stripPrefix`仅适用于中设置的前缀`zuul.prefix`。它对给定路由中定义的前缀没有任何影响`path`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

在前面的例子中，请求到`/myusers/101`转发到`/myusers/101`在`users`服务。

这些`zuul.routes`条目实际上绑定到类型为的对象`ZuulProperties`。如果查看该对象的属性，则可以看到它也有一个`retryable`标志。设置该标志以`true`使功能区客户端自动重试失败的请求。您还可以将该标志设置为何`true`时需要修改使用功能区客户端配置的重试操作的参数。

默认情况下，`X-Forwarded-Host`标头被添加到转发的请求中。要关闭它，请设置`zuul.addProxyHeaders = false`。默认情况下，前缀路径被剥离，并且到后端的请求选择一个`X-Forwarded-Prefix`标头（`/myusers`在前面显示的示例中）。

如果设置默认路由（`/`），则具有的应用程序`@EnableZuulProxy`可以充当独立服务器。例如，`zuul.route.home: /`将所有流量（“ / **”）路由到“ home”服务。

如果需要更细粒度的忽略，则可以指定要忽略的特定模式。这些模式在路线定位过程开始时进行评估，这意味着模式中应包含前缀以保证匹配。被忽略的模式跨越所有服务，并取代任何其他路由规范。以下示例显示了如何创建忽略的模式：

application.yml

```yaml
 zuul:
  ignoredPatterns: /**/admin/**
  routes:
    users: /myusers/**
```

前面的示例装置，其所有的呼叫（例如`/myusers/101`）被转发到`/101`对`users`服务。但是，包括在内的呼叫`/admin/`无法解决。

|      | 如果您需要保留路由的顺序，则需要使用YAML文件，因为使用属性文件时顺序会丢失。以下示例显示了这样的YAML文件： |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

application.yml

```yaml
 zuul:
  routes:
    users:
      path: /myusers/**
    legacy:
      path: /**
```

如果要使用属性文件，则该`legacy`路径可能会终止于该`users` 路径的前面，从而导致该`users`路径不可访问。

### [9.3。Zuul Http客户端](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-http-client)

Zuul使用的默认HTTP客户端现在由Apache HTTP客户端而不是已弃用的Ribbon支持`RestClient`。要使用`RestClient`或，分别`okhttp3.OkHttpClient`设置`ribbon.restclient.enabled=true`或`ribbon.okhttp.enabled=true`。如果要定制Apache HTTP客户端或OK HTTP客户端，请提供类型为`CloseableHttpClient`或的Bean `OkHttpClient`。

### [9.4。Cookie和敏感标题](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#cookies-and-sensitive-headers)

您可以在同一系统中的服务之间共享标头，但您可能不希望敏感标头泄漏到下游到外部服务器中。您可以在路由配置中指定忽略的标头列表。Cookies发挥着特殊的作用，因为它们在浏览器中具有定义明确的语义，并且始终将它们视为敏感内容。如果代理的使用者是浏览器，那么下游服务的cookie也会给用户带来麻烦，因为它们都混杂在一起（所有下游服务看起来都来自同一位置）。

如果您对服务的设计很谨慎（例如，如果只有一个下游服务设置cookie），则可以让它们从后端一直流到调用者。另外，如果您的代理设置cookie，并且您的所有后端服务都在同一系统中，则自然可以简单地共享它们（例如，使用Spring Session将它们链接到某些共享状态）。除此之外，由下游服务设置的任何cookie可能对调用者都没有用，因此建议您（至少）将`Set-Cookie`其`Cookie`放入不属于域的路由的敏感标头中。即使对于属于您网域的路由，在让Cookie在它们和代理之间流动之前，也应仔细考虑其含义。

可以将敏感头配置为每个路由的逗号分隔列表，如以下示例所示：

application.yml

```yaml
 zuul:
  routes:
    users:
      path: /myusers/**
      sensitiveHeaders: Cookie,Set-Cookie,Authorization
      url: https://downstream
```

|      | 这是的默认值`sensitiveHeaders`，因此除非您希望它与众不同，否则无需进行设置。这是Spring Cloud Netflix 1.1中的新功能（在1.0中，用户无法控制标题，并且所有cookie都双向流动）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

该`sensitiveHeaders`是一个黑名单，默认是不为空。因此，要使Zuul发送所有标头（`ignored`那些标头除外），必须将其显式设置为空列表。如果要将Cookie或授权标头传递到后端，则必须这样做。以下示例显示如何使用`sensitiveHeaders`：

application.yml

```yaml
 zuul:
  routes:
    users:
      path: /myusers/**
      sensitiveHeaders:
      url: https://downstream
```

您还可以通过设置设置敏感标题`zuul.sensitiveHeaders`。如果`sensitiveHeaders`在路径上设置了，则它将覆盖全局`sensitiveHeaders`设置。

### [9.5。忽略标题](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#ignored-headers)

除了对路由敏感的标头之外，您还可以设置一个称为`zuul.ignoredHeaders`值的全局值（请求和响应），在与下游服务进行交互时应将其丢弃。默认情况下，如果Spring Security不在类路径中，则它们为空。否则，它们将被初始化为Spring Security指定的一组众所周知的“安全”标头（例如，涉及缓存）。在这种情况下的假设是，下游服务也可以添加这些标头，但是我们需要来自代理的值。要在Spring Security位于类路径上时不丢弃这些众所周知的安全标头，可以设置`zuul.ignoreSecurityHeaders`为`false`。如果您在Spring Security中禁用了HTTP Security响应标头并需要下游服务提供的值，则这样做很有用。

### [9.6。管理端点](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#management-endpoints)

默认情况下，如果`@EnableZuulProxy`与Spring Boot Actuator一起使用，则启用两个附加端点：

- 路线
- 筛选器

#### [9.6.1。路线终点](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#routes-endpoint)

到路由端点的GET `/routes`返回映射的路由列表：

GET /路线

```json
{
  /stores/**: "http://localhost:8081"
}
```

可以通过向中添加`?format=details`查询字符串来请求其他路线详细信息`/routes`。这样做会产生以下输出：

GET /路线/细节

```json
{
  "/stores/**": {
    "id": "stores",
    "fullPath": "/stores/**",
    "location": "http://localhost:8081",
    "path": "/**",
    "prefix": "/stores",
    "retryable": false,
    "customSensitiveHeaders": false,
    "prefixStripped": true
  }
}
```

一`POST`到`/routes`部队现有路由的更新（例如，当在服务目录进行了变更）。您可以通过设置`endpoints.routes.enabled`为禁用此端点`false`。

|      | 路由应该自动响应服务目录中的更改，但是`POST`to `/routes`是一种使更改立即发生的方法。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### [9.6.2。过滤端点](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#filters-endpoint)

`GET`到过滤器端点的A `/filters`按类型返回Zuul过滤器的映射。对于地图中的每种过滤器类型，您将获得该类型的所有过滤器的列表以及它们的详细信息。

### [9.7。勒索模式和本地前锋](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#strangulation-patterns-and-local-forwards)

迁移现有应用程序或API时，常见的模式是“扼杀”旧的端点，并用不同的实现方式慢慢替换它们。Zuul代理是一个有用的工具，因为您可以使用它来处理来自旧端点客户端的所有流量，但可以将一些请求重定向到新请求。

以下示例显示了“扼杀”方案的配置详细信息：

application.yml

```yaml
 zuul:
  routes:
    first:
      path: /first/**
      url: https://first.example.com
    second:
      path: /second/**
      url: forward:/second
    third:
      path: /third/**
      url: forward:/3rd
    legacy:
      path: /**
      url: https://legacy.example.com
```

在前面的示例中，我们扼杀了“旧版”应用程序，该应用程序映射到与其他模式之一不匹配的所有请求。输入的路径`/first/**`已使用外部URL提取到新服务中。`/second/**`转发路径，以便可以在本地处理它们（例如，使用常规Spring `@RequestMapping`）。中的路径`/third/**`也被转发，但是具有不同的前缀（`/third/foo`转发到`/3rd/foo`）。

|      | 被忽略的模式不会被完全忽略，它们不会由代理处理（因此它们也可以在本地有效转发）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [9.8。通过Zuul上传文件](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#uploading-files-through-zuul)

如果使用`@EnableZuulProxy`，则可以使用代理路径上载文件，只要文件很小，它就可以正常工作。对于大文件`DispatcherServlet`，“ / zuul / *”中有一个替代路径绕过Spring （以避免进行多部分处理）。换句话说，如果您有`zuul.routes.customers=/customers/**`，则可以将`POST`大文件添加到`/zuul/customers/*`。servlet路径通过外部化`zuul.servletPath`。如果代理路由带您通过功能区负载平衡器，则极大的文件也需要提高超时设置，如以下示例所示：

application.yml

```yaml
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 60000
ribbon:
  ConnectTimeout: 3000
  ReadTimeout: 60000
```

请注意，要使流技术处理大文件，您需要在请求中使用分块编码（某些浏览器默认不这样做），如以下示例所示：

```bash
$ curl -v -H "Transfer-Encoding: chunked" \
    -F "file=@mylarge.iso" localhost:9999/zuul/simple/file
```

### [9.9。查询字符串编码](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#query-string-encoding)

在处理传入请求时，查询参数将被解码，以便可以在Zuul过滤器中进行可能的修改。然后将它们重新编码，在路由过滤器中重建后端请求。如果（例如）使用Javascript `encodeURIComponent()`方法对结果进行编码，则结果可能与原始输入不同。尽管这在大多数情况下不会引起问题，但某些Web服务器可能对复杂查询字符串的编码很挑剔。

要强制对查询字符串进行原始编码，可以向传递一个特殊标志，`ZuulProperties`以便按此方法使用查询字符串`HttpServletRequest::getQueryString`，如以下示例所示：

application.yml

```yaml
 zuul:
  forceOriginalQueryStringEncoding: true
```

|      | 此特殊标志仅适用于`SimpleHostRoutingFilter`。此外，您松散了使用轻松覆盖查询参数的功能`RequestContext.getCurrentContext().setRequestQueryParams(someOverriddenParameters)`，因为现在直接在原始上获取查询字符串`HttpServletRequest`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [9.10。请求URI编码](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#request-uri-encoding)

在处理传入请求时，在将请求URI与路由匹配之前，先对其进行解码。然后在路由过滤器中重建后端请求时，将对请求URI进行重新编码。如果您的URI包含编码的“ /”字符，则可能导致某些意外行为。

要使用原始请求URI，可以向'ZuulProperties'传递一个特殊标志，以便该URI与该`HttpServletRequest::getRequestURI`方法一样被使用，如以下示例所示：

application.yml

```yaml
 zuul:
  decodeUrl: false
```

|      | 如果使用`requestURI`RequestContext属性覆盖请求URI，并且此标志设置为false，则不会对在请求上下文中设置的URL进行编码。确保URL已被编码是您的责任。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [9.11。纯色嵌入式Zuul](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#plain-embedded-zuul)

如果您使用`@EnableZuulServer`（而不是`@EnableZuulProxy`），还可以运行Zuul服务器而不进行代理，也可以选择性地打开部分代理平台。您添加到类型应用程序中的所有bean都会`ZuulFilter`自动安装（与一起使用`@EnableZuulProxy`），但是不会自动添加任何代理过滤器。

在这种情况下，仍然可以通过配置“ zuul.routes。*”来指定进入Zuul服务器的路由，但是没有服务发现也没有代理。因此，“ serviceId”和“ url”设置将被忽略。以下示例将“ / api / **”中的所有路径映射到Zuul过滤器链：

application.yml

```yaml
 zuul:
  routes:
    api: /api/**
```

### [9.12。禁用Zuul过滤器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#disable-zuul-filters)

Zuul for Spring Cloud附带了许多`ZuulFilter`在代理和服务器模式下默认启用的bean。有关可以启用的过滤器列表，请参见[Zuul过滤器包](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-zuul/src/main/java/org/springframework/cloud/netflix/zuul/filters)。如果要禁用一个，请设置`zuul...disable=true`。按照惯例，后面的包`filters`是Zuul过滤器类型。例如禁用`org.springframework.cloud.netflix.zuul.filters.post.SendResponseFilter`，设置`zuul.SendResponseFilter.post.disable=true`。

### [9.13。提供路线的Hystrix后备](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#hystrix-fallbacks-for-routes)

当Zuul中给定路线的电路跳闸时，您可以通过创建type的bean提供后备响应`FallbackProvider`。在此bean中，您需要指定回退的路由ID，并提供一个`ClientHttpResponse`作为回退的return。以下示例显示了一个相对简单的`FallbackProvider`实现：

```java
class MyFallbackProvider implements FallbackProvider {

    @Override
    public String getRoute() {
        return "customers";
    }

    @Override
    public ClientHttpResponse fallbackResponse(String route, final Throwable cause) {
        if (cause instanceof HystrixTimeoutException) {
            return response(HttpStatus.GATEWAY_TIMEOUT);
        } else {
            return response(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    private ClientHttpResponse response(final HttpStatus status) {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return status;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return status.value();
            }

            @Override
            public String getStatusText() throws IOException {
                return status.getReasonPhrase();
            }

            @Override
            public void close() {
            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("fallback".getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }
}
```

以下示例显示了上一个示例的路由配置可能如何显示：

```yaml
zuul:
  routes:
    customers: /customers/**
```

如果您想为所有路由提供默认的后备，则可以创建一个类型为bean `FallbackProvider`的`getRoute`方法，使其具有return `*`或方法`null`，如以下示例所示：

```java
class MyFallbackProvider implements FallbackProvider {
    @Override
    public String getRoute() {
        return "*";
    }

    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable throwable) {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return 200;
            }

            @Override
            public String getStatusText() throws IOException {
                return "OK";
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("fallback".getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }
}
```

### [9.14。Zuul超时](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-timeouts)

如果要为通过Zuul代理的请求配置套接字超时和读取超时，则根据您的配置，有两种选择：

- 如果Zuul使用服务发现，则需要使用`ribbon.ReadTimeout`和`ribbon.SocketTimeout`功能区属性配置这些超时 。

如果通过指定URL配置了Zuul路由，则需要使用 `zuul.host.connect-timeout-millis`和`zuul.host.socket-timeout-millis`。

### [9.15。重写`Location`标题](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-redirect-location-rewrite)

如果Zuul在Web应用程序的前面，则`Location`当Web应用程序通过HTTP状态代码重定向时，您可能需要重新编写标头`3XX`。否则，浏览器将重定向到Web应用程序的URL，而不是Zuul URL。您可以配置`LocationRewriteFilter`Zuul过滤器以将`Location`标头重写为Zuul的URL。它还添加回去的全局前缀和特定于路由的前缀。以下示例通过使用Spring Configuration文件添加过滤器：

```java
import org.springframework.cloud.netflix.zuul.filters.post.LocationRewriteFilter;
...

@Configuration
@EnableZuulProxy
public class ZuulConfig {
    @Bean
    public LocationRewriteFilter locationRewriteFilter() {
        return new LocationRewriteFilter();
    }
}
```

|      | 小心使用此过滤器。筛选器作用于`Location`所有`3XX`响应代码的标头，这可能不适用于所有情况，例如将用户重定向到外部URL时。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### [9.16。启用跨源请求](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#enabling-cross-origin-requests)

默认情况下，Zuul将所有跨源请求（CORS）路由到服务。如果您想让Zuul处理这些请求，可以通过提供自定义`WebMvcConfigurer`bean 来完成：

```java
@Bean
public WebMvcConfigurer corsConfigurer() {
    return new WebMvcConfigurer() {
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/path-1/**")
                    .allowedOrigins("https://allowed-origin.com")
                    .allowedMethods("GET", "POST");
        }
    };
}
```

在上面的示例中，我们允许`GET`和`POST`从的方法`allowed-origin.com`将跨域请求发送到以开头的端点`path-1`。您可以使用`/**`映射将CORS配置应用于特定的路径模式或整个应用程序的全局路径。您可以自定义属性：`allowedOrigins`，`allowedMethods`，`allowedHeaders`，`exposedHeaders`，`allowCredentials`并`maxAge`通过此配置。

### [9.17。指标](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#metrics)

Zuul将在执行器指标终结点下提供指标，以解决路由请求时可能发生的任何故障。您可以点击来查看这些指标`/actuator/metrics`。指标将具有格式为的名称 `ZUUL::EXCEPTION:errorCause:statusCode`。

### [9.18。Zuul开发人员指南](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-developer-guide)

有关Zuul如何工作的一般概述，请参见[Zuul Wiki](https://github.com/Netflix/zuul/wiki/How-it-Works)。

#### [9.18.1。Zuul Servlet](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#the-zuul-servlet)

Zuul被实现为Servlet。对于一般情况，Zuul已嵌入到Spring Dispatch机制中。这使Spring MVC可以控制路由。在这种情况下，Zuul缓冲请求。如果需要不缓存请求就通过Zuul（例如，用于大文件上传），则Servlet也会安装在Spring Dispatcher的外部。默认情况下，该servlet的地址为`/zuul`。可以使用该`zuul.servlet-path`属性更改此路径。

#### [9.18.2。Zuul RequestContext](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-requestcontext)

要在过滤器之间传递信息，Zuul使用[`RequestContext`](https://github.com/Netflix/zuul/blob/1.x/zuul-core/src/main/java/com/netflix/zuul/context/RequestContext.java)。它的数据保存在`ThreadLocal`每个请求的特定内容中。有关在何处路由请求，错误以及实际`HttpServletRequest`和`HttpServletResponse`的信息存储在此处。在`RequestContext`扩展`ConcurrentHashMap`，所以什么都可以存储在上下文。[`FilterConstants`](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-zuul/src/main/java/org/springframework/cloud/netflix/zuul/filters/support/FilterConstants.java)包含由Spring Cloud Netflix安装的过滤器使用的密钥（[稍后会](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-developer-guide-enable-filters)详细介绍）。

#### [9.18.3。`@EnableZuulProxy`与`@EnableZuulServer`](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#enablezuulproxy-vs-enablezuulserver)

Spring Cloud Netflix安装了许多过滤器，具体取决于用于启用Zuul的注释。`@EnableZuulProxy`是的超集`@EnableZuulServer`。换句话说，`@EnableZuulProxy`包含所安装的所有过滤器`@EnableZuulServer`。“代理”中的其他过滤器启用路由功能。如果您想要“空白” Zuul，则应使用`@EnableZuulServer`。

#### [9.18.4。`@EnableZuulServer`筛选器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-developer-guide-enable-filters)

`@EnableZuulServer`创建一个`SimpleRouteLocator`从Spring Boot配置文件加载路由定义的。

安装了以下过滤器（作为普通的Spring Bean）：

- 前置过滤器：
  - `ServletDetectionFilter`：检测请求是否通过Spring Dispatcher。设置键为的布尔值`FilterConstants.IS_DISPATCHER_SERVLET_REQUEST_KEY`。
  - `FormBodyWrapperFilter`：解析表单数据并为下游请求重新编码。
  - `DebugFilter`：如果`debug`设置了请求参数，则将`RequestContext.setDebugRouting()`和设置`RequestContext.setDebugRequest()`为`true`。
- 路线过滤器：
  - `SendForwardFilter`：通过使用Servlet转发请求`RequestDispatcher`。转发位置存储在`RequestContext`属性中`FilterConstants.FORWARD_TO_KEY`。这对于转发到当前应用程序中的端点很有用。
- 帖子过滤器：
  - `SendResponseFilter`：将代理请求的响应写入当前响应。
- 错误过滤器：
  - `SendErrorFilter`：`/error`如果`RequestContext.getThrowable()`不为null，则转发到（默认情况下）。您可以`/error`通过设置`error.path`属性来更改默认转发路径（）。

#### [9.18.5。`@EnableZuulProxy`筛选器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#enablezuulproxy-filters)

创建一个`DiscoveryClientRouteLocator`可从`DiscoveryClient`（例如Eureka）以及从属性加载路线定义的。路线为每个创建`serviceId`从所述`DiscoveryClient`。添加新服务后，将刷新路由。

除了前面描述的过滤器之外，还安装了以下过滤器（作为普通的Spring Bean）：

- 前置过滤器：
  - `PreDecorationFilter`：根据提供的确定位置和路线`RouteLocator`。它还为下游请求设置了各种与代理相关的标头。
- 路线过滤器：
  - `RibbonRoutingFilter`：使用Ribbon，Hystrix和可插拔HTTP客户端发送请求。服务ID在`RequestContext`属性中找到`FilterConstants.SERVICE_ID_KEY`。此过滤器可以使用不同的HTTP客户端：
    - Apache `HttpClient`：默认客户端。
    - Squareup `OkHttpClient`v3：通过将`com.squareup.okhttp3:okhttp`库放在类路径上并设置来启用`ribbon.okhttp.enabled=true`。
    - Netflix Ribbon HTTP客户端：通过设置启用`ribbon.restclient.enabled=true`。该客户端具有局限性，包括不支持PATCH方法，但是还具有内置的重试功能。
  - `SimpleHostRoutingFilter`：通过Apache HttpClient将请求发送到预定的URL。网址位于中`RequestContext.getRouteHost()`。

#### [9.18.6。自定义Zuul过滤器示例](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#custom-zuul-filter-examples)

下面的大多数“如何编写”示例都包含在[示例Zuul过滤器](https://github.com/spring-cloud-samples/sample-zuul-filters)项目中。在该存储库中也有一些处理请求或响应正文的示例。

本节包括以下示例：

- [如何编写前置过滤器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-developer-guide-sample-pre-filter)
- [如何编写路由过滤器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-developer-guide-sample-route-filter)
- [如何编写帖子过滤器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-developer-guide-sample-post-filter)

##### [如何编写前置过滤器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-developer-guide-sample-pre-filter)

前置过滤器会在中设置数据，以`RequestContext`供下游过滤器使用。主要用例是设置路由过滤器所需的信息。以下示例显示了Zuul预过滤器：

```java
public class QueryParamPreFilter extends ZuulFilter {
    @Override
    public int filterOrder() {
        return PRE_DECORATION_FILTER_ORDER - 1; // run before PreDecoration
    }

    @Override
    public String filterType() {
        return PRE_TYPE;
    }

    @Override
    public boolean shouldFilter() {
        RequestContext ctx = RequestContext.getCurrentContext();
        return !ctx.containsKey(FORWARD_TO_KEY) // a filter has already forwarded
                && !ctx.containsKey(SERVICE_ID_KEY); // a filter has already determined serviceId
    }
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        if (request.getParameter("sample") != null) {
            // put the serviceId in `RequestContext`
            ctx.put(SERVICE_ID_KEY, request.getParameter("foo"));
        }
        return null;
    }
}
```

前面的过滤器`SERVICE_ID_KEY`从`sample`request参数填充。实际上，您不应该执行这种直接映射。相反，应该从`sample`相反的值中查找服务ID 。

现在`SERVICE_ID_KEY`已填充，将`PreDecorationFilter`无法运行`RibbonRoutingFilter`。

|      | 如果要路由到完整URL，请致电`ctx.setRouteHost(url)`。 |
| ---- | ---------------------------------------------------- |
|      |                                                      |

要修改路由过滤器转发到的路径，请设置`REQUEST_URI_KEY`。

##### [如何编写路由过滤器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-developer-guide-sample-route-filter)

路由过滤器在预过滤器之后运行，并向其他服务发出请求。这里的许多工作是将请求和响应数据与客户端所需的模型相互转换。以下示例显示了Zuul路由过滤器：

```java
public class OkHttpRoutingFilter extends ZuulFilter {
    @Autowired
    private ProxyRequestHelper helper;

    @Override
    public String filterType() {
        return ROUTE_TYPE;
    }

    @Override
    public int filterOrder() {
        return SIMPLE_HOST_ROUTING_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        return RequestContext.getCurrentContext().getRouteHost() != null
                && RequestContext.getCurrentContext().sendZuulResponse();
    }

    @Override
    public Object run() {
        OkHttpClient httpClient = new OkHttpClient.Builder()
                // customize
                .build();

        RequestContext context = RequestContext.getCurrentContext();
        HttpServletRequest request = context.getRequest();

        String method = request.getMethod();

        String uri = this.helper.buildZuulRequestURI(request);

        Headers.Builder headers = new Headers.Builder();
        Enumeration<String> headerNames = request.getHeaderNames();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            Enumeration<String> values = request.getHeaders(name);

            while (values.hasMoreElements()) {
                String value = values.nextElement();
                headers.add(name, value);
            }
        }

        InputStream inputStream = request.getInputStream();

        RequestBody requestBody = null;
        if (inputStream != null && HttpMethod.permitsRequestBody(method)) {
            MediaType mediaType = null;
            if (headers.get("Content-Type") != null) {
                mediaType = MediaType.parse(headers.get("Content-Type"));
            }
            requestBody = RequestBody.create(mediaType, StreamUtils.copyToByteArray(inputStream));
        }

        Request.Builder builder = new Request.Builder()
                .headers(headers.build())
                .url(uri)
                .method(method, requestBody);

        Response response = httpClient.newCall(builder.build()).execute();

        LinkedMultiValueMap<String, String> responseHeaders = new LinkedMultiValueMap<>();

        for (Map.Entry<String, List<String>> entry : response.headers().toMultimap().entrySet()) {
            responseHeaders.put(entry.getKey(), entry.getValue());
        }

        this.helper.setResponse(response.code(), response.body().byteStream(),
                responseHeaders);
        context.setRouteHost(null); // prevent SimpleHostRoutingFilter from running
        return null;
    }
}
```

前面的过滤器将Servlet请求信息转换为OkHttp3请求信息，执行HTTP请求，并将OkHttp3响应信息转换为Servlet响应。

##### [如何编写帖子过滤器](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-developer-guide-sample-post-filter)

后置过滤器通常操纵响应。以下过滤器将随机数添加`UUID`为`X-Sample`标题：

```java
public class AddResponseHeaderFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return POST_TYPE;
    }

    @Override
    public int filterOrder() {
        return SEND_RESPONSE_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext context = RequestContext.getCurrentContext();
        HttpServletResponse servletResponse = context.getResponse();
        servletResponse.addHeader("X-Sample", UUID.randomUUID().toString());
        return null;
    }
}
```

|      | 其他操作，例如转换响应主体，则更加复杂且计算量大。 |
| ---- | -------------------------------------------------- |
|      |                                                    |

#### [9.18.7。Zuul错误如何工作](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#how-zuul-errors-work)

如果在Zuul过滤器生命周期的任何部分抛出异常，则将执行错误过滤器。该`SendErrorFilter`如果只运行`RequestContext.getThrowable()`不`null`。然后，它`javax.servlet.error.*`在请求中设置特定的属性，并将请求转发到Spring Boot错误页面。

#### [9.18.8。Zuul Eager应用程序上下文加载](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul-eager-application-context-loading)

Zuul在内部使用Ribbon来调用远程URL。默认情况下，丝带云客户端在第一次调用时由Spring Cloud延迟加载。可以通过使用以下配置为Zuul更改此行为，这将导致在应用程序启动时急于加载与子Ribbon相关的应用程序上下文。以下示例显示了如何启用即时加载：

application.yml

```
祖尔：
  带：
    急切负载：
      已启用：true
```

## [10. Sidecar支持多语种](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#polyglot-support-with-sidecar)

您是否要使用非JVM语言来利用Eureka，Ribbon和Config Server？Spring Cloud Netflix Sidecar的灵感来自[Netflix Prana](https://github.com/Netflix/Prana)。它包括一个HTTP API，用于获取给定服务的所有实例（按主机和端口）。您也可以通过嵌入式Zuul代理来代理服务呼叫，该代理从Eureka获取其路由条目。可以通过主机查找或通过Zuul代理直接访问Spring Cloud Config Server。非JVM应用程序应该执行运行状况检查，以便Sidecar可以向Eureka报告应用程序是打开还是关闭。

要将Sidecar包含在您的项目中，请使用组ID为`org.springframework.cloud` 和工件ID为或的依赖项`spring-cloud-netflix-sidecar`。

要启用Sidecar，请使用创建一个Spring Boot应用程序`@EnableSidecar`。这个注解包括`@EnableCircuitBreaker`，`@EnableDiscoveryClient`，和`@EnableZuulProxy`。在与非JVM应用程序相同的主机上运行结果应用程序。

要配置的车边，加`sidecar.port`和`sidecar.health-uri`来`application.yml`。该`sidecar.port`属性是非JVM应用程序侦听的端口。这样，Sidecar可以在Eureka中正确注册该应用程序。这些`sidecar.secure-port-enabled`选项提供了一种启用流量安全端口的方法。该`sidecar.health-uri`是一个URI的非JVM的应用程序，模仿春引导健康指标接近。它应该返回类似于以下内容的JSON文档：

health-uri-document

```json
{
  "status":"UP"
}
```

以下application.yml示例显示了Sidecar应用程序的示例配置：

application.yml

```yaml
server:
  port: 5678
spring:
  application:
    name: sidecar

sidecar:
  port: 8000
  health-uri: http://localhost:8000/health.json
```

该`DiscoveryClient.getInstances()`方法的API 是`/hosts/{serviceId}`。以下示例响应用于`/hosts/customers`返回不同主机上的两个实例：

/主机/客户

```json
[
    {
        "host": "myhost",
        "port": 9000,
        "uri": "https://myhost:9000",
        "serviceId": "CUSTOMERS",
        "secure": false
    },
    {
        "host": "myhost2",
        "port": 9000,
        "uri": "https://myhost2:9000",
        "serviceId": "CUSTOMERS",
        "secure": false
    }
]
```

非JVM应用程序（如果Sidecar位于端口5678上）可通过访问该API `localhost:5678/hosts/{serviceId}`。

Zuul代理会自动为Eureka中已知的每个服务添加路由`/`，因此可以在上使用客户服务`/customers`。非JVM应用程序可以在以下位置访问客户服务`localhost:5678/customers`（假设Sidecar正在侦听5678端口）。

如果Config Server已向Eureka注册，则非JVM应用程序可以通过Zuul代理对其进行访问。如果`serviceId`ConfigServer的为`configserver`，并且Sidecar在端口5678上，则可以在[localhost：5678 / configserver上](http://localhost:5678/configserver)对其进行访问。

非JVM应用程序可以利用Config Server返回YAML文档的功能。例如，对[sidecar.local.spring.io:5678/configserver/default-master.yml](https://sidecar.local.spring.io:5678/configserver/default-master.yml)的调用 可能会导致YAML文档类似于以下内容：

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
  password: password
info:
  description: Spring Cloud Samples
  url: https://github.com/spring-cloud-samples
```

为了使健康检查请求在使用设置`sidecar.accept-all-ssl-certificates`为true的HTTP时能够接受所有证书。

## [11.重试失败的请求](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#retrying-failed-requests)

Spring Cloud Netflix提供了多种发出HTTP请求的方式。您可以使用负载平衡`RestTemplate`，功能区或伪装。无论您如何选择创建HTTP请求，始终都有一个请求失败的机会。当请求失败时，您可能希望自动重试该请求。为此，在使用Sping Cloud Netflix时，需要在应用程序的类路径中包括[Spring Retry](https://github.com/spring-projects/spring-retry)。当出现Spring Retry时，负载平衡`RestTemplates`，Feign和Zuul会自动重试任何失败的请求（假设您的配置允许这样做）。

### [11.1。退避政策](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#backoff-policies)

默认情况下，重试请求时不使用任何退避策略。如果要配置退避策略，则需要创建一个类型为Bean `LoadBalancedRetryFactory`并`createBackOffPolicy`为给定服务覆盖方法，如以下示例所示：

```java
@Configuration
public class MyConfiguration {
    @Bean
    LoadBalancedRetryFactory retryFactory() {
        return new LoadBalancedRetryFactory() {
            @Override
            public BackOffPolicy createBackOffPolicy(String service) {
                return new ExponentialBackOffPolicy();
            }
        };
    }
}
```

### [11.2。组态](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#configuration)

当将Ribbon与Spring Retry一起使用时，可以通过配置某些Ribbon属性来控制重试功能。要做到这一点，设置`client.ribbon.MaxAutoRetries`，`client.ribbon.MaxAutoRetriesNextServer`和`client.ribbon.OkToRetryOnAllOperations`性能。有关这些属性的描述，请参见[功能区文档](https://github.com/Netflix/ribbon/wiki/Getting-Started#the-properties-file-sample-clientproperties)。

|      | 启用`client.ribbon.OkToRetryOnAllOperations`包括重试POST请求，由于请求正文的缓冲，这可能会对服务器资源产生影响。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | The property names are case-sensitive, and since some of these properties are defined in the Netflix Ribbon project, they are in Pascal Case and the ones from Spring Cloud are in Camel Case. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

In addition, you may want to retry requests when certain status codes are returned in the response. You can list the response codes you would like the Ribbon client to retry by setting the `clientName.ribbon.retryableStatusCodes` property, as shown in the following example:

```yaml
clientName:
  ribbon:
    retryableStatusCodes: 404,502
```

You can also create a bean of type `LoadBalancedRetryPolicy` and implement the `retryableStatusCode` method to retry a request given the status code.

#### [11.2.1. Zuul](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zuul)

You can turn off Zuul’s retry functionality by setting `zuul.retryable` to `false`. You can also disable retry functionality on a route-by-route basis by setting `zuul.routes.routename.retryable` to `false`.

## [12. HTTP Clients](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#http-clients)

Spring Cloud Netflix会自动为您创建Ribbon，Feign和Zuul使用的HTTP客户端。但是，您也可以根据需要提供自定义的HTTP客户端。为此，`CloseableHttpClient`如果您正在使用Apache HTTP Client或`OkHttpClient`使用OK HTTP ，则可以创建一个类型的bean 。

|      | 创建自己的HTTP客户端时，您还负责为这些客户端实施正确的连接管理策略。这样做不当会导致资源管理问题。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## [13.维护模式下的模块](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#modules-in-maintenance-mode)

将模块置于维护模式意味着Spring Cloud团队将不再向该模块添加新功能。我们将修复阻止程序错误和安全性问题，还将考虑并审查社区的一些小请求。

自格林威治发布列车全面上市以来，我们打算继续为这些模块提供至少一年的支持。

以下Spring Cloud Netflix模块和相应的启动器将进入维护模式：

- 春云netflix-archaius
- Spring-Cloud-NetFlix-Hystrix合同
- Spring-Cloud-NetFlix-Hystrix-仪表板
- Spring-Cloud-NetFlix-Hystrix-流
- 春云netflix-hystrix
- 春天云netflix丝带
- 春云netflix涡轮流
- 弹簧云netflix涡轮
- 春天云netflix-zuul

|      | 这不包括Eureka或并发限制模块。 |
| ---- | ------------------------------ |
|      |                                |

## [14.配置属性](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#configuration-properties)

要查看所有与Spring Cloud Netflix相关的配置属性的列表，请检查[附录页面](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/appendix.html)。

<iframe id="blockbyte-bs-sidebar" class="notranslate" data-pos="left" style="box-sizing: border-box; opacity: 0; pointer-events: none; position: fixed; top: 0px; left: 0px; width: 350px; max-width: none; height: 0px; z-index: 2147483646; border: none; transform: translate3d(-350px, 0px, 0px); transition: width 0s ease 0.3s, height 0s ease 0.3s, opacity 0.3s ease 0s, transform 0.3s ease 0s; background-color: rgba(255, 255, 255, 0.8) !important; display: block !important; color: rgb(0, 0, 0); font-family: Karla, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;"></iframe>