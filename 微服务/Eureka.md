Eureka

⼀ 服务发现

\1. 服务发现与注册

在微服务架构中，服务发现（Service Discovery）是关键原则之⼀。⼿动配置每个客

户端或某种形式的约定是很难做的，并且很脆弱。Spring Cloud提供了多种服务发现

的实现⽅式，例如：Eureka、Consul、Zookeeper。Spring Cloud⽀持得最好的是

Eureka，其次是Consul，最次是Zookeeper。

\2. 服务发现模式

服务发现有两种模式,⼀是客户端发现模式,⼆是服务端发现模式

2.1 客户端发现模式

使⽤客户端发现模式时，客户端(服务消费者)决定相应服务实例(服务提供者)的⽹络

位置，并且对请求实现负载均衡。客户端查询服务注册表，服务注册表是⼀个保存有

所有可⽤服务的⼀个数据库等具有存储功能的地⽅；然后客户端使⽤负载均衡算法从

中选择⼀个提供者实例，并发出请求。相当于我们去吃⾃助,吃什么,吃多少都是我们

⾃⼰决定的,哪怕其中的菜每种都有多盘,我们也可以⾃⼰决定从哪盘中拿,他们只需要

把所有的东⻄摆放在那⾥就⾏

下图显示了这种模式的架构：

下图中可以看出左侧的消费者(客户端)⾃⼰从注册中⼼查询右侧的所有的服务,然后⾃⼰决定去请求

哪个服务2.2 服务端发现模式

与客户端发现⽅式不同,服务端发现是由专⻔的某个或者某⼏个服务器来发现服务后

再对外提供信息的,客户端(消费者)并不知道这个服务到底有多少个,就像我们在⻝堂

打饭,我们只需要告诉打饭的阿姨你吃什么,阿姨她来决定从哪盘⾥⾯打给你(当然⻝堂

⾥⾯你还能看到菜⽽已,我们的服务中你⽆法看到服务)

下图就是服务的发现的⼀个结构图:可以看到我们的服务消费者并不是直接再连接注册中⼼查询服务并决定加载哪个了,⽽是连接负载均

衡服务器告诉它我需要⼀个什么服务,负载均衡服务器会去注册中⼼查看有哪些服务,并通过负载均

衡返回⼀个服务交给消费者调⽤,这个时候客户端是⽆法决定⾃⼰到底访问哪个实例的

服务端发现模式它最⼤的优点是客户端⽆需关注发现的细节，只需要简单地向负载均衡器发送请求，

这减少了编程语⾔框架需要完成的发现逻辑。并且如上⽂所述，某些部署环境免费提供这⼀功能。这

种模式也有缺点。除⾮负载均衡器由部署环境提供，否则会成为⼀个需要配置和管理的⾼可⽤系统组

件

2.3 服务注册表

服务注册表是服务发现的核⼼部分，是包含服务实例的⽹络地址的数据库。服务注册

表需要⾼可⽤⽽且随时更新。客户端能够缓存从服务注册表中获取的⽹络地址，然

⽽，这些信息最终会过时，客户端也就⽆法发现服务实例。因此，服务注册表会包含

若⼲服务端，使⽤复制协议保持⼀致性。

常⻅的服务注册表: Eureka,Zookeeper,Consul

3 ⼩结在微服务应⽤中，服务实例的运⾏环境会动态变化，实例⽹络地址也是如此。因此，客户端为了访问

服务必须使⽤服务发现机制。

服务注册表是服务发现的关键部分。服务注册表是可⽤服务实例的数据库，提供管理 API 和查询

API。服务实例使⽤管理 API 来实现注册和注销，系统组件使⽤查询 API 来发现可⽤的服务实

例。

服务发现有两种主要模式：客户端发现和服务端发现。在使⽤客户端服务发现的系统中，客户端查询

服务注册表，选择可⽤的服务实例，然后发出请求。在使⽤服务端发现的系统中，客户端通过路由转

发请求，路由器查询服务注册表并转发请求到可⽤的实例。

服务实例的注册和注销也有两种⽅式。⼀种是服务实例⾃⼰注册到服务注册表中，即⾃注册模式；另

⼀种则是由其它系统组件处理注册和注销，也就是第三⽅注册模式。

在⼀些部署环境中，需要使⽤ Netflix Eureka、etcd、Apache Zookeeper 等服务发现来设

