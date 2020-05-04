# SpringBoot入门

可以使用Spring Boot创建Java应用程序，应用程序可以通过使用`java -jar`或更传统的部署来启动。我们还提供了一个运行“ spring脚本”的命令行工具。

目标：

- 为所有Spring开发提供根本上更快且可广泛访问的入门经验。
- 开箱即用，但随着需求开始偏离默认值，您会很快摆脱困境。
- 提供一系列大型项目通用的非功能性功能（例如嵌入式服务器，安全性，指标，运行状况检查和外部化配置）。
- 完全没有代码生成，也不需要XML配置。

```xml
 <!-- Inherit defaults from Spring Boot -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
    </parent>
```

Spring Boot依赖项使用`org.springframework.boot` `groupId`。通常Maven POM文件从`spring-boot-starter-parent`项目继承，是一个特殊的启动提供有用的Maven的默认值，并声明对一个或多个[“启动器”的](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-starter)依赖关系。Spring Boot还提供了一个可选的[Maven插件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins-maven-plugin)来创建可执行jar。

`mvn dependency:tree`命令显示项目依赖关系的树形表示。您可以看到它`spring-boot-starter-parent`本身不提供任何依赖关系。

## 简单启动

```java
@RestController
@EnableAutoConfiguration
public class Example {
@RequestMapping("/")
String home() {
    return "Hello World!";
}

public static void main(String[] args) {
    SpringApplication.run(Example.class, args);
}
}
```

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

## @RestController和@RequestMapping

​	`@RestController`。这被称为*构造型*注释。它为阅读代码的人和Spring提供了提示，提示该类起特定的作用。在这种情况下，我们的类是web `@Controller`，因此Spring在处理传入的Web请求时会考虑使用它。该`@RestController`注解告诉Spring使得到的字符串直接返回给调用者。

​	`@RequestMapping`注释提供“路由”的信息。它告诉Spring任何具有`/`路径的HTTP请求都应映射到该`home`方法。

> ​	Spring参考文档中的[MVC部分](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc)

### @EnableAutoConfiguration

​	`@EnableAutoConfiguration`注释告诉Spring Boot根据所添加的jar依赖关系“猜测”您如何配置Spring。由于`spring-boot-starter-web`添加了Tomcat和Spring MVC，因此自动配置假定您正在开发Web应用程序并相应地设置Spring。

​	启动器和自动配置

自动配置旨在与“启动器”配合使用，但是这两个概念并没有直接联系在一起。您可以自由选择启动器之外的jar依赖项。Spring Boot仍然尽其最大努力来自动配置您的应用程序。

## Main -> run(Class,args)

​	`main`方法只是遵循Java约定的应用程序入口点的标准方法。main方法`SpringApplication`通过调用委托给Spring Boot的类`run`。 `SpringApplication`引导我们的应用程序，启动Spring，这反过来又启动了自动配置的Tomcat Web服务器。我们需要将`Example.class`一个参数传递给该`run`方法，以判断`SpringApplication`哪个是主要的Spring组件。该`args`数组也将通过以公开任何命令行参数。

## 运行`mvn spring-boot:run`

​	由于使用了`spring-boot-starter-parent`POM，因此有一个有用的`run`目标，可以用来启动应用程序。`mvn spring-boot:run`从根项目目录中键入以启动应用程序。

## 创建可执行jar

​	创建一个可以在生产环境中运行的完全独立的可执行jar文件。	包含您的已编译类以及代码需要运行的所有jar依赖项的归档文件。

可执行jar和Java

​	Java没有提供加载嵌套jar文件（jar中本身包含的jar文件）的标准方法。我们需要将添加`spring-boot-maven-plugin`到`pom.xml`。在`dependencies`部分下方插入以下行

```xml
<build>
    <plugins>
        <plugin>        
          	<groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

从命令行运行`mvn package`，查看`target`目录，应该看到`myproject-0.0.1-SNAPSHOT.jar`，可以使用`jar tvf`解压查看细节，要运行该应用程序，使用`java -jar`

> java -jar target / myproject-0.0.1-SNAPSHOT.jar

# 使用Spring Boot

本节介绍使用Spring Boot涵盖了诸如构建系统，自动配置以及如何运行应用程序之类的主题。还将介绍一些Spring Boot最佳实践。

## 1  构建系统

建议选择一个支持[依赖关系管理](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-dependency-management)并且可以使用发布到“ Maven Central”存储库的工件的构建系统。我们建议您选择Maven或Gradle。

### 1.1 依赖管理

​	每个Spring Boot版本都与Spring Framework的基本版本相关联。实际不需要为构建配置中的所有这些依赖项提供版本，因为Spring Boot会为您管理该版本。当您升级Spring Boot本身时，这些依赖项也会以一致的方式升级。

> ​	您仍然可以指定版本，并在需要时覆盖Spring Boot的建议。
>

### 1.2 Maven

Maven用户可以从`spring-boot-starter-parent`项目继承以获得合理的默认值。父项目提供以下功能：

- Java 1.8是默认的编译器级别。
- UTF-8源编码。
- 一个[依赖管理部分](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-dependency-management)，从春天启动依赖性POM，管理公共依赖的版本继承。当在您自己的pom中使用时，这种依赖关系管理使您可以为这些依赖项省略<version>标记。
- 具有执行ID 的[`repackage`目标](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/maven-plugin//repackage-mojo.html)的`repackage`执行。
- 明智的[资源过滤](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html)。
- 合理的插件配置（[exec插件](https://www.mojohaus.org/exec-maven-plugin/)，[Git提交ID](https://github.com/ktoso/maven-git-commit-id-plugin)和[shade](https://maven.apache.org/plugins/maven-shade-plugin/)）。
- 针对`application.properties`和`application.yml`包括特定于配置文件的文件（例如`application-dev.properties`和`application-dev.yml`）进行明智的资源过滤

请注意，由于`application.properties`和`application.yml`文件都接受Spring样式的占位符（`${…}`），因此Maven过滤已更改为使用`@..@`占位符。（您可以通过设置名为的Maven属性来覆盖它`resource.delimiter`。）

#### 1.2.1 继承入门级父级

要将您的项目配置为继承自`spring-boot-starter-parent`，请设置`parent`如下：

```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.6.RELEASE</version>
</parent>
```

|      | 您只需要为此依赖项指定Spring Boot版本号。如果导入其他启动器，则可以安全地省略版本号。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

使用该设置，您还可以通过覆盖自己项目中的属性来覆盖各个依赖项。例如，要升级到另一个Spring Data发布火车，您可以将以下内容添加到您的`pom.xml`：

```xml
<properties>
    <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

|      | 检查[`spring-boot-dependencies`pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml)以获取受支持属性的列表。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.2.2。在没有父POM的情况下使用Spring Boot

并非每个人都喜欢从`spring-boot-starter-parent`POM 继承。您可能需要使用自己的公司标准父级，或者可能希望显式声明所有Maven配置。

如果您不想使用`spring-boot-starter-parent`，仍然可以通过使用`scope=import`依赖项来保留依赖项管理（而不是插件管理）的好处，如下所示：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

如上所述，前面的示例设置不允许您使用属性来覆盖各个依赖项。为了达到同样的效果，你需要添加的条目`dependencyManagement`项目的**之前**的`spring-boot-dependencies`条目。例如，要升级到另一个Spring Data发布系列，可以将以下元素添加到`pom.xml`：

