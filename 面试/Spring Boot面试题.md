# 基础知识篇

## Q1. 什么是Spring Boot并说明它的必要性？

​	Spring Boot 是 Spring 的一个模块，目的是要简化 Java 开发中 Spring 框架的使用。它可以在 Spring 的基础上创建独立的、可直接运行的应用程序。因此，它删除了许多配置和依赖项。针对快 速的应用程序开发，Spring Boot 框架提供了自动依赖解决方案、嵌入式 HTTP 服务器、自动配置、 管理端点和 Spring Boot CLI。 由此可见，Spring Boot 不仅提高了工作效率，而且为编写自己的业 务逻辑提供了很多便利。 

## Q2. Spring Boot和Spring的区别是什么？

基于 Java Web 的应用程序框架 

Spring 模块  提供可自定义的 Web 应用程序工具和库 用于创建一个可执行的 Spring 应用程序项目 

Spring Boot 没有 Spring 框架复杂 

## Q3. Spring Boot有什么优点？

提供自动加载一组默认配置的功能来快速启动应用程序 
对大型项目类中常见的一些列肺功能特性的独立应用程序 
内置 Tomcat、Servlet 容器和 Jetty，避免使用 WAR 文件 
Spring Boot 提供了视图功能，减少了开发人员的工作量，简化了 Maven 配置 
提供 CLI 工具来开发和测试应用程序 
附带 Spring 启动器，以确保依赖项管理，并提供各种安全指标 
包含广泛的 API，用于监控和管理开发和 prod 中的应用程序。 
与 Spring JDBC、Spring ORM、Spring Data、Spring Security 等 Spring 生态系统集成， 避免样板代码。 

## Q4. Srping Boot的特性

1. **Spring CLI**：允许使用 Groovy 编写 Spring 引导应用程序，并避免了样板代码。 
2. **启动依赖项**：在此功能的帮助下，Spring Boot 将公共依赖项聚集在一起，最终提高了工作 
效率。 
3. **Spring Initializer**：这基本上是一个 Web 应用程序，它可以自动创建一个内部项目结构，不 
必再手动设置。 
4. **自动配置**：Spring Boot 的自动配置特性可以为正在处理的项目加载默认配置。这正方式可 
以避免不必要的 WAR 文件。 
5. **Spring** **驱动器**：这一功能为运行 Spring Boot 应用程序提供帮助。 
6. **日志和安全性**：Spring Boot 的日志和安全性特性，确保所有使用 Spring Boot 的应用程序 
都得到了适当的保护，没有任何麻烦。 

## Q5.什么是Spring Boot，什么是可用的启动器？

Spring 启动程序是一组方便的依赖项管理提供程序，可以在应用程序中使用它们来启用依赖项。这 
些启动器使开发变得简单和快速。所有可用的启动程序都在 org.springframework 下。引导组。以 
下是一些流行的开胃菜： 
spring-boot-starter：这是核心 starter，包括日志记录、自动配置支持和 YAML。 
spring-boot-starter-jdbc：该启动程序用于与 JDBC 的 HikariCP 连接池 
spring-boot-starter-web：是使用 Spring MVC 构建 Web 应用程序（包括 RESTful 应用程 序）的起点 
spring-boot-starter-Dat-JPA：是在 Hibernate 中使用 Spring Data JPA 的第一步 
spring-boot-starter-Security：是用于 Spring Security 的启动器 
spring-boot-starter-AOP：这个 starter 用于使用 AspectJ 和 Spring AOP 进行面向方面的 编程 
spring-boot -starter-test：用于测试 Spring 引导应用程序 

## Q6.什么是Spring Boot依赖项管理？

Spring Boot 依赖项管理主要用于自动管理依赖项和配置，而不需要为任何依赖项指定版本。 

## Q7.提到JPA和Hibernate之间的区别

JPA 是一个数据访问抽象，用于减少样板代 码的数量 
Hibernate 是 Java Persistence API 的一个实现，提 供了松散耦合

## Q8. RequestMapping和GetMapping的区别是什么？

	@GetMapping 是一个复合注释，它充当 @RequestMapping(method = RequestMethod.GET) 的快 捷方式。这两种方法都支持。
consume 选项有： 
	consume = " text / plain 
	consume = {" text/plain "， " application/* "} 

## Q9. Spring Boot的核心注解是哪个？它主要由哪几个注解组成的？

@SpringBootApplication 是启动类上面的注解，同时也是 Spring Boot 的核心注解，它主要组合包 

含了以下 3 个注解： 

@SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。 
@EnableAutoConfiguration：打开 或关闭自动配 置，如关闭 数据源自动 配置功能： 
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。 
@ComponentScan：Spring 组件扫描。 

## Q10.由Spring Boot配置的默认H2数据库的名称是什么？

默认的 H2 数据库的名称是 testdb。请参考下面： 

spring.datasource.name=testdb #数据源的名称 

注意：为以防万一，您可以使用 Spring Boot 的名称来命名正在使用的 H2 内存数据库。 

## Q11.解释Spring Actuator及其优点

Spring Actuator 是 Spring Boot 的一个很酷的特性，通过它可以看到正在运行的应用程序中发生了 什么。每当你想调试你的应用程序，并需要分析日志时你需要了解在应用程序中发生了什么。在这种 情况下，Spring Actuator 提供了对诸如标识 Bean、CPU 使用情况等特性的简单访问。Spring Actuator 提供了一种非常简单的方法来访问准备生产的 REST 点并从 Web 获取各种信息。使用 Spring Security 的内容协商策略保护这些点。

## Q12. @SpringBootApplication和@EnableAutoConfiguration注释有什么区别？

@SpringBootApplication 在主类或引导类中使用 它 是 
@Configuration 、 @ComponentScan 和 @EnableAutoConfiguration 注释的组合。 

# 应用实战篇

## Q13.解释如何使用Maven创建Spring启动应用程序

使用 Maven 来创建 Spring Boot 应用程序的方法有很多，下面列出其中的一部分： 
Spring Boot CLI 
Spring Starter 项目向导 
Spring Initializr 
Spring Maven 项目 

## Q14.说明外部配置的可能来源

Spring Boot 通过提供外部配置支持可以允许开发人员在不同的环境中运行相同的应用程序。它通过 
使用环境变量、属性文件、命令行参数、YAML 文件和系统属性来说明所需的配置属性。另外，@value 注释可以获得对属性的访问。由此可见，外部配置最可能的来源如下： 
	应用程序属性：默认情况下，Spring Boot 在当前目录、类路径、根目录或配置目录中搜索 应用程序属性文件或其 YAML 文件以加载属性。 
	命令行属性：Spring Boot 提供命令行参数，并将这些参数转换为属性，然后将它们添加到 环境属性集。 
	特定于概要文件的属性：这些属性是从应用程序中加载的 -{概要文件}。属性文件或其 YAML 
文件。此文件与非特定属性文件位于相同的位置，{profile} 占位符指的是活动的概要文件。

## Q15.您能解释一下当Spring启动应用程序“作为Java应用程序运行”时在后台会发生什么吗？

当 Spring 启动应用程序以“作为 Java 应用程序运行”的方式执行时，当它看到您正在开发 Web 应 用程序时，就会自动启动 Tomcat 服务器。 当 Spring Boot 应用程序以 Java 应用程序”的方式执行的时候，就会启动 Tomcat 服务器。 

## 16. Spring Boot系统的最低要求是什么？