置⾃⼰的服务发现基础设施。⽽另⼀些部署环境则内置了服务发现。例如，Kubernetes 和

Marathon 处理服务实例的注册和注销，它们也在每个集群主机上运⾏代理，这个代理具有服务端

发现路由的功能。

HTTP 反向代理和 NGINX 这样的负载均衡器能够⽤做服务器端的服务发现均衡器。服务注册表能

够将路由信息推送到 NGINX，激活配置更新，譬如使⽤ Cosul Template。NGINX Plus ⽀持

额外的动态配置机制，能够通过 DNS 从注册表中获取服务实例的信息，并为远程配置提供 API

⼆ Eureka

2.1 Eureka介绍

Eureka是Netflix开发的服务发现框架，本身是⼀个基于REST的服务，主要⽤于定位运

⾏在AWS域中的中间层服务，以达到负载均衡和中间层服务故障转移的⽬的。Spring

Cloud将它集成在其⼦项⽬spring-cloud-netflix中，以实现Spring Cloud的服务发现功

能。

Github地址:https://github.com/Netflix/eureka

2.2 Eureka原理

wiki地址:https://github.com/Netflix/eureka/wiki/Eureka-at-a-glance下图是Eureka的结构图:

上图是来⾃Eureka官⽅的架构图，⼤致描述了Eureka集群的⼯作过程

\- Application Service 就相当于服务提供者，Application Client就相当于服务消费

者；

\- Make Remote Call，可以简单理解为调⽤RESTful的接⼝；

\- us-east-1c、us-east-1d等是zone，它们都属于us-east-1这个region；类似于我们阿⾥

云上⾯的华东2区下的⻘岛,其中⻘岛就是zone,华东就是region

由图可知，Eureka包含两个组件：Eureka Server 和 Eureka Client。

Eureka Server提供服务注册服务，各个节点启动后，会在Eureka Server中进⾏注册，这样

Eureka Server中的服务注册表中将会存储所有可⽤服务节点的信息，服务节点的信息可以在界⾯

中直观的看到。

Eureka Client是⼀个Java客户端，⽤于简化与Eureka Server的交互，客户端同时也具备⼀

个内置的、使⽤轮询（round-robin）负载算法的负载均衡器。在应⽤启动后，将会向Eureka Server发送⼼跳（默认周期为30秒）。如果Eureka Server在多

个⼼跳周期内没有接收到某个节点的⼼跳，Eureka Server将会从服务注册表中把这个服务节点移

除（默认90秒）。

Eureka Server之间将会通过复制的⽅式完成数据的同步.

Eureka还提供了客户端缓存的机制，即使所有的Eureka Server都挂掉，客户端依然可以利⽤缓

存中的信息消费其他服务的API.

综上，Eureka通过⼼跳检测、健康检查、客户端缓存等机制，确保了系统的⾼可⽤性、灵活性和可

伸缩性。

2.3 Eureka和Zookeeper⽐较

著名的CAP理论指出，⼀个分布式系统不可能同时满⾜C(⼀致性)、A(可⽤性)和P(分区

容错性)。由于分区容错性在是分布式系统中必须要保证的，因此我们只能在A和C之

间进⾏权衡。在此Zookeeper保证的是CP, ⽽Eureka则是AP。

2.3.1 Zookeeper保证CP

当向注册中⼼查询服务列表时，我们可以容忍注册中⼼返回的是⼏分钟以前的注册信息，但不能接受

服务直接down掉不可⽤。也就是说，服务注册功能对可⽤性的要求要⾼于⼀致性。但是zk会出现这

样⼀种情况，当master节点因为⽹络故障与其他节点失去联系时，剩余节点会重新进⾏leader选

举。问题在于，选举leader的时间太⻓，30 ~ 120s, 且选举期间整个zk集群都是不可⽤的，这

就导致在选举期间注册服务瘫痪。在云部署的环境下，因⽹络问题使得zk集群失去master节点是较

⼤概率会发⽣的事，虽然服务能够最终恢复，但是漫⻓的选举时间导致的注册⻓期不可⽤是不能容忍

的。

2.3.2 Eureka保证APEureka看明⽩了这⼀点，因此在设计时就优先保证可⽤性。Eureka各个节点都是平等的，⼏个节

点挂掉不会影响正常节点的⼯作，剩余的节点依然可以提供注册和查询服务。⽽Eureka的客户端在