```xml
<dependencyManagement>
    <dependencies>
        <!-- Override Spring Data release train provided by Spring Boot -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-releasetrain</artifactId>
            <version>Fowler-SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

|      | 在前面的示例中，我们指定了*BOM*，但是可以以相同的方式覆盖任何依赖项类型。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.2.3。使用Spring Boot Maven插件

Spring Boot包含一个[Maven插件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins-maven-plugin)，可以将项目打包为可执行jar。如果要使用插件，请将该插件添加到您的部分，如以下示例所示：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

|      | 如果您使用Spring Boot启动器的父pom，则只需添加插件。除非您要更改父级中定义的设置，否则无需对其进行配置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.3 Gradle

要了解有关将Spring Boot与Gradle结合使用的信息，请参阅Spring Boot的Gradle插件的文档：

- 参考（[HTML](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/html/)和[PDF](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/pdf/spring-boot-gradle-plugin-reference.pdf)）
- [API](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/api/)

### 1.4 Ant

可以使用Apache Ant + Ivy构建Spring Boot项目。该`spring-boot-antlib`“的antlib”模块还可以帮助蚂蚁创建可执行的JAR文件。

为了声明依赖关系，典型`ivy.xml`文件看起来类似于以下示例：

```xml
<ivy-module version="2.0">
    <info organisation="org.springframework.boot" module="spring-boot-sample-ant" />
    <configurations>
        <conf name="compile" description="everything needed to compile this module" />
        <conf name="runtime" extends="compile" description="everything needed to run this module" />
    </configurations>
    <dependencies>
        <dependency org="org.springframework.boot" name="spring-boot-starter"
            rev="${spring-boot.version}" conf="compile" />
    </dependencies>
</ivy-module>
```

典型`build.xml`的示例如下所示：

```xml
<project
    xmlns:ivy="antlib:org.apache.ivy.ant"
    xmlns:spring-boot="antlib:org.springframework.boot.ant"
    name="myapp" default="build">

    <property name="spring-boot.version" value="2.2.6.RELEASE" />

    <target name="resolve" description="--> retrieve dependencies with ivy">
        <ivy:retrieve pattern="lib/[conf]/[artifact]-[type]-[revision].[ext]" />
    </target>

    <target name="classpaths" depends="resolve">
        <path id="compile.classpath">
            <fileset dir="lib/compile" includes="*.jar" />
        </path>
    </target>

    <target name="init" depends="classpaths">
        <mkdir dir="build/classes" />
    </target>

    <target name="compile" depends="init" description="compile">
        <javac srcdir="src/main/java" destdir="build/classes" classpathref="compile.classpath" />
    </target>

    <target name="build" depends="compile">
        <spring-boot:exejar destfile="build/myapp.jar" classes="build/classes">
            <spring-boot:lib>
                <fileset dir="lib/runtime" />
            </spring-boot:lib>
        </spring-boot:exejar>
    </target>
</project>
```

|      | 如果您不想使用该`spring-boot-antlib`模块，请参见*[howto.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-build-an-executable-archive-with-ant)* “操作方法”。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.5 Starters

入门程序是一组方便的依赖项描述符，您可以在应用程序中包括它们。您可以一站式购买所需的所有Spring和相关技术，而不必搜寻示例代码和依赖描述符的复制粘贴负载。例如，如果要开始使用Spring和JPA进行数据库访问，请`spring-boot-starter-data-jpa`在项目中包括依赖项。

入门程序包含许多启动项目并快速运行所需的依赖项，并且具有一组受支持的受管传递性依赖项。

名字叫什么

所有**官方**入门者都遵循类似的命名方式。`spring-boot-starter-*`，其中`*`是特定类型的应用程序。这种命名结构旨在在您需要寻找入门者时提供帮助。许多IDE中的Maven集成使您可以按名称搜索依赖项。例如，在安装了适当的Eclipse或STS插件的情况下，您可以按`ctrl-space`POM编辑器，然后键入“ spring-boot-starter”以获取完整列表。

如“ [创建您自己的启动程序](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-custom-starter) ”部分中所述，第三方启动程序不应以开头`spring-boot`，因为它保留用于正式的Spring Boot构件。而是，第三方启动程序通常以项目名称开头。例如，`thirdpartyproject`通常将名为的第三方启动程序项目命名为`thirdpartyproject-spring-boot-starter`。

Spring Boot在该`org.springframework.boot`组下提供了以下应用程序启动器：

| 名称                                          | 描述                                                         | Pom                                                          |
| :-------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter`                         | 核心入门工具，包括自动配置支持，日志记录和YAML               | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter/pom.xml) |
| `spring-boot-starter-activemq`                | 使用Apache ActiveMQ的JMS消息传递入门                         | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-activemq/pom.xml) |
| `spring-boot-starter-amqp`                    | 使用Spring AMQP和Rabbit MQ的入门                             | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-amqp/pom.xml) |
| `spring-boot-starter-aop`                     | 使用Spring AOP和AspectJ进行面向方面编程的入门                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-aop/pom.xml) |
| `spring-boot-starter-artemis`                 | 使用Apache Artemis的JMS消息传递入门                          | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-artemis/pom.xml) |
| `spring-boot-starter-batch`                   | 使用Spring Batch的入门                                       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-batch/pom.xml) |
| `spring-boot-starter-cache`                   | 开始使用Spring Framework的缓存支持                           | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cache/pom.xml) |
| `spring-boot-starter-cloud-connectors`        | 使用Spring Cloud Connectors的入门程序，可简化与Cloud Foundry和Heroku等云平台中服务的连接。不推荐使用Java CFEnv | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cloud-connectors/pom.xml) |
| `spring-boot-starter-data-cassandra`          | 使用Cassandra分布式数据库和Spring Data Cassandra的入门       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra/pom.xml) |
| `spring-boot-starter-data-cassandra-reactive` | 使用Cassandra分布式数据库和Spring Data Cassandra Reactive的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra-reactive/pom.xml) |
| `spring-boot-starter-data-couchbase`          | 使用Couchbase面向文档的数据库和Spring Data Couchbase的入门   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase/pom.xml) |
| `spring-boot-starter-data-couchbase-reactive` | 使用Couchbase面向文档的数据库和Spring Data Couchbase Reactive的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase-reactive/pom.xml) |
| `spring-boot-starter-data-elasticsearch`      | 使用Elasticsearch搜索和分析引擎以及Spring Data Elasticsearch的入门者 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-elasticsearch/pom.xml) |
| `spring-boot-starter-data-jdbc`               | 使用Spring Data JDBC的入门                                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jdbc/pom.xml) |
| `spring-boot-starter-data-jpa`                | 将Spring Data JPA与Hibernate结合使用的入门                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jpa/pom.xml) |
| `spring-boot-starter-data-ldap`               | 使用Spring Data LDAP的入门                                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-ldap/pom.xml) |
| `spring-boot-starter-data-mongodb`            | 使用MongoDB面向文档的数据库和Spring Data MongoDB的入门       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb/pom.xml) |
| `spring-boot-starter-data-mongodb-reactive`   | 使用MongoDB面向文档的数据库和Spring Data MongoDB Reactive的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb-reactive/pom.xml) |
| `spring-boot-starter-data-neo4j`              | 使用Neo4j图形数据库和Spring Data Neo4j的入门                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-neo4j/pom.xml) |
| `spring-boot-starter-data-redis`              | 使用Redis键值数据存储与Spring Data Redis和Lettuce客户端的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis/pom.xml) |
| `spring-boot-starter-data-redis-reactive`     | 将Redis键值数据存储与Spring Data Redis Reacting和Lettuce客户端一起使用的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis-reactive/pom.xml) |
| `spring-boot-starter-data-rest`               | 使用Spring Data REST通过REST公开Spring数据存储库的入门       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-rest/pom.xml) |
| `spring-boot-starter-data-solr`               | 将Apache Solr搜索平台与Spring Data Solr结合使用的入门        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-solr/pom.xml) |
| `spring-boot-starter-freemarker`              | 使用FreeMarker视图构建MVC Web应用程序的入门                  | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-freemarker/pom.xml) |
| `spring-boot-starter-groovy-templates`        | 使用Groovy模板视图构建MVC Web应用程序的入门                  | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-groovy-templates/pom.xml) |
| `spring-boot-starter-hateoas`                 | 使用Spring MVC和Spring HATEOAS构建基于超媒体的RESTful Web应用程序的入门者 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-hateoas/pom.xml) |
| `spring-boot-starter-integration`             | 使用Spring Integration的入门                                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-integration/pom.xml) |
| `spring-boot-starter-jdbc`                    | 结合使用JDBC和HikariCP连接池的入门                           | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jdbc/pom.xml) |
| `spring-boot-starter-jersey`                  | 使用JAX-RS和Jersey构建RESTful Web应用程序的入门。的替代品[`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-web) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jersey/pom.xml) |
| `spring-boot-starter-jooq`                    | 使用jOOQ访问SQL数据库的入门。替代[`spring-boot-starter-data-jpa`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-data-jpa)或[`spring-boot-starter-jdbc`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-jdbc) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jooq/pom.xml) |
| `spring-boot-starter-json`                    | 读写JSON入门                                                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-json/pom.xml) |
| `spring-boot-starter-jta-atomikos`            | 使用Atomikos的JTA交易入门                                    | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-atomikos/pom.xml) |
| `spring-boot-starter-jta-bitronix`            | 使用Bitronix的JTA交易入门                                    | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-bitronix/pom.xml) |
| `spring-boot-starter-mail`                    | 使用Java Mail和Spring Framework的电子邮件发送支持的入门      | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mail/pom.xml) |
| `spring-boot-starter-mustache`                | 使用Mustache视图构建Web应用程序的入门                        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mustache/pom.xml) |
| `spring-boot-starter-oauth2-client`           | 使用Spring Security的OAuth2 / OpenID Connect客户端功能的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-oauth2-client/pom.xml) |
| `spring-boot-starter-oauth2-resource-server`  | 使用Spring Security的OAuth2资源服务器功能的入门              | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-oauth2-resource-server/pom.xml) |
| `spring-boot-starter-quartz`                  | 入门使用Quartz Scheduler                                     | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-quartz/pom.xml) |
| `spring-boot-starter-rsocket`                 | 用于构建RSocket客户端和服务器的入门。                        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-rsocket/pom.xml) |
| `spring-boot-starter-security`                | 使用Spring Security的入门                                    | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-security/pom.xml) |
| `spring-boot-starter-test`                    | 用于使用包括JUnit，Hamcrest和Mockito在内的库测试Spring Boot应用程序的入门程序 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-test/pom.xml) |
| `spring-boot-starter-thymeleaf`               | 使用Thymeleaf视图构建MVC Web应用程序的入门                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-thymeleaf/pom.xml) |
| `spring-boot-starter-validation`              | 通过Hibernate Validator使用Java Bean验证的入门               | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-validation/pom.xml) |
| `spring-boot-starter-web`                     | 使用Spring MVC构建Web（包括RESTful）应用程序的入门者。使用Tomcat作为默认的嵌入式容器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web/pom.xml) |
| `spring-boot-starter-web-services`            | 使用Spring Web Services的入门                                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web-services/pom.xml) |
| `spring-boot-starter-webflux`                 | 使用Spring Framework的Reactive Web支持构建WebFlux应用程序的入门者 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-webflux/pom.xml) |
| `spring-boot-starter-websocket`               | 使用Spring Framework的WebSocket支持构建WebSocket应用程序的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-websocket/pom.xml) |

除了应用程序启动程序，以下启动程序可用于添加*[生产就绪](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready)*功能：

| 名称                           | 描述                                                         | Pom                                                          |
| :----------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter-actuator` | 使用Spring Boot的Actuator的入门程序，它提供了生产就绪功能，可帮助您监视和管理应用程序 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-actuator/pom.xml) |