Spring boot 2.1.7.RELEASE 需要 
Java 8 + 
Spring Framework 5.1.9 + 
明确的支持 
Maven 3.3 + 
Gradle 4.4 + 
Servlet 容器支持 
Tomcat 9.0 - Servlet 版本 4.0 
Jetty 9.4 - Servlet 版本 3.1 
Undertow 2.0 - Servlet 版本 4.0 

## Q17.解释什么是thymeleaf以及如何使用thymeleaf？

Thymeleaf 是一个服务器端 Java 模板引擎用于 Web 应用程序。它的目标是为您的 Web 应用程 
序带来自然的模板，并且能够很好地与 Spring 框架和 HTML5 Java Web 应用程序集成。要使用 
Thymeleaf，您需要在 pom.xml 文件中添加以下代码：


## Q18.我们可以在Spring Boot中更改嵌入式Tomcat服务器的端口吗？

是的，我们可以使用应用程序属性文件更改嵌入式 Tomcat 服务器的端口。在这个文件中，必须添 
加“server”属性。并将其分配到您希望的任何端口。例如，如果希望将其分配给 8081，则必须提 
到 server.port=8081。一旦提到端口号，应用程序属性文件将在 Spring 引导时自动加载，所需的 
配置将应用于应用程序。 

## Q19.如何在Spring Boot中启用HTTP/2支持

您可以在 Spring Boot by 中启用 HTTP/2 支持： `server.http2.enabled=true`

## Q20.如何在Spring Boot执行器中创建自定义端点？

在 Spring Boot 2.x 中创建自定义端点可以使用 @Endpoint 注释。Spring Boot 还在 Spring MVC、 
Jersey 等的帮助下，通过 HTTP 使用 @WebEndpointor、@WebEndpointExtension 来公开端点。 

## Q21.将Spring引导Web应用程序部署为JAR和WAR文件的步骤是什么？

要部署 Spring Boot Web 应用程序，只需在 pom.xml 文件中添加以下插件： 

通过使用上面的插件，您将得到一个执行包阶段的 JAR。这个 JAR 将包含所有必需的库和依赖项。 
它还包含一个嵌入式服务器。因此，您基本上可以像普通 JAR 文件一样运行应用程序。 