向某个Eureka注册或时如果发现连接失败，则会⾃动切换⾄其它节点，只要有⼀台Eureka还在，

就能保证注册服务可⽤(保证可⽤性)，只不过查到的信息可能不是最新的(不保证强⼀致性)。除此

之外，Eureka还有⼀种⾃我保护机制，如果在15分钟内超过85%的节点都没有正常的⼼跳，那么

Eureka就认为客户端与注册中⼼出现了⽹络故障，此时会出现以下⼏种情况：

\1. Eureka不再从注册列表中移除因为⻓时间没收到⼼跳⽽应该过期的服务

\2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节

点依然可⽤)

\3. 当⽹络稳定时，当前实例新的注册信息会被同步到其它节点中

因此， Eureka可以很好的应对因⽹络故障导致部分节点失去联系的情况，⽽不会像zookeeper那

样使整个注册服务瘫痪。

Eureka作为单纯的服务注册中⼼来说要⽐zookeeper更加“专业”，因为注册服务更重

要的是可⽤性，我们可以接受短期内达不到⼀致性的状况。不过Eureka⽬前1.X版本

的实现是基于servlet的Java web应⽤，它的极限性能肯定会受到影响。期待正在开

发之中的2.X版本能够从servlet中独⽴出来成为单独可部署执⾏的服务。

2.4 Eureka 基本使⽤

2.4.1 搭建Eureka服务端

因为作为注册中⼼⼀定是独⽴于其他程序之外的,所以我们需要搭建⼀个Eureka的服

务端,与Zookeeper不同的是,Eureka的服务端是⼀个JavaWeb程序

参考代码中 03eurekaserver

2.4.1.1 Pom.xml

此pom中只包含了依赖,<dependencies>

<!--Eureka server 的依赖-->

<dependency> 

<groupId>org.springframework.cloud</groupId> 

<artifactId>spring-cloud-starter-netflix-eureka

server</artifactId>

</dependency>

</dependencies>

2.4.1.2 application.yml

在resources⽬录下创建application.yml

server:

 port: 10000 #程序通过springboot启动后tomcat的端⼝

eureka:

 client:

 register-with-eureka: false #eureka单机版的配置

 fetchRegistry: false #表示当前是个服务端

 service-url:

 defaultZone: http://localhost:10000/eureka #Eureka对外提供服务的注册地

址,也就是eureka最终运⾏的地址,此处我们使⽤的是localhost,springboot模式启动,所以是

localhost:10000/eureka

spring:

 application:

 name: eureka-server #程序的名字

2.4.1.3 启动主程序

Eureka默认情况下不需要其他的额外代码,只需要⼀个主程序即可

import org.springframework.boot.SpringApplication;

import org.springframework.boot.autoconfigure.SpringBootApplication;

import

org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

/**

\* Created by jackiechan on 19-6-8 上午2:50

*

\* @Author jackiechan

*/

@SpringBootApplication@EnableEurekaServer//开启eurekaserver,会⾃动帮我们配置

public class EurekaStartApp {

public static void main (String[] args){

SpringApplication.run(EurekaStartApp.class,args);

 }

}

2.4.1.4 效果

启动后访问http://localhost:10000/

2.4.2 服务提供者修改

参考 04provider-eureka,消费者配置同提供者,但是调⽤的时候需要修改⼀些东⻄,所以

消费者单独写

2.4.2.1 POM.xml中添加依赖<dependency>

<!--注意是starter client-->

<groupId>org.springframework.cloud</groupId> 

<artifactId>spring-cloud-starter-netflix-eureka

client</artifactId>

</dependency>

2.4.2.2 application.yml

server:

 port: 8001

spring:

 datasource:

 username: root

 password: qishimeiyoumima

 driver-class-name: com.mysql.jdbc.Driver

 url: jdbc:mysql:///db_shopping?useUnicode=true&characterEncoding=utf8

 type: com.alibaba.druid.pool.DruidDataSource

\#添加程序的名字

 application:

 name: 04provider-eureka #注意这个名字,eureka server 区分服务的⽅式就是通过

这个名字,相同名字的服务会被归为集群

\#配置eureka服务端地址

eureka: #注册中⼼的地址

 client:

 serviceUrl:

 defaultZone: http://localhost:10000/eureka #这⾥说eurekaserver运⾏起

来后的地址

2.4.2.3 主程序修改

在主程序中ProviderStarter添加注解,注册到eureka

import org.mybatis.spring.annotation.MapperScan;