最后，Spring Boot还包括以下启动程序，如果您想排除或交换特定的技术方面，可以使用这些启动程序：

| 名称                                | 描述                                                         | Pom                                                          |
| :---------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter-jetty`         | 使用Jetty作为嵌入式servlet容器的入门者。的替代品[`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-tomcat) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jetty/pom.xml) |
| `spring-boot-starter-log4j2`        | 使用Log4j2进行日志记录的启动器。的替代品[`spring-boot-starter-logging`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-logging) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-log4j2/pom.xml) |
| `spring-boot-starter-logging`       | 使用Logback进行日志记录的启动器。默认记录启动器              | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-logging/pom.xml) |
| `spring-boot-starter-reactor-netty` | 使用Reactor Netty作为嵌入式反应式HTTP服务器的入门者。        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-reactor-netty/pom.xml) |
| `spring-boot-starter-tomcat`        | 入门，用于将Tomcat用作嵌入式Servlet容器。默认使用的servlet容器启动器[`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-web) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-tomcat/pom.xml) |
| `spring-boot-starter-undertow`      | 使用Undertow作为嵌入式servlet容器的入门。的替代品[`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-tomcat) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-undertow/pom.xml) |

|      | 有关社区贡献的其他入门者的列表，请参阅GitHub 中模块中的[README文件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/README.adoc)`spring-boot-starters`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 2.构建代码

Spring Boot不需要任何特定的代码布局即可工作。但是，有一些最佳实践可以帮助您。

### 2.1 Using the “default” Package

当一个类不包含`package`声明时，它被认为是在“默认包”中。通常不建议使用“默认程序包”，应避免使用。这可能会导致使用了Spring启动应用程序的特殊问题`@ComponentScan`，`@ConfigurationPropertiesScan`，`@EntityScan`，或`@SpringBootApplication`注解，因为从每一个罐子每一个类被读取。

|      | 我们建议您遵循Java建议的程序包命名约定，并使用反向域名（例如`com.example.project`）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.2 Locating the Main Application Class

我们通常建议您将主应用程序类放在其他类之上的根包中。该[`@SpringBootApplication`注解](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-using-springbootapplication-annotation)往往放在你的主类，它隐含地定义为某些项目一基地“搜索包”。例如，如果您正在编写JPA应用程序，`@SpringBootApplication`则使用带注释的类的包搜索`@Entity`项目。使用根软件包还允许组件扫描仅应用于您的项目。

|      | 如果您不想使用`@SpringBootApplication`，则导入的`@EnableAutoConfiguration`和`@ComponentScan`注释会定义该行为，因此您也可以使用它们。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下清单显示了典型的布局：

```
com 
 +-示例
     +-myapplication 
         + -Application.java 
         | 
         +-客户
         | +-Customer.java 
         | +-CustomerController.java 
         | +-CustomerService.java 
         | +-CustomerRepository.java 
         | 
         +-订单
             +-Order.java 
             +-OrderController.java 
             + -OrderService.java + 
             -OrderRepository.java
```

该`Application.java`文件将声明`main`方法以及basic方法，`@SpringBootApplication`如下所示：

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

## 3.Configuration Classes

Spring Boot支持基于Java的配置。尽管可以`SpringApplication`与XML源一起使用，但是我们通常建议您的主要源为单个`@Configuration`类。通常，定义`main`方法的类是主要的候选者`@Configuration`。

|      | Internet上已经发布了许多使用XML配置的Spring配置示例。如果可能，请始终尝试使用等效的基于Java的配置。搜索`Enable*`注释可以是一个很好的起点。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 3.1 `@Import`和`@Configuration`导入其他配置类

您无需将所有内容都`@Configuration`放在一个类中。所述`@Import`注释可以用于导入额外的配置类。另外，您可以`@ComponentScan`用来自动拾取所有Spring组件，包括`@Configuration`类。

### 3.2 `@ImportResource`导入XML配置