注意：构建一个 JAR 文件，需要在 pom.xml 中设置：<**packaging**>jar</**packaging**> 
类似地，如果您想要构建一个 WAR 文件，那么需要在 pom.xml 中设置： 
<**packaging**>war</**packaging**> 

## Q22.您能解释一下如何使用Spring Boot部署到不同的服务器吗？

要使用 Spring Boot 部署不同的服务器，请遵循以下步骤： 
从项目中生成 WAR 包 
然后，将 WAR 文件部署到您喜欢的服务器上 

注意：在服务器上部署 WAR 文件的步骤取决于您选择的服务器。 

## Q23.在Spring Boot中添加自定义JS代码的步骤是什么？

使用 Spring Boot 添加自定义 JS 代码的步骤如下： 
在 resources 文件夹下创建一个文件夹，并将其命名为 static 
可以将静态内容放入该文件夹 

注意：以防浏览器抛出未经授权的错误，您可以调整安全策略或在日志文件中搜索密码，并放在在请 求头中。 

## Q24.使用Spring Boot公开自定义应用程序配置的最佳方式是什么？** 

在 Spring Boot 中公开自定义应用程序配置的一种方法是使用 @Value 注释。但是，这个注释的惟 一问题是所有的配置值都分布在整个应用程序中。相反，您可以使用集中式方法。 
通过集中式方法，我的意思是您可以使用 @ConfigurationProperties 定义配置组件，如下所示：根据上面的代码片段，应用程序中配置的值。属性如下： 

##  25.连接外部数据库（如MySQ或Oracle）的步骤是什么？

要连接外部数据库，您必须遵循以下步骤： 

首先，将 MySQL 连接器的依赖项添加到 pom.xml 
然后，从 pom.xml 中删除 H2 依赖项 
第三，设置 MySQL 数据库并配置到 MySQL 数据库的连接 
最后，重启项目 

# 理论实现篇

## Q26. Spring Boot DevTools需要什么？

Spring Boot 开发工具是一组精心设计的工具，旨在简化应用程序的开发过程。如果应用程序在生产 环境中运行，则此模块将自动禁用，缺省情况下也不包括重新打包存档。因此，Spring Boot 开发人 
员工具将属性应用于相应的开发环境要包含 DevTools，您只需将以下依赖项添加到 pom.xml 文件 中：

## Q27.介绍使用Spring Initializr创建Spring启动项目的步骤

Spring Initializr 是 Spring 提供的一个 Web 工具。在这个工具的帮助下，您可以通过提供项目细节 
来创建 Spring Boot 项目。使用 Spring Initializr 创建 Spring 启动项目需要遵循以下步骤： 
选择 Maven 项目和所需的依赖项。然后填写其他需要的细节，如组、工件，然后单击 
Generate Project。 
下载项目后，将项目解压缩到您的系统中。 
接下来，您必须使用 Spring 工具套件 IDE 上的 import 选项导入这个项目。 
在导入项目时，请记住，您必须选择项目类型为 Maven，源项目应该包含 pom.xml 文件。 
一旦执行了上述所有步骤，您将看到 Spring Boot 项目是使用所有必需的依赖项创建的。 

## Q28.介绍使用JDBC将Spring启动应用程序连接到数据库的步骤 

Spring Boot starter 项目提供了将应用程序与 JDBC 连接所需的库。因此，例如，如果你只需要创 
建一个应用程序，并将其与 MySQL 数据库连接，你可以遵循以下步骤。 
STEP 1：创建 MySQL 数据库： **CREATE DATABASE** spring_boot; 
STEP 2：然后必须在这个数据库中创建一个表。 
STEP 3：现在，创建一个 Spring Boot 项目并提供所需的详细信息 
STEP 4：添加 JDBC、MySQL 和 Web 依赖项。 
STEP 5：创建项目之后，必须将数据库配置为应用程序属性： 
STEP 6：Java 类应该有以下代码：STEP 7：接下来，你必须创建一个控制器来处理 HTTP 请求，通过以下代码： 
STEP 8：最后，将此项目作为 Java 应用程序执行。 
STEP 9：接下来，打开 URL（localhost:8080/insert），您将看到数据输入成功的输出。您还可以进 

一步检查是否将数据输入到表中。 

##  Q29.@RequestMapping和@RestController注释的用途是什么？

**@RequestMapping** 

**@RestController** 

该注释用于提供路由信息，并告诉 Spring 任何 

HTTP 请求都必须映射到相应的方法。 

此注释用于将 @ResponseBody 和 

@Controller 注释添加到类中 要使用这个注释，您必须导入**@RequestMapping** 

**@RestController** 

org.springframework.web.bind.annotation.Reque stMapping; 

org.springframework.web.bind.annotation.Res tController; 

示例：假设您有一个方法 example()，它应该映射 /example 的 URL。 

## Q30.什么是Spring Boot CLI，如何使用Boot CLI执行Spring Boot项目？

Spring Boot CLI 是官方支持 Spring 框的工具。执行 Spring Boot 项目的步骤如下： 

从官方网站下载 CLI 工具并解压，Spring 启动程序在其设置中的 bin 文件夹内。 

由于 Spring Boot CLI 执行 Groovy 文件，因此需要为 Spring Boot 应用程序创建一个 Groovy 文件。为此，打开 terminal 并进入 bin 文件夹目录，打开一个 Groovy 文件（例 如 Sample.groovy），在这个文件中创建一个控制器如下： 

然后通过以下方式执行 Groovy 文件： 

./spring run Sample.groovy; 

一旦项目被执行，进入 URL（localhost:8080:/example），你就会看到结果。**Q31. Spring** **解释数据** 

Spring Data 的目标是使开发人员能够轻松地使用关系和非关系数据库、基于云的数据服务和其他数 

据访问技术。因此，基本上，它使数据访问变得容易，并且仍然保留底层数据。 

## Q32. 您对Spring Boot中的自动配置有什么理解？如何禁用 自动配置？

Auto-configuration 用于自动配置应用程序所需的配置。例如，如果在应用程序的类路径中有一个数 据源 Bean，那么它会自动配置 JDBC 模板。在自动配置的帮助下，您可以轻松地创建 Java 应用 程序，因为它可以自动配置所需的 Bean、控制器等。 

要禁用自动配置属性，您必须排除 @EnableAutoConfiguration 属性，在不希望应用该属性的场景中。 

@**EnableAutoConfiguration**(**exclude**={**DataSourceAutoConfiguration**.class} 

如果类不在类路径中，那么要排除自动配置，你必须提到以下代码： 

@**EnableAutoConfiguration**(**excludeName**={**Sample**.class}) 

除此之外，Spring Boot 还提供了通过使用 Spring .autoconfigure.exclude 属性来排除自动配置类列 表的功能。您可以继续，并将其添加到应用程序中，属性或添加以逗号分隔的多个类。 

## Q33.你能举一个事务管理中ReadOnly为true的例子吗？

事务管理中 ReadOnly 为 true 的例子如下： 

假设您必须从数据库中读取数据。例如，假设您有一个客户数据库，希望读取客户详细信息，如 customerID 和 customername。如果不想检查实体中的更改，需要将事务设置为只读。 

## 34.我们可以在Spring Boot 中创建一个非Web应用程序吗？

是的，我们可以通过从类路径中删除 Web 依赖项以及改变 Spring Boot 创建应用程序上下文的方式 来创建非 Web 应用程序。

## Q35.叙述YAML文件比属性文件的优优势在哪里，以及在Spring Boot中加载YAML文件的不同方式

YAML 文件比属性文件的优势在于数据以分层格式存储。如果出现问题，开发人员很容易进行调试。 
当您在类路径上使用 SnakeYAML 库时，SpringApplication 类支持 YAML 文件作为属性的替代。 
在 Spring Boot 中加载 YAML 文件的不同方法如下： 

使用 YamlMapFactoryBean 加载 YAML 作为地图 
使用 YamlPropertiesFactoryBean 加载 YAML 作为属性 

## Q36. 如何在不进行任何配置的情况下选择Hibernate 作为JPA 的默认实现？

当我们使用 Spring Boot 默认配置时，Spring-Boot-starter-data-jpa 依赖项会自动添加到 pom.xml 文件中。现在，由于这个依赖项对 JPA 和 Hibernate 有一个可传递的依赖项，Spring Boot 会自动 将 Hibernate 配置为 JPA 的默认实现，只要它在类路径中看到 Hibernate。 

## Q37. 您对Spring Date REST的理解是什么？ 

Spring Data REST 用于公开 Spring 数据库周围的 RESTful 资源。例如： 

现在，为了公开 REST 服务，您可以 POST 一下的信息： 

{"customername": "Rohit"} 

响应内容：注意，响应内容包含新创建的资源的 href。 

## Q38.事务的边界应该从哪一层开始？

事务的边界应该从服务层开始，因为业务的事务逻辑存在于该层本身。 

## Q39. path = "sample"，collectionResourceRel ="sample"如何使用Spring Data Rest？

@RepositoryRestResource(collectionResourceRel = "sample", path = "sample")**public interface** 

**SampleRepository** **extends****PagingAndSortingRepository**<**Sample**, **Long**> 

path：用来配置资源导出的路径。 

collectionResourceRel：此值用于生成集合资源的链接。 

## Q40.如何配置Log4j进行日志记录？

Spring Boot 支持 Log4j2 配置，排除 Logback 并包含 Log4j2 来为日志做记录。但这种方式只有 

在使用 starter 项目时才可以。 

## Q41.提到WAR和嵌入式容器之间的区别

**WAR** 

**嵌入式容器** 

WAR 在很大程度上得益于 Spring Boot Spring Boot 只有一个组件，在改进期间使用
## Q42.你认为对个人资料的需求是什么？

配置文件用于提供一种方法来隔离应用程序配置的不同部分，并使其可用于各种环境。任何 
@Component 或 @Configuration 都可以标记为 @Profile 来限制它的加载。假设你有多个环境 
Dev 
QA 
Stage 
Production 

您希望在每个环境中都有不同的应用程序配置，您可以使用配置文件为不同的环境提供不同的应用程 序配置。Spring 和 Spring Boot 提供了一些特性，您可以通过这些特性指定： 

特定环境的活动概要 
各种配置文件的各种环境配置。 

## Q43.当Bean存在时，如何指示自动配置退出？

要指示一个自动配置类在 Bean 存在时退出，您必须使用 @ConditionalOnMissingBean 注释。该注 
释的属性如下： 
value：该属性存储要检查的 Bean 的类型 
name：该属性存储要检查的 Bean 的名称 

## Q44.为什么在实际应用程序中不推荐使用Spring DataREST？

在实际应用程序中不建议使用 Spring Data REST，在数据库实体直接公开为 REST 服务，设计 RESTful 服务时，我们考虑的两个最重要的事情是域模型和使用者。但是，在使用 Spring Data REST 时，这些参数都没有考虑，实体是直接暴露的。因此可以使用 Spring Data REST 来进行项目的初始 演化。 

## Q45.如果H2不在类路径中，会出现什么错误？

如果类路径中没有 H2，则会出现无法为数据库类型 NONE 确定嵌入式数据库驱动程序类的情况。 要解决此错误，请将 H2 的依赖添加到 pom.xml 文件中，然后重新启动服务器。 
依赖内容如下：
## Q46.使用profiles配置特定环境的方式是什么？

Profiles 是识别环境的关键，这是众所周知的事实，让我们考虑以下两个概要文件的例子： 
dev 
prod 

考虑应用程序属性文件中存在的以下属性： 
如果您想要自定义应用程序属性，然后创建一个名为 application-dev 的文件，其中属性为要自定义 的属性。你可以提到以下代码： 
example.message: Dynamic Message **in** Dev 
类似地，如果希望自定义应用程序。prod 配置文件的属性，然后您可以提到以下代码片段： 
example.message: Dynamic Message **in** Prod 
完成特定概要文件的配置后，必须在环境中设置活动概要文件。要做到这一点，你可以 
设置参数 -Dspring.profiles.active = prod 
在程序里设置属性文件 spring.profiles.active=prod 

## Q47. Spring Boot可以通过依赖项启动JPA应用程序并连接到内存数据库H2吗？

可以，方法如下： 
Web starter h2 data JPA starter 
要包含依赖项，请参考以下代码： 

## Q48.您对Spring Boot支持轻松绑定的理解是什么？

轻松绑定，是一种属性名称不需要与环境属性的键匹配的方式。在 Spring Boot，轻松绑定适用于配 
置属性的类型安全绑定。例如，如果使用了带有 @ConfigurationPropertie 注释的 Bean 类中的属 性 sampleProp，那么它可以绑定到以下任何环境属性： 

sampleProp 
sample-Prop 
sample_Prop 
SAMPLE_PROP 

## Q49.指定的数据库连接信息在哪里？它如何自动连接到H2？

这个问题的答案很简单。正是由于 Spring Boot 自动配置配置了应用程序的依赖项。因此，数据库连 接信息和数据库到 H2 的自动连接是通过 auto-configuration （自动配置）属性完成的。

## 50.您认为在spring-boot-starter-web中可以使用Jetty而不是Tomcat吗？

是的，我们可以在 spring-boot-starter-web 中使用 Jetty 而不是 Tomcat，方法是删除现有的依赖 
项并添加以下内容：

```xml
          <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
```


# 面试题10

## 1、Spring Boot 的自动配置是如何实现的？

Spring Boot 项目的启动注解是：@SpringBootApplication，其实它就是由下面三个注解组成的：

· @Configuration

· @ComponentScan

· @EnableAutoConfiguration

其中 @EnableAutoConfiguration 是实现自动配置的入口，该注解又通过 @Import 注解导入了AutoConfigurationImportSelector，在该类中加载 META-INF/spring.factories 的配置信息。然后筛选出以 EnableAutoConfiguration 为 key 的数据，加载到 IOC 容器中，实现自动配置功能！

## 2、什么是嵌入式服务器？为什么要使用嵌入式服务器呢?

思考一下在你的虚拟机上部署应用程序需要些什么。

第一步：安装 Java

第二部：安装 Web 或者是应用程序的服务器（Tomat/Wbesphere/Weblogic 等等）

第三部：部署应用程序 war 包

如果我们想简化这些步骤，应该如何做呢？

让我们来思考如何使服务器成为应用程序的一部分？

你只需要一个安装了 Java 的虚拟机，就可以直接在上面部署应用程序了，

是不是很爽？

这个想法是嵌入式服务器的起源。

当我们创建一个可以部署的应用程序的时候，我们将会把服务器（例如，tomcat）嵌入到可部署的服务器中。

例如，对于一个 Spring Boot 应用程序来说，你可以生成一个包含 Embedded Tomcat 的应用程序 jar。你就可以像运行正常 Java 应用程序一样来运行 web 应用程序了。

嵌入式服务器就是我们的可执行单元包含服务器的二进制文件（例如，tomcat.jar）。

## 3、微服务同时调用多个接口，怎么支持事务的啊？

支持分布式事务，可以使用Spring Boot集成 Aatomikos来解决，但是我一般不建议这样使用，因为使用分布式事务会增加请求的响应时间，影响系统的TPS。一般在实际工作中，会利用消息的补偿机制来处理分布式的事务。

## 4、shiro和oauth还有CAS他们之间的关系是什么？问下您公司权限是如何设计，还有就是这几个概念的区别。

cas和oauth是一个解决单点登录的组件，shiro主要是负责权限安全方面的工作，所以功能点不一致。但往往需要单点登陆和权限控制一起来使用，所以就有 cas+shiro或者oauth+shiro这样的组合。

token一般是客户端登录后服务端生成的令牌，每次访问服务端会进行校验，一般保存到内存即可，也可以放到其他介质；redis可以做Session共享，如果前端web服务器有几台负载，但是需要保持用户登录的状态，这场景使用比较常见。

我们公司使用oauth+shiro这样的方式来做后台权限的管理，oauth负责多后台统一登录认证，shiro负责给登录用户赋予不同的访问权限。

## 5、服务之间通信Restful和Rpc这2种方式如何做选择？

在传统的SOA治理中，使用rpc的居多；Spring Cloud默认使用restful进行服务之间的通讯。rpc通讯效率会比restful要高一些，但是对于大多数公司来讲，这点效率影响甚微。我建议使用restful这种方式，易于在不同语言实现的服务之间通讯。

## 6、怎么设计无状态服务？

对于无状态服务，首先说一下什么是状态：如果一个数据需要被多个服务共享，才能完成一笔交易，那么这个数据被称为状态。进而依赖这个“状态”数据的服务被称为有状态服务，反之称为无状态服务。

那么这个无状态服务原则并不是说在微服务架构里就不允许存在状态，表达的真实意思是要把有状态的业务服务改变为无状态的计算类服务，那么状态数据也就相应的迁移到对应的“有状态数据服务”中。

场景说明：例如我们以前在本地内存中建立的数据缓存、Session缓存，到现在的微服务架构中就应该把这些数据迁移到分布式缓存中存储，让业务服务变成一个无状态的计算节点。迁移后，就可以做到按需动态伸缩，微服务应用在运行时动态增删节点，就不再需要考虑缓存数据如何同步的问题。

## 7、Spring Cache 三种常用的缓存注解和意义？

@Cacheable ，用来声明方法是可缓存，将结果存储到缓存中以便后续使用相同参数调用时不需执行实际的方法，直接从缓存中取值。

@CachePut，使用 @CachePut 标注的方法在执行前，不会去检查缓存中是否存在之前执行过的结果，而是每次都会执行该方法，并将执行结果以键值对的形式存入指定的缓存中。

@CacheEvict，是用来标注在需要清除缓存元素的方法或类上的，当标记在一个类上时表示其中所有的方法的执行都会触发缓存的清除操作。

## 8、Spring Boot 如何设置支持跨域请求？

现代浏览器出于安全的考虑， HTTP 请求时必须遵守同源策略，否则就是跨域的 HTTP 请求，默认情况下是被禁止的，IP（域名）不同、或者端口不同、协议不同（比如 HTTP、HTTPS）都会造成跨域问题。

一般前端的解决方案有：

· ① 使用 JSONP 来支持跨域的请求，JSONP 实现跨域请求的原理简单的说，就是动态创建<script>标签，然后利用<script>的 SRC 不受同源策略约束来跨域获取数据。缺点是需要后端配合输出特定的返回信息。

· ② 利用反向代理的机制来解决跨域的问题，前端请求的时候先将请求发送到同源地址的后端，通过后端请求转发来避免跨域的访问。

后来 HTML5 支持了 CORS 协议。CORS 是一个 W3C 标准，全称是”跨域资源共享”（Cross-origin resource sharing），允许浏览器向跨源服务器，发出 XMLHttpRequest 请求，从而克服了 AJAX 只能同源使用的限制。它通过服务器增加一个特殊的 Header[Access-Control-Allow-Origin]来告诉客户端跨域的限制，如果浏览器支持 CORS、并且判断 Origin 通过的话，就会允许 XMLHttpRequest 发起跨域请求。

前端使用了 CORS 协议，就需要后端设置支持非同源的请求，Spring Boot 设置支持非同源的请求有两种方式。

第一，配置 CorsFilter。

```java
@Configuration
public class GlobalCorsConfig {

  @Bean
  public CorsFilter corsFilter() {
     CorsConfiguration config = new CorsConfiguration();
     config.addAllowedOrigin("*");
     config.setAllowCredentials(true);
     config.addAllowedMethod("*");
     config.addAllowedHeader("*");
     config.addExposedHeader("*");
     UrlBasedCorsConfigurationSource configSource = new UrlBasedCorsConfigurationSource();
    configSource.registerCorsConfiguration("/**", config)
    return new CorsFilter(configSource);
  }
}
```

需要配置上述的一段代码。第二种方式稍微简单一些。

第二，在启动类上添加：

```java
public class Application extends WebMvcConfigurerAdapter {  

  @Override  
  public void addCorsMappings(CorsRegistry registry) {  
    registry.addMapping("/**")  
        .allowCredentials(true)  
        .allowedHeaders("*")  
        .allowedOrigins("*")  
        .allowedMethods("*");  
  }  
}  
```

## 9、JPA 和 Hibernate 区别？JPA 可以支持动态 SQL 吗？

JPA本身是一种规范，它的本质是一种ORM规范（不是ORM框架，因为JPA并未提供ORM实现，只是制定了规范）因为JPA是一种规范，所以，只是提供了一些相关的接口，但是接口并不能直接使用，JPA底层需要某种JPA实现，Hibernate 是 JPA 的一个实现集。

JPA 是根据实体类的注解来创建对应的表和字段，如果需要动态创建表或者字段，需要动态构建对应的实体类，再重新调用Jpa刷新整个Entity。动态SQL，mybatis支持的最好，jpa也可以支持，但是没有Mybatis那么灵活。

## 10、Spring 、Spring Boot 和 Spring Cloud 的关系?

Spring 最初最核心的两大核心功能 Spring Ioc 和 Spring Aop 成就了 Spring，Spring 在这两大核心的功能上不断的发展，才有了 Spring 事务、Spring Mvc 等一系列伟大的产品，最终成就了 Spring 帝国，到了后期 Spring 几乎可以解决企业开发中的所有问题。

Spring Boot 是在强大的 Spring 帝国生态基础上面发展而来，发明 Spring Boot 不是为了取代 Spring ,是为了让人们更容易的使用 Spring 。

Spring Cloud 是一系列框架的有序集合。它利用 Spring Boot 的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用 Spring Boot 的开发风格做到一键启动和部署。

Spring Cloud 是为了解决微服务架构中服务治理而提供的一系列功能的开发框架，并且 Spring Cloud 是完全基于 Spring Boot 而开发，Spring Cloud 利用 Spring Boot 特性整合了开源行业中优秀的组件，整体对外提供了一套在微服务架构中服务治理的解决方案。

用一组不太合理的包含关系来表达它们之间的关系。

Spring ioc/aop > Spring > Spring Boot > Spring Cloud

# 面试题22

## 问题一 什么是 Spring Boot？

​	多年来，随着新功能的增加，spring 变得越来越复杂。只需访问 https://spring.io/projects 页面，我们就会看到可以在我们的应用程序中使用的所有 Spring 项目的不同功能。如果必须启动一个新的 Spring 项目，我们必须添加构建路径或添加 Maven 依赖关系，配置应用程序服务器，添加 spring 配置。因此，开始一个新的 spring 项目需要很多努力，因为我们现在必须从头开始做所有事情。

	Spring Boot 是解决这个问题的方法。Spring Boot 已经建立在现有 spring 框架之上。使用spring 启动，我们避免了之前我们必须做的所有样板代码和配置。因此，Spring Boot 可以帮助我们以最少的工作量，更加健壮地使用现有的 Spring 功能。

## 问题二 Spring Boot 有哪些优点？

	减少开发，测试时间和努力。
	使用 JavaConfig 有助于避免使用 XML。
	避免大量的 Maven 导入和各种版本冲突。
	通过提供默认值快速开始开发。
	没有单独的 Web 服务器需要。这意味着你不再需要启动 Tomcat，Glassfish 或其他任何东西。
	需要更少的配置 因为没有 web.xml 文件。只需添加用@ Configuration 注释的类，然后添加用@Bean 注释的方法，Spring 将自动加载对象并像以前一样对其进行管理。您甚至可以将@Autowired 添加到 bean 方法中，以使 Spring 自动装入需要的依赖关系中。
	基于环境的配置 
		使用这些属性，您可以将您正在使用的环境传递到应用程序：-
		Dspring.profiles.active = {enviornment}。
		在加载主应用程序属性文件后，Spring 将在（application{environment} .properties）中加载后续的应用程序属性文件。

## 问题三 什么是 JavaConfig？

	Spring JavaConfig 是 Spring 社区的产品，它提供了配置 Spring IoC 容器的纯 Java 方法。因此
它有助于避免使用 XML 配置。使用 JavaConfig 的优点在于：面向对象的配置。由于配置被定义为 JavaConfig 中的类，因此用户可以充分利用 Java 中的面向对象功能。一个配置类可以继承另一个，重写它的@Bean 方法等。减少或消除 XML 配置。基于依赖注入原则的外化配置的好处已被证明。但是，许多开发人员不希望在 XML 和 Java 之间来回切换。JavaConfig 为开发人员提供了一种纯 Java 方法来配置与 XML 配置概念相似的 Spring 容器。从技术角度来讲，只使用 JavaConfig 配置类来配置容器是可行的，但实际上很多人认为将 JavaConfig 与 XML 混合匹配是理想的。类型安全和重构友好。JavaConfig 提供了一种类型安全的方法来配置 Spring 容器。由于Java 5.0 对泛型的支持，现在可以按类型而不是按名称检索 bean，不需要任何强制转换或基于字符串的查找。

## 问题四 如何重新加载 Spring Boot 上的更改，而无需重新启动服务器？

	这可以使用 DEV 工具来实现。通过这种依赖关系，您可以节省任何更改，嵌入式 tomcat将重新启动。Spring Boot 有一个开发工具（DevTools）模块，它有助于提高开发人员的生产力。Java 开发人员面临的一个主要挑战是将文件更改自动部署到服务器并自动重启服务器。开发人员可以重新加载 Spring Boot 上的更改，而无需重新启动服务器。这将消除每次手动部署更改的需要。Spring Boot 在发布它的第一个版本时没有这个功能。这是开发人员最需要的功能。DevTools 模块完全满足开发人员的需求。该模块将在生产环境中被禁用。它还提供 H2 数据库控制台以更好地测试应用程序。
org.springframework.boot spring-boot-devtools true

## 问题五 Spring Boot 中的监视器是什么？

	Spring boot actuator 是 spring 启动框架中的重要功能之一。Spring boot 监视器可帮助您访
问生产环境中正在运行的应用程序的当前状态。有几个指标必须在生产环境中进行检查和监控。即使一些外部应用程序可能正在使用这些服务来向相关人员触发警报消息。监视器模块公开了一组可直接作为 HTTP URL 访问的 REST 端点来检查状态。

## 问题六 如何在 Spring Boot 中禁用 Actuator 端点安全性？

	默认情况下，所有敏感的 HTTP 端点都是安全的，只有具有 ACTUATOR 角色的用户才能访问它们。安全性是使用标准的 HttpServletRequest.isUserInRole 方法实施的。 我们可以使用
`management.security.enabled = false`来禁用安全性。只有在执行机构端点在防火墙后访问时，才建议禁用安全性。

## 问题七 如何在自定义端口上运行 Spring Boot 应用程序？

为了在自定义端口上运行 Spring Boot 应用程序，您可以在 application.properties 中指定端口。`server.port = 8090`

## 问题八 什么是 YAML？

​	YAML 是一种人类可读的数据序列化语言。它通常用于配置文件。与属性文件相比，如果我们想要在配置文件中添加复杂的属性，YAML 文件就更加结构化，而且更少混淆。可以看出 YAML 具有分层配置数据。

## 问题九 如何实现 Spring Boot 应用程序的安全性？

​	为了实现 Spring Boot 的安全性，我们使用 spring-boot-starter-security 依赖项，并且必须添加安全配置。它只需很少的代码。配置类将必须扩展 `WebSecurityConfigurerAdapter `并覆盖其方法。

## 问题十 如何集成 Spring Boot 和 ActiveMQ？

​	对于集成 Spring Boot 和 ActiveMQ，我们使用`spring-boot-starter-activemq`依赖关系。 它只需要很少的配置，并且不需要样板代码。

## 问题十一 如何使用 Spring Boot 实现分页和排序？

​	使用 Spring Boot 实现分页非常简单。使用 Spring Data-JPA 可以实现将分页的`org.springframework.data.domain.Pageable`传递给存储库方法。

## 问题十二 什么是 Swagger？你用 Spring Boot 实现了它吗？

​	Swagger 广泛用于可视化 API，使用 Swagger UI 为前端开发人员提供在线沙箱。Swagger 是用于生成 RESTful Web 服务的可视化表示的工具，规范和完整框架实现。它使文档能够以与服务器相同的速度更新。当通过 Swagger 正确定义时，消费者可以使用最少量的实现逻辑来理解远程服务并与其进行交互。因此，Swagger 消除了调用服务时的猜测。

## 问题十三 什么是 Spring Profiles？

​	Spring Profiles 允许用户根据配置文件（dev，test，prod 等）来注册 bean。因此，当应用程序在开发中运行时，只有某些 bean 可以加载，而在 PRODUCTION 中，某些其他 bean 可以加载。假设我们的要求是 Swagger 文档仅适用于 QA 环境，并且禁用所有其他文档。这可以使用配置文件来完成。Spring Boot 使得使用配置文件非常简单。

## 问题十四 什么是 Spring Batch？

​	Spring Boot Batch 提供可重用的函数，这些函数在处理大量记录时非常重要，包括日志/跟踪，事务管理，作业处理统计信息，作业重新启动，跳过和资源管理。它还提供了更先进的技术服务和功能，通过优化和分区技术，可以实现极高批量和高性能批处理作业。简单以及复杂的大批量批处理作业可以高度可扩展的方式利用框架处理重要大量的信息。

## 问题十五 什么是 FreeMarker 模板？

​	FreeMarker 是一个基于 Java 的模板引擎，最初专注于使用 MVC 软件架构进行动态网页生成。使用 Freemarker 的主要优点是表示层和业务层的完全分离。程序员可以处理应用程序代码，而设计人员可以处理 html 页面设计。最后使用 freemarker 可以将这些结合起来，给出最终的输出页面。

## 问题十六 如何使用 Spring Boot 实现异常处理？

​	Spring 提供了一种使用 `ControllerAdvice `处理异常的非常有用的方法。 我们通过实现一个ControlerAdvice 类，来处理控制器类抛出的所有异常。

## 问题十七 您使用了哪些 starter maven 依赖项？

​	使用了下面的一些依赖项`spring-boot-starter-activemq`、 `spring-boot-starter-security`、`spring-boot-starter-web`这有助于增加更少的依赖关系，并减少版本的冲突。

## 问题十八 什么是 CSRF 攻击？

​	CSRF 代表跨站请求伪造。这是一种攻击，迫使最终用户在当前通过身份验证的 Web 应用程序上执行不需要的操作。CSRF 攻击专门针对状态改变请求，而不是数据窃取，因为攻击者无法查看对伪造请求的响应。

## 问题十九 什么是 WebSockets？

​	WebSocket 是一种计算机通信协议，通过单个 TCP 连接提供全双工通信信道。WebSocket 是双向的 -使用 WebSocket 客户端或服务器可以发起消息发送。WebSocket 是全双工的 -客户端和服务器通信是相互独立的。

​	单个 TCP 连接 -初始连接使用 HTTP，然后将此连接升级到基于套接字的连接。然后这个单一连接用于所有未来的通信，与 http 相比，WebSocket 消息数据交换要轻得多。

## 问题二十 什么是 AOP？

​	在软件开发过程中，跨越应用程序多个点的功能称为交叉问题。这些交叉问题与应用程序的主要业务逻辑不同。因此，将这些横切关注与业务逻辑分开是面向方面编程（AOP）的地方。

## 问题二十一 什么是 Apache Kafka？

​	Apache Kafka 是一个分布式发布 - 订阅消息系统。它是一个可扩展的，容错的发布 - 订阅消息系统，它使我们能够构建分布式应用程序。这是一个 Apache 顶级项目。Kafka 适合离线和在线消息消费。

## 问题二十二 我们如何监视所有 Spring Boot 微服务？

​	Spring Boot 提供监视器端点以监控各个微服务的度量。这些端点对于获取有关应用程序的信息（如它们是否已启动）以及它们的组件（如数据库等）是否正常运行很有帮助。但是，使用监视器的一个主要缺点或困难是，我们必须单独打开应用程序的知识点以了解其状态或健康状况。想象一下涉及 50 个应用程序的微服务，管理员将不得不击中所有 50 个应用程序的执行终端。

# 面试题35

## 问题一 Spring Boot、Spring MVC 和 Spring 有什么区别？

1、Spring
	Spring最重要的特征是依赖注入。所有 SpringModules 不是依赖注入就是 IOC 控制反转。
当我们恰当的使用 DI 或者是 IOC 的时候，我们可以开发松耦合应用。松耦合应用的单元测试可以很容易的进行。
2、Spring MVC
	Spring MVC 提供了一种分离式的方法来开发 Web 应用。通过运用像 DispatcherServelet，MoudlAndView 和 ViewResolver 等一些简单的概念，开发 Web 应用将会变的非常简单。
3、SpringBoot
	Spring 和 SpringMVC 的问题在于需要配置大量的参数。

Spring Boot 通过一个自动配置和启动的项来目解决这个问题。为了更快的构建产品就绪应用程序，Spring Boot 提供了一些非功能性特征。

## 问题二 什么是自动配置？

Spring 和 SpringMVC 的问题在于需要配置大量的参数。

当一个 MVC JAR 添加到应用程序中的时候，我们能否自动配置一些 beans？

Spring 查看（CLASSPATH 上可用的框架）已存在的应用程序的配置。在此基础上，Spring Boot 提供了配置应用程序和框架所需要的基本配置。这就是自动配置。

## 问题三 什么是 Spring Boot Stater ？

​	启动器是一套方便的依赖描述符，它可以放在自己的程序中。你可以一站式的获取你所需要的 Spring 和相关技术，而不需要依赖描述符的通过示例代码搜索和复制黏贴的负载。

​	例如，如果你想使用 Sping 和 JPA 访问数据库，只需要你的项目包含 spring-boot-starter-data-jpa 依赖项，你就可以完美进行。

## 问题四 你能否举一个例子来解释更多 Staters 的内容？

​	让我们来思考一个 Stater 的例子 -Spring Boot Stater Web。
如果你想开发一个 web 应用程序或者是公开 REST 服务的应用程序。Spring Boot Start Web 是首选。让我们使用 Spring Initializr 创建一个 Spring Boot Start Web 的快速项目。
Spring Boot Start Web 的依赖项

依赖项可以被分为：

```
Spring - core，beans，context，aop
Web MVC - （Spring MVC）
Jackson - for JSON Binding
Validation - Hibernate,Validation API
Enbedded Servlet Container - Tomcat
Logging - logback,slf4j
```

任何经典的 Web 应用程序都会使用所有这些依赖项。Spring Boot Starter Web 预先打包了这些依赖项。
作为一个开发者，我不需要再担心这些依赖项和它们的兼容版本。

## 问题五 Spring Boot 还提供了其它的哪些 Starter Project Options？

Spring Boot 也提供了其它的启动器项目包括，包括用于开发特定类型应用程序的典型依赖项。
spring-boot-starter-web-services - SOAP Web Services；
spring-boot-starter-web - Web 和 RESTful 应用程序；
spring-boot-starter-test - 单元测试和集成测试；
spring-boot-starter-jdbc - 传统的 JDBC；
spring-boot-starter-hateoas - 为服务添加 HATEOAS 功能；
spring-boot-starter-security - 使用 SpringSecurity 进行身份验证和授权；
spring-boot-starter-data-jpa - 带有 Hibeernate 的 Spring Data JPA；
spring-boot-starter-data-rest - 使用 Spring Data REST 公布简单的 REST 服务；

## 问题六 Spring 是如何快速创建产品就绪应用程序的？

​	Spring Boot 致力于快速产品就绪应用程序。为此，它提供了一些譬如高速缓存，日志记录，监控和嵌入式服务器等开箱即用的非功能性特征。
spring-boot-starter-actuator - 使用一些如监控和跟踪应用的高级功能
spring-boot-starter-undertow, spring-boot-starter-jetty, spring-boot-starter-tomcat - 选择您的特定嵌入式 Servlet 容器
spring-boot-starter-logging - 使用 logback 进行日志记录
spring-boot-starter-cache - 启用 Spring Framework 的缓存支持
Spring2 和 Spring5 所需要的最低 Java 版本是什么？
Spring Boot 2.0 需要 Java8 或者更新的版本。Java6 和 Java7 已经不再支持。
推荐阅读：
https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0.0-M1-Release-Notes

## 问题七 创建一个 Spring Boot Project 的最简单的方法是什么？

Spring Initializr是启动 Spring Boot Projects 的一个很好的工具。
登录 Spring Initializr，按照以下方式进行选择：
选择 com.in28minutes.springboot 为组
选择 studet-services 为组件
选择下面的依赖项
Web
Actuator
DevTools
点击生 GenerateProject
将项目导入 Eclipse。文件 - 导入 - 现有的 Maven 项目

## 问题八Spring Initializr 是创建 Spring Boot Projects 的唯一方法吗？

不是的。
Spring Initiatlizr 让创建 Spring Boot 项目变的很容易，但是，你也可以通过设置一个 maven 项目并添加正确的依赖项来开始一个项目。
在我们的 Spring 课程中，我们使用两种方法来创建项目。
第一种方法是 start.spring.io 。
另外一种方法是在项目的标题为“Basic Web Application”处进行手动设置。
手动设置一个 maven 项目
这里有几个重要的步骤：
在 Eclipse 中，使用文件 - 新建 Maven 项目来创建一个新项目
添加依赖项。
添加 maven 插件。
添加 Spring Boot 应用程序类。
到这里，准备工作已经做好！

## 问题九 为什么我们需要 spring-boot-maven-plugin?

spring-boot-maven-plugin 提供了一些像 jar 一样打包或者运行应用程序的命令。
spring-boot:run 运行你的 SpringBooty 应用程序。
spring-boot:repackage 重新打包你的 jar 包或者是 war 包使其可执行
spring-boot:start 和 spring-boot：stop 管理 Spring Boot 应用程序的生命周期（也可以说是为了集成测试）。
spring-boot:build-info 生成执行器可以使用的构造信息。

## 问题十 如何使用 SpringBoot 自动重装我的应用程序？

使用 Spring Boot 开发工具。
把 Spring Boot 开发工具添加进入你的项目是简单的。
把下面的依赖项添加至你的 Spring Boot Project pom.xml 中

重启应用程序，然后就可以了。
同样的，如果你想自动装载页面，有可以看看 FiveReload
http://www.logicbig.com/tutorials/spring-framework/spring-boot/boot-live-reload/.
在我测试的时候，发现了 LiveReload 漏洞，如果你测试时也发现了，请一定要告诉我们。

## 问题十一 什么是嵌入式服务器？我们为什么要使用嵌入式服务器呢?

思考一下在你的虚拟机上部署应用程序需要些什么。
第一步： 安装 Java
第二部： 安装 Web 或者是应用程序的服务器（Tomat/Wbesphere/Weblogic 等等）
第三部： 部署应用程序 war 包
如果我们想简化这些步骤，应该如何做呢？
让我们来思考如何使服务器成为应用程序的一部分？
你只需要一个安装了 Java 的虚拟机，就可以直接在上面部署应用程序了，
是不是很爽？
这个想法是嵌入式服务器的起源。
当我们创建一个可以部署的应用程序的时候，我们将会把服务器（例如，tomcat）嵌入到可部署的服务器中。
例如，对于一个 Spring Boot 应用程序来说，你可以生成一个包含 Embedded Tomcat 的应用程序 jar。你就可以想运行正常 Java 应用程序一样来运行 web 应用程序了。
嵌入式服务器就是我们的可执行单元包含服务器的二进制文件（例如，tomcat.jar）。

## 问题十二 如何在 Spring Boot 中添加通用的 JS 代码？

在源文件夹下，创建一个名为 static 的文件夹。然后，你可以把你的静态的内容放在这里面。
例如，myapp.js 的路径是 resources\static\js\myapp.js
你可以参考它在 jsp 中的使用方法：

错误：HAL browser gives me unauthorized error - Full authenticaition is required to access this resource.
该如何来修复这个错误呢？

两种方法：
方法 1：关闭安全验证
application.properties
management.security.enabled:FALSE
方法二：在日志中搜索密码并传递至请求标头中

## 问题十三 什么是 Spring Data？

来自：//projects.spring.io/spring- data/

Spring Data 的使命是在保证底层数据存储特殊性的前提下，为数据访问提供一个熟悉的，一致性的，基于 Spring 的编程模型。这使得使用数据访问技术，关系数据库和非关系数据库，map-reduce 框架以及基于云的数据服务变得很容易。
为了让它更简单一些，Spring Data 提供了不受底层数据源限制的 Abstractions 接口。

下面来举一个例子:

你可以定义一简单的库，用来插入，更新，删除和检索代办事项，而不需要编写大量的代码。

## 问题十四 什么是 Spring Data REST?

Spring Data TEST 可以用来发布关于 Spring 数据库的 HATEOAS RESTful 资源。
下面是一个使用 JPA 的例子:

不需要写太多代码，我们可以发布关于 Spring 数据库的 RESTful API。
下面展示的是一些关于 TEST 服务器的例子
POST:
URL:http：//localhost：8080/todos
Use Header:Content-Type:Type:application/json
Request Content
代码如下：

响应内容：

响应包含新创建资源的 href。

##  问题十五 path=”users”, collectionResourceRel=”users” 如何与 Spring Data Rest 一起使用？

path- 这个资源要导出的路径段。
collectionResourceRel- 生成指向集合资源的链接时使用的 rel 值。在生成 HATEOAS 链接时使用。

## 问题十六 当 Spring Boot 应用程序作为 Java 应用程序运行时，后台会发生什么？

如果你使用 Eclipse IDE，Eclipse maven 插件确保依赖项或者类文件的改变一经添加，就会被编译并在目标文件中准备好！在这之后，就和其它的 Java 应用程序一样了。
当你启动 java 应用程序的时候，spring boot 自动配置文件就会魔法般的启用了。
当 Spring Boot 应用程序检测到你正在开发一个 web 应用程序的时候，它就会启动 tomcat。

## 问题十七 我们能否在 spring-boot-starter-web 中用 jetty 代替 tomcat？
在 spring-boot-starter-web 移除现有的依赖项，并把下面这些添加进去。

## 问题十八 如何使用 Spring Boot 生成一个 WAR 文件？

推荐阅读:
https://spring.io/guides/gs/convert-jar-to-war/
下面有 spring 说明文档直接的链接地址：
https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#build-tool-plugins-maven-packaging

## 问题十九 如何使用 Spring Boot 部署到不同的服务器？

你需要做下面两个步骤：
在一个项目中生成一个 war 文件。
将它部署到你最喜欢的服务器（websphere 或者 Weblogic 或者 Tomcat and so on）。
第一步：这本入门指南应该有所帮助：
https://spring.io/guides/gs/convert-jar-to-war/
第二步：取决于你的服务器。

## 问题二十 RequestMapping 和 GetMapping 的不同之处在哪里？

RequestMapping 具有类属性的，可以进行 GET,POST,PUT 或者其它的注释中具有的请求方法。
GetMapping 是 GET 请求方法中的一个特例。它只是 ResquestMapping 的一个延伸，目的是为了提高清晰度。

## 问题二十一 为什么我们不建议在实际的应用程序中使用 Spring Data Rest?

我们认为 Spring Data Rest 很适合快速原型制造！在大型应用程序中使用需要谨慎。
通过 Spring Data REST 你可以把你的数据实体作为 RESTful 服务直接发布。
当你设计 RESTful 服务器的时候，最佳实践表明，你的接口应该考虑到两件重要的事情：
你的模型范围。
你的客户。
通过 With Spring Data REST，你不需要再考虑这两个方面，只需要作为 TEST 服务发布实体。
这就是为什么我们建议使用 Spring Data Rest 在快速原型构造上面，或者作为项目的初始解决方法。对于完整演变项目来说，这并不是一个好的注意。

## 问题二十二 在 Spring Initializer 中，如何改变一个项目的包名字？

好消息是你可以定制它。点击链接“转到完整版本”。你可以配置你想要修改的包名称！

## 问题二十三 可以配置 application.propertierde 的完整的属性列表在哪里可以找到？

这里是完整的指南：
https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html

## 问题二十四 JPA 和 Hibernate 有哪些区别？

简而言之
JPA 是一个规范或者接口
Hibernate 是 JPA 的一个实现
当我们使用 JPA 的时候，我们使用 javax.persistence 包中的注释和接口时，不需要使用 hibernate 的导入包。
我们建议使用 JPA 注释，因为我们没有将其绑定到 Hibernate 作为实现。我们可以使用另一种 JPA 实现。

## 问题二十五 业务边界应该从哪一层开始？

我们建议在服务层管理业务。商业业务逻辑在商业层或者服务层，与此同时，你想要执行的业务管理也在该层。

## 问题二十六 使用 Spring Boot 启动连接到内存数据库 H2 的 JPA 应用程序需要哪些依赖项？

在 Spring Boot 项目中，当你确保下面的依赖项都在类路里面的时候，你可以加载 H2 控制台。
web 启动器
h2
jpa 数据启动器
其它的依赖项在下面：

需要注意的一些地方：
一个内部数据内存只在应用程序执行期间存在。这是学习框架的有效方式。
这不是你希望的真是世界应用程序的方式。
在问题“如何连接一个外部数据库？”中，我们解释了如何连接一个你所选择的数据库。

## 问题二十七 如何不通过任何配置来选择 Hibernate 作为 JPA 的默认实现？

因为 Spring Boot 是自动配置的。
下面是我们添加的依赖项:

spring-boot-stater-data-jpa 对于 Hibernate 和 JPA 有过渡依赖性。
当 Spring Boot 在类路径中检测到 Hibernate 中，将会自动配置它为默认的 JPA 实现。

## 问题二十八 指定的数据库连接信息在哪里？它是如何知道自动连接至 H2 的？

这就是 Spring Boot 自动配置的魔力。
来自：https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-auto-configuration.html
Spring Boot auto-configuration 试图自动配置你已经添加的基于 jar 依赖项的 Spring 应用程序。比如说，如果 HSQLDBis 存在你的类路径中，并且，数据库连接 bean 还没有手动配置，那么我们可以自动配置一个内存数据库。
进一步的阅读：
http://www.springboottutorial.com/spring-boot-auto-configuration

## 问题二十九 我们如何连接一个像 MySQL 或者Orcale 一样的外部数据库？

让我们以 MySQL 为例来思考这个问题：
第一步 - 把 mysql 连接器的依赖项添加至 pom.xml

第二步 - 从 pom.xml 中移除 H2 的依赖项
或者至少把它作为测试的范围。

第三步 - 安装你的 MySQL 数据库
更多的来看看这里 -https://github.com/in28minutes/jpa-with-hibernate#installing-and-setting-up-mysql
第四步 - 配置你的 MySQL 数据库连接
配置 application.properties
spring.jpa.hibernate.ddl-auto=none spring.datasource.url=jdbc:mysql://localhost:3306/todo_example spring.datasource.username=todouser spring.datasource.password=YOUR_PASSWORD
第五步 - 重新启动，你就准备好了！
就是这么简单！

## 问题三十 Spring Boot 配置的默认 H2 数据库的名字是上面？为什么默认的数据库名字是 testdb？

在 application.properties 里面，列出了所有的默认值
https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html
找到下面的属性 Name of the datasource.

spring.datasource.name=testdb
如果你使用了 H2 内部存储数据库，它里面确定了 Spring Boot 用来安装你的 H2 数据库的名字。

## 问题三十一 如果 H2 不在类路径里面，会出现上面情况？

将会报下面的错误
Cannot determine embedded database driver class for database type NONE
把 H2 添加至 pom.xml 中，然后重启你的服务器

## 问题三十二 你能否举一个以 ReadOnly 为事务管理的例子？

当你从数据库读取内容的时候，你想把事物中的用户描述或者是其它描述设置为只读模式，以便于 Hebernate 不需要再次检查实体的变化。这是非常高效的。

## 问题三十三发布 Spring Boot 用户应用程序自定义配置的最好方法是什么？

@Value 的问题在于，您可以通过应用程序分配你配置值。更好的操作是采取集中的方法。
你可以使用 @ConfigurationProperties 定义一个配置组件。

你可以在 application.properties 中配置参数。
basic.value: true
basic.message: Dynamic Message
basic.number: 100

## 问题三十四 配置文件的需求是什么？

企业应用程序的开发是复杂的，你需要混合的环境：
Dev
QA
Stage
Production
在每个环境中，你想要不同的应用程序配置。
配置文件有助于在不同的环境中进行不同的应用程序配置。
Spring 和 Spring Boot 提供了你可以制定的功能。
不同配置文件中，不同环境的配置是什么？
为一个制定的环境设置活动的配置文件。
Spring Boot 将会根据特定环境中设置的活动配置文件来选择应用程序的配置。

## 问题三十五 如何使用配置文件通过 Spring Boot 配置特定环境的配置？

配置文件不是设别环境的关键。
在下面的例子中，我们将会用到两个配置文件
dev
prod
缺省的应用程序配置在 application.properties 中。让我们来看下面的例子：
application.properties
basic.value= true
basic.message= Dynamic Message
basic.number= 100
我们想要为 dev 文件自定义 application.properties 属性。我们需要创建一个名为 application-dev.properties 的文件，并且重写我们想要自定义的属性。
application-dev.properties
basic.message: Dynamic Message in DEV
一旦你特定配置了配置文件，你需要在环境中设定一个活动的配置文件。
有多种方法可以做到这一点：
在 VM 参数中使用 Dspring.profiles.active=prod
在 application.properties 中使用 spring.profiles.active=prod