import org.springframework.boot.SpringApplication;

import org.springframework.boot.autoconfigure.SpringBootApplication;

import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

/**

\* Created by jackiechan on 19-6-8 上午12:09

*

\* @Author jackiechan*/

@SpringBootApplication

@MapperScan("com.qianfeng.microservice.provider.mapper")

@EnableEurekaClient //开启服务注册和发现,只要添加此注解

public class ProviderStarter {

public static void main (String[] args){

SpringApplication.run(ProviderStarter.class,args);

 }

}

2.4.2.4 启动提供者

启动后再刷新eureka⽹⻚,可以看到有⼀个服务已经注册上来了

2.5 Eureka 密码保护

2.5.1 Eureka Server修改

2.5.1.1 POM.xml添加依赖

SpringCloud通过Spring security来进⾏安全管理,所以需要导⼊这个依赖包<!--

安全组件,⽤于设置访问的时候需要密码

-->

<dependency> 

<groupId>org.springframework.boot</groupId> 

<artifactId>spring-boot-starter-security</artifactId>

</dependency>

2.5.1.2 设置密码

导⼊security后如果不设置⽤户名密码会有默认的⽤户名密码,⽤户名是user,密码是每

次启动的时候会在控制台打印⼀个随机密码,所以我们需要在application.yml中设置密

码

server:

 port: 10000 #程序通过springboot启动后tomcat的端⼝

eureka:

 client:

 register-with-eureka: false #eureka单机版的配置

 fetchRegistry: false

 serviceUrl:

 defaultZone: http://localhost:10000/eureka #Eureka对外提供服务的注册地

址,也就是eureka最终运⾏的地址,此处我们使⽤的是localhost,springboot模式启动,所以是

localhost:10000/eureka

spring:

 application:

 name: eureka-server #程序的名字

 security: #设置密码,注意是在spring层级下⾯

 user:

 password: abc #密码

 name: zhangsan #⽤户名

2.5.1.3 修改安全配置

当我们开启安全配置后,我们发现我们的服务会⽆法注册到eureka上⾯,即便是客户端

也指定了⽤户名和密码仍旧不可以,原因是spring security 2开始,默认对csrf进⾏了拦

截,导致我们的服务注册也会被拦截,⽽我们的springcloud F版本使⽤的就是

springboot2,那么导⼊的security依赖也是2的版本,所以我们需要修改安全配置来允许服务注册

在项⽬中添加⼀个类

import org.springframework.context.annotation.Configuration;

import

org.springframework.security.config.annotation.web.builders.HttpSecurity;

import

org.springframework.security.config.annotation.web.configuration.EnableWe

bSecurity;

import

org.springframework.security.config.annotation.web.configuration.WebSecur

ityConfigurerAdapter;

/**

\* Created by jackiechan on 19-4-20/上午11:14

\* 解决springcloud F版本开始的问题

\* @Author jackiechan

*/

@Configuration //注意要加这个注解

@EnableWebSecurity//注意要加这个注解,开启安全配置

public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

@Override