如果绝对必须使用基于XML的配置，我们建议您仍然从一个`@Configuration`类开始。然后，您可以使用`@ImportResource`批注来加载XML配置文件。

## 4.自动配置

Spring Boot自动配置会尝试根据添加的jar依赖关系自动配置Spring应用程序。例如，如果`HSQLDB`位于类路径上，并且尚未手动配置任何数据库连接bean，那么Spring Boot会自动配置内存数据库。

您需要通过将`@EnableAutoConfiguration`或`@SpringBootApplication`注释添加到一个`@Configuration`类中来选择自动配置。

|      | 您只能添加一个`@SpringBootApplication`或`@EnableAutoConfiguration`注释。我们通常建议您仅将一个或另一个添加到您的主要`@Configuration` class。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.1 逐渐取代自动配置

自动配置是非侵入性的。在任何时候，您都可以开始定义自己的配置，以替换自动配置的特定部分。例如，如果您添加自己的`DataSource`bean，则默认的嵌入式数据库支持将退出。

如果您需要找出当前正在应用的自动配置以及原因，请使用`--debug`开关启动应用程序。这样做可以启用调试日志，以选择核心记录器，并将条件报告记录到控制台。

### 4.2 禁用特定的自动配置类

如果发现正在应用不需要的特定自动配置类，则可以使用exclude属性`@SpringBootApplication`来禁用它们，如以下示例所示：

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;

@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
public class MyApplication {
}
```

如果该类不在类路径中，则可以使用`excludeName`注释的属性，并指定完全限定的名称。如果您喜欢使用`@EnableAutoConfiguration`而不是`@SpringBootApplication`，`exclude`并且`excludeName`也可以使用。最后，您还可以使用`spring.autoconfigure.exclude`属性来控制要排除的自动配置类的列表。

|      | 您可以在注释级别和使用属性来定义排除项。 |
| ---- | ---------------------------------------- |
|      |                                          |

|      | 即使有自动配置类`public`，该类的唯一方面也被认为是公共API，该类的名称就是可用于禁用自动配置的类的名称。这些类的实际内容（例如嵌套配置类或Bean方法）仅供内部使用，我们不建议直接使用它们。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 5.Spring Beans和依赖注入

您可以自由使用任何标准的Spring Framework技术来定义bean及其注入的依赖项。为简单起见，使用`@ComponentScan`（查找bean）和使用`@Autowired`（进行构造函数注入）效果很好。

如果按照上面的建议构造代码（将应用程序类放在根包中），则可以添加`@ComponentScan`而无需任何参数。您的所有应用程序组件（的`@Component`，`@Service`，`@Repository`，`@Controller`等）自动注册为Spring Beans。

以下示例显示了一个`@Service`使用构造函数注入来获取所需`RiskAssessor`Bean的Bean：

```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

如果bean具有一个构造函数，则可以省略`@Autowired`，如以下示例所示：

```java
@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

|      | 注意，使用构造函数注入将`riskAssessor`字段标记为`final`，指示该字段随后不能更改。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 6.使用@SpringBootApplication注释

许多Spring Boot开发人员喜欢他们的应用程序使用自动配置，组件扫描，并能够在其“应用程序类”上定义额外的配置。单个`@SpringBootApplication`注释可用于启用这三个功能，即：

- `@EnableAutoConfiguration`：启用[Spring Boot的自动配置机制](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-auto-configuration)
- `@ComponentScan`：启用`@Component`对应用程序所在的软件包的扫描（请参阅[最佳实践](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-structuring-your-code)）
- `@Configuration`：允许在上下文中注册额外的bean或导入其他配置类

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

|      | `@SpringBootApplication`还提供了别名定制的属性`@EnableAutoConfiguration`和`@ComponentScan`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

​	 这些功能都不是强制性的，您可以选择用它启用的任何功能替换此单个注释。例如，您可能不想在应用程序中使用组件扫描或配置属性扫描：



```java
@Configuration(proxyBeanMethods = false)
@EnableAutoConfiguration
@Import({ MyConfig.class, MyAnotherConfig.class })
public class Application {
public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
}
```

​	在此示例中，`Application`与其他任何Spring Boot应用程序一样，除了不会自动检测`@Component`-annotated类和`@ConfigurationProperties`-annotated类，并且显式导入用户定义的Bean（请参阅参考资料`@Import`）

## 7.运行您的应用程序

将应用程序打包为jar并使用嵌入式HTTP服务器的最大优势之一是，您可以像运行其他应用程序一样运行应用程序。调试Spring Boot应用程序也很容易。您不需要任何特殊的IDE插件或扩展。

|      | 本节仅介绍基于jar based packaging。如果选择将应用程序打包为war文件，则应参考服务器和IDE文档。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 7.1 从IDE运行

您可以将IDE中的Spring Boot应用程序作为简单的Java应用程序运行。但是，您首先需要导入您的项目。导入步骤因您的IDE和构建系统而异。大多数IDE可以直接导入Maven项目。例如，Eclipse用户可以从菜单中选择`Import…`→ `Existing Maven Projects`->`File`

如果您不能直接将项目导入IDE，则可以使用构建插件生成IDE元数据。Maven包括[Eclipse](https://maven.apache.org/plugins/maven-eclipse-plugin/)和[IDEA的](https://maven.apache.org/plugins/maven-idea-plugin/)插件。Gradle提供了用于[各种IDE的](https://docs.gradle.org/current/userguide/userguide.html)插件。

|      | 如果不小心两次运行Web应用程序，则会看到“端口已在使用中”错误。STS用户可以使用`Relaunch`按钮而不是`Run`按钮来确保关闭任何现有实例。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 7.2 作为打包的应用程序运行

如果您使用Spring Boot Maven或Gradle插件来创建可执行jar，则可以使用来运行您的应用程序`java -jar`，如以下示例所示：

```
$ java -jar target / myapplication-0.0.1-SNAPSHOT.jar
```

也可以在启用了远程调试支持的情况下运行打包的应用程序。这样可以将调试器附加到打包的应用程序，如以下示例所示：

```
$ java -Xdebug -Xrunjdwp：server = y，transport = dt_socket，address = 8000，suspend = n \ 
       -jar target / myapplication-0.0.1-SNAPSHOT.jar