protected void configure(HttpSecurity http) throws Exception {

super.configure(http);

//http.csrf().disable().httpBasic(); //⽅式1,禁⽤csrf并使⽤http的普

通⽅式,但是使⽤这个后,我们访问eureka会发现配置的⽤户密码是不需要写的,失效了

http.csrf().ignoringAntMatchers("/eureka/**");//我们的服务注册地址是

在/eureka下⾯,所以我们可以通过姜/eureka/**进⾏忽略安全过滤来达到允许服务注册到⽬的,

⽽且其他地址的访问会仍旧需要密码

 }

}

2.5.1.4 启动Eureka Server

启动后访问 http://localhost:10000/ 发现需要⽤户名和密码2.5.2 服务提供者修改

2.5.2.1 修改application.yml

我们的服务只需要修改eureka的地址⻛格即可

server:

 port: 8001

spring:

 datasource:

 username: root

 password: qishimeiyoumima

 driver-class-name: com.mysql.jdbc.Driver

 url: jdbc:mysql:///db_shopping?useUnicode=true&characterEncoding=utf8

 type: com.alibaba.druid.pool.DruidDataSource

 application:

 name: 04provider-eureka

eureka: #注册中⼼的地址

 instance:

 prefer-ip-address: true #在 eureka 后台中访问当前项⽬的时候使⽤ip 地址访

问,默认是机器名的⽅式

 client:

 serviceUrl: defaultZone: http://zhangsan:abc@localhost:10000/eureka #curl⻛格 意

思是⽤户名是zhangsan密码是abc 访问的主机是localhost:10000

2.5.2.2 启动测试

启动后发现服务仍旧可以注册到eureka上, 并且访问时的信息显示为ip

2.6 服务消费者改进

前⾯的操作都是服务提供者的,现在我们开始修改服务消费者

注册到Eureka上⾯来说我们的服务提供者和消费者时⼀致的,我们需要解决的是在没

有使⽤eureka的情况下,我们消费者访问提供者的时候是直接写的⽹址,现在我们需要

通过eureka来获取提供者地址然后发起访问

除了修改controller外其他地⽅和服务提供者修改没区别

参考 05consumer-eureka

2.6.1 POM.xml

添加eureka的依赖⽤于注册服并从eureka上⾯获取服务⽤<dependency> 

<groupId>org.springframework.cloud</groupId> 

<artifactId>spring-cloud-starter-netflix-eureka

client</artifactId>

</dependency>

2.6.2 修改application.yml

server:

 port: 9000

eureka: #注册中⼼的地址

instance:

 prefer-ip-address: true #在 eureka 后台中访问当前项⽬的时候使⽤ip 地址访

问,默认是机器名的⽅式

 client:

 service-url:

 defaultZone: http://zhangsan:abc@localhost:10000/eureka #curl⻛格

spring:

 application:

 name: 05consumer-eureka

2.6.3 修改Controller

因为我们现在要动态获取服务提供者地址,所以我们需要通过代码获取服务提供者的

最新地址

import com.netflix.appinfo.InstanceInfo;

import com.netflix.discovery.EurekaClient;

import com.qianfeng.microservice.consumer.pojo.User;

import org.springframework.beans.factory.annotation.Autowired;

import org.springframework.web.bind.annotation.*;

import org.springframework.web.client.RestTemplate;

/**

\* Created by jackiechan on 19-6-8 下午12:00

*

\* @Author jackiechan

*/

@RestController

@RequestMapping("/userconsumer")

public class UserController {

@Autowiredprivate RestTemplate template;

@Autowired

private EurekaClient eurekaClient; //这个对象spring会⾃动创建不需要我们额

外创建,⽤于帮我们连接eureka

@GetMapping("/info/{id}")

public User getUserById(@PathVariable Long id) throws Exception {

InstanceInfo instanceInfo =

eurekaClient.getNextServerFromEureka("04PROVIDER-EUREKA", false);//从

euerka上获取指定名字的服务的实例,注意⼤⼩写

String url = instanceInfo.getHomePageUrl();//获取服务实例的地址 也就

是 类似于 http://localhost:8000

System.out.println("服务提供者的地址:======>"+url);

User user = template.getForObject(url+"/user/info/" + id,

User.class);//请求指定地址并将返回的json字符串转换为User对象

return user;

 }

@PostMapping("/save")

public String addUser(@RequestBody User user) {

//此⽅法没有修改

String result =

template.postForObject("http://localhost:8000/user/save/", user,

String.class);//post⽅式请求这个地址,并将user作为参数传递过去(json格式),并将返回

结果转换为String类型

return result;

 }

}

2.6.4 修改主程序并启动测试

启动后我们访问这个服务的接⼝地址,发现可以继续获取数据,说明我们的消费者从

eureka上⾯获取到了数据并返回了

import org.springframework.boot.SpringApplication;

import org.springframework.boot.autoconfigure.SpringBootApplication;

import org.springframework.cloud.netflix.eureka.EnableEurekaClient;import org.springframework.context.annotation.Bean;

import org.springframework.web.client.RestTemplate;

/**

\* Created by jackiechan on 19-6-8 下午12:04

*

\* @Author jackiechan

*/

@SpringBootApplication

@EnableEurekaClient //开启服务发现

public class ConsumerStarter {

public static void main (String[] args){

SpringApplication.run(ConsumerStarter.class,args);

 }

@Bean

public RestTemplate restTemplate() {

RestTemplate restTemplate = new RestTemplate();

return restTemplate;

 }

} 

三 常⻅错误

\1. Eureka Server没有启动或者是微服务中配置的Eureka注册中⼼地址错误

我们⽆法注册服务的时候基本都会报错as unable to refresh its cache! status = Cannot

execute request on any known server ,因为错误会⽐较⻓,要详细看,⽐如当前⾥⾯还有

java.net.ConnectException: Connection refused (Connection refused)这个错误,代表我

们的微服务连接的eureka server地址不对或者是没启动com.sun.jersey.api.client.ClientHandlerException:

java.net.ConnectException: Connection refused (Connection refused)

at

com.sun.jersey.client.apache4.ApacheHttpClient4Handler.handle(ApacheHttpC

lient4Handler.java:187) ~[jersey-apache-client4-1.19.1.jar:1.19.1]

......

2019-07-02 11:45:39.019 WARN 62122 --- [ main] 

c.n.d.s.t.d.RetryableEurekaHttpClient : Request execution failed with

message: java.net.ConnectException: Connection refused (Connection

refused)

2019-07-02 11:45:39.021 ERROR 62122 --- [ main]

com.netflix.discovery.DiscoveryClient : DiscoveryClient_04PROVIDER

EUREKA/bogon:04provider-eureka:8001 - was unable to refresh its cache!

status = Cannot execute request on any known server

......

com.netflix.discovery.shared.transport.TransportException: Cannot execute

request on any known server

at

com.netflix.discovery.shared.transport.decorator.RetryableEurekaHttpClien

t.execute(RetryableEurekaHttpClient.java:112) ~[eureka-client-

1.9.3.jar:1.9.3]

\2. 密码问题

当我们的Eureka服务端设置了密码 但是没有将csrf关闭或者是忽略eureka的注册地址

的时候会出现⽆法注册,但是错误提示和上⾯基本⼀样, 都是 Cannot execute request

on any known server 但是可能会出现⼀个Eureka javax.ws.rs.WebApplicationException:

null ,这个时候基本上就是我们没有关闭csrf,通过上⾯的密码保护部分来关闭即可2019-07-02 11:52:53.655 WARN 62155 --- [nfoReplicator-0] 

c.n.discovery.InstanceInfoReplicator : There was a problem with the

instance info replicator

com.netflix.discovery.shared.transport.TransportException: Cannot execute

request on any known server

at

com.netflix.discovery.shared.transport.decorator.RetryableEurekaHttpClien

t.execute(RetryableEurekaHttpClient.java:112) ~[eureka-client-1.9.3.jar:1.9.3]

at

com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorato

r.register(EurekaHttpClientDecorator.java:56) ~[eureka-client-

1.9.3.jar:1.9.3]

at

com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorato

r$1.execute(EurekaHttpClientDecorator.java:59) ~[eureka-client-

1.9.3.jar:1.9.3]

at

com.netflix.discovery.shared.transport.decorator.SessionedEurekaHttpClien

t.execute(SessionedEurekaHttpClient.java:77) ~[eureka-client-

1.9.3.jar:1.9.3]

at

com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorato

r.register(EurekaHttpClientDecorator.java:56) ~[eureka-client-

1.9.3.jar:1.9.3] 

四 Eureka⼏处坑(了解)

4.1 Eureka的⼏处缓存

Eureka的wiki上有⼀句话，⼤意是⼀个服务启动后最⻓可能需要2分钟时间才能被其它服务

感知到，但是⽂档并没有解释为什么会有这2分钟。其实这是由三处缓存 + ⼀处延迟造成

的。

⾸先，Eureka对HTTP响应做了缓存。在Eureka的”控制器”类 ApplicationResource 的

109⾏可以看到有⼀⾏

String payLoad = responseCache.get(cacheKey);11的调⽤，该代码所在的 getApplication() ⽅法的功能是响应客户端查询某个服务信息

的HTTP请求：

String payLoad = responseCache.get(cacheKey); // 从cache中拿响应数据

if (payLoad != null) {

logger.debug("Found: {}", appName);

return Response.ok(payLoad).build();

} else {

logger.debug("Not Found: {}", appName);

return Response.status(Status.NOT_FOUND).build();

}

上⾯的代码中，responseCache引⽤的是ResponseCache类型，该类型是⼀个接⼝，其get()

⽅法⾸先会去缓存中查询数据，如果没有则⽣成数据返回（即真正去查询注册列表），且

缓存的有效时间为30s。也就是说，客户端拿到Eureka的响应并不⼀定是即时的，⼤部分时

候只是缓存信息。

其次，Eureka Client对已经获取到的注册信息也做了30s缓存。即服务通过eureka客户端第

⼀次查询到可⽤服务地址后会将结果缓存，下次再调⽤时就不会真正向Eureka发起HTTP请

求了。

再次， 负载均衡组件Ribbon也有30s缓存。Ribbon会从上⾯提到的Eureka Client获取服务

列表，然后将结果缓存30s。

最后，如果你并不是在Spring Cloud环境下使⽤这些组件(Eureka, Ribbon)，你的服务启动

后并不会⻢上向Eureka注册，⽽是需要等到第⼀次发送⼼跳请求时才会注册。⼼跳请求的

发送间隔也是30s。（Spring Cloud对此做了修改，服务启动后会⻢上注册）

以上这四个30秒正是官⽅wiki上写服务注册最⻓需要2分钟的原因。

4.2服务注册信息不会被⼆次传播

如果在集群环境下Eureka A的peer指向了B, B的peer指向了C，那么当服务向Eureka A注册

时，Eureka B中会有该服务的注册信息，但是Eureka C中没有。也就是说，如果你希望只

要向⼀台Eureka注册然后其它所有Eureka实例都能得到注册信息，那么就必须把其它所有

节点都配置到当前Eureka的 peer 属性中。这⼀逻辑是

在 PeerAwareInstanceRegistryImpl#replicateToPeers() ⽅法中实现的：

private void replicateToPeers(Action action, String appName, String id,

InstanceInfo info /* optional */,

InstanceStatus newStatus /* optional

*/, boolean isReplication) {

Stopwatch tracer = action.getTimer().start();

try {if (isReplication) {

numberOfReplicationsLastMin.increment();

 }

// 如果这条注册信息是其它Eureka同步过的则不会再继续传播给⾃⼰的peer节 

点

if (peerEurekaNodes == Collections.EMPTY_LIST ||

isReplication) {

return;

 }

for (final PeerEurekaNode node :

peerEurekaNodes.getPeerEurekaNodes()) {

// 不要向⾃⼰发同步请求

if (peerEurekaNodes.isThisMyUrl(node.getServiceUrl())) {

continue;

 }

replicateInstanceActionsToPeers(action, appName, id,

info, newStatus, node);

 }

 } finally {

tracer.stop();

 }

 }

4.3多⽹卡环境下的IP选择问题

如果机器上(客户端 服务端)安装了多块⽹卡，它们分别对应IP地址A, B, C，此时： Eureka

会选择IP合法(标准ipv4地址)、索引值最⼩(eth0, eth1中eth0优先)且不在忽略列表中(可 

在 application.properites 中配置忽略哪些⽹卡)的⽹卡地址作为服务IP。

我们的服务在注册到Eureka的时候,在Eureka管理⻚⾯可能会发现,服务的ip并不是我们真正

的外部ip,这种情况⼀般发⽣在电脑上⾯装了虚拟机的时候产⽣了多个⽹卡

这个坑的详细分析⻅：http://blog.csdn.net/neosmith/article/details/53126924

三 Eureka 集群

Eureka为了保证⾼可⽤,需要搭建集群,Eureka的集群搭建很简单,只要eureka服务端之

间相互注册就⾏了,每个Eureka作为其他的Eureka的客户端,配置⽅式和我们配置普通

服务没区别#第⼀个Eureka的配置⽂件

spring:

 application:

 name: EUREKA-HA

server:

 port: 8761

eureka:

 instance:

 hostname: localhost

 client:

 serviceUrl:

 defaultZone:

http://localhost:8762/eureka/,http://localhost:8763/eureka/ ##配置另外两个

的eureka的地址,主要我们之前单机的时候eureka指定为单机版,此处需要删除

\#第⼆个的配置⽂件

server:

 port: 8762

spring:

 application:

 name: EUREKA-HA

eureka:

 instance:

 hostname: localhost

 client:

 serviceUrl:

 defaultZone:

http://localhost:8761/eureka/,http://localhost:8763/eureka/ ##配置另外两个

的eureka的地址,主要我们之前单机的时候eureka指定为单机版,此处需要删除

\#第三个eureka的配置⽂件

server:

 port: 8763

spring:

 application:

 name: EUREKA-HA

eureka:

 instance:

 hostname: localhost

 client:

 serviceUrl: defaultZone:

http://localhost:8761/eureka/,http://localhost:8762/eureka/ ##配置另外两个

的eureka的地址,主要我们之前单机的时候eureka指定为单机版,此处需要删除