```

### 7.3 使用Maven插件

Spring Boot Maven插件包含一个`run`目标，可用于快速编译和运行您的应用程序。应用程序以爆炸形式运行，就像在IDE中一样。以下示例显示了运行Spring Boot应用程序的典型Maven命令：

```
$ mvn spring-boot：运行
```

您可能还想使用`MAVEN_OPTS`操作系统环境变量，如以下示例所示：

```
$ export MAVEN_OPTS = -Xmx1024m
```

### 7.4 使用Gradle插件

Spring Boot Gradle插件还包含一个`bootRun`任务，可用于以爆炸形式运行您的应用程序。`bootRun`每当您应用`org.springframework.boot`和`java`插件时，都会添加该任务，并在以下示例中显示：

```
$ gradle bootRun
```

您可能还想使用`JAVA_OPTS`操作系统环境变量，如以下示例所示：

```
$ export JAVA_OPTS = -Xmx1024m
```

### 7.5 Hot Swapping

由于Spring Boot应用程序只是普通的Java应用程序，因此JVM热交换应该可以立即使用。JVM热插拔在一定程度上受到它可以替换的字节码的限制。对于更完整的解决方案，可以使用[JRebel](https://www.jrebel.com/products/jrebel)。

该`spring-boot-devtools`模块还包括对应用程序快速重启的支持。有关详细信息，请参见本章稍后的“ [开发人员工具”](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools)部分和[热插拔“操作方法”](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-hotswapping)。

## 8.开发人员工具

Spring Boot包括一组额外的工具，这些工具可以使应用程序开发体验更加愉快。该`spring-boot-devtools`模块可以包含在任何项目中，以提供其他开发时功能。要包括devtools支持，请将模块依赖项添加到您的构建中，如以下Maven和Gradle清单所示：

**Maven**

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

**Gradle**

```groovy
configurations {
    developmentOnly
    runtimeClasspath {
        extendsFrom developmentOnly
    }
}
dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```

|      | 运行完全打包的应用程序时，将自动禁用开发人员工具。如果您的应用程序是`java -jar`从某个特殊的类加载器启动的，或者从特殊的类加载器启动的，则将其视为“生产应用程序”。如果那对您不适用（即，如果您从容器中运行应用程序），请考虑排除devtools或设置`-Dspring.devtools.restart.enabled=false`系统属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 重新打包的存档默认情况下不包含devtools。如果要使用[某个远程devtools功能](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-remote)，则需要禁用`excludeDevtools`build属性以包含它。Maven和Gradle插件均支持该属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 8.1 属性默认值

Spring Boot支持的一些库使用缓存来提高性能。例如，[模板引擎](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-template-engines)缓存已编译的模板，以避免重复解析模板文件。而且，Spring MVC可以在提供静态资源时向响应添加HTTP缓存标头。

尽管缓存在生产中非常有益，但在开发过程中可能适得其反，从而使您无法看到自己刚刚在应用程序中所做的更改。因此，默认情况下，spring-boot-devtools禁用缓存选项。

缓存选项通常由`application.properties`文件中的设置配置。例如，Thymeleaf提供了该`spring.thymeleaf.cache`属性。该`spring-boot-devtools`模块无需自动设置这些属性，而是自动应用合理的开发时配置。

因为在开发Spring MVC和Spring WebFlux应用程序时需要有关Web请求的更多信息，所以开发人员工具将启用`DEBUG`日志`web`记录组的日志记录。这将为您提供有关传入请求，正在处理的处理程序，响应结果等的信息。如果您希望记录所有请求的详细信息（包括潜在的敏感信息），则可以打开`spring.http.log-request-details`配置属性。

|      | 如果你不想被应用属性默认可以设置`spring.devtools.add-properties`到`false`你`application.properties`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 有关devtools应用的属性的完整列表，请参见[DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 8.2 自动重启

`spring-boot-devtools`只要类路径上的文件发生更改，使用的应用程序就会自动重新启动。在IDE中工作时，这可能是一个有用的功能，因为它为代码更改提供了非常快速的反馈循环。默认情况下，将监视类路径上指向文件夹的任何条目的更改。请注意，某些资源（例如静态资产和视图模板）[不需要重新启动应用程序](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-restart-exclude)。

触发重启

当DevTools监视类路径资源时，触发重启的唯一方法是更新类路径。导致类路径更新的方式取决于您使用的IDE。在Eclipse中，保存修改后的文件将导致类路径被更新并触发重新启动。在IntelliJ IDEA中，构建项目（`Build +→+ Build Project`）具有相同的效果。

|      | 只要启用了分叉，您还可以使用受支持的构建插件（Maven和Gradle）来启动应用程序，因为DevTools需要隔离的应用程序类加载器才能正常运行。默认情况下，Gradle和Maven插件会分叉应用程序进程。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 与LiveReload一起使用时，自动重启非常有效。 [有关](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-livereload)详细信息，[请参见LiveReload部分](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-livereload)。如果使用JRebel，则禁用自动重新启动，而支持动态类重新加载。其他devtools功能（例如LiveReload和属性替代）仍可以使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | DevTools依赖于应用程序上下文的关闭挂钩在重新启动期间将其关闭。如果您禁用了关机挂钩（`SpringApplication.setRegisterShutdownHook(false)`），它将无法正常工作。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 当决定是否在类路径中的条目应该触发重新启动时，它的变化，DevTools自动忽略命名的项目`spring-boot`，`spring-boot-devtools`，`spring-boot-autoconfigure`，`spring-boot-actuator`，和`spring-boot-starter`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | DevTools需求来定制`ResourceLoader`使用的`ApplicationContext`。如果您的应用程序已经提供了，它将被包装。不支持直接覆盖`getResource`方法`ApplicationContext`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

重新启动与重新加载

Spring Boot提供的重启技术通过使用两个类加载器来工作。不**变的类（例如，来自第三方jar的类）将被加载到基类加载器中**。**开发的类将加载到重新启动类加载器中**。重新启动应用程序时，将丢弃重新启动类加载器，并创建一个新的类加载器。这种方法意味着应用程序的重启通常比“冷启动”要快得多，因为基本类加载器已经可用并已填充。

如果发现重新启动对于您的应用程序来说不够快，或者遇到类加载问题，则可以考虑从ZeroTurnaround 重新加载技术，例如[JRebel](https://jrebel.com/software/jrebel/)。这些方法通过在加载类时重写类来使它们更适合于重新加载。

#### 8.2.1 记录条件评估中的更改

默认情况下，每次应用程序重新启动时，都会记录一个报告，其中显示了条件评估增量。该报告显示了在进行诸如添加或删除bean以及设置配置属性之类的更改时对应用程序自动配置的更改。

要禁用报告的日志记录，请设置以下属性：

```
spring.devtools.restart.log-condition-evaluation-delta = false
```

#### 8.2.2 排除资源

某些资源在更改时不一定需要触发重新启动。例如，Thymeleaf模板可以就地编辑。默认情况下，改变资源`/META-INF/maven`，`/META-INF/resources`，`/resources`，`/static`，`/public`，或`/templates`不触发重新启动，但确会触发[现场重装](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-livereload)。如果要自定义这些排除项，则可以使用该`spring.devtools.restart.exclude`属性。例如，仅排除`/static`，`/public`您将设置以下属性：

```
spring.devtools.restart.exclude=static/**,public/**
```

|      | 如果要保留这些默认值并添加其他排除项，请改用该`spring.devtools.restart.additional-exclude`属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 8.2.3 监视其他路径

当您对不在类路径上的文件进行更改时，您可能希望重新启动或重新加载应用程序。为此，请使用该`spring.devtools.restart.additional-paths`属性来配置其他路径以监视更改。您可以使用[前面描述](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-restart-exclude)的`spring.devtools.restart.exclude`属性来控制其他路径下的更改是触发完全重新启动还是[实时重新加载](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-livereload)。

#### 8.2.4 禁用重启

如果您不想使用重新启动功能，则可以使用该`spring.devtools.restart.enabled`属性将其禁用。在大多数情况下，可以在自己的设备中设置此属性`application.properties`（这样做仍会初始化重新启动类加载器，但它不会监视文件更改）。

如果您需要*完全*禁用重启支持，则需要在调用`SpringApplication.run(…)`之前将`spring.devtools.restart.enabled` `System`属性设置为`false`，如以下示例所示：

```java
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```

#### 8.2.5 使用触发文件

如果使用持续编译更改文件的IDE，则可能更喜欢仅在特定时间触发重新启动。为此，您可以使用“触发文件”，这是一个特殊文件，当您要实际触发重新启动检查时必须对其进行修改。

|      | 对文件的任何更新都会触发检查，但是只有在Devtools检测到有事情要做的情况下，重启才会真正发生。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要使用触发文件，请将`spring.devtools.restart.trigger-file`属性设置为触发文件的名称（不包括任何路径）。触发器文件必须出现在类路径上的某个位置。

例如，如果您的项目具有以下结构：

```
src 
+-主
   +-资源
      +-.reloadtrigger
```

那么您的`trigger-file`  property将是：

```properties
spring.devtools.restart.trigger-file=.reloadtrigger
```

现在，仅在`src/main/resources/.reloadtrigger`更新时才会重新启动。

|      | 您可能希望将其设置`spring.devtools.restart.trigger-file`为[全局设置](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-globalsettings)，以便所有项目都以相同的方式运行。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

某些IDE具有使您不必手动更新触发器文件的功能。 [Eclipse的Spring工具](https://spring.io/tools)和[IntelliJ IDEA（最终版）](https://www.jetbrains.com/idea/)都具有这种支持。使用Spring Tools，您可以从控制台视图中使用“重新加载”按钮（只要您`trigger-file`名为`.reloadtrigger`）。对于IntelliJ，您可以按照[其文档中](https://www.jetbrains.com/help/idea/spring-boot.html#configure-application-update-policies-with-devtools)的[说明进行操作](https://www.jetbrains.com/help/idea/spring-boot.html#configure-application-update-policies-with-devtools)。

#### 8.2.6 自定义重启类加载器

如前面的“ [重新启动与重新加载”](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-spring-boot-restart-vs-reload)部分所述，重新启动功能是通过使用两个类加载器实现的。对于大多数应用程序，此方法效果很好。但是，有时可能会导致类加载问题。

默认情况下，IDE中任何打开的项目都使用“重新启动”类加载器加载，而任何常规`.jar`文件都使用“基本”类加载器加载。如果您在多模块项目上工作，并且并非每个模块都导入到IDE中，则可能需要自定义内容。为此，您可以创建一个`META-INF/spring-devtools.properties`文件。

该`spring-devtools.properties`文件可以包含以`restart.exclude`和开头的属性`restart.include`。该`include`元素是应该被拉高到“重启”的类加载器的项目，以及`exclude`要素是应该向下推入“基地”类加载器的项目。该属性的值是一个应用于类路径的正则表达式模式，如以下示例所示：

```properties
restart.exclude.companycommonlibs=/mycorp-common-[\\w\\d-\.]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w\\d-\.]+\.jar
```

|      | 所有属性键都必须是唯一的。只要一个属性以`restart.include.`或开头`restart.exclude.`就可以考虑。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 所有`META-INF/spring-devtools.properties`从类路径加载。您可以将文件打包到项目内部或项目使用的库中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 8.2.7 已知局限性

重新启动功能不适用于通过使用标准反序列化的对象`ObjectInputStream`。如果你需要反序列化的数据，你可能需要使用Spring的`ConfigurableObjectInputStream`结合`Thread.currentThread().getContextClassLoader()`。

不幸的是，一些第三方库在不考虑上下文类加载器的情况下反序列化。如果发现这样的问题，则需要向原始作者请求修复。

### 8.3 LiveReload

该`spring-boot-devtools`模块包括一个嵌入式LiveReload服务器，该服务器可用于在更改资源时触发浏览器刷新。可从[livereload.com](http://livereload.com/extensions/)免费获得适用于Chrome，Firefox和Safari的LiveReload浏览器扩展。

如果您不想在应用程序运行时启动LiveReload服务器，则可以将`spring.devtools.livereload.enabled`属性设置为`false`。

|      | 一次只能运行一台LiveReload服务器。在启动应用程序之前，请确保没有其他LiveReload服务器正在运行。如果从IDE启动多个应用程序，则只有第一个具有LiveReload支持。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 8.4 全局设置

您可以通过将以下任何文件添加到`$HOME/.config/spring-boot`文件夹来配置全局devtools设置：

1. `spring-boot-devtools.properties`
2. `spring-boot-devtools.yaml`
3. `spring-boot-devtools.yml`

添加到这些文件的任何属性都将应用于使用devtools的计算机上的所有 Spring Boot应用程序。例如，要将重启配置为始终使用[触发文件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-restart-triggerfile)，您可以添加以下属性：

〜/ .config / spring-boot / spring-boot-devtools.properties

```properties
spring.devtools.restart.trigger-file=.reloadtrigger
```

|      | 如果在中找不到devtools配置文件`$HOME/.config/spring-boot`，则在`$HOME`文件夹的根目录中搜索是否存在`.spring-boot-devtools.properties`文件。这使您可以与不支持该`$HOME/.config/spring-boot`位置的旧版Spring Boot上的应用程序共享devtools全局配置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 在上述文件中激活的配置文件不会影响[特定于配置文件的配置文件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-profile-specific-properties)的加载。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 8.5 远程应用

Spring Boot开发人员工具不仅限于本地开发。远程运行应用程序时，您还可以使用多种功能。选择启用远程支持，因为启用它可能会带来安全风险。仅当在受信任的网络上运行或使用SSL保护时，才应启用它。如果这两个选项都不可用，则不应使用DevTools的远程支持。您永远不要在生产部署上启用支持。

要启用它，您需要确保已将`devtools`其包含在重新打包的存档中，如以下清单所示：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludeDevtools>false</excludeDevtools>
            </configuration>
        </plugin>
    </plugins>
</build>
```

然后，您需要设置`spring.devtools.remote.secret`属性。像任何重要的密码或机密一样，该值应唯一且强壮，以免被猜测或强行使用。

远程devtools支持分为两个部分：接受连接的服务器端端点和在IDE中运行的客户端应用程序。`spring.devtools.remote.secret`设置属性后，将自动启用服务器组件。客户端组件必须手动启动。

#### 8.5.1 运行远程客户端应用程序

远程客户端应用程序旨在在您的IDE中运行。您需要`org.springframework.boot.devtools.RemoteSpringApplication`使用与所连接的远程项目相同的类路径来运行。应用程序的唯一必需参数是它连接到的远程URL。

例如，如果您使用的是Eclipse或STS，并且有一个`my-app`已部署到Cloud Foundry的项目，则应执行以下操作：

- 选择`Run Configurations…`从`Run`菜单。
- 创建一个新的`Java Application`“启动配置”。
- 浏览该`my-app`项目。
- 使用`org.springframework.boot.devtools.RemoteSpringApplication`作为主类。
- 添加`https://myapp.cfapps.io`到`Program arguments`（或任何您的远程URL）。

正在运行的远程客户端可能类似于以下清单：

```
  。____ _ __ _ _ 
 / \\ / ___'_ __ _ _（_）_ __ __ _ ___ _ \ \ \ \ 
（（）\ ___ |'_ |'_ | |'_ \ / _` | | _ \ ___ _ __ ___ | | _ ___ \ \ \ \ 
 \\ / ___）| | _）| | | | | || （_ | [] :::::: [] / -_）'\ / _ \ _ / -_））））））
  '| ____ | .__ | _ | | _ | _ | | _ \ __，| | _ | _ \ ___ | _ | _ | _____ / \ __ \ ___ | / / / = 
 ========= __ | ============= | ___ / ================================= /////// 
 :: Spring Boot Remote :: 2.2.6。发布

2015-06-10 18：25：06.632信息14938 --- [main] osbdevtools.RemoteSpringApplication：使用PID 14938在pwmbp上启动RemoteSpringApplication（/ Users / pwebb / projects / spring-boot / code / spring-boot-project / spring -boot-devtools / target /由pwebb在/ Users / pwebb / projects / spring-boot / code中启动的类）
2015-06-10 18：25：06.671 INFO 14938 --- [main] scaAnnotationConfigApplicationContext：刷新org.spring 框架.context.annotation.AnnotationConfigApplicationContext @ 2a17b7b6：启动日期[PDT 2015年6月10日星期三18:25:06]；上下文层次结构的根
2015-06-10 18：25：07.043 WARN 14938 --- [main] osbdrcRemoteClientConfiguration：与http：// localhost：8080的连接不安全。您应使用以“ https：//”开头的URL。
2015-06-10 18：25：07.074 INFO 14938 --- [main] osbdaOptionalLiveReloadServer：LiveReload服务器在端口35729上运行
2015-06-10 18：25：07.130 INFO 14938 --- [main] osbdevtools.RemoteSpringApplication：已启动RemoteSpringApplication在0.74秒内运行（JVM运行1.105）
```

|      | 因为远程客户端使用与真实应用程序相同的类路径，所以它可以直接读取应用程序属性。这就是`spring.devtools.remote.secret`读取属性并将其传递给服务器进行身份验证的方式。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 始终建议将其`https://`用作连接协议，以便对通信进行加密并且不能截获密码。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果您需要使用代理来访问远程应用程序，请配置`spring.devtools.remote.proxy.host`和`spring.devtools.remote.proxy.port`属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 8.5.2 远程更新

远程客户端以与[本地重新启动](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-restart)相同的方式监视应用程序类路径中的更改。任何更新的资源都会推送到远程应用程序，并且（*如果需要*）会触发重新启动。如果您迭代使用本地没有的云服务的功能，这将很有帮助。通常，远程更新和重新启动比完整的重建和部署周期要快得多。

|      | 仅在远程客户端正在运行时监视文件。如果在启动远程客户端之前更改文件，则不会将其推送到远程服务器。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 8.5.3 配置文件系统观察器

[FileSystemWatcher的](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/filewatch/FileSystemWatcher.java)工作方式是按一定时间间隔轮询类更改，然后等待预定义的静默期以确保没有更多更改。然后将更改上传到远程应用程序。在较慢的开发环境中，可能会发生静默期不够的情况，并且类中的更改可能会分为几批。第一批类更改上传后，服务器将重新启动。由于服务器正在重新启动，因此下一批不能发送到应用程序。

这通常通过`RemoteSpringApplication`日志中关于无法上载某些类的警告以及随后的重试来表明。但是，这也可能导致应用程序代码不一致，并且在上传第一批更改后无法重新启动。

如果您经常观察到此类问题，请尝试将`spring.devtools.restart.poll-interval`和`spring.devtools.restart.quiet-period`参数增加到适合您的开发环境的值：

```properties
spring.devtools.restart.poll-interval=2s
spring.devtools.restart.quiet-period=1s
```

现在每2秒轮询一次受监视的classpath文件夹是否有更改，并保持1秒钟的静默时间以确保没有其他类更改。

## 9.打包生产应用

可执行jar可以用于生产部署。由于它们是独立的，因此它们也非常适合基于云的部署。

对于其他“生产准备就绪”功能，例如运行状况，审核和度量REST或JMX端点，请考虑添加`spring-boot-actuator`。有关详细信息，请参见*[production-ready-features.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready)*。

## 10.下一步阅读

现在，您应该了解如何使用Spring Boot以及应遵循的一些最佳实践。现在，您可以继续深入了解特定的*[Spring Boot功能](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features)*，或者可以跳过并阅读有关Spring Boot 的“ [生产就绪](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready) ”方面的信息。

最后更新时间2020-03-26 11:43:46 UTC





# Spring Boot自动配置原理

​	Spring Boot自动配置会尝试根据添加的jar依赖关系自动配置Spring应用程序。需要通过将`@EnableAutoConfiguration`或`@SpringBootApplication`注释添加到一个`@Configuration`类中来选择自动配置。

## @SpringBootApplication组件扫描和自动配置

典型的Spring Boot应用的启动类一般均位于 src/main/java根路径下，比如 MoonApplication类：

```java
@SpringBootApplication
publicclassMoonApplication{
	public static void main(String[] args) {
	SpringApplication.run(MoonApplication.class, args);
	}
}
```

**@SpringBootApplication**开启组件扫描和自动配置，而 SpringApplication.run则负责启动引导应用程序。@SpringBootApplication是一个复合 Annotation，它将三个有用的注解组合在一起：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
	@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
	@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication{
// ......
}
```
### @SpringBootConfiguration

​	@SpringBootConfiguration就是 @Configuration，它是Spring框架的注解，标明该类是一个 JavaConfig配置类。

### @ComponentScan

​	@ComponentScan启用组件扫描。

### @EnableAutoConfiguration

​	注解表示开启Spring Boot自动配置功能，Spring Boot会根据应用的依赖、自定义的bean、classpath下有没有某个类 等等因素来猜测你需要的bean，然后注册到IOC容器中。

#### @AutoConfigurationPackage

 	自动扫描当前路径下的包，所以启动类一般都在最外层

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
    Class<?>[] exclude() default {};
    String[] excludeName() default {};
}
```

#### AutoConfigurationImportSelector

​	@Import注解用于导入该类，并将这个类作为一个bean的定义注册到容器中，这里它将把 **EnableAutoConfigurationImportSelector**作为bean注入到容器中，而这个类会将所有符合条件的**@Configuration**配置都加载到容器中。

##### selectImports(AnnotationMetadata）返回所有的配置类 configurations

​	selectImports方法是在容器启动过程中执行：**AbstractApplicationContext.refresh()**。

​	选择并返回应基于导入Configuration类的 AnnotationMetadata 导入哪些类的名称，然后Spring会加载这些类

```java
//DeferredImportSelector处理 EnableAutoConfiguration 自动配置。
//如果需要 @EnableAutoConfiguration 的自定义变体，则也可以将该类作为子类
//@see EnableAutoConfiguration
public class AutoConfigurationImportSelector
		implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware,
		BeanFactoryAware, EnvironmentAware, Ordered {
//选择并返回应基于导入Configuration类的 AnnotationMetadata 导入哪些类的名称，然后Spring会加载这些类
public String[] selectImports(AnnotationMetadata annotationMetadata) {
  //传入beanClassLoader，读取META-INF/spring-autoconfigure-metadata.properties下所有类  
  //由属性文件支持的{@link AutoConfigurationMetadata}实现。
AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader);
  //根据导入的 @Configuration类的AnnotationMetadata 返回 AutoConfigurationEntry
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(
				autoConfigurationMetadata, annotationMetadata);
         //返回所有的配置类 configurations
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    
// 这句代码被封装在AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
List<String> factories = newArrayList<String>(newLinkedHashSet<String>(
SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class, this.beanClassLoader)));
}
```

这个类会扫描所有的jar包，将所有符合条件的**@Configuration**配置类注入的容器中，何为符合条件，下面spring.factories文件中的自动配置类

### spring-boot-autoconfigure/META-INF/spring.factories

```xml
// 配置的key = EnableAutoConfiguration，与代码中一致
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration\
```

以 DataSourceAutoConfiguration为例，看看Spring Boot是如何自动配置的：

```java
@Configuration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class})
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ Registrar.class, DataSourcePoolMetadataProvidersConfiguration.class})
public class DataSourceAutoConfiguration{
}
```

分别说一说：

**@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })**：当Classpath中存在DataSource或者EmbeddedDatabaseType类时才启用这个配置，否则这个配置将被忽略。

**@EnableConfigurationProperties(DataSourceProperties.class)**：将DataSource的默认配置类注入到IOC容器中，DataSourceproperties定义为：

```java
// 提供对datasource配置信息的支持，所有的配置前缀为：spring.datasource
@ConfigurationProperties(prefix = "spring.datasource")
public classDataSourceProperties{
    private ClassLoader classLoader;
    private Environment environment;
    private String name = "testdb";
	......
}
```

**@Import({ Registrar.class, DataSourcePoolMetadataProvidersConfiguration.class })**：导入其他额外的配置，就以 DataSourcePoolMetadataProvidersConfiguration为例吧。

```java
@Configuration
public class DataSourcePoolMetadataProvidersConfiguration{

@Configuration
@ConditionalOnClass(org.apache.tomcat.jdbc.pool.DataSource.class)
static class TomcatDataSourcePoolMetadataProviderConfiguration{
@Bean
public DataSourcePoolMetadataProvider tomcatPoolDataSourceMetadataProvider() {
.....
}
}
```

**DataSourcePoolMetadataProvidersConfiguration**是数据库连接池提供者的一个配置类，即Classpath中存在 org.apache.tomcat.jdbc.pool.**DataSource.class**，则使用**tomcat-jdbc连接池**，**如果Classpath中存在 HikariDataSource.class则使用Hikari连接池**

这里仅描述了DataSourceAutoConfiguration的冰山一角，但足以说明Spring Boot如何利用条件话配置来实现自动配置的。回顾一下， @EnableAutoConfiguration中导入了EnableAutoConfigurationImportSelector类，而这个类的 selectImports()通过SpringFactoriesLoader得到了大量的配置类，而每一个配置类则根据条件化配置来做出决策，以实现自动配置。

# Eureka 启动分析

## @EnableEurekaServer开启注册中心服务

```java
@SpringBootApplication
@EnableEurekaServer
public class SpringCloudEurekaApplication {
    public static void main(String[] args) {
      SpringApplication.run(SpringCloudEurekaApplication.class, args);
    }

}
```

## **@EnableEurekaServer**注解

​	导入配置类EurekaServerMarkerConfiguration

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(EurekaServerMarkerConfiguration.class)
public @interface EnableEurekaServer {

}
```

## **EurekaServerMarkerConfiguration**配置类

​	负责添加标记bean以激活  **EurekaServerAutoConfiguration**

```java
@Configuration
public class EurekaServerMarkerConfiguration {

   @Bean
   public Marker eurekaServerMarkerBean() {
      return new Marker();
   }

   class Marker {
   }
}
```

## spring-cloud-netflix-eureka-server/META-INF/spring.factories

**EnableAutoConfiguration = EurekaServerAutoConfiguration**

```xml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.cloud.netflix.eureka.server.EurekaServerAutoConfiguration
```

## EurekaServerAutoConfiguration自动配置类

```java
@Configuration
//导入EurekaServerInitializerConfiguration类
@Import(EurekaServerInitializerConfiguration.class)
//EurekaServerMarkerConfiguration.Marker存在时才可用
@ConditionalOnBean(EurekaServerMarkerConfiguration.Marker.class)
@EnableConfigurationProperties({ EurekaDashboardProperties.class,
      InstanceRegistryProperties.class })
@PropertySource("classpath:/eureka/server.properties")
public class EurekaServerAutoConfiguration extends WebMvcConfigurerAdapter {
   //包含Eureka服务器所需的Jersey资源的软件包列表
   private static final String[] EUREKA_PACKAGES = new String[] { "com.netflix.discovery","com.netflix.eureka" };
```



### jerseyFilterRegistration（Application）注册Jersey过滤器

```java
@Bean
public FilterRegistrationBean jerseyFilterRegistration(
      javax.ws.rs.core.Application eurekaJerseyApp) {
   FilterRegistrationBean bean = new FilterRegistrationBean();
   bean.setFilter(new ServletContainer(eurekaJerseyApp));
   bean.setOrder(Ordered.LOWEST_PRECEDENCE);
   bean.setUrlPatterns(
   Collections.singletonList(EurekaConstants.DEFAULT_PREFIX + "/*"));
   return bean;
}
```

### jerseyApplication(Environment,ResourceLoader)

使用Eureka服务器所需的所有资源构建Jersey Application

```java
@Bean
public javax.ws.rs.core.Application jerseyApplication(Environment environment,ResourceLoader resourceLoader) {
  
   ClassPathScanningCandidateComponentProvider provider = new ClassPathScanningCandidateComponentProvider(
         false, environment);
   // 过滤以仅包括具有特定注释的类
   provider.addIncludeFilter(new AnnotationTypeFilter(Path.class));
   provider.addIncludeFilter(new AnnotationTypeFilter(Provider.class));

   // 在Eureka包（或子包）中查找类
   Set<Class<?>> classes = new HashSet<>();
   for (String basePackage : EUREKA_PACKAGES) {
      Set<BeanDefinition> beans = provider.findCandidateComponents(basePackage);
      for (BeanDefinition bd : beans) {
         Class<?> cls = ClassUtils.resolveClassName(bd.getBeanClassName(),
               resourceLoader.getClassLoader());
         classes.add(cls);
      }
   }

   // 构造Jersey ResourceConfig
   Map<String, Object> propsAndFeatures = new HashMap<>();
   propsAndFeatures.put(
         // 跳过网络应用使用的静态内容
     ServletContainer.PROPERTY_WEB_PAGE_CONTENT_REGEX,
     EurekaConstants.DEFAULT_PREFIX + "/(fonts|images|css|js)/.*");

   DefaultResourceConfig rc = new DefaultResourceConfig(classes);
   rc.setPropertiesAndFeatures(propsAndFeatures);

   return rc;
}
```



## @EnableAutoConfiguration

​	启用Spring Application Context的自动配置，尝试猜测和配置您可能需要的bean。自动配置类通常根据您的类路径和定义的bean应用。如果您的类路径上有 tomcat-embedded.jar，则可能需要 TomcatServletWebServerFactory（除非定义了自己的ServletWebServerFactory bean）。

​	使用 SpringBootApplication 时，会自动启用上下文的自动配置，因此添加此批注不会产生任何其他影响

​	自动配置会尝试尽可能智能化，并且会在您定义更多自己的配置时回退。您始终可以手动 exclude（）不要应用的任何配置（如果您没有访问权限，使用 excludeName（））。您也可以通过 { spring.autoconfigure.exclude}属性排除它们。在注册用户定义的bean之后，始终会应用自动配置。

​	用 @EnableAutoConfiguration 注释的类的包通常通过 @SpringBootApplication进行注释，具有特定的意义，并且经常用作“默认值”。例如，在扫描{ @Entity}类时将使用它。 通常建议将{ @EnableAutoConfiguration}（如果您不使用{ @SpringBootApplication}）放在根包中，以便可以搜索所有子包和类。

​	自动配置类是常规的Spring  Configuration bean。 使用 SpringFactoriesLoader 机制定位（针对此类）。 通常，自动配置bean是 @Conditional bean（大多数经常使用 @ConditionalOnClass 和 @ConditionalOnMissingBean 注释）。

```java
/**@see ConditionalOnBean
 * @see ConditionalOnMissingBean
 * @see ConditionalOnClass
 * @see AutoConfigureAfter
 * @see SpringBootApplication
 */
@AutoConfigurationPackage //扫描当前路径下所有配置类
@Import(AutoConfigurationImportSelector.class)//导入配置类
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY= "spring.boot.enableautoconfiguration";

   /**
    * 排除特定的自动配置类，使其永远不会应用。
    * 返回要排除的类
    */
   Class<?>[] exclude() default {};

   /**
    * 排除特定的自动配置类名称，以使它们永远不会被应用
    * @返回要排除的类名
    * @since 1.3.0
    */
   String[] excludeName() default {};

}
```



## AutoConfigurationImportSelector配置类

​	DeferredImportSelector 处理 EnableAutoConfiguration 自动配置。如果需要  @EnableAutoConfiguration的自定义变体，则也可以将该类作为子类。

### selectImports（AnnotationMetadata）返回所有的配置类 configurations

​	selectImports方法是在容器启动过程中执行：**AbstractApplicationContext.refresh()**。

​	选择并返回应基于导入 Configuration 类的 AnnotationMetadata导入哪些类的名称。

```java
public class AutoConfigurationImportSelector
      implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware,
      BeanFactoryAware, EnvironmentAware, Ordered {
        
	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader);
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(
				autoConfigurationMetadata, annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}     
        
```

![image-20200504002337743](/Users/houshaojie/Library/Application Support/typora-user-images/image-20200504002337743.png)

![image-20200504003802273](/Users/houshaojie/Library/Application Support/typora-user-images/image-20200504003802273.png)

