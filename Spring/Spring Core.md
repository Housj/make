# 核心技术



版本5.2.6.RELEASE https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans

[返回索引](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/index.html)

参考文档的这一部分涵盖了Spring框架绝对必要的所有技术。

其中最重要的是Spring框架的控制反转（IoC）容器。对Spring框架的IoC容器进行彻底处理之后，将全面介绍Spring的面向方面编程（AOP）技术。Spring框架拥有自己的AOP框架，该框架在概念上易于理解，并且成功解决了Java企业编程中AOP要求的80％的难题。

还提供了Spring与AspectJ的集成（就功能而言，目前是最丰富的-当然肯定是Java企业领域中最成熟的AOP实现）。

## 1. IoC容器

本章介绍了Spring的控制反转（IoC）容器。

### 1.1。Spring IoC容器和Bean简介

本章介绍了控制反转（IoC）原理的Spring框架实现。IoC也称为依赖注入（DI）。在此过程中，对象仅通过构造函数参数，工厂方法的参数或在构造或从工厂方法返回后在对象实例上设置的属性来定义其依赖项（即，与它们一起使用的其他对象） 。然后，容器在创建bean时注入那些依赖项。此过程从根本上讲是通过使用类的直接构造或诸如服务定位器模式的机制来控制其依赖项的实例化或位置的Bean本身的逆过程（因此称为Control Inversion）。

在`org.springframework.beans`和`org.springframework.context`包是Spring框架的IoC容器的基础。该 [`BeanFactory`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html) 界面提供了一种高级配置机制，能够管理任何类型的对象。 [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html) 是的子接口`BeanFactory`。它增加了：

- 与Spring的AOP功能轻松集成
- 消息资源处理（用于国际化）
- 活动发布
- 应用层特定的上下文，例如`WebApplicationContext` 用于Web应用程序中的。

简而言之，`BeanFactory`提供了配置框架和基本功能，并`ApplicationContext`增加了更多针对企业的功能。该`ApplicationContext`是对一个完整的超集`BeanFactory`，并在Spring的IoC容器的描述本章独占使用。有关使用的详细信息`BeanFactory`，而不是`ApplicationContext,`看到 [的`BeanFactory`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-beanfactory)。

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。Bean是由Spring IoC容器实例化，组装和以其他方式管理的对象。否则，bean仅仅是应用程序中许多对象之一。Bean及其之间的依赖关系反映在容器使用的配置元数据中。

### 1.2。容器概述

该`org.springframework.context.ApplicationContext`接口代表Spring IoC容器，并负责实例化，配置和组装Bean。容器通过读取配置元数据来获取有关要实例化，配置和组装哪些对象的指令。配置元数据以XML，Java批注或Java代码表示。它使您能够表达组成应用程序的对象以及这些对象之间的丰富相互依赖关系。

`ApplicationContext`Spring提供了该接口的几种实现。在独立应用程序中，通常创建[`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) 或的实例 [`FileSystemXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html)。尽管XML是定义配置元数据的传统格式，但是您可以通过提供少量XML配置来声明性地启用对这些其他元数据格式的支持，从而指示容器将Java注释或代码用作元数据格式。

在大多数应用场景中，不需要显式的用户代码来实例化一个Spring IoC容器的一个或多个实例。例如，在Web应用程序场景中，应用程序文件中的简单八行（约）样板Web描述符XML `web.xml`通常就足够了（请参阅[Web应用程序的便捷ApplicationContext实例化](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#context-create)）。如果您使用 [Spring Tools for Eclipse](https://spring.io/tools)（Eclipse支持的开发环境），则只需单击几下鼠标或击键即可轻松创建此样板配置。

下图显示了Spring的工作原理的高级视图。您的应用程序类与配置元数据结合在一起，以便在`ApplicationContext`创建和初始化之后，您便拥有了一个完全配置且可执行的系统或应用程序。

![容器魔术](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/images/container-magic.png)

图1. Spring IoC容器

#### 1.2.1。配置元数据

如上图所示，Spring IoC容器使用一种形式的配置元数据。此配置元数据表示您作为应用程序开发人员如何告诉Spring容器实例化，配置和组装应用程序中的对象。

传统上，配置元数据以简单直观的XML格式提供，这是本章大部分内容用来传达Spring IoC容器的关键概念和功能的内容。

|      | 基于XML的元数据不是配置元数据的唯一允许形式。Spring IoC容器本身与实际写入此配置元数据的格式完全脱钩。如今，许多开发人员为他们的Spring应用程序选择 [基于Java的配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

有关在Spring容器中使用其他形式的元数据的信息，请参见：

- [基于注释的配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-annotation-config)：Spring 2.5引入了对基于注释的配置元数据的支持。
- [基于Java的配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java)：从Spring 3.0开始，Spring JavaConfig项目提供的许多功能成为核心Spring Framework的一部分。因此，您可以使用Java而不是XML文件来定义应用程序类外部的bean。要使用这些新功能，请参阅 [`@Configuration`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)， [`@Bean`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html)， [`@Import`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html)，和[`@DependsOn`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html)注释。

Spring配置由容器必须管理的至少一个（通常是一个以上）bean定义组成。基于XML的配置元数据将这些bean配置为``顶级元素内的``元素。Java配置通常`@Bean`在`@Configuration`类中使用带注释的方法。

这些bean定义对应于组成应用程序的实际对象。通常，您定义服务层对象，数据访问对象（DAO），表示对象（例如Struts `Action`实例），基础结构对象（例如Hibernate `SessionFactories`，JMS `Queues`等）。通常，不会在容器中配置细粒度的域对象，因为创建和加载域对象通常是DAO和业务逻辑的职责。但是，您可以使用Spring与AspectJ的集成来配置在IoC容器的控制范围之外创建的对象。请参阅[使用AspectJ通过Spring依赖注入域对象](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-atconfigurable)。

以下示例显示了基于XML的配置元数据的基本结构：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

|      | 该`id`属性是标识单个bean定义的字符串。              |
| ---- | --------------------------------------------------- |
|      | 该`class`属性定义bean的类型，并使用完全限定的类名。 |

该`id`属性的值是指协作对象。在此示例中未显示用于引用协作对象的XML。有关更多信息，请参见 [依赖项](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-dependencies)。

#### 1.2.2。实例化容器

提供给`ApplicationContext`构造函数的位置路径是资源字符串，这些资源字符串使容器可以从各种外部资源（例如本地文件系统，Java等）加载配置元数据`CLASSPATH`。

爪哇

科特林

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

|      | 在了解了Spring的IoC容器之后，您可能想了解更多有关Spring的 `Resource`抽象（如[参考资料中所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources)），它提供了一种方便的机制，用于从URI语法中定义的位置读取InputStream。特别`Resource`是，如[应用程序上下文和资源路径中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-app-ctx)所述， 路径用于构造应用[程序上下文](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-app-ctx)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下示例显示了服务层对象`(services.xml)`配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

以下示例显示了数据访问对象`daos.xml`文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

在前面的示例中，服务层由的`PetStoreServiceImpl`类和类型的两个数据访问对象`JpaAccountDao`和`JpaItemDao`（基于JPA对象关系映射标准）。该`property name`元素是指JavaBean属性的名称，以及`ref`元素指的是另一个bean定义的名称。`id`和`ref`元素之间的这种联系表达了协作对象之间的依赖性。有关配置对象的依赖关系的详细信息，请参见 [依赖关系](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-dependencies)。

##### 组成基于XML的配置元数据

使bean定义跨越多个XML文件可能很有用。通常，每个单独的XML配置文件都代表体系结构中的逻辑层或模块。

您可以使用应用程序上下文构造函数从所有这些XML片段中加载bean定义。`Resource`如上[一节中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-instantiation)所示，该构造函数具有多个位置 。或者，使用一个或多个出现的``元素从另一个文件中加载bean定义。以下示例显示了如何执行此操作：

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

在前面的例子中，外部豆定义是从三个文件加载： `services.xml`，`messageSource.xml`，和`themeSource.xml`。所有位置路径都相对于进行导入的定义文件，因此`services.xml`必须与进行导入的文件位于同一目录或类路径位置， `messageSource.xml`并且`themeSource.xml`必须位于`resources`导入文件位置下方的位置。如您所见，斜杠被忽略。但是，鉴于这些路径是相对的，最好不要使用任何斜线。``根据Spring Schema，导入的文件的内容（包括顶层元素）必须是有效的XML bean定义。

|      | 可以但不建议使用相对的“ ../”路径引用父目录中的文件。这样做会创建对当前应用程序外部文件的依赖。特别是，不建议将该引用用于`classpath:`URL（例如`classpath:../services.xml`），在URL 中，运行时解析过程将选择“最近”的类路径根，然后查看其父目录。类路径配置的更改可能导致选择其他错误的目录。您始终可以使用完全限定的资源位置来代替相对路径：例如`file:C:/config/services.xml`或`classpath:/config/services.xml`。但是，请注意，您正在将应用程序的配置耦合到特定的绝对位置。通常最好为这样的绝对位置保留一个间接寻址-例如，通过在运行时针对JVM系统属性解析的“ $ {…}”占位符。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

命名空间本身提供了导入指令功能。Spring提供的一系列XML名称空间（例如`context`和`util`名称空间）中提供了超出普通bean定义的其他配置功能。

##### Groovy Bean定义DSL

作为外部化配置元数据的另一个示例，Bean定义也可以在Spring的Groovy Bean定义DSL中表达，如Grails框架所知。通常，这种配置位于“ .groovy”文件中，其结构如以下示例所示：

```groovy
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

这种配置样式在很大程度上等同于XML bean定义，甚至支持Spring的XML配置名称空间。它还允许通过`importBeans`指令导入XML bean定义文件。

#### 1.2.3。使用容器

该`ApplicationContext`是一个维护bean定义以及相互依赖的注册表的高级工厂的接口。通过使用方法 `T getBean(String name, Class requiredType)`，您可以检索bean的实例。

将`ApplicationContext`让你读bean定义和访问它们，如下例所示：

爪哇

科特林

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

使用Groovy配置，引导看起来非常相似。它有一个不同的上下文实现类，该类可识别Groovy（但也了解XML Bean定义）。以下示例显示了Groovy配置：

爪哇

科特林

```java
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```

最灵活的变体是`GenericApplicationContext`与读取器委托结合使用，例如，与`XmlBeanDefinitionReader`XML文件结合使用，如以下示例所示：

爪哇

科特林

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

您也可以将`GroovyBeanDefinitionReader`Groovy文件用于，如以下示例所示：

爪哇

科特林

```java
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

您可以`ApplicationContext`在不同的配置源中从相同的Bean中混合和匹配此类阅读器委托，以读取Bean定义。

然后，您可以`getBean`用来检索bean的实例。该`ApplicationContext` 接口还有其他几种检索bean的方法，但是理想情况下，您的应用程序代码永远不要使用它们。实际上，您的应用程序代码应该根本不调用该 `getBean()`方法，因此完全不依赖于Spring API。例如，Spring与Web框架的集成为各种Web框架组件（例如控制器和JSF管理的Bean）提供了依赖注入，使您可以通过元数据（例如自动装配注释）声明对特定Bean的依赖。

### 1.3。Bean总览

Spring IoC容器管理一个或多个bean。这些bean是使用您提供给容器的配置元数据创建的（例如，以XML ``定义的形式 ）。

在容器本身内，这些bean定义表示为`BeanDefinition` 对象，这些对象包含（除其他信息外）以下元数据：

- 包限定的类名：通常，定义了Bean的实际实现类。
- Bean行为配置元素，用于声明Bean在容器中的行为（作用域，生命周期回调等）。
- 引用其他bean完成其工作所需的bean。这些引用也称为协作者或依赖项。
- 要在新创建的对象中设置的其他配置设置-例如，池的大小限制或要在管理连接池的bean中使用的连接数。

此元数据转换为构成每个bean定义的一组属性。下表描述了这些属性：

| 属性           | 在...中解释                                                  |
| :------------- | :----------------------------------------------------------- |
| 类             | [实例化豆](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-class) |
| 名称           | [命名豆](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-beanname) |
| 范围           | [豆范围](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes) |
| 构造函数参数   | [依赖注入](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-collaborators) |
| 物产           | [依赖注入](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-collaborators) |
| 自动接线方式   | [自动装配协作器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-autowire) |
| 延迟初始化模式 | [懒初始化豆](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lazy-init) |
| 初始化方法     | [初始化回调](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) |
| 销毁方式       | [销毁回调](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean) |

除了包含有关如何创建特定bean的信息的bean定义之外，`ApplicationContext`实现还允许注册在容器外部（由用户）创建的现有对象。这是通过通过方法访问ApplicationContext的BeanFactory来完成的`getBeanFactory()`，该方法返回BeanFactory `DefaultListableBeanFactory`实现。`DefaultListableBeanFactory` 通过`registerSingleton(..)`和 `registerBeanDefinition(..)`方法支持此注册。但是，典型的应用程序只能与通过常规bean定义元数据定义的bean一起使用。

|      | Bean元数据和手动提供的单例实例需要尽早注册，以便容器在自动装配和其他自省步骤中正确地推理它们。虽然在某种程度上支持覆盖现有的元数据和现有的单例实例，但是在运行时（与对工厂的实时访问同时）对新bean的注册未得到正式支持，并且可能导致并发访问异常，bean容器中的状态不一致或都。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.3.1。命名豆

每个bean具有一个或多个标识符。这些标识符在承载Bean的容器内必须唯一。一个bean通常只有一个标识符。但是，如果需要多个，则可以将多余的别名视为别名。

在基于XML配置文件，您可以使用`id`属性，该`name`属性，或两者来指定bean标识符。该`id`属性使您可以精确指定一个ID。按照惯例，这些名称是字母数字（“ myBean”，“ someService”等），但它们也可以包含特殊字符。如果要为bean引入其他别名，还可以在`name` 属性中指定它们，并用逗号（`,`），分号（`;`）或空格分隔。作为历史记录，在Spring 3.1之前的版本中，该`id`属性被定义为一种`xsd:ID`类型，该类型限制了可能的字符。从3.1开始，它被定义为`xsd:string`类型。请注意，Bean `id`唯一性仍由容器强制执行，尽管XML解析器不再如此。

您不需要为bean 提供`name`或`id`。如果不提供 `name`或`id`显式提供，容器将为该bean生成一个唯一的名称。但是，如果要通过名称引用该bean，则必须通过使用`ref`元素或服务定位器样式查找来提供名称。不提供名称的动机与使用[内部bean](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-inner-beans)和[自动装配合作者有关](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-autowire)。

Bean命名约定

约定是在命名bean时将标准Java约定用于实例字段名称。也就是说，bean名称以小写字母开头，并从那里用驼峰式大小写。这样的名字的例子包括`accountManager`， `accountService`，`userDao`，`loginController`，等等。

一致地命名Bean使您的配置更易于阅读和理解。另外，如果您使用Spring AOP，则在将建议应用于名称相关的一组bean时，它会很有帮助。

|      | 通过在类路径中进行组件扫描，Spring会按照前面描述的规则为未命名的组件生成Bean名称：本质上，采用简单的类名称并将其初始字符转换为小写。但是，在（不寻常的）特殊情况下，如果有多个字符并且第一个和第二个字符均为大写字母，则会保留原始大小写。这些规则与`java.beans.Introspector.decapitalize`（由Spring在此处使用）定义的规则相同。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 在Bean定义之外别名Bean

在bean定义本身中，可以通过使用由`id`属性指定的最多一个名称和属性中任意数量的其他名称的组合来为bean提供多个名称`name`。这些名称可以是同一个bean的等效别名，并且在某些情况下很有用，例如，通过使用特定于该组件本身的bean名称，让应用程序中的每个组件都引用一个公共依赖项。

但是，在实际定义bean的地方指定所有别名并不总是足够的。有时需要为在别处定义的bean引入别名。在大型系统中通常是这种情况，在大型系统中，配置是在每个子系统之间分配的，每个子系统都有自己的对象定义集。在基于XML的配置元数据中，您可以使用``元素来完成此任务。以下示例显示了如何执行此操作：

```xml
<alias name="fromName" alias="toName"/>
```

在这种情况下，`fromName`在使用该别名定义之后，命名为Bean（位于同一容器中）的豆也可以称为`toName`。

例如，子系统A的配置元数据可以使用名称来引用数据源`subsystemA-dataSource`。子系统B的配置元数据可以使用的名称引用数据源`subsystemB-dataSource`。组成使用这两个子系统的主应用程序时，主应用程序使用名称引用数据源`myApp-dataSource`。要使所有三个名称都引用同一个对象，可以将以下别名定义添加到配置元数据中：

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

现在，每个组件和主应用程序都可以通过唯一的名称引用数据源，并保证不与任何其他定义冲突（有效地创建名称空间），但它们引用的是同一bean。

Java配置

如果使用Java配置，则`@Bean`注释可用于提供别名。有关详细信息，请参见[使用`@Bean`注释](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-bean-annotation)。

#### 1.3.2。实例化豆

Bean定义实质上是创建一个或多个对象的方法。当被询问时，容器将查看命名bean的配方，并使用该bean定义封装的配置元数据来创建（或获取）实际对象。

如果使用基于XML的配置元数据，则`class`在``元素的属性中指定要实例化的对象的类型（或类）。此 `class`属性（在内部是 实例的`Class`属性`BeanDefinition`）通常是必需的。（有关异常，请参阅 [使用实例工厂方法实例化](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-class-instance-factory-method)和[Bean定义继承](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-child-bean-definitions)。）可以通过以下`Class`两种方式之一使用该属性：

- 通常，在容器本身通过反射性地调用其构造函数直接创建Bean的情况下，指定要构造的Bean类，这在某种程度上等同于使用`new`运算符的Java代码。
- 要指定包含`static`工厂方法的实际类，该方法被调用以创建对象，在不太常见的情况下，容器将`static`在类上调用 工厂方法来创建Bean。从`static`工厂方法的调用返回的对象类型可以是相同的类，也可以是完全不同的另一类。

内部班级名称

如果要为`static`嵌套类配置Bean定义，则必须使用嵌套类的二进制名称。

例如，如果`SomeThing`在`com.example`包中有一个名为的类，并且 `SomeThing`该类有一个`static`名为的嵌套类`OtherThing`，则`class` bean定义上的属性值将为`com.example.SomeThing$OtherThing`。

请注意，名称中使用了`$`字符，以将嵌套的类名与外部类名分开。

##### 用构造函数实例化

当通过构造函数方法创建bean时，所有普通类都可以被Spring使用并与之兼容。也就是说，正在开发的类不需要实现任何特定的接口或以特定的方式进行编码。只需指定bean类就足够了。但是，根据您用于该特定bean的IoC的类型，您可能需要一个默认（空）构造函数。

Spring IoC容器几乎可以管理您要管理的任何类。它不仅限于管理真正的JavaBean。大多数Spring用户更喜欢实际的JavaBean，它们仅具有默认（无参数）构造函数，并具有根据容器中的属性建模的适当的setter和getter。您还可以在容器中具有更多奇特的非bean样式类。例如，如果您需要使用绝对不符合JavaBean规范的旧式连接池，则Spring也可以对其进行管理。

使用基于XML的配置元数据，您可以如下指定bean类：

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

有关向构造函数提供参数（如果需要）并在构造对象之后设置对象实例属性的机制的详细信息，请参见 [注入依赖项](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-collaborators)。

##### 用静态工厂方法实例化

在定义使用静态工厂方法创建的bean时，请使用`class` 属性指定包含`static`工厂方法的类，并使用命名`factory-method`为属性的属性来指定工厂方法本身的名称。您应该能够调用此方法（使用可选参数，如稍后所述）并返回一个活动对象，该对象随后将被视为已通过构造函数创建。这种bean定义的一种用法是`static`用旧版代码调用工厂。

以下bean定义指定通过调用工厂方法来创建bean。该定义不指定返回对象的类型（类），而仅指定包含工厂方法的类。在此示例中，该`createInstance()` 方法必须是静态方法。以下示例显示如何指定工厂方法：

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

以下示例显示了一个可与前面的bean定义一起使用的类：

爪哇

科特林

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

有关为工厂方法提供（可选）参数并在从工厂返回对象之后设置对象实例属性的机制的详细信息，请参见[Dependencies and Configuration in Detail](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-properties-detailed)。

##### 使用实例工厂方法实例化

类似于通过[静态工厂方法进行](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-class-static-factory-method)实例化，使用实例工厂方法进行实例化会从容器中调用现有bean的非静态方法来创建新bean。要使用此机制，请将`class`属性保留为空，并在`factory-bean`属性中指定当前（或父或祖先）容器中的Bean名称，该容器包含将被调用以创建对象的实例方法。使用`factory-method`属性设置工厂方法本身的名称。以下示例显示了如何配置此类Bean：

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

以下示例显示了相应的类：

爪哇

科特林

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以包含一个以上的工厂方法，如以下示例所示：

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

以下示例显示了相应的类：

爪哇

科特林

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

这种方法表明，工厂Bean本身可以通过依赖项注入（DI）进行管理和配置。[详细信息，](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-properties-detailed)请参见[依赖性和配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-properties-detailed)。

|      | 在Spring文档中，“ factory bean”是指在Spring容器中配置并通过[实例](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-class-instance-factory-method)或 [静态](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-class-static-factory-method)工厂方法创建对象的bean 。相反， `FactoryBean`（注意大小写）是指特定于Spring的 [`FactoryBean` ](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-extension-factorybean)实现类。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 确定Bean的运行时类型

确定特定bean的运行时类型并非易事。Bean元数据定义中的指定类仅仅是初始类引用，可能与声明的工厂方法结合使用，或者是`FactoryBean`可能导致Bean的运行时类型不同的类，或者在实例的情况下根本不设置-级别工厂方法（通过指定`factory-bean`名称解析）。此外，AOP代理可以使用基于接口的代理包装Bean实例，而目标Bean的实际类型（仅是其实现的接口）的暴露程度有限。

找出特定bean的实际运行时类型的推荐方法是`BeanFactory.getType`调用指定的bean名称。这考虑了上述所有情况，并返回了`BeanFactory.getBean`要针对相同bean名称返回的对象的类型。

### 1.4。依存关系

典型的企业应用程序不包含单个对象（或Spring的bean）。即使是最简单的应用程序，也有一些对象可以协同工作，以呈现最终用户视为一致的应用程序。下一部分将说明如何从定义多个独立的Bean定义到实现对象协作以实现目标的完全实现的应用程序。

#### 1.4.1。依赖注入

依赖注入（DI）是一个过程，通过该过程，对象只能通过构造函数参数，工厂方法的参数或在构造或创建对象实例后在对象实例上设置的属性来定义其依赖关系（即，与它们一起工作的其他对象）。从工厂方法返回。然后，容器在创建bean时注入那些依赖项。从根本上讲，此过程是通过使用类的直接构造或服务定位器模式来控制bean自身依赖关系的实例化或位置的bean本身的逆过程（因此称为Control Inversion）。

使用DI原理，代码更加简洁，当为对象提供依赖项时，去耦会更有效。该对象不查找其依赖项，也不知道依赖项的位置或类。结果，您的类变得更易于测试，尤其是当依赖项依赖于接口或抽象基类时，它们允许在单元测试中使用存根或模拟实现。

DI存在两个主要变体：[基于构造函数的依赖注入](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-constructor-injection)和[基于Setter的依赖注入](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-setter-injection)。

##### 基于构造函数的依赖注入

基于构造函数的DI是通过容器调用具有多个参数（每个参数代表一个依赖项）的构造函数来完成的。调用`static`带有特定参数的工厂方法来构造Bean几乎是等效的，并且本次讨论将构造函数和`static`工厂方法的参数视为类似。以下示例显示了只能通过构造函数注入进行依赖项注入的类：

爪哇

科特林

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

注意，该类没有什么特别的。它是一个POJO，不依赖于特定于容器的接口，基类或注释。

###### 构造函数参数解析

构造函数参数解析匹配通过使用参数的类型进行。如果Bean定义的构造函数参数中不存在潜在的歧义，则在实例化Bean时，在Bean定义中定义构造函数参数的顺序就是将这些参数提供给适当的构造函数的顺序。考虑以下类别：

爪哇

科特林

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

假设`ThingTwo`和`ThingThree`类不是通过继承关联的，则不存在潜在的歧义。因此，以下配置可以正常工作，并且您无需在`` 元素中显式指定构造函数参数索引或类型。

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当引用另一个bean时，类型是已知的，并且可以发生匹配（与前面的示例一样）。当使用诸如的简单类型时 `true`，Spring无法确定值的类型，因此在没有帮助的情况下无法按类型进行匹配。考虑以下类别：

爪哇

科特林

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

构造函数参数类型匹配

在上述情况下，如果通过使用`type`属性显式指定构造函数参数的类型，则容器可以使用简单类型的类型匹配。如以下示例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

构造函数参数索引

您可以使用该`index`属性来明确指定构造函数参数的索引，如以下示例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

除了解决多个简单值的歧义性之外，指定索引还可以解决歧义，其中构造函数具有两个相同类型的参数。

|      | 索引从0开始。 |
| ---- | ------------- |
|      |               |

构造函数参数名称

您还可以使用构造函数参数名称来消除歧义，如以下示例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

请记住，要立即使用该功能，必须在启用调试标志的情况下编译代码，以便Spring可以从构造函数中查找参数名称。如果您不能或不希望使用debug标志编译代码，则可以使用 [@ConstructorProperties](https://download.oracle.com/javase/8/docs/api/java/beans/ConstructorProperties.html) JDK注释显式命名构造函数参数。然后，样本类必须如下所示：

爪哇

科特林

```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

##### 基于Setter的依赖注入

基于设置器的DI是通过在调用无参数构造函数或无参数`static`工厂方法以实例化您的bean 之后，在您的bean上调用setter方法来完成的。

下面的示例显示只能通过使用纯setter注入来依赖注入的类。此类是常规的Java。它是一个POJO，不依赖于容器特定的接口，基类或注释。

爪哇

科特林

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

将`ApplicationContext`支持基于构造函数和的Setter DI为它所管理的豆类。在已经通过构造函数方法注入了某些依赖项之后，它还支持基于setter的DI。您可以以的形式配置依赖项，将`BeanDefinition`其与`PropertyEditor`实例结合使用以将属性从一种格式转换为另一种格式。但是，大多数Spring用户并不直接（即以编程方式）使用这些类，而是使用XML `bean` 定义，带注释的组件（即带有`@Component`， `@Controller`等等的类）或`@Bean`基于Java的`@Configuration`类中的方法。然后将这些源在内部转换为的实例，`BeanDefinition`并用于加载整个Spring IoC容器实例。

基于构造函数或基于setter的DI？

由于可以混合使用基于构造函数的DI和基于setter的DI，因此，将构造函数用于强制性依赖项，将setter方法或配置方法用于可选性依赖项是一个很好的经验法则。请注意，可以 在setter方法上使用[@Required](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-required-annotation)批注，以使该属性成为必需的依赖项。但是，最好使用带有参数的程序验证的构造函数注入。

Spring团队通常提倡构造函数注入，因为它使您可以将应用程序组件实现为不可变对象，并确保不存在必需的依赖项`null`。此外，构造函数注入的组件始终以完全初始化的状态返回到客户端（调用）代码。附带说明一下，大量的构造函数自变量是一种不好的代码味道，这意味着该类可能承担了太多的职责，应该对其进行重构以更好地解决关注点分离问题。

Setter注入主要应仅用于可以在类中分配合理的默认值的可选依赖项。否则，必须在代码使用依赖项的任何地方执行非空检查。setter注入的一个好处是，setter方法可使该类的对象在以后重新配置或重新注入。因此，通过[JMX MBean进行](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#jmx)管理是用于setter注入的引人注目的用例。

使用最适合特定班级的DI风格。有时，在处理您没有源代码的第三方类时，将为您做出选择。例如，如果第三方类未公开任何setter方法，则构造函数注入可能是DI的唯一可用形式。

##### 依赖性解析过程

容器执行bean依赖项解析，如下所示：

- 使用`ApplicationContext`描述所有bean的配置元数据创建和初始化。可以通过XML，Java代码或注释指定配置元数据。
- 对于每个bean，其依赖项都以属性，构造函数参数或static-factory方法的参数的形式表示（如果使用它而不是普通的构造函数）。实际创建Bean时，会将这些依赖项提供给Bean。
- 每个属性或构造函数参数都是要设置的值的实际定义，或者是对容器中另一个bean的引用。
- 作为值的每个属性或构造函数参数都将从其指定的格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring能够以String类型提供值转换成所有内置类型，比如`int`， `long`，`String`，`boolean`，等等。

在创建容器时，Spring容器会验证每个bean的配置。但是，在实际创建Bean之前，不会设置Bean属性本身。创建容器时，将创建具有单例作用域并设置为预先实例化（默认）的Bean。范围在[Bean范围](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes)中定义。否则，仅在请求时才创建Bean。创建和分配bean的依赖关系及其依赖关系（依此类推）时，创建bean可能会导致创建一个bean图。请注意，这些依赖项之间的分辨率不匹配可能会显示得较晚-即在第一次创建受影响的bean时。

循环依赖

如果主要使用构造函数注入，则可能会创建无法解决的循环依赖方案。

例如：类A通过构造函数注入需要B类的实例，而类B通过构造函数注入需要B类的实例。如果您为将类A和B相互注入而配置了bean，则Spring IoC容器会在运行时检测到此循环引用，并抛出 `BeanCurrentlyInCreationException`。

一种可能的解决方案是编辑某些类的源代码，这些类将由设置者而不是构造函数来配置。或者，避免构造函数注入，而仅使用setter注入。换句话说，尽管不建议这样做，但是您可以使用setter注入配置循环依赖关系。

与典型情况（没有循环依赖关系）不同，Bean A和Bean B之间的循环依赖关系迫使其中一个Bean在完全完全初始化之前被注入另一个Bean（经典的“鸡与蛋”场景）。

通常，您可以信任Spring做正确的事。它在容器加载时检测配置问题，例如对不存在的Bean的引用和循环依赖项。在实际创建Bean时，Spring设置属性并尽可能晚地解决依赖关系。这意味着已经正确加载的Spring容器以后可以在请求对象时生成异常，如果在创建该对象或其依赖项之一时遇到问题-例如，由于缺少或无效，Bean会引发异常属性。某些配置问题的这种潜在的延迟可见性是为什么`ApplicationContext`默认情况下，实现会实例化单例bean。在实际需要这些bean之前，要花一些前期时间和内存来创建它们，您会在创建bean时发现配置问题`ApplicationContext`，而不是稍后发现。您仍然可以覆盖此默认行为，以便单例bean延迟初始化，而不是预先实例化。

如果不存在循环依赖关系，则在将一个或多个协作Bean注入从属Bean时，每个协作Bean都将被完全配置，然后再注入到从属Bean中。这意味着，如果bean A依赖于bean B，则Spring IoC容器会在对bean A调用setter方法之前完全配置beanB。换句话说，bean被实例化（如果不是预先实例化的singleton）。 ），设置其依赖项，并调用相关的生命周期方法（例如已[配置的init方法](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) 或[InitializingBean回调方法](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)）。

##### 依赖注入的例子

以下示例将基于XML的配置元数据用于基于setter的DI。Spring XML配置文件的一小部分指定了一些bean定义，如下所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的`ExampleBean`类：

爪哇

科特林

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的示例中，声明了setter以与XML文件中指定的属性匹配。以下示例使用基于构造函数的DI：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的`ExampleBean`类：

爪哇

科特林

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

Bean定义中指定的构造函数参数用作的构造函数的参数`ExampleBean`。

现在考虑该示例的一个变体，在该变体中，不是使用构造函数，而是告诉Spring调用`static`工厂方法以返回对象的实例：

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的`ExampleBean`类：

爪哇

科特林

```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

`static`工厂方法的参数由``元素提供，就像实际使用构造函数一样。factory方法返回的类的类型不必与包含`static`factory方法的类具有相同的类型（尽管在此示例中是）。实例（非静态）工厂方法可以以本质上相同的方式使用（除了使用`factory-bean`属性代替`class`属性之外），因此在此不讨论这些细节。

#### 1.4.2。依赖性和详细配置

如上[一节所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-collaborators)，您可以将bean属性和构造函数参数定义为对其他托管bean（协作者）的引用或内联定义的值。Spring的基于XML的配置元数据为此目的在其``和``元素中支持子元素类型。

##### 直值（原语，字符串等）

在`value`所述的属性``元素指定属性或构造器参数的人类可读的字符串表示。Spring的 [转换服务](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#core-convert-ConversionService-API)用于将这些值从转换`String`为属性或参数的实际类型。以下示例显示了设置的各种值：

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```

下面的示例使用[p-namespace](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-p-namespace)进行更简洁的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>

</beans>
```

前面的XML更简洁。但是，除非在创建bean定义时使用支持自动属性完成的IDE（例如[IntelliJ IDEA](https://www.jetbrains.com/idea/)或[Spring Tools for Eclipse](https://spring.io/tools)），否则错字是在运行时而不是设计时发现的。强烈建议您使用此类IDE帮助。

您还可以配置`java.util.Properties`实例，如下所示：

```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

Spring容器使用JavaBeans 机制将``元素内的文本转换为 `java.util.Properties`实例`PropertyEditor`。这是一个不错的捷径，并且是Spring团队偏爱使用嵌套``元素而非`value`属性样式的少数几个地方之一。

###### 该`idref`元素

所述`idref`元件是一个简单的防错方法对通过`id`（一个字符串值-而不是参考）在该容器另一个bean的一个``或`` 元件。以下示例显示了如何使用它：

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

前面的bean定义片段（在运行时）与下面的片段完全等效：

```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种形式优于第二种形式，因为使用`idref`标记可以使容器在部署时验证所引用的命名Bean实际上是否存在。在第二个变体中，不对传递给bean `targetName`属性的值进行验证`client`。拼写错误仅在`client`实际实例化bean 时才发现（可能会导致致命的结果）。如果该`client` bean是[原型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes) bean，则可能在部署容器后很长时间才发现此错字和所产生的异常。

|      | 元素 的`local`属性在`idref`4.0 bean XSD中不再受支持，因为它不再提供常规`bean`引用上的值。升级到4.0模式时，将现有`idref local`引用更改`idref bean`为。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

其中一个共同的地方（至少在早期比Spring 2.0版本）``元素带来的值在配置[AOP拦截](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-pfb-1)在 `ProxyFactoryBean`bean定义。``在指定拦截器名称时使用元素可防止您拼写错误的拦截器ID。

##### 对其他Bean的引用（协作者）

所述`ref`元件是内部的最终元件``或`` 定义元素。在这里，您将Bean的指定属性的值设置为对容器管理的另一个Bean（协作者）的引用。引用的bean是要设置其属性的bean的依赖关系，并且在设置属性之前根据需要对其进行初始化。（如果协作者是单例bean，则它可能已经由容器初始化了。）所有引用最终都是对另一个对象的引用。范围和验证取决于是否通过`bean`或`parent`属性指定另一个对象的ID或名称。

通过标记的`bean`属性指定目标bean ``是最通用的形式，并且允许在相同容器或父容器中创建对任何bean的引用，而不管它是否在同一XML文件中。该`bean`属性的值 可以`id`与目标Bean 的属性相同，也可以与目标Bean 的属性之一相同`name`。以下示例显示如何使用`ref`元素：

```xml
<ref bean="someBean"/>
```

通过`parent`属性指定目标Bean 将创建对当前容器的父容器中的Bean的引用。该`parent` 属性的值可以`id`与目标Bean 的属性或目标Bean的`name`属性中的值之一相同。目标Bean必须位于当前容器的父容器中。主要在具有容器层次结构并且要使用与父bean名称相同的代理将现有bean封装在父容器中时，才应使用此bean参考变量。以下清单对显示了如何使用该`parent`属性：

```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

|      | 元素 的`local`属性在`ref`4.0 bean XSD中不再受支持，因为它不再提供常规`bean`引用上的值。升级到4.0模式时，将现有`ref local`引用更改`ref bean`为。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 内豆

甲``内部的元件``或``元件限定内部豆，如下面的示例所示：

```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean定义不需要定义的ID或名称。如果指定，则容器不使用该值作为标识符。容器还会忽略`scope`创建时的标志，因为内部Bean始终是匿名的，并且始终与外部Bean一起创建。不可能独立地访问内部bean或将它们注入到协作bean中而不是封装在bean中。

作为一个特例，可以从自定义范围接收破坏回调，例如，针对单例bean中包含的请求范围内的bean。内部bean实例的创建与其包含的bean绑定在一起，但是销毁回调使它可以参与请求范围的生命周期。这不是常见的情况。内部bean通常只共享其包含bean的作用域。

##### 馆藏

的``，``，``，和``元件设置Java的属性和参数`Collection`类型`List`，`Set`，`Map`，和`Properties`，分别。以下示例显示了如何使用它们：

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

映射键或值的值或设置值也可以是以下任意元素：

```xml
bean | ref | idref | list | set | map | props | value | null
```

###### 集合合并

Spring容器还支持合并集合。应用程序开发人员可以定义父<list/>，<map/>，<set/>或<props/>元素，并有孩子<list/>，<map/>，<set/>或<props/>元素继承和父集合覆盖值。也就是说，子集合的值是合并父集合和子集合的元素的结果，子集合的元素将覆盖父集合中指定的值。

关于合并的本节讨论了父子bean机制。不熟悉父bean和子bean定义的读者可能希望在继续之前阅读 [相关部分](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-child-bean-definitions)。

下面的示例演示了集合合并：

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

注意，在bean定义的`merge=true`属性的``元素 上使用了属性。当bean解析并由容器实例化时，结果实例具有一个集合，该集合包含将子项的集合与父项的集合合并的结果 。以下清单显示了结果：`adminEmails``child``child``adminEmails` `Properties``adminEmails``adminEmails`

```
管理员
=administrator@example.com 
sales=sales@example.com support=support@example.co.uk
```

孩子`Properties`集合的值设置继承父所有属性元素``，和孩子的为值`support`值将覆盖父集合的价值。

这一合并行为同样适用于``，``和`` 集合类型。在``元素的特定情况下，将 维护与`List`集合类型关联的语义（即`ordered`值集合的概念）。父级的值位于所有子级列表的值之前。在的情况下`Map`，`Set`和`Properties`集合类型，没有顺序存在。因此，没有排序的语义在背后的关联的集合类型的效果`Map`，`Set`以及`Properties`实现类型，容器内部使用。

###### 馆藏合并的局限性

您不能合并不同的集合类型（例如`Map`和`List`）。如果尝试这样做，`Exception`则会抛出一个适当的值。该`merge`属性必须在下面的继承的子定义中指定。`merge`在父集合定义上指定属性是多余的，不会导致所需的合并。

###### 强类型集合

随着Java 5中通用类型的引入，您可以使用强类型集合。也就是说，可以声明一个`Collection`类型，使其仅包含（例如）`String`元素。如果使用Spring将强类型依赖注入`Collection`到Bean中，则可以利用Spring的类型转换支持，以便在将强类型`Collection` 实例的元素添加到之前将其转换为适当的类型`Collection`。以下Java类和bean定义显示了如何执行此操作：

爪哇

科特林

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

当准备注入bean 的`accounts`属性时，可以通过反射获得`something`有关强类型的元素类型的泛型信息`Map`。因此，Spring的类型转换基础结构将各种值元素识别为type `Float`，并将字符串值（`9.99, 2.75`和 `3.99`）转换为实际`Float`类型。

##### 空字符串值和空字符串值

Spring将属性等的空参数视为empty `Strings`。以下基于XML的配置元数据片段将`email`属性设置为空 `String`值（“”）。

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

前面的示例等效于以下Java代码：

爪哇

科特林

```java
exampleBean.setEmail("");
```

该``元素处理`null`的值。以下清单显示了一个示例：

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

前面的配置等效于下面的Java代码：

爪哇

科特林

```java
exampleBean.setEmail(null);
```

##### 具有p-命名空间的XML快捷方式

p-namespace允许您使用`bean`元素的属性（而不是嵌套 ``元素）来描述协作bean的属性值，或同时使用这两者。

Spring支持基于XML Schema定义的[具有名称空间的](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-schemas)可扩展配置格式。`beans`本章讨论的配置格式在XML Schema文档中定义。但是，p命名空间未在XSD文件中定义，仅存在于Spring的核心中。

以下示例显示了两个XML代码段（第一个使用标准XML格式，第二个使用p-命名空间），它们可以解析为相同的结果：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```

该示例显示了`email`在bean定义中调用的p-namespace中的属性。这告诉Spring包含一个属性声明。如前所述，p名称空间没有架构定义，因此可以将属性名称设置为属性名称。

下一个示例包括另外两个bean定义，它们都引用了另一个bean：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

此示例不仅包括使用p-namespace的属性值，还使用特殊格式声明属性引用。第一个bean定义用于``创建从bean `john`到bean 的引用 `jane`，而第二个bean定义`p:spouse-ref="jane"`用作属性来执行完全相同的操作。在这种情况下，`spouse`属性名称是，而该`-ref`部分表示这不是一个直接值，而是对另一个bean的引用。

|      | p命名空间不如标准XML格式灵活。例如，用于声明属性引用的格式与以结尾的属性发生冲突`Ref`，而标准XML格式则没有。我们建议您仔细选择方法，并与团队成员进行交流，以避免同时使用这三种方法生成XML文档。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 具有c-namespace的XML快捷方式

与[具有p-namespace](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-p-namespace)的[XML Shortcut](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-p-namespace)相似，在Spring 3.1中引入的c-namespace允许使用内联属性来配置构造函数参数，而不是嵌套`constructor-arg`元素。

以下示例使用`c:`名称空间执行与 [基于构造函数的依赖注入相同的操作](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-constructor-injection)：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

该`c:`命名空间使用相同的约定作为`p:`一个（尾部`-ref`的bean引用），供他们的名字设置构造函数的参数。同样，即使未在XSD模式中定义它（也存在于Spring内核中），也需要在XML文件中声明它。

对于极少数情况下无法使用构造函数自变量名称的情况（通常，如果字节码是在没有调试信息的情况下编译的），可以对参数索引使用后备，如下所示：

```xml
<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```

|      | 由于XML语法的原因，索引表示法要求使用前导`_`，因为XML属性名称不能以数字开头（即使某些IDE允许）。相应的索引符号也可用于``元素，但不常用，因为声明的简单顺序通常就足够了。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

实际上，构造函数解析 [机制](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-ctor-arguments-resolution)在匹配参数方面非常有效，因此，除非您确实需要，否则我们建议在整个配置过程中使用名称表示法。

##### 复合属性名称

设置bean属性时，可以使用复合属性名称或嵌套属性名称，只要路径中除最终属性名称之外的所有组件都没有`null`。考虑以下bean定义：

```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

所述`something`豆具有`fred`属性，该属性具有`bob`属性，其具有`sammy` 特性，并且最终`sammy`属性被设置为值`123`。为了使其工作，bean 的`fred`属性`something`和的`bob`属性`fred`一定不能`null`在bean构建之后。否则，将`NullPointerException`引发a。

#### 1.4.3。使用`depends-on`

如果一个bean是另一个bean的依赖项，则通常意味着将一个bean设置为另一个bean的属性。通常，您可以使用基于XML的配置元数据中的[`` 元素](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-ref-element)来完成此操作。但是，有时bean之间的依赖性不太直接。一个示例是何时需要触发类中的静态初始值设定项，例如用于数据库驱动程序注册。该`depends-on`属性可以在初始化使用此元素的bean之前显式强制初始化一个或多个bean。以下示例使用该`depends-on`属性表示对单个bean的依赖关系：

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表达对多个bean的依赖关系，请提供一个bean名称列表作为该`depends-on`属性的值（逗号，空格和分号是有效的分隔符）：

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

|      | 该`depends-on`属性既可以指定初始化时间依赖性，也可以仅在[单例](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-singleton) bean 的情况下指定相应的销毁时间依赖性。定义`depends-on`与给定bean 的关系的从属bean 首先被销毁，然后再销毁给定bean本身。这样，`depends-on`还可以控制关机顺序。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.4.4。懒初始化豆

默认情况下，`ApplicationContext`实现会在初始化过程中积极创建和配置所有 [单例](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-singleton) bean。通常，这种预初始化是可取的，因为与数小时甚至数天后相比，会立即发现配置或周围环境中的错误。如果不希望这种行为，则可以通过将bean定义标记为延迟初始化来防止单例bean的预实例化。延迟初始化的bean告诉IoC容器在首次请求时（而不是在启动时）创建一个bean实例。

在XML中，此行为由 元素`lazy-init`上的属性控制``，如以下示例所示：

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

当上述配置是通过消耗`ApplicationContext`，所述`lazy`豆是不是提前被实例化时`ApplicationContext`开始，而`not.lazy`豆急切预实例化。

但是，当延迟初始化的bean是未进行延迟初始化的单例bean的依赖项`ApplicationContext`时，由于在启动时必须满足单例的依赖关系，所以Bean在启动时会创建延迟初始化的bean。延迟初始化的bean被注入到其他未延迟初始化的单例bean中。

您还可以通过使用元素`default-lazy-init`上的属性在容器级别上控制延迟初始化， ``以下示例显示：

```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

#### 1.4.5。自动装配协作器

Spring容器可以自动装配协作bean之间的关系。您可以通过检查的内容，让Spring自动为您的bean解决协作者（其他bean）`ApplicationContext`。自动装配具有以下优点：

- 自动装配可以大大减少指定属性或构造函数参数的需要。（[在本章其他地方讨论的](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-child-bean-definitions)其他机制，例如Bean模板 [，](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-child-bean-definitions)在这方面也很有价值。）
- 随着对象的发展，自动装配可以更新配置。例如，如果您需要向类中添加依赖项，则无需修改配置即可自动满足该依赖项。因此，自动装配在开发过程中特别有用，而不必担心在代码库变得更稳定时切换到显式接线的选择。

使用基于XML的配置元数据时（请参阅[Dependency Injection](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-collaborators)），您可以使用元素的`autowire`属性为 bean定义指定自动装配模式``。自动装配功能具有四种模式。您可以为每个bean指定自动装配，因此可以选择要自动装配的装配。下表描述了四种自动装配模式：

| 模式          | 说明                                                         |
| :------------ | :----------------------------------------------------------- |
| `no`          | （默认）无自动装配。Bean引用必须由`ref`元素定义。对于较大的部署，建议不要更改默认设置，因为明确指定协作者可以提供更好的控制和清晰度。在某种程度上，它记录了系统的结构。 |
| `byName`      | 按属性名称自动布线。Spring寻找与需要自动装配的属性同名的bean。例如，如果一个bean定义被设置为按名称自动装配并且包含一个`master`属性（即它具有一个 `setMaster(..)`方法），那么Spring将查找一个名为的bean定义，`master`并使用它来设置该属性。 |
| `byType`      | 如果容器中恰好存在一个属性类型的bean，则使该属性自动连接。如果存在多个错误，则会引发致命异常，这表明您可能无法`byType`对该bean 使用自动装配。如果没有匹配的bean，则什么都不会发生（未设置该属性）。 |
| `constructor` | 类似于`byType`但适用于构造函数参数。如果容器中不存在构造函数参数类型的一个bean，则将引发致命错误。 |

使用`byType`或`constructor`自动装配模式，您可以连接阵列和键入的集合。在这种情况下，将提供容器中与期望类型匹配的所有自动装配候选，以满足相关性。`Map`如果期望的密钥类型为，则可以自动连接强类型实例`String`。自动装配`Map` 实例的值包括与期望类型匹配的所有bean实例，并且 `Map`实例的键包含相应的bean名称。

##### 自动接线的局限性和缺点

当在项目中一致使用自动装配时，自动装配效果最佳。如果通常不使用自动装配，那么使用开发人员仅连接一个或两个bean定义可能会使开发人员感到困惑。

考虑自动装配的局限性和缺点：

- 显式依赖项`property`和`constructor-arg`设置始终会覆盖自动装配。您无法自动装配简单的属性，例如基元 `Strings`，和`Classes`（以及此类简单属性的数组）。此限制是设计使然。
- 自动装配不如显式接线精确。尽管如前所述，Spring还是谨慎地避免在可能产生意外结果的模棱两可的情况下进行猜测。Spring管理的对象之间的关系不再明确记录。
- 接线信息可能不适用于可能从Spring容器生成文档的工具。
- 容器中的多个bean定义可能与要自动装配的setter方法或构造函数参数指定的类型匹配。对于数组，集合或 `Map`实例，这不一定是问题。但是，对于期望单个值的依赖项，不会任意解决此歧义。如果没有唯一的bean定义可用，则引发异常。

在后一种情况下，您有几种选择：

- 放弃自动布线，转而使用明确的布线。
- 如[下一节](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-autowire-candidate)所述，通过将其`autowire-candidate`属性设置为来避免自动装配bean定义。`false`
- 通过将单个bean定义`primary`的``元素属性设置为，将其指定为主要候选对象 `true`。
- 如[基于注释的容器配置中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-annotation-config)所述，[通过基于](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-annotation-config)注释的配置实现更细粒度的控件。

##### 从自动装配中排除Bean

在每个bean的基础上，您可以从自动装配中排除一个bean。使用Spring的XML格式，将元素的`autowire-candidate`属性设置``为`false`。容器使特定的bean定义对自动装配基础结构不可用（包括注释样式配置，例如[`@Autowired`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-autowired-annotation)）。

|      | 该`autowire-candidate`属性旨在仅影响基于类型的自动装配。它不会影响按名称显示的显式引用，即使未将指定的Bean标记为自动装配候选，该名称也可得到解析。结果，如果名称匹配，按名称自动装配仍然会注入一个bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以基于与Bean名称的模式匹配来限制自动装配候选。顶级``元素在其`default-autowire-candidates`属性内接受一个或多个模式 。例如，要将自动装配候选状态限制为名称以结尾的任何bean `Repository`，请提供值`*Repository`。要提供多种模式，请在以逗号分隔的列表中定义它们。`true`或`false`Bean定义`autowire-candidate`属性的显式值 始终优先。对于此类bean，模式匹配规则不适用。

这些技术对于您不希望通过自动装配将其注入到其他bean中的bean非常有用。这并不意味着排除的bean本身不能使用自动装配进行配置。相反，bean本身不是自动装配其他bean的候选对象。

#### 1.4.6。方法注入

在大多数应用场景中，容器中的大多数bean是 [singletons](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-singleton)。当单例Bean需要与另一个单例Bean协作或非单例Bean需要与另一个非单例Bean协作时，通常可以通过将一个Bean定义为另一个Bean的属性来处理依赖性。当bean的生命周期不同时会出现问题。假设单例bean A可能需要使用非单例（原型）bean B，也许是在A的每个方法调用上使用。容器仅创建一次单例bean A，因此只有一次机会来设置属性。每次需要一个容器时，容器都无法为bean A提供一个新的bean B实例。

一个解决方案是放弃某些控制反转。您可以通过实现接口，并通过每次[容器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-client) A都需要[容器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-client) B [的](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-client)[调用来](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-client)请求（通常是新的）bean B实例，来[使bean A知道容器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-aware)。以下示例显示了此方法：`ApplicationContextAware`[`getBean("B")`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-client)

爪哇

科特林

```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

前面的内容是不理想的，因为业务代码知道并耦合到Spring框架。方法注入是Spring IoC容器的一项高级功能，使您可以干净地处理此用例。

您可以在[此博客条目中](https://spring.io/blog/2004/08/06/method-injection/)了解有关方法注入动机的更多信息 。

##### 查找方法注入

查找方法注入是容器覆盖容器管理的Bean上的方法并返回容器中另一个命名Bean的查找结果的能力。查找通常涉及原型bean，如上[一节中所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-method-injection)。Spring框架通过使用从CGLIB库生成字节码来动态生成覆盖该方法的子类来实现此方法注入。

|      | 为了使此动态子类起作用，Spring Bean容器子类的类也不能为`final`，要覆盖的方法也不能为`final`。对具有`abstract`方法的类进行单元测试需要您自己对该类进行子类化，并提供该`abstract`方法的存根实现。组件扫描也需要具体的方法，这需要具体的类。另一个关键限制是，查找方法不适用于工厂方法，尤其不适`@Bean`用于配置类中的方法，因为在这种情况下，容器不负责创建实例，因此无法在其上创建运行时生成的子类。苍蝇。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

对于`CommandManager`前面的代码片段中的类，Spring容器动态地覆盖该`createCommand()` 方法的实现。该`CommandManager`班没有任何Spring的依赖，因为返工例所示：

爪哇

科特林

```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

在包含要注入的方法的客户端类（`CommandManager`在本例中为）中，要注入的方法需要以下形式的签名：

```xml
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法为`abstract`，则动态生成的子类将实现该方法。否则，动态生成的子类将覆盖原始类中定义的具体方法。考虑以下示例：

```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

只要需要新的bean 实例，被标识为的bean 就会`commandManager`调用其自己的`createCommand()`方法`myCommand`。`myCommand`如果确实需要，您必须小心地将bean 部署为原型。如果是[单例](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-singleton)，`myCommand` 则每次都返回Bean 的相同实例。

另外，在基于注释的组件模型中，您可以通过`@Lookup`注释声明一个查找方法，如以下示例所示：

爪哇

科特林

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

或者，更习惯地说，您可以依赖于针对查找方法的声明返回类型解析的目标bean：

爪哇

科特林

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```

请注意，通常应使用具体的存根实现声明此类带注释的查找方法，以使它们与Spring的组件扫描规则（默认情况下抽象类会被忽略）兼容。此限制不适用于显式注册或显式导入的Bean类。

|      | 访问范围不同的目标bean的另一种方法是`ObjectFactory`/ `Provider`注入点。请参阅将[范围Bean作为依赖项](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-other-injection)。您可能还会发现`ServiceLocatorFactoryBean`（在 `org.springframework.beans.factory.config`包装中）有用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 任意方法替换

与查找方法注入相比，方法注入的一种不太有用的形式是能够用另一种方法实现替换托管bean中的任意方法。您可以放心地跳过本节的其余部分，直到您真正需要此功能为止。

借助基于XML的配置元数据，您可以使用`replaced-method`元素将现有方法实现替换为已部署bean的另一个实现。考虑以下类，该类具有一个`computeValue`我们要重写的方法：

爪哇

科特林

```java
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```

实现该`org.springframework.beans.factory.support.MethodReplacer` 接口的类提供了新的方法定义，如以下示例所示：

爪哇

科特林

```java
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```

用于部署原始类并指定方法重写的Bean定义类似于以下示例：

```xml
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

您可以``在元素内使用一个或多个元素`` 来指示要覆盖的方法的方法签名。仅当方法重载且类中存在多个变体时，才需要对参数签名。为了方便起见，参数的类型字符串可以是完全限定类型名称的子字符串。例如，以下所有匹配项 `java.lang.String`：

```java
java.lang.String
String
Str
```

因为参数的数量通常足以区分每个可能的选择，所以通过让您仅键入与参数类型匹配的最短字符串，此快捷方式可以节省很多输入。

### 1.5。豆范围

创建bean定义时，将创建一个配方来创建该bean定义所定义的类的实际实例。bean定义是配方的想法很重要，因为它意味着与类一样，您可以从一个配方中创建许多对象实例。

您不仅可以控制要插入到从特定bean定义创建的对象中的各种依赖项和配置值，还可以控制从特定bean定义创建的对象的范围。这种方法功能强大且灵活，因为您可以选择通过配置创建的对象的范围，而不必在Java类级别上烘烤对象的范围。可以将Bean定义为部署在多个范围之一中。Spring框架支持六个范围，其中只有在使用web感知时才可用`ApplicationContext`。您还可以创建 [自定义范围。](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-custom)

下表描述了受支持的范围：

| 范围                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [单身人士](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-singleton) | （默认值）将每个Spring IoC容器的单个bean定义范围限定为单个对象实例。 |
| [原型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-prototype) | 将单个bean定义的作用域限定为任意数量的对象实例。             |
| [请求](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-request) | 将单个bean定义的范围限定为单个HTTP请求的生命周期。也就是说，每个HTTP请求都有一个在单个bean定义后面创建的bean实例。仅在可感知网络的Spring上下文中有效`ApplicationContext`。 |
| [会议](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-session) | 将单个bean定义的范围限定为HTTP的生命周期`Session`。仅在可感知网络的Spring上下文中有效`ApplicationContext`。 |
| [应用](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-application) | 将单个bean定义的作用域限定为的生命周期`ServletContext`。仅在可感知网络的Spring上下文中有效`ApplicationContext`。 |
| [网络套接字](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-websocket-scope) | 将单个bean定义的作用域限定为的生命周期`WebSocket`。仅在可感知网络的Spring上下文中有效`ApplicationContext`。 |

|      | 从Spring 3.0开始，线程作用域可用，但默认情况下未注册。有关更多信息，请参见的文档 [`SimpleThreadScope`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/support/SimpleThreadScope.html)。有关如何注册此自定义范围或任何其他自定义范围的说明，请参阅 [使用自定义范围](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-custom-using)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.5.1。单例范围

仅管理一个singleton bean的一个共享实例，并且所有对具有ID或与该bean定义相匹配的ID的bean的请求都会导致该特定的bean实例由Spring容器返回。

换句话说，当您定义一个bean定义并且其作用域为单例时，Spring IoC容器将为该bean定义所定义的对象创建一个实例。该单个实例存储在此类单例bean的高速缓存中，并且对该命名bean的所有后续请求和引用都返回该高速缓存的对象。下图显示了单例作用域的工作方式：

![单身人士](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/images/singleton.png)

Spring的singleton bean的概念不同于“四人帮（Gang of Four，GoF）模式”一书中定义的singleton模式。GoF单例对对象的范围进行硬编码，以使每个ClassLoader只能创建一个特定类的一个实例。最好将Spring单例的范围描述为每个容器和每个bean。这意味着，如果您在单个Spring容器中为特定类定义一个bean，则Spring容器将创建该bean定义所定义的类的一个且只有一个实例。单例作用域是Spring中的默认作用域。要将bean定义为XML中的单例，可以定义bean，如以下示例所示：

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

#### 1.5.2。原型范围

每次对特定bean提出请求时，bean部署的非单一原型范围都会导致创建一个新bean实例。也就是说，将Bean注入到另一个Bean中，或者您可以通过`getBean()`容器上的方法调用来请求它。通常，应将原型作用域用于所有有状态Bean，将单例作用域用于无状态Bean。

下图说明了Spring原型范围：

![原型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/images/prototype.png)

（数据访问对象（DAO）通常不配置为原型，因为典型的DAO不拥有任何对话状态。对于我们而言，重用单例图的核心更为容易。）

以下示例将bean定义为XML原型：

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

与其他作用域相反，Spring不管理原型Bean的完整生命周期。容器实例化，配置或组装原型对象，然后将其交给客户端，而没有该原型实例的进一步记录。因此，尽管在不考虑范围的情况下在所有对象上都调用了初始化生命周期回调方法，但对于原型而言，不会调用已配置的销毁生命周期回调。客户端代码必须清除原型作用域内的对象并释放原型Bean拥有的昂贵资源。要使Spring容器释放由原型作用域的bean 占用的资源，请尝试使用自定义[bean后处理器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-extension-bpp)，其中包含对需要清理的bean的引用。

在某些方面，Spring容器在原型作用域bean方面的角色是Java `new`运算符的替代。超过该时间点的所有生命周期管理必须由客户端处理。（有关Spring容器中bean的生命周期的详细信息，请参阅[Lifecycle Callbacks](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle)。）

#### 1.5.3。具有原型Bean依赖关系的Singleton Bean

当您使用对原型bean有依赖性的单例作用域Bean时，请注意，依赖关系在实例化时已解决。因此，如果将依赖项原型的bean依赖项注入到单例范围的bean中，则将实例化新的原型bean，然后将依赖项注入到单例bean中。原型实例是曾经提供给单例范围的bean的唯一实例。

但是，假设您希望单例作用域的bean在运行时重复获取原型作用域的bean的新实例。您不能将原型作用域的bean依赖项注入到您的单例bean中，因为当Spring容器实例化单例bean并解析并注入其依赖项时，该注入仅发生一次。如果在运行时不止一次需要原型bean的新实例，请参见[方法注入](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-method-injection)

#### 1.5.4。请求，会话，应用程序和WebSocket范围

在`request`，`session`，`application`，和`websocket`范围只有当你使用一个基于web的Spring可`ApplicationContext`实现（例如 `XmlWebApplicationContext`）。如果您使用这些范围与普通的Spring IoC容器，如`ClassPathXmlApplicationContext`，一个`IllegalStateException`是抱怨一个未知的bean作用域被抛出。

##### 初始Web配置

为了支持豆的范围界定在`request`，`session`，`application`，和 `websocket`（即具有web作用域bean），需要做少量的初始配置定义你的豆之前。（对于标准示波器，不需要此初始设置：`singleton`和`prototype`。）

如何完成此初始设置取决于您的特定Servlet环境。

如果您在Spring Web MVC中访问作用域化的bean，实际上是在Spring处理的请求中，则`DispatcherServlet`不需要特殊的设置。 `DispatcherServlet`已经公开了所有相关状态。

如果您使用Servlet 2.5 Web容器，并且在Spring之外处理请求 `DispatcherServlet`（例如，使用JSF或Struts时），则需要注册 `org.springframework.web.context.request.RequestContextListener` `ServletRequestListener`。对于Servlet 3.0+，可以使用该`WebApplicationInitializer` 接口以编程方式完成此操作。或者，或者对于较旧的容器，将以下声明添加到Web应用程序的`web.xml`文件中：

```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

另外，如果您的监听器设置存在问题，请考虑使用Spring的 `RequestContextFilter`。过滤器映射取决于周围的Web应用程序配置，因此您必须适当地对其进行更改。以下清单显示了Web应用程序的过滤器部分：

```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

`DispatcherServlet`，`RequestContextListener`和`RequestContextFilter`都做完全相同的事情，即将HTTP请求对象绑定到`Thread`为该请求提供服务的对象。这使得在请求链和会话范围内的bean可以在调用链的更下游使用。

##### 要求范围

考虑将以下XML配置用于bean定义：

```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

Spring容器`LoginAction`通过`loginAction`为每个HTTP请求使用bean定义来创建bean 的新实例。也就是说， `loginAction`bean的作用域位于HTTP请求级别。您可以根据需要更改创建实例的内部状态，因为从同一`loginAction`bean定义创建的其他实例看不到这些状态变化。它们特定于单个请求。当请求完成处理时，将丢弃作用于该请求的Bean。

使用注释驱动的组件或Java配置时，`@RequestScope`可以使用注释将组件分配给`request`作用域。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

##### 会议范围

考虑将以下XML配置用于bean定义：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

Spring容器`UserPreferences`通过在`userPreferences`单个HTTP的生存期内使用bean定义来创建bean 的新实例`Session`。换句话说，`userPreferences`bean有效地在HTTP `Session`级别范围内。与请求范围的Bean一样，您可以根据需要任意更改所创建实例的内部状态，因为知道其他`Session`也在使用从相同`userPreferences`Bean定义创建的实例的HTTP 实例也看不到这些状态变化，因为它们特定于单个HTTP `Session`。当`Session`最终丢弃HTTP时，作用于该特定HTTP的bean `Session`也将被丢弃。

使用注释驱动的组件或Java配置时，可以使用 `@SessionScope`注释将组件分配给`session`作用域。

爪哇

科特林

```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

##### 适用范围

考虑将以下XML配置用于bean定义：

```xml
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

Spring容器`AppPreferences`通过`appPreferences`对整个Web应用程序使用Bean定义一次来创建Bean 的新实例。也就是说， `appPreferences`bean的作用域在该`ServletContext`级别上，并存储为常规 `ServletContext`属性。这有点类似于Spring单例bean，但是有两个重要的区别：它是每个`ServletContext`而不是每个Spring'ApplicationContext' 的单例（在任何给定的Web应用程序中可能都有多个），并且实际上是公开的，因此可见作为`ServletContext`属性。

使用注释驱动的组件或Java配置时，可以使用 `@ApplicationScope`注释将组件分配给`application`作用域。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

##### 范围豆作为依赖项

Spring IoC容器不仅管理对象（bean）的实例化，而且还管理协作者（或依赖项）的连接。如果要将（例如）HTTP请求范围的Bean注入（例如）另一个作用域更长的Bean，则可以选择注入AOP代理来代替已定义范围的Bean。也就是说，您需要注入一个代理对象，该对象公开与范围对象相同的公共接口，但也可以从相关范围（例如HTTP请求）中检索实际目标对象，并将方法调用委托给实际对象。

|      | 您也可以``在范围为的Bean之间`singleton`使用，引用然后经过可序列化的中间代理，因此可以在反序列化时重新获得目标单例Bean。在声明``范围bean时`prototype`，共享代理上的每个方法调用都会导致创建新的目标实例，然后将该调用转发到该目标实例。同样，作用域代理不是以生命周期安全的方式从较短的作用域访问bean的唯一方法。您还可以将注入点（即，构造函数或setter参数或自动连接的字段）声明为`ObjectFactory`，从而允许`getObject()`每次需要时调用按需检索当前实例，而无需保留该实例或将其单独存储。作为扩展变体，您可以声明`ObjectProvider`，它提供了几个附加的访问变体，包括`getIfAvailable`和`getIfUnique`。对此的JSR-330变体将被调用`Provider`，`Provider` 并在`get()`每次检索尝试中与声明和相应的调用一起使用。有关JSR-330总体的更多详细信息，请参见[此处](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-standard-annotations)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下示例中的配置仅一行，但是了解其背后的“原因”和“方式”很重要：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> 
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

|      | 定义代理的行。 |
| ---- | -------------- |
|      |                |

要创建这样的代理，请将子``元素插入到有作用域的bean定义中（请参阅[选择要创建的代理类型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-other-injection-proxies)和 [基于XML Schema的配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-schemas)）。为什么Bean的定义范围为`request`，`session`并且自定义范围级别需要``元素？考虑以下单例bean定义，并将其与您需要为上述范围定义的内容进行对比（请注意，以下 `userPreferences`bean定义不完整）：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

在前面的示例中，单例bean（`userManager`）注入了对HTTP `Session`范围的bean（`userPreferences`）的引用。这里的要点是， `userManager`bean是单例的：每个容器仅实例化一次，并且它的依赖项（在这种情况下，仅一个，即`userPreferences`bean）也仅注入一次。这意味着`userManager`Bean仅在完全相同的`userPreferences`对象（即最初注入该对象的对象）上操作。

将寿命较短的作用域bean注入寿命较长的作用域bean时，这不是您想要的行为（例如，将HTTP `Session`范围的协作bean作为依赖项注入到singleton bean中）。相反，您只需要一个`userManager` 对象，并且在HTTP的生存期内`Session`，您需要一个`userPreferences`特定于HTTP 的对象`Session`。因此，容器创建了一个对象，该对象公开了与`UserPreferences`类完全相同的公共接口（理想情况下是作为`UserPreferences`实例的对象），该`UserPreferences`对象可以从范围确定机制（HTTP请求`Session`等）中获取实际 对象。容器将此代理对象注入到`userManager`Bean中，而后者并不知道此`UserPreferences`引用是代理。在此示例中，当 `UserManager`实例在依赖项注入`UserPreferences` 对象上调用方法，实际上是在代理上调用方法。然后，代理 `UserPreferences`从HTTP（在这种情况下）`Session`获取真实`UserPreferences`对象，并将方法调用委托给检索到的真实对象。

因此，在将Bean `request-`和`session-scoped`Bean注入到协作对象中时，需要以下配置（正确和完整） ，如以下示例所示：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

###### 选择要创建的代理类型

默认情况下，当Spring容器为使用``元素标记的bean创建代理时，将创建基于CGLIB的类代理。

|      | CGLIB代理仅拦截公共方法调用！不要在此类代理上调用非公共方法。它们没有被委派给实际的作用域目标对象。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

另外，您可以通过指定元素`false`的`proxy-target-class`属性值，将Spring容器配置为为此类作用域的Bean创建基于标准JDK接口的代理``。使用基于JDK接口的代理意味着您不需要应用程序类路径中的其他库即可影响此类代理。但是，这也意味着作用域Bean的类必须实现至少一个接口，并且作用域Bean注入到其中的所有协作者必须通过其接口之一引用该Bean。以下示例显示基于接口的代理：

```xml
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.stuff.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

有关选择基于类或基于接口的代理的更多详细信息，请参阅[代理机制](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-proxying)。

#### 1.5.5。自定义范围

Bean作用域机制是可扩展的。您可以定义自己的作用域，甚至重新定义现有作用域，尽管后者被认为是不好的做法，并且您不能覆盖内置作用域`singleton`和`prototype`作用域。

##### 创建自定义范围

要将自定义范围集成到Spring容器中，您需要实现`org.springframework.beans.factory.config.Scope`本节中描述的 接口。有关如何实现自己的范围的想法，请参阅`Scope` Spring Framework本身和[`Scope`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/Scope.html)javadoc 附带的实现 ，其中详细说明了需要实现的方法。

该`Scope`接口有四种方法可以从作用域中获取对象，将其从作用域中删除，然后将其销毁。

例如，会话范围实现返回会话范围的Bean（如果不存在，则该方法将其绑定到会话上以供将来参考之后，将返回该Bean的新实例）。以下方法从基础范围返回对象：

爪哇

科特林

```java
Object get(String name, ObjectFactory<?> objectFactory)
```

会话范围的实现，例如，从基础会话中删除了会话范围的bean。应该返回该对象，但是如果找不到具有指定名称的对象，则可以返回null。以下方法将对象从基础范围中删除：

爪哇

科特林

```java
Object remove(String name)
```

以下方法注册在销毁作用域或销毁作用域中的指定对象时作用域应执行的回调：

爪哇

科特林

```java
void registerDestructionCallback(String name, Runnable destructionCallback)
```

有关 销毁回调的更多信息，请参见[javadoc](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/Scope.html#registerDestructionCallback)或Spring范围实现。

以下方法获取基础范围的会话标识符：

爪哇

科特林

```java
String getConversationId()
```

每个范围的标识符都不相同。对于会话范围的实现，此标识符可以是会话标识符。

##### 使用自定义范围

在编写和测试一个或多个自定义`Scope`实现之后，您需要使Spring容器意识到您的新作用域。以下方法是`Scope`在Spring容器中注册新方法的主要方法：

爪哇

科特林

```java
void registerScope(String scopeName, Scope scope);
```

此方法在`ConfigurableBeanFactory`接口上声明，该接口可通过 Spring附带的`BeanFactory`大多数具体`ApplicationContext`实现上的属性获得。

该`registerScope(..)`方法的第一个参数是与范围关联的唯一名称。Spring容器本身中的此类名称示例为`singleton`和 `prototype`。该`registerScope(..)`方法的第二个参数是`Scope`您希望注册和使用的自定义实现的实际实例。

假设您编写了自定义`Scope`实现，然后注册它，如下面的示例所示。

|      | 下一个示例使用`SimpleThreadScope`Spring附带的，但默认情况下未注册。对于您自己的自定义`Scope` 实现，说明将是相同的。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

爪哇

科特林

```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

然后，您可以按照您的custom的作用域规则创建bean定义， `Scope`如下所示：

```xml
<bean id="..." class="..." scope="thread">
```

使用自定义`Scope`实现，您不仅限于范围的程序注册。您还可以`Scope`使用`CustomScopeConfigurer`该类以声明方式进行注册 ，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="thing2" class="x.y.Thing2" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="thing1" class="x.y.Thing1">
        <property name="thing2" ref="thing2"/>
    </bean>

</beans>
```

|      | 当您放置``在`FactoryBean`实现中时，作用域是工厂Bean本身，而不是从中返回的对象`getObject()`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.6。自定义豆的性质

Spring框架提供了许多接口，可用于自定义Bean的性质。本节将它们分组如下：

- [生命周期回调](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle)
- [`ApplicationContextAware` 和 `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-aware)
- [其他`Aware`介面](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aware-list)

#### 1.6.1。生命周期回调

为了与容器对bean生命周期的管理进行交互，可以实现Spring `InitializingBean`和`DisposableBean`接口。容器要求 `afterPropertiesSet()`前者和`destroy()`后者使bean在初始化和销毁bean时执行某些操作。

|      | 通常，JSR-250 `@PostConstruct`和`@PreDestroy`注释被认为是在现代Spring应用程序中接收生命周期回调的最佳实践。使用这些注释意味着您的bean没有耦合到特定于Spring的接口。有关详细信息，请参见[使用`@PostConstruct`和`@PreDestroy`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations)。如果你不希望使用JSR-250注解，但你仍然要删除的耦合，考虑`init-method`和`destroy-method`bean定义元数据。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

在内部，Spring Framework使用`BeanPostProcessor`实现来处理它可以找到的任何回调接口并调用适当的方法。如果您需要自定义功能或其他生命周期行为，Spring默认不提供，则您可以`BeanPostProcessor`自己实现。有关更多信息，请参见 [容器扩展点](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-extension)。

除了初始化和销毁回调，Spring管理的对象还可以实现`Lifecycle`接口，以便这些对象可以在容器自身生命周期的驱动下参与启动和关闭过程。

本节介绍了生命周期回调接口。

##### 初始化回调

`org.springframework.beans.factory.InitializingBean`容器在bean上设置了所有必需的属性后，该接口使bean可以执行初始化工作。该`InitializingBean`接口指定一个方法：

爪哇

科特林

```java
void afterPropertiesSet() throws Exception;
```

我们建议您不要使用该`InitializingBean`接口，因为它不必要地将代码耦合到Spring。另外，我们建议使用[`@PostConstruct`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations)注释或指定POJO初始化方法。对于基于XML的配置元数据，可以使用`init-method`属性指定具有无效无参数签名的方法的名称。通过Java配置，您可以使用的`initMethod`属性 `@Bean`。请参阅[接收生命周期回调](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-lifecycle-callbacks)。考虑以下示例：

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

爪哇

科特林

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

前面的示例与下面的示例（由两个清单组成）几乎具有完全相同的效果：

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

爪哇

科特林

```java
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

但是，前面两个示例中的第一个示例并未将代码耦合到Spring。

##### 销毁回调

`org.springframework.beans.factory.DisposableBean`当包含该接口的容器被销毁时，实现该接口可使Bean获得回调。该 `DisposableBean`接口指定一个方法：

爪哇

科特林

```java
void destroy() throws Exception;
```

我们建议您不要使用`DisposableBean`回调接口，因为它不必要地将代码耦合到Spring。另外，我们建议使用[`@PreDestroy`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations)注释或指定bean定义支持的通用方法。对于基于XML的配置元数据，您可以在`destroy-method`上使用属性``。通过Java配置，您可以使用的`destroyMethod`属性`@Bean`。请参阅 [接收生命周期回调](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-lifecycle-callbacks)。考虑以下定义：

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

爪哇

科特林

```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

前面的定义与下面的定义几乎具有完全相同的效果：

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

爪哇

科特林

```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

但是，前面两个定义中的第一个没有将代码耦合到Spring。

|      | 您可以`destroy-method`为``元素的属性分配一个特殊 `(inferred)`值，该值指示Spring自动检测特定bean类上的公共`close`或 `shutdown`方法。（实现`java.lang.AutoCloseable`或`java.io.Closeable`匹配的任何类 。）您还可以`(inferred)`在元素的`default-destroy-method`属性 上设置此特殊值，``以将该行为应用于整个bean集（请参见 [Default Initialization and Destroy Methods](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-default-init-destroy-methods)）。请注意，这是Java配置的默认行为。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 默认初始化和销毁方法

当你写的初始化和销毁不使用Spring的具体方法回调`InitializingBean`和`DisposableBean`回调接口，你的名字，如通常的写入方法`init()`，`initialize()`，`dispose()`，等等。理想情况下，此类生命周期回调方法的名称应在整个项目中标准化，以便所有开发人员都使用相同的方法名称并确保一致性。

您可以将Spring容器配置为“查找”命名的初始化，并销毁每个bean上的回调方法名称。这意味着作为应用程序开发人员，您可以编写应用程序类并使用称为的初始化回调 `init()`，而不必为`init-method="init"`每个bean定义配置属性。Spring IoC容器在创建bean时（并按照[前面描述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle)的标准生命周期回调协定）调用该方法。此功能还对初始化和销毁方法回调强制执行一致的命名约定。

假设您的初始化回调方法已命名，`init()`而destroy回调方法已命名`destroy()`。然后，您的课程类似于以下示例中的课程：

爪哇

科特林

```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

然后，您可以在类似于以下内容的Bean中使用该类：

```xml
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

`default-init-method`顶级``元素属性上存在该属性会导致Spring IoC容器将`init`在bean类上调用的方法识别为初始化方法回调。创建和组装bean时，如果bean类具有这种方法，则会在适当的时间调用它。

您可以使用`default-destroy-method`顶级``元素上的属性，以类似方式（在XML中）配置destroy方法回调 。

如果现有的Bean类已经具有按惯例命名的回调方法，则可以通过使用 自身的`init-method`and `destroy-method`属性指定（在XML中）方法名称来覆盖默认值``。

Spring容器保证在为bean提供所有依赖项后立即调用已配置的初始化回调。因此，在原始bean引用上调用了初始化回调，这意味着AOP拦截器等尚未应用于bean。首先完全创建目标bean，然后应用带有其拦截器链的AOP代理（例如）。如果目标Bean和代理分别定义，则您的代码甚至可以绕过代理与原始目标Bean进行交互。因此，将拦截器应用于该`init`方法将是不一致的，因为这样做会将目标Bean的生命周期耦合到其代理或拦截器，并且当您的代码直接与原始目标Bean进行交互时会留下奇怪的语义。

##### 组合生命周期机制

从Spring 2.5开始，您可以使用三个选项来控制Bean生命周期行为：

- 在[`InitializingBean`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)和 [`DisposableBean`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean)回调接口
- 习惯`init()`和`destroy()`方法
- 在[`@PostConstruct`和`@PreDestroy` 注释](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations)。您可以结合使用这些机制来控制给定的bean。

|      | 如果为一个bean配置了多个生命周期机制，并且为每个机制配置了不同的方法名称，则将按照此注释后列出的顺序执行每个已配置的方法。但是，如果`init()`为多个生命周期机制中的多个生命周期机制（例如，对于初始化方法）配置了相同的方法名 ，则该方法将执行一次，如上 [一节所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-default-init-destroy-methods)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

为同一个bean配置的具有不同初始化方法的多种生命周期机制称为：

1. 用注释的方法 `@PostConstruct`
2. `afterPropertiesSet()`由`InitializingBean`回调接口定义
3. 定制配置的`init()`方法

销毁方法的调用顺序相同：

1. 用注释的方法 `@PreDestroy`
2. `destroy()`由`DisposableBean`回调接口定义
3. 定制配置的`destroy()`方法

##### 启动和关机回调

该`Lifecycle`接口为具有自己生命周期要求（例如启动和停止某些后台进程）的任何对象定义了基本方法：

爪哇

科特林

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

任何Spring管理的对象都可以实现该`Lifecycle`接口。然后，当 `ApplicationContext`自身接收到启动和停止信号时（例如，对于运行时的停止/重新启动场景），它将这些调用级联到`Lifecycle`在该上下文中定义的所有实现。它通过委派给来完成此操作`LifecycleProcessor`，如以下清单所示：

爪哇

科特林

```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

请注意，`LifecycleProcessor`本身是`Lifecycle` 接口的扩展。它还添加了两种其他方法来对刷新和关闭的上下文做出反应。

|      | 请注意，常规`org.springframework.context.Lifecycle`接口是用于显式启动和停止通知的普通协议，并不意味着在上下文刷新时自动启动。为了对特定bean的自动启动进行精细控制（包括启动阶段），请考虑实施`org.springframework.context.SmartLifecycle`。另外，请注意，不能保证会在销毁之前发出停止通知。在常规关闭时，`Lifecycle`在传播常规销毁回调之前，所有Bean首先都会收到停止通知。但是，在上下文生存期内的热刷新或中止的刷新尝试中，仅调用destroy方法。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

启动和关闭调用的顺序可能很重要。如果任何两个对象之间存在“依赖”关系，则依赖方在其依赖之后开始，而在依赖之前停止。但是，有时直接依赖项是未知的。您可能只知道某种类型的对象应该先于另一种类型的对象开始。在这些情况下，`SmartLifecycle`接口定义另一个选项，即`getPhase()`在其超级接口上定义的方法 `Phased`。以下清单显示了`Phased`接口的定义：

爪哇

科特林

```java
public interface Phased {

    int getPhase();
}
```

以下清单显示了`SmartLifecycle`接口的定义：

爪哇

科特林

```java
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

启动时，相位最低的对象首先启动。停止时，遵循相反的顺序。因此，实现`SmartLifecycle`并`getPhase()`返回其方法的对象`Integer.MIN_VALUE`将是第一个启动且最后一个停止的对象。在频谱的另一端，相位值 `Integer.MAX_VALUE`表示应该最后启动对象，然后首先停止对象（可能是因为它取决于正在运行的其他进程）。当考虑相位值，同样重要的是要知道，对于任何“正常”的默认阶段 `Lifecycle`目标没有实现`SmartLifecycle`的`0`。因此，任何负相位值都表明对象应在这些标准组件之前开始（并在它们之后停止）。对于任何正相位值，反之亦然。

定义的stop方法`SmartLifecycle`接受回调。`run()`在该实现的关闭过程完成之后，任何实现都必须调用该回调的方法。由于`LifecycleProcessor`接口 的默认实现`DefaultLifecycleProcessor`会在每个阶段内的对象组等待其超时值，以调用该回调，因此可以在必要时启用异步关闭。默认的每阶段超时是30秒。您可以通过定义`lifecycleProcessor`上下文中命名的bean来覆盖默认的生命周期处理器实例 。如果只想修改超时，则定义以下内容即可：

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

如前所述，该`LifecycleProcessor`接口还定义了用于刷新和关闭上下文的回调方法。后者驱动关闭过程，就好像`stop()`已显式调用了它一样，但是它在上下文关闭时发生。另一方面，“刷新”回调启用了`SmartLifecycle`bean的另一个功能 。刷新上下文后（在所有对象都实例化和初始化之后），该回调将被调用。此时，默认的生命周期处理器将检查每个`SmartLifecycle`对象的`isAutoStartup()`方法返回的布尔值 。如果为`true`，则在该点启动该对象，而不是等待上下文或其自身的显式调用`start()`方法（与上下文刷新不同，对于标准上下文实现，上下文启动不会自动发生）。该`phase`值与任何“依赖式”的关系确定为前面所述的启动顺序。

##### 在非Web应用程序中正常关闭Spring IoC容器

|      | 本节仅适用于非Web应用程序。Spring的基于Web的 `ApplicationContext`实现已经有了相应的代码，可以在相关Web应用程序关闭时正常关闭Spring IoC容器。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果您在非Web应用程序环境中（例如，在富客户端桌面环境中）使用Spring的IoC容器，请向JVM注册一个关闭钩子。这样做可以确保正常关机，并在您的Singleton bean上调用相关的destroy方法，以便释放所有资源。您仍然必须正确配置和实现这些destroy回调。

要注册关闭钩子，请调用接口`registerShutdownHook()`上声明的方法`ConfigurableApplicationContext`，如以下示例所示：

爪哇

科特林

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

#### 1.6.2。`ApplicationContextAware`和`BeanNameAware`

当`ApplicationContext`创建创建实现该`org.springframework.context.ApplicationContextAware`接口的对象实例时，该实例将获得对该 接口的引用`ApplicationContext`。以下清单显示了`ApplicationContextAware`接口的定义：

爪哇

科特林

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

因此，bean可以`ApplicationContext`通过`ApplicationContext`接口或通过将引用转换为该接口的已知子类（例如`ConfigurableApplicationContext`，公开其他功能）来以编程方式操纵创建它们的bean 。一种用途是通过编程方式检索其他bean。有时，此功能很有用。但是，通常应避免使用它，因为它会将代码耦合到Spring，并且不遵循控制反转样式，在该样式中，将协作者作为属性提供给bean。提供的其他方法 `ApplicationContext`提供对文件资源的访问，发布应用程序事件以及访问`MessageSource`。这些附加功能在的 [附加功能中进行了介绍`ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#context-introduction)。

自动装配是获得对的引用的另一种方法 `ApplicationContext`。的*传统的* `constructor`和`byType`自动装配模式（在所描述的[自动装配协作者](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-autowire)）可以提供类型的依赖 `ApplicationContext`于构造器参数或设置器方法参数，分别。要获得更大的灵活性，包括能够自动连接字段和使用多个参数方法，请使用基于注释的自动装配功能。如果您这样做，则将`ApplicationContext`自动将其连接到需要该`ApplicationContext`类型的字段，构造函数参数或方法参数中（如果相关的字段，构造函数或方法带有`@Autowired`注释）。有关更多信息，请参见 [使用`@Autowired`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-autowired-annotation)。

当`ApplicationContext`创建一个实现该 `org.springframework.beans.factory.BeanNameAware`接口的类时，该类将获得对其关联对象定义中定义的名称的引用。以下清单显示了BeanNameAware接口的定义：

爪哇

科特林

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

回调正常bean属性的人口之后，但在一个初始化回调诸如调用`InitializingBean`，`afterPropertiesSet`或自定义的初始化方法。

#### 1.6.3。其他`Aware`介面

除了`ApplicationContextAware`和`BeanNameAware`（前面[已经](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-aware)讨论[过](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-aware)），Spring还提供了各种各样的`Aware`回调接口，这些接口使bean可以向容器指示它们需要某种基础结构依赖性。通常，名称表示依赖项类型。下表总结了最重要的`Aware`接口：

| 名称                             | 注入依赖                                                     | 在...中解释                                                  |
| :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `ApplicationContextAware`        | 宣告`ApplicationContext`。                                   | [`ApplicationContextAware` 和 `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-aware) |
| `ApplicationEventPublisherAware` | 附件的事件发布者`ApplicationContext`。                       | [的其他功能 `ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#context-introduction) |
| `BeanClassLoaderAware`           | 类加载器，用于加载Bean类。                                   | [实例化豆](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-class) |
| `BeanFactoryAware`               | 宣告`BeanFactory`。                                          | [`ApplicationContextAware` 和 `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-aware) |
| `BeanNameAware`                  | 声明bean的名称。                                             | [`ApplicationContextAware` 和 `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-aware) |
| `BootstrapContextAware`          | `BootstrapContext`容器在其中运行的资源适配器。通常仅在支持JCA的`ApplicationContext`实例中可用。 | [JCA CCI](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#cci) |
| `LoadTimeWeaverAware`            | 定义的编织器，用于在加载时处理类定义。                       | [在Spring Framework中使用AspectJ进行加载时编织](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-aj-ltw) |
| `MessageSourceAware`             | 解决消息的已配置策略（支持参数化和国际化）。                 | [的其他功能 `ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#context-introduction) |
| `NotificationPublisherAware`     | Spring JMX通知发布者。                                       | [通知事项](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#jmx-notifications) |
| `ResourceLoaderAware`            | 配置的加载程序，用于对资源的低级别访问。                     | [资源资源](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources) |
| `ServletConfigAware`             | 当前`ServletConfig`容器在其中运行。仅在可感知网络的Spring中有效 `ApplicationContext`。 | [春季MVC](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc) |
| `ServletContextAware`            | 当前`ServletContext`容器在其中运行。仅在可感知网络的Spring中有效 `ApplicationContext`。 | [春季MVC](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc) |

再次注意，使用这些接口会将您的代码与Spring API绑定在一起，并且不遵循“控制反转”样式。因此，我们建议将它们用于需要以编程方式访问容器的基础结构Bean。

### 1.7。Bean定义继承

Bean定义可以包含许多配置信息，包括构造函数参数，属性值和特定于容器的信息，例如初始化方法，静态工厂方法名称等。子bean定义从父定义继承配置数据。子定义可以覆盖某些值或根据需要添加其他值。使用父bean和子bean定义可以节省很多输入。实际上，这是一种模板形式。

如果您以`ApplicationContext`编程方式使用接口，则子bean定义由`ChildBeanDefinition`类表示。大多数用户不在此级别上与他们合作。相反，它们在诸如之类的类中声明性地配置Bean定义`ClassPathXmlApplicationContext`。当使用基于XML的配置元数据时，可以通过使用`parent`属性来指定子bean定义，并将父bean指定为该属性的值。以下示例显示了如何执行此操作：

```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

|      | 注意该`parent`属性。 |
| ---- | -------------------- |
|      |                      |

如果未指定子bean定义，则使用父定义中的bean类，但也可以覆盖它。在后一种情况下，子bean类必须与父类兼容（也就是说，它必须接受父类的属性值）。

子bean定义从父项继承范围，构造函数参数值，属性值和方法替代，并可以选择添加新值。`static`您指定的任何范围，初始化方法，destroy方法或工厂方法设置都会覆盖相应的父设置。

其余设置始终从子定义中获取：依赖项，自动装配模式，依赖项检查，单例和惰性初始化。

前面的示例通过使用`abstract`属性将父bean定义显式标记为抽象。如果父定义未指定类，请根据`abstract`需要显式标记父bean定义，如以下示例所示：

```xml
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```

父bean不能单独实例化，因为它不完整，并且也被显式标记为`abstract`。当定义`abstract`为时，只能用作纯模板bean定义，用作子定义的父定义。尝试`abstract`通过将其称为另一个bean的ref属性来单独使用此类父bean或`getBean()`使用父bean ID 进行显式调用会返回错误。同样，容器的内部 `preInstantiateSingletons()`方法将忽略定义为抽象的bean定义。

|      | `ApplicationContext`默认情况下预先实例化所有单例。因此，重要的是（至少对于单例bean），如果有一个（父）bean定义仅打算用作模板，并且此定义指定了一个类，则必须确保将*abstract*属性设置为*true*，否则应用程序上下文将实际（试图）预先实例化`abstract`bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.8。集装箱延伸点

通常，应用程序开发人员不需要对`ApplicationContext` 实现类进行子类化。相反，可以通过插入特殊集成接口的实现来扩展Spring IoC容器。接下来的几节描述了这些集成接口。

#### 1.8.1。使用a定制Bean`BeanPostProcessor`

该`BeanPostProcessor`接口定义了回调方法，您可以实现这些回调方法以提供自己的（或覆盖容器的默认值）实例化逻辑，依赖关系解析逻辑等。如果您想在Spring容器完成实例化，配置和初始化bean之后实现一些自定义逻辑，则可以插入一个或多个自定义`BeanPostProcessor`实现。

您可以配置多个`BeanPostProcessor`实例，并且可以`BeanPostProcessor`通过设置`order`属性来控制这些实例的执行顺序。仅当`BeanPostProcessor`实现`Ordered` 接口时才可以设置此属性。如果您编写自己的代码`BeanPostProcessor`，则也应该考虑实现该`Ordered`接口。有关更多详细信息，请参见[`BeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html) 和[`Ordered`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/Ordered.html)接口的javadoc 。另请参见有关[实例](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-programmatically-registering-beanpostprocessors)[编程注册`BeanPostProcessor`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-programmatically-registering-beanpostprocessors)的说明。

|      | `BeanPostProcessor`实例在bean（或对象）实例上运行。也就是说，Spring IoC容器实例化一个bean实例，然后`BeanPostProcessor` 实例执行其工作。`BeanPostProcessor`实例是按容器划分作用域的。仅在使用容器层次结构时，这才有意义。如果`BeanPostProcessor`在一个容器中定义一个，它将仅对该容器中的bean进行后处理。换句话说，一个容器中定义的bean不会被`BeanPostProcessor`另一个容器中的定义进行后处理，即使这两个容器是同一层次结构的一部分也是如此。要更改实际的bean定义（即定义bean的蓝图），您需要使用a `BeanFactoryPostProcessor`，如 使用“ [定制配置元数据”中所述`BeanFactoryPostProcessor`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-extension-factory-postprocessors)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

该`org.springframework.beans.factory.config.BeanPostProcessor`接口恰好由两个回调方法组成。当此类被注册为容器的后处理器时，对于容器创建的每个bean实例，后处理器都会在容器初始化方法（例如`InitializingBean.afterPropertiesSet()`或任何声明的`init`方法）被使用之前从容器获得回调。在任何bean初始化回调之后调用。后处理器可以对bean实例执行任何操作，包括完全忽略回调。Bean后处理器通常检查回调接口，或者可以用代理包装Bean。一些Spring AOP基础结构类被实现为bean后处理器，以提供代理包装逻辑。

A `ApplicationContext`自动检测实现该`BeanPostProcessor`接口的配置元数据中定义的所有bean 。将 `ApplicationContext`这些Bean注册为后处理器，以便以后在创建Bean时调用它们。Bean后处理器可以与其他Bean相同的方式部署在容器中。

请注意，在配置类上`BeanPostProcessor`通过使用`@Bean`工厂方法声明a时，工厂方法的返回类型应该是实现类本身或至少是`org.springframework.beans.factory.config.BeanPostProcessor` 接口，以清楚地表明该bean的后处理器性质。否则， `ApplicationContext`无法在完全创建之前按类型自动检测它。由于a `BeanPostProcessor`需要提前实例化以便应用于上下文中其他bean的初始化，因此这种早期类型检测至关重要。

|      | 以编程方式注册`BeanPostProcessor`实例虽然建议的`BeanPostProcessor`注册方法是通过 `ApplicationContext`自动检测（如前所述），但是您可以`ConfigurableBeanFactory`使用`addBeanPostProcessor` 方法以编程方式针对进行注册。当您需要在注册之前评估条件逻辑，甚至需要跨层次结构中的上下文复制bean后处理器时，这将非常有用。但是请注意，以`BeanPostProcessor`编程方式添加的实例不遵守该`Ordered`接口。在这里，注册的顺序决定了执行的顺序。还要注意，以`BeanPostProcessor`编程方式注册的实例总是在通过自动检测注册的实例之前进行处理，而不考虑任何明确的顺序。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | `BeanPostProcessor` 实例和AOP自动代理实现该`BeanPostProcessor`接口的类是特殊的，并且容器对它们的处理方式有所不同。`BeanPostProcessor`它们直接引用的所有实例和bean在启动时都会实例化，作为的特殊启动阶段的一部分`ApplicationContext`。接下来，`BeanPostProcessor`以排序的方式注册所有实例，并将其应用于容器中的所有其他bean。因为AOP自动代理`BeanPostProcessor`本身是实现的，所以`BeanPostProcessor` 实例或它们直接引用的bean都没有资格进行自动代理，因此没有编织的方面。对于任何此类bean，您应该看到一条参考日志消息：`Bean someBean is not eligible for getting processed by all BeanPostProcessor interfaces (for example: not eligible for auto-proxying)`。如果您`BeanPostProcessor`使用自动装配或`@Resource`（可能会退回到自动装配）将Bean连接到您的 bean ，则Spring在搜索类型匹配的依赖项候选对象时可能会访问意外的bean，因此使它们不符合自动代理或其他种类的bean的要求。处理。例如，如果您有一个依赖项，`@Resource`其中字段或设置器名称不直接与bean的声明名称相对应，并且不使用name属性，则Spring将访问其他bean以按类型匹配它们。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下示例显示了如何在中编写，注册和使用`BeanPostProcessor`实例`ApplicationContext`。

##### 示例：Hello World，`BeanPostProcessor` -style

第一个示例说明了基本用法。该示例显示了一个定制 `BeanPostProcessor`实现，该实现调用`toString()`容器创建每个bean 的方法并将其输出到系统控制台。

以下清单显示了定制`BeanPostProcessor`实现类定义：

爪哇

科特林

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```

以下`beans`元素使用`InstantiationTracingBeanPostProcessor`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/lang
        https://www.springframework.org/schema/lang/spring-lang.xsd">

    <lang:groovy id="messenger"
            script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
        <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
    </lang:groovy>

    <!--
    when the above bean (messenger) is instantiated, this custom
    BeanPostProcessor implementation will output the fact to the system console
    -->
    <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>
```

注意`InstantiationTracingBeanPostProcessor`仅如何定义。它甚至没有名称，并且因为它是Bean，所以可以像其他任何Bean一样对其进行依赖注入。（前面的配置还定义了一个由Groovy脚本支持的bean。Spring动态语言支持在标题为[Dynamic Language Support](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/languages.html#dynamic-language)的章节中有详细介绍 。）

以下Java应用程序运行上述代码和配置：

爪哇

科特林

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
        Messenger messenger = ctx.getBean("messenger", Messenger.class);
        System.out.println(messenger);
    }

}
```

前面的应用程序的输出类似于以下内容：

```
创建的Bean“信使”：org.springframework.scripting.groo vy.GroovyMessenger@272961 
org.springframework.scripting.groovy.GroovyMessenger@272961
```

##### 示例： `RequiredAnnotationBeanPostProcessor`

将回调接口或注释与自定义`BeanPostProcessor`实现结合使用 是扩展Spring IoC容器的常用方法。一个例子是Spring的`RequiredAnnotationBeanPostProcessor` -一个`BeanPostProcessor`与Spring发行版一起提供的实现，该 实现可确保在Bean上标有（任意）批注的JavaBean属性实际上（配置为）依赖注入了一个值。

#### 1.8.2。使用以下命令自定义配置元数据`BeanFactoryPostProcessor`

我们要看的下一个扩展点是 `org.springframework.beans.factory.config.BeanFactoryPostProcessor`。该接口的语义与相似，但`BeanPostProcessor`有一个主要区别：`BeanFactoryPostProcessor`对Bean配置元数据进行操作。也就是说，Spring IoC容器允许`BeanFactoryPostProcessor`读取配置元数据，并有可能在容器实例化实例以外的任何bean *之前*更改它`BeanFactoryPostProcessor`。

您可以配置多个`BeanFactoryPostProcessor`实例，并且可以`BeanFactoryPostProcessor`通过设置`order`属性来控制这些实例的运行顺序。但是，只有在`BeanFactoryPostProcessor`实现`Ordered`接口的情况下 才能设置此属性。如果您编写自己的代码`BeanFactoryPostProcessor`，则也应该考虑实现该`Ordered`接口。有关更多详细信息，请参见[`BeanFactoryPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html)和[`Ordered`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/Ordered.html)接口的javadoc 。

|      | 如果要更改实际的bean实例（即，从配置元数据创建的对象），则需要使用a `BeanPostProcessor` （前面在[使用a定制Bean中所述`BeanPostProcessor`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-extension-bpp)）。从技术上讲，可以使用`BeanFactoryPostProcessor`（例如，通过使用 `BeanFactory.getBean()`）中的bean实例，但这会导致bean实例化过早，从而违反了标准容器的生命周期。这可能会导致负面影响，例如绕过bean后处理。同样，`BeanFactoryPostProcessor`实例是按容器划分作用域的。仅在使用容器层次结构时才有意义。如果`BeanFactoryPostProcessor`在一个容器中定义a ，则仅将其应用于该容器中的bean定义。一个容器中的Bean定义不会被`BeanFactoryPostProcessor`另一个容器中的实例后处理，即使两个容器都属于同一层次结构也是如此。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

将Bean工厂后处理器声明为时，它将自动执行 `ApplicationContext`，以便将更改应用于定义容器的配置元数据。Spring包含许多预定义的bean工厂后处理器，例如`PropertyOverrideConfigurer`和 `PropertySourcesPlaceholderConfigurer`。您还可以使用自定义`BeanFactoryPostProcessor` -例如，注册自定义属性编辑器。

A `ApplicationContext`自动检测实现该`BeanFactoryPostProcessor`接口的部署到其中的所有bean 。它在适当的时候将这些bean用作bean工厂的后处理器。您可以像部署其他任何bean一样部署这些后处理器bean。

|      | 与`BeanPostProcessor`s一样，您通常不希望将`BeanFactoryPostProcessor`s 配置 为延迟初始化。如果没有其他bean引用a `Bean(Factory)PostProcessor`，则该后处理器将完全不会实例化。因此，将其标记为延迟初始化将被忽略，并且`Bean(Factory)PostProcessor`即使您在元素声明中将`default-lazy-init`属性设置为， 也会立即实例化 。 `true``` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 示例：类名替换 `PropertySourcesPlaceholderConfigurer`

您可以使用`PropertySourcesPlaceholderConfigurer`标准Java `Properties`格式，使用来从单独文件中的bean定义外部化属性值。这样做使部署应用程序的人员可以自定义特定于环境的属性，例如数据库URL和密码，而无需为修改容器的主要XML定义文件而复杂或冒风险。

考虑以下基于XML的配置元数据片段，其中`DataSource` 定义了带有占位符的值：

```xml
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
    <property name="locations" value="classpath:com/something/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

该示例显示了从外部`Properties`文件配置的属性。在运行时，将a `PropertySourcesPlaceholderConfigurer`应用于替换数据源某些属性的元数据。将要替换的值指定为形式的占位符，该形式`${property-name}`遵循Ant和log4j和JSP EL样式。

实际值来自标准Java `Properties`格式的另一个文件：

```
jdbc.driverClassName = org.hsqldb.jdbcDriver
jdbc.url = jdbc：hsqldb：hsql：// production：9002
jdbc.username = sa
jdbc.password =根
```

因此，`${jdbc.username}`在运行时将字符串替换为值“ sa”，并且其他与属性文件中的键匹配的占位符值也适用。在`PropertySourcesPlaceholderConfigurer`为大多数属性和bean定义的属性占位符检查。此外，您可以自定义占位符前缀和后缀。

借助`context`Spring 2.5中引入的名称空间，您可以使用专用配置元素配置属性占位符。您可以在`location`属性中提供一个或多个位置作为逗号分隔的列表，如以下示例所示：

```xml
<context:property-placeholder location="classpath:com/something/jdbc.properties"/>
```

在`PropertySourcesPlaceholderConfigurer`不仅将查找在属性`Properties` 指定的文件。默认情况下，如果无法在指定的属性文件中找到属性，则将检查Spring `Environment`属性和常规Java `System`属性。

|      | 您可以使用`PropertySourcesPlaceholderConfigurer`代替类名，当您必须在运行时选择特定的实现类时，这有时很有用。以下示例显示了如何执行此操作：`            classpath:com/something/strategy.properties                custom.strategy.class=com.something.DefaultStrategy      `如果无法在运行时将类解析为有效的类，则在将要创建该bean时（在非延迟初始化bean 的`preInstantiateSingletons()` 阶段），将无法解析该`ApplicationContext`bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 示例： `PropertyOverrideConfigurer`

在`PropertyOverrideConfigurer`另一个bean工厂后处理器，类似 `PropertySourcesPlaceholderConfigurer`，但不同的是后者，原来的定义可以在所有的bean属性有缺省值或者根本没有值。如果覆盖 `Properties`文件没有某个bean属性的条目，则使用默认的上下文定义。

注意，bean定义不知道会被覆盖，因此从XML定义文件中不能立即看出正在使用覆盖配置器。如果有多个`PropertyOverrideConfigurer`实例为同一个bean属性定义了不同的值，则由于覆盖机制，最后一个实例将获胜。

属性文件配置行采用以下格式：

```
beanName.property =值
```

下面的清单显示了格式的示例：

```
dataSource.driverClassName = com.mysql.jdbc.Driver
dataSource.url = jdbc：mysql：mydb
```

此示例文件可与包含定义为`dataSource`具有`driver`和`url`属性的bean的容器定义一起使用 。

只要路径的每个组成部分（最终属性被覆盖）之外的所有组成部分都已经为非空（可能是由构造函数初始化），则也支持复合属性名。在以下示例中，bean `sammy`的`bob`property的`fred`property属性`tom`设置为标量值`123`：

```
tom.fred.bob.sammy = 123
```

|      | 指定的替代值始终是文字值。它们不会转换为bean引用。当XML bean定义中的原始值指定bean引用时，此约定也适用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

使用`context`Spring 2.5中引入的名称空间，可以使用专用配置元素配置属性覆盖，如以下示例所示：

```xml
<context:property-override location="classpath:override.properties"/>
```

#### 1.8.3。自定义实例化逻辑`FactoryBean`

您可以`org.springframework.beans.factory.FactoryBean`为本身就是工厂的对象实现接口。

该`FactoryBean`接口是可插入Spring IoC容器的实例化逻辑的一点。如果您拥有复杂的初始化代码，而不是（可能）冗长的XML量，可以用Java更好地表达，则可以创建自己的代码 `FactoryBean`，在该类中编写复杂的初始化，然后将自定义`FactoryBean`插入容器。

该`FactoryBean`界面提供了三种方法：

- `Object getObject()`：返回此工厂创建的对象的实例。实例可以共享，具体取决于该工厂是否返回单例或原型。
- `boolean isSingleton()`：`true`如果`FactoryBean`返回单例或`false`其他则返回 。
- `Class getObjectType()`：返回`getObject()`方法返回的对象类型，或者`null`如果类型未知则返回该对象类型。

`FactoryBean`Spring框架中的许多地方都使用了该概念和接口。`FactoryBean`Spring附带了50多种接口实现。

当您需要向容器询问`FactoryBean`本身而不是由它产生的bean的实际实例时，请在调用的方法时在该bean的`id`前面加上“＆”符号（`&`）。因此，对于给定 与的，调用在容器回报的产品，而调用返回的 实例本身。`getBean()``ApplicationContext``FactoryBean``id``myBean``getBean("myBean")``FactoryBean``getBean("&myBean")``FactoryBean`

### 1.9。基于注释的容器配置

在配置Spring时，注释是否比XML更好？

基于注释的配置的引入提出了一个问题，即这种方法是否比XML“更好”。简短的答案是“取决于情况”。长的答案是每种方法都有其优缺点，通常，由开发人员决定哪种策略更适合他们。由于定义方式的不同，注释在声明中提供了很多上下文，从而使配置更短，更简洁。但是，XML擅长连接组件而不接触其源代码或重新编译它们。一些开发人员更喜欢将布线放置在靠近源的位置，而另一些开发人员则认为带注释的类不再是POJO，而且，该配置变得分散且难以控制。

无论选择如何，Spring都可以容纳两种样式，甚至可以将它们混合在一起。值得指出的是，通过[JavaConfig](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java)选项，Spring允许以非侵入方式使用注释，而无需接触目标组件的源代码，并且就工具而言，[Spring Tools for Eclipse](https://spring.io/tools)支持所有配置样式 。

基于注释的配置提供了XML设置的替代方法，该配置依赖字节码元数据来连接组件，而不是尖括号声明。通过使用相关类，方法或字段声明上的注释，开发人员无需使用XML来描述bean的连接，而是将配置移入组件类本身。如[示例中所述：将`RequiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-extension-bpp-examples-rabpp)，`BeanPostProcessor`与注释结合使用是扩展Spring IoC容器的常用方法。例如，Spring 2.0引入了通过[`@Required`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-required-annotation)注释强制执行必需属性的可能性。Spring 2.5使遵循相同的通用方法来驱动Spring的依赖注入成为可能。本质上，`@Autowired`注解提供的功能与[自动装配协作器中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-autowire)所述的功能相同，但具有更细粒度的控制和更广泛的适用性。Spring 2.5还添加了对JSR-250批注（例如 `@PostConstruct`和）的支持`@PreDestroy`。弹簧3.0 JSR-330（Java依赖注入）加入支持注释包含在`javax.inject`包如`@Inject` 和`@Named`。有关这些注释的详细信息，请参见 [相关章节](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-standard-annotations)。

|      | 注释注入在XML注入之前执行。因此，XML配置将覆盖通过两种方法连接的属性的注释。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

与往常一样，您可以将它们注册为单独的bean定义，但也可以通过在基于XML的Spring配置中包括以下标记来隐式注册它们（注意，包括`context`名称空间）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

（隐式注册的后处理器包括 [`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html)， [`CommonAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/CommonAnnotationBeanPostProcessor.html)和 [`PersistenceAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/orm/jpa/support/PersistenceAnnotationBeanPostProcessor.html)，以及上述 [`RequiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/RequiredAnnotationBeanPostProcessor.html)。）

|      | ``只在定义它的相同应用程序上下文中查找关于bean的注释。这意味着，如果 ``在中输入`WebApplicationContext`for `DispatcherServlet`，则仅检查`@Autowired`控制器中的bean，而不检查服务中的bean。有关更多信息，请参见 [DispatcherServlet](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-servlet)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.9.1。@需要

该`@Required`注释适用于bean属性setter方法，如下面的例子：

爪哇

科特林

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

此注释指示必须在配置时通过bean定义中的显式属性值或通过自动装配来填充受影响的bean属性。如果尚未填充受影响的bean属性，则容器将引发异常。这允许急切和显式的故障，避免`NullPointerException` 以后再发生实例等。我们仍然建议您将断言放入bean类本身中（例如，放入init方法中）。这样做会强制执行那些必需的引用和值，即使您在容器外部使用该类也是如此。

|      | 从`@Required`Spring Framework 5.1开始，正式弃用了该批注，以支持对所需的设置（或`InitializingBean.afterPropertiesSet()`Bean属性setter方法的自定义实现）使用构造函数注入 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.9.2。使用`@Autowired`

|      | 在本节中的示例中，`@Inject`可以使用JSR 330的注释代替Spring的`@Autowired`注释。有关更多详细信息，请参见[此处](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-standard-annotations)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您可以将`@Autowired`注释应用于构造函数，如以下示例所示：

爪哇

科特林

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

|      | 从Spring Framework 4.3开始，`@Autowired`如果目标bean仅定义一个构造函数作为开始，则不再需要在此类构造函数上添加注释。但是，如果有几个构造函数可用，并且没有主/默认构造函数，则必须至少注释一个构造函数，`@Autowired`以指示容器使用哪个构造函数。有关详细信息，请参见有关[构造函数解析](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-autowired-annotation-constructor-resolution)的讨论 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以将`@Autowired`注释应用于*传统的* setter方法，如以下示例所示：

爪哇

科特林

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

您还可以将注释应用于具有任意名称和多个参数的方法，如以下示例所示：

爪哇

科特林

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

您还可以将其应用于`@Autowired`字段，甚至将其与构造函数混合使用，如以下示例所示：

爪哇

科特林

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

|      | 确保目标组件（例如`MovieCatalog`或`CustomerPreferenceDao`）由用于带`@Autowired`注释的注入点的类型一致地声明。否则，注入可能会由于运行时出现“找不到类型匹配”错误而失败。对于通过类路径扫描找到的XML定义的bean或组件类，容器通常预先知道具体的类型。但是，对于`@Bean`工厂方法，您需要确保声明的返回类型具有足够的表现力。对于实现多个接口的组件或可能由其实现类型引用的组件，请考虑在工厂方法中声明最具体的返回类型（至少根据引用您的bean的注入点的要求具体声明）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以`ApplicationContext`通过将`@Autowired`注释添加到需要该类型数组的字段或方法中，指示Spring提供特定类型的所有bean ，如以下示例所示：

爪哇

科特林

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```

如下例所示，这同样适用于类型化集合：

爪哇

科特林

```java
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

|      | 如果要使数组或列表中的项目以特定顺序排序，则目标bean可以实现`org.springframework.core.Ordered`接口或使用`@Order`或标准`@Priority`注释。否则，它们的顺序将遵循容器中相应目标bean定义的注册顺序。您可以`@Order`在目标类级别和`@Bean`方法上声明注释，可能是针对单个bean定义（在使用同一bean类的多个定义的情况下）。`@Order`值可能会影响注入点的优先级，但请注意它们不会影响单例启动顺序，这是由依赖关系和`@DependsOn`声明确定的正交关注点。请注意，标准`javax.annotation.Priority`注释在该`@Bean`级别不可用 ，因为无法在方法上声明它。可以通过将每种类型的`@Order`值与`@Primary`单个bean 结合使用来对其语义进行建模。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`Map`只要预期的密钥类型为，即使是键入的实例也可以自动装配`String`。映射值包含所有预期类型的bean，并且键包含相应的bean名称，如以下示例所示：

爪哇

科特林

```java
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

默认情况下，当给定注入点没有匹配的候选bean可用时，自动装配将失败。对于声明的数组，集合或映射，至少应有一个匹配元素。

默认行为是将带注释的方法和字段视为指示所需的依赖项。您可以更改此行为，如以下示例所示，使框架可以通过将其标记为不需要来跳过不满意的注入点（即，将`required`属性设置`@Autowired`为`false`）：

爪哇

科特林

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

如果不需要的方法（或在多个参数的情况下，其中一个依赖项）不可用，则根本不会调用该方法。在这种情况下，完全不需要填充非必需字段，而将其默认值保留在适当的位置。

注入的构造函数和工厂方法参数是一种特殊情况，因为由于Spring的构造函数解析算法可能会处理多个构造函数，所以`required` in `@Autowired`中的属性含义有所不同。默认情况下，有效地需要构造函数和工厂方法参数，但是在单构造函数场景中有一些特殊规则，例如，如果没有可用的匹配bean，则多元素注入点（数组，集合，映射）解析为空实例。这允许一种通用的实现模式，其中所有依赖项都可以在唯一的多参数构造函数中声明，例如，声明为不带`@Autowired`注释的单个公共构造函数。

|      | 只有任何给定的bean类的一个构造可以宣告`@Autowired`与`required` 属性设置为`true`，表明*该*构造函数自动装配作为一个Spring bean使用时。结果，如果将`required`属性保留为其默认值`true`，则只能使用注释单个构造函数`@Autowired`。如果多个构造函数声明注解，则都必须声明它们`required=false`才能被视为自动装配的候选对象（类似于`autowire=constructor`XML）。将选择通过匹配Spring容器中的bean可以满足的依赖关系数量最多的构造函数。如果没有一个候选者满意，则将使用主/默认构造函数（如果存在）。类似地，如果一个类声明了多个构造函数，但都没有用注释`@Autowired`，则将使用主/默认构造函数（如果存在）。如果一个类仅声明一个单一的构造函数开始，即使没有注释，也将始终使用它。请注意，带注释的构造函数不必是公共的。建议 在setter方法的不建议使用的批注上使用的`required`属性。将属性设置为表示该属性对于自动装配不是必需的，并且如果不能自动装配该属性，则将忽略该属性。另一方面，它更强大，因为它强制通过容器支持的任何方式来设置属性，并且如果未定义任何值，则会引发相应的异常。`@Autowired``@Required``required``false``@Required` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

另外，您可以通过Java 8来表达特定依赖项的非必需性质`java.util.Optional`，如以下示例所示：

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

从Spring Framework 5.0开始，您还可以使用`@Nullable`注释（任何包中的任何注释，例如，`javax.annotation.Nullable`来自JSR-305 的注释），或仅利用Kotlin内置的null安全支持：

爪哇

科特林

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

您还可以使用`@Autowired`对于那些众所周知的解析依赖接口：`BeanFactory`，`ApplicationContext`，`Environment`，`ResourceLoader`， `ApplicationEventPublisher`，和`MessageSource`。这些接口及其扩展接口（例如`ConfigurableApplicationContext`或`ResourcePatternResolver`）将自动解析，而无需进行特殊设置。以下示例自动装配`ApplicationContext`对象：

爪哇

科特林

```java
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

|      | 在`@Autowired`，`@Inject`，`@Value`，和`@Resource`注释由Spring处理 `BeanPostProcessor`实现。这意味着您不能在自己的类型`BeanPostProcessor`或`BeanFactoryPostProcessor`类型（如果有）中应用这些注释。这些类型必须使用XML或Spring `@Bean`方法显式地“连接” 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.9.3。通过微调基于注释的自动装配`@Primary`

由于按类型自动布线可能会导致多个候选对象，因此通常有必要对选择过程进行更多控制。实现此目的的一种方法是使用Spring的 `@Primary`注释。`@Primary`指示当多个bean是要自动装配到单值依赖项的候选对象时，应给予特定bean优先权。如果候选中恰好存在一个主bean，它将成为自动装配的值。

考虑以下定义`firstMovieCatalog`为主要配置的配置`MovieCatalog`：

爪哇

科特林

```java
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

使用前面的配置，以下内容`MovieRecommender`将自动连接到 `firstMovieCatalog`：

爪哇

科特林

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

相应的bean定义如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

#### 1.9.4。使用限定符对基于注释的自动装配进行微调

`@Primary`当可以确定一个主要候选对象时，它是在几种情况下按类型使用自动装配的有效方法。当您需要更好地控制选择过程时，可以使用Spring的`@Qualifier`注释。您可以将限定符值与特定的参数相关联，从而缩小类型匹配的范围，以便为每个参数选择特定的bean。在最简单的情况下，这可以是简单的描述性值，如以下示例所示：

爪哇

科特林

```java
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```

您还可以`@Qualifier`在各个构造函数参数或方法参数上指定注释，如以下示例所示：

爪哇

科特林

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main") MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

以下示例显示了相应的bean定义。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

|      | 具有`main`限定符值的Bean与限定有相同值的构造函数参数连接。   |
| ---- | ------------------------------------------------------------ |
|      | 具有`action`限定符值的Bean与限定有相同值的构造函数参数连接。 |

对于后备匹配，bean名称被视为默认的限定符值。因此，可以使用`id`of `main`而不是嵌套的限定符元素来定义bean ，从而得到相同的匹配结果。但是，尽管可以使用此约定按名称引用特定的bean，但从`@Autowired`根本上讲，它是带有可选语义限定符的类型驱动的注入。这意味着限定符值，即使具有bean名称回退，也总是在类型匹配集中具有狭窄的语义。它们没有在语义上表示对唯一bean的引用`id`。好的限定符值是`main` 或`EMEA`或`persistent`，表示独立于Bean的特定组件的特征`id`，如果是匿名Bean定义（例如上例中的定义），则可以自动生成。

限定符还适用于类型化的集合，如前所述（例如，应用于） `Set`。在这种情况下，根据声明的限定词，所有匹配的bean都作为一个集合注入。这意味着限定词不必是唯一的。相反，它们构成了过滤标准。例如，您可以`MovieCatalog`使用相同的限定符值“ action” 来定义多个bean，所有这些都将注入到带有的`Set`注释中`@Qualifier("action")`。

|      | 在类型匹配的候选对象中，让限定符值针对目标bean名称进行选择，在`@Qualifier`注入点不需要注释。如果没有其他解析度指示符（例如限定词或主标记），则对于非唯一依赖性情况，Spring将注入点名称（即字段名称或参数名称）与目标Bean名称进行匹配，然后选择同名候选人（如果有）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

就是说，如果您打算按名称表示注释驱动的注入，则不要主要使用`@Autowired`，即使它能够在类型匹配的候选对象中按bean名称进行选择。而是使用JSR-250 `@Resource`批注，该批注的语义定义是通过其唯一名称来标识特定目标组件，而声明的类型与匹配过程无关。`@Autowired`具有不同的语义：按类型选择候选bean之后，`String` 仅在那些类型选择的候选对象中考虑指定的限定符值（例如，将`account`限定符与标记有相同限定符标签的bean进行匹配）。

对于本身定义为collection `Map`或array类型的`@Resource` bean是一个很好的解决方案，可以通过唯一的名称引用特定的collection或array bean。也就是说，从4.3版本开始，只要元素类型信息保留在返回类型签名或集合继承层次结构中，就可以`Map`通过Spring的`@Autowired`类型匹配算法来匹配和数组类型 `@Bean`。在这种情况下，您可以使用限定符值在同类型的集合中进行选择，如上一段所述。

从4.3开始，`@Autowired`还考虑了自我注入的引用（即，对当前注入的Bean的引用）。请注意，自我注入是一个后备。对其他组件的常规依赖始终优先。从这个意义上说，自我推荐不参与常规的候选人选择，因此尤其是绝不是主要的。相反，它们总是以最低优先级结束。实际上，您应该仅将自引用用作最后的手段（例如，通过bean的事务代理在同一实例上调用其他方法）。考虑在这种情况下将受影响的方法分解为单独的委托bean。或者，您可以使用`@Resource`，它可以通过其唯一名称获取返回到当前bean的代理。

|      | 尝试从`@Bean`相同配置类上的方法中注入结果也是有效的自引用方案。要么在实际需要的方法签名中延迟解析这些引用（与配置类中的自动装配字段相对），要么将受影响的`@Bean`方法声明为`static`，将它们与包含的配置类实例及其生命周期脱钩。否则，仅在回退阶段考虑此类Bean，而将其他配置类上的匹配Bean选作主要候选对象（如果可用）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`@Autowired`适用于字段，构造函数和多参数方法，从而允许在参数级别缩小限定符注释的范围。相反，`@Resource` 仅支持具有单个参数的字段和bean属性设置器方法。因此，如果注入目标是构造函数或多参数方法，则应坚持使用限定符。

您可以创建自己的自定义限定符注释。为此，请定义一个注释并`@Qualifier`在您的定义中提供该注释，如以下示例所示：

爪哇

科特林

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```

然后，您可以在自动连接的字段和参数上提供自定义限定符，如以下示例所示：

爪哇

科特林

```java
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;

    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...
}
```

接下来，您可以提供有关候选bean定义的信息。您可以将``标签添加为 标签的子元素，``然后指定`type`和 `value`以匹配您的自定义限定符注释。该类型与注释的完全限定的类名匹配。另外，为方便起见，如果不存在名称冲突的风险，则可以使用简短的类名。下面的示例演示了两种方法：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="example.Genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

在“ [类路径扫描和托管组件”中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-classpath-scanning)，您可以看到基于注释的替代方法，以XML提供限定符元数据。具体来说，请参阅[为Qualifier元数据提供注释](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-scanning-qualifiers)。

在某些情况下，使用没有值的注释就足够了。当注释具有更一般的用途并且可以应用于几种不同类型的依赖项时，这将很有用。例如，您可以提供一个脱机目录，当没有Internet连接可用时，可以对其进行搜索。首先，定义简单注释，如以下示例所示：

爪哇

科特林

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {

}
```

然后将注释添加到要自动装配的字段或属性，如以下示例所示：

爪哇

科特林

```java
public class MovieRecommender {

    @Autowired
    @Offline 
    private MovieCatalog offlineCatalog;

    // ...
}
```

|      | 这行添加`@Offline`注释。 |
| ---- | ------------------------ |
|      |                          |

现在，bean定义只需要一个限定符`type`，如以下示例所示：

```xml
<bean class="example.SimpleMovieCatalog">
    <qualifier type="Offline"/> 
    <!-- inject any dependencies required by this bean -->
</bean>
```

|      | 该元素指定限定词。 |
| ---- | ------------------ |
|      |                    |

您还可以定义自定义限定符批注，该批注除了简单`value`属性之外或代替简单属性，还接受命名属性。如果随后在要自动装配的字段或参数上指定了多个属性值，则Bean定义必须与所有此类属性值匹配才能被视为自动装配候选。例如，请考虑以下注释定义：

爪哇

科特林

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();
}
```

在这种情况下`Format`是一个枚举，定义如下：

爪哇

科特林

```java
public enum Format {
    VHS, DVD, BLURAY
}
```

要自动装配的字段将用定制限定符进行注释，并包括这两个属性的值：`genre`和`format`，如以下示例所示：

爪哇

科特林

```java
public class MovieRecommender {

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Action")
    private MovieCatalog actionVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Comedy")
    private MovieCatalog comedyVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.DVD, genre="Action")
    private MovieCatalog actionDvdCatalog;

    @Autowired
    @MovieQualifier(format=Format.BLURAY, genre="Comedy")
    private MovieCatalog comedyBluRayCatalog;

    // ...
}
```

最后，bean定义应包含匹配的限定符值。此示例还演示了可以使用bean元属性代替 ``元素。如果可用，则``元素及其属性优先，但是，``如果不存在这样的限定符，则自动装配机制将退回到标签内提供的值 ，如以下示例中的最后两个bean定义：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Action"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Comedy"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="DVD"/>
        <meta key="genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="BLURAY"/>
        <meta key="genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

</beans>
```

#### 1.9.5。将泛型用作自动装配限定符

除了`@Qualifier`注释之外，您还可以将Java泛型类型用作资格的隐式形式。例如，假设您具有以下配置：

爪哇

科特林

```java
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
```

假设前面的bean实现了一个通用接口（即`Store`和 `Store`），则可以`@Autowire`将该`Store`接口和通用用作限定符，如以下示例所示：

爪哇

科特林

```java
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
```

当自动装配列表，`Map`实例和数组时，通用限定符也适用。下面的示例自动连接泛型`List`：

爪哇

科特林

```java
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```

#### 1.9.6。使用`CustomAutowireConfigurer`

[`CustomAutowireConfigurer`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/CustomAutowireConfigurer.html) 是一个`BeanFactoryPostProcessor`，即使您没有使用Spring的`@Qualifier`注释来注释自己的自定义限定符注释类型，也可以使用它。以下示例显示如何使用`CustomAutowireConfigurer`：

```xml
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```

通过以下方式`AutowireCandidateResolver`确定自动装配候选对象：

- `autowire-candidate`每个bean定义的值
- 元素`default-autowire-candidates`上可用的任何模式``
- `@Qualifier`注释的存在以及向NET注册的任何自定义注释`CustomAutowireConfigurer`

当多个bean符合自动装配候选条件时，确定“主要”的步骤如下：如果候选中恰好有一个bean定义将`primary` 属性设置为`true`，则将其选中。

#### 1.9.7。注射用`@Resource`

Spring还通过在字段或bean属性设置器方法上使用JSR-250 `@Resource`批注（`javax.annotation.Resource`）支持注入。这是Java EE中的一种常见模式：例如，在JSF管理的Bean和JAX-WS端点中。Spring也为Spring管理的对象支持此模式。

`@Resource`具有名称属性。默认情况下，Spring将该值解释为要注入的Bean名称。换句话说，它遵循名称语义，如以下示例所示：

爪哇

科特林

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder") 
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

|      | 这行注入`@Resource`。 |
| ---- | --------------------- |
|      |                       |

如果未明确指定名称，则默认名称是从字段名称或setter方法派生的。如果是字段，则采用字段名称。在使用setter方法的情况下，它采用bean属性名称。以下示例将名为bean `movieFinder`的setter方法注入：

爪哇

科特林

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

|      | 提供注解的名称解析由一个bean的名称 `ApplicationContext`，其中的`CommonAnnotationBeanPostProcessor`知道。如果您[`SimpleJndiBeanFactory`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/jndi/support/SimpleJndiBeanFactory.html) 显式配置Spring的名称，则可以通过JNDI解析名称 。但是，我们建议您依赖默认行为并使用Spring的JNDI查找功能来保留间接级别。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

在专属情况下，`@Resource`不指定明确的名称，以及类似的使用`@Autowired`，`@Resource`发现的主要类型匹配，而不是一个具体的bean并解决众所周知的解析依存关系：`BeanFactory`， `ApplicationContext`，`ResourceLoader`，`ApplicationEventPublisher`，和`MessageSource` 接口。

因此，在以下示例中，该`customerPreferenceDao`字段首先查找名为“ customerPreferenceDao”的bean，然后回退到该类型的主类型匹配项 `CustomerPreferenceDao`：

爪哇

科特林

```java
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context; 

    public MovieRecommender() {
    }

    // ...
}
```

|      | 该`context`字段是根据已知的可解析依赖类型注入的 `ApplicationContext`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.9.8。使用`@Value`

`@Value` 通常用于注入外部属性：

爪哇

科特林

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}
```

使用以下配置：

爪哇

科特林

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```

和以下`application.properties`文件：

```java
catalog.name=MovieCatalog
```

在这种情况下，`catalog`参数和字段将等于`MovieCatalog`值。

Spring提供了一个默认的宽松内嵌值解析器。它将尝试解析属性值，如果无法解析，`${catalog.name}`则将注入属性名称（例如）作为值。如果要严格控制不存在的值，则应声明一个`PropertySourcesPlaceholderConfigurer`bean，如以下示例所示：

爪哇

科特林

```java
@Configuration
public class AppConfig {

     @Bean
     public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
           return new PropertySourcesPlaceholderConfigurer();
     }
}
```

|      | 当配置`PropertySourcesPlaceholderConfigurer`使用JavaConfig，该 `@Bean`方法必须是`static`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果`${}` 无法解析任何占位符，则使用上述配置可确保Spring初始化失败。也可以使用`setPlaceholderPrefix`，，之类的方法 `setPlaceholderSuffix`或`setValueSeparator`自定义占位符。

|      | 默认情况下，Spring Boot配置一个`PropertySourcesPlaceholderConfigurer`将从`application.properties`和获取属性的bean `application.yml`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Spring提供的内置转换器支持允许自动处理简单的类型转换（例如转换为`Integer` 或`int`）。多个逗号分隔的值可以自动转换为String数组，而无需付出额外的努力。

可以提供如下默认值：

爪哇

科特林

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name:defaultCatalog}") String catalog) {
        this.catalog = catalog;
    }
}
```

Spring `BeanPostProcessor`使用`ConversionService`幕后处理将String值转换`@Value`为目标类型的过程。如果要为自己的自定义类型提供转换支持，则可以提供自己的 `ConversionService`bean实例，如以下示例所示：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean
    public ConversionService conversionService() {
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();
        conversionService.addConverter(new MyCustomConverter());
        return conversionService;
    }
}
```

当`@Value`包含[`SpEL`表达式时，](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions)该值将在运行时动态计算，如以下示例所示：

爪哇

科特林

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("#{systemProperties['user.catalog'] + 'Catalog' }") String catalog) {
        this.catalog = catalog;
    }
}
```

SpEL还支持使用更复杂的数据结构：

爪哇

科特林

```java
@Component
public class MovieRecommender {

    private final Map<String, Integer> countOfMoviesPerCatalog;

    public MovieRecommender(
            @Value("#{{'Thriller': 100, 'Comedy': 300}}") Map<String, Integer> countOfMoviesPerCatalog) {
        this.countOfMoviesPerCatalog = countOfMoviesPerCatalog;
    }
}
```

#### 1.9.9。使用`@PostConstruct`和`@PreDestroy`

将`CommonAnnotationBeanPostProcessor`不仅承认了`@Resource`注解也是JSR-250的生命周期注解：`javax.annotation.PostConstruct`和 `javax.annotation.PreDestroy`。在Spring 2.5中引入了对这些注释的支持，为[初始化回调](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)和 [销毁回调中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean)描述的生命周期回调机制提供了一种替代 [方法](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean)。假设 `CommonAnnotationBeanPostProcessor`在Spring内注册`ApplicationContext`，则在生命周期的同一点将调用带有这些注释之一的方法作为相应的Spring生命周期接口方法或显式声明的回调方法。在以下示例中，缓存在初始化时预先填充，并在销毁时清除：

爪哇

科特林

```java
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```

有关组合各种生命周期机制的效果的详细信息，请参见 [组合生命周期机制](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-combined-effects)。

|      | 像和一样`@Resource`，`@PostConstruct`和`@PreDestroy`注释类型是JDK 6到8的标准Java库的一部分。但是，整个`javax.annotation` 包都与JDK 9中的核心Java模块分开，并最终在JDK 11中删除了。如果需要，需要对`javax.annotation-api`工件进行处理。现在可以通过Maven Central获得，只需像其他任何库一样将其添加到应用程序的类路径中即可。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.10。类路径扫描和托管组件

本章中的大多数示例都使用XML来指定`BeanDefinition`在Spring容器中生成每个配置的配置元数据。上一节（[基于注释的容器配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-annotation-config)）演示了如何通过源级注释提供大量配置元数据。但是，即使在这些示例中，“基本” bean定义也已在XML文件中明确定义，而注释仅驱动依赖项注入。本节介绍了通过扫描类路径来隐式检测候选组件的选项。候选组件是与过滤条件匹配的类，并具有在容器中注册的相应bean定义。这消除了使用XML进行bean注册的需要。相反，您可以使用注释（例如`@Component`），AspectJ类型表达式或您自己的自定义过滤条件来选择哪些类已向容器注册了bean定义。

|      | 从Spring 3.0开始，Spring JavaConfig项目提供的许多功能是核心Spring Framework的一部分。这使您可以使用Java而不是使用传统的XML文件来定义bean。看看的`@Configuration`，`@Bean`， `@Import`，和`@DependsOn`注释有关如何使用这些新功能的例子。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.10.1。`@Component`和更多的刻板印象注释

的`@Repository`注释是针对满足的存储库（也被称为数据访问对象或DAO）的作用或者固定型的任何类的标记。如[Exception Translation中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#orm-exception-translation)所述，此标记的用途是自动翻译 [异常](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#orm-exception-translation)。

Spring提供进一步典型化注解：`@Component`，`@Service`，和 `@Controller`。`@Component`是任何Spring托管组件的通用构造型。 `@Repository`，`@Service`和`@Controller`是`@Component`针对更特定用例的专业化（分别在持久性，服务和表示层）。因此，您可以来注解你的组件类有 `@Component`，但是，通过与注解它们`@Repository`，`@Service`或者`@Controller` ，你的类能更好地适合于通过工具处理，或与切面进行关联。例如，这些构造型注释成为切入点的理想目标。`@Repository`，`@Service`和，并且`@Controller`在Spring框架的将来版本中还可以包含其他语义。因此，如果您选择使用`@Component`或`@Service`对于您的服务层，`@Service`显然是更好的选择。同样，如前所述，`@Repository`在持久层中已经支持作为自动异常转换的标记。

#### 1.10.2。使用元注释和组合注释

Spring提供的许多注释都可以在您自己的代码中用作元注释。元注释是可以应用于另一个注释的注释。例如，`@Service`注释提及[早期](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-stereotype-annotations) 为间注释有`@Component`，如下面的示例所示：

爪哇

科特林

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component 
public @interface Service {

    // ...
}
```

|      | 将`Component`导致`@Service`以同样的方式来对待`@Component`。 |
| ---- | ----------------------------------------------------------- |
|      |                                                             |

您还可以结合使用元注释来创建“组合注释”。例如，`@RestController`Spring MVC中的注释由`@Controller`和 组成`@ResponseBody`。

此外，组合注释可以选择从元注释中重新声明属性，以允许自定义。当您只希望公开元注释属性的子集时，这特别有用。例如，Spring的 `@SessionScope`注释将作用域名称硬编码为，`session`但仍允许自定义`proxyMode`。以下清单显示了`SessionScope`注释的定义 ：

爪哇

科特林

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {

    /**
     * Alias for {@link Scope#proxyMode}.
     * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
     */
    @AliasFor(annotation = Scope.class)
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;

}
```

然后，您`@SessionScope`无需声明`proxyMode`以下即可使用：

爪哇

科特林

```java
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```

您还可以覆盖的值`proxyMode`，如以下示例所示：

爪哇

科特林

```java
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
    // ...
}
```

有关更多详细信息，请参见 [Spring Annotation编程模型](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model) Wiki页面。

#### 1.10.3。自动检测类并注册Bean定义

Spring可以自动检测构造型类，并使用来注册相应的 `BeanDefinition`实例`ApplicationContext`。例如，以下两个类别可进行这种自动检测：

爪哇

科特林

```java
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

爪哇

科特林

```java
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```

要自动检测这些类并注册相应的bean，您需要添加 `@ComponentScan`到`@Configuration`类中，其中`basePackages`属性是两个类的公共父包。（或者，您可以指定用逗号，分号或空格分隔的列表，其中包括每个类的父包。）

爪哇

科特林

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

|      | 为简洁起见，前面的示例可能使用`value`了注释的属性（即`@ComponentScan("org.example")`）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下替代方法使用XML：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```

|      | 使用``隐式启用的功能 ``。``使用时，通常无需包含 元素``。 |
| ---- | -------------------------------------------------------- |
|      |                                                          |

|      | 扫描类路径包需要在类路径中存在相应的目录条目。使用Ant构建JAR时，请确保不要激活JAR任务的仅文件开关。此外，在某些环境中，可能不会基于安全策略公开类路径目录，例如，在JDK 1.7.0_45及更高版本上的独立应用程序（这需要在清单中设置“受信任的库”，请参见 [https://stackoverflow.com/ Questions / 19394570 / java-jre-7u45-breaks-classloader-getresources](https://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources)）。在JDK 9的模块路径（Jigsaw）上，Spring的类路径扫描通常可以按预期进行。但是，请确保在您的`module-info` 描述符中导出了组件类。如果您期望Spring调用您的类的非公共成员，请确保它们是“打开的”（也就是说，它们使用`opens`声明而不是描述符中的 `exports`声明`module-info`）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

此外，当您使用component-scan元素时，`AutowiredAnnotationBeanPostProcessor`和 `CommonAnnotationBeanPostProcessor`都隐式包括在内。这意味着将自动检测这两个组件并将它们连接在一起，而这一切都不需要XML中提供的任何bean配置元数据。

|      | 您可以禁用注册`AutowiredAnnotationBeanPostProcessor`并 `CommonAnnotationBeanPostProcessor`通过包括`annotation-config`与属性的值`false`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.10.4。使用过滤器自定义扫描

默认情况下，类注有`@Component`，`@Repository`，`@Service`，`@Controller`， `@Configuration`，或自定义的注释，与自身的注解`@Component`是唯一检测到的候选组件。但是，您可以通过应用自定义过滤器来修改和扩展此行为。将它们添加为`includeFilters`或注释的`excludeFilters`属性`@ComponentScan`（或XML配置中元素的``或 ``子元素``）。每个过滤器元素都需要`type`和`expression`属性。下表描述了过滤选项：

| 过滤器类型   | 范例表达                     | 描述                                                         |
| :----------- | :--------------------------- | :----------------------------------------------------------- |
| 注释（默认） | `org.example.SomeAnnotation` | 在目标组件中的类型级别上*存在*或*元存在*的注释。             |
| 可分配的     | `org.example.SomeClass`      | 目标组件可分配给（扩展或实现）的类（或接口）。               |
| 方面         | `org.example..*Service+`     | 目标组件要匹配的AspectJ类型表达式。                          |
| 正则表达式   | `org\.example\.Default.*`    | 要与目标组件的类名匹配的正则表达式。                         |
| 习俗         | `org.example.MyTypeFilter`   | `org.springframework.core.type.TypeFilter`接口的自定义实现。 |

以下示例显示了忽略所有`@Repository`注释并改为使用“存根”存储库的配置：

爪哇

科特林

```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```

以下清单显示了等效的XML：

```xml
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```

|      | 您也可以通过设置`useDefaultFilters=false`注释或通过提供元素`use-default-filters="false"`的属性来 禁用默认过滤器``。这样有效地禁止的注释的或与间注解的类自动检测`@Component`，`@Repository`，`@Service`，`@Controller`， `@RestController`，或`@Configuration`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.10.5。在组件中定义Bean元数据

Spring组件还可以将bean定义元数据贡献给容器。您可以`@Bean`使用与在带`@Configuration` 注释的类中定义Bean元数据相同的注释来执行此操作。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```

上一类是Spring组件，其`doWork()`方法中包含特定于应用程序的代码 。但是，它也提供了具有工厂方法的bean定义，该工厂方法引用了method `publicInstance()`。该`@Bean`注释标识工厂方法和其它bean定义特性，如通过一个限定值`@Qualifier`注释。可以指定其他方法级别的注解是 `@Scope`，`@Lazy`和自定义限定器注解。

|      | 除了用于组件初始化的角色外，您还可以将`@Lazy`注释放置在标有`@Autowired`或的注入点上`@Inject`。在这种情况下，它导致注入了惰性解析代理。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如前所述，支持自动连线的字段和方法，并自动支持`@Bean`方法的附加支持。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters
    @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @RequestScope
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }
}
```

该示例将`String`方法参数自动连接`country`到`age` 另一个名为的bean上的属性值`privateInstance`。Spring Expression Language元素通过符号定义属性的值`#{  }`。对于`@Value` 注释，表达式解析器已预先配置为在解析表达式文本时查找bean名称。

从Spring Framework 4.3开始，您还可以声明类型`InjectionPoint`（或其更具体的子类：）的工厂方法参数， `DependencyDescriptor`以访问触发当前bean创建的请求注入点。注意，这仅适用于实际创建bean实例，而不适用于注入现有实例。因此，此功能对原型范围的bean最有意义。对于其他作用域，factory方法仅在给定作用域中看到触发创建新bean实例的注入点（例如，触发创建惰性单例bean的依赖项）。在这种情况下，可以将提供的注入点元数据与语义一起使用。以下示例显示如何使用`InjectionPoint`：

爪哇

科特林

```java
@Component
public class FactoryMethodComponent {

    @Bean @Scope("prototype")
    public TestBean prototypeInstance(InjectionPoint injectionPoint) {
        return new TestBean("prototypeInstance for " + injectionPoint.getMember());
    }
}
```

将`@Bean`在普通的Spring组件方法比春天里的同行处理方式不同`@Configuration`类。不同之处在于`@Component` CGLIB并未增强类来拦截方法和字段的调用。CGLIB代理是一种方法，通过该方法可以调用类中的方法或`@Bean`方法中的字段来`@Configuration`创建Bean元数据引用以协作对象。此类方法不是用普通的Java语义调用的，而是通过容器进行的，以提供Spring Bean的常规生命周期管理和代理，即使通过程序调用`@Bean`方法引用其他Bean时也是如此。相反，`@Bean`在普通区域内调用方法或方法中的字段`@Component` 类具有标准的Java语义，没有特殊的CGLIB处理或其他限制。

|      | 您可以将`@Bean`方法声明为`static`，从而允许在不将其包含配置类创建为实例的情况下调用它们。在定义后处理器Bean（例如类型`BeanFactoryPostProcessor` 或`BeanPostProcessor`）时，这特别有意义，因为此类Bean在容器生命周期的早期进行了初始化，并且应避免在此时触发配置的其他部分。由于技术限制，对静态`@Bean`方法的调用永远不会被容器拦截，即使在`@Configuration`类内也不会 （如本节前面所述），因为技术限制：CGLIB子类只能覆盖非静态方法。结果，直接调用另一个`@Bean`方法具有标准的Java语义，从而导致直接从工厂方法本身返回一个独立的实例。方法的Java语言可见性`@Bean`不会对Spring容器中的结果bean定义产生直接影响。您可以随意声明自己的工厂方法（如果您认为适合非`@Configuration`类），也可以在任何地方声明静态方法。但是，类中的常规`@Bean`方法`@Configuration`必须是可重写的-也就是说，不得将其声明为`private`或`final`。`@Bean`还可以在给定组件或配置类的基类上以及在由组件或配置类实现的接口中声明的Java 8默认方法上发现方法。这为组合复杂的配置安排提供了很大的灵活性，从Spring 4.2开始，通过Java 8默认方法甚至可以进行多重继承。最后，单个类可以`@Bean`为同一个bean 保留多个方法，这取决于在运行时可用的依赖项，以安排使用多个工厂方法。这与在其他配置方案中选择“最贪婪”的构造函数或工厂方法的算法相同：在构造时选择具有最大可满足依赖关系数量的变量，类似于容器在多个`@Autowired`构造函数之间进行选择的方式。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.10.6。命名自动检测的组件

当组件被自动检测为扫描过程的一部分时，其bean名称由该`BeanNameGenerator`扫描器已知的策略生成。默认情况下，任何Spring刻板印象注释（`@Component`，`@Repository`，`@Service`和 `@Controller`），其中包含一个名称`value`，从而提供了名称，相应的bean定义。

如果这样的注释不包含名称，`value`或者不包含任何其他检测到的组件（例如，由自定义过滤器发现的组件），则缺省bean名称生成器将返回不使用大写字母的非限定类名称。例如，如果检测到以下组件类，则名称为`myMovieLister`和`movieFinderImpl`：

爪哇

科特林

```java
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```

爪哇

科特林

```java
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

如果不想依赖默认的Bean命名策略，则可以提供自定义Bean命名策略。首先，实现 [`BeanNameGenerator`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/support/BeanNameGenerator.html) 接口，并确保包括默认的无参数构造函数。然后，在配置扫描程序时提供完全限定的类名，如以下示例注释和Bean定义所示。

|      | 如果由于多个自动检测到的组件具有相同的非限定类名而遇到命名冲突（即，具有相同名称但位于不同包中的类），则可能需要配置一个`BeanNameGenerator`默认生成的完全限定类名豆名称。从Spring Framework 5.2.3开始， `FullyQualifiedAnnotationBeanNameGenerator`位于package中的包 `org.springframework.context.annotation`可用于此类目的。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

爪哇

科特林

```java
@Configuration
@ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
public class AppConfig {
    // ...
}
```

```xml
<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>
```

作为一般规则，每当其他组件可能对其进行显式引用时，请考虑使用注释指定名称。另一方面，只要容器负责接线，自动生成的名称就足够了。

#### 1.10.7。提供自动检测组件的范围

一般而言，与Spring管理的组件一样，自动检测到的组件的默认范围也是最常见的范围是`singleton`。但是，有时您需要`@Scope`注释可以指定的其他范围。您可以在批注中提供范围的名称，如以下示例所示：

爪哇

科特林

```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

|      | `@Scope`注释仅在具体的bean类（对于带注释的组件）或工厂方法（对于`@Bean`方法）上进行内省。与XML bean定义相反，没有bean定义继承的概念，并且在类级别的继承层次结构与元数据目的无关。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

有关特定于Web的范围的详细信息，例如Spring上下文中的“ request”或“ session”，请参阅[Request，Session，Application和WebSocket Scope](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-other)。与这些作用域的预构建批注一样，您也可以使用Spring的元注释方法来编写自己的作用域注释：例如，用`@Scope("prototype")`，注释自定义注释的自定义注释，也可能会声明自定义作用域代理模式。

|      | 要提供用于范围解析的自定义策略，而不是依赖于基于注释的方法，可以实现该 [`ScopeMetadataResolver`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/ScopeMetadataResolver.html) 接口。确保包括默认的无参数构造函数。然后，可以在配置扫描器时提供完全限定的类名，如以下注释和Bean定义示例所示： |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

爪哇

科特林

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class)
public class AppConfig {
    // ...
}
```

```xml
<beans>
    <context:component-scan base-package="org.example" scope-resolver="org.example.MyScopeResolver"/>
</beans>
```

使用某些非单作用域时，可能有必要为作用域对象生成代理。在[范围Bean中将](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-other-injection)推理描述[为依赖项](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-other-injection)。为此，在component-scan元素上可以使用scoped-proxy属性。三个可能的值是：`no`，`interfaces`，和`targetClass`。例如，以下配置生成标准的JDK动态代理：

爪哇

科特林

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
    // ...
}
```

```xml
<beans>
    <context:component-scan base-package="org.example" scoped-proxy="interfaces"/>
</beans>
```

#### 1.10.8。提供带有注释的限定符元数据

在`@Qualifier`注释中讨论[，基于注解微调自动装配与预选赛](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-autowired-annotation-qualifiers)。该部分中的示例演示了如何使用`@Qualifier`注释和自定义限定符注释在解析自动装配候选时提供细粒度的控制。由于这些示例基于XML Bean定义，因此通过使用XML 中的元素的`qualifier`或`meta`子元素，在候选Bean定义上提供了限定符元数据`bean`。当依靠类路径扫描来自动检测组件时，可以在候选类上为限定符元数据提供类型级别的注释。下面的三个示例演示了此技术：

爪哇

科特林

```java
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

爪哇

科特林

```java
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

爪哇

科特林

```java
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```

|      | 与大多数基于注释的替代方法一样，请记住，注释元数据绑定到类定义本身，而XML的使用允许相同类型的多个bean提供其限定符元数据的变体，因为该元数据是按-instance而不是按类。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.10.9。生成候选组件的索引

尽管类路径扫描非常快，但是可以通过在编译时创建候选静态列表来提高大型应用程序的启动性能。在这种模式下，作为组件扫描目标的所有模块都必须使用此机制。

|      | 您现有的`@ComponentScan`或`指令必须保持原样，以请求上下文扫描某些软件包中的候选对象。当 `ApplicationContext`检测到这样的索引时，它将自动使用它而不是扫描类路径。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要生成索引，请向每个包含组件的模块添加附加依赖关系，这些组件是组件扫描指令的目标。以下示例显示了如何使用Maven进行操作：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.2.6.RELEASE</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```

对于Gradle 4.5及更早版本，应在`compileOnly` 配置中声明依赖项，如以下示例所示：

```groovy
dependencies {
    compileOnly "org.springframework:spring-context-indexer:5.2.6.RELEASE"
}
```

对于Gradle 4.6和更高版本，应在`annotationProcessor` 配置中声明依赖项，如以下示例所示：

```groovy
dependencies {
    annotationProcessor "org.springframework:spring-context-indexer:{spring-version}"
}
```

该过程将生成一个`META-INF/spring.components`包含在jar文件中的文件。

|      | 在IDE中使用此模式时，`spring-context-indexer`必须将其注册为注释处理器，以确保在更新候选组件时索引是最新的。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 当`META-INF/spring.components`在类路径上找到a时，索引将自动启用。如果某个索引对于某些库（或用例）部分可用，但无法为整个应用程序构建，则可以通过将设置`spring.index.ignore`为 `true`，来回退到常规的类路径安排（好像根本没有索引）属性或`spring.properties`类路径根目录下的文件中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.11。使用JSR 330标准注释

从Spring 3.0开始，Spring提供对JSR-330标准注释（依赖注入）的支持。这些注释的扫描方式与Spring注释的扫描方式相同。要使用它们，您需要在类路径中有相关的jar。

|      | 如果您使用Maven，`javax.inject`则可以在标准Maven存储库（ https://repo1.maven.org/maven2/javax/inject/javax.inject/1/）中。您可以将以下依赖项添加到文件pom.xml中：`    javax.inject    javax.inject    1 ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.11.1。与`@Inject`和的依赖注入`@Named`

除了`@Autowired`，您可以使用`@javax.inject.Inject`以下方法：

爪哇

科特林

```java
import javax.inject.Inject;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.findMovies(...);
        // ...
    }
}
```

与一样`@Autowired`，您可以`@Inject`在字段级别，方法级别和构造函数参数级别使用。此外，您可以将注入点声明为 `Provider`，以允许按需访问范围更短的bean，或者通过`Provider.get()`调用延迟访问其他bean 。以下示例提供了前面示例的变体：

爪哇

科特林

```java
import javax.inject.Inject;
import javax.inject.Provider;

public class SimpleMovieLister {

    private Provider<MovieFinder> movieFinder;

    @Inject
    public void setMovieFinder(Provider<MovieFinder> movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.get().findMovies(...);
        // ...
    }
}
```

如果要为应注入的依赖项使用限定名称，则应使用`@Named`批注，如以下示例所示：

爪哇

科特林

```java
import javax.inject.Inject;
import javax.inject.Named;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

与一样`@Autowired`，`@Inject`也可以与`java.util.Optional`或 一起使用`@Nullable`。这在这里更为适用，因为`@Inject`它没有`required`属性。以下示例展示了如何使用`@Inject`和 `@Nullable`：

```java
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        // ...
    }
}
```

爪哇

科特林

```java
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        // ...
    }
}
```

#### 1.11.2。`@Named`和`@ManagedBean`：`@Component`注释的标准等效项

代替`@Component`，您可以使用`@javax.inject.Named`或`javax.annotation.ManagedBean`，如以下示例所示：

爪哇

科特林

```java
import javax.inject.Inject;
import javax.inject.Named;

@Named("movieListener")  // @ManagedBean("movieListener") could be used as well
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

在`@Component`不指定组件名称的情况下使用非常常见。 `@Named`可以类似的方式使用，如以下示例所示：

爪哇

科特林

```java
import javax.inject.Inject;
import javax.inject.Named;

@Named
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

使用`@Named`或时`@ManagedBean`，可以使用与使用Spring注释完全相同的方式来使用组件扫描，如以下示例所示：

爪哇

科特林

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

|      | 与相比`@Component`，JSR-330 `@Named`和JSR-250 `ManagedBean` 注释是不可组合的。您应该使用Spring的构造型模型来构建自定义组件注释。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.11.3。JSR-330标准注释的局限性

当使用标准注释时，您应该知道某些重要功能不可用，如下表所示：

| 弹簧                   | javax.inject。*       | javax.inject限制/注释                                        |
| :--------------------- | :-------------------- | :----------------------------------------------------------- |
| @Autowired             | @注入                 | `@Inject`没有“必填”属性。可以与Java 8一起使用`Optional`。    |
| @零件                  | @Named / @ManagedBean | JSR-330不提供可组合的模型，仅提供一种识别命名组件的方法。    |
| @Scope（“ singleton”） | @辛格尔顿             | JSR-330的默认范围类似于Spring的`prototype`。但是，为了使其与Spring的默认默认值保持一致，默认情况下，在Spring容器中声明的JSR-330 bean是a `singleton`。为了使用之外的范围`singleton`，您应该使用Spring的`@Scope`注释。`javax.inject`还提供了 [@Scope](https://download.oracle.com/javaee/6/api/javax/inject/Scope.html)批注。但是，此仅用于创建自己的注释。 |
| @Qualifier             | @Qualifier / @命名    | `javax.inject.Qualifier`只是用于构建自定义限定符的元注释。具体的`String`限定词（如`@Qualifier`带有值的Spring的限定词）可以通过关联`javax.inject.Named`。 |
| @值                    | --                    | 没有等效                                                     |
| @需要                  | --                    | 没有等效                                                     |
| @懒                    | --                    | 没有等效                                                     |
| 对象工厂               | 提供者                | `javax.inject.Provider`是Spring的直接替代方法`ObjectFactory`，只是`get()`方法名称较短。它也可以与Spring `@Autowired`或非注释构造函数和setter方法结合使用。 |

### 1.12。基于Java的容器配置

本节介绍如何在Java代码中使用注释来配置Spring容器。它包括以下主题：

- [基本概念：`@Bean`和`@Configuration`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-basic-concepts)
- [使用实例化Spring容器 `AnnotationConfigApplicationContext`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-instantiating-container)
- [使用`@Bean`注释](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-bean-annotation)
- [使用`@Configuration`注释](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-configuration-annotation)
- [组成基于Java的配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-composing-configuration-classes)
- [Bean定义配置文件](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-definition-profiles)
- [`PropertySource` 抽象化](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-property-source-abstraction)
- [使用 `@PropertySource`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-using-propertysource)
- [声明中的占位符解析](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-placeholder-resolution-in-statements)

#### 1.12.1。基本概念：`@Bean`和`@Configuration`

Spring的新Java配置支持中的主要工件是-带 `@Configuration`注释的类和-带`@Bean`注释的方法。

该`@Bean`注释被用于指示一个方法实例，可以配置，并初始化到由Spring IoC容器进行管理的新对象。对于那些熟悉Spring的``XML配置的人来说，`@Bean`注释的作用与``元素相同。您可以`@Bean`对任何Spring 使用带注释的方法 `@Component`。但是，它们最常与`@Configuration`bean一起使用。

用注释类`@Configuration`表示其主要目的是作为Bean定义的来源。此外，`@Configuration`类允许通过调用`@Bean`同一类中的其他方法来定义Bean之间的依赖关系。最简单的`@Configuration`类如下：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

上一`AppConfig`类等效于以下Spring ``XML：

```xml
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

完整的@Configuration与“精简” @Bean模式？

如果`@Bean`在未使用注释的类中声明方法，则 `@Configuration`它们被称为以“精简”模式进行处理。在一个`@Component`或什至在一个普通的旧类中声明的Bean方法被认为是“精简版”，其包含类的主要目的不同，而`@Bean`方法在那儿是一种奖励。例如，服务组件可以通过`@Bean`每个适用组件类上的其他方法向容器公开管理视图。在这种情况下，`@Bean`方法是通用的工厂方法机制。

与full不同`@Configuration`，lite `@Bean`方法无法声明Bean间的依赖关系。相反，它们在其包含组件的内部状态上进行操作，并且还可以根据可能声明的自变量进行操作。`@Bean`因此，此类方法不应调用其他 `@Bean`方法。每个此类方法实际上只是针对特定bean引用的工厂方法，而没有任何特殊的运行时语义。这里的积极副作用是，不必在运行时应用CGLIB子类，因此在类设计方面没有任何限制（也就是说，包含类可以是`final`依此类推）。

在常见情况下，`@Bean`方法将在`@Configuration`类中声明，以确保始终使用“完全”模式，并且因此将跨方法引用重定向到容器的生命周期管理。这样可以防止`@Bean`通过常规Java调用意外地调用同一 方法，从而有助于减少在“精简”模式下运行时难以追查的细微错误。

的`@Bean`和`@Configuration`注解的深度在以下章节中讨论。但是，首先，我们介绍了通过基于Java的配置使用创建spring容器的各种方法。

#### 1.12.2。使用实例化Spring容器`AnnotationConfigApplicationContext`

以下各节`AnnotationConfigApplicationContext`介绍了Spring 3.0中引入的Spring。这种通用的`ApplicationContext`实现方式不仅可以接受`@Configuration`类作为输入，还可以接受 普通`@Component`类和使用JSR-330元数据注释的类。

当`@Configuration`提供类作为输入时，`@Configuration`该类本身将注册为Bean定义，并且`@Bean`该类中所有已声明的方法也将注册为Bean定义。

当提供`@Component`和JSR-330类时，它们被注册为bean定义，并且假定在必要时在这些类中使用DI元数据，例如`@Autowired`或`@Inject`。

##### 施工简单

与实例化a时将Spring XML文件用作输入的方式几乎相同，实例化a时 `ClassPathXmlApplicationContext`可以将`@Configuration`类用作输入`AnnotationConfigApplicationContext`。如下面的示例所示，这允许Spring容器的使用完全不依赖XML：

爪哇

科特林

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

如前所述，`AnnotationConfigApplicationContext`不仅限于仅使用`@Configuration`类。`@Component`可以将任何或带有JSR-330注释的类作为输入提供给构造函数，如以下示例所示：

爪哇

科特林

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

前面的例子中假定`MyServiceImpl`，`Dependency1`以及`Dependency2`使用Spring依赖注入注解，例如`@Autowired`。

##### 通过使用编程方式构建容器 `register(Class…)`

您可以`AnnotationConfigApplicationContext`使用no-arg构造函数实例化一个，然后使用`register()`方法进行配置。以编程方式构建.NET时，此方法特别有用`AnnotationConfigApplicationContext`。以下示例显示了如何执行此操作：

爪哇

科特林

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

##### 使用启用组件扫描 `scan(String…)`

要启用组件扫描，您可以`@Configuration`按如下方式注释您的类：

爪哇

科特林

```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    ...
}
```

|      | 此注释启用组件扫描。 |
| ---- | -------------------- |
|      |                      |

|      | 有经验的Spring用户可能熟悉Spring `context:`命名空间中的XML声明，如以下示例所示：`     ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

在前面的示例中，将`com.acme`扫描软件包以查找任何带 `@Component`注释的类，并将这些类注册为容器内的Spring bean定义。`AnnotationConfigApplicationContext`公开此 `scan(String…)`方法以允许相同的组件扫描功能，如以下示例所示：

爪哇

科特林

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

|      | 请记住，`@Configuration`类是[元注释](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-meta-annotations) 用`@Component`，所以他们是组件扫描候选人。在前面的示例中，假设`AppConfig`在`com.acme`包（或下面的任何包）中声明，则在调用期间将其拾取`scan()`。基于`refresh()`，其所有`@Bean` 方法都将在容器内处理并注册为Bean定义。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 支持Web应用程序 `AnnotationConfigWebApplicationContext`

一个`WebApplicationContext`变种`AnnotationConfigApplicationContext`是可用的`AnnotationConfigWebApplicationContext`。在配置Spring `ContextLoaderListener`servlet侦听器，Spring MVC等时 `DispatcherServlet`，可以使用此实现。以下`web.xml`代码片段配置了典型的Spring MVC Web应用程序（请注意使用`contextClass`context-param和init-param）：

```xml
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

#### 1.12.3。使用`@Bean`注释

`@Bean`是方法级别的注释，是XML ``元素的直接模拟。注释支持提供的某些属性``，例如：* [init-method](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) * [destroy-method](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean) * [autowiring](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-autowire) * `name`。

你可以使用`@Bean`一个注释`@Configuration`-annotated或在 `@Component`-annotated类。

##### 声明一个豆

要声明一个bean，可以用注解对方法进行`@Bean`注解。您可以使用此方法在`ApplicationContext`指定为该方法的返回值的类型内注册Bean定义。缺省情况下，bean名称与方法名称相同。以下示例显示了`@Bean`方法声明：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

前面的配置与下面的Spring XML完全等效：

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

双方的声明作出的bean `transferService`中可用 `ApplicationContext`，势必类的对象实例`TransferServiceImpl`，如下面的文字图像显示：

```
transferService-> com.acme.TransferServiceImpl
```

您还可以`@Bean`使用接口（或基类）返回类型声明您的方法，如以下示例所示：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

但是，这将高级类型预测的可见性限制为指定的接口类型（`TransferService`）。然后，使用`TransferServiceImpl`仅一次容器已知的完整类型（），实例化受影响的单例bean。非惰性单例bean根据其声明顺序实例化，因此您可能会看到不同的类型匹配结果，具体取决于另一个组件何时尝试通过未声明的类型进行匹配（例如`@Autowired TransferServiceImpl`，，仅`transferService`在实例化bean之后才解析）。

|      | 如果您通过声明的服务接口一致地引用类型，则 `@Bean`返回类型可以安全地加入该设计决策。但是，对于实现多个接口的组件或由其实现类型潜在引用的组件，声明可能的最具体的返回类型（至少与引用您的bean的注入点所要求的具体类型一样）更为安全。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### Bean依赖

带`@Bean`注释的方法可以具有任意数量的参数，这些参数描述构建该bean所需的依赖关系。例如，如果我们`TransferService` 需要一个`AccountRepository`，我们可以使用方法参数来实现该依赖关系，如以下示例所示：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

解析机制与基于构造函数的依赖注入几乎相同。[有关](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-constructor-injection)更多详细信息，请参见[相关部分](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-constructor-injection)。

##### 接收生命周期回调

用`@Bean`注释定义的任何类都支持常规的生命周期回调，并且可以使用JSR-250中的`@PostConstruct`和`@PreDestroy`注释。有关更多详细信息，请参见 [JSR-250注释](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations)。

常规Spring [生命周期](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-nature)回调也得到完全支持。如果bean实现`InitializingBean`，`DisposableBean`或`Lifecycle`，则容器将调用它们各自的方法。

也完全支持标准`*Aware`接口集（例如[BeanFactoryAware](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-beanfactory)， [BeanNameAware](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-aware)， [MessageSourceAware](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#context-functionality-messagesource)， [ApplicationContextAware](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-aware)等）。

该`@Bean`注释支持指定任意初始化和销毁回调方法，就像Spring XML中的`init-method`和`destroy-method`属性的`bean`元素，如下面的示例所示：

爪哇

科特林

```java
public class BeanOne {

    public void init() {
        // initialization logic
    }
}

public class BeanTwo {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

|      | 默认情况下，用Java配置定义的具有公共`close`或`shutdown` 方法的bean 会自动通过销毁回调进行登记。如果您有公共 `close`或`shutdown`方法，并且不希望在容器关闭时调用它，则可以添加`@Bean(destroyMethod="")`到bean定义中以禁用默认`(inferred)`模式。默认情况下，您可能要对通过JNDI获取的资源执行此操作，因为其生命周期是在应用程序外部进行管理的。特别是，请确保始终对进行操作`DataSource`，因为这在Java EE应用程序服务器上是有问题的。以下示例显示了如何防止对的自动销毁回调 `DataSource`：爪哇科特林`@Bean(destroyMethod="") public DataSource dataSource() throws NamingException {    return (DataSource) jndiTemplate.lookup("MyDS"); }`此外，对于`@Bean`方法，通常使用程序化JNDI查找，方法是使用Spring `JndiTemplate`或`JndiLocatorDelegate`辅助方法，或者直接`InitialContext`使用JNDI 用法，但不使用`JndiObjectFactoryBean`变体（这会迫使您将返回类型声明为`FactoryBean`类型，而不是实际的目标类型，这使得它很难实现。在`@Bean`打算引用此处提供的资源的其他方法中用于交叉引用调用）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

对于`BeanOne`前面注释中的示例，`init()` 在构造期间直接调用该方法同样有效，如以下示例所示：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        BeanOne beanOne = new BeanOne();
        beanOne.init();
        return beanOne;
    }

    // ...
}
```

|      | 当您直接使用Java工作时，您可以对对象执行任何操作，而不必总是依赖于容器的生命周期。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 指定Bean范围

Spring包含`@Scope`注释，以便您可以指定bean的范围。

###### 使用`@Scope`注释

您可以指定使用`@Bean`批注定义的bean 应该具有特定范围。您可以使用[Bean Scopes](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes)部分中指定的任何标准范围 。

默认范围是`singleton`，但是您可以使用`@Scope`注释覆盖它，如以下示例所示：

爪哇

科特林

```java
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```

###### `@Scope` 和 `scoped-proxy`

Spring提供了一种通过[作用域代理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-other-injection)处理作用域依赖关系的便捷方法 。使用XML配置时创建此类代理的最简单方法是``元素。使用`@Scope`注释在Java中配置bean 可以为该`proxyMode`属性提供同等的支持。默认值为no proxy（`ScopedProxyMode.NO`），但您可以指定`ScopedProxyMode.TARGET_CLASS`或`ScopedProxyMode.INTERFACES`。

如果将XML参考文档（请参阅[作用域代理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-other-injection)）中的作用域代理示例移植到我们`@Bean`使用Java的示例中 ，则它类似于以下内容：

爪哇

科特林

```java
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```

##### 自定义Bean命名

默认情况下，配置类使用`@Bean`方法的名称作为结果bean的名称。但是，可以使用`name`属性覆盖此功能，如以下示例所示：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean(name = "myThing")
    public Thing thing() {
        return new Thing();
    }
}
```

##### Bean别名

如[Naming Beans中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-beanname)所讨论的，有时希望为单个Bean提供多个名称，否则称为Bean别名。 为此`name`，`@Bean`注释的属性接受String数组。以下示例显示了如何为bean设置多个别名：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

##### 豆描述

有时，提供有关bean的更详细的文本描述会很有帮助。当出于监视目的而暴露（可能通过JMX）bean时，这特别有用。

要将说明添加到`@Bean`，可以使用 [`@Description`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/Description.html) 批注，如以下示例所示：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Thing thing() {
        return new Thing();
    }
}
```

#### 1.12.4。使用`@Configuration`注释

`@Configuration`是类级别的注释，指示对象是Bean定义的源。`@Configuration`类通过公共`@Bean`注释方法声明bean 。`@Bean`对`@Configuration`类的方法的调用也可以用于定义Bean之间的依赖关系。请参阅[基本概念：`@Bean`以及`@Configuration`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-basic-concepts)一般性介绍。

##### 注入bean间的依赖关系

当bean彼此依赖时，表达这种依赖就像让一个bean方法调用另一个依赖一样简单，如以下示例所示：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

在前面的示例中，通过构造函数注入`beanOne`接收对的引用`beanTwo`。

|      | 仅当`@Bean`在`@Configuration`类中声明该方法时，此声明bean间依赖关系的方法才有效。您不能使用普通`@Component`类声明bean间的依赖关系。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 查找方法注入

如前所述，[查找方法注入](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-method-injection)是一项高级功能，您应该很少使用。在单例作用域的bean依赖于原型作用域的bean的情况下，这很有用。将Java用于这种类型的配置为实现这种模式提供了自然的方法。以下示例显示如何使用查找方法注入：

爪哇

科特林

```java
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

通过使用Java配置，可以创建一个覆盖`CommandManager`抽象`createCommand()`方法的子类，该方法将以某种方式查找新的（原型）命令对象。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with createCommand()
    // overridden to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```

##### 有关基于Java的配置如何在内部工作的更多信息

考虑以下示例，该示例显示了一个带`@Bean`注释的方法被调用两次：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```

`clientDao()`被称为一次`clientService1()`和一次`clientService2()`。由于此方法创建的新实例`ClientDaoImpl`并返回它，因此通常希望有两个实例（每个服务一个）。那肯定是有问题的：在Spring中，实例化的bean在`singleton`默认情况下具有作用域。这就是神奇的地方：所有`@Configuration`类在启动时都使用子类化`CGLIB`。在子类中，子方法在调用父方法并创建新实例之前，首先检查容器中是否有任何缓存（作用域）的bean。

|      | 根据bean的范围，行为可能有所不同。我们在这里谈论单例。 |
| ---- | ------------------------------------------------------ |
|      |                                                        |

|      | 从Spring 3.2开始，不再需要将CGLIB添加到您的类路径中，因为CGLIB类已经被重新打包`org.springframework.cglib`并直接包含在spring-core JAR中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 由于CGLIB在启动时会动态添加功能，因此存在一些限制。特别是，配置类不能是最终的。但是，从4.3版本开始，配置类中允许使用任何构造函数，包括`@Autowired`对默认注入使用 或单个非默认构造函数声明。如果您希望避免任何CGLIB施加的限制，请考虑`@Bean` 在非`@Configuration`类上声明您的方法（例如，在普通`@Component`类上声明）。`@Bean`然后不会截获方法之间的跨方法调用，因此您必须专门依赖那里的构造函数或方法级别的依赖项注入。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.12.5。组成基于Java的配置

Spring的基于Java的配置功能使您可以编写批注，从而可以降低配置的复杂性。

##### 使用`@Import`注释

就像``在Spring XML文件中使用元素来帮助模块化配置一样，`@Import`注释允许`@Bean`从另一个配置类加载定义，如以下示例所示：

爪哇

科特林

```java
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```

现在，无需同时指定两者`ConfigA.class`和`ConfigB.class`实例化上下文，只需`ConfigB`显式提供，如以下示例所示：

爪哇

科特林

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

这种方法简化了容器的实例化，因为只需要处理一个类，而不是要求您`@Configuration`在构造过程中记住大量潜在的 类。

|      | 从Spring Framework 4.2开始，`@Import`还支持对常规组件类的引用，类似于该`AnnotationConfigApplicationContext.register`方法。如果要通过使用一些配置类作为入口点来显式定义所有组件，从而避免组件扫描，则此功能特别有用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

###### 注入对导入`@Bean`定义的依赖

前面的示例有效，但过于简单。在大多数实际情况下，Bean在配置类之间相互依赖。使用XML时，这不是问题，因为不涉及任何编译器，并且您可以声明 `ref="someBean"`并信任Spring以便在容器初始化期间对其进行处理。使用`@Configuration`类时，Java编译器会在配置模型上施加约束，因为对其他bean的引用必须是有效的Java语法。

幸运的是，解决这个问题很简单。正如[我们已经讨论的那样](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-dependencies)，一种`@Bean`方法可以具有任意数量的描述Bean依赖关系的参数。考虑以下具有多个`@Configuration` 类的更真实的场景，每个类都取决于其他类中声明的bean：

爪哇

科特林

```java
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

还有另一种方法可以达到相同的结果。请记住，`@Configuration`类是在容器中最终只有另一个bean：这意味着他们可以利用 `@Autowired`与`@Value`注入等功能相同的其它bean。

|      | 确保以这种方式注入的依赖项只是最简单的一种。`@Configuration` 在上下文的初始化期间很早就处理了类，并且强制以这种方式注入依赖项可能会导致意外的早期初始化。如上例所示，尽可能使用基于参数的注入。另外，还要特别注意`BeanPostProcessor`和的`BeanFactoryPostProcessor`定义`@Bean`。通常应将那些声明为`static @Bean`方法，而不触发其包含的配置类的实例化。否则，`@Autowired`并且`@Value`可能不会对配置类本身的工作，因为可以将其创建为一个bean实例早于 [`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下示例显示如何将一个bean自动连接到另一个bean：

爪哇

科特林

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

|      | `@Configuration`从Spring Framework 4.3开始，仅支持 在类中进行构造函数注入。还要注意，无需指定`@Autowired`目标bean是否仅定义一个构造函数。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

完全合格的进口豆，易于浏览

在前面的场景中，使用`@Autowired`效果很好，并提供了所需的模块化，但是要确切确定声明自动装配的bean定义的位置仍然有些模棱两可。例如，当开发人员查看时`ServiceConfig`，您如何确切知道该`@Autowired AccountRepository`bean的声明位置？它在代码中不是明确的，这可能很好。请记住， [Spring Tools for Eclipse](https://spring.io/tools)提供了可以渲染图形的工具，这些图形显示了所有接线的方式，这可能就是您所需要的。另外，您的Java IDE可以轻松找到该`AccountRepository`类型的所有声明和使用，并快速向您显示`@Bean`返回该类型的方法的位置。

如果这种歧义是不可接受的，并且您希望从IDE内部直接从一个`@Configuration`类导航到另一个类，请考虑自动装配配置类本身。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}
```

在前面的情况中，哪里`AccountRepository`定义是完全明确的。但是，`ServiceConfig`现在与紧密耦合`RepositoryConfig`。那是权衡。通过使用基于接口的类或基于抽象类的`@Configuration`类，可以在某种程度上缓解这种紧密耦合。考虑以下示例：

爪哇

科特林

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();
}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }
}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class})  // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

现在`ServiceConfig`就具体而言松散耦合 `DefaultRepositoryConfig`，并且内置的IDE工具仍然有用：您可以轻松地获得实现的类型层次结构`RepositoryConfig`。通过这种方式，导航`@Configuration`类及其依赖项与导航基于接口的代码的通常过程没有什么不同。

|      | 如果要影响某些Bean的启动创建顺序，请考虑将其中一些声明为`@Lazy`（用于首次访问而不是在启动时创建）或声明为`@DependsOn`其他某些Bean（确保在当前Bean之前创建特定的其他Bean），后者的直接依赖意味着什么）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 有条件地包含`@Configuration`类或`@Bean`方法

基于某些任意系统状态，有条件地启用或禁用完整的`@Configuration`类甚至单个`@Bean`方法通常很有用。一个常见的示例是`@Profile`仅在Spring中启用了特定配置文件时才使用注释激活Bean `Environment`（ 有关详细[信息](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-definition-profiles)，请参见[Bean定义配置文件](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-definition-profiles)）。

该`@Profile`注释是通过使用一种称为更灵活的注释实际执行[`@Conditional`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/Conditional.html)。该`@Conditional`注释指示特定 `org.springframework.context.annotation.Condition`前应谘询的实施`@Bean`是注册。

`Condition`接口的实现提供了`matches(…)` 返回`true`或的方法`false`。例如，以下清单显示了`Condition`用于的实际 实现`@Profile`：

爪哇

科特林

```java
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    // Read the @Profile annotation attributes
    MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
    if (attrs != null) {
        for (Object value : attrs.get("value")) {
            if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                return true;
            }
        }
        return false;
    }
    return true;
}
```

有关[`@Conditional`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/Conditional.html) 更多详细信息，请参见javadoc。

##### 结合Java和XML配置

Spring的`@Configuration`类支持并非旨在100％完全替代Spring XML。某些工具（例如Spring XML名称空间）仍然是配置容器的理想方法。在使用XML方便或有必要的情况下，您可以选择：通过使用，以“以XML为中心”的方式实例化容器`ClassPathXmlApplicationContext`，或者通过使用`AnnotationConfigApplicationContext`和`@ImportResource`注释以以“ Java中心”的方式 实例化容器。 根据需要导入XML。

###### 以XML为中心的`@Configuration`类使用

最好从XML引导Spring容器并`@Configuration`以即席方式包含 类。例如，在使用Spring XML的大型现有代码库中，根据需要创建`@Configuration`类并从现有XML文件中包含类会更容易。在本节的后面，我们将介绍`@Configuration`在这种“以XML为中心”的情况下使用类的选项。

将`@Configuration`类声明为纯Spring ``元素

请记住，`@Configuration`类最终是容器中的bean定义。在本系列示例中，我们创建一个`@Configuration`名为的类，`AppConfig`并将其包含在其中`system-test-config.xml`作为``定义。因为 ``已打开，所以容器会识别 `@Configuration`注释并 正确处理`@Bean`声明的方法`AppConfig`。

以下示例显示了Java中的普通配置类：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }
}
```

以下示例显示了示例`system-test-config.xml`文件的一部分：

```xml
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

以下示例显示了一个可能的`jdbc.properties`文件：

```
jdbc.url = jdbc：hsqldb：hsql：// localhost / xdb
jdbc.username = sa
jdbc.password =
```

爪哇

科特林

```java
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

|      | 在`system-test-config.xml`文件中，`AppConfig` ``没有声明`id` 元素。尽管这样做是可以接受的，但由于没有其他bean引用过它，因此这是没有必要的，并且不太可能通过名称从容器中显式获取。同样，`DataSource`仅按类型自动对Bean进行接线，因此`id` 严格要求不使用显式Bean 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

使用<context：component-scan />拾取`@Configuration`类

因为`@Configuration`是间注释有`@Component`，`@Configuration`-annotated类自动对于组件扫描的候选者。使用与上一示例相同的方案，我们可以重新定义`system-test-config.xml`以利用组件扫描的优势。请注意，在这种情况下，我们无需显式声明 ``，因为``启用了相同的功能。

以下示例显示了修改后的`system-test-config.xml`文件：

```xml
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

###### `@Configuration` 以类为中心的XML使用 `@ImportResource`

在`@Configuration`类是配置容器的主要机制的应用程序中，仍然有必要至少使用一些XML。在这些情况下，您可以`@ImportResource`根据需要使用和定义尽可能多的XML。这样做实现了一种“以Java为中心”的方法来配置容器，并将XML保持在最低限度。以下示例（包括配置类，定义Bean的XML文件，属性文件和`main`该类）显示了如何使用`@ImportResource`注释来实现按需使用XML的“以Java为中心”的配置：

爪哇

科特林

```java
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}
```

```xml
properties-config.xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```

```
jdbc.properties
jdbc.url = jdbc：hsqldb：hsql：// localhost / xdb
jdbc.username = sa
jdbc.password =
```

爪哇

科特林

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

### 1.13。环境抽象

该[`Environment`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/env/Environment.html)接口是集成在容器中的抽象，可以对应用程序环境的两个关键方面进行建模：[profile](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-definition-profiles) 和[properties](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-property-source-abstraction)。

概要文件是仅在给定概要文件处于活动状态时才向容器注册的Bean定义的命名逻辑组。可以将Bean分配给概要文件，无论是以XML定义还是带有注释。`Environment`对象与配置文件相关的作用是确定当前哪些配置文件（如果有）处于活动状态，以及默认情况下哪些配置文件（如果有）应处于活动状态。

属性在几乎所有应用程序中都起着重要作用，并且可能源自各种来源：属性文件，JVM系统属性，系统环境变量，JNDI，Servlet上下文参数，即席`Properties`对象，`Map`对象等。`Environment`对象与属性有关的角色是为用户提供方便的服务界面，用于配置属性源并从中解析属性。

#### 1.13.1。Bean定义配置文件

Bean定义配置文件在核心容器中提供了一种机制，该机制允许在不同环境中注册不同的Bean。“环境”一词对不同的用户可能具有不同的含义，并且此功能可以帮助解决许多用例，包括：

- 在开发中针对内存中的数据源进行工作，而不是在进行QA或生产时从JNDI查找相同的数据源。
- 仅在将应用程序部署到性能环境中时注册监视基础结构。
- 为客户A和客户B部署注册bean的自定义实现。

考虑实际应用中第一个用例的需求 `DataSource`。在测试环境中，配置可能类似于以下内容：

爪哇

科特林

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```

现在，假设该应用程序的数据源已在生产应用程序服务器的JNDI目录中注册，请考虑如何将该应用程序部署到QA或生产环境中。`dataSource`现在，我们的bean看起来像下面的清单：

爪哇

科特林

```java
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

问题是如何根据当前环境在使用这两种变体之间进行切换。随着时间的流逝，Spring用户已经设计出许多方法来完成此任务，通常依靠系统环境变量和``包含以下内容的XML 语句的组合`${placeholder}`根据环境变量的值解析为正确的配置文件路径的令牌。Bean定义配置文件是一个核心容器功能，可提供此问题的解决方案。

如果我们概括前面特定于环境的Bean定义示例中所示的用例，那么最终需要在某些上下文中而不是在其他上下文中注册某些Bean定义。您可能会说您要在情况A中注册一个特定的bean定义配置文件，在情况B中注册一个不同的配置文件。我们首先更新配置以反映这种需求。

##### 使用 `@Profile`

该[`@Profile`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/Profile.html) 注解让你指示组件有资格登记在一个或多个指定的配置文件是活动的。使用前面的示例，我们可以`dataSource`如下重写配置：

爪哇

科特林

```java
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```

爪哇

科特林

```java
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

|      | 如前所述，使用`@Bean`方法时，通常选择使用程序化JNDI查找，方法是使用Spring的`JndiTemplate`/ `JndiLocatorDelegate`helpers或`InitialContext`前面显示的直接JNDI 用法，而不使用`JndiObjectFactoryBean` 变体，这将迫使您将返回类型声明为该`FactoryBean`类型。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

配置文件字符串可以包含简单的配置文件名称（例如`production`）或配置文件表达式。配置文件表达式允许表达更复杂的配置文件逻辑（例如`production & us-east`）。概要文件表达式中支持以下运算符：

- `!`：配置文件的逻辑“不”
- `&`：配置文件的逻辑“与”
- `|`：配置文件的逻辑“或”

|      | 您不能在不使用括号的情况下混合使用`&`and `|`运算符。例如， `production & us-east | eu-central`不是有效的表达式。它必须表示为 `production & (us-east | eu-central)`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您可以将其`@Profile`用作[元注释](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-meta-annotations)，以创建自定义的组合注释。以下示例定义了一个自定义 `@Production`批注，您可以将其用作替代品 `@Profile("production")`：

爪哇

科特林

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

|      | 如果用`@Configuration`标记了一个类，则除非一个或多个指定的配置文件处于活动状态，否则将忽略与该类关联的`@Profile`所有`@Bean`方法和 `@Import`注释。如果使用标记了`@Component`或`@Configuration`类`@Profile({"p1", "p2"})`，除非激活了配置文件“ p1”或“ p2”，否则该类不会注册或处理。如果给定的配置文件以NOT运算符（`!`）为前缀，则仅在该配置文件不活动时才注册带注释的元素。例如，给定`@Profile({"p1", "!p2"})`，如果配置文件“ p1”处于活动状态或配置文件“ p2”未处于活动状态，则会进行注册。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`@Profile` 也可以在方法级别声明为仅包含配置类的一个特定bean（例如，特定bean的替代变体），如以下示例所示：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") 
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") 
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

|      | 该`standaloneDataSource`方法仅在`development`配置文件中可用。 |
| ---- | ------------------------------------------------------------ |
|      | 该`jndiDataSource`方法仅在`production`配置文件中可用。       |

|      | 使用`@Profile`on `@Bean`方法时，可能会出现特殊情况：对于具有`@Bean`相同Java方法名称的重载方法（类似于构造函数重载），`@Profile`需要在所有重载方法上一致声明条件。如果条件不一致，则仅重载方法中第一个声明的条件很重要。因此，`@Profile`不能用于选择具有特定自变量签名的重载方法。在创建时，同一bean的所有工厂方法之间的解析都遵循Spring的构造函数解析算法。如果要定义具有不同概要文件条件的备用Bean，请使用不同的Java方法名称，它们通过使用`@Bean`name属性指向相同的Bean名称，如前面的示例所示。如果参数签名都相同（例如，所有变体都具有no-arg工厂方法），则这是首先在有效Java类中表示这种排列的唯一方法（因为只能有一个特定名称和参数签名的方法）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### XML Bean定义配置文件

XML对应项是元素的`profile`属性``。我们前面的示例配置可以重写为两个XML文件，如下所示：

```xml
<beans profile="development"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>
```

```xml
<beans profile="production"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```

也可以避免``在同一文件中拆分和嵌套元素，如以下示例所示：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="development">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
</beans>
```

在`spring-bean.xsd`受到了制约，使这些元素只能作为文件中的最后一个人。这应该有助于提供灵活性，而不会引起XML文件混乱。

|      | XML对应项不支持前面描述的配置文件表达式。但是，可以通过使用`!`运算符来取消配置文件。也可以通过嵌套配置文件来应用逻辑“和”，如以下示例所示：`                                           `在前面的示例中，`dataSource`如果`production`和 `us-east`配置文件都处于活动状态，则该bean被公开。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 激活个人资料

现在我们已经更新了配置，我们仍然需要指示Spring哪个配置文件处于活动状态。如果我们现在启动示例应用程序，则会看到`NoSuchBeanDefinitionException`抛出的错误，因为容器找不到名为的Spring bean `dataSource`。

可以通过多种方式来激活配置文件，但是最直接的方法是针对`Environment`通过可以使用的API以 编程方式进行配置`ApplicationContext`。以下示例显示了如何执行此操作：

爪哇

科特林

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

另外，您还可以通过`spring.profiles.active`属性声明性地激活配置文件，可以通过系统环境变量，JVM系统属性，中的servlet上下文参数`web.xml`或什至作为JNDI中的条目来指定概要文件 （请参见[`PropertySource`Abstraction](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-property-source-abstraction)）。在集成测试中，可以通过使用 模块中的`@ActiveProfiles`注释来声明活动配置文件`spring-test`（请参阅[环境配置文件的上下文配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/testing.html#testcontext-ctx-management-env-profiles)）。

请注意，配置文件不是“非此即彼”的命题。您可以一次激活多个配置文件。通过编程，您可以为该`setActiveProfiles()`方法提供多个配置文件名称，该名称 接受`String…`varargs。以下示例激活多个配置文件：

爪哇

科特林

```java
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```

以声明的方式，`spring.profiles.active`可以接受以逗号分隔的配置文件名称列表，如以下示例所示：

```
    -Dspring.profiles.active =“ profile1，profile2”
```

##### 默认配置文件

默认配置文件表示默认情况下启用的配置文件。考虑以下示例：

爪哇

科特林

```java
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

如果没有任何配置文件处于活动状态，则将`dataSource`创建。您可以看到这是为一个或多个bean提供默认定义的一种方法。如果启用了任何配置文件，则默认配置文件将不适用。

您可以通过更改默认的配置文件的名称`setDefaultProfiles()`上`Environment`，或者声明，通过使用`spring.profiles.default`属性。

#### 1.13.2。`PropertySource`抽象化

Spring的`Environment`抽象提供了可配置属性源层次结构上的搜索操作。考虑以下清单：

爪哇

科特林

```java
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("my-property");
System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
```

在前面的代码片段中，我们看到了一种询问Spring是否`my-property`为当前环境定义了该属性的高级方法。为了回答这个问题，`Environment`对象在一组对象上执行搜索[`PropertySource`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/env/PropertySource.html) 。A `PropertySource`是对任何键-值对源的简单抽象，Spring的[`StandardEnvironment`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/env/StandardEnvironment.html) 配置有两个PropertySource对象-一个代表JVM系统属性集（`System.getProperties()`）和一个代表系统环境变量集（`System.getenv()`）。

|      | 这些默认属性源存在于中`StandardEnvironment`，供在独立应用程序中使用。[`StandardServletEnvironment`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/context/support/StandardServletEnvironment.html) 用其他默认属性源（包括servlet配置和servlet上下文参数）填充。它可以选择启用[`JndiPropertySource`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/jndi/JndiPropertySource.html)。有关详细信息，请参见javadoc。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

具体来说，当您使用时`StandardEnvironment`，`env.containsProperty("my-property")` 如果运行时存在`my-property`系统属性或`my-property`环境变量，则对的调用将返回true 。

|      | 执行的搜索是分层的。默认情况下，系统属性优先于环境变量。因此，如果`my-property`在调用时在两个地方同时设置了`env.getProperty("my-property")`该属性，则系统属性值将“获胜”并返回。请注意，属性值不会合并，而是会被前面的条目完全覆盖。对于common `StandardServletEnvironment`，完整层次结构如下，最高优先级条目位于顶部：ServletConfig参数（如果适用，例如在`DispatcherServlet`上下文的情况下）ServletContext参数（web.xml上下文参数条目）JNDI环境变量（`java:comp/env/`条目）JVM系统属性（`-D`命令行参数）JVM系统环境（操作系统环境变量） |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

最重要的是，整个机制是可配置的。也许您具有要集成到此搜索中的自定义属性源。为此，请实现并实例化自己的实例`PropertySource`并将其添加到`PropertySources`current 的集合中`Environment`。以下示例显示了如何执行此操作：

爪哇

科特林

```java
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

在前面的代码中，`MyPropertySource`已在搜索中添加了最高优先级。如果它包含一个`my-property`属性，则会检测到并返回该属性，从而支持`my-property`任何其他属性`PropertySource`。该 [`MutablePropertySources`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/env/MutablePropertySources.html) API公开了许多方法，这些方法可以精确地控制属性源集。

#### 1.13.3。使用`@PropertySource`

该[`@PropertySource`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/annotation/PropertySource.html) 注解提供便利和声明的机制添加`PropertySource` 到Spring的`Environment`。

给定一个`app.properties`包含键-值对的名为的文件`testbean.name=myTestBean`，以下`@Configuration`类以`@PropertySource`一种调用`testBean.getName()`return 的方式使用`myTestBean`：

爪哇

科特林

```java
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

资源位置中`${…}`存在的所有占位符`@PropertySource`都是根据已经针对环境注册的一组属性源来解析的，如以下示例所示：

爪哇

科特林

```java
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

假设`my.placeholder`存在于已注册的属性源之一（例如，系统属性或环境变量）中，则占位符将解析为相应的值。如果不是，则`default/path`用作默认值。如果未指定默认值并且无法解析属性， `IllegalArgumentException`则抛出。

|      | 该`@PropertySource`注释是可重复的，根据Java的8约定。但是，所有此类`@PropertySource`批注都需要在同一级别上声明，可以直接在配置类上声明，也可以在同一自定义批注中声明为元批注。不建议将直接注释和元注释混合使用，因为直接注释会有效地覆盖元注释。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.13.4。声明中的占位符解析

从历史上看，元素中占位符的值只能根据JVM系统属性或环境变量来解析。这已不再是这种情况。由于`Environment`抽象是在整个容器中集成的，因此很容易通过它路由占位符的解析。这意味着您可以按照自己喜欢的任何方式配置解析过程。您可以更改搜索系统属性和环境变量的优先级，也可以完全删除它们。您还可以根据需要将自己的属性源添加到组合中。

具体而言，无论该`customer` 属性在何处定义，以下语句均有效，只要该属性在以下位置可用`Environment`：

```xml
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```

### 1.14。注册一个`LoadTimeWeaver`

当`LoadTimeWeaver`类加载到Java虚拟机（JVM）中时，Spring会使用来动态转换类。

要启用加载时编织，可以将`@EnableLoadTimeWeaving`_ 添加到您的一个 `@Configuration`类，如以下示例所示：

爪哇

科特林

```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```

另外，对于XML配置，可以使用`context:load-time-weaver`元素：

```xml
<beans>
    <context:load-time-weaver/>
</beans>
```

一旦针对进行了配置，则其中的`ApplicationContext`任何bean都`ApplicationContext` 可以实现`LoadTimeWeaverAware`，从而接收到对加载时韦弗实例的引用。与[Spring的JPA支持](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#orm-jpa)结合使用时，该功能特别有用， 因为对于JPA类转换，可能需要加载时间编织。有关[`LocalContainerEntityManagerFactoryBean`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/orm/jpa/LocalContainerEntityManagerFactoryBean.html) 更多详细信息，请查阅 javadoc。有关AspectJ加载时编织的更多信息，请参见[Spring Framework中的AspectJ加载时编织](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-aj-ltw)。

### 1.15。的其他功能`ApplicationContext`

如[本章简介](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans)中所述，该`org.springframework.beans.factory` 软件包提供了用于管理和操纵Bean的基本功能，包括以编程方式。的`org.springframework.context`封装增加了 [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html) 接口，它扩展了`BeanFactory`界面，除了延伸其它接口以更应用面向框架的方式提供附加的功能。许多人`ApplicationContext`以完全声明性的方式使用，甚至没有以编程方式创建它，而是依靠支持类，例如在Java EE Web应用程序正常启动过程中`ContextLoader`自动实例化一个 `ApplicationContext`。

为了`BeanFactory`以更加面向框架的方式增强功能，上下文包还提供以下功能：

- 通过`MessageSource`界面访问i18n样式的消息。
- 通过`ResourceLoader`界面访问资源，例如URL和文件。
- 事件发布，即`ApplicationListener`通过使用接口向实现接口的bean `ApplicationEventPublisher`。
- 加载多个（分层）上下文，使每个上下文都通过`HierarchicalBeanFactory`界面集中在一个特定的层上，例如应用程序的Web层 。

#### 1.15.1。国际化使用`MessageSource`

该`ApplicationContext`接口扩展了称为的接口`MessageSource`，因此提供了国际化（“ i18n”）功能。Spring还提供了 `HierarchicalMessageSource`接口，该接口可以分层解析消息。这些接口一起提供了Spring影响消息解析的基础。这些接口上定义的方法包括：

- `String getMessage(String code, Object[] args, String default, Locale loc)`：用于从中检索消息的基本方法`MessageSource`。如果找不到指定语言环境的消息，则使用默认消息。使用`MessageFormat`标准库提供的功能，传入的所有参数都将成为替换值。
- `String getMessage(String code, Object[] args, Locale loc)`：与以前的方法基本相同，但有一个区别：不能指定默认消息。如果找不到该消息，`NoSuchMessageException`则引发a。
- `String getMessage(MessageSourceResolvable resolvable, Locale locale)`：前述方法中使用的所有属性也都包装在名为的类中 `MessageSourceResolvable`，您可以将其与此方法一起使用。

当`ApplicationContext`被加载时，它自动搜索`MessageSource` 在上下文中定义的bean。Bean必须具有名称`messageSource`。如果找到了这样的bean，则对先前方法的所有调用都将委派给消息源。如果找不到消息源，则`ApplicationContext`尝试查找包含同名bean的父对象。如果是这样，它将使用该bean作为`MessageSource`。如果`ApplicationContext`无法找到任何消息源，则将 `DelegatingMessageSource`实例化为空 ，以便能够接受对上述方法的调用。

Spring提供了两种`MessageSource`实现，`ResourceBundleMessageSource`和 `StaticMessageSource`。两者都是`HierarchicalMessageSource`为了执行嵌套消息传递而实现的。在`StaticMessageSource`很少使用，但提供编程的方式向消息源添加消息。以下示例显示`ResourceBundleMessageSource`：

```xml
<beans>
    <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>format</value>
                <value>exceptions</value>
                <value>windows</value>
            </list>
        </property>
    </bean>
</beans>
```

该示例假定您有三个资源包，称为`format`，`exceptions`并`windows` 在类路径中定义。任何解析消息的请求都通过JDK标准的通过`ResourceBundle`对象解析消息的方式来处理。就本示例而言，假定上述两个资源束文件的内容如下：

```
    ＃在format.properties中
    message =扬子鳄摇滚！
```

```
    ＃在exceptions.properties中
    arguments.required =需要{0}参数。
```

下一个示例显示了执行该`MessageSource`功能的程序。请记住，所有`ApplicationContext`实现也是`MessageSource` 实现，因此可以强制转换为`MessageSource`接口。

爪哇

科特林

```java
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", Locale.ENGLISH);
    System.out.println(message);
}
```

以上程序的结果输出如下：

```
短吻鳄摇滚！
```

概括而言，`MessageSource`定义在名为的文件中`beans.xml`，该文件位于类路径的根目录下。该`messageSource`bean定义是指通过它的一些资源包的`basenames`属性。这是在列表中传递的三个文件`basenames`属性存在于你的classpath根目录的文件，被称为`format.properties`，`exceptions.properties`和 `windows.properties`分别。

下一个示例显示了传递给消息查找的参数。这些参数将转换为`String`对象，并插入到查找消息中的占位符中。

```xml
<beans>

    <!-- this MessageSource is being used in a web application -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="exceptions"/>
    </bean>

    <!-- lets inject the above MessageSource into this POJO -->
    <bean id="example" class="com.something.Example">
        <property name="messages" ref="messageSource"/>
    </bean>

</beans>
```

爪哇

科特林

```java
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", Locale.ENGLISH);
        System.out.println(message);
    }
}
```

该`execute()`方法的调用结果如下：

```
userDao参数是必需的。
```

关于国际化（“ i18n”），Spring的各种`MessageSource` 实现遵循与标准JDK相同的语言环境解析和后备规则 `ResourceBundle`。总之，和继续该示例`messageSource`先前定义的，如果你想对英国（解析的消息`en-GB`）的语言环境，你需要创建的文件名为`format_en_GB.properties`，`exceptions_en_GB.properties`和 `windows_en_GB.properties`分别。

通常，语言环境解析由应用程序的周围环境管理。在以下示例中，手动指定了针对其解析（英国）消息的语言环境：

```
＃in exceptions_zh_CN.properties
arguments.required = Ebagum lad，我说“ {0}”参数是必需的。
```

爪哇

科特林

```java
public static void main(final String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("argument.required",
        new Object [] {"userDao"}, "Required", Locale.UK);
    System.out.println(message);
}
```

运行上述程序的结果输出如下：

```
Ebagum伙计，“ userDao”参数是必需的，我说是必需的。
```

您还可以使用该`MessageSourceAware`接口获取对`MessageSource`已定义的任何内容的引用 。在创建和配置Bean时，将在`ApplicationContext`实现该`MessageSourceAware`接口的中定义的任何Bean都 与应用程序上下文一起注入`MessageSource`。

|      | 作为替代`ResourceBundleMessageSource`，Spring提供了一个 `ReloadableResourceBundleMessageSource`类。该变体支持相同的捆绑文件格式，但比基于标准JDK的`ResourceBundleMessageSource`实现更灵活 。特别是，它允许从任何Spring资源位置（不仅是从类路径）读取文件，并支持热重装捆绑属性文件（同时在它们之间进行有效缓存）。有关[`ReloadableResourceBundleMessageSource`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/support/ReloadableResourceBundleMessageSource.html) 详细信息，请参见javadoc。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.15.2。标准和自定义事件

`ApplicationContext`通过`ApplicationEvent` 类和`ApplicationListener`接口提供中的事件处理。如果将实现`ApplicationListener`接口的Bean 部署到上下文中，则每次 将Bean `ApplicationEvent`发布到时`ApplicationContext`，都会通知该Bean。本质上，这是标准的观察者设计模式。

|      | 从Spring 4.2开始，事件基础结构得到了显着改进，并提供了[基于注释的模型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#context-functionality-events-annotation)以及发布任意事件的能力（即，对象不一定从扩展`ApplicationEvent`）。发布此类对象后，我们会为您包装一个事件。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

下表描述了Spring提供的标准事件：

| 事件                         | 说明                                                         |
| :--------------------------- | :----------------------------------------------------------- |
| `ContextRefreshedEvent`      | 在`ApplicationContext`初始化或刷新时发布（例如，通过使用接口`refresh()`上的方法`ConfigurableApplicationContext`）。在这里，“已初始化”表示已加载所有bean，检测并激活了后处理器bean，已预先实例化单例，并且该`ApplicationContext`对象已准备就绪可以使用。只要尚未关闭上下文，就可以多次触发刷新，前提是所选对象`ApplicationContext`实际上支持这种“热”刷新。例如，`XmlWebApplicationContext`支持热刷新，但`GenericApplicationContext`不支持 。 |
| `ContextStartedEvent`        | `ApplicationContext`使用界面`start()`上的方法 启动时发布`ConfigurableApplicationContext`。在这里，“启动”意味着所有`Lifecycle` bean都收到一个明确的启动信号。通常，此信号用于在显式停止后重新启动Bean，但也可以用于启动尚未配置为自动启动的组件（例如，尚未在初始化时启动的组件）。 |
| `ContextStoppedEvent`        | `ApplicationContext`通过使用界面`stop()`上的方法 停止时发布`ConfigurableApplicationContext`。此处，“已停止”表示所有`Lifecycle` bean均收到明确的停止信号。停止的上下文可以通过`start()`调用重新启动 。 |
| `ContextClosedEvent`         | 当发布时间`ApplicationContext`是由使用封闭`close()`方法的上`ConfigurableApplicationContext`接口或经由JVM关闭挂钩。在这里，“封闭”意味着所有单例豆将被销毁。关闭上下文后，它将达到使用寿命，无法刷新或重新启动。 |
| `RequestHandledEvent`        | 一个特定于Web的事件，告诉所有Bean HTTP请求已得到服务。请求完成后，将发布此事件。此事件仅适用于使用Spring的Web应用程序`DispatcherServlet`。 |
| `ServletRequestHandledEvent` | 该类的子类`RequestHandledEvent`添加了特定于Servlet的上下文信息。 |

您还可以创建和发布自己的自定义事件。以下示例显示了一个简单的类，该类扩展了Spring的`ApplicationEvent`基类：

爪哇

科特林

```java
public class BlackListEvent extends ApplicationEvent {

    private final String address;
    private final String content;

    public BlackListEvent(Object source, String address, String content) {
        super(source);
        this.address = address;
        this.content = content;
    }

    // accessor and other methods...
}
```

要发布自定义`ApplicationEvent`，请在`publishEvent()`上调用方法 `ApplicationEventPublisher`。通常，这是通过创建一个实现`ApplicationEventPublisherAware`并注册为Spring bean 的类来完成的 。以下示例显示了此类：

爪哇

科特林

```java
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blackList;
    private ApplicationEventPublisher publisher;

    public void setBlackList(List<String> blackList) {
        this.blackList = blackList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String content) {
        if (blackList.contains(address)) {
            publisher.publishEvent(new BlackListEvent(this, address, content));
            return;
        }
        // send email...
    }
}
```

在配置时，Spring容器检测到该`EmailService`实现 `ApplicationEventPublisherAware`并自动调用 `setApplicationEventPublisher()`。实际上，传入的参数是Spring容器本身。您正在通过其`ApplicationEventPublisher`界面与应用程序上下文进行 交互。

要接收该定制`ApplicationEvent`，您可以创建一个实现 `ApplicationListener`并注册为Spring bean的类。以下示例显示了此类：

爪哇

科特林

```java
public class BlackListNotifier implements ApplicationListener<BlackListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    public void onApplicationEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

注意，`ApplicationListener`通常使用自定义事件的类型来参数化（`BlackListEvent`在前面的示例中）。这意味着该`onApplicationEvent()`方法可以保持类型安全，而无需进行向下转换。您可以根据需要注册任意数量的事件侦听器，但是请注意，默认情况下，事件侦听器会同步接收事件。这意味着该`publishEvent()`方法将阻塞，直到所有侦听器都已完成对事件的处理为止。这种同步和单线程方法的一个优点是，当侦听器收到事件时，如果有可用的事务上下文，它将在发布者的事务上下文内部进行操作。如果需要其他用于事件发布的策略，请参阅javadoc中Spring的 [`ApplicationEventMulticaster`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/event/ApplicationEventMulticaster.html)界面和[`SimpleApplicationEventMulticaster`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/context/event/SimpleApplicationEventMulticaster.html) 实现的配置选项。

以下示例显示了用于注册和配置上述每个类的Bean定义：

```xml
<bean id="emailService" class="example.EmailService">
    <property name="blackList">
        <list>
            <value>known.spammer@example.org</value>
            <value>known.hacker@example.org</value>
            <value>john.doe@example.org</value>
        </list>
    </property>
</bean>

<bean id="blackListNotifier" class="example.BlackListNotifier">
    <property name="notificationAddress" value="blacklist@example.org"/>
</bean>
```

将所有内容放在一起，当调用bean 的`sendEmail()`方法时`emailService`，如果有任何电子邮件消息应列入黑名单，则将`BlackListEvent`发布一个类型的自定义事件 。该`blackListNotifier`bean被注册为 `ApplicationListener`接收的`BlackListEvent`，在这一点上，可以通知有关各方。

|      | Spring的事件机制旨在在同一应用程序上下文内在Spring bean之间进行简单的通信。但是，对于更复杂的企业集成需求，单独维护的 [Spring Integration](https://projects.spring.io/spring-integration/)项目提供了对基于众所周知的Spring编程模型构建轻量级，[面向模式](https://www.enterpriseintegrationpatterns.com/)，事件驱动的架构的完整支持 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 基于注释的事件侦听器

从Spring 4.2开始，您可以使用`@EventListener`注释在托管Bean的任何公共方法上注册事件侦听器。该`BlackListNotifier`可改写如下：

爪哇

科特林

```java
public class BlackListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlackListEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

方法签名再次声明其侦听的事件类型，但是这次使用灵活的名称且未实现特定的侦听器接口。只要实际事件类型在其实现层次结构中解析您的通用参数，也可以通过通用类型来缩小事件类型。

如果您的方法应该侦听多个事件，或者您要完全不使用任何参数来定义它，则事件类型也可以在注释本身上指定。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    // ...
}
```

也可以通过使用`condition`定义[`SpEL`表达式](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions)的注释的属性来添加其他运行时过滤，该[表达式](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions)应匹配以针对特定事件实际调用该方法。

以下示例说明了仅当`content`事件的属性等于时，才可以重写我们的通知程序以进行调用 `my-event`：

爪哇

科特林

```java
@EventListener(condition = "#blEvent.content == 'my-event'")
public void processBlackListEvent(BlackListEvent blEvent) {
    // notify appropriate parties via notificationAddress...
}
```

每个`SpEL`表达式针对专用上下文进行评估。下表列出了可用于上下文的项目，以便您可以将它们用于条件事件处理：

| 名称       | 位置     | 描述                                                         | 例                                                           |
| :--------- | :------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 事件       | 根对象   | 实际的`ApplicationEvent`。                                   | `#root.event` 要么 `event`                                   |
| 参数数组   | 根对象   | 用于调用方法的参数（作为对象数组）。                         | `#root.args`或`args`; `args[0]`访问第一个参数，等等。        |
| *参数名称* | 评价背景 | 任何方法参数的名称。如果由于某种原因这些名称不可用（例如，因为在已编译的字节码中没有调试信息），则还可以使用`#a<#arg>`其中`<#arg>`参数代表参数索引（从0开始）的语法提供各个参数。 | `#blEvent`或`#a0`（您也可以使用`#p0`或`#p<#arg>`参数符号作为别名） |

请注意`#root.event`，即使您的方法签名实际上指向已发布的任意对象，也可以使您访问基础事件。

如果由于处理另一个事件而需要发布一个事件，则可以更改方法签名以返回应发布的事件，如以下示例所示：

爪哇

科特林

```java
@EventListener
public ListUpdateEvent handleBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress and
    // then publish a ListUpdateEvent...
}
```

|      | [异步侦听](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#context-functionality-events-async) 器不支持此功能 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

此新方法`ListUpdateEvent`为`BlackListEvent`上述方法处理的每个操作发布一个新方法。如果您需要发布多个事件，则可以返回一个`Collection`事件。

##### 异步侦听器

如果您希望特定的侦听器异步处理事件，则可以重用 [常规`@Async`支持](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#scheduling-annotation-support-async)。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@EventListener
@Async
public void processBlackListEvent(BlackListEvent event) {
    // BlackListEvent is processed in a separate thread
}
```

使用异步事件时，请注意以下限制：

- 如果异步事件侦听器抛出`Exception`，则不会传播到调用者。请参阅`AsyncUncaughtExceptionHandler`以获取更多详细信息。
- 异步事件侦听器方法无法通过返回值来发布后续事件。如果您需要发布另一个事件作为处理的结果，请插入一个 [`ApplicationEventPublisher`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/aop/interceptor/AsyncUncaughtExceptionHandler.html) 以手动发布事件。

##### 订购听众

如果需要先调用一个侦听器，则可以将`@Order` 注释添加到方法声明中，如以下示例所示：

爪哇

科特林

```java
@EventListener
@Order(42)
public void processBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress...
}
```

##### 一般事件

您还可以使用泛型来进一步定义事件的结构。考虑使用 `EntityCreatedEvent`where `T`是创建的实际实体的类型。例如，您可以创建以下侦听器定义只接收`EntityCreatedEvent`了 `Person`：

爪哇

科特林

```java
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    // ...
}
```

由于类型擦除，只有在触发的事件解析了事件侦听器所基于的通用参数（即`class PersonCreatedEvent extends EntityCreatedEvent { … }`）时，此方法才起作用 。

在某些情况下，如果所有事件都遵循相同的结构，这可能会变得很乏味（就像前面示例中的事件一样）。在这种情况下，您可以实现`ResolvableTypeProvider`超出运行时环境提供的范围之外的框架指导。以下事件显示了如何执行此操作：

爪哇

科特林

```java
public class EntityCreatedEvent<T> extends ApplicationEvent implements ResolvableTypeProvider {

    public EntityCreatedEvent(T entity) {
        super(entity);
    }

    @Override
    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(getClass(), ResolvableType.forInstance(getSource()));
    }
}
```

|      | 这不仅适用于`ApplicationEvent`作为事件发送的任何对象，而且适用于该对象。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.15.3。方便地访问低级资源

为了最佳使用和理解应用程序上下文，您应该熟悉Spring的`Resource`抽象，如[参考资料中所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources)。

应用程序上下文是`ResourceLoader`，可用于加载`Resource`对象。A `Resource`本质上是JDK `java.net.URL`类的功能更丰富的版本。实际上，在适当`Resource`的情况下`java.net.URL`，wrap 的实现包装了一个实例。A `Resource`可以以透明的方式从几乎任何位置获取低级资源，包括从类路径，文件系统位置，可使用标准URL描述的任何位置以及一些其他变体。如果资源位置字符串是没有任何特殊前缀的简单路径，则这些资源的来源是特定的，并且适合于实际的应用程序上下文类型。

您可以配置部署到应用程序上下文中的Bean，以实现特殊的回调接口`ResourceLoaderAware`，以便在初始化时以应用程序上下文本身作为传入自动回调`ResourceLoader`。您还可以公开type属性，`Resource`用于访问静态资源。它们像其他任何属性一样注入其中。您可以将这些`Resource` 属性指定为简单`String`路径，并在`Resource`部署Bean时依靠从这些文本字符串到实际对象的自动转换。

提供给`ApplicationContext`构造函数的一个或多个位置路径实际上是资源字符串，并且根据特定的上下文实现以简单的形式对其进行了适当处理。例如，`ClassPathXmlApplicationContext`将简单的位置路径视为类路径位置。您也可以使用带有特殊前缀的位置路径（资源字符串）来强制从类路径或URL中加载定义，而不管实际的上下文类型如何。

#### 1.15.4。Web应用程序的便捷ApplicationContext实例化

您可以`ApplicationContext`使用声明性地创建实例 `ContextLoader`。当然，您也可以`ApplicationContext`使用其中一种`ApplicationContext`实现以编程方式创建实例。

您可以`ApplicationContext`使用来注册一个`ContextLoaderListener`，如以下示例所示：

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

侦听器检查`contextConfigLocation`参数。如果该参数不存在，那么侦听器将使用它`/WEB-INF/applicationContext.xml`作为默认值。当参数确实存在时，侦听器将`String`使用预定义的定界符（逗号，分号和空格）将分隔，并将这些值用作搜索应用程序上下文的位置。还支持蚂蚁风格的路径模式。示例是`/WEB-INF/*Context.xml`（对于名称以结尾 `Context.xml`且位于`WEB-INF`目录`/WEB-INF/**/*Context.xml` 中的所有文件）和（对于的任何子目录中的所有此类文件`WEB-INF`）。

#### 1.15.5。将Spring部署`ApplicationContext`为Java EE RAR文件

可以将Spring部署`ApplicationContext`为RAR文件，将上下文及其所有必需的Bean类和库JAR封装在Java EE RAR部署单元中。这等效于引导`ApplicationContext`能够访问Java EE服务器功能的独立服务器（仅托管在Java EE环境中）。对于部署无头WAR文件的情况，RAR部署是一种更自然的选择-实际上，这种WAR文件没有任何HTTP入口点，仅用于`ApplicationContext`在Java EE环境中引导Spring 。

对于不需要HTTP入口点而仅由消息端点和计划的作业组成的应用程序上下文，RAR部署是理想的选择。在这样的上下文中，Bean可以使用应用程序服务器资源，例如JTA事务管理器以及绑定到JNDI的JDBC `DataSource`实例和JMS `ConnectionFactory`实例，还可以通过该平台的JMX服务器注册-全部通过Spring的标准事务管理以及JNDI和JMX支持工具。应用程序组件还可以`WorkManager`通过Spring的`TaskExecutor`抽象与应用程序服务器的JCA 进行交互。

有关[`SpringContextResourceAdapter`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/jca/context/SpringContextResourceAdapter.html) RAR部署中涉及的配置详细信息，请参见该类的javadoc 。

为了将Spring ApplicationContext作为Java EE RAR文件进行简单部署：

1. 将所有应用程序类打包到RAR文件（这是具有不同文件扩展名的标准JAR文件）中。将所有必需的库JAR添加到RAR归档文件的根目录中。添加一个 `META-INF/ra.xml`部署描述符（如[javadoc中的`SpringContextResourceAdapter`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/jca/context/SpringContextResourceAdapter.html)所示）和相应的Spring XML bean定义文件（通常为 `META-INF/applicationContext.xml`）。
2. 将生成的RAR文件拖放到应用程序服务器的部署目录中。

|      | 这种RAR部署单元通常是独立的。它们不会将组件暴露给外界，甚至不会暴露给同一应用程序的其他模块。与基于RAR的交互`ApplicationContext`通常是通过与其他模块共享的JMS目标进行的。基于RAR的文件`ApplicationContext`还可以例如计划一些作业或对文件系统（或类似文件）中的新文件做出反应。如果需要允许来自外部的同步访问，则可以（例如）导出RMI端点，该端点可以由同一台计算机上的其他应用程序模块使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.16。的`BeanFactory`

该`BeanFactory`API为Spring的IoC功能提供了基础。它的特定合同主要与Spring的其他部分以及相关的第三方框架集成在一起使用，并且其`DefaultListableBeanFactory`实现是更高级别`GenericApplicationContext`容器中的关键委托。

`BeanFactory`和相关接口（例如`BeanFactoryAware`，`InitializingBean`， `DisposableBean`）对于其他框架组件的重要结合点。通过不需要任何注释，甚至不需要反射，它们可以在容器及其组件之间进行非常有效的交互。应用程序级Bean可以使用相同的回调接口，但通常更喜欢通过注释或通过程序配置进行声明式依赖注入。

请注意，核心`BeanFactory`API级别及其`DefaultListableBeanFactory` 实现不对配置格式或要使用的任何组件注释进行假设。所有这些风味都通过扩展名（例如`XmlBeanDefinitionReader`和`AutowiredAnnotationBeanPostProcessor`）进入，并在共享`BeanDefinition`对象上作为核心元数据表示形式进行操作。这就是使Spring的容器如此灵活和可扩展的本质。

#### 1.16.1。`BeanFactory`还是`ApplicationContext`？

本节说明`BeanFactory`和 `ApplicationContext`容器级别之间的区别以及对引导的影响。

`ApplicationContext`除非有充分的理由，否则应使用an，除非将`GenericApplicationContext`其及其子类`AnnotationConfigApplicationContext` 用作自定义引导的常见实现，除非您有充分的理由 。对于所有常见目的，这些都是Spring核心容器的主要入口点：加载配置文件，触发类路径扫描，以编程方式注册Bean定义和带注释的类，以及（从5.0版本开始）注册功能性Bean定义。

因为an `ApplicationContext`包含a的所有功能`BeanFactory`，所以通常建议在平原上使用`BeanFactory`，除非需要完全控制bean处理的场景。在一个`ApplicationContext`（例如， `GenericApplicationContext`实现）中，按照约定（即，按Bean名称或Bean类型-特别是后处理器）检测到几种Bean，而普通`DefaultListableBeanFactory`的对任何特殊的Bean均不可知。

对于许多扩展的容器功能，例如注释处理和AOP代理，[`BeanPostProcessor`扩展点](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-extension-bpp)至关重要。如果仅使用Plain `DefaultListableBeanFactory`，则默认情况下不会检测到此类后处理器并将其激活。这种情况可能会造成混乱，因为您的bean配置实际上没有任何问题。而是在这种情况下，需要通过其他设置完全引导容器。

下表列出了`BeanFactory`and `ApplicationContext`接口和实现所提供的功能。

| 特征                                    | `BeanFactory` | `ApplicationContext` |
| :-------------------------------------- | :------------ | :------------------- |
| Bean实例化/接线                         | 是            | 是                   |
| 集成生命周期管理                        | 没有          | 是                   |
| 自动`BeanPostProcessor`注册             | 没有          | 是                   |
| 自动`BeanFactoryPostProcessor`注册      | 没有          | 是                   |
| 方便的`MessageSource`访问（用于内部化） | 没有          | 是                   |
| 内置`ApplicationEvent`发布机制          | 没有          | 是                   |

要使用显式注册Bean后处理器`DefaultListableBeanFactory`，您需要以编程方式调用`addBeanPostProcessor`，如以下示例所示：

爪哇

科特林

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// populate the factory with bean definitions

// now register any needed BeanPostProcessor instances
factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
factory.addBeanPostProcessor(new MyBeanPostProcessor());

// now start using the factory
```

要将a `BeanFactoryPostProcessor`应用于平原`DefaultListableBeanFactory`，您需要调用其`postProcessBeanFactory`方法，如以下示例所示：

爪哇

科特林

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));

// bring in some property values from a Properties file
PropertySourcesPlaceholderConfigurer cfg = new PropertySourcesPlaceholderConfigurer();
cfg.setLocation(new FileSystemResource("jdbc.properties"));

// now actually do the replacement
cfg.postProcessBeanFactory(factory);
```

在这两种情况下，显式登记步骤是不方便的，这就是为什么在各种`ApplicationContext`变体是优选的超过一个纯 `DefaultListableBeanFactory`在弹簧支持的应用程序，依靠特别是当`BeanFactoryPostProcessor`与`BeanPostProcessor`在一个典型的企业设置实例延长容器的功能性。

|      | 一个`AnnotationConfigApplicationContext`拥有注册的所有常见的标注后处理器并通过配置注解的封面下方额外的处理器，如可能带来`@EnableTransactionManagement`。在Spring基于注释的配置模型的抽象级别上，bean后处理器的概念仅是内部容器详细信息。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 2.资源

本章介绍了Spring如何处理资源以及如何在Spring中使用资源。它包括以下主题：

- [介绍](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-introduction)
- [资源接口](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-resource)
- [内置资源实现](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-implementations)
- [的 `ResourceLoader`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-resourceloader)
- [该`ResourceLoaderAware`接口](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-resourceloaderaware)
- [资源依赖](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-as-dependencies)
- [应用程序上下文和资源路径](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-app-ctx)

### 2.1。介绍

`java.net.URL`不幸的是，Java的用于各种URL前缀的标准类和标准处理程序不足以使所有对低级资源的访问都足够。例如，没有标准化的`URL`实现可用于访问需要从类路径或相对于类路径获取的资源 `ServletContext`。虽然可以为专用`URL` 前缀注册新的处理程序（类似于诸如这样的前缀的现有处理程序`http:`），但这通常很复杂，并且`URL`接口仍然缺少某些理想的功能，例如一种检查资源是否存在的方法。指向。

### 2.2。资源接口

Spring的`Resource`接口旨在成为一种功能更强大的接口，用于抽象化对低级资源的访问。以下清单显示了`Resource`接口定义：

爪哇

科特林

```java
public interface Resource extends InputStreamSource {

    boolean exists();

    boolean isOpen();

    URL getURL() throws IOException;

    File getFile() throws IOException;

    Resource createRelative(String relativePath) throws IOException;

    String getFilename();

    String getDescription();
}
```

如`Resource`接口的定义所示，它扩展了`InputStreamSource` 接口。以下清单显示了`InputStreamSource` 接口的定义：

爪哇

科特林

```java
public interface InputStreamSource {

    InputStream getInputStream() throws IOException;
}
```

该`Resource`界面中一些最重要的方法是：

- `getInputStream()`：找到并打开资源，并返回一个资源以`InputStream`供读取。预计每次调用都会返回一个新的 `InputStream`。呼叫者有责任关闭流。
- `exists()`：返回`boolean`指示此资源是否实际以物理形式存在。
- `isOpen()`：返回，`boolean`指示此资源是否表示具有打开流的句柄。如果为`true`，`InputStream`则不能多次读取，必须只读取一次，然后将其关闭以避免资源泄漏。返回`false`所有常用资源实现（除外）`InputStreamResource`。
- `getDescription()`：返回对此资源的描述，以便在使用该资源时用于错误输出。这通常是标准文件名或资源的实际URL。

其他方法可让您获取表示资源的实际对象`URL`或`File`对象（如果基础实现兼容并且支持该功能）。

`Resource`当需要资源时，Spring本身广泛使用抽象作为许多方法签名中的参数类型。某些Spring API中的其他方法（例如，各种`ApplicationContext`实现的构造函数）采用的 `String`形式，以无修饰或简单的形式创建`Resource`适合该上下文实现的方法，或者通过`String`路径上的特殊前缀，让调用者指定特定的`Resource`实现必须创建和使用。

尽管`Resource`Spring和Spring经常使用该接口，但实际上，在您自己的代码中单独使用该接口作为通用实用工具类来访问资源也非常有用，即使您的代码不了解或不关心其他任何部分春天。虽然这会将您的代码耦合到Spring，但实际上仅将其耦合到这套实用程序类，它们可以作为功能更强大的替代品，`URL`并且可以等同于您为此目的使用的任何其他库。

|      | 该`Resource`抽象并没有改变功能。它尽可能地包装它。例如，`UrlResource`包装器包装URL，然后使用包装`URL`器完成其工作。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.3。内置资源实现

Spring包含以下`Resource`实现：

- [`UrlResource`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-implementations-urlresource)
- [`ClassPathResource`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-implementations-classpathresource)
- [`FileSystemResource`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-implementations-filesystemresource)
- [`ServletContextResource`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-implementations-servletcontextresource)
- [`InputStreamResource`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-implementations-inputstreamresource)
- [`ByteArrayResource`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-implementations-bytearrayresource)

#### 2.3.1。 `UrlResource`

`UrlResource`包装一个`java.net.URL`，可用于访问通常可以通过URL访问的任何对象，例如文件，HTTP目标，FTP目标等。所有URL都有一个标准化的`String`表示形式，因此适当的标准化前缀可用于指示另一种URL类型。这包括`file:`访问文件系统路径，`http:`通过HTTP协议 `ftp:`访问资源，通过FTP访问资源等。

A `UrlResource`是由Java代码通过显式使用`UrlResource`构造函数创建的，但通常在调用带有`String` 用于表示路径的参数的API方法时隐式创建。对于后一种情况，JavaBeans `PropertyEditor`最终决定`Resource`创建哪种类型。如果路径字符串包含众所周知的前缀（例如`classpath:`），那么它将`Resource`为该前缀创建一个适当的专门名称。但是，如果它不能识别前缀，则假定该字符串是标准URL字符串并创建一个`UrlResource`。

#### 2.3.2。 `ClassPathResource`

此类表示应从类路径获取的资源。它使用线程上下文类加载器，给定的类加载器或给定的类来加载资源。

此`Resource`实现支持解析，就`java.io.File`好像类路径资源驻留在文件系统中一样，而不支持类路径资源驻留在jar中并且尚未（通过servlet引擎或任何环境被扩展）到文件系统中。为了解决这个问题，各种`Resource`实现始终支持将解析作为`java.net.URL`。

A `ClassPathResource`是由Java代码通过显式使用`ClassPathResource` 构造函数创建的，但通常在调用带有`String`用于表示路径的参数的API方法时隐式创建 。对于后一种情况，JavaBeans会 在字符串路径上`PropertyEditor`识别特殊前缀，`classpath:`并`ClassPathResource`在这种情况下创建一个。

#### 2.3.3。 `FileSystemResource`

这是一个`Resource`执行`java.io.File`和`java.nio.file.Path`手柄。它支持分辨率为`File`和`URL`。

#### 2.3.4。 `ServletContextResource`

这是资源的`Resource`实现，该`ServletContext`资源解释相关Web应用程序的根目录内的相对路径。

它始终支持流访问和URL访问，但`java.io.File`仅在扩展Web应用程序档案且资源实际位于文件系统上时才允许访问。它是在文件系统上扩展还是直接扩展，或者是否可以直接从JAR或其他类似数据库（可以想到的）中访问，实际上取决于Servlet容器。

#### 2.3.5。 `InputStreamResource`

一个`InputStreamResource`是`Resource`给定的实现`InputStream`。仅当没有特定的`Resource`实现适用时才应使用它。特别是，在可能`ByteArrayResource`的`Resource`情况下，首选 或基于文件的实现中的任何一种。

与其他`Resource`实现相反，这是一个已经打开的资源的描述符。因此，它`true`从返回`isOpen()`。如果需要将资源描述符保存在某个地方，或者需要多次读取流，请不要使用它。

#### 2.3.6。 `ByteArrayResource`

这是`Resource`给定字节数组的实现。它`ByteArrayInputStream`为给定的字节数组创建一个 。

这对于从任何给定的字节数组加载内容而不必求助于一次使用很有用`InputStreamResource`。

### 2.4。的`ResourceLoader`

该`ResourceLoader`接口旨在由可以返回（即加载）`Resource`实例的对象实现。以下清单显示了`ResourceLoader` 接口定义：

爪哇

科特林

```java
public interface ResourceLoader {

    Resource getResource(String location);
}
```

所有应用程序上下文均实现该`ResourceLoader`接口。因此，所有应用程序上下文都可用于获取`Resource`实例。

当您调用`getResource()`特定的应用程序上下文时，并且指定的位置路径没有特定的前缀时，您将获得`Resource`适合该特定应用程序上下文的类型。例如，假设针对`ClassPathXmlApplicationContext`实例执行了以下代码片段：

爪哇

科特林

```java
Resource template = ctx.getResource("some/resource/path/myTemplate.txt");
```

针对`ClassPathXmlApplicationContext`，该代码返回`ClassPathResource`。如果对`FileSystemXmlApplicationContext`实例执行相同的方法，则将返回 `FileSystemResource`。对于`WebApplicationContext`，它将返回 `ServletContextResource`。类似地，它将为每个上下文返回适当的对象。

结果，您可以以适合特定应用程序上下文的方式加载资源。

另一方面，`ClassPathResource`无论应用程序上下文类型如何，您都可以通过指定特殊`classpath:`前缀来强制使用，如以下示例所示：

爪哇

科特林

```java
Resource template = ctx.getResource("classpath:some/resource/path/myTemplate.txt");
```

同样，您可以`UrlResource`通过指定任何标准 `java.net.URL`前缀来强制使用a 。以下两个示例使用`file`和`http` 前缀：

爪哇

科特林

```java
Resource template = ctx.getResource("file:///some/resource/path/myTemplate.txt");
```

爪哇

科特林

```java
Resource template = ctx.getResource("https://myhost.com/resource/path/myTemplate.txt");
```

下表总结了将`String`对象转换为`Resource`对象的策略：

| 字首     | 例                               | 说明                                                         |
| :------- | :------------------------------- | :----------------------------------------------------------- |
| 类路径： | `classpath:com/myapp/config.xml` | 从类路径加载。                                               |
| 文件：   | `file:///data/config.xml`        | `URL`从文件系统作为加载。另请参见[`FileSystemResource`警告](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources-filesystemresource-caveats)。 |
| http：   | `https://myserver/logo.png`      | 载入为`URL`。                                                |
| （没有） | `/data/config.xml`               | 取决于基础`ApplicationContext`。                             |

### 2.5。该`ResourceLoaderAware`接口

该`ResourceLoaderAware`接口是一个特殊的回调接口，用于标识期望提供`ResourceLoader`引用的组件。以下清单显示了`ResourceLoaderAware`接口的定义：

爪哇

科特林

```java
public interface ResourceLoaderAware {

    void setResourceLoader(ResourceLoader resourceLoader);
}
```

当一个类实现`ResourceLoaderAware`并部署到应用程序上下文中（作为Spring托管的bean）时，该类被应用程序上下文识别为`ResourceLoaderAware`。然后`setResourceLoader(ResourceLoader)`，应用程序上下文调用，将自身作为参数提供（请记住，Spring中的所有应用程序上下文都实现了该`ResourceLoader`接口）。

由于an `ApplicationContext`是a `ResourceLoader`，因此bean也可以实现 `ApplicationContextAware`接口并直接使用提供的应用程序上下文来加载资源。但是，通常，`ResourceLoader` 如果需要的话，最好使用专用接口。该代码将仅耦合到资源加载接口（可以视为实用程序接口），而不耦合到整个Spring `ApplicationContext`接口。

在应用程序组件中，您还可以依赖的自动装配`ResourceLoader`作为实现`ResourceLoaderAware`接口的替代方法。“传统” `constructor`和`byType`自动[装配](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-autowire)模式（如“ 自动[装配](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-autowire) ” 中所述）能够分别为`ResourceLoader`构造函数参数或setter方法参数提供a 。为了获得更大的灵活性（包括自动装配字段和多个参数方法的能力），请考虑使用基于注释的自动装配功能。在这种情况下，只要有问题的字段，构造函数或方法带有注释，它们`ResourceLoader`就会自动连接到期望`ResourceLoader`类型的字段，构造函数参数或方法参数`@Autowired`。有关更多信息，请参见[使用`@Autowired`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-autowired-annotation)。

### 2.6。资源依赖

如果Bean本身将通过某种动态过程来确定并提供资源路径，那么Bean使用`ResourceLoader` 接口来加载资源可能是有意义的。例如，考虑加载某种模板，其中所需的特定资源取决于用户的角色。如果资源是静态的，则有必要完全消除对`ResourceLoader` 接口的使用，让Bean公开所需的`Resource`属性，并期望将其注入其中。

注入这些属性的琐事是，所有应用程序上下文都注册并使用了特殊的JavaBeans `PropertyEditor`，可以将`String`路径转换为`Resource`对象。因此，如果`myBean`具有type的template属性`Resource`，则可以为该资源配置一个简单的字符串，如以下示例所示：

```xml
<bean id="myBean" class="...">
    <property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
```

请注意，资源路径没有前缀。因此，由于应用程序上下文本身将用作，因此根据上下文的确切类型，`ResourceLoader`资源本身通过a `ClassPathResource`，a `FileSystemResource`或a 加载 `ServletContextResource`。

如果需要强制使用特定`Resource`类型，则可以使用前缀。以下两个示例显示了如何强制a `ClassPathResource`和a `UrlResource`（后者用于访问文件系统文件）：

```xml
<property name="template" value="classpath:some/resource/path/myTemplate.txt">
```

```xml
<property name="template" value="file:///some/resource/path/myTemplate.txt"/>
```

### 2.7。应用程序上下文和资源路径

本节介绍如何使用资源创建应用程序上下文，包括使用XML的快捷方式，如何使用通配符以及其他详细信息。

#### 2.7.1。构造应用程序上下文

应用程序上下文构造函数（针对特定的应用程序上下文类型）通常采用字符串或字符串数组作为资源的位置路径，例如构成上下文定义的XML文件。

当这样的位置路径没有前缀时，`Resource`从该路径构建并用于加载Bean定义的特定类型取决于特定应用程序上下文，并且适合于该特定应用程序上下文。例如，请考虑以下示例，该示例创建一个 `ClassPathXmlApplicationContext`：

爪哇

科特林

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("conf/appContext.xml");
```

Bean定义是从类路径加载的，因为使用了a `ClassPathResource`。但是，请考虑以下示例，该示例创建一个`FileSystemXmlApplicationContext`：

爪哇

科特林

```java
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("conf/appContext.xml");
```

现在，bean定义是从文件系统位置（在这种情况下，是相对于当前工作目录）加载的。

请注意，在位置路径上使用特殊的classpath前缀或标准URL前缀会覆盖`Resource`为加载定义而创建的默认类型。考虑以下示例：

爪哇

科特林

```java
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("classpath:conf/appContext.xml");
```

使用`FileSystemXmlApplicationContext`从类路径加载bean定义。但是，它仍然是 `FileSystemXmlApplicationContext`。如果随后将其用作`ResourceLoader`，则所有未前缀的路径仍将视为文件系统路径。

##### 构造`ClassPathXmlApplicationContext`实例-快捷方式

在`ClassPathXmlApplicationContext`提供了多种构造方法以便于实例。基本思想是，您只能提供一个字符串数组，其中仅包含XML文件本身的文件名（不包含前导路径信息），还提供一个`Class`。所述`ClassPathXmlApplicationContext` 然后导出从提供类的路径信息。

请考虑以下目录布局：

```
com /
  foo /
    services.xml
    daos.xml
    MessengerService.class
```

以下示例显示了如何`ClassPathXmlApplicationContext`实例化一个由实例化的实例，该实例由在文件named `services.xml`和`daos.xml`（在类路径中）定义的bean组成：

爪哇

科特林

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext(
    new String[] {"services.xml", "daos.xml"}, MessengerService.class);
```

有关[`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/jca/context/SpringContextResourceAdapter.html) 各种构造函数的详细信息，请参见javadoc。

#### 2.7.2。应用程序上下文构造函数资源路径中的通配符

应用程序上下文构造函数值中的资源路径可以是简单路径（如前所述），每个路径都具有到目标的一对一映射`Resource`，或者可以包含特殊的“ classpath *：”前缀或内部Ant-样式正则表达式（使用Spring的`PathMatcher`实用程序进行匹配）。后者都是有效的通配符。

这种机制的一种用途是当您需要进行组件样式的应用程序组装时。所有组件都可以将上下文定义片段“发布”到一个众所周知的位置路径，并且，当使用前缀为的相同路径创建最终应用程序上下文时 `classpath*:`，所有组件片段都会被自动提取。

请注意，此通配符特定于在应用程序上下文构造函数中使用资源路径（或当您`PathMatcher`直接使用实用程序类层次结构时），并在构造时解决。它与`Resource`类型本身无关。您不能使用`classpath*:`前缀构造一个real `Resource`，因为资源一次仅指向一个资源。

##### 蚂蚁风格的图案

路径位置可以包含Ant样式的模式，如以下示例所示：

```
/WEB-INF/*-context.xml
com / mycompany / ** / applicationContext.xml
文件：C：/ some / path / *-context.xml
classpath：com / mycompany / ** / applicationContext.xml
```

当路径位置包含Ant样式的模式时，解析程序将遵循更复杂的过程来尝试解析通配符。它`Resource`为到达最后一个非通配符段的路径生成一个，并从中获取一个URL。如果此URL不是`jar:`URL或特定`zip:`于容器的变体（例如，在WebLogic中，`wsjar`在WebSphere中，等等），`java.io.File`则从中获取a 并将其用于遍历文件系统来解析通配符。对于jar URL，解析器可以从中获取a `java.net.JarURLConnection`或手动解析jar URL，然后遍历jar文件的内容以解析通配符。

###### 对可移植性的影响

如果指定的路径已经是文件URL（由于基路径是一个文件`ResourceLoader`系统，所以它是隐 式的，或者是显式的），则保证通配符可以完全可移植的方式工作。

如果指定的路径是类路径位置，则解析器必须通过`Classloader.getResource()`调用来获取最后一个非通配符路径段URL 。由于这只是路径的一个节点（而不是末尾的文件），因此实际上（在`ClassLoader`javadoc中）未定义 确切返回的是哪种URL。实际上，它总是`java.io.File`代表目录（类路径资源解析为文件系统位置）或某种jar URL（类路径资源解析为jar位置）。尽管如此，此操作仍存在可移植性问题。

如果为最后一个非通配符段获取了jar URL，则解析程序必须能够从中获取a `java.net.JarURLConnection`或手动解析jar URL，以便能够遍历jar的内容并解析通配符。这在大多数环境中确实有效，但在其他环境中则无效，因此我们强烈建议您在依赖特定环境之前，对来自jars的资源的通配符解析进行彻底测试。

##### 该`classpath*:`前缀

构造基于XML的应用程序上下文时，位置字符串可以使用特殊`classpath*:`前缀，如以下示例所示：

爪哇

科特林

```java
ApplicationContext ctx =
    new ClassPathXmlApplicationContext("classpath*:conf/appContext.xml");
```

这个特殊的前缀指定必须获取与给定名称匹配的所有类路径资源（内部，这实际上是通过调用来实现 `ClassLoader.getResources(…)`），然后合并以形成最终的应用程序上下文定义。

|      | 通配符类路径依赖于`getResources()`基础类加载器的方法。由于当今大多数应用程序服务器都提供自己的类加载器实现，因此行为可能有所不同，尤其是在处理jar文件时。一种简单的检查是否可行的测试`classpath*`是使用类加载器从classpath：的jar中加载文件 `getClass().getClassLoader().getResources("")`。尝试对具有相同名称但位于两个不同位置的文件进行此测试。如果返回了不合适的结果，请检查应用程序服务器文档中可能影响类加载器行为的设置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以在其余的位置路径（例如，）中将`classpath*:`前缀与`PathMatcher`模式结合使用`classpath*:META-INF/*-beans.xml`。在这种情况下，解析策略非常简单：`ClassLoader.getResources()`在最后一个非通配符路径段上使用一个调用，以获取类加载器层次结构中的所有匹配资源，然后在每个资源之外，使用与`PathMatcher`前面所述相同的解析策略来进行处理。通配符子路径。

##### 有关通配符的其他说明

请注意，`classpath*:`与Ant样式的模式结合使用时，除非模式文件实际存在目标文件中，否则在模式启动之前，它至少必须与至少一个根目录可靠地配合使用。这意味着诸如之类的模式 `classpath*:*.xml`可能不会从jar文件的根目录检索文件，而只会从扩展目录的根目录检索文件。

Spring检索类路径条目的能力源自JDK的 `ClassLoader.getResources()`方法，该方法仅返回文件系统位置的空字符串（指示可能要搜索的根目录）。Spring还评估 `URLClassLoader`运行时配置和`java.class.path`jar文件中的清单，但这不能保证会导致可移植行为。

|      | 扫描类路径包需要在类路径中存在相应的目录条目。使用Ant构建JAR时，请勿激活JAR任务的仅文件开关。同样，在某些环境中，基于安全策略，可能不会暴露类路径目录，例如，在JDK 1.7.0_45及更高版本上的独立应用程序（这需要在清单中设置“受信任的库”。请参阅 [https：/ /stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources](https://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources)）。在JDK 9的模块路径（Jigsaw）上，Spring的类路径扫描通常可以按预期进行。强烈建议在此处将资源放入专用目录，以避免在搜索jar文件根目录级别时出现上述可移植性问题。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`classpath:`如果要搜索的根包在多个类路径位置可用，则不能保证带有资源的蚂蚁样式模式找到匹配的资源。考虑以下资源位置示例：

```
com / mycompany / package1 / service-context.xml
```

现在考虑某人可能用来尝试找到该文件的Ant样式的路径：

```
classpath：com / mycompany / ** / service-context.xml
```

这样的资源可能只在一个位置，但是当使用诸如前面示例的路径尝试解析它时，解析器将根据返回的（第一个）URL工作 `getResource("com/mycompany");`。如果此基本包节点存在于多个类加载器位置，则实际的最终资源可能不存在。因此，在这种情况下，您应该首选使用`classpath*:`相同的Ant样式模式，该模式将搜索包含根包的所有类路径位置。

#### 2.7.3。`FileSystemResource`注意事项

一个`FileSystemResource`未连接到`FileSystemApplicationContext`（也就是，当一个`FileSystemApplicationContext`不实际的`ResourceLoader`）治疗的绝对和相对路径，如你所愿。相对路径是相对于当前工作目录的，而绝对路径是相对于文件系统的根的。

但是，出于向后兼容性（历史）的原因，当`FileSystemApplicationContext`是时，情况会发生变化 `ResourceLoader`。该 `FileSystemApplicationContext`部队所有连接的`FileSystemResource`情况下，把所有的位置路径为相对的，他们是否具有领先的斜线或无法启动。实际上，这意味着以下示例是等效的：

爪哇

科特林

```java
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("conf/context.xml");
```

爪哇

科特林

```java
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("/conf/context.xml");
```

以下示例也是等效的（即使它们有所不同也有意义，因为一种情况是相对的，另一种情况是绝对的）：

爪哇

科特林

```java
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("some/resource/path/myTemplate.txt");
```

爪哇

科特林

```java
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("/some/resource/path/myTemplate.txt");
```

实际上，如果需要真正的绝对文件系统路径，则应避免将绝对路径与`FileSystemResource`或一起`FileSystemXmlApplicationContext`使用，而应`UrlResource`通过使用`file:`URL前缀来强制使用a 。以下示例显示了如何执行此操作：

爪哇

科特林

```java
// actual context type doesn't matter, the Resource will always be UrlResource
ctx.getResource("file:///some/resource/path/myTemplate.txt");
```

爪哇

科特林

```java
// force this FileSystemXmlApplicationContext to load its definition via a UrlResource
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("file:///conf/context.xml");
```

## 3.验证，数据绑定和类型转换

考虑将验证作为业务逻辑是有利有弊，Spring提供了一种验证（和数据绑定）设计，但并不排除其中任何一个。具体来说，验证不应与网络层绑定，并且应该易于本地化，并且应该可以插入任何可用的验证器。考虑到这些问题，Spring提供了一个`Validator`基本的合同，并且该合同在应用程序的每一层都非常有用。

数据绑定对于使用户输入动态绑定到应用程序的域模型（或用于处理用户输入的任何对象）非常有用。Spring提供了恰如其分的名称`DataBinder`来做到这一点。在`Validator`和 `DataBinder`补`validation`包，它主要在使用，但不限于网层。

该`BeanWrapper`是Spring框架中的基本概念，并在很多地方被使用。但是，您可能不需要`BeanWrapper` 直接使用。但是，因为这是参考文档，所以我们认为可能需要进行一些解释。我们将`BeanWrapper`在本章中进行说明，因为如果您要使用它，那么在尝试将数据绑定到对象时很可能会这样做。

Spring的`DataBinder`和较低级别的`BeanWrapper`都使用`PropertyEditorSupport` 实现来解析和格式化属性值。该`PropertyEditor`和 `PropertyEditorSupport`类型是JavaBean规范的一部分，这一章中也有介绍。Spring 3引入了一个`core.convert`提供通用类型转换功能的软件包，以及一个用于格式化UI字段值的高级“ format”软件包。您可以将这些软件包用作`PropertyEditorSupport`实现的更简单替代 方案。本章还将对它们进行讨论。

Spring通过安装基础结构和Spring自己的`Validator`合同的适配器来支持Java Bean验证。应用程序可以全局启用一次Bean验证，如[Java Bean验证中所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#validation-beanvalidation)，并将其专用于所有验证需求。在网层，应用程序可以进一步寄存器控制器本地弹簧 `Validator`每实例`DataBinder`，如在描述的[配置`DataBinder`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#validation-binder)，其可用于在定制验证逻辑堵塞是有用的。

### 3.1。使用Spring的Validator接口进行验证

Spring具有一个`Validator`可用于验证对象的界面。该 `Validator`接口通过使用`Errors`对象来工作，以便验证器在验证时可以向`Errors`对象报告验证失败。

考虑以下小数据对象的示例：

爪哇

科特林

```java
public class Person {

    private String name;
    private int age;

    // the usual getters and setters...
}
```

下一个示例`Person`通过实现`org.springframework.validation.Validator`接口的以下两种方法来提供类的验证行为：

- `supports(Class)`：这可以`Validator`验证所提供实例`Class`吗？
- `validate(Object, org.springframework.validation.Errors)`：验证给定对象，并在出现验证错误的情况下，向给定`Errors`对象注册那些对象。

实现a `Validator`非常简单，尤其是当您知道`ValidationUtils`Spring Framework也提供的 helper类时。以下示例`Validator`为`Person`实例实现：

爪哇

科特林

```java
public class PersonValidator implements Validator {

    /**
     * This Validator validates only Person instances
     */
    public boolean supports(Class clazz) {
        return Person.class.equals(clazz);
    }

    public void validate(Object obj, Errors e) {
        ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
        Person p = (Person) obj;
        if (p.getAge() < 0) {
            e.rejectValue("age", "negativevalue");
        } else if (p.getAge() > 110) {
            e.rejectValue("age", "too.darn.old");
        }
    }
}
```

在`static` `rejectIfEmpty(..)`对方法`ValidationUtils`类用于拒绝该`name`属性，如果它是`null`或空字符串。看看 [`ValidationUtils`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/validation/ValidationUtils.html)javadoc，看看它提供了除了先前显示的示例之外的什么功能。

虽然可以实现单个`Validator`类来验证丰富对象中的每个嵌套对象，但最好将对象的每个嵌套类的验证逻辑封装在自己的`Validator`实现中。一个“丰富”对象的简单示例是一个`Customer`由两个`String` 属性（第一个和第二个名称）和一个复杂`Address`对象组成的对象。`Address`对象可以独立于`Customer`对象使用，因此`AddressValidator` 已实现了不同的对象。如果您想`CustomerValidator`重用`AddressValidator`该类中包含的逻辑而不求助于复制和粘贴，则可以`AddressValidator`在您的中依赖注入或实例化一个`CustomerValidator`，如以下示例所示：

爪哇

科特林

```java
public class CustomerValidator implements Validator {

    private final Validator addressValidator;

    public CustomerValidator(Validator addressValidator) {
        if (addressValidator == null) {
            throw new IllegalArgumentException("The supplied [Validator] is " +
                "required and must not be null.");
        }
        if (!addressValidator.supports(Address.class)) {
            throw new IllegalArgumentException("The supplied [Validator] must " +
                "support the validation of [Address] instances.");
        }
        this.addressValidator = addressValidator;
    }

    /**
     * This Validator validates Customer instances, and any subclasses of Customer too
     */
    public boolean supports(Class clazz) {
        return Customer.class.isAssignableFrom(clazz);
    }

    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "field.required");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "surname", "field.required");
        Customer customer = (Customer) target;
        try {
            errors.pushNestedPath("address");
            ValidationUtils.invokeValidator(this.addressValidator, customer.getAddress(), errors);
        } finally {
            errors.popNestedPath();
        }
    }
}
```

验证错误会报告给`Errors`传递给验证器的对象。对于Spring Web MVC，您可以使用``标签来检查错误消息，但是您也可以`Errors`自己检查对象。关于它提供的方法的更多信息可以在[javadoc中](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframeworkvalidation/Errors.html)找到。

### 3.2。将代码解析为错误消息

我们介绍了数据绑定和验证。本节介绍与验证错误相对应的输出消息。在上[一节中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#validator)显示的示例中，我们拒绝了`name`and `age`字段。如果我们想使用来输出错误消息 `MessageSource`，我们可以使用拒绝字段时提供的错误代码（在这种情况下为'name'和'age'）来实现。当您从接口调用（直接或间接地，例如通过使用`ValidationUtils`类）`rejectValue`或其他`reject`方法之一时`Errors`，基础实现不仅会注册您传入的代码，还会注册许多其他错误代码。在`MessageCodesResolver` 确定该错误代码的`Errors`接口寄存器。默认情况下， `DefaultMessageCodesResolver`使用（例如），它不仅使用您提供的代码注册一条消息，而且还注册包含您传递给reject方法的字段名称的消息。因此，如果您通过使用拒绝字段`rejectValue("age", "too.darn.old")`，则除了`too.darn.old`代码外，Spring还将注册`too.darn.old.age`和 `too.darn.old.age.int`（第一个包含字段名称，第二个包含字段类型）。这样做是为了方便开发人员在定位错误消息时提供帮助。

关于更多信息`MessageCodesResolver`以及默认的策略可以在javadoc的发现 [`MessageCodesResolver`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/validation/MessageCodesResolver.html)和 [`DefaultMessageCodesResolver`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.html)分别。

### 3.3。Bean操作和`BeanWrapper`

该`org.springframework.beans`软件包符合JavaBeans标准。JavaBean是具有默认无参数构造函数的类，并且遵循命名约定，在命名约定中，例如，名为的属性`bingoMadness`将具有setter方法`setBingoMadness(..)`和getter方法`getBingoMadness()`。有关JavaBean和规范的更多信息，请参见 [javabeans](https://docs.oracle.com/javase/8/docs/api/java/beans/package-summary.html)。

bean软件包中一个非常重要的类是`BeanWrapper`接口及其相应的实现（`BeanWrapperImpl`）。正如从Javadoc引用的那样，这些 `BeanWrapper`功能提供了设置和获取属性值（单独或批量），获取属性描述符以及查询属性以确定它们是否可读或可写的功能。此外，`BeanWrapper`还提供对嵌套属性的支持，从而可以将子属性上的属性设置为无限深度。该 `BeanWrapper`还支持标准JavaBean的能力`PropertyChangeListeners` 和`VetoableChangeListeners`，而不需要在辅助代码。最后但并非最不重要的一点是，它`BeanWrapper`提供了对设置索引属性的支持。在`BeanWrapper`通常不使用直接应用的代码，但使用由 `DataBinder`和所述`BeanFactory`。

`BeanWrapper`它的名称部分表示其工作方式：它包装一个bean以对该bean执行操作，例如设置和检索属性。

#### 3.3.1。设置和获取基本和嵌套属性

设置和获取属性是通过的`setPropertyValue`和 `getPropertyValue`重载方法变体完成的`BeanWrapper`。有关详细信息，请参见其Javadoc。下表显示了这些约定的一些示例：

| 表达                   | 说明                                                         |
| :--------------------- | :----------------------------------------------------------- |
| `name`                 | 指示与或 和方法`name`相对应的属性。`getName()``isName()``setName(..)` |
| `account.name`         | 表示与或方法相对应`name`的属性的嵌套属性。`account``getAccount().setName()``getAccount().getName()` |
| `account[2]`           | 指示索引属性的*第三个*元素`account`。索引属性可能是的`array`，`list`或其它天然有序集合。 |
| `account[COMPANYNAME]` | 指示由 属性的`COMPANYNAME`键索引的地图条目的值`account` `Map`。 |

（如果您不打算`BeanWrapper`直接使用Direct，则下一部分对您而言并不重要。如果仅使用`DataBinder`和`BeanFactory` 和及其默认实现，则应跳至上的 [部分`PropertyEditors`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-beans-conversion)。）

以下两个示例类使用的`BeanWrapper`get和set属性：

爪哇

科特林

```java
public class Company {

    private String name;
    private Employee managingDirector;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Employee getManagingDirector() {
        return this.managingDirector;
    }

    public void setManagingDirector(Employee managingDirector) {
        this.managingDirector = managingDirector;
    }
}
```

爪哇

科特林

```java
public class Employee {

    private String name;

    private float salary;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public float getSalary() {
        return salary;
    }

    public void setSalary(float salary) {
        this.salary = salary;
    }
}
```

下面的代码片断展示了如何检索和操作的一些实例属性的一些例子`Companies`和`Employees`：

爪哇

科特林

```java
BeanWrapper company = new BeanWrapperImpl(new Company());
// setting the company name..
company.setPropertyValue("name", "Some Company Inc.");
// ... can also be done like this:
PropertyValue value = new PropertyValue("name", "Some Company Inc.");
company.setPropertyValue(value);

// ok, let's create the director and tie it to the company:
BeanWrapper jim = new BeanWrapperImpl(new Employee());
jim.setPropertyValue("name", "Jim Stravinsky");
company.setPropertyValue("managingDirector", jim.getWrappedInstance());

// retrieving the salary of the managingDirector through the company
Float salary = (Float) company.getPropertyValue("managingDirector.salary");
```

#### 3.3.2。内置`PropertyEditor`实现

Spring使用a的概念`PropertyEditor`来实现an `Object`和a 之间的转换 `String`。以不同于对象本身的方式表示属性可能很方便。例如，`Date` 可以在人类可读的方式（如代表`String`：`'2007-14-09'`），同时我们还可以将人类可读的形式返回到原来的日期（或者甚至更好，转换成一个人类可读的形式返回到输入的任何日期`Date`的对象）。通过注册type为的自定义编辑器，可以实现此行为 `java.beans.PropertyEditor`。`BeanWrapper`在特定的IoC容器中（或在特定的IoC容器中）注册自定义编辑器（如上一章所述），使它具有如何将属性转换为所需类型的知识。欲了解更多有关 `PropertyEditor`，请参阅[的javadoc的`java.beans`Oracle的软件包](https://docs.oracle.com/javase/8/docs/api/java/beans/package-summary.html)。

在Spring中使用属性编辑的两个示例：

- 通过使用`PropertyEditor`实现来设置bean的属性。当您`String`用作在XML文件中声明的某些bean的属性的值时，Spring（如果相应属性的设置器具有`Class` 参数）将`ClassEditor`尝试将参数解析为`Class`对象。
- 在Spring的MVC框架中，解析HTTP请求参数的方法是使用各种`PropertyEditor`实现，您可以在的所有子类中手动绑定这些实现 `CommandController`。

Spring具有许多内置`PropertyEditor`实现，以简化生活。它们都位于`org.springframework.beans.propertyeditors` 包装中。默认情况下，大多数（但不是全部，如下表所示）是由注册的 `BeanWrapperImpl`。如果可以通过某种方式配置属性编辑器，则仍可以注册自己的变体以覆盖默认变体。下表描述了`PropertyEditor`Spring提供的各种实现：

| 类                        | 说明                                                         |
| :------------------------ | :----------------------------------------------------------- |
| `ByteArrayPropertyEditor` | 字节数组的编辑器。将字符串转换为其相应的字节表示形式。默认注册为`BeanWrapperImpl`。 |
| `ClassEditor`             | 将代表类的字符串解析为实际类，反之亦然。当找不到一个类时，将`IllegalArgumentException`抛出一个。默认情况下，由注册 `BeanWrapperImpl`。 |
| `CustomBooleanEditor`     | 属性的可定制属性编辑器`Boolean`。默认情况下，由注册， `BeanWrapperImpl`但是可以通过将其自定义实例注册为自定义编辑器来覆盖。 |
| `CustomCollectionEditor`  | 集合的属性编辑器，可将任何源`Collection`转换为给定的目标 `Collection`类型。 |
| `CustomDateEditor`        | 可定制的属性编辑器`java.util.Date`，支持自定义`DateFormat`。默认未注册。必须根据需要以适当的格式进行用户注册。 |
| `CustomNumberEditor`      | 定制的属性编辑器`Number`的子类，如`Integer`，`Long`，`Float`，或 `Double`。默认情况下，由注册，`BeanWrapperImpl`但是可以通过将其自定义实例注册为自定义编辑器来覆盖。 |
| `FileEditor`              | 将字符串解析为`java.io.File`对象。默认情况下，由注册 `BeanWrapperImpl`。 |
| `InputStreamEditor`       | 单向属性编辑器，它可以采用字符串并产生（通过中间`ResourceEditor`和`Resource`）an，`InputStream`以便`InputStream` 可以将属性直接设置为字符串。请注意，默认用法不会`InputStream`为您关闭。默认情况下，由注册`BeanWrapperImpl`。 |
| `LocaleEditor`            | 可以将字符串解析为`Locale`对象，反之亦然（字符串格式为 `*[country]*[variant]`，与的`toString()`方法 相同`Locale`）。默认情况下，由注册`BeanWrapperImpl`。 |
| `PatternEditor`           | 可以将字符串解析为`java.util.regex.Pattern`对象，反之亦然。  |
| `PropertiesEditor`        | 可以将字符串（以`java.util.Properties`类的javadoc中定义的格式格式化 ）转换为`Properties`对象。默认情况下，由注册`BeanWrapperImpl`。 |
| `StringTrimmerEditor`     | 修剪字符串的属性编辑器。（可选）允许将空字符串转换为`null`值。默认情况下未注册-必须是用户注册的。 |
| `URLEditor`               | 可以将URL的字符串表示形式解析为实际`URL`对象。默认情况下，由注册`BeanWrapperImpl`。 |

Spring使用`java.beans.PropertyEditorManager`来设置可能需要的属性编辑器的搜索路径。搜索路径还包括`sun.bean.editors`，其包括`PropertyEditor`实现为类型，例如`Font`，`Color`和最原始类型。还要注意，如果标准JavaBeans基础结构`PropertyEditor`与所处理的类位于同一包中并且与该类具有相同的名称，并且带有`Editor`附加名称，则标准JavaBeans基础结构会自动发现这些类（无需显式注册它们）。例如，可以具有以下类和包结构，这足以使`SomethingEditor`该类被识别并用作`PropertyEditor`for- `Something`typed属性。

```
com
  CHANK
    流行音乐
      东西
      SomethingEditor // Something类的PropertyEditor
```

请注意，您也可以使用标准的`BeanInfo`JavaBeans的机制在这里也描述（在一定程度上 [在这里](https://docs.oracle.com/javase/tutorial/javabeans/advanced/customization.html)）。以下示例使用该`BeanInfo`机制向`PropertyEditor`关联类的属性显式注册一个或多个实例：

```
com
  CHANK
    流行音乐
      东西
      SomethingBeanInfo // Something类的BeanInfo
```

所引用`SomethingBeanInfo`类的以下Java源代码将a `CustomNumberEditor`与该类的`age`属性相关联`Something`：

爪哇

科特林

```java
public class SomethingBeanInfo extends SimpleBeanInfo {

    public PropertyDescriptor[] getPropertyDescriptors() {
        try {
            final PropertyEditor numberPE = new CustomNumberEditor(Integer.class, true);
            PropertyDescriptor ageDescriptor = new PropertyDescriptor("age", Something.class) {
                public PropertyEditor createPropertyEditor(Object bean) {
                    return numberPE;
                };
            };
            return new PropertyDescriptor[] { ageDescriptor };
        }
        catch (IntrospectionException ex) {
            throw new Error(ex.toString());
        }
    }
}
```

##### 注册其他自定义`PropertyEditor`实现

当将bean属性设置为字符串值时，Spring IoC容器最终会使用标准JavaBeans `PropertyEditor`实现将这些字符串转换为属性的复杂类型。Spring预注册了许多自定义`PropertyEditor`实现（例如，将表示为字符串的类名转换为`Class`对象）。另外，Java的标准JavaBeans `PropertyEditor`查找机制允许`PropertyEditor` 为一个类适当地命名，并与它提供支持的类放在同一包中，以便可以自动找到它。

如果需要注册其他custom `PropertyEditors`，则可以使用几种机制。假设您有参考，最通常不方便或不建议使用的最手动方法是使用界面的`registerCustomEditor()`方法 。另一种（稍微方便些）的机制是使用称为的特殊bean工厂后处理器。尽管您可以在实现中使用Bean工厂后处理器，但是却具有嵌套的属性设置，因此我们强烈建议您将其与一起使用 ，在其中您可以将其以与其他任何Bean相似的方式进行部署，并且可以在其中自动检测和应用。`ConfigurableBeanFactory``BeanFactory``CustomEditorConfigurer``BeanFactory``CustomEditorConfigurer``ApplicationContext`

请注意，所有bean工厂和应用程序上下文通过使用a `BeanWrapper`来处理属性转换，都会自动使用许多内置的属性编辑器。[上一节](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-beans-conversion)`BeanWrapper` 中列出了寄存器的标准属性编辑器。此外，还重写或添加其他编辑器以适合于特定应用程序上下文类型的方式处理资源查找。`ApplicationContexts`

标准JavaBeans `PropertyEditor`实例用于将以字符串表示的属性值转换为属性的实际复杂类型。您可以使用 `CustomEditorConfigurer`Bean工厂后处理器来方便地将对其他`PropertyEditor`实例的支持添加到中`ApplicationContext`。

考虑以下示例，该示例定义了一个名为的用户类`ExoticType`和另一个名为的类`DependsOnExoticType`，需要`ExoticType`将其设置为属性：

爪哇

科特林

```java
package example;

public class ExoticType {

    private String name;

    public ExoticType(String name) {
        this.name = name;
    }
}

public class DependsOnExoticType {

    private ExoticType type;

    public void setType(ExoticType type) {
        this.type = type;
    }
}
```

正确设置之后，我们希望能够将type属性分配为字符串，然后`PropertyEditor`将a 转换为实际 `ExoticType`实例。以下bean定义显示了如何建立这种关系：

```xml
<bean id="sample" class="example.DependsOnExoticType">
    <property name="type" value="aNameForExoticType"/>
</bean>
```

该`PropertyEditor`实现可能类似于以下内容：

爪哇

科特林

```java
// converts string representation to ExoticType object
package example;

public class ExoticTypeEditor extends PropertyEditorSupport {

    public void setAsText(String text) {
        setValue(new ExoticType(text.toUpperCase()));
    }
}
```

最后，下面的例子说明了如何使用`CustomEditorConfigurer`注册新`PropertyEditor`用 `ApplicationContext`，然后就可以使用它作为需要：

```xml
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
    <property name="customEditors">
        <map>
            <entry key="example.ExoticType" value="example.ExoticTypeEditor"/>
        </map>
    </property>
</bean>
```

###### 使用 `PropertyEditorRegistrar`

向Spring容器注册属性编辑器的另一种机制是创建和使用`PropertyEditorRegistrar`。当需要在几种不同情况下使用同一组属性编辑器时，此接口特别有用。您可以编写相应的注册商，并在每种情况下重复使用它。 `PropertyEditorRegistrar`实例与称为 `PropertyEditorRegistry`的接口`BeanWrapper` （由Spring （和`DataBinder`）实现的接口）结合使用。`PropertyEditorRegistrar`实例与结合使用`CustomEditorConfigurer`（[在此](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-beans-conversion-customeditor-registration)描述 ）时特别方便，该实例公开了称为的属性`setPropertyEditorRegistrars(..)`。以这种方式`PropertyEditorRegistrar`添加到的实例`CustomEditorConfigurer`可以轻松与`DataBinder`和Spring MVC控制器。此外，它避免了在自定义编辑器上进行同步的需求：`PropertyEditorRegistrar`期望A `PropertyEditor` 为每次创建bean尝试创建新的实例。

以下示例显示了如何创建自己的`PropertyEditorRegistrar`实现：

爪哇

科特林

```java
package com.foo.editors.spring;

public final class CustomPropertyEditorRegistrar implements PropertyEditorRegistrar {

    public void registerCustomEditors(PropertyEditorRegistry registry) {

        // it is expected that new PropertyEditor instances are created
        registry.registerCustomEditor(ExoticType.class, new ExoticTypeEditor());

        // you could register as many custom property editors as are required here...
    }
}
```

另请参阅`org.springframework.beans.support.ResourceEditorRegistrar`示例 `PropertyEditorRegistrar`实现。请注意，在该`registerCustomEditors(..)`方法的实现中 ，它如何创建每个属性编辑器的新实例。

下一个示例显示如何配置a `CustomEditorConfigurer`并将其实例 `CustomPropertyEditorRegistrar`注入其中：

```xml
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
    <property name="propertyEditorRegistrars">
        <list>
            <ref bean="customPropertyEditorRegistrar"/>
        </list>
    </property>
</bean>

<bean id="customPropertyEditorRegistrar"
    class="com.foo.editors.spring.CustomPropertyEditorRegistrar"/>
```

最后（对于使用[Spring的MVC Web框架的人来说](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc)，`PropertyEditorRegistrars`与本章的重点有所偏离），结合使用数据绑定`Controllers`（例如`SimpleFormController`）可以非常方便。以下示例`PropertyEditorRegistrar`在`initBinder(..)`方法的实现中使用：

爪哇

科特林

```java
public final class RegisterUserController extends SimpleFormController {

    private final PropertyEditorRegistrar customPropertyEditorRegistrar;

    public RegisterUserController(PropertyEditorRegistrar propertyEditorRegistrar) {
        this.customPropertyEditorRegistrar = propertyEditorRegistrar;
    }

    protected void initBinder(HttpServletRequest request,
            ServletRequestDataBinder binder) throws Exception {
        this.customPropertyEditorRegistrar.registerCustomEditors(binder);
    }

    // other methods to do with registering a User
}
```

这种`PropertyEditor`注册样式可以导致代码简洁（实现的`initBinder(..)`长度只有一行），并且可以将常见的`PropertyEditor` 注册代码封装在一个类中，然后在`Controllers`所需的数量之间共享 。

### 3.4。弹簧类型转换

Spring 3引入了一个`core.convert`提供通用类型转换系统的软件包。该系统定义了一个用于实现类型转换逻辑的SPI和一个用于在运行时执行类型转换的API。在Spring容器中，您可以使用此系统作为`PropertyEditor`实现的替代方法，以将外部化的bean属性值字符串转换为所需的属性类型。您还可以在应用程序中需要类型转换的任何地方使用公共API。

#### 3.4.1。转换器SPI

如以下接口定义所示，用于实现类型转换逻辑的SPI非常简单且类型严格。

爪哇

科特林

```java
package org.springframework.core.convert.converter;

public interface Converter<S, T> {

    T convert(S source);
}
```

要创建自己的转换器，请实现该`Converter`接口并将其参数`S` 化为要转换`T`的类型和要转换为的类型。如果还`S`需要将一组`T`委托或数组转换为的数组或集合，则还可以透明地应用此类转换器，前提是还已注册了一个委托数组或集合转换器（`DefaultConversionService`默认情况下会这样做）。

对于每次调用`convert(S)`，保证源参数都不为空。`Converter`如果转换失败，您 可能会抛出任何未经检查的异常。具体来说，它应该抛出一个 `IllegalArgumentException`报告无效的源值。注意确保您的`Converter`实现是线程安全的。

`core.convert.support`为方便起见，包装中提供了几种转换器实现。这些包括从字符串到数字和其他常见类型的转换器。下面的清单显示了`StringToInteger`该类，这是一个典型的`Converter`实现：

爪哇

科特林

```java
package org.springframework.core.convert.support;

final class StringToInteger implements Converter<String, Integer> {

    public Integer convert(String source) {
        return Integer.valueOf(source);
    }
}
```

#### 3.4.2。使用`ConverterFactory`

当需要集中整个类层次结构的转换逻辑时（例如，从转换`String`为`Enum`对象时），可以实现 `ConverterFactory`，如以下示例所示：

爪哇

科特林

```java
package org.springframework.core.convert.converter;

public interface ConverterFactory<S, R> {

    <T extends R> Converter<S, T> getConverter(Class<T> targetType);
}
```

参数化S为您要从中转换的类型，参数化R为为基本类型，以定义可以转换为的类的*范围*。然后实现`getConverter(Class)`，其中T是R的子类。

考虑以下`StringToEnumConverterFactory`示例：

爪哇

```java
package org.springframework.core.convert.support;

final class StringToEnumConverterFactory implements ConverterFactory<String, Enum> {

    public <T extends Enum> Converter<String, T> getConverter(Class<T> targetType) {
        return new StringToEnumConverter(targetType);
    }

    private final class StringToEnumConverter<T extends Enum> implements Converter<String, T> {

        private Class<T> enumType;

        public StringToEnumConverter(Class<T> enumType) {
            this.enumType = enumType;
        }

        public T convert(String source) {
            return (T) Enum.valueOf(this.enumType, source.trim());
        }
    }
}
```

#### 3.4.3。使用`GenericConverter`

当您需要复杂的`Converter`实现时，请考虑使用 `GenericConverter`接口。与相比，它具有更灵活但类型不那么严格的签名`Converter`，`GenericConverter`支持在多种源和目标类型之间进行转换。另外，`GenericConverter`在实现转换逻辑时可以使用可用的源字段和目标字段上下文。这种上下文允许类型转换由字段注释或在字段签名上声明的通用信息驱动。以下清单显示了的接口定义`GenericConverter`：

爪哇

科特林

```java
package org.springframework.core.convert.converter;

public interface GenericConverter {

    public Set<ConvertiblePair> getConvertibleTypes();

    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
}
```

要实现`GenericConverter`，请`getConvertibleTypes()`返回支持的source→target类型对。然后实施`convert(Object, TypeDescriptor, TypeDescriptor)`以包含您的转换逻辑。源`TypeDescriptor`提供对包含要转换的值的源字段的访问。通过目标，`TypeDescriptor` 可以访问要设置转换值的目标字段。

一个很好的例子`GenericConverter`是在Java数组和集合之间进行转换的转换器。这样会`ArrayToCollectionConverter`检查声明目标集合类型的字段以解析集合的元素类型。这样就可以在将集合设置到目标字段上之前，将源数组中的每个元素转换为集合元素类型。

|      | 因为它`GenericConverter`是一个更复杂的SPI接口，所以仅应在需要时使用它。赞成`Converter`还是`ConverterFactory`出于基本类型转换的需要。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 使用 `ConditionalGenericConverter`

有时，您只希望在`Converter`满足特定条件的情况下运行。例如，您可能只想`Converter`在目标字段上存在特定注释时才运行，或者`Converter`只`static valueOf`在目标类上定义了特定方法（如方法）时才运行。 `ConditionalGenericConverter`是`GenericConverter`和`ConditionalConverter`接口的并集 ，可让您定义此类自定义匹配条件：

爪哇

科特林

```java
public interface ConditionalConverter {

    boolean matches(TypeDescriptor sourceType, TypeDescriptor targetType);
}

public interface ConditionalGenericConverter extends GenericConverter, ConditionalConverter {
}
```

a的一个很好的例子`ConditionalGenericConverter`是`EntityConverter`在持久性实体标识符和实体引用之间转换的。`EntityConverter` 仅当目标实体类型声明了静态查找器方法（例如）时，此类才可能匹配 `findAccount(Long)`。您可以在的实现中执行这种finder方法检查 `matches(TypeDescriptor, TypeDescriptor)`。

#### 3.4.4。该`ConversionService`API

`ConversionService`定义用于在运行时执行类型转换逻辑的统一API。转换器通常在以下外观界面后面执行：

爪哇

科特林

```java
package org.springframework.core.convert;

public interface ConversionService {

    boolean canConvert(Class<?> sourceType, Class<?> targetType);

    <T> T convert(Object source, Class<T> targetType);

    boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);

    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);

}
```

大多数`ConversionService`实现也实现`ConverterRegistry`，它为注册转换器提供SPI。在内部，一个`ConversionService` 实现委派其注册的转换器执行类型转换逻辑。

软件包中`ConversionService`提供了可靠的实现`core.convert.support`。`GenericConversionService`是适用于大多数环境的通用实现。`ConversionServiceFactory`为创建通用`ConversionService`配置提供了方便的工厂。

#### 3.4.5。配置一个`ConversionService`

A `ConversionService`是无状态对象，旨在在应用程序启动时实例化，然后在多个线程之间共享。在Spring应用程序中，通常需要`ConversionService`为每个Spring容器（或`ApplicationContext`）配置一个实例。`ConversionService`当框架需要执行类型转换时，Spring会选择并使用它。您也可以将其注入 `ConversionService`到任何bean中并直接调用它。

|      | 如果没有`ConversionService`向Spring注册，`PropertyEditor`则使用基于原始的系统。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要注册一个默认`ConversionService`使用Spring，用添加以下bean定义`id`的`conversionService`：

```xml
<bean id="conversionService"
    class="org.springframework.context.support.ConversionServiceFactoryBean"/>
```

默认值`ConversionService`可以在字符串，数字，枚举，集合，映射和其他常见类型之间转换。要用您自己的自定义转换器补充或覆盖默认转换器，请设置`converters`属性。属性值可以实现任何的`Converter`，`ConverterFactory`或者`GenericConverter`界面。

```xml
<bean id="conversionService"
        class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="example.MyCustomConverter"/>
        </set>
    </property>
</bean>
```

`ConversionService`在Spring MVC应用程序中使用也是很常见的。参见 Spring MVC一章中的[转换和格式化](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-conversion)。

在某些情况下，您可能希望在转换过程中应用格式设置。有关[使用`FormatterRegistry`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#format-FormatterRegistry-SPI)的详细信息，请参见 [SPI](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#format-FormatterRegistry-SPI)`FormattingConversionServiceFactoryBean`。

#### 3.4.6。以`ConversionService`编程方式使用

要以`ConversionService`编程方式使用实例，可以像对待其他任何bean一样注入对该实例的引用。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@Service
public class MyService {

    public MyService(ConversionService conversionService) {
        this.conversionService = conversionService;
    }

    public void doIt() {
        this.conversionService.convert(...)
    }
}
```

在大多数情况下，您可以使用`convert`指定的方法`targetType`，但不适用于更复杂的类型，例如参数化元素的集合。例如，如果你想转换`List`的`Integer`到`List`的`String`程序，您需要提供的源和目标类型的正式定义。

幸运的是，`TypeDescriptor`提供了各种选项来使操作变得简单明了，如以下示例所示：

爪哇

科特林

```java
DefaultConversionService cs = new DefaultConversionService();

List<Integer> input = ...
cs.convert(input,
    TypeDescriptor.forObject(input), // List<Integer> type descriptor
    TypeDescriptor.collection(List.class, TypeDescriptor.valueOf(String.class)));
```

请注意，将`DefaultConversionService`自动注册适用于大多数环境的转换器。这包括收集转换器，标量转换器和基本`Object`到`String`转换器。您可以`ConverterRegistry`使用类`addDefaultConverters` 上的静态方法将相同的转换器注册到任何转换器`DefaultConversionService`。

值类型转换器重新用于数组和集合，所以没有必要从创建特定的转换器转换`Collection`的`S`到 `Collection`的`T`，假设标准收集处理是适当的。

### 3.5。Spring字段格式

如上一节所述，[`core.convert`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#core-convert)是一种通用类型转换系统。它提供了统一的`ConversionService`API和强类型的`Converter`SPI，用于实现从一种类型到另一种类型的转换逻辑。Spring容器使用此系统绑定bean属性值。另外，Spring Expression Language（SpEL）和`DataBinder`使用此系统来绑定字段值。例如，当使用SpEL需要强迫一个`Short`到一个`Long`完成的`expression.setValue(Object bean, Object value)`尝试中，`core.convert` 系统执行的胁迫。

现在考虑典型客户端环境（例如Web或桌面应用程序）的类型转换要求。在这样的环境中，您通常会从转换`String` 为支持客户端回发过程，以及转换`String`为支持视图渲染过程。另外，您经常需要本地化`String`值。更为通用的`core.convert` `Converter`SPI不能直接解决此类格式化要求。为了直接解决这些问题，Spring 3引入了一个方便的`Formatter`SPI，它`PropertyEditor`为客户端环境的实现提供了一种简单而强大的替代方案。

通常，`Converter`当您需要实现通用类型转换逻辑时（例如，用于在a `java.util.Date`和a 之间进行转换），可以使用SPI `Long`。`Formatter`在客户端环境（例如Web应用程序）中工作并且需要解析和打印本地化的字段值时，可以使用SPI。在`ConversionService` 为双方提供的SPI统一类型转换API。

#### 3.5.1。该`Formatter`SPI

`Formatter`用于实现字段格式逻辑的SPI非常简单且类型严格。以下清单显示了`Formatter`接口定义：

爪哇

```java
package org.springframework.format;

public interface Formatter<T> extends Printer<T>, Parser<T> {
}
```

`Formatter`从`Printer`和`Parser`构件接口扩展。以下清单显示了这两个接口的定义：

爪哇

科特林

```java
public interface Printer<T> {

    String print(T fieldValue, Locale locale);
}
```

爪哇

科特林

```java
import java.text.ParseException;

public interface Parser<T> {

    T parse(String clientValue, Locale locale) throws ParseException;
}
```

要创建自己的`Formatter`，请实现`Formatter`前面显示的界面。参数`T`化为您希望格式化的对象的类型，例如 `java.util.Date`。实现`print()`操作以打印的实例`T`以在客户端语言环境中显示。实现`parse()`操作以`T`从客户端语言环境返回的格式化表示形式解析实例 。你`Formatter` 应该抛出一个`ParseException`或`IllegalArgumentException`如果在分析尝试失败。注意确保您的`Formatter`实现是线程安全的。

在`format`子包提供若干`Formatter`实施方式中，为方便起见。的`number`包提供`NumberStyleFormatter`，`CurrencyStyleFormatter`和 `PercentStyleFormatter`格式化`Number`对象的是使用一个`java.text.NumberFormat`。所述`datetime`包提供`DateFormatter`到格式`java.util.Date`与对象`java.text.DateFormat`。该`datetime.joda`软件包提供了基于[Joda-Time库的](https://www.joda.org/joda-time/)全面的日期时间格式支持。

以下`DateFormatter`是示例`Formatter`实现：

爪哇

科特林

```java
package org.springframework.format.datetime;

public final class DateFormatter implements Formatter<Date> {

    private String pattern;

    public DateFormatter(String pattern) {
        this.pattern = pattern;
    }

    public String print(Date date, Locale locale) {
        if (date == null) {
            return "";
        }
        return getDateFormat(locale).format(date);
    }

    public Date parse(String formatted, Locale locale) throws ParseException {
        if (formatted.length() == 0) {
            return null;
        }
        return getDateFormat(locale).parse(formatted);
    }

    protected DateFormat getDateFormat(Locale locale) {
        DateFormat dateFormat = new SimpleDateFormat(this.pattern, locale);
        dateFormat.setLenient(false);
        return dateFormat;
    }
}
```

Spring团队欢迎社区推动的`Formatter`贡献。请参阅 [GitHub问题](https://github.com/spring-projects/spring-framework/issues)以做出贡献。

#### 3.5.2。注释驱动的格式

可以通过字段类型或注释配置字段格式。要将注释绑定到`Formatter`，请实施`AnnotationFormatterFactory`。以下清单显示了`AnnotationFormatterFactory`接口的定义：

爪哇

科特林

```java
package org.springframework.format;

public interface AnnotationFormatterFactory<A extends Annotation> {

    Set<Class<?>> getFieldTypes();

    Printer<?> getPrinter(A annotation, Class<?> fieldType);

    Parser<?> getParser(A annotation, Class<?> fieldType);
}
```

要创建一个实现：。将A参数`annotationType`化为要与格式逻辑关联的字段，例如`org.springframework.format.annotation.DateTimeFormat`。。已`getFieldTypes()`返回可在其上使用注释的字段类型。。已经`getPrinter()`返回`Printer`到打印注释字段的值。。已`getParser()`返回一个`Parser`解析`clientValue`字段为一个带注释的字段。

以下示例`AnnotationFormatterFactory`实现将`@NumberFormat` 注释绑定到格式化程序，以指定数字样式或模式：

爪哇

科特林

```java
public final class NumberFormatAnnotationFormatterFactory
        implements AnnotationFormatterFactory<NumberFormat> {

    public Set<Class<?>> getFieldTypes() {
        return new HashSet<Class<?>>(asList(new Class<?>[] {
            Short.class, Integer.class, Long.class, Float.class,
            Double.class, BigDecimal.class, BigInteger.class }));
    }

    public Printer<Number> getPrinter(NumberFormat annotation, Class<?> fieldType) {
        return configureFormatterFrom(annotation, fieldType);
    }

    public Parser<Number> getParser(NumberFormat annotation, Class<?> fieldType) {
        return configureFormatterFrom(annotation, fieldType);
    }

    private Formatter<Number> configureFormatterFrom(NumberFormat annotation, Class<?> fieldType) {
        if (!annotation.pattern().isEmpty()) {
            return new NumberStyleFormatter(annotation.pattern());
        } else {
            Style style = annotation.style();
            if (style == Style.PERCENT) {
                return new PercentStyleFormatter();
            } else if (style == Style.CURRENCY) {
                return new CurrencyStyleFormatter();
            } else {
                return new NumberStyleFormatter();
            }
        }
    }
}
```

要触发格式，可以使用@NumberFormat注释字段，如以下示例所示：

爪哇

科特林

```java
public class MyModel {

    @NumberFormat(style=Style.CURRENCY)
    private BigDecimal decimal;
}
```

##### 格式注释API

`org.springframework.format.annotation` 软件包中存在可移植格式注释API 。您可以使用`@NumberFormat`格式化`Number`领域，如`Double`和 `Long`，并`@DateTimeFormat`以格式`java.util.Date`，`java.util.Calendar`，`Long` （为毫秒时间戳）以及JSR-310 `java.time`和乔达时间值类型。

以下示例用于`@DateTimeFormat`将a格式化`java.util.Date`为ISO日期（yyyy-MM-dd）：

爪哇

科特林

```java
public class MyModel {

    @DateTimeFormat(iso=ISO.DATE)
    private Date date;
}
```

#### 3.5.3。该`FormatterRegistry`SPI

的`FormatterRegistry`是用于登记格式化器和转换器的SPI。 `FormattingConversionService`是`FormatterRegistry`适合大多数环境的实现。您可以通过编程方式或声明方式将此变体配置为Spring bean，例如通过使用`FormattingConversionServiceFactoryBean`。由于此实现还实现`ConversionService`，因此您可以直接配置它以与Spring的`DataBinder`和Spring Expression Language（SpEL）一起使用。

以下清单显示了`FormatterRegistry`SPI：

爪哇

科特林

```java
package org.springframework.format;

public interface FormatterRegistry extends ConverterRegistry {

    void addFormatterForFieldType(Class<?> fieldType, Printer<?> printer, Parser<?> parser);

    void addFormatterForFieldType(Class<?> fieldType, Formatter<?> formatter);

    void addFormatterForFieldType(Formatter<?> formatter);

    void addFormatterForAnnotation(AnnotationFormatterFactory<?> factory);
}
```

如前面的清单所示，您可以按字段类型或注解注册格式化程序。

该`FormatterRegistry`SPI允许您配置格式规则集中，而不是在你的控制器重复这样的配置。例如，您可能需要强制以某种方式设置所有日期字段的格式或以某种方式设置带有特定批注的字段的格式。使用shared `FormatterRegistry`，您只需定义一次这些规则，并且在需要格式化时就会应用它们。

#### 3.5.4。该`FormatterRegistrar`SPI

`FormatterRegistrar`是用于通过FormatterRegistry注册格式器和转换器的SPI。以下清单显示了其接口定义：

爪哇

科特林

```java
package org.springframework.format;

public interface FormatterRegistrar {

    void registerFormatters(FormatterRegistry registry);
}
```

`FormatterRegistrar`当为给定的格式类别（例如日期格式）注册多个相关的转换器和格式器时，A 很有用。在声明式注册不足的情况下（例如，当格式化程序需要在与其自身不同的特定字段类型下建立索引时，``或者在注册一个`Printer`/ `Parser`对时），它也很有用。下一节将提供有关转换器和格式化程序注册的更多信息。

#### 3.5.5。在Spring MVC中配置格式

参见Spring MVC一章中的[转换和格式化](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-conversion)。

### 3.6。配置全局日期和时间格式

默认情况下，`@DateTimeFormat`使用`DateFormat.SHORT`样式从字符串转换未注释的日期和时间字段。如果愿意，可以通过定义自己的全局格式来更改此设置。

为此，请确保Spring不注册默认格式器。相反，可以借助以下方法手动注册格式器：

- `org.springframework.format.datetime.standard.DateTimeFormatterRegistrar`
- `org.springframework.format.datetime.DateFormatterRegistrar`或 `org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar`Joda-Time。

例如，以下Java配置注册了全局`yyyyMMdd`格式：

爪哇

科特林

```java
@Configuration
public class AppConfig {

    @Bean
    public FormattingConversionService conversionService() {

        // Use the DefaultFormattingConversionService but do not register defaults
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService(false);

        // Ensure @NumberFormat is still supported
        conversionService.addFormatterForFieldAnnotation(new NumberFormatAnnotationFormatterFactory());

        // Register JSR-310 date conversion with a specific global format
        DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
        registrar.setDateFormatter(DateTimeFormatter.ofPattern("yyyyMMdd"));
        registrar.registerFormatters(conversionService);

        // Register date conversion with a specific global format
        DateFormatterRegistrar registrar = new DateFormatterRegistrar();
        registrar.setFormatter(new DateFormatter("yyyyMMdd"));
        registrar.registerFormatters(conversionService);

        return conversionService;
    }
}
```

如果您喜欢基于XML的配置，则可以使用 `FormattingConversionServiceFactoryBean`。以下示例显示了如何执行此操作（这次使用Joda Time）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd>

    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="registerDefaultFormatters" value="false" />
        <property name="formatters">
            <set>
                <bean class="org.springframework.format.number.NumberFormatAnnotationFormatterFactory" />
            </set>
        </property>
        <property name="formatterRegistrars">
            <set>
                <bean class="org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar">
                    <property name="dateFormatter">
                        <bean class="org.springframework.format.datetime.joda.DateTimeFormatterFactoryBean">
                            <property name="pattern" value="yyyyMMdd"/>
                        </bean>
                    </property>
                </bean>
            </set>
        </property>
    </bean>
</beans>
```

请注意，在Web应用程序中配置日期和时间格式时，还有其他注意事项。请参阅 [WebMVC转换和格式](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-conversion)或 [WebFlux转换和格式](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-config-conversion)。

### 3.7。Java Bean验证

Spring框架提供了对 [Java Bean验证](https://beanvalidation.org/) API的支持。

#### 3.7.1。Bean验证概述

Bean验证为Java应用程序提供了通过约束声明和元数据进行验证的通用方法。要使用它，您需要使用声明性验证约束对域模型属性进行注释，然后由运行时强制实施。有内置的约束，您也可以定义自己的自定义约束。

考虑以下示例，该示例显示了`PersonForm`具有两个属性的简单模型：

爪哇

科特林

```java
public class PersonForm {
    private String name;
    private int age;
}
```

Bean验证使您可以声明约束，如以下示例所示：

爪哇

科特林

```java
public class PersonForm {

    @NotNull
    @Size(max=64)
    private String name;

    @Min(0)
    private int age;
}
```

然后，Bean验证验证器根据声明的约束来验证此类的实例。有关该API的一般信息，请参见[Bean验证](https://beanvalidation.org/)。有关特定限制，请参见[Hibernate Validator](https://hibernate.org/validator/)文档。要了解如何将bean验证提供程序设置为Spring bean，请继续阅读。

#### 3.7.2。配置Bean验证提供程序

Spring提供了对Bean验证API的全面支持，包括将Bean验证提供程序作为Spring Bean进行引导。这样，您就可以在应用程序中注入`javax.validation.ValidatorFactory`或`javax.validation.Validator`在需要验证的地方进行注入 。

您可以使用`LocalValidatorFactoryBean`来将默认的Validator配置为Spring Bean，如以下示例所示：

爪哇

XML格式

```java
import org.springframework.validation.beanvalidation.LocalValidatorFactoryBean;

@Configuration

public class AppConfig {

    @Bean
    public LocalValidatorFactoryBean validator() {
        return new LocalValidatorFactoryBean;
    }
}
```

前面示例中的基本配置触发Bean验证以使用其默认引导机制进行初始化。Bean验证提供程序（例如Hibernate Validator）应该出现在类路径中，并会自动检测到。

##### 注入验证器

`LocalValidatorFactoryBean`实现`javax.validation.ValidatorFactory`和 `javax.validation.Validator`，以及Spring的`org.springframework.validation.Validator`。您可以将对这两个接口之一的引用注入需要调用验证逻辑的bean中。

`javax.validation.Validator`如果希望直接使用Bean Validation API，则可以注入一个引用，如以下示例所示：

爪哇

科特林

```java
import javax.validation.Validator;

@Service
public class MyService {

    @Autowired
    private Validator validator;
}
```

您可以注入一个引用，以了解`org.springframework.validation.Validator`您的bean是否需要Spring Validation API，如以下示例所示：

爪哇

科特林

```java
import org.springframework.validation.Validator;

@Service
public class MyService {

    @Autowired
    private Validator validator;
}
```

##### 配置自定义约束

每个bean验证约束都包括两个部分：

- 一个`@Constraint`声明的约束和它的配置属性注释。
- `javax.validation.ConstraintValidator`接口的实现，用于实现约束的行为。

为了将声明与实现相关联，每个`@Constraint`注释都引用一个对应的`ConstraintValidator`实现类。在运行时，`ConstraintValidatorFactory`当在域模型中遇到约束注释时， 实例化引用的实现。

默认情况下，`LocalValidatorFactoryBean`configure `SpringConstraintValidatorFactory` 会使用Spring创建`ConstraintValidator`实例。这使您的自定义 `ConstraintValidators`可以像其他任何Spring bean一样从依赖项注入中受益。

以下示例显示了一个自定义`@Constraint`声明，其后是一个`ConstraintValidator`使用Spring进行依赖项注入的关联 实现：

爪哇

科特林

```java
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy=MyConstraintValidator.class)
public @interface MyConstraint {
}
```

爪哇

科特林

```java
import javax.validation.ConstraintValidator;

public class MyConstraintValidator implements ConstraintValidator {

    @Autowired;
    private Foo aDependency;

    // ...
}
```

如前面的示例所示，`ConstraintValidator`实现可以`@Autowired`像其他任何Spring bean一样具有其依赖项 。

##### 弹簧驱动方法验证

您可以通过Bean定义将Bean验证1.1（以及作为自定义扩展，还包括Hibernate Validator 4.3）支持的方法验证功能集成到Spring上下文中 `MethodValidationPostProcessor`：

爪哇

XML格式

```java
import org.springframework.validation.beanvalidation.MethodValidationPostProcessor;

@Configuration

public class AppConfig {

    @Bean
    public MethodValidationPostProcessor validationPostProcessor() {
        return new MethodValidationPostProcessor;
    }
}
```

为了有资格进行Spring驱动的方法验证，所有目标类都需要使用Spring的`@Validated`注释进行注释，该注释也可以选择声明要使用的验证组。有关[`MethodValidationPostProcessor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/validation/beanvalidation/MethodValidationPostProcessor.html) Hibernate Validator和Bean Validation 1.1提供程序的设置详细信息，请参见 。

|      | 方法验证依赖于目标类周围的[AOP代理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-introduction-proxies)，即接口上方法的JDK动态代理或CGLIB代理。代理的使用存在某些限制，《[理解AOP代理》](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-understanding-aop-proxies)中介绍了其中的一些限制 。另外，请记住在代理类上始终使用方法和访问器；直接现场访问将不起作用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 其他配置选项

在`LocalValidatorFactoryBean`大多数情况下，默认配置就足够了。从消息插值到遍历解析，有许多用于各种Bean验证构造的配置选项。有关[`LocalValidatorFactoryBean`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/validation/beanvalidation/LocalValidatorFactoryBean.html) 这些选项的更多信息，请参见 javadoc。

#### 3.7.3。配置一个`DataBinder`

从Spring 3开始，您可以使用配置`DataBinder`实例`Validator`。配置完成后，您可以`Validator`通过调用调用`binder.validate()`。任何验证都会 `Errors`自动添加到活页夹的中`BindingResult`。

以下示例显示了`DataBinder`绑定到目标对象后如何以编程方式使用来调用验证逻辑：

爪哇

科特林

```java
Foo target = new Foo();
DataBinder binder = new DataBinder(target);
binder.setValidator(new FooValidator());

// bind to the target object
binder.bind(propertyValues);

// validate the target object
binder.validate();

// get BindingResult that includes any validation errors
BindingResult results = binder.getBindingResult();
```

您还可以通过和配置`DataBinder`具有多个`Validator`实例的 。当将全局配置的bean验证与在DataBinder实例上本地配置的Spring结合使用时，这很有用。参见 [Spring MVC验证配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-validation)。`dataBinder.addValidators``dataBinder.replaceValidators``Validator`

#### 3.7.4。Spring MVC 3验证

参见Spring MVC一章中的[验证](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-validation)。

## 4.春季表达语言（SpEL）

Spring表达式语言（简称“ SpEL”）是一种功能强大的表达式语言，支持在运行时查询和操作对象图。语言语法类似于Unified EL，但是提供了其他功能，最著名的是方法调用和基本的字符串模板功能。

尽管还有其他几种Java表达式语言可用-OGNL，MVEL和JBoss EL，仅举几例，但Spring Expression Language的创建是为了向Spring社区提供一种受良好支持的表达式语言，该语言可用于该版本中的所有产品。春季投资组合。其语言功能由Spring产品组合中的项目要求所驱动，包括[Spring Tools for Eclipse中](https://spring.io/tools)代码完成支持的[工具要求](https://spring.io/tools)。也就是说，SpEL基于与技术无关的API，如有需要，该API可以集成其他表达语言实现。

尽管SpEL是Spring产品组合中表达评估的基础，但它并不直接与Spring绑定，而是可以独立使用。为了自成一体，本章中的许多示例都将SpEL当作一种独立的表达语言来使用。这需要创建一些自举基础结构类，例如解析器。大多数Spring用户不需要处理这种基础结构，而是只能编写表达式字符串进行评估。这种典型用法的一个示例是将SpEL集成到创建XML或基于注释的Bean定义中，如[Expression对定义Bean定义的支持](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-beandef)所示 。

本章介绍了表达语言，其API和语言语法的功能。在几个地方，`Inventor`和`Society`类为目标的表达评价对象使用。这些类声明以及用于填充它们的数据在本章末尾列出。

表达式语言支持以下功能：

- 文字表达
- 布尔运算符和关系运算符
- 常用表达
- 类表达式
- 访问属性，数组，列表和映射
- 方法调用
- 关系运算符
- 分配
- 调用构造函数
- Bean参考
- 阵列构造
- 内联列表
- 内联地图
- 三元运算符
- 变数
- 用户定义的功能
- 集合投影
- 馆藏选择
- 模板表达式

### 4.1。评价

本节介绍SpEL接口及其表达语言的简单用法。完整的语言参考可以在[语言参考中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-language-ref)找到 。

以下代码介绍了SpEL API来评估文字字符串表达式 `Hello World`。

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'"); 
String message = (String) exp.getValue();
```

|      | message变量的值为`'Hello World'`。 |
| ---- | ---------------------------------- |
|      |                                    |

您最可能使用的SpEL类和接口位于 `org.springframework.expression`包及其子包中，例如`spel.support`。

该`ExpressionParser`接口负责解析表达式字符串。在前面的示例中，表达式字符串是由周围的单引号表示的字符串文字。该`Expression`接口负责评估先前定义的表达式字符串。分别调用和时，可以引发`ParseException`和的 两个异常。`EvaluationException``parser.parseExpression``exp.getValue`

SpEL支持多种功能，例如调用方法，访问属性和调用构造函数。

在以下方法调用示例中，我们`concat`在字符串文字上调用该方法：

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')"); 
String message = (String) exp.getValue();
```

|      | 现在的值`message`是“ Hello World！”。 |
| ---- | ------------------------------------- |
|      |                                       |

以下调用JavaBean属性的示例将调用该`String`属性`Bytes`：

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();

// invokes 'getBytes()'
Expression exp = parser.parseExpression("'Hello World'.bytes"); 
byte[] bytes = (byte[]) exp.getValue();
```

|      | 这行将文字转换为字节数组。 |
| ---- | -------------------------- |
|      |                            |

SpEL还通过使用标准的点符号（例如`prop1.prop2.prop3`）以及相应的属性值设置来支持嵌套属性 。也可以访问公共字段。

下面的示例演示如何使用点表示法获取文字的长度：

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();

// invokes 'getBytes().length'
Expression exp = parser.parseExpression("'Hello World'.bytes.length"); 
int length = (Integer) exp.getValue();
```

|      | `'Hello World'.bytes.length` 给出文字的长度。 |
| ---- | --------------------------------------------- |
|      |                                               |

可以调用String的构造函数，而不是使用字符串文字，如以下示例所示：

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("new String('hello world').toUpperCase()"); 
String message = exp.getValue(String.class);
```

|      | `String`从文字构造一个新的，并使其大写。 |
| ---- | ---------------------------------------- |
|      |                                          |

请注意一般方法的使用：`public  T getValue(Class desiredResultType)`。使用此方法无需将表达式的值强制转换为所需的结果类型。一个`EvaluationException`如果该值不能转换到的类型抛出`T`或通过使用已注册的类型转换器转换。

SpEL的更常见用法是提供一个表达式字符串，该字符串针对特定的对象实例（称为根对象）进行评估。下面的示例演示如何`name`从`Inventor`类的实例中检索属性或创建布尔条件：

爪哇

科特林

```java
// Create and set a calendar
GregorianCalendar c = new GregorianCalendar();
c.set(1856, 7, 9);

// The constructor arguments are name, birthday, and nationality.
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();

Expression exp = parser.parseExpression("name"); // Parse name as an expression
String name = (String) exp.getValue(tesla);
// name == "Nikola Tesla"

exp = parser.parseExpression("name == 'Nikola Tesla'");
boolean result = exp.getValue(tesla, Boolean.class);
// result == true
```

#### 4.1.1。理解`EvaluationContext`

`EvaluationContext`在评估表达式以解析属性，方法或字段并帮助执行类型转换时，使用该接口。Spring提供了两种实现。

- `SimpleEvaluationContext`：针对不需要全部SpEL语言语法范围且应加以有意义限制的表达式类别，公开了SpEL基本语言功能和配置选项的子集。示例包括但不限于数据绑定表达式和基于属性的过滤器。
- `StandardEvaluationContext`：公开SpEL语言功能和配置选项的全部集合。您可以使用它来指定默认的根对象并配置每个可用的评估相关策略。

`SimpleEvaluationContext`设计为仅支持SpEL语言语法的子集。它不包括Java类型引用，构造函数和Bean引用。它还要求您明确选择对表达式中的属性和方法的支持级别。默认情况下，`create()`静态工厂方法仅启用对属性的读取访问。您还可以获取构建器来配置所需的确切支持级别，并针对以下一种或某些组合：

- `PropertyAccessor`仅自定义（无反射）
- 只读访问的数据绑定属性
- 读写的数据绑定属性

##### 类型转换

默认情况下，SpEL使用Spring core（`org.springframework.core.convert.ConversionService`）中可用的转换服务。此转换服务附带许多内置转换器，用于常见转换，但也可以完全扩展，以便您可以在类型之间添加自定义转换。此外，它是泛型感知的。这意味着，当您在表达式中使用泛型类型时，SpEL会尝试进行转换以维护遇到的任何对象的类型正确性。

在实践中这意味着什么？假设使用分配`setValue()`来设置`List`属性。该属性的类型实际上是`List`。SpEL认识到列表中的元素`Boolean`在放入列表之前需要先进行转换。以下示例显示了如何执行此操作：

爪哇

科特林

```java
class Simple {
    public List<Boolean> booleanList = new ArrayList<Boolean>();
}

Simple simple = new Simple();
simple.booleanList.add(true);

EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

// "false" is passed in here as a String. SpEL and the conversion service
// will recognize that it needs to be a Boolean and convert it accordingly.
parser.parseExpression("booleanList[0]").setValue(context, simple, "false");

// b is false
Boolean b = simple.booleanList.get(0);
```

#### 4.1.2。解析器配置

可以使用解析器配置对象（`org.springframework.expression.spel.SpelParserConfiguration`）配置SpEL表达式解析器。配置对象控制某些表达式组件的行为。例如，如果您索引到数组或集合中，并且指定索引处的元素为`null`，则可以自动创建该元素。当使用由属性引用链组成的表达式时，这很有用。如果您在数组或列表中建立索引并指定了超出数组或列表当前大小末尾的索引，则可以自动增长数组或列表以容纳该索引。下面的示例演示如何自动增加列表：

爪哇

科特林

```java
class Demo {
    public List<String> list;
}

// Turn on:
// - auto null reference initialization
// - auto collection growing
SpelParserConfiguration config = new SpelParserConfiguration(true,true);

ExpressionParser parser = new SpelExpressionParser(config);

Expression expression = parser.parseExpression("list[3]");

Demo demo = new Demo();

Object o = expression.getValue(demo);

// demo.list will now be a real collection of 4 entries
// Each entry is a new empty String
```

#### 4.1.3。SpEL编译

Spring Framework 4.1包含一个基本的表达式编译器。通常对表达式进行解释，这在评估过程中提供了很大的动态灵活性，但没有提供最佳性能。对于偶尔使用表达式，这很好，但是当与其他组件（如Spring Integration）一起使用时，性能可能非常重要，并且不需要动态性。

SpEL编译器旨在满足这一需求。在评估过程中，编译器生成一个Java类，该类体现了运行时的表达式行为，并使用该类来实现更快的表达式评估。由于缺少在表达式周围输入内容的信息，因此编译器在执行编译时会使用在表达式的解释求值过程中收集的信息。例如，它不仅仅从表达式中就知道属性引用的类型，而是在第一次解释求值时就知道它是什么。当然，如果各种表达元素的类型随时间变化，则基于此类派生信息进行编译会在以后引起麻烦。因此，编译最适合类型信息在重复求值时不会改变的表达式。

考虑以下基本表达式：

```
someArray [0] .someProperty.someOtherProperty <0.1
```

因为前面的表达式涉及数组访问，一些属性取消引用和数字运算，所以性能提升可能非常明显。在一个示例中，进行了50000次迭代的微基准测试，使用解释器评估需要75毫秒，而使用表达式的编译版本仅需要3毫秒。

##### 编译器配置

默认情况下不打开编译器，但是您可以通过两种不同的方式之一来打开它。当SpEL用法嵌入到另一个组件中时，可以使用解析器配置过程（[如前所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-parser-configuration)）或使用系统属性来打开它。本节讨论这两个选项。

编译器可以在`org.springframework.expression.spel.SpelCompilerMode`枚举中捕获的三种模式之一下运行 。模式如下：

- `OFF` （默认）：编译器已关闭。
- `IMMEDIATE`：在立即模式下，将尽快编译表达式。通常在第一次解释评估之后。如果编译的表达式失败（通常是由于类型更改，如前所述），则表达式求值的调用者将收到异常。
- `MIXED`：在混合模式下，表达式会随着时间静默在解释模式和编译模式之间切换。经过一定数量的解释运行后，它们将切换到编译形式，如果编译形式出了问题（例如，如前面所述的类型更改），则表达式会自动再次切换回解释形式。稍后，它可能会生成另一个已编译的表单并切换到该表单。基本上，用户进入`IMMEDIATE`模式的异常是在内部处理的。

`IMMEDIATE`之所以存在mode，是因为`MIXED`mode可能会导致具有副作用的表达式出现问题。如果已编译的表达式在部分成功后就崩溃了，则它可能已经完成了影响系统状态的操作。如果发生这种情况，调用者可能不希望它在解释模式下静默地重新运行，因为表达式的一部分可能运行了两次。

选择模式后，使用`SpelParserConfiguration`来配置解析器。以下示例显示了如何执行此操作：

爪哇

科特林

```java
SpelParserConfiguration config = new SpelParserConfiguration(SpelCompilerMode.IMMEDIATE,
    this.getClass().getClassLoader());

SpelExpressionParser parser = new SpelExpressionParser(config);

Expression expr = parser.parseExpression("payload");

MyMessage message = new MyMessage();

Object payload = expr.getValue(message);
```

当指定编译器模式时，还可以指定一个类加载器（允许传递null）。编译的表达式在提供的任何子类下创建的子类加载器中定义。重要的是要确保，如果指定了类加载器，则它可以查看表达式评估过程中涉及的所有类型。如果未指定类加载器，则使用默认的类加载器（通常是在表达式求值期间运行的线程的上下文类加载器）。

第二种配置编译器的方法是将SpEL嵌入到其他组件中，并且可能无法通过配置对象进行配置。在这些情况下，可以使用系统属性。您可以设置`spring.expression.compiler.mode` 属性为一个`SpelCompilerMode`枚举值（`off`，`immediate`，或`mixed`）。

##### 编译器限制

从Spring Framework 4.1开始，已经有了基本的编译框架。但是，该框架尚不支持编译每种表达式。最初的重点是可能在性能关键型上下文中使用的通用表达式。目前无法编译以下类型的表达式：

- 涉及赋值的表达
- 表达式依赖转换服务
- 使用自定义解析器或访问器的表达式
- 使用选择或投影的表达式

将来会编译更多类型的表达。

### 4.2。Bean定义中的表达式

您可以将SpEL表达式与基于XML或基于注释的配置元数据一起使用来定义`BeanDefinition`实例。在这两种情况下，用于定义表达式的语法均为形式`#{  }`。

#### 4.2.1。XML配置

可以使用表达式来设置属性或构造函数参数值，如以下示例所示：

```xml
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>

    <!-- other properties -->
</bean>
```

该`systemProperties`变量是预定义的，因此您可以在表达式中使用它，如以下示例所示：

```xml
<bean id="taxCalculator" class="org.spring.samples.TaxCalculator">
    <property name="defaultLocale" value="#{ systemProperties['user.region'] }"/>

    <!-- other properties -->
</bean>
```

请注意，`#` 在此上下文中，您不必在预定义变量之前添加符号。

您还可以按名称引用其他bean属性，如以下示例所示：

```xml
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>

    <!-- other properties -->
</bean>

<bean id="shapeGuess" class="org.spring.samples.ShapeGuess">
    <property name="initialShapeSeed" value="#{ numberGuess.randomNumber }"/>

    <!-- other properties -->
</bean>
```

#### 4.2.2。注释配置

要指定默认值，可以将`@Value`注释放在字段，方法以及方法或构造函数参数上。

以下示例设置字段变量的默认值：

爪哇

科特林

```java
public class FieldValueTestBean {

    @Value("#{ systemProperties['user.region'] }")
    private String defaultLocale;

    public void setDefaultLocale(String defaultLocale) {
        this.defaultLocale = defaultLocale;
    }

    public String getDefaultLocale() {
        return this.defaultLocale;
    }
}
```

下面的示例显示等效的但使用属性设置器方法的示例：

爪哇

科特林

```java
public class PropertyValueTestBean {

    private String defaultLocale;

    @Value("#{ systemProperties['user.region'] }")
    public void setDefaultLocale(String defaultLocale) {
        this.defaultLocale = defaultLocale;
    }

    public String getDefaultLocale() {
        return this.defaultLocale;
    }
}
```

自动装配的方法和构造函数也可以使用`@Value`注释，如以下示例所示：

爪哇

科特林

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;
    private String defaultLocale;

    @Autowired
    public void configure(MovieFinder movieFinder,
            @Value("#{ systemProperties['user.region'] }") String defaultLocale) {
        this.movieFinder = movieFinder;
        this.defaultLocale = defaultLocale;
    }

    // ...
}
```

爪哇

科特林

```java
public class MovieRecommender {

    private String defaultLocale;

    private CustomerPreferenceDao customerPreferenceDao;

    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao,
            @Value("#{systemProperties['user.country']}") String defaultLocale) {
        this.customerPreferenceDao = customerPreferenceDao;
        this.defaultLocale = defaultLocale;
    }

    // ...
}
```

### 4.3。语言参考

本节描述了Spring Expression Language的工作方式。它涵盖以下主题：

- [文字表达](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-ref-literal)
- [属性，数组，列表，映射和索引器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-properties-arrays)
- [内联列表](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-inline-lists)
- [内联地图](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-inline-maps)
- [阵列构造](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-array-construction)
- [方法](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-methods)
- [经营者](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-operators)
- [种类](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-types)
- [建设者](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-constructors)
- [变数](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-ref-variables)
- [功能](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-ref-functions)
- [Bean参考](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-bean-references)
- [三元运算符（If-Then-Else）](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-operator-ternary)
- [猫王算子](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-operator-elvis)
- [安全导航操作员](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-operator-safe-navigation)

#### 4.3.1。文字表达

支持的文字表达式的类型为字符串，数值（int，实数，十六进制），布尔值和null。字符串由单引号引起来。要将单引号本身放在字符串中，请使用两个单引号字符。

以下清单显示了文字的简单用法。通常，它们不是像这样孤立地使用，而是作为更复杂的表达式的一部分使用-例如，在逻辑比较运算符的一侧使用文字。

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();

// evals to "Hello World"
String helloWorld = (String) parser.parseExpression("'Hello World'").getValue();

double avogadrosNumber = (Double) parser.parseExpression("6.0221415E+23").getValue();

// evals to 2147483647
int maxValue = (Integer) parser.parseExpression("0x7FFFFFFF").getValue();

boolean trueValue = (Boolean) parser.parseExpression("true").getValue();

Object nullValue = parser.parseExpression("null").getValue();
```

数字支持使用负号，指数符号和小数点。默认情况下，使用Double.parseDouble（）解析实数。

#### 4.3.2。属性，数组，列表，映射和索引器

使用属性引用进行导航很容易。为此，请使用句点来指示嵌套的属性值。`Inventor`类`pupin`和的实例`tesla`被[示例](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-example-classes)部分[使用](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-example-classes)的[类中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-example-classes)列出的数据填充。要向下导航并获取特斯拉的出生年份和普平的出生城市，我们使用以下表达式：

爪哇

科特林

```java
// evals to 1856
int year = (Integer) parser.parseExpression("Birthdate.Year + 1900").getValue(context);

String city = (String) parser.parseExpression("placeOfBirth.City").getValue(context);
```

属性名称的首字母允许不区分大小写。数组和列表的内容通过使用方括号表示法获得，如以下示例所示：

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

// Inventions Array

// evaluates to "Induction motor"
String invention = parser.parseExpression("inventions[3]").getValue(
        context, tesla, String.class);

// Members List

// evaluates to "Nikola Tesla"
String name = parser.parseExpression("Members[0].Name").getValue(
        context, ieee, String.class);

// List and Array navigation
// evaluates to "Wireless communication"
String invention = parser.parseExpression("Members[0].Inventions[6]").getValue(
        context, ieee, String.class);
```

通过在方括号内指定文字键值可以获取映射的内容。在以下示例中，由于`Officers`映射的键是字符串，因此我们可以指定字符串文字：

爪哇

科特林

```java
// Officer's Dictionary

Inventor pupin = parser.parseExpression("Officers['president']").getValue(
        societyContext, Inventor.class);

// evaluates to "Idvor"
String city = parser.parseExpression("Officers['president'].PlaceOfBirth.City").getValue(
        societyContext, String.class);

// setting values
parser.parseExpression("Officers['advisors'][0].PlaceOfBirth.Country").setValue(
        societyContext, "Croatia");
```

#### 4.3.3。内联列表

您可以使用`{}`符号直接在表达式中表达列表。

爪哇

科特林

```java
// evaluates to a Java list containing the four numbers
List numbers = (List) parser.parseExpression("{1,2,3,4}").getValue(context);

List listOfLists = (List) parser.parseExpression("{{'a','b'},{'x','y'}}").getValue(context);
```

`{}`本身意味着一个空列表。出于性能原因，如果列表本身完全由固定文字组成，则会创建一个常量列表来表示该表达式（而不是在每次求值时都构建一个新列表）。

#### 4.3.4。内联地图

您也可以使用`{key:value}`符号直接在表达式中表达地图。以下示例显示了如何执行此操作：

爪哇

科特林

```java
// evaluates to a Java map containing the two entries
Map inventorInfo = (Map) parser.parseExpression("{name:'Nikola',dob:'10-July-1856'}").getValue(context);

Map mapOfMaps = (Map) parser.parseExpression("{name:{first:'Nikola',last:'Tesla'},dob:{day:10,month:'July',year:1856}}").getValue(context);
```

`{:}`本身意味着一个空的地图。出于性能原因，如果映射图本身由固定的文字或其他嵌套的常量结构（列表或映射图）组成，则会创建一个常量映射图来表示该表达式（而不是在每次求值时都新建一个映射图）。映射键的引用是可选的。上面的示例不使用带引号的键。

#### 4.3.5。阵列构造

您可以使用熟悉的Java语法来构建数组，可以选择提供一个初始化程序以在构造时填充该数组。以下示例显示了如何执行此操作：

爪哇

科特林

```java
int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue(context);

// Array with initializer
int[] numbers2 = (int[]) parser.parseExpression("new int[]{1,2,3}").getValue(context);

// Multi dimensional array
int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue(context);
```

构造多维数组时，当前无法提供初始化程序。

#### 4.3.6。方法

您可以使用典型的Java编程语法来调用方法。您还可以在文字上调用方法。还支持变量参数。下面的示例演示如何调用方法：

爪哇

科特林

```java
// string literal, evaluates to "bc"
String bc = parser.parseExpression("'abc'.substring(1, 3)").getValue(String.class);

// evaluates to true
boolean isMember = parser.parseExpression("isMember('Mihajlo Pupin')").getValue(
        societyContext, Boolean.class);
```

#### 4.3.7。经营者

Spring表达式语言支持以下几种运算符：

- [关系运算符](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-operators-relational)
- [逻辑运算符](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-operators-logical)
- [数学运算符](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-operators-mathematical)
- [赋值运算符](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions-assignment)

##### 关系运算符

使用标准运算符表示法支持关系运算符（等于，不等于，小于，小于或等于，大于和大于或等于）。以下清单显示了一些运算符示例：

爪哇

科特林

```java
// evaluates to true
boolean trueValue = parser.parseExpression("2 == 2").getValue(Boolean.class);

// evaluates to false
boolean falseValue = parser.parseExpression("2 < -5.0").getValue(Boolean.class);

// evaluates to true
boolean trueValue = parser.parseExpression("'black' < 'block'").getValue(Boolean.class);
```

|      | 与之进行的大于和小于比较`null`遵循一个简单的规则：`null`被视为无（不是零）。结果，任何其他值始终大于`null`（`X > null`始终为`true`），并且其他任何值都不小于零（`X < null`始终为`false`）。如果您更喜欢使用数字比较，请避免基于数字的比较，而`null`建议使用零进行比较（例如`X > 0`或`X < 0`）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

除了标准的关系运算符外，SpEL还支持`instanceof`和基于正则表达式的`matches`运算符。以下清单显示了两个示例：

爪哇

科特林

```java
// evaluates to false
boolean falseValue = parser.parseExpression(
        "'xyz' instanceof T(Integer)").getValue(Boolean.class);

// evaluates to true
boolean trueValue = parser.parseExpression(
        "'5.00' matches '^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);

//evaluates to false
boolean falseValue = parser.parseExpression(
        "'5.0067' matches '^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);
```

|      | 请注意基本类型，因为它们会立即装箱成包装类型，因此按预期将`1 instanceof T(int)`值计算为`false`while ，将`1 instanceof T(Integer)` 值计算`true`为。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

每个符号运算符也可以指定为纯字母等效项。这样可以避免使用的符号对于嵌入表达式的文档类型具有特殊含义的问题（例如在XML文档中）。等效的文字是：

- `lt`（`<`）
- `gt`（`>`）
- `le`（`<=`）
- `ge`（`>=`）
- `eq`（`==`）
- `ne`（`!=`）
- `div`（`/`）
- `mod`（`%`）
- `not`（`!`）。

所有文本运算符都不区分大小写。

##### 逻辑运算符

SpEL支持以下逻辑运算符：

- `and`（`&&`）
- `or`（`||`）
- `not`（`!`）

下面的示例演示如何使用逻辑运算符

爪哇

科特林

```java
// -- AND --

// evaluates to false
boolean falseValue = parser.parseExpression("true and false").getValue(Boolean.class);

// evaluates to true
String expression = "isMember('Nikola Tesla') and isMember('Mihajlo Pupin')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- OR --

// evaluates to true
boolean trueValue = parser.parseExpression("true or false").getValue(Boolean.class);

// evaluates to true
String expression = "isMember('Nikola Tesla') or isMember('Albert Einstein')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- NOT --

// evaluates to false
boolean falseValue = parser.parseExpression("!true").getValue(Boolean.class);

// -- AND and NOT --
String expression = "isMember('Nikola Tesla') and !isMember('Mihajlo Pupin')";
boolean falseValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);
```

##### 数学运算符

您可以在数字和字符串上使用加法运算符。您只能对数字使用减法，乘法和除法运算符。您还可以使用模数（％）和指数幂（^）运算符。强制执行标准运算符优先级。以下示例显示了正在使用的数学运算符：

爪哇

科特林

```java
// Addition
int two = parser.parseExpression("1 + 1").getValue(Integer.class);  // 2

String testString = parser.parseExpression(
        "'test' + ' ' + 'string'").getValue(String.class);  // 'test string'

// Subtraction
int four = parser.parseExpression("1 - -3").getValue(Integer.class);  // 4

double d = parser.parseExpression("1000.00 - 1e4").getValue(Double.class);  // -9000

// Multiplication
int six = parser.parseExpression("-2 * -3").getValue(Integer.class);  // 6

double twentyFour = parser.parseExpression("2.0 * 3e0 * 4").getValue(Double.class);  // 24.0

// Division
int minusTwo = parser.parseExpression("6 / -3").getValue(Integer.class);  // -2

double one = parser.parseExpression("8.0 / 4e0 / 2").getValue(Double.class);  // 1.0

// Modulus
int three = parser.parseExpression("7 % 4").getValue(Integer.class);  // 3

int one = parser.parseExpression("8 / 5 % 2").getValue(Integer.class);  // 1

// Operator precedence
int minusTwentyOne = parser.parseExpression("1+2-3*8").getValue(Integer.class);  // -21
```

##### 赋值运算符

要设置属性，请使用赋值运算符（`=`）。这通常在的调用中完成，`setValue`但也可以在的调用中完成`getValue`。下面的清单显示了使用赋值运算符的两种方法：

爪哇

科特林

```java
Inventor inventor = new Inventor();
EvaluationContext context = SimpleEvaluationContext.forReadWriteDataBinding().build();

parser.parseExpression("Name").setValue(context, inventor, "Aleksandar Seovic");

// alternatively
String aleks = parser.parseExpression(
        "Name = 'Aleksandar Seovic'").getValue(context, inventor, String.class);
```

#### 4.3.8。种类

您可以使用特殊`T`运算符来指定`java.lang.Class`（类型）的实例。静态方法也可以通过使用此运算符来调用。在 `StandardEvaluationContext`使用`TypeLocator`查找类型以及 `StandardTypeLocator`（可替换）是建立与所述的理解 `java.lang`包。这意味着`T()`对其中的类型的引用`java.lang`不需要完全限定，但是所有其他类型的引用都必须是完全限定的。以下示例显示如何使用`T`运算符：

爪哇

科特林

```java
Class dateClass = parser.parseExpression("T(java.util.Date)").getValue(Class.class);

Class stringClass = parser.parseExpression("T(String)").getValue(Class.class);

boolean trueValue = parser.parseExpression(
        "T(java.math.RoundingMode).CEILING < T(java.math.RoundingMode).FLOOR")
        .getValue(Boolean.class);
```

#### 4.3.9。建设者

您可以使用`new`运算符来调用构造函数。除了基本类型（`int`，`float`等）和String之外，您都应使用完全限定的类名。下面的示例演示如何使用`new`运算符来调用构造函数：

爪哇

科特林

```java
Inventor einstein = p.parseExpression(
        "new org.spring.samples.spel.inventor.Inventor('Albert Einstein', 'German')")
        .getValue(Inventor.class);

//create new inventor instance within add method of List
p.parseExpression(
        "Members.add(new org.spring.samples.spel.inventor.Inventor(
            'Albert Einstein', 'German'))").getValue(societyContext);
```

#### 4.3.10。变数

您可以使用`#variableName`语法在表达式中引用变量。通过`setVariable`在`EvaluationContext`实现中使用方法来设置变量。

|      | 有效的变量名称必须由以下一个或多个受支持的字符组成。信：`A`到`Z`和`a`到`z`位数：`0`至`9`下划线： `_`美元符号： `$` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下示例显示了如何使用变量。

爪哇

科特林

```java
Inventor tesla = new Inventor("Nikola Tesla", "Serbian");

EvaluationContext context = SimpleEvaluationContext.forReadWriteDataBinding().build();
context.setVariable("newName", "Mike Tesla");

parser.parseExpression("Name = #newName").getValue(context, tesla);
System.out.println(tesla.getName())  // "Mike Tesla"
```

##### 在`#this`和`#root`变量

`#this`始终定义该变量，并引用当前的评估对象（针对不合格的引用，将对其进行解析）。`#root`始终定义该变量，并引用根上下文对象。尽管`#this`随着表达式的组成部分的求值可能会有所不同，但`#root`始终指的是根。以下示例显示了如何使用`#this`和`#root`变量：

爪哇

科特林

```java
// create an array of integers
List<Integer> primes = new ArrayList<Integer>();
primes.addAll(Arrays.asList(2,3,5,7,11,13,17));

// create parser and set variable 'primes' as the array of integers
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataAccess();
context.setVariable("primes", primes);

// all prime numbers > 10 from the list (using selection ?{...})
// evaluates to [11, 13, 17]
List<Integer> primesGreaterThanTen = (List<Integer>) parser.parseExpression(
        "#primes.?[#this>10]").getValue(context);
```

#### 4.3.11。功能

您可以通过注册可以在表达式字符串中调用的用户定义函数来扩展SpEL。该功能通过进行注册`EvaluationContext`。下面的示例显示如何注册用户定义的函数：

爪哇

科特林

```java
Method method = ...;

EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();
context.setVariable("myFunction", method);
```

例如，考虑以下用于反转字符串的实用程序方法：

爪哇

科特林

```java
public abstract class StringUtils {

    public static String reverseString(String input) {
        StringBuilder backwards = new StringBuilder(input.length());
        for (int i = 0; i < input.length(); i++) {
            backwards.append(input.charAt(input.length() - 1 - i));
        }
        return backwards.toString();
    }
}
```

然后，您可以注册并使用前面的方法，如以下示例所示：

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();

EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();
context.setVariable("reverseString",
        StringUtils.class.getDeclaredMethod("reverseString", String.class));

String helloWorldReversed = parser.parseExpression(
        "#reverseString('hello')").getValue(context, String.class);
```

#### 4.3.12。Bean参考

如果评估上下文已使用bean解析器配置，则可以使用`@`符号从表达式中查找bean 。以下示例显示了如何执行此操作：

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"something") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("@something").getValue(context);
```

要访问工厂bean本身，应改为在bean名称前添加一个`&`符号。以下示例显示了如何执行此操作：

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"&foo") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("&foo").getValue(context);
```

#### 4.3.13。三元运算符（If-Then-Else）

您可以使用三元运算符在表达式内部执行if-then-else条件逻辑。以下清单显示了一个最小的示例：

爪哇

科特林

```java
String falseString = parser.parseExpression(
        "false ? 'trueExp' : 'falseExp'").getValue(String.class);
```

在这种情况下，布尔值`false`导致返回字符串value `'falseExp'`。一个更现实的示例如下：

爪哇

科特林

```java
parser.parseExpression("Name").setValue(societyContext, "IEEE");
societyContext.setVariable("queryName", "Nikola Tesla");

expression = "isMember(#queryName)? #queryName + ' is a member of the ' " +
        "+ Name + ' Society' : #queryName + ' is not a member of the ' + Name + ' Society'";

String queryResultString = parser.parseExpression(expression)
        .getValue(societyContext, String.class);
// queryResultString = "Nikola Tesla is a member of the IEEE Society"
```

有关三元运算符的更短语法，请参阅关于Elvis运算符的下一部分。

#### 4.3.14。猫王算子

Elvis运算符是三元运算符语法的简化，并且在 [Groovy](http://www.groovy-lang.org/operators.html#_elvis_operator)语言中使用。使用三元运算符语法，通常必须将变量重复两次，如以下示例所示：

```groovy
String name = "Elvis Presley";
String displayName = (name != null ? name : "Unknown");
```

相反，您可以使用Elvis运算符（其命名类似于Elvis的发型）。以下示例显示了如何使用Elvis运算符：

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();

String name = parser.parseExpression("name?:'Unknown'").getValue(String.class);
System.out.println(name);  // 'Unknown'
```

以下清单显示了一个更复杂的示例：

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
String name = parser.parseExpression("Name?:'Elvis Presley'").getValue(context, tesla, String.class);
System.out.println(name);  // Nikola Tesla

tesla.setName(null);
name = parser.parseExpression("Name?:'Elvis Presley'").getValue(context, tesla, String.class);
System.out.println(name);  // Elvis Presley
```

|      | 您可以使用Elvis运算符在表达式中应用默认值。以下示例显示如何在`@Value`表达式中使用Elvis运算符：`@Value("#{systemProperties['pop3.port'] ?: 25}")``pop3.port`如果定义，将注入系统属性，否则将注入25。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 4.3.15。安全导航操作员

安全导航运算符用于避免a，`NullPointerException`并且来自[Groovy](http://www.groovy-lang.org/operators.html#_safe_navigation_operator) 语言。通常，当您引用一个对象时，可能需要在访问该对象的方法或属性之前验证其是否为null。为了避免这种情况，安全导航运算符返回null而不是引发异常。下面的示例演示如何使用安全导航操作符：

爪哇

科特林

```java
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
tesla.setPlaceOfBirth(new PlaceOfBirth("Smiljan"));

String city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, tesla, String.class);
System.out.println(city);  // Smiljan

tesla.setPlaceOfBirth(null);
city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, tesla, String.class);
System.out.println(city);  // null - does not throw NullPointerException!!!
```

#### 4.3.16。馆藏选择

选择是一种强大的表达语言功能，可让您通过从源集合中进行选择来将其转换为另一个集合。

选择使用的语法`.?[selectionExpression]`。它过滤集合并返回一个包含原始元素子集的新集合。例如，通过选择，我们可以轻松地获得塞尔维亚发明者的列表，如以下示例所示：

爪哇

科特林

```java
List<Inventor> list = (List<Inventor>) parser.parseExpression(
        "Members.?[Nationality == 'Serbian']").getValue(societyContext);
```

在列表和地图上都可以选择。对于列表，将针对每个单独的列表元素评估选择标准。针对地图，针对每个地图条目（Java类型的对象）评估选择标准 `Map.Entry`。每个地图条目都有其键和值，可作为属性进行访问，以供选择使用。

以下表达式返回一个新地图，该地图由原始地图中条目值小于27的那些元素组成：

爪哇

科特林

```java
Map newMap = parser.parseExpression("map.?[value<27]").getValue();
```

除了返回所有选定的元素外，您只能检索第一个或最后一个值。要获得与所选内容匹配的第一个条目，语法为 `.^[selectionExpression]`。要获取最后的匹配选择，语法为 `.$[selectionExpression]`。

#### 4.3.17。集合投影

投影使集合可以驱动子表达式的求值，结果是一个新的集合。投影的语法为`.![projectionExpression]`。例如，假设我们有一个发明人列表，但想要他们出生的城市的列表。实际上，我们希望为发明人列表中的每个条目评估“ placeOfBirth.city”。下面的示例使用投影来做到这一点：

爪哇

科特林

```java
// returns ['Smiljan', 'Idvor' ]
List placesOfBirth = (List)parser.parseExpression("Members.![placeOfBirth.city]");
```

您还可以使用地图来驱动投影，在这种情况下，将根据地图中的每个条目（以Java表示`Map.Entry`）对投影表达式进行求值。跨地图的投影结果是一个列表，其中包含针对每个地图条目的投影表达式的评估。

#### 4.3.18。表达式模板

表达式模板允许将文字文本与一个或多个评估块混合。每个评估块都由您可以定义的前缀和后缀字符定界。常见的选择是`#{ }`用作定界符，如以下示例所示：

爪哇

科特林

```java
String randomPhrase = parser.parseExpression(
        "random number is #{T(java.lang.Math).random()}",
        new TemplateParserContext()).getValue(String.class);

// evaluates to "random number is 0.7038186818312008"
```

通过将文字文本`'random number is '`与对`#{ }`定界符内的表达式进行求值的结果（在本例中为调用该`random()`方法的结果）来对字符串进行求值。该`parseExpression()`方法的第二个参数是类型`ParserContext`。该`ParserContext`接口用于影响表达式的解析方式，以支持表达式模板功能。定义`TemplateParserContext`如下：

爪哇

科特林

```java
public class TemplateParserContext implements ParserContext {

    public String getExpressionPrefix() {
        return "#{";
    }

    public String getExpressionSuffix() {
        return "}";
    }

    public boolean isTemplate() {
        return true;
    }
}
```

### 4.4。示例中使用的类

本节列出了本章示例中使用的类。

发明家

发明家

```java
package org.spring.samples.spel.inventor;

import java.util.Date;
import java.util.GregorianCalendar;

public class Inventor {

    private String name;
    private String nationality;
    private String[] inventions;
    private Date birthdate;
    private PlaceOfBirth placeOfBirth;

    public Inventor(String name, String nationality) {
        GregorianCalendar c= new GregorianCalendar();
        this.name = name;
        this.nationality = nationality;
        this.birthdate = c.getTime();
    }

    public Inventor(String name, Date birthdate, String nationality) {
        this.name = name;
        this.nationality = nationality;
        this.birthdate = birthdate;
    }

    public Inventor() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getNationality() {
        return nationality;
    }

    public void setNationality(String nationality) {
        this.nationality = nationality;
    }

    public Date getBirthdate() {
        return birthdate;
    }

    public void setBirthdate(Date birthdate) {
        this.birthdate = birthdate;
    }

    public PlaceOfBirth getPlaceOfBirth() {
        return placeOfBirth;
    }

    public void setPlaceOfBirth(PlaceOfBirth placeOfBirth) {
        this.placeOfBirth = placeOfBirth;
    }

    public void setInventions(String[] inventions) {
        this.inventions = inventions;
    }

    public String[] getInventions() {
        return inventions;
    }
}
```

PlaceOfBirth.java

出生地

```java
package org.spring.samples.spel.inventor;

public class PlaceOfBirth {

    private String city;
    private String country;

    public PlaceOfBirth(String city) {
        this.city=city;
    }

    public PlaceOfBirth(String city, String country) {
        this(city);
        this.country = country;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String s) {
        this.city = s;
    }

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
    }
}
```

社会.java

社会

```java
package org.spring.samples.spel.inventor;

import java.util.*;

public class Society {

    private String name;

    public static String Advisors = "advisors";
    public static String President = "president";

    private List<Inventor> members = new ArrayList<Inventor>();
    private Map officers = new HashMap();

    public List getMembers() {
        return members;
    }

    public Map getOfficers() {
        return officers;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public boolean isMember(String name) {
        for (Inventor inventor : members) {
            if (inventor.getName().equals(name)) {
                return true;
            }
        }
        return false;
    }
}
```

## 5.使用Spring进行面向方面的编程

面向方面的编程（AOP）通过提供另一种思考程序结构的方式来补充面向对象的编程（OOP）。OOP中模块化的关键单元是类，而在AOP中模块化是方面。方面使关注点（例如事务管理）的模块化跨越了多个类型和对象。（这种关注在AOP文献中通常被称为“跨领域”关注。）

Spring的关键组件之一是AOP框架。虽然Spring IoC容器不依赖于AOP（这意味着您不需要使用AOP），但AOP是对Spring IoC的补充，以提供功能非常强大的中间件解决方案。

具有AspectJ切入点的Spring AOP

Spring通过使用[基于模式的方法](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-schema)或[@AspectJ批注样式，](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj)提供了编写自定义方面的简单而强大的方法 。这两种样式都提供了完全类型化的建议，并使用了AspectJ切入点语言，同时仍然使用Spring AOP进行编织。

本章讨论基于架构和基于@AspectJ的AOP支持。[下一章](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-api)将讨论较低级别的AOP支持。

AOP在Spring Framework中用于：

- 提供声明式企业服务。这种服务最重要的是 [声明式事务管理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#transaction-declarative)。
- 让用户实现自定义方面，并通过AOP补充其对OOP的使用。

|      | 如果您只对通用声明性服务或其他预打包的声明性中间件服务（例如池）感兴趣，则无需直接使用Spring AOP，并且可以跳过本章的大部分内容。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 5.1。AOP概念

让我们首先定义一些重要的AOP概念和术语。这些术语不是特定于Spring的。不幸的是，AOP术语并不是特别直观。但是，如果使用Spring自己的术语，将会更加令人困惑。

- 方面：涉及多个类别的关注点的模块化。事务管理是企业Java应用程序中横切关注的一个很好的例子。在Spring AOP中，方面是通过使用常规类（[基于模式的方法](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-schema)）或使用注释进行`@Aspect`注释的常规类 （[@AspectJ样式](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj)）来实现的。
- 连接点：程序执行过程中的一个点，例如方法执行或异常处理。在Spring AOP中，连接点始终代表方法的执行。
- 建议：方面在特定的连接点处采取的操作。不同类型的建议包括“周围”，“之前”和“之后”建议。（建议类型将在后面讨论。）包括Spring在内的许多AOP框架都将建议建模为拦截器，并在连接点周围维护一系列拦截器。
- 切入点：与连接点匹配的谓词。建议与切入点表达式关联，并在与该切入点匹配的任何连接点处运行（例如，执行具有特定名称的方法）。切入点表达式匹配的连接点的概念是AOP的核心，并且Spring默认使用AspectJ切入点表达语言。
- 简介：代表类型声明其他方法或字段。Spring AOP允许您向任何建议对象引入新的接口（和相应的实现）。例如，您可以使用简介使Bean实现 `IsModified`接口，以简化缓存。（在AspectJ社区中，介绍被称为类型间声明。）
- 目标对象：一个或多个方面建议的对象。也称为“建议对象”。由于Spring AOP是使用运行时代理实现的，因此该对象始终是代理对象。
- AOP代理：由AOP框架创建的一个对象，用于实现方面协定（建议方法执行等）。在Spring Framework中，AOP代理是JDK动态代理或CGLIB代理。
- 编织：将方面与其他应用程序类型或对象链接以创建建议的对象。这可以在编译时（例如，使用AspectJ编译器），加载时或在运行时完成。像其他纯Java AOP框架一样，Spring AOP在运行时执行编织。

Spring AOP包括以下类型的建议：

- 在建议之前：在连接点之前运行的建议，但是它不能阻止执行流程继续进行到连接点（除非它引发异常）。
- 返回建议后：在连接点正常完成之后要运行的建议（例如，如果方法返回而没有引发异常）。
- 抛出建议后：如果方法因抛出异常而退出，则执行建议。
- 建议（最终）之后：无论连接点退出的方式如何（正常或特殊返回），均应执行建议。
- 围绕建议：围绕连接点的建议，例如方法调用。这是最有力的建议。周围建议可以在方法调用之前和之后执行自定义行为。它还负责选择是返回连接点还是通过返回其自身的返回值或引发异常来捷径建议的方法执行。

围绕建议是最通用的建议。由于Spring AOP与AspectJ一样，提供了各种建议类型，因此我们建议您使用功能最弱的建议类型，以实现所需的行为。例如，如果您只需要使用方法的返回值更新缓存，则最好使用返回后的建议而不是周围的建议，尽管周围的建议可以完成相同的事情。使用最具体的建议类型可提供更简单的编程模型，并减少潜在的错误。例如，您不需要`proceed()` 在`JoinPoint`用于周围建议的方法上调用方法，因此，您不会失败。

所有建议参数都是静态类型的，因此您可以使用适当类型（例如，方法执行的返回值的类型）而不是`Object`数组的建议参数。

切入点匹配的连接点的概念是AOP的关键，它与仅提供拦截功能的旧技术有所不同。切入点使建议的目标独立于面向对象的层次结构。例如，您可以将提供声明性事务管理的环绕建议应用于跨越多个对象（例如服务层中的所有业务操作）的一组方法。

### 5.2。春季AOP能力和目标

Spring AOP是用纯Java实现的。不需要特殊的编译过程。Spring AOP不需要控制类加载器的层次结构，因此适合在Servlet容器或应用程序服务器中使用。

Spring AOP当前仅支持方法执行连接点（建议在Spring Bean上执行方法）。尽管可以添加对字段侦听的支持而不破坏核心Spring AOP API，但是字段侦听并未实现。如果需要建议字段访问和更新连接点，请考虑使用诸如AspectJ之类的语言。

Spring AOP的AOP方法不同于大多数其他AOP框架。目的不是提供最完整的AOP实现（尽管Spring AOP相当强大）。相反，其目的是在AOP实现和Spring IoC之间提供紧密的集成，以帮助解决企业应用程序中的常见问题。

因此，例如，通常将Spring Framework的AOP功能与Spring IoC容器结合使用。通过使用常规bean定义语法来配置方面（尽管这允许强大的“自动代理”功能）。这是与其他AOP实现的关键区别。使用Spring AOP无法轻松或高效地完成某些事情，例如建议非常细粒度的对象（通常是域对象）。在这种情况下，AspectJ是最佳选择。但是，我们的经验是，Spring AOP为AOP可以解决的企业Java应用程序中的大多数问题提供了出色的解决方案。

Spring AOP从未努力与AspectJ竞争以提供全面的AOP解决方案。我们认为，基于代理的框架（如Spring AOP）和成熟的框架（如AspectJ）都是有价值的，它们是互补的，而不是竞争。Spring无缝地将Spring AOP和IoC与AspectJ集成在一起，以在基于Spring的一致应用程序架构中支持AOP的所有使用。这种集成不会影响Spring AOP API或AOP Alliance API。Spring AOP仍然向后兼容。有关 Spring AOP API的讨论，请参见[下一章](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-api)。

|      | Spring框架的中心宗旨之一是非侵入性。这是一个想法，不应强迫您将特定于框架的类和接口引入业务或域模型。但是，在某些地方，Spring Framework确实为您提供了将特定于Spring Framework的依赖项引入代码库的选项。提供此类选项的基本原理是，在某些情况下，以这种方式阅读或编码某些特定功能可能会更加容易。但是，Spring框架（几乎）总是为您提供选择：您可以自由地就哪个选项最适合您的特定用例或场景做出明智的决定。与本章相关的一种选择是选择哪种AOP框架（以及哪种AOP样式）。您可以选择AspectJ和/或Spring AOP。您也可以选择@AspectJ注释样式方法或Spring XML配置样式方法。本章选择首先介绍@AspectJ样式的方法这一事实不应被视为表明Spring团队比Spring XML配置样式更喜欢@AspectJ注释样式的方法。有关每种样式的“为什么和为什么”的更完整讨论，请参见[选择要使用的AOP声明样式](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-choosing)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 5.3。AOP代理

Spring AOP默认将标准JDK动态代理用于AOP代理。这使得可以代理任何接口（或一组接口）。

Spring AOP也可以使用CGLIB代理。这对于代理类而不是接口是必需的。默认情况下，如果业务对象未实现接口，则使用CGLIB。由于对接口而不是对类进行编程是一种好习惯，因此业务类通常实现一个或多个业务接口。在那些需要建议在接口上未声明的方法或需要将代理对象作为具体类型传递给方法的情况下（在极少数情况下），可以 [强制使用CGLIB](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-proxying)。

掌握Spring AOP是基于代理的这一事实很重要。有关完全了解此实现细节实际含义的详细信息，请参阅 [了解AOP代理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-understanding-aop-proxies)。

### 5.4。@AspectJ支持

@AspectJ是一种将方面声明为带有注释的常规Java类的样式。@AspectJ样式是[AspectJ项目](https://www.eclipse.org/aspectj)在AspectJ 5版本中引入的 。Spring使用AspectJ提供的用于切入点解析和匹配的库来解释与AspectJ 5相同的注释。但是，AOP运行时仍然是纯Spring AOP，并且不依赖于AspectJ编译器或编织器。

|      | 使用AspectJ编译器和weaver可以使用完整的AspectJ语言，有关在[Spring Applications](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-using-aspectj)中[使用AspectJ进行](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-using-aspectj)了讨论。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 5.4.1。启用@AspectJ支持

要在Spring配置中使用@AspectJ方面，您需要启用Spring支持以基于@AspectJ方面配置Spring AOP，并基于这些方面是否建议对Bean进行自动代理。通过自动代理，我们的意思是，如果Spring确定一个或多个方面建议一个bean，它会自动为该bean生成一个代理来拦截方法调用并确保按需执行建议。

可以使用XML或Java样式的配置启用@AspectJ支持。无论哪种情况，您都需要确保AspectJ的`aspectjweaver.jar`库位于应用程序的类路径（版本1.8或更高版本）上。该库在`lib`AspectJ发行版本的目录中或从Maven Central存储库中可用 。

##### 通过Java配置启用@AspectJ支持

要使用Java启用@AspectJ支持`@Configuration`，请添加`@EnableAspectJAutoProxy` 注释，如以下示例所示：

爪哇

科特林

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```

##### 通过XML配置启用@AspectJ支持

要使用基于XML的配置启用@AspectJ支持，请使用`aop:aspectj-autoproxy` 元素，如以下示例所示：

```xml
<aop:aspectj-autoproxy/>
```

假定您使用[基于XML Schema的配置中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-schemas)所述的模式支持 。请参阅[AOP模式](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-schemas-aop)以了解如何在`aop`名称空间中导入标签。

#### 5.4.2。声明一个方面

启用@AspectJ支持后，`@Aspect`Spring会自动检测在应用程序上下文中使用@AspectJ方面（具有注释）的类定义的任何bean，并用于配置Spring AOP。接下来的两个示例显示了一个不太有用的方面所需的最小定义。

这两个示例中的第一个示例显示了应用程序上下文中的常规bean定义，该定义指向具有`@Aspect`注释的bean类：

```xml
<bean id="myAspect" class="org.xyz.NotVeryUsefulAspect">
    <!-- configure properties of the aspect here -->
</bean>
```

这两个示例中的第二个示例显示了`NotVeryUsefulAspect`类定义，该类定义带有`org.aspectj.lang.annotation.Aspect`批注；

爪哇

科特林

```java
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class NotVeryUsefulAspect {

}
```

方面（带有注释的类`@Aspect`）可以具有方法和字段，与任何其他类相同。它们还可以包含切入点，建议和引入（类型间）声明。

|      | 通过组件扫描自动检测方面您可以将方面类注册为Spring XML配置中的常规bean，也可以通过类路径扫描自动检测它们-与其他任何Spring管理的bean一样。但是，请注意，`@Aspect`注释不足以在类路径中进行自动检测。为此，您需要添加一个单独的`@Component`注释（或者，或者，按照Spring的组件扫描程序的规则，一个合格的自定义原型注释）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 向其他方面提供建议？在Spring AOP中，方面本身不能成为其他方面的建议目标。`@Aspect`类上的注释将其标记为一个方面，因此将其从自动代理中排除。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 5.4.3。声明切入点

切入点确定了感兴趣的连接点，从而使我们能够控制执行建议的时间。Spring AOP仅支持Spring Bean的方法执行连接点，因此您可以将切入点视为与Spring Bean上的方法执行匹配。切入点声明由两部分组成：一个包含名称和任何参数的签名，以及一个切入点表达式，该切入点表达式精确地确定我们感兴趣的方法执行。在AOP的@AspectJ批注样式中，常规方法定义提供了切入点签名。 ，并通过使用`@Pointcut`注释指示切入点表达式（用作切入点签名的方法必须具有`void`返回类型）。

一个示例可能有助于使切入点签名和切入点表达式之间的区别变得清晰。以下示例定义了一个名为的切入点`anyOldTransfer`，该切入点匹配任何名为的方法的执行`transfer`：

爪哇

科特林

```java
@Pointcut("execution(* transfer(..))") // the pointcut expression
private void anyOldTransfer() {} // the pointcut signature
```

形成`@Pointcut`注释值的切入点表达式是一个常规的AspectJ 5切入点表达式。有关AspectJ的切入点语言的完整讨论，请参见[AspectJ编程指南](https://www.eclipse.org/aspectj/doc/released/progguide/index.html)（以及扩展，包括 [AspectJ 5开发人员的笔记本](https://www.eclipse.org/aspectj/doc/released/adk15notebook/index.html)）或有关AspectJ的书籍之一（如Colyer等人的*Eclipse AspectJ*，或《*AspectJ in Action》*，由Ramnivas Laddad撰写）。

##### 支持的切入点指示符

Spring AOP支持以下在切入点表达式中使用的AspectJ切入点指示符（PCD）：

- `execution`：用于匹配方法执行的连接点。这是使用Spring AOP时要使用的主要切入点指示符。
- `within`：将匹配限制为某些类型内的连接点（使用Spring AOP时，在匹配类型内声明的方法的执行）。
- `this`：限制匹配到连接点（使用Spring AOP时方法的执行）的匹配，其中bean引用（Spring AOP代理）是给定类型的实例。
- `target`：在目标对象（代理的应用程序对象）是给定类型的实例的情况下，将匹配限制为连接点（使用Spring AOP时方法的执行）。
- `args`：在参数为给定类型的实例的情况下，将匹配限制为连接点（使用Spring AOP时方法的执行）。
- `@target`：在执行对象的类具有给定类型的注释的情况下，将匹配限制为连接点（使用Spring AOP时方法的执行）。
- `@args`：限制匹配的连接点（使用Spring AOP时方法的执行），其中传递的实际参数的运行时类型具有给定类型的注释。
- `@within`：将匹配限制为具有给定注释的类型内的连接点（使用Spring AOP时，使用给定注释的类型中声明的方法的执行）。
- `@annotation`：将匹配限制在连接点的主题（Spring AOP中正在执行的方法）具有给定注释的连接点上。

其他切入点类型

完整的AspectJ切入点语言支持未在Spring支持额外的切入点指示符：`call`，`get`，`set`，`preinitialization`， `staticinitialization`，`initialization`，`handler`，`adviceexecution`，`withincode`，`cflow`， `cflowbelow`，`if`，`@this`，和`@withincode`。在Spring AOP解释的切入点表达式中使用这些切入点指示符会导致`IllegalArgumentException`抛出异常。

Spring AOP支持的切入点指示符集合可能会在将来的版本中扩展，以支持更多的AspectJ切入点指示符。

由于Spring AOP仅将匹配限制为仅方法执行连接点，因此前面对切入点指示符的讨论所提供的定义比AspectJ编程指南中的定义要窄。除此之外，AspectJ本身具有基于类型的语义和，在执行的连接点，无论是`this`和`target`指的是相同的对象：对象执行方法。Spring AOP是基于代理的系统，可区分代理对象本身（绑定到`this`）和代理后面的目标对象（绑定到`target`）。

|      | 由于Spring的AOP框架基于代理的性质，因此根据定义，不会拦截目标对象内的调用。对于JDK代理，只能拦截代理上的公共接口方法调用。使用CGLIB，将拦截代理上的公共方法和受保护的方法调用（必要时甚至包可见的方法）。但是，通过代理进行的常见交互应该始终通过公共签名进行设计。请注意，切入点定义通常与任何拦截方法匹配。如果严格地将切入点设置为仅公开使用，即使在CGLIB代理方案中通过代理可能存在非公开交互，也需要相应地进行定义。如果您的拦截需要在目标类中包括方法调用甚至构造函数，请考虑使用Spring驱动的[本机AspectJ编织，](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-aj-ltw)而不是Spring的基于代理的AOP框架。这构成了具有不同特性的AOP使用模式，因此请确保在做出决定之前先熟悉编织。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Spring AOP还支持一个名为的附加PCD `bean`。使用此PCD，可以将连接点的匹配限制为特定的命名Spring Bean或一组命名Spring Bean（使用通配符时）。该`bean`PCD形式如下：

爪哇

科特林

```java
bean(idOrNameOfBean)
```

该`idOrNameOfBean`令牌可以是任何Spring bean的名字。提供了使用`*`字符的有限通配符支持，因此，如果为Spring bean建立了一些命名约定，则可以编写`bean`PCD表达式来选择它们。与其他切入点指示符一样，`bean`PCD也可以与`&&`（和），`||`（或）和`!`（取反）运算符一起使用。

|      | 该`bean`PCD在本机AspectJ织只在Spring AOP和不支持。它是AspectJ定义的标准PCD的特定于Spring的扩展，因此不适用于`@Aspect`模型中声明的方面。该`bean`PCD在实例级别（建设于Spring bean的概念），而不是在类型级别（到织造为主AOP是有限的）工作。基于实例的切入点指示符是Spring基于代理的AOP框架及其与Spring bean工厂的紧密集成的一种特殊功能，在该工厂中自然而直接地通过名称来标识特定的bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 组合切入点表达式

您可以使用`&&,` `||`和组合切入点表达式`!`。您也可以按名称引用切入点表达式。以下示例显示了三个切入点表达式：

爪哇

科特林

```java
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {} 

@Pointcut("within(com.xyz.someapp.trading..*)")
private void inTrading() {} 

@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {} 
```

|      | `anyPublicOperation` 如果方法执行连接点表示任何公共方法的执行，则匹配。 |
| ---- | ------------------------------------------------------------ |
|      | `inTrading` 如果交易模块中有方法执行，则匹配。               |
|      | `tradingOperation` 如果方法执行代表交易模块中的任何公共方法，则匹配。 |

最佳实践是从较小的命名组件中构建更复杂的切入点表达式，如先前所示。按名称引用切入点时，将应用常规的Java可见性规则（您可以看到相同类型的私有切入点，层次结构中受保护的切入点，任何地方的公共切入点，等等）。可见性不影响切入点匹配。

##### 共享通用切入点定义

在使用企业应用程序时，开发人员通常希望从多个方面引用应用程序的模块和特定的操作集。我们建议为此定义一个“ SystemArchitecture”方面，以捕获常见的切入点表达式。这样的方面通常类似于以下示例：

爪哇

科特林

```java
package com.xyz.someapp;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class SystemArchitecture {

    /**
     * A join point is in the web layer if the method is defined
     * in a type in the com.xyz.someapp.web package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.web..*)")
    public void inWebLayer() {}

    /**
     * A join point is in the service layer if the method is defined
     * in a type in the com.xyz.someapp.service package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.service..*)")
    public void inServiceLayer() {}

    /**
     * A join point is in the data access layer if the method is defined
     * in a type in the com.xyz.someapp.dao package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.dao..*)")
    public void inDataAccessLayer() {}

    /**
     * A business service is the execution of any method defined on a service
     * interface. This definition assumes that interfaces are placed in the
     * "service" package, and that implementation types are in sub-packages.
     *
     * If you group service interfaces by functional area (for example,
     * in packages com.xyz.someapp.abc.service and com.xyz.someapp.def.service) then
     * the pointcut expression "execution(* com.xyz.someapp..service.*.*(..))"
     * could be used instead.
     *
     * Alternatively, you can write the expression using the 'bean'
     * PCD, like so "bean(*Service)". (This assumes that you have
     * named your Spring service beans in a consistent fashion.)
     */
    @Pointcut("execution(* com.xyz.someapp..service.*.*(..))")
    public void businessService() {}

    /**
     * A data access operation is the execution of any method defined on a
     * dao interface. This definition assumes that interfaces are placed in the
     * "dao" package, and that implementation types are in sub-packages.
     */
    @Pointcut("execution(* com.xyz.someapp.dao.*.*(..))")
    public void dataAccessOperation() {}

}
```

您可以在需要切入点表达式的任何地方引用在这样的方面中定义的切入点。例如，要使服务层具有事务性，您可以编写以下内容：

```xml
<aop:config>
    <aop:advisor
        pointcut="com.xyz.someapp.SystemArchitecture.businessService()"
        advice-ref="tx-advice"/>
</aop:config>

<tx:advice id="tx-advice">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```

的``和``元件在讨论[基于Schema的AOP支持](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-schema)。[事务管理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#transaction)中讨论了[事务](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#transaction)元素。

##### 例子

Spring AOP用户可能`execution`最经常使用切入点指示符。执行表达式的格式如下：

```
    执行（修饰符模式？ret类型模式声明类型模式？名称模式（参数模式）
                抛出模式？）
```

除了返回类型模式（`ret-type-pattern`在前面的代码片段中），名称模式和参数模式以外的所有部分都是可选的。返回类型模式确定该方法的返回类型必须是什么才能使连接点匹配。 `*`最常用作返回类型模式。它匹配任何返回类型。仅当方法返回给定类型时，完全合格的类型名称才匹配。名称模式与方法名称匹配。您可以将`*`通配符用作名称模式的全部或一部分。如果指定了声明类型模式，请在其末尾添加一个尾随`.`以将其连接到名称模式组件。参数模式稍微复杂一些：`()`匹配不带参数的方法，而`(..)`匹配任意数量（零个或多个）的参数。的`(*)`模式匹配可以接收任意类型的一个参数的方法。 `(*,String)`与采用两个参数的方法匹配。第一个可以是任何类型，而第二个必须是`String`。有关更多信息，请查阅AspectJ编程指南的“ [语言语义”](https://www.eclipse.org/aspectj/doc/released/progguide/semantics-pointcuts.html)部分。

以下示例显示了一些常见的切入点表达式：

- 任何公共方法的执行：

  ```
      执行（公开* *（..））
  ```

- 名称以开头的任何方法的执行`set`：

  ```
      执行（*设置*（..））
  ```

- `AccountService`接口定义的任何方法的执行：

  ```
      执行（* com.xyz.service.AccountService。*（..））
  ```

- `service`包中定义的任何方法的执行：

  ```
      执行（* com.xyz.service。*。*（..））
  ```

- 服务包或其子包之一中定义的任何方法的执行：

  ```
      执行（* com.xyz.service .. *。*（..））
  ```

- 服务包中的任何连接点（仅在Spring AOP中执行方法）：

  ```
      内部（com.xyz.service。*）
  ```

- 服务包或其子包之一中的任何连接点（仅在Spring AOP中执行方法）：

  ```
      内部（com.xyz.service .. *）
  ```

- 代理实现`AccountService`接口的任何连接点（仅在Spring AOP中执行方法） ：

  ```
      this（com.xyz.service.AccountService）
  ```

  |      | “ this”通常以绑定形式使用。有关 如何在建议正文中使代理对象可用的信息，请参阅“ [声明建议](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-advice) ”部分。 |
  | ---- | ------------------------------------------------------------ |
  |      |                                                              |

- 目标对象实现`AccountService`接口的任何连接点（仅在Spring AOP中执行方法）：

  ```
      目标（com.xyz.service.AccountService）
  ```

  |      | “目标”通常以绑定形式使用。有关如何使目标对象在建议正文中可用的信息，请参见“ [声明建议”](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-advice)部分。 |
  | ---- | ------------------------------------------------------------ |
  |      |                                                              |

- 任何采用单个参数且运行时传递的参数为的连接点（仅在Spring AOP中是方法执行）`Serializable`：

  ```
      args（java.io.Serializable）
  ```

  |      | “ args”更通常以绑定形式使用。有关如何使方法参数在建议正文中可用的信息，请参见“ [声明建议”](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-advice)部分。 |
  | ---- | ------------------------------------------------------------ |
  |      |                                                              |

  请注意，此示例中给出的切入点不同于`execution(* *(java.io.Serializable))`。如果在运行时传递的参数为`Serializable`，则args版本匹配 ，如果方法签名声明单个type类型的参数，则执行版本匹配`Serializable`。

- 目标对象带有`@Transactional`注释的任何连接点（仅在Spring AOP中执行方法） ：

  ```
      @target（org.springframework.transaction.annotation.Transactional）
  ```

  |      | 您也可以在绑定形式中使用“ @target”。有关如何使注释对象在建议正文中可用的信息，请参见“ [声明建议”](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-advice)部分。 |
  | ---- | ------------------------------------------------------------ |
  |      |                                                              |

- 目标对象的声明类型具有`@Transactional`注释的任何连接点（仅在Spring AOP中执行方法）：

  ```
      @within（org.springframework.transaction.annotation.Transactional）
  ```

  |      | 您也可以在绑定形式中使用“ @within”。有关如何使注释对象在建议正文中可用的信息，请参见“ [声明建议”](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-advice)部分。 |
  | ---- | ------------------------------------------------------------ |
  |      |                                                              |

- 执行方法带有`@Transactional`注释的任何连接点（仅在Spring AOP中为方法执行） ：

  ```
      @annotation（org.springframework.transaction.annotation.Transactional）
  ```

  |      | 您也可以在绑定形式中使用“ @annotation”。有关如何使注释对象在建议正文中可用的信息，请参见“ [声明建议”](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-advice)部分。 |
  | ---- | ------------------------------------------------------------ |
  |      |                                                              |

- 任何采用单个参数的联接点（仅在Spring AOP中是方法执行），并且传递的参数的运行时类型带有`@Classified`注释：

  ```
      @args（com.xyz.security.Classified）
  ```

  |      | 您也可以在绑定形式中使用“ @args”。请参见“ [声明建议”](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-advice)部分，如何使[建议](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-advice)对象中的注释对象可用。 |
  | ---- | ------------------------------------------------------------ |
  |      |                                                              |

- 名为`tradeService`：的Spring bean上的任何连接点（仅在Spring AOP中执行方法） ：

  ```
      豆（tradeService）
  ```

- 名称与通配符表达式匹配的Spring bean上的任何连接点（仅在Spring AOP中才执行方法）`*Service`：

  ```
      bean（*服务）
  ```

##### 编写好的切入点

在编译期间，AspectJ处理切入点以优化匹配性能。检查代码并确定每个连接点是否（静态或动态）匹配给定的切入点是一个昂贵的过程。（动态匹配意味着无法从静态分析中完全确定匹配，并且在代码中进行了测试以确定在运行代码时是否存在实际匹配）。首次遇到切入点声明时，AspectJ将其重写为匹配过程的最佳形式。这是什么意思？基本上，切入点以DNF（析取范式）重写，并且对切入点的组件进行排序，以便首先检查那些较便宜的组件。

但是，AspectJ只能使用所告诉的内容。为了获得最佳的匹配性能，您应该考虑他们试图实现的目标，并在定义中尽可能缩小匹配的搜索空间。现有的指示符自然属于以下三类之一：同类，作用域和上下文：

- Kinded代号选择特定类型的连接点： `execution`，`get`，`set`，`call`，和`handler`。
- 作用域指定者选择一组感兴趣的连接点（可能有多种）：`within`和`withincode`
- 语境指示符基于上下文匹配（和任选的绑定）： ， `this`，`target`和`@annotation`

编写正确的切入点至少应包括前两种类型（种类和作用域）。您可以包括上下文指示符以根据连接点上下文进行匹配，也可以绑定该上下文以在建议中使用。仅提供同类的标识符或仅提供上下文的标识符是可行的，但是由于额外的处理和分析，可能会影响编织性能（使用的时间和内存）。范围指示符的匹配非常快，使用它们的使用意味着AspectJ可以非常迅速地消除不应进一步处理的连接点组。一个好的切入点应始终包括一个切入点。

#### 5.4.4。宣告建议

建议与切入点表达式关联，并且在切入点匹配的方法执行之前，之后或周围运行。切入点表达式可以是对命名切入点的简单引用，也可以是就地声明的切入点表达式。

##### 咨询前

您可以使用`@Before`注释在方面中在建议之前声明：

爪哇

科特林

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }

}
```

如果使用就地切入点表达式，则可以将前面的示例重写为以下示例：

爪哇

科特林

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("execution(* com.xyz.myapp.dao.*.*(..))")
    public void doAccessCheck() {
        // ...
    }

}
```

##### 返回建议后

返回建议后，当匹配的方法执行正常返回时，运行建议。您可以使用`@AfterReturning`批注进行声明：

爪哇

科特林

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }

}
```

|      | 您可以在同一方面内拥有多个建议声明（以及其他成员）。在这些示例中，我们仅显示单个建议声明，以集中每个建议的效果。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

有时，您需要在建议正文中访问返回的实际值。您可以使用`@AfterReturning`绑定返回值的形式来获取该访问权限，如以下示例所示：

爪哇

科特林

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning(
        pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }

}
```

`returning`属性中使用的名称必须与advice方法中的参数名称相对应。当方法执行返回时，返回值将作为相应的参数值传递到通知方法。甲`returning`子句也限制了只能匹配到返回指定类型的值（在这种情况下，那些方法执行`Object`，它匹配任何返回值）。

请注意，返回建议后使用时，不可能返回完全不同的参考。

##### 投掷建议后

抛出建议后，当匹配的方法执行通过抛出异常退出时运行建议。您可以使用`@AfterThrowing`批注进行声明，如以下示例所示：

爪哇

科特林

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doRecoveryActions() {
        // ...
    }

}
```

通常，您希望通知仅在引发给定类型的异常时才运行，并且您通常还需要访问通知正文中的引发异常。您可以使用该 `throwing`属性来限制匹配（如果需要，请使用- `Throwable`否则用作异常类型），并将抛出的异常绑定到advice参数。以下示例显示了如何执行此操作：

爪哇

科特林

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing(
        pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
        throwing="ex")
    public void doRecoveryActions(DataAccessException ex) {
        // ...
    }

}
```

`throwing`属性中使用的名称必须与advice方法中的参数名称相对应。当通过引发异常退出方法执行时，该异常将作为相应的参数值传递给通知方法。甲`throwing` 子句也限制了只能匹配到抛出指定类型的异常（那些方法执行`DataAccessException`，在这种情况下）。

##### 经过（最后）建议

当匹配的方法执行退出时，通知（最终）运行。通过使用`@After`批注进行声明。之后必须准备处理正常和异常返回条件的建议。它通常用于释放资源和类似目的。以下示例显示了最终建议后的用法：

爪哇

科特林

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.After;

@Aspect
public class AfterFinallyExample {

    @After("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doReleaseLock() {
        // ...
    }

}
```

##### 忠告

最后一种建议是围绕建议。围绕建议在匹配方法的执行过程中“围绕”运行。它有机会在方法执行之前和之后进行工作，并确定何时，如何以及什至根本不执行该方法。如果需要以线程安全的方式（例如，启动和停止计时器）在方法执行之前和之后共享状态，则通常使用环绕建议。始终使用最不符合要求的建议形式（即，在建议可以使用之前，不要在建议周围使用）。

通过使用`@Around`注释来声明周围建议。咨询方法的第一个参数必须为类型`ProceedingJoinPoint`。在通知的主体，要求`proceed()`对`ProceedingJoinPoint`原因的基本方法来执行。该`proceed`方法还可以传递`Object[]`。数组中的值用作方法执行时的参数。

|      | 的行为`proceed`时有一个叫`Object[]`比的行为有点不同`proceed`around建议通过AspectJ编译器。对于使用传统的AspectJ语言编写的周围建议，通过传递给的参数数量 `proceed`其的参数数量必须与传递给环绕通知的参数数量（而不是基础连接点采用的参数数量）相匹配，并且传递给定值以在给定条件下继续进行参数位置会取代该值绑定到的实体的连接点处的原始值（不要担心，如果这现在没有意义）。Spring采取的方法更简单，并且更适合其基于代理的，仅执行的语义。如果您编译为Spring编写的@AspectJ方面并使用，则只需要意识到这种区别即可。`proceed`与AspectJ编译器和编织器一起使用的参数。有一种方法可以在Spring AOP和AspectJ之间100％兼容，并且在[下面有关建议参数的部分中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj-advice-params)对此进行了讨论 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下示例显示了如何使用周围建议：

爪哇

科特林

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
public class AroundExample {

    @Around("com.xyz.myapp.SystemArchitecture.businessService()")
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
        // start stopwatch
        Object retVal = pjp.proceed();
        // stop stopwatch
        return retVal;
    }

}
```

周围建议返回的值是该方法的调用者看到的返回值。例如，一个简单的缓存方面可以从一个缓存中返回一个值（如果有一个值），然后调用一个`proceed()`没有的值。请注意，`proceed`在周围建议的正文中可能调用一次，多次调用或根本不调用。所有这些都是合法的。

##### 建议参数

Spring提供了完全类型化的建议，这意味着您可以在建议签名中声明所需的参数（如我们先前在返回和抛出示例中所见），而不是一直使用`Object[]`数组。我们将在本节的后面部分介绍如何使参数和其他上下文值可用于建议主体。首先，我们看一下如何编写通用建议，以了解该建议当前建议的方法。

###### 访问当前 `JoinPoint`

任何通知方法都可以将类型的参数声明为它的第一个参数 `org.aspectj.lang.JoinPoint`（请注意，必须在通知周围声明类型的第一个参数`ProceedingJoinPoint`，它是的子类`JoinPoint`。该 `JoinPoint`接口提供了许多有用的方法：

- `getArgs()`：返回方法参数。
- `getThis()`：返回代理对象。
- `getTarget()`：返回目标对象。
- `getSignature()`：返回建议使用的方法的描述。
- `toString()`：打印有关所建议方法的有用说明。

有关更多详细信息，请参见[javadoc](https://www.eclipse.org/aspectj/doc/released/runtime-api/org/aspectj/lang/JoinPoint.html)。

###### 将参数传递给建议

我们已经看到了如何绑定返回的值或异常值（在返回和引发建议之后使用）。要使参数值可用于建议正文，可以使用的绑定形式`args`。如果在args表达式中使用参数名称代替类型名称，则在调用建议时会将相应参数的值作为参数值传递。一个例子应该使这一点更清楚。假设您想建议以`Account` 对象为第一个参数的DAO操作的执行，并且您需要访问建议正文中的帐户。您可以编写以下内容：

爪哇

科特林

```java
@Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation() && args(account,..)")
public void validateAccount(Account account) {
    // ...
}
```

`args(account,..)`切入点表达式的一部分用于两个目的。首先，它将匹配限制为仅方法采用至少一个参数且传递给该参数的参数为的实例的方法执行`Account`。其次，它`Account`通过`account` 参数使实际对象可用于建议。

编写此代码的另一种方法是声明一个切入点，`Account` 当切入点与连接点匹配时“提供” 对象值，然后从通知中引用命名的切入点。如下所示：

爪哇

科特林

```java
@Pointcut("com.xyz.myapp.SystemArchitecture.dataAccessOperation() && args(account,..)")
private void accountDataAccessOperation(Account account) {}

@Before("accountDataAccessOperation(account)")
public void validateAccount(Account account) {
    // ...
}
```

有关更多详细信息，请参见AspectJ编程指南。

代理对象（`this`），目标对象（`target`），和说明（`@within`， `@target`，`@annotation`，和`@args`）都可以以类似的方式结合。接下来的两个示例显示了如何匹配使用注释注释的方法的执行 `@Auditable`并提取审计代码：

这两个示例中的第一个示例显示了`@Auditable`注释的定义：

爪哇

科特林

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Auditable {
    AuditCode value();
}
```

这两个示例中的第二个示例显示了与`@Auditable`方法执行相匹配的建议：

爪哇

科特林

```java
@Before("com.xyz.lib.Pointcuts.anyPublicMethod() && @annotation(auditable)")
public void audit(Auditable auditable) {
    AuditCode code = auditable.value();
    // ...
}
```

###### 咨询参数和泛型

Spring AOP可以处理类声明和方法参数中使用的泛型。假设您具有如下通用类型：

爪哇

科特林

```java
public interface Sample<T> {
    void sampleGenericMethod(T param);
    void sampleGenericCollectionMethod(Collection<T> param);
}
```

您可以通过在要拦截方法的参数类型中键入advice参数，将方法类型的拦截限制为某些参数类型：

爪哇

科特林

```java
@Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
public void beforeSampleMethod(MyType param) {
    // Advice implementation
}
```

这种方法不适用于通用集合。因此，您不能按以下方式定义切入点：

爪哇

科特林

```java
@Before("execution(* ..Sample+.sampleGenericCollectionMethod(*)) && args(param)")
public void beforeSampleMethod(Collection<MyType> param) {
    // Advice implementation
}
```

为了使这项工作有效，我们将不得不检查集合中的每个元素，这是不合理的，因为我们也无法决定`null`总体上如何对待价值。要实现与此类似的功能，您必须在上键入参数`Collection`并手动检查元素的类型。

###### 确定参数名称

通知调用中的参数绑定依赖于切入点表达式中使用的名称与通知和切入点方法签名中声明的参数名称的匹配。通过Java反射无法获得参数名称，因此Spring AOP使用以下策略来确定参数名称：

- 如果用户已明确指定参数名称，则使用指定的参数名称。建议和切入点注释均具有可选`argNames`属性，您可以使用该属性指定带注释方法的参数名称。这些参数名称在运行时可用。以下示例显示如何使用`argNames`属性：

爪哇

科特林

```java
@Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
        argNames="bean,auditable")
public void audit(Object bean, Auditable auditable) {
    AuditCode code = auditable.value();
    // ... use code and bean
}
```

如果第一个参数是的`JoinPoint`，`ProceedingJoinPoint`或 `JoinPoint.StaticPart`类型，你可以从价值离开了参数的名称`argNames`属性。例如，如果您修改前面的建议以接收连接点对象，则该`argNames`属性不必包含它：

爪哇

科特林

```java
@Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
        argNames="bean,auditable")
public void audit(JoinPoint jp, Object bean, Auditable auditable) {
    AuditCode code = auditable.value();
    // ... use code, bean, and jp
}
```

给出的第一个参数的特殊待遇`JoinPoint`， `ProceedingJoinPoint`和`JoinPoint.StaticPart`类型是不收取任何其它连接上下文的通知情况下，特别方便。在这种情况下，您可以忽略该`argNames`属性。例如，以下建议无需声明`argNames`属性：

爪哇

科特林

```java
@Before("com.xyz.lib.Pointcuts.anyPublicMethod()")
public void audit(JoinPoint jp) {
    // ... use jp
}
```

- 使用该`'argNames'`属性有点笨拙，因此，如果`'argNames'`未指定该属性，Spring AOP将查看该类的调试信息，并尝试从局部变量表中确定参数名称。只要已使用调试信息编译了类（`'-g:vars'`至少），该信息就会存在。启用此标志时进行编译的结果是：（1）您的代码稍微易于理解（反向工程），（2）类文件的大小略大（通常无关紧要），（3）优化以删除未使用的本地文件变量不适用于您的编译器。换句话说，通过启用此标志，您应该不会遇到任何困难。

  |      | 如果即使没有调试信息，AspectJ编译器（ajc）都已编译@AspectJ方面，则无需添加`argNames`属性，因为编译器会保留所需的信息。 |
  | ---- | ------------------------------------------------------------ |
  |      |                                                              |

- 如果在没有必要调试信息的情况下编译了代码，Spring AOP将尝试推断绑定变量与参数的配对（例如，如果切入点表达式中仅绑定了一个变量，并且advice方法仅接受一个参数，则配对很明显）。如果在给定可用信息的情况下变量的绑定不明确，`AmbiguousBindingException`则会引发an 。

- 如果以上所有策略均失败，`IllegalArgumentException`则抛出。

###### 进行论证

前面我们提到，我们将描述如何编写一个`proceed`在Spring AOP和AspectJ中始终有效的参数调用。解决方案是确保建议签名按顺序绑定每个方法参数。以下示例显示了如何执行此操作：

爪哇

科特林

```java
@Around("execution(List<Account> find*(..)) && " +
        "com.xyz.myapp.SystemArchitecture.inDataAccessLayer() && " +
        "args(accountHolderNamePattern)")
public Object preProcessQueryPattern(ProceedingJoinPoint pjp,
        String accountHolderNamePattern) throws Throwable {
    String newPattern = preProcess(accountHolderNamePattern);
    return pjp.proceed(new Object[] {newPattern});
}
```

在许多情况下，无论如何都要进行此绑定（如上例所示）。

##### 咨询订购

当多条建议都希望在同一连接点上运行时会发生什么？Spring AOP遵循与AspectJ相同的优先级规则来确定建议执行的顺序。优先级最高的建议首先“在途中”运行（因此，考虑到两条先行建议，优先级最高的建议首先运行）。从连接点“出路”时，优先级最高的建议将最后运行（因此，给定两条后置通知，优先级最高的建议将第二次运行）。

当在不同方面定义的两条建议都需要在同一连接点上运行时，除非另行指定，否则执行顺序是不确定的。您可以通过指定优先级来控制执行顺序。通过`org.springframework.core.Ordered`在方面类中实现接口或使用注释对其进行`Order`注释，可以通过常规的Spring方式完成此操作。给定两个方面，从中返回较低值`Ordered.getValue()`（或注释值）的方面具有较高的优先级。

当在同一方面定义的两条建议都需要在同一连接点上运行时，其顺序是未定义的（因为无法通过反射来获取javac编译类的声明顺序）。考虑将这些建议方法折叠为每个方面类中每个连接点的一个建议方法，或将建议重构为可在方面级别订购的单独方面类。

#### 5.4.5。引言

简介（在AspectJ中称为类型间声明）使方面可以声明建议对象实现给定的接口，并代表那些对象提供该接口的实现。

您可以使用`@DeclareParents`注释进行介绍。此批注用于声明匹配类型具有新的父代（因此而得名）。例如，给定名为的接口`UsageTracked`和名为的接口的实现`DefaultUsageTracked`，以下方面声明服务接口的所有实现者也都实现该`UsageTracked`接口（例如，通过JMX公开统计信息）：

爪哇

科特林

```java
@Aspect
public class UsageTracking {

    @DeclareParents(value="com.xzy.myapp.service.*+", defaultImpl=DefaultUsageTracked.class)
    public static UsageTracked mixin;

    @Before("com.xyz.myapp.SystemArchitecture.businessService() && this(usageTracked)")
    public void recordUsage(UsageTracked usageTracked) {
        usageTracked.incrementUseCount();
    }

}
```

要实现的接口由带注释的字段的类型确定。注释的 `value`属性`@DeclareParents`是AspectJ类型的模式。任何匹配类型的bean都会实现该`UsageTracked`接口。注意，在前面示例的建议中，服务Bean可以直接用作`UsageTracked`接口的实现。如果以编程方式访问bean，则应编写以下内容：

爪哇

科特林

```java
UsageTracked usageTracked = (UsageTracked) context.getBean("myService");
```

#### 5.4.6。方面实例化模型

|      | 这是一个高级主题。如果您刚开始使用AOP，则可以放心地跳过它，直到以后。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

默认情况下，应用程序上下文中每个方面都有一个实例。AspectJ将此称为单例实例化模型。可以使用备用生命周期来定义方面。Spring支持AspectJ的`perthis`和`pertarget` 实例化模型（`percflow, percflowbelow,`和`pertypewithin`目前不支持）。

您可以`perthis`通过`perthis`在`@Aspect` 注释中指定子句来声明方面。考虑以下示例：

爪哇

科特林

```java
@Aspect("perthis(com.xyz.myapp.SystemArchitecture.businessService())")
public class MyAspect {

    private int someState;

    @Before(com.xyz.myapp.SystemArchitecture.businessService())
    public void recordServiceUsage() {
        // ...
    }

}
```

在前面的示例中，该`'perthis'`子句的作用是为每个执行业务服务的唯一服务对象（每个与切入点表达式匹配的联接点绑定到“ this”的唯一对象）创建一个方面实例。方面实例是在服务对象上首次调用方法时创建的。当服务对象超出范围时，方面将超出范围。在创建方面实例之前，其中的任何建议都不会执行。创建方面实例后，在其中声明的建议将在匹配的连接点处执行，但仅当服务对象是与此方面相关联的对象时才执行。有关`per`条款的更多信息，请参见AspectJ编程指南。

该`pertarget`实例化样板工程完全相同的方式`perthis`，但在匹配的连接点，每个独立目标对象创建一个切面实例。

#### 5.4.7。AOP示例

既然您已经了解了所有组成部分的工作方式，那么我们可以将它们组合在一起以做一些有用的事情。

有时由于并发问题（例如，死锁失败者），业务服务的执行可能会失败。如果重试该操作，则很可能在下一次尝试中成功。对于适合在这种情况下重试的业务服务（不需要为解决冲突而需要返回给用户的幂等操作），我们希望透明地重试该操作以避免客户端看到 `PessimisticLockingFailureException`。这项要求明确地跨越了服务层中的多个服务，因此非常适合通过一个方面实施。

因为我们想重试该操作，所以我们需要使用“周围”建议，以便我们可以`proceed`多次调用。以下清单显示了基本方面的实现：

爪哇

科特林

```java
@Aspect
public class ConcurrentOperationExecutor implements Ordered {

    private static final int DEFAULT_MAX_RETRIES = 2;

    private int maxRetries = DEFAULT_MAX_RETRIES;
    private int order = 1;

    public void setMaxRetries(int maxRetries) {
        this.maxRetries = maxRetries;
    }

    public int getOrder() {
        return this.order;
    }

    public void setOrder(int order) {
        this.order = order;
    }

    @Around("com.xyz.myapp.SystemArchitecture.businessService()")
    public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
        int numAttempts = 0;
        PessimisticLockingFailureException lockFailureException;
        do {
            numAttempts++;
            try {
                return pjp.proceed();
            }
            catch(PessimisticLockingFailureException ex) {
                lockFailureException = ex;
            }
        } while(numAttempts <= this.maxRetries);
        throw lockFailureException;
    }

}
```

请注意，方面实现了`Ordered`接口，因此我们可以将方面的优先级设置为高于事务建议（每次重试时都希望有新的事务）。该`maxRetries`和`order`性能都Spring配置。主要动作发生在`doConcurrentOperation`周围建议中。请注意，目前，我们将重试逻辑应用于每个`businessService()`。我们尝试继续，如果失败了`PessimisticLockingFailureException`，我们将再次尝试，除非我们用尽了所有的重试尝试。

相应的Spring配置如下：

```xml
<aop:aspectj-autoproxy/>

<bean id="concurrentOperationExecutor" class="com.xyz.myapp.service.impl.ConcurrentOperationExecutor">
    <property name="maxRetries" value="3"/>
    <property name="order" value="100"/>
</bean>
```

为了完善方面，使其仅重试幂等操作，我们可以定义以下 `Idempotent`注释：

爪哇

科特林

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Idempotent {
    // marker annotation
}
```

然后，我们可以使用注释来注释服务操作的实现。更改为仅重试幂等操作的方面涉及改进切入点表达式，以便仅`@Idempotent`操作匹配，如下所示：

爪哇

科特林

```java
@Around("com.xyz.myapp.SystemArchitecture.businessService() && " +
        "@annotation(com.xyz.myapp.service.Idempotent)")
public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
    // ...
}
```

### 5.5。基于架构的AOP支持

如果您更喜欢基于XML的格式，Spring还提供了对使用新`aop`名称空间标签定义方面的支持。支持与使用@AspectJ样式时完全相同的切入点表达式和建议类型。因此，在本节中，我们将重点放在新语法上，并使读者参考上一节中的讨论（[@AspectJ support](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj)），以了解编写切入点表达式和建议参数的绑定。

要使用本节中描述的aop名称空间标记，您需要导入 `spring-aop`模式，如[基于XML Schema的配置中所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-schemas)。请参阅[AOP模式](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-schemas-aop) 以了解如何在`aop`名称空间中导入标签。

在您的Spring配置中，所有方面和顾问程序元素都必须放在一个``元素内（``在应用程序上下文配置中可以有多个元素）。一个``元素可以包含切入点，顾问和纵横元件（注意这些必须按照这个顺序进行声明）。

|      | 该``风格的配置使得大量使用Spring的的 [自动代理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-autoproxy)机制。如果您已经通过使用`BeanNameAutoProxyCreator`或类似方法使用显式自动代理，则可能会导致问题（例如，未编制建议） 。推荐的使用模式是为使用唯一的``样式或仅`AutoProxyCreator`风格，从不混合。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 5.5.1。声明一个方面

使用模式支持时，方面是在Spring应用程序上下文中定义为Bean的常规Java对象。状态和行为在对象的字段和方法中捕获，切入点和建议信息在XML中捕获。

您可以使用<aop：aspect>元素声明一个方面，并通过使用该`ref`属性来引用支持Bean ，如以下示例所示：

```xml
<aop:config>
    <aop:aspect id="myAspect" ref="aBean">
        ...
    </aop:aspect>
</aop:config>

<bean id="aBean" class="...">
    ...
</bean>
```

支持方面的Bean（`aBean`在这种情况下）当然可以像配置任何其他Spring Bean一样进行配置并注入依赖项。

#### 5.5.2。声明切入点

您可以在``元素内声明命名的切入点，从而使切入点定义在多个方面和顾问程序之间共享。

可以定义代表服务层中任何业务服务的执行的切入点：

```xml
<aop:config>

    <aop:pointcut id="businessService"
        expression="execution(* com.xyz.myapp.service.*.*(..))"/>

</aop:config>
```

请注意，切入点表达式本身使用的是[@AspectJ support中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj)描述的AspectJ切入点表达式语言。如果使用基于架构的声明样式，则可以引用在切入点表达式中的类型（@Aspects）中定义的命名切入点。定义上述切入点的另一种方法如下：

```xml
<aop:config>

    <aop:pointcut id="businessService"
        expression="com.xyz.myapp.SystemArchitecture.businessService()"/>

</aop:config>
```

假定您具有“ [共享公共切入点定义”中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-common-pointcuts)`SystemArchitecture`所述的一个方面。

然后，在方面中声明切入点与声明顶级切入点非常相似，如以下示例所示：

```xml
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..))"/>

        ...

    </aop:aspect>

</aop:config>
```

与@AspectJ方面几乎相同，使用基于架构的定义样式声明的切入点可以收集连接点上下文。例如，以下切入点将`this`对象收集为连接点上下文，并将其传递给建议：

```xml
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..)) && this(service)"/>

        <aop:before pointcut-ref="businessService" method="monitor"/>

        ...

    </aop:aspect>

</aop:config>
```

必须声明通知，以通过包含匹配名称的参数来接收收集的连接点上下文，如下所示：

爪哇

科特林

```java
public void monitor(Object service) {
    // ...
}
```

当需要连接子表达式，`&&`是一个XML文档中的尴尬，这样你就可以使用`and`，`or`以及`not`到位的关键字`&&`，`||`和`!`分别。例如，上一个切入点可以更好地编写如下：

```xml
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..)) and this(service)"/>

        <aop:before pointcut-ref="businessService" method="monitor"/>

        ...
    </aop:aspect>
</aop:config>
```

请注意，以这种方式定义的切入点由其XML引用，`id`不能用作命名切入点以形成复合切入点。因此，基于架构的定义样式中的命名切入点支持比@AspectJ样式所提供的更受限制。

#### 5.5.3。宣告建议

基于模式的AOP支持使用与@AspectJ样式相同的五种建议，并且它们具有完全相同的语义。

##### 咨询前

在匹配的方法执行之前运行建议之前。``使用<aop：before>元素在里面声明它 ，如以下示例所示：

```xml
<aop:aspect id="beforeExample" ref="aBean">

    <aop:before
        pointcut-ref="dataAccessOperation"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```

这`dataAccessOperation`是`id`在顶层（``）定义的切入点的。要改为定义切入点内联，请用`pointcut-ref`属性替换`pointcut`属性，如下所示：

```xml
<aop:aspect id="beforeExample" ref="aBean">

    <aop:before
        pointcut="execution(* com.xyz.myapp.dao.*.*(..))"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```

正如我们在@AspectJ样式的讨论中所指出的那样，使用命名的切入点可以显着提高代码的可读性。

该`method`属性标识`doAccessCheck`提供建议正文的方法（）。必须为包含建议的Aspect元素所引用的bean定义此方法。在执行数据访问操作（与切入点表达式匹配的方法执行连接点）之前，将`doAccessCheck`调用Aspect Bean上的方法。

##### 返回建议后

返回的建议在匹配的方法执行正常完成时运行。在内部以``与之前建议相同的方式声明它。以下示例显示了如何声明它：

```xml
<aop:aspect id="afterReturningExample" ref="aBean">

    <aop:after-returning
        pointcut-ref="dataAccessOperation"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```

与@AspectJ样式一样，您可以在建议正文中获取返回值。为此，使用returning属性指定返回值应传递到的参数的名称，如以下示例所示：

```xml
<aop:aspect id="afterReturningExample" ref="aBean">

    <aop:after-returning
        pointcut-ref="dataAccessOperation"
        returning="retVal"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```

该`doAccessCheck`方法必须声明一个名为的参数`retVal`。该参数的类型以与所述相同的方式约束匹配`@AfterReturning`。例如，您可以声明方法签名，如下所示：

爪哇

科特林

```java
public void doAccessCheck(Object retVal) {...
```

##### 投掷建议后

抛出建议后，当匹配的方法执行通过抛出异常退出时执行建议。``通过使用after-throwing元素在里面声明它，如以下示例所示：

```xml
<aop:aspect id="afterThrowingExample" ref="aBean">

    <aop:after-throwing
        pointcut-ref="dataAccessOperation"
        method="doRecoveryActions"/>

    ...

</aop:aspect>
```

与@AspectJ样式一样，您可以在通知正文中获取引发的异常。为此，请使用throwing属性指定异常应传递到的参数的名称，如以下示例所示：

```xml
<aop:aspect id="afterThrowingExample" ref="aBean">

    <aop:after-throwing
        pointcut-ref="dataAccessOperation"
        throwing="dataAccessEx"
        method="doRecoveryActions"/>

    ...

</aop:aspect>
```

该`doRecoveryActions`方法必须声明一个名为的参数`dataAccessEx`。该参数的类型以与所述相同的方式约束匹配`@AfterThrowing`。例如，方法签名可以声明如下：

爪哇

科特林

```java
public void doRecoveryActions(DataAccessException dataAccessEx) {...
```

##### 经过（最后）建议

无论最终如何执行匹配的方法，建议（最终）都会运行。您可以使用`after`元素声明它，如以下示例所示：

```xml
<aop:aspect id="afterFinallyExample" ref="aBean">

    <aop:after
        pointcut-ref="dataAccessOperation"
        method="doReleaseLock"/>

    ...

</aop:aspect>
```

##### 忠告

最后一种建议是围绕建议。围绕建议在匹配的方法执行“周围”运行。它有机会在方法执行之前和之后进行工作，并确定何时，如何以及什至根本不执行该方法。围绕建议通常用于以线程安全的方式（例如，启动和停止计时器）在方法执行之前和之后共享状态。始终使用最不强大的建议形式，以满足您的要求。如果建议可以完成这项工作，请不要在建议周围使用。

您可以使用`aop:around`元素在建议周围进行声明。咨询方法的第一个参数必须为类型`ProceedingJoinPoint`。在通知的主体，要求`proceed()`对`ProceedingJoinPoint`原因的基本方法来执行。`proceed`也可以使用调用该方法`Object[]`。数组中的值用作方法执行时的参数。见 [around通知](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj-around-advice)的注意事项调用`proceed`有`Object[]`。以下示例显示了如何在XML中围绕建议声明：

```xml
<aop:aspect id="aroundExample" ref="aBean">

    <aop:around
        pointcut-ref="businessService"
        method="doBasicProfiling"/>

    ...

</aop:aspect>
```

该`doBasicProfiling`建议的实现可以与@AspectJ示例完全相同（当然要减去注释），如以下示例所示：

爪哇

科特林

```java
public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
    // start stopwatch
    Object retVal = pjp.proceed();
    // stop stopwatch
    return retVal;
}
```

##### 建议参数

基于架构的声明样式以与@AspectJ支持相同的方式支持完全类型的建议，即通过名称与建议方法参数匹配切入点参数。有关详细信息，请参见[建议参数](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj-advice-params)。如果您希望为建议方法明确指定参数名称（不依赖于先前描述的检测策略），则可以通过使用`arg-names` advice元素的属性来实现，该属性的处理方式与`argNames` 建议批注中的属性相同（如[确定参数名称中所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj-advice-params-names)）。以下示例显示如何在XML中指定参数名称：

```xml
<aop:before
    pointcut="com.xyz.lib.Pointcuts.anyPublicMethod() and @annotation(auditable)"
    method="audit"
    arg-names="auditable"/>
```

该`arg-names`属性接受以逗号分隔的参数名称列表。

以下基于XSD的方法中涉及程度稍高的示例显示了一些与一些强类型参数结合使用的建议：

爪哇

科特林

```java
package x.y.service;

public interface PersonService {

    Person getPerson(String personName, int age);
}

public class DefaultFooService implements FooService {

    public Person getPerson(String name, int age) {
        return new Person(name, age);
    }
}
```

接下来是方面。请注意，该`profile(..)`方法接受许多强类型参数，这一事实恰好是用于进行方法调用的连接点。此参数的存在指示 `profile(..)`将使用用作`around`建议，如以下示例所示：

爪哇

科特林

```java
package x.y;

import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.util.StopWatch;

public class SimpleProfiler {

    public Object profile(ProceedingJoinPoint call, String name, int age) throws Throwable {
        StopWatch clock = new StopWatch("Profiling for '" + name + "' and '" + age + "'");
        try {
            clock.start(call.toShortString());
            return call.proceed();
        } finally {
            clock.stop();
            System.out.println(clock.prettyPrint());
        }
    }
}
```

最后，以下示例XML配置影响了特定连接点的上述建议的执行：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- this is the object that will be proxied by Spring's AOP infrastructure -->
    <bean id="personService" class="x.y.service.DefaultPersonService"/>

    <!-- this is the actual advice itself -->
    <bean id="profiler" class="x.y.SimpleProfiler"/>

    <aop:config>
        <aop:aspect ref="profiler">

            <aop:pointcut id="theExecutionOfSomePersonServiceMethod"
                expression="execution(* x.y.service.PersonService.getPerson(String,int))
                and args(name, age)"/>

            <aop:around pointcut-ref="theExecutionOfSomePersonServiceMethod"
                method="profile"/>

        </aop:aspect>
    </aop:config>

</beans>
```

考虑以下驱动程序脚本：

爪哇

科特林

```java
import org.springframework.beans.factory.BeanFactory;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import x.y.service.PersonService;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        BeanFactory ctx = new ClassPathXmlApplicationContext("x/y/plain.xml");
        PersonService person = (PersonService) ctx.getBean("personService");
        person.getPerson("Pengo", 12);
    }
}
```

有了这样的Boot类，我们将在标准输出上获得类似于以下内容的输出：

```
秒表“分析”“ Pengo”和“ 12”：运行时间（毫秒）= 0
-----------------------------------------
ms％任务名称
-----------------------------------------
00000？执行（getFoo）
```

##### 咨询订购

当需要在同一连接点（执行方法）上执行多个建议时，排序规则如“ [建议排序”中所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj-advice-ordering)。方面之间的优先级是通过将`Order`注释添加到支持方面的Bean或通过使Bean实现`Ordered`接口来确定的。

#### 5.5.4。引言

简介（在AspectJ中称为类型间声明）使方面可以声明建议的对象实现给定的接口，并代表那些对象提供该接口的实现。

您可以通过使用中的`aop:declare-parents`元素进行介绍`aop:aspect`。您可以使用`aop:declare-parents`元素来声明匹配类型具有新的父对象（因此为名称）。例如，给定名为的接口`UsageTracked`和名为的接口的实现 `DefaultUsageTracked`，以下方面声明服务接口的所有实现者也都实现该`UsageTracked`接口。（例如，为了通过JMX公开统计信息。）

```xml
<aop:aspect id="usageTrackerAspect" ref="usageTracking">

    <aop:declare-parents
        types-matching="com.xzy.myapp.service.*+"
        implement-interface="com.xyz.myapp.service.tracking.UsageTracked"
        default-impl="com.xyz.myapp.service.tracking.DefaultUsageTracked"/>

    <aop:before
        pointcut="com.xyz.myapp.SystemArchitecture.businessService()
            and this(usageTracked)"
            method="recordUsage"/>

</aop:aspect>
```

支持`usageTracking`bean 的类将包含以下方法：

爪哇

科特林

```java
public void recordUsage(UsageTracked usageTracked) {
    usageTracked.incrementUseCount();
}
```

要实现的接口由`implement-interface`属性确定。该`types-matching`属性的值是AspectJ类型的模式。任何匹配类型的bean都会实现该`UsageTracked`接口。注意，在前面示例的建议中，服务Bean可以直接用作`UsageTracked`接口的实现。要以编程方式访问bean，可以编写以下代码：

爪哇

科特林

```java
UsageTracked usageTracked = (UsageTracked) context.getBean("myService");
```

#### 5.5.5。方面实例化模型

模式定义方面唯一受支持的实例化模型是单例模型。在将来的版本中可能会支持其他实例化模型。

#### 5.5.6。顾问

“顾问”的概念来自Spring中定义的AOP支持，并且在AspectJ中没有直接等效的概念。顾问就像一个独立的小方面，只有一条建议。通知本身由bean表示，并且必须实现[Spring的“建议类型”中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-api-advice-types)描述的建议接口 [之一](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-api-advice-types)。顾问可以利用AspectJ切入点表达式。

Spring支持带有``元素的顾问程序概念。您最常看到它与事务建议结合使用，事务建议在Spring中也有其自己的名称空间支持。以下示例显示顾问程序：

```xml
<aop:config>

    <aop:pointcut id="businessService"
        expression="execution(* com.xyz.myapp.service.*.*(..))"/>

    <aop:advisor
        pointcut-ref="businessService"
        advice-ref="tx-advice"/>

</aop:config>

<tx:advice id="tx-advice">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```

除了`pointcut-ref`前面的示例中使用的`pointcut`属性，您还可以使用该 属性来内联定义切入点表达式。

要定义顾问程序的优先级，以便该建议书可以参与订购，请使用该`order`属性来定义`Ordered`顾问程序的值。

#### 5.5.7。AOP模式示例

本节说明使用模式支持重写时，[AOP示例中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj-example)的并发锁定失败重试示例 [的](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj-example)外观。

有时由于并发问题（例如，死锁失败者），业务服务的执行可能会失败。如果重试该操作，则很可能在下一次尝试中成功。对于适合在这种情况下重试的业务服务（不需要为解决冲突而需要返回给用户的幂等操作），我们希望透明地重试该操作以避免客户端看到 `PessimisticLockingFailureException`。这项要求明确地跨越了服务层中的多个服务，因此非常适合通过一个方面实施。

因为我们想重试该操作，所以我们需要使用“周围”建议，以便我们可以`proceed`多次调用。以下清单显示了基本方面的实现（这是使用模式支持的常规Java类）：

爪哇

科特林

```java
public class ConcurrentOperationExecutor implements Ordered {

    private static final int DEFAULT_MAX_RETRIES = 2;

    private int maxRetries = DEFAULT_MAX_RETRIES;
    private int order = 1;

    public void setMaxRetries(int maxRetries) {
        this.maxRetries = maxRetries;
    }

    public int getOrder() {
        return this.order;
    }

    public void setOrder(int order) {
        this.order = order;
    }

    public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
        int numAttempts = 0;
        PessimisticLockingFailureException lockFailureException;
        do {
            numAttempts++;
            try {
                return pjp.proceed();
            }
            catch(PessimisticLockingFailureException ex) {
                lockFailureException = ex;
            }
        } while(numAttempts <= this.maxRetries);
        throw lockFailureException;
    }

}
```

请注意，方面实现了`Ordered`接口，因此我们可以将方面的优先级设置为高于事务建议（每次重试时都希望有新的事务）。该`maxRetries`和`order`性能都Spring配置。主要动作发生在“ `doConcurrentOperation`周围建议”方法中。我们尝试继续。如果我们失败了`PessimisticLockingFailureException`，我们将再试一次，除非我们用尽了所有的重试尝试。

|      | 该类与@AspectJ示例中使用的类相同，但是除去了注释。 |
| ---- | -------------------------------------------------- |
|      |                                                    |

相应的Spring配置如下：

```xml
<aop:config>

    <aop:aspect id="concurrentOperationRetry" ref="concurrentOperationExecutor">

        <aop:pointcut id="idempotentOperation"
            expression="execution(* com.xyz.myapp.service.*.*(..))"/>

        <aop:around
            pointcut-ref="idempotentOperation"
            method="doConcurrentOperation"/>

    </aop:aspect>

</aop:config>

<bean id="concurrentOperationExecutor"
    class="com.xyz.myapp.service.impl.ConcurrentOperationExecutor">
        <property name="maxRetries" value="3"/>
        <property name="order" value="100"/>
</bean>
```

请注意，目前，我们假设所有业务服务都是幂等的。如果不是这种情况，我们可以通过引入一个`Idempotent`注释并使用该注释来注释服务操作的实现来完善方面，使其仅重试真正的幂等操作，如以下示例所示：

爪哇

科特林

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Idempotent {
    // marker annotation
}
```

更改为仅重试幂等操作的方面涉及改进切入点表达式，以便仅`@Idempotent`操作匹配，如下所示：

```xml
<aop:pointcut id="idempotentOperation"
        expression="execution(* com.xyz.myapp.service.*.*(..)) and
        @annotation(com.xyz.myapp.service.Idempotent)"/>
```

### 5.6。选择要使用的AOP声明样式

一旦确定方面是实现给定需求的最佳方法，您如何在使用Spring AOP或AspectJ以及在Aspect语言（代码）样式，@ AspectJ批注样式或Spring XML样式之间做出选择？这些决定受许多因素影响，包括应用程序需求，开发工具和团队对AOP的熟悉程度。

#### 5.6.1。Spring AOP还是Full AspectJ？

使用最简单的方法即可。Spring AOP比使用完整的AspectJ更简单，因为不需要在开发和构建过程中引入AspectJ编译器/编织器。如果您只需要建议在Spring bean上执行操作，则Spring AOP是正确的选择。如果您需要建议不受Spring容器管理的对象（通常是域对象），则需要使用AspectJ。如果您希望建议除简单方法执行之外的连接点（例如，字段获取或设置连接点等），则还需要使用AspectJ。

使用AspectJ时，可以选择AspectJ语言语法（也称为“代码样式”）或@AspectJ注释样式。显然，如果您不使用Java 5+，则可以为您做出选择：使用代码样式。如果方面在您的设计中起着重要作用，并且您能够使用用于Eclipse 的[AspectJ开发工具（AJDT）](https://www.eclipse.org/ajdt/)插件，则AspectJ语言语法是首选。它更干净，更简单，因为该语言是专为编写方面而设计的。如果您不使用Eclipse或只有少数几个方面在您的应用程序中不起作用，那么您可能要考虑使用@AspectJ样式，在IDE中坚持常规Java编译，并向其中添加方面编织阶段您的构建脚本。

#### 5.6.2。@AspectJ或Spring AOP的XML？

如果选择使用Spring AOP，则可以选择@AspectJ或XML样式。有各种折衷考虑。

XML样式可能是现有Spring用户最熟悉的，并且得到了真正的POJO的支持。当使用AOP作为配置企业服务的工具时，XML是一个不错的选择（一个很好的测试是您是否将切入点表达式视为配置的一部分，而您可能希望独立更改）。使用XML样式，可以说从您的配置中可以更清楚地了解系统中存在哪些方面。

XML样式有两个缺点。首先，它没有完全将它所解决的需求的实现封装在一个地方。DRY原则说，系统中的任何知识都应该有单一，明确，权威的表示。当使用XML样式时，关于如何实现需求的知识会在配置文件中的后备bean类的声明和XML中分散。当您使用@AspectJ样式时，此信息将封装在一个模块中：方面。其次，与@AspectJ样式相比，XML样式在表达能力上有更多限制：仅支持“单例”方面实例化模型，并且无法组合以XML声明的命名切入点。例如，使用@AspectJ样式，您可以编写如下内容：

爪哇

科特林

```java
@Pointcut("execution(* get*())")
public void propertyAccess() {}

@Pointcut("execution(org.xyz.Account+ *(..))")
public void operationReturningAnAccount() {}

@Pointcut("propertyAccess() && operationReturningAnAccount()")
public void accountPropertyAccess() {}
```

在XML样式中，您可以声明前两个切入点：

```xml
<aop:pointcut id="propertyAccess"
        expression="execution(* get*())"/>

<aop:pointcut id="operationReturningAnAccount"
        expression="execution(org.xyz.Account+ *(..))"/>
```

XML方法的缺点是您无法 `accountPropertyAccess`通过组合这些定义来定义切入点。

@AspectJ样式支持其他实例化模型和更丰富的切入点组合。它具有将方面保持为模块化单元的优势。它还具有的优点是，Spring AOP和AspectJ都可以理解@AspectJ方面。因此，如果您以后决定需要AspectJ的功能来实现其他要求，则可以轻松地迁移到经典的AspectJ设置。总而言之，Spring团队在自定义方面更喜欢@AspectJ样式，而不是简单地配置企业服务。

### 5.7。混合方面类型

通过使用自动代理支持，模式定义的``方面，``声明的顾问程序，甚至是同一配置中其他样式的代理和拦截器，完全可以混合@AspectJ样式的方面。所有这些都是通过使用相同的基础支持机制来实现的，并且可以毫无困难地共存。

### 5.8。代理机制

Spring AOP使用JDK动态代理或CGLIB创建给定目标对象的代理。JDK内置了JDK动态代理，而CGLIB是一个通用的开源类定义库（重新打包为`spring-core`）。

如果要代理的目标对象实现至少一个接口，则使用JDK动态代理。代理了由目标类型实现的所有接口。如果目标对象未实现任何接口，则将创建CGLIB代理。

如果要强制使用CGLIB代理（例如，代理为目标对象定义的每个方法，而不仅是由其接口实现的方法），都可以这样做。但是，您应该考虑以下问题：

- 使用CGLIB，`final`无法建议方法，因为无法在运行时生成的子类中覆盖方法。
- 从Spring 4.0开始，由于CGLIB代理实例是通过Objenesis创建的，因此不再调用代理对象的构造函数两次。仅当您的JVM不允许绕过构造函数时，您才可能从Spring的AOP支持中看到两次调用和相应的调试日志条目。

要强制使用CGLIB代理，请将元素的`proxy-target-class`属性值设置``为true，如下所示：

```xml
<aop:config proxy-target-class="true">
    <!-- other beans defined here... -->
</aop:config>
```

要在使用@AspectJ自动代理支持时强制CGLIB代理，请`proxy-target-class`将该``元素的属性设置 为`true`，如下所示：

```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

|      | 多个``部分在运行时折叠到一个统一的自动代理创建器中，该创建器将应用任何部分（通常来自不同XML Bean定义文件）指定的*最强的*代理设置 ``。这也适用于``和`` 元素。明确地说，`proxy-target-class="true"`在``， ``或``上使用元素会强制*对所有三个*元素使用CGLIB代理。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 5.8.1。了解AOP代理

Spring AOP是基于代理的。在编写自己的方面或使用Spring框架随附的任何基于Spring AOP的方面之前，掌握最后一条语句实际含义的语义至关重要。

首先考虑您有一个普通的，未经代理的，无特殊要求的，直接的对象引用的场景，如以下代码片段所示：

爪哇

科特林

```java
public class SimplePojo implements Pojo {

    public void foo() {
        // this next method invocation is a direct call on the 'this' reference
        this.bar();
    }

    public void bar() {
        // some logic...
    }
}
```

如果在对象引用上调用方法，则直接在该对象引用上调用该方法，如下图和清单所示：

![aop代理普通pojo电话](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/images/aop-proxy-plain-pojo-call.png)

爪哇

科特林

```java
public class Main {

    public static void main(String[] args) {
        Pojo pojo = new SimplePojo();
        // this is a direct method call on the 'pojo' reference
        pojo.foo();
    }
}
```

当客户端代码具有的引用是代理时，情况会稍有变化。考虑以下图表和代码片段：

![aop代理呼叫](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/images/aop-proxy-call.png)

爪哇

科特林

```java
public class Main {

    public static void main(String[] args) {
        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());

        Pojo pojo = (Pojo) factory.getProxy();
        // this is a method call on the proxy!
        pojo.foo();
    }
}
```

这里要了解的关键是，类`main(..)`方法内的客户端代码`Main`具有对代理的引用。这意味着该对象引用上的方法调用是代理上的调用。结果，代理可以委派给与该特定方法调用相关的所有拦截器（建议）。但是，一旦调用最终到达目标对象（`SimplePojo`在这种情况下为参考），它可能会对自身进行的任何方法调用（例如`this.bar()`或 `this.foo()`）都将针对该`this`引用而不是代理进行调用。这具有重要意义。这意味着自调用不会导致与方法调用相关的建议得到执行的机会。

好吧，那该怎么办？最佳方法（在这里宽松地使用术语“最佳”）是重构代码，以免发生自调用。这确实需要您做一些工作，但这是最好的，侵入性最小的方法。下一种方法绝对可怕，我们正要指出这一点，恰恰是因为它是如此可怕。您可以（对我们来说是痛苦的）完全将类中的逻辑与Spring AOP绑定在一起，如以下示例所示：

爪哇

科特林

```java
public class SimplePojo implements Pojo {

    public void foo() {
        // this works, but... gah!
        ((Pojo) AopContext.currentProxy()).bar();
    }

    public void bar() {
        // some logic...
    }
}
```

这将您的代码完全耦合到Spring AOP，并且使类本身意识到在AOP上下文中使用它的事实，而AOP上下文却是如此。创建代理时，它还需要一些其他配置，如以下示例所示：

爪哇

科特林

```java
public class Main {

    public static void main(String[] args) {
        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());
        factory.setExposeProxy(true);

        Pojo pojo = (Pojo) factory.getProxy();
        // this is a method call on the proxy!
        pojo.foo();
    }
}
```

最后，必须注意，AspectJ没有此自调用问题，因为它不是基于代理的AOP框架。

### 5.9。以编程方式创建@AspectJ代理

除了使用`` 或来声明配置中的各个方面外，``还可以通过编程方式创建建议目标对象的代理。有关Spring的AOP API的完整详细信息，请参见 [下一章](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-api)。在这里，我们要重点介绍通过使用@AspectJ方面自动创建代理的功能。

您可以使用`org.springframework.aop.aspectj.annotation.AspectJProxyFactory`该类为一个或多个@AspectJ方面建议的目标对象创建代理。此类的基本用法非常简单，如以下示例所示：

爪哇

科特林

```java
// create a factory that can generate a proxy for the given target object
AspectJProxyFactory factory = new AspectJProxyFactory(targetObject);

// add an aspect, the class must be an @AspectJ aspect
// you can call this as many times as you need with different aspects
factory.addAspect(SecurityManager.class);

// you can also add existing aspect instances, the type of the object supplied must be an @AspectJ aspect
factory.addAspect(usageTracker);

// now get the proxy object...
MyInterfaceType proxy = factory.getProxy();
```

有关更多信息，请参见[javadoc](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/aop/aspectj/annotation/AspectJProxyFactory.html)。

### 5.10。在Spring应用程序中使用AspectJ

到目前为止，本章介绍的所有内容都是纯Spring AOP。在本节中，我们研究了如果您的需求超出了Spring AOP所提供的功能，那么如何使用AspectJ编译器或weaver代替Spring AOP或除Spring AOP之外使用。

Spring附带了一个小的AspectJ方面库，可以在您的发行版中独立使用`spring-aspects.jar`。您需要将其添加到您的类路径中才能使用其中的方面。[使用AspectJ依赖于Spring](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-atconfigurable)和[AspectJ的其他Spring方面来](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ajlib-other)[注入域对象](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-atconfigurable)以及[AspectJ](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ajlib-other)讨论了该库的内容以及如何使用它。[使用Spring IoC配置AspectJ方面](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-aj-configure)讨论了如何依赖注入使用AspectJ编译器编织的AspectJ方面。最后， [Spring Framework](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-aj-ltw)中使用AspectJ进行的加载时编织为使用AspectJ的Spring应用程序提供了加载时编织的介绍。

#### 5.10.1。使用AspectJ通过Spring依赖注入域对象

Spring容器实例化并配置在您的应用程序上下文中定义的bean。给定包含要应用的配置的Bean定义的名称，也可以要求Bean工厂配置预先存在的对象。 `spring-aspects.jar`包含注释驱动的方面，该方面利用此功能允许依赖项注入任何对象。该支撑旨在用于在任何容器的控制范围之外创建的对象。域对象通常属于此类，因为它们通常是`new`通过数据库查询的结果由操作员或ORM工具以编程方式创建的 。

该`@Configurable`注释标记一个类为通过Spring驱动的配置。在最简单的情况下，您可以将其纯粹用作标记注释，如以下示例所示：

爪哇

科特林

```java
package com.xyz.myapp.domain;

import org.springframework.beans.factory.annotation.Configurable;

@Configurable
public class Account {
    // ...
}
```

当以这种方式用作标记接口时，Spring `Account`通过使用具有与完全限定的类型名称（`com.xyz.myapp.domain.Account`）同名的bean定义（通常为原型作用域）来配置带注释类型的新实例（在本例中为）。由于Bean的默认名称是其类型的完全限定名称，因此声明原型定义的便捷方法是忽略该`id`属性，如以下示例所示：

```xml
<bean class="com.xyz.myapp.domain.Account" scope="prototype">
    <property name="fundsTransferService" ref="fundsTransferService"/>
</bean>
```

如果要显式指定要使用的原型bean定义的名称，则可以直接在批注中这样做，如以下示例所示：

爪哇

科特林

```java
package com.xyz.myapp.domain;

import org.springframework.beans.factory.annotation.Configurable;

@Configurable("account")
public class Account {
    // ...
}
```

Spring现在查找名为的bean定义`account`，并将其用作配置新`Account`实例的定义。

您也可以使用自动装配来避免完全指定专用的bean定义。要让Spring应用自动装配，请使用 注释的`autowire`属性`@Configurable`。您可以分别通过类型或名称指定`@Configurable(autowire=Autowire.BY_TYPE)`或 `@Configurable(autowire=Autowire.BY_NAME`用于自动布线。作为替代方案，最好`@Configurable`通过`@Autowired`或`@Inject` 在字段或方法级别为bean 指定显式的，注释驱动的依赖项注入（有关更多详细信息，请参见[基于注释的容器配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-annotation-config)）。

最后，您可以使用`dependencyCheck`属性（例如， `@Configurable(autowire=Autowire.BY_NAME,dependencyCheck=true)`）来为新创建和配置的对象中的对象引用启用Spring依赖项检查。如果将此属性设置为`true`，则Spring在配置后验证是否已设置所有属性（不是基元或集合）。

请注意，单独使用注释不会执行任何操作。注释的存在是通过 `AnnotationBeanConfigurerAspect`in `spring-aspects.jar`起作用的。从本质上讲，方面说：“从带有注释类型的新对象的初始化返回之后`@Configurable`，根据注释的属性使用Spring配置新创建的对象”。在这种情况下，“初始化”是指新实例化的对象（例如，用`new`运算符实例化的对象）以及`Serializable`正在进行反序列化的对象（例如，通过 [readResolve（）](https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html)）。

|      | 上段中的关键短语之一是“本质上”。在大多数情况下，“从新对象的初始化返回后”的确切语义是可以的。在这种情况下，“初始化之后”是指在构造对象之后注入依赖项。这意味着该依赖项不可在类的构造函数体中使用。如果要在构造函数主体执行之前注入依赖项，从而可以在构造函数主体中使用依赖项，则需要在`@Configurable`声明中定义此代码 ，如下所示：爪哇科特林`@Configurable(preConstruction = true)`您可以[在](https://www.eclipse.org/aspectj/doc/next/progguide/semantics-joinPoints.html)《[AspectJ编程指南》的](https://www.eclipse.org/aspectj/doc/next/progguide/index.html)[附录](https://www.eclipse.org/aspectj/doc/next/progguide/semantics-joinPoints.html)中找到有关AspectJ [中](https://www.eclipse.org/aspectj/doc/next/progguide/semantics-joinPoints.html)各种切入点类型的语言语义的更多信息 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

为此，必须将带注释的类型与AspectJ编织器编织在一起。您可以使用构建时的Ant或Maven任务来执行此操作（例如，请参见 [AspectJ开发环境指南](https://www.eclipse.org/aspectj/doc/released/devguide/antTasks.html)），也可以使用加载时编织（请参见[Spring Framework中的使用AspectJ进行加载时编织](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-aj-ltw)）。的 `AnnotationBeanConfigurerAspect`本身需要由弹簧（为了获得对bean工厂要被用于配置新的对象的引用）构成。如果使用基于Java的配置，则可以将其添加`@EnableSpringConfigured`到任何 `@Configuration`类，如下所示：

爪哇

科特林

```java
@Configuration
@EnableSpringConfigured
public class AppConfig {
}
```

如果您更喜欢基于XML的配置，Spring [`context`名称空间](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-schemas-context) 定义了一个方便的`context:spring-configured`元素，您可以按如下方式使用它：

```xml
<context:spring-configured/>
```

`@Configurable`在配置方面之前创建的对象实例会导致向调试日志发出消息，并且未进行对象配置。一个例子可能是Spring配置中的一个bean，当它由Spring初始化时会创建域对象。在这种情况下，可以使用 `depends-on`bean属性来手动指定bean取决于配置方面。以下示例显示如何使用`depends-on`属性：

```xml
<bean id="myService"
        class="com.xzy.myapp.service.MyService"
        depends-on="org.springframework.beans.factory.aspectj.AnnotationBeanConfigurerAspect">

    <!-- ... -->

</bean>
```

|      | `@Configurable`除非您真的想在运行时依赖它的语义，否则 不要通过bean配置器方面来激活处理。特别是，请确保不要`@Configurable`在容器中使用已注册为常规Spring bean的bean类。这样做将导致两次初始化，一次是通过容器，一次是通过方面。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 单元测试`@Configurable`对象

`@Configurable`支持的目标之一是实现域对象的独立单元测试，而不会遇到与硬编码查找相关的困难。如果`@Configurable`AspectJ尚未编织类型，则注释在单元测试期间不起作用。您可以在被测对象中设置模拟或存根属性引用，然后照常进行。如果`@Configurable`AspectJ已经编织了类型，您仍然可以像往常一样在容器外部进行单元测试，但是每次构造一个`@Configurable`对象时都会看到一条警告消息，指示该对象尚未由Spring进行配置。

##### 使用多个应用程序上下文

的`AnnotationBeanConfigurerAspect`是，用于实现`@Configurable`支持是一个AspectJ singleton切面。单例方面的范围与`static`成员的范围相同：每个类加载器都有一个方面实例来定义类型。这意味着，如果您在同一个类加载器层次结构中定义多个应用程序上下文，则需要考虑在何处定义`@EnableSpringConfigured`bean以及在`spring-aspects.jar`类路径中的放置位置。

考虑一个典型的Spring Web应用程序配置，该配置具有一个共享的父应用程序上下文，该上下文定义了通用的业务服务，支持那些服务所需的一切，以及每个Servlet的一个子应用程序上下文（其中包含该Servlet的特定定义）。所有这些上下文共存于相同的类加载器层次结构中，因此`AnnotationBeanConfigurerAspect`只能保留对其中一个的引用。在这种情况下，我们建议`@EnableSpringConfigured`在共享（父）应用程序上下文中定义bean。这定义了您可能想注入域对象的服务。结果是，您不能使用@Configurable机制（通过子对象（特定于servlet的）上下文中定义的Bean的引用来配置域对象）（无论如何，这都不是您想做的事情）。

在同一容器内部署多个Web应用程序时，请确保每个Web应用程序都`spring-aspects.jar`使用自己的类加载器（例如，放置`spring-aspects.jar`在中`'WEB-INF/lib'`）来加载类型。如果`spring-aspects.jar` 仅将if 添加到容器范围的类路径中（并因此由共享的父类加载器加载），则所有Web应用程序共享相同的方面实例（这可能不是您想要的）。

#### 5.10.2。AspectJ的其他Spring方面

除了`@Configurable`方面之外，还`spring-aspects.jar`包含一个AspectJ方面，您可以使用它来驱动对带有`@Transactional`注释的类型和方法进行Spring的事务管理。这主要适用于希望在Spring容器之外使用Spring Framework的事务支持的用户。

解释`@Transactional`注释的方面是 `AnnotationTransactionAspect`。使用此方面时，必须注释实现类（或该类中的方法，或两者），而不是注释该类所实现的接口（如果有）。AspectJ遵循Java的规则，即不继承接口上的注释。

一`@Transactional`类上注解指定任何公开操作的类执行默认事务语义。

`@Transactional`类内方法的注释会覆盖类注释（如果存在）给出的默认事务语义。可以标注任何可见性的方法，包括私有方法。直接注释非公共方法是执行此类方法而获得事务划分的唯一方法。

|      | 从Spring Framework 4.2开始，`spring-aspects`提供了类似的方面，为标准`javax.transaction.Transactional`注释提供了完全相同的功能。检查 `JtaAnnotationTransactionAspect`更多细节。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

对于希望使用Spring配置和事务管理支持但又不想（或不能）使用注释的AspectJ程序员，它们`spring-aspects.jar` 也包含`abstract`可以扩展以提供自己的切入点定义的方面。请参阅来源`AbstractBeanConfigurerAspect`和 `AbstractTransactionAspect`获取更多信息的方面。例如，以下摘录显示了如何编写方面来使用与完全限定的类名匹配的原型Bean定义来配置域模型中定义的对象的所有实例：

```java
public aspect DomainObjectConfiguration extends AbstractBeanConfigurerAspect {

    public DomainObjectConfiguration() {
        setBeanWiringInfoResolver(new ClassNameBeanWiringInfoResolver());
    }

    // the creation of a new bean (any object in the domain model)
    protected pointcut beanCreation(Object beanInstance) :
        initialization(new(..)) &&
        SystemArchitecture.inDomainModel() &&
        this(beanInstance);
}
```

#### 5.10.3。使用Spring IoC配置AspectJ Aspects

当您将AspectJ方面与Spring应用程序一起使用时，既自然又希望能够使用Spring配置这些方面。AspectJ运行时本身负责方面的创建，通过Spring配置AspectJ创建的方面的方式取决于方面所使用的AspectJ实例化模型（`per-xxx`子句）。

AspectJ的大多数方面都是单例方面。这些方面的配置很容易。您可以创建一个bean定义，该bean定义按常规引用方面类型并包含`factory-method="aspectOf"`bean属性。这样可以确保Spring通过向AspectJ索要长宽比实例，而不是尝试自己创建实例来获得长宽比实例。以下示例显示如何使用`factory-method="aspectOf"`属性：

```xml
<bean id="profiler" class="com.xyz.profiler.Profiler"
        factory-method="aspectOf"> 

    <property name="profilingStrategy" ref="jamonProfilingStrategy"/>
</bean>
```

|      | 注意`factory-method="aspectOf"`属性 |
| ---- | ----------------------------------- |
|      |                                     |

非单一方面很难配置。然而，可以通过bean原型定义和使用这样做`@Configurable`从支持 `spring-aspects.jar`配置方面情况，一旦他们由AspectJ runtime创建。

如果您有一些要与AspectJ编织的@AspectJ方面（例如，对域模型类型使用加载时编织）以及要与Spring AOP一起使用的其他@AspectJ方面，那么这些方面都已在Spring中配置，您需要告诉Spring AOP @AspectJ自动代理支持，应使用配置中定义的@AspectJ方面的确切子集进行自动代理。您可以通过``在`` 声明中使用一个或多个元素来执行此操作。每个``元素指定一个名称模式，并且只有名称与至少一个模式匹配的bean才用于Spring AOP自动代理配置。以下示例显示如何使用``元素：

```xml
<aop:aspectj-autoproxy>
    <aop:include name="thisBean"/>
    <aop:include name="thatBean"/>
</aop:aspectj-autoproxy>
```

|      | 不要被``元素的名称所迷惑。使用它可以创建Spring AOP代理。这里使用了@AspectJ样式的声明，但是没有涉及AspectJ运行时。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 5.10.4。在Spring Framework中使用AspectJ进行加载时编织

加载时编织（LTW）是指将AspectJ方面加载到应用程序的类文件中时将其编织到Java虚拟机（JVM）中的过程。本部分的重点是在Spring框架的特定上下文中配置和使用LTW。本节不是LTW的一般介绍。有关LTW的详细信息以及仅使用AspectJ配置LTW（完全不涉及Spring）的详细信息，请参阅[《 AspectJ开发环境指南》](https://www.eclipse.org/aspectj/doc/released/devguide/ltw.html)的 [LTW部分](https://www.eclipse.org/aspectj/doc/released/devguide/ltw.html)。

Spring框架为AspectJ LTW带来的价值在于能够对编织过程进行更精细的控制。“ Vanilla” AspectJ LTW是通过使用Java（5+）代理来实现的，该代理在启动JVM时通过指定VM参数来打开。因此，它是JVM范围的设置，在某些情况下可能很好，但通常有点过于粗糙。启用了Spring的LTW允许您`ClassLoader`逐个打开LTW ，它的粒度更细，并且在“单JVM-多应用程序”环境中（例如在典型的应用程序服务器环境中可以找到）更有意义。 ）。

此外，[在某些环境中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-aj-ltw-environments)，此支持可实现加载时编织，而无需对添加`-javaagent:path/to/aspectjweaver.jar`或所需的应用程序服务器的启动脚本进行任何修改（如本节稍后所述）`-javaagent:path/to/spring-instrument.jar`。开发人员将应用程序上下文配置为启用加载时编织，而不是依赖通常负责部署配置（例如启动脚本）的管理员。

现在，销售工作已经结束，让我们首先浏览一个使用Spring的AspectJ LTW的快速示例，然后详细介绍该示例中引入的元素。有关完整示例，请参见 [Petclinic示例应用程序](https://github.com/spring-projects/spring-petclinic)。

##### 第一个例子

假定您是一位负责诊断系统中某些性能问题的原因的应用程序开发人员。与其使用分析工具，不如使用一个简单的分析方面，使我们能够快速获得一些性能指标。然后，我们可以立即在该特定区域应用更细粒度的分析工具。

|      | 此处提供的示例使用XML配置。您还可以配置@AspectJ并将其与[Java配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java)一起使用。具体来说，您可以将 `@EnableLoadTimeWeaving`注释用作替代方法`` （有关详细信息，请参见[下文](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-aj-ltw-spring)）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下示例显示了配置方面的信息，这并不理想。这是一个基于时间的探查器，它使用@AspectJ样式的方面声明：

爪哇

科特林

```java
package foo;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.util.StopWatch;
import org.springframework.core.annotation.Order;

@Aspect
public class ProfilingAspect {

    @Around("methodsToBeProfiled()")
    public Object profile(ProceedingJoinPoint pjp) throws Throwable {
        StopWatch sw = new StopWatch(getClass().getSimpleName());
        try {
            sw.start(pjp.getSignature().getName());
            return pjp.proceed();
        } finally {
            sw.stop();
            System.out.println(sw.prettyPrint());
        }
    }

    @Pointcut("execution(public * foo..*.*(..))")
    public void methodsToBeProfiled(){}
}
```

我们还需要创建一个`META-INF/aop.xml`文件，以通知AspectJ编织器我们要将其编织`ProfilingAspect`到类中。此文件约定，即在Java类路径上存在一个（或多个）文件的情况称为`META-INF/aop.xml`标准AspectJ。以下示例显示了该`aop.xml`文件：

```xml
<!DOCTYPE aspectj PUBLIC "-//AspectJ//DTD//EN" "https://www.eclipse.org/aspectj/dtd/aspectj.dtd">
<aspectj>

    <weaver>
        <!-- only weave classes in our application-specific packages -->
        <include within="foo.*"/>
    </weaver>

    <aspects>
        <!-- weave in just this aspect -->
        <aspect name="foo.ProfilingAspect"/>
    </aspects>

</aspectj>
```

现在，我们可以继续进行配置中特定于Spring的部分。我们需要配置一个`LoadTimeWeaver`（稍后说明）。该加载时织布器是必不可少的组件，负责将一个或多个`META-INF/aop.xml`文件中的方面配置编织到应用程序的类中。好处是，它不需要很多配置（您可以指定一些其他选项，但是稍后会详细介绍），如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- a service object; we will be profiling its methods -->
    <bean id="entitlementCalculationService"
            class="foo.StubEntitlementCalculationService"/>

    <!-- this switches on the load-time weaving -->
    <context:load-time-weaver/>
</beans>
```

现在所有必需的构件（方面，`META-INF/aop.xml` 文件和Spring配置）都就位了，我们可以创建以下驱动程序类，并使用一种`main(..)`方法来演示LTW的作用：

爪哇

科特林

```java
package foo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Main {

    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml", Main.class);

        EntitlementCalculationService entitlementCalculationService =
                (EntitlementCalculationService) ctx.getBean("entitlementCalculationService");

        // the profiling aspect is 'woven' around this method execution
        entitlementCalculationService.calculateEntitlement();
    }
}
```

我们还有最后一件事要做。本节的引言确实说过，可以`ClassLoader`使用Spring 选择性地打开LTW ，这是事实。但是，在此示例中，我们使用Java代理（Spring随附）打开LTW。我们使用以下命令来运行`Main`前面显示的类：

```
java -javaagent：C：/projects/foo/lib/global/spring-instrument.jar foo.Main
```

该`-javaagent`标志用于指定[代理](https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html)并使 [代理能够对在JVM上运行的程序进行检测](https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html)。Spring Framework附带了一个代理，该代理`InstrumentationSavingAgent`打包在中，该 代理`spring-instrument.jar`作为`-javaagent`上一示例中的参数值提供。

`Main`程序执行的输出类似于下一个示例。（我`Thread.sleep(..)`在`calculateEntitlement()` 实现中引入了一条语句，以便探查器实际上捕获的不是0毫秒（`01234`毫秒不是AOP引入的开销）。以下清单显示了运行探查器时得到的输出：

```
计算权利

秒表'ProfilingAspect'：运行时间（毫秒）= 1234
------ ----- ----------------------------
ms％任务名称
------ ----- ----------------------------
01234 100％计算权利
```

由于此LTW是通过使用成熟的AspectJ来实现的，因此我们不仅限于建议Spring Bean。在`Main`程序上进行以下细微改动会产生相同的结果：

爪哇

科特林

```java
package foo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Main {

    public static void main(String[] args) {
        new ClassPathXmlApplicationContext("beans.xml", Main.class);

        EntitlementCalculationService entitlementCalculationService =
                new StubEntitlementCalculationService();

        // the profiling aspect will be 'woven' around this method execution
        entitlementCalculationService.calculateEntitlement();
    }
}
```

请注意，在前面的程序中，我们如何引导Spring容器，然后创建`StubEntitlementCalculationService`完全不在Spring上下文外部的新实例。剖析建议仍会被应用。

诚然，这个例子很简单。但是，在前面的示例中已经介绍了Spring对LTW支持的基础，本节的其余部分详细解释了每一位配置和用法背后的“原因”。

|      | 在`ProfilingAspect`本例中使用可能是基本的，但它是非常有用的。这是开发时方面的一个很好的示例，开发人员可以在开发期间使用它，然后轻松地将其从部署到UAT或生产中的应用程序构建中排除。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 方面

您在LTW中使用的方面必须是AspectJ方面。您可以使用AspectJ语言本身来编写它们，也可以使用@AspectJ风格来编写方面。这样，您的方面就是有效的AspectJ和Spring AOP方面。此外，编译的方面类需要在类路径上可用。

##### 'META-INF / aop.xml'

通过使用`META-INF/aop.xml` Java类路径上的一个或多个文件（直接或通常在jar文件中）来配置AspectJ LTW基础结构。

该文件的结构和内容在[AspectJ参考文档](https://www.eclipse.org/aspectj/doc/released/devguide/ltw-configuration.html)的LTW部分中进行了详细 [说明](https://www.eclipse.org/aspectj/doc/released/devguide/ltw-configuration.html)。由于该`aop.xml`文件是100％AspectJ，因此在此不再赘述。

##### 所需的库（JARS）

至少，您需要以下库来使用Spring Framework对AspectJ LTW的支持：

- `spring-aop.jar`
- `aspectjweaver.jar`

如果您使用[Spring提供的代理来启用检测](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-aj-ltw-environments-generic)，则还需要：

- `spring-instrument.jar`

##### 弹簧配置

Spring的LTW支持中的关键组件是`LoadTimeWeaver`接口（在 `org.springframework.instrument.classloading`包装中），以及Spring发行版附带的众多实现。A `LoadTimeWeaver`负责在运行时`java.lang.instrument.ClassFileTransformers`向a 添加一个或多个`ClassLoader`，这为各种有趣的应用程序打开了大门，其中之一恰好是方面的LTW。

|      | 如果您不熟悉运行时类文件转换的概念，请`java.lang.instrument`在继续之前查看该软件包的javadoc API文档。尽管该文档并不全面，但至少您可以看到关键的接口和类（在您通读本节时供参考）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`LoadTimeWeaver`为一个特定的配置a `ApplicationContext`就像添加一行一样容易。（请注意，您几乎肯定需要使用a `ApplicationContext`作为您的Spring容器-通常，仅使用a `BeanFactory`是不够的，因为LTW支持使用`BeanFactoryPostProcessors`。）

要启用Spring Framework的LTW支持，您需要配置`LoadTimeWeaver`，通常通过使用`@EnableLoadTimeWeaving`批注来完成，如下所示：

爪哇

科特林

```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```

或者，如果您更喜欢基于XML的配置，请使用 ``元素。请注意，元素是在`context`名称空间中定义的 。以下示例显示如何使用``：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:load-time-weaver/>

</beans>
```

前面的配置会自动为您定义并注册许多LTW特定的基础结构Bean，例如a `LoadTimeWeaver`和an `AspectJWeavingEnabler`。默认`LoadTimeWeaver`值为`DefaultContextLoadTimeWeaver`class，它尝试装饰自动检测到的`LoadTimeWeaver`。`LoadTimeWeaver` “自动检测” 的确切类型取决于您的运行时环境。下表总结了各种`LoadTimeWeaver`实现：

| 运行环境                                                     | `LoadTimeWeaver` 实作           |
| :----------------------------------------------------------- | :------------------------------ |
| 在[Apache Tomcat中](https://tomcat.apache.org/)运行          | `TomcatLoadTimeWeaver`          |
| 在[GlassFish中](https://eclipse-ee4j.github.io/glassfish/)运行（仅限EAR部署） | `GlassFishLoadTimeWeaver`       |
| 在Red Hat的[JBoss AS](https://www.jboss.org/jbossas/)或[WildFly中运行](https://www.wildfly.org/) | `JBossLoadTimeWeaver`           |
| 在IBM的[WebSphere中](https://www-01.ibm.com/software/webservers/appserv/was/)运行 | `WebSphereLoadTimeWeaver`       |
| 在Oracle的[WebLogic中](https://www.oracle.com/technetwork/middleware/weblogic/overview/index-085209.html)运行 | `WebLogicLoadTimeWeaver`        |
| JVM从Spring `InstrumentationSavingAgent` （`java -javaagent:path/to/spring-instrument.jar`）开始 | `InstrumentationLoadTimeWeaver` |
| 回退，期望基础ClassLoader遵循通用约定（即方法`addTransformer`，可选地`getThrowawayClassLoader`） | `ReflectiveLoadTimeWeaver`      |

请注意，该表仅列出`LoadTimeWeavers`使用时自动检测到的`DefaultContextLoadTimeWeaver`。您可以确切指定`LoadTimeWeaver` 要使用的实现。

要指定特定`LoadTimeWeaver`的Java配置，请实现该 `LoadTimeWeavingConfigurer`接口并覆盖该`getLoadTimeWeaver()`方法。以下示例指定一个`ReflectiveLoadTimeWeaver`：

爪哇

科特林

```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig implements LoadTimeWeavingConfigurer {

    @Override
    public LoadTimeWeaver getLoadTimeWeaver() {
        return new ReflectiveLoadTimeWeaver();
    }
}
```

如果使用基于XML的配置，则可以将完全限定的类名指定为 元素`weaver-class`上属性的值``。同样，以下示例指定了`ReflectiveLoadTimeWeaver`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:load-time-weaver
            weaver-class="org.springframework.instrument.classloading.ReflectiveLoadTimeWeaver"/>

</beans>
```

该`LoadTimeWeaver`中定义并且由配置注册可从Spring容器通过使用公知的名称以后检索`loadTimeWeaver`。请记住，`LoadTimeWeaver`仅作为Spring LTW基础结构添加一个或多个机制的存在`ClassFileTransformers`。实际 `ClassFileTransformer`执行LTW的是`ClassPreProcessorAgentAdapter`（从`org.aspectj.weaver.loadtime`包中）类。有关`ClassPreProcessorAgentAdapter`更多详细信息，请参见该类的类级javadoc ，因为实际上如何实现编织的细节不在本文档的讨论范围之内。

剩下的最后一个配置属性需要讨论：该`aspectjWeaving` 属性（或者`aspectj-weaving`使用XML）。此属性控制是否启用LTW。它接受三个可能值之一，默认值是 `autodetect`如果属性不存在。下表总结了三个可能的值：

| 注释值       | XML值        | 说明                                                         |
| :----------- | :----------- | :----------------------------------------------------------- |
| `ENABLED`    | `on`         | AspectJ正在编织，并且在加载时适当地编织了方面。              |
| `DISABLED`   | `off`        | LTW已关闭。加载时不编织任何方面。                            |
| `AUTODETECT` | `autodetect` | 如果Spring LTW基础结构可以找到至少一个`META-INF/aop.xml`文件，则AspectJ编织已开始。否则，它关闭。这是默认值。 |

##### 特定于环境的配置

最后一部分包含在应用程序服务器和Web容器等环境中使用Spring的LTW支持时所需的任何其他设置和配置。

###### Tomcat，JBoss，WebSphere，WebLogic

Tomcat，JBoss / WildFly，IBM WebSphere Application Server和Oracle WebLogic Server均提供了`ClassLoader`能够进行本地检测的通用应用程序。Spring的本地LTW可以利用这些ClassLoader实现来提供AspectJ编织。您可以简单地启用加载时编织，[如前所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-using-aspectj)。具体来说，您无需修改JVM启动脚本即可添加 `-javaagent:path/to/spring-instrument.jar`。

请注意，在JBoss上，您可能需要禁用应用服务器扫描，以防止它在应用程序实际启动之前加载类。一个快速的解决方法是将一个`WEB-INF/jboss-scanning.xml`具有以下内容的文件添加到您的工件中：

```xml
<scanning xmlns="urn:jboss:scanning:1.0"/>
```

###### 通用Java应用程序

在特定`LoadTimeWeaver`实现不支持的环境中需要类检测时，JVM代理是通用解决方案。对于这种情况，Spring提供了`InstrumentationLoadTimeWeaver`一个需要Spring特定（但非常通用）的JVM代理的程序，该JVM代理`spring-instrument.jar`可以通过common `@EnableLoadTimeWeaving`和``setups 自动检测到。

要使用它，必须通过提供以下JVM选项来使用Spring代理启动虚拟机：

```
-javaagent：/path/to/spring-instrument.jar
```

请注意，这需要修改JVM启动脚本，这可能会阻止您在应用程序服务器环境中使用它（取决于您的服务器和您的操作策略）。也就是说，对于每个JVM一个应用程序的部署（例如独立的Spring Boot应用程序），无论如何，您通常都可以控制整个JVM的设置。

### 5.11。更多资源

可以在[AspectJ网站](https://www.eclipse.org/aspectj)上找到有关[AspectJ的](https://www.eclipse.org/aspectj)更多信息。

*Eclipse AspectJ*，作者：Adrian Colyer等。等 （Addison-Wesley，2005年）为AspectJ语言提供了全面的介绍和参考。

强烈推荐Ramnivas Laddad撰写的*《 AspectJ in Action》*第二版（Manning，2009年）。本书的重点是AspectJ，但是（在一定程度上）探讨了许多通用的AOP主题。

## 6. Spring AOP API

上一章通过@AspectJ和基于模式的方面定义描述了Spring对AOP的支持。在本章中，我们讨论了较低级别的Spring AOP API。对于常见的应用程序，我们建议将Spring AOP与AspectJ切入点一起使用，如上一章所述。

### 6.1。Spring中的Pointcut API

本节描述了Spring如何处理关键切入点概念。

#### 6.1.1。概念

Spring的切入点模型使切入点重用不受建议类型的影响。您可以使用相同的切入点来定位不同的建议。

该`org.springframework.aop.Pointcut`接口是中央接口，用于将建议定向到特定的类和方法。完整的界面如下：

爪哇

科特林

```java
public interface Pointcut {

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();

}
```

将`Pointcut`接口分为两部分，可以重用类和方法匹配的部分，以及细粒度的合成操作（例如与另一个方法匹配器执行“联合”）。

该`ClassFilter`接口用于将切入点限制为给定的一组目标类。如果该`matches()`方法始终返回true，则匹配所有目标类。以下清单显示了`ClassFilter`接口定义：

爪哇

科特林

```java
public interface ClassFilter {

    boolean matches(Class clazz);
}
```

该`MethodMatcher`接口通常更重要。完整的界面如下：

爪哇

科特林

```java
public interface MethodMatcher {

    boolean matches(Method m, Class targetClass);

    boolean isRuntime();

    boolean matches(Method m, Class targetClass, Object[] args);
}
```

该`matches(Method, Class)`方法用于测试此切入点是否与目标类上的给定方法匹配。创建AOP代理时可以执行此评估，以避免需要对每个方法调用进行测试。如果对于给定`matches`方法返回了两个参数的方法`true`，并且该`isRuntime()`方法返回了MethodMatcher `true`，则在每次方法调用时都会调用三个参数的match方法。这样，切入点就可以在执行目标建议之前立即查看传递给方法调用的参数。

大多数`MethodMatcher`实现都是静态的，这意味着它们的`isRuntime()`方法将返回`false`。在这种情况下，`matches`永远不会调用三参数方法。

|      | 如果可能，请尝试使切入点成为静态，从而允许AOP框架在创建AOP代理时缓存切入点评估的结果。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 6.1.2。切入点的操作

Spring支持切入点上的操作（特别是联合和相交）。

联合表示两个切入点都匹配的方法。相交是指两个切入点都匹配的方法。联合通常更有用。您可以通过使用类中的静态方法`org.springframework.aop.support.Pointcuts`或使用`ComposablePointcut`同一包中的类 来编写切入点 。但是，使用AspectJ切入点表达式通常是一种更简单的方法。

#### 6.1.3。AspectJ表达切入点

从2.0开始，Spring使用的最重要的切入点类型是 `org.springframework.aop.aspectj.AspectJExpressionPointcut`。这是一个切入点，该切入点使用AspectJ提供的库来解析AspectJ切入点表达式字符串。

有关支持的AspectJ切入点原语的讨论，请参见[上一章](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop)。

#### 6.1.4。便捷切入点实现

Spring提供了几种方便的切入点实现。您可以直接使用其中一些。其他的则应归入特定于应用程序的切入点中。

##### 静态切入点

静态切入点基于方法和目标类，并且不能考虑方法的参数。静态切入点对于大多数用途来说已经足够，最好。首次调用方法时，Spring只能评估一次静态切入点。之后，无需在每次方法调用时再次评估切入点。

本节的其余部分描述了Spring附带的一些静态切入点实现。

###### 正则表达式切入点

指定静态切入点的一种明显方法是正则表达式。除了Spring之外，还有几个AOP框架使之成为可能。 `org.springframework.aop.support.JdkRegexpMethodPointcut`是通用的正则表达式切入点，它使用JDK中的正则表达式支持。

使用`JdkRegexpMethodPointcut`该类，您可以提供模式字符串的列表。如果其中任何一个匹配，则切入点的计算结果为`true`。（因此，结果实际上是这些切入点的并集。）

以下示例显示如何使用`JdkRegexpMethodPointcut`：

```xml
<bean id="settersAndAbsquatulatePointcut"
        class="org.springframework.aop.support.JdkRegexpMethodPointcut">
    <property name="patterns">
        <list>
            <value>.*set.*</value>
            <value>.*absquatulate</value>
        </list>
    </property>
</bean>
```

Spring提供了一个名为的便捷类`RegexpMethodPointcutAdvisor`，它使我们也可以引用`Advice`（记住`Advice`，在建议，引发建议等之前，an 可以是拦截器）。在后台，Spring使用`JdkRegexpMethodPointcut`。使用`RegexpMethodPointcutAdvisor`简化了接线，因为一个bean封装了切入点和建议，如以下示例所示：

```xml
<bean id="settersAndAbsquatulateAdvisor"
        class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <property name="advice">
        <ref bean="beanNameOfAopAllianceInterceptor"/>
    </property>
    <property name="patterns">
        <list>
            <value>.*set.*</value>
            <value>.*absquatulate</value>
        </list>
    </property>
</bean>
```

您可以使用`RegexpMethodPointcutAdvisor`任何`Advice`类型。

###### 属性驱动的切入点

静态切入点的一种重要类型是元数据驱动的切入点。这将使用元数据属性的值（通常是源级别的元数据）。

##### 动态切入点

动态切入点比静态切入点更昂贵。它们考虑了方法参数以及静态信息。这意味着必须在每次方法调用时对它们进行评估，并且由于参数会有所不同，因此无法缓存结果。

主要的例子是`control flow`切入点。

###### 控制流切入点

弹簧控制流切入点在概念上类似于AspectJ `cflow`切入点，尽管功能不那么强大。（当前无法指定一个切入点在与另一个切入点匹配的连接点下执行。）控制流切入点与当前调用堆栈匹配。例如，如果连接点是由`com.mycompany.web`包中的方法或`SomeCaller`类调用的，则可能会触发。使用`org.springframework.aop.support.ControlFlowPointcut`类指定控制流切入点。

|      | 与其他动态切入点相比，控制流切入点在运行时进行评估要昂贵得多。在Java 1.4中，成本大约是其他动态切入点的五倍。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 6.1.5。切入点超类

Spring提供了有用的切入点超类，以帮助您实现自己的切入点。

因为静态切入点最有用，所以您可能应该subclass `StaticMethodMatcherPointcut`。这仅需要实现一个抽象方法（尽管您可以覆盖其他方法以自定义行为）。以下示例显示了如何进行子类化`StaticMethodMatcherPointcut`：

爪哇

科特林

```java
class TestStaticPointcut extends StaticMethodMatcherPointcut {

    public boolean matches(Method m, Class targetClass) {
        // return true if custom criteria match
    }
}
```

还有动态切入点的超类。您可以将自定义切入点与任何建议类型一起使用。

#### 6.1.6。自定义切入点

由于Spring AOP中的切入点是Java类，而不是语言功能（如AspectJ），因此可以声明自定义切入点，无论是静态还是动态。Spring中的自定义切入点可以任意复杂。但是，如果可以的话，我们建议使用AspectJ切入点表达语言。

|      | 更高版本的Spring可能提供对JAC提供的“语义切入点”的支持，例如，“更改目标对象中实例变量的所有方法”。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 6.2。春季咨询API

现在，我们可以检查Spring AOP如何处理建议。

#### 6.2.1。咨询生命周期

每个建议都是一个Spring bean。建议实例可以在所有建议对象之间共享，或者对于每个建议对象都是唯一的。这对应于每个班级或每个实例的建议。

每班建议最常用。适用于一般建议，例如交易顾问。这些不依赖于代理对象的状态或添加新状态。它们仅作用于方法和参数。

每个实例的建议都适合引入，以支持混合。在这种情况下，建议将状态添加到代理对象。

您可以在同一AOP代理中混合使用共享建议和基于实例的建议。

#### 6.2.2。春天的建议类型

Spring提供了几种建议类型，并且可以扩展以支持任意建议类型。本节介绍基本概念和标准建议类型。

##### 拦截咨询

Spring中最基本的建议类型是围绕建议的拦截。

`Alliance`对于使用方法拦截的建议，Spring符合AOP 接口。实现`MethodInterceptor`和围绕建议实施的类还应该实现以下接口：

爪哇

科特林

```java
public interface MethodInterceptor extends Interceptor {

    Object invoke(MethodInvocation invocation) throws Throwable;
}
```

方法的`MethodInvocation`参数`invoke()`公开了正在调用的方法，目标连接点，AOP代理以及方法的参数。该 `invoke()`方法应返回调用的结果：连接点的返回值。

以下示例显示了一个简单的`MethodInterceptor`实现：

爪哇

科特林

```java
public class DebugInterceptor implements MethodInterceptor {

    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("Before: invocation=[" + invocation + "]");
        Object rval = invocation.proceed();
        System.out.println("Invocation returned");
        return rval;
    }
}
```

请注意对的`proceed()`方法的调用`MethodInvocation`。这沿着拦截器链向下到达连接点。大多数拦截器都调用此方法并返回其返回值。但是，a `MethodInterceptor`像任何周围的建议一样，可以返回一个不同的值或引发异常，而不是调用proceed方法。但是，您不希望在没有充分理由的情况下执行此操作。

|      | `MethodInterceptor`实现提供与其他符合AOP Alliance的AOP实现的互操作性。本节其余部分讨论的其他建议类型将实现常见的AOP概念，但以特定于Spring的方式。尽管使用最具体的建议类型有一个优势，但是`MethodInterceptor`如果您可能想在另一个AOP框架中运行方面，则坚持使用周围建议。请注意，切入点当前无法在框架之间互操作，并且AOP Alliance当前未定义切入点接口。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 咨询前

一种更简单的建议类型是事前建议。这不需要`MethodInvocation` 对象，因为仅在进入方法之前才调用它。

先行建议的主要优点是无需调用该`proceed()` 方法，因此，不会无意中导致无法沿拦截器链继续进行下去。

以下清单显示了该`MethodBeforeAdvice`接口：

爪哇

科特林

```java
public interface MethodBeforeAdvice extends BeforeAdvice {

    void before(Method m, Object[] args, Object target) throws Throwable;
}
```

（尽管通常的对象适用于字段拦截，并且Spring不太可能实现它，但Spring的API设计允许先于字段咨询。）

请注意，返回类型为`void`。通知可以在联接点执行之前插入自定义行为，但不能更改返回值。如果之前的建议引发异常，它将中止拦截器链的进一步执行。异常会传播回拦截链。如果未选中它或在调用的方法的签名上，则将其直接传递给客户端。否则，它将被AOP代理包装在未经检查的异常中。

下面的示例显示了Spring中的before建议，该建议对所有方法调用进行计数：

爪哇

科特林

```java
public class CountingBeforeAdvice implements MethodBeforeAdvice {

    private int count;

    public void before(Method m, Object[] args, Object target) throws Throwable {
        ++count;
    }

    public int getCount() {
        return count;
    }
}
```

|      | 在将建议与任何切入点一起使用之前。 |
| ---- | ---------------------------------- |
|      |                                    |

##### 提出建议

如果联接点引发异常，则在联接点返回之后调用引发通知。Spring提供类型化的抛出建议。请注意，这意味着该 `org.springframework.aop.ThrowsAdvice`接口不包含任何方法。它是一个标签接口，用于标识给定对象实现了一个或多个类型化的throws通知方法。这些应采用以下形式：

```java
afterThrowing([Method, args, target], subclassOfThrowable)
```

仅最后一个参数是必需的。方法签名可以具有一个或四个参数，具体取决于建议方法是否对该方法和参数感兴趣。接下来的两个清单显示了作为引发建议示例的类。

如果`RemoteException`抛出a（包括从子类），则调用以下建议：

爪哇

科特林

```java
public class RemoteThrowsAdvice implements ThrowsAdvice {

    public void afterThrowing(RemoteException ex) throws Throwable {
        // Do something with remote exception
    }
}
```

与前面的建议不同，下一个示例声明四个参数，以便可以访问被调用的方法，方法参数和目标对象。如果`ServletException`抛出a，则调用以下建议：

爪哇

科特林

```java
public class ServletThrowsAdviceWithArguments implements ThrowsAdvice {

    public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
        // Do something with all arguments
    }
}
```

最后一个示例说明了如何在处理`RemoteException`和的单个类中使用这两种方法`ServletException`。可以在单个类中组合任意数量的引发建议方法。以下清单显示了最后一个示例：

爪哇

科特林

```java
public static class CombinedThrowsAdvice implements ThrowsAdvice {

    public void afterThrowing(RemoteException ex) throws Throwable {
        // Do something with remote exception
    }

    public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
        // Do something with all arguments
    }
}
```

|      | 如果throws-advice方法本身引发异常，则它会覆盖原始异常（即，它会将引发给用户的异常更改）。重写异常通常是RuntimeException，它与任何方法签名都兼容。但是，如果throws-advice方法抛出一个已检查的异常，则它必须与目标方法的已声明异常匹配，因此在某种程度上与特定的目标方法签名关联。*不要抛出与目标方法签名不兼容的未声明检查异常！* |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 抛出建议可以与任何切入点一起使用。 |
| ---- | ---------------------------------- |
|      |                                    |

##### 返回建议后

在Spring中，返回通知后必须实现`org.springframework.aop.AfterReturningAdvice`接口，以下清单显示了该 接口：

爪哇

科特林

```java
public interface AfterReturningAdvice extends Advice {

    void afterReturning(Object returnValue, Method m, Object[] args, Object target)
            throws Throwable;
}
```

After After Returning建议可以访问返回值（它不能修改），调用的方法，方法的参数和目标。

返回建议后的以下内容将计数所有未引发异常的成功方法调用：

爪哇

科特林

```java
public class CountingAfterReturningAdvice implements AfterReturningAdvice {

    private int count;

    public void afterReturning(Object returnValue, Method m, Object[] args, Object target)
            throws Throwable {
        ++count;
    }

    public int getCount() {
        return count;
    }
}
```

该建议不会更改执行路径。如果抛出异常，则会将其抛出拦截器链，而不是返回值。

|      | 返回后，建议可以与任何切入点一起使用。 |
| ---- | -------------------------------------- |
|      |                                        |

##### 简介建议

Spring将介绍建议视为一种特殊的拦截建议。

简介需要`IntroductionAdvisor`and `IntroductionInterceptor`来实现以下接口：

爪哇

科特林

```java
public interface IntroductionInterceptor extends MethodInterceptor {

    boolean implementsInterface(Class intf);
}
```

`invoke()`从AOP Alliance `MethodInterceptor`接口继承的方法必须实现介绍。也就是说，如果被调用的方法在引入的接口上，则引入拦截器负责处理方法调用-它不能调用`proceed()`。

简介建议不能与任何切入点一起使用，因为它仅适用于类而不是方法级别。您只能将引入建议与一起使用`IntroductionAdvisor`，该建议具有 以下方法：

爪哇

科特林

```java
public interface IntroductionAdvisor extends Advisor, IntroductionInfo {

    ClassFilter getClassFilter();

    void validateInterfaces() throws IllegalArgumentException;
}

public interface IntroductionInfo {

    Class<?>[] getInterfaces();
}
```

没有`MethodMatcher`任何`Pointcut`介绍建议，因此也没有。只有类过滤是合乎逻辑的。

该`getInterfaces()`方法返回此顾问程序引入的接口。

该`validateInterfaces()`方法在内部使用，以查看引入的接口是否可以由configure的接口实现`IntroductionInterceptor`。

考虑一下Spring测试套件中的一个示例，并假设我们想为一个或多个对象引入以下接口：

爪哇

科特林

```java
public interface Lockable {
    void lock();
    void unlock();
    boolean locked();
}
```

这说明了混合。我们希望能够将建议对象强制转换为`Lockable`，无论它们的类型如何，并调用lock和unlock方法。如果调用该`lock()`方法，则希望所有的setter方法都抛出一个`LockedException`。因此，我们可以添加一个方面，使对象在不了解对象的情况下变得不可变：AOP的一个很好的例子。

首先，我们需要一个`IntroductionInterceptor`繁重的工作。在这种情况下，我们扩展了`org.springframework.aop.support.DelegatingIntroductionInterceptor` 便利类。我们可以`IntroductionInterceptor`直接实现，但是`DelegatingIntroductionInterceptor`在大多数情况下使用 是最好的。

该`DelegatingIntroductionInterceptor`设计将导入委托到真正实现导入接口，隐藏拦截的使用来做到这一点。您可以使用构造函数参数将委托设置为任何对象。默认委托（使用无参数构造函数时）为`this`。因此，在下一个示例中，委托是的`LockMixin`子类`DelegatingIntroductionInterceptor`。给定一个委托（默认情况下为本身），`DelegatingIntroductionInterceptor`实例将查找由委托实现的所有接口（除外`IntroductionInterceptor`），并支持针对任何接口的 介绍。诸如的子类`LockMixin`可以调用该`suppressInterface(Class intf)` 方法来抑制不应公开的接口。但是，无论`IntroductionInterceptor`准备支持多少个接口， `IntroductionAdvisor`用于控制实际公开哪些接口。引入的接口隐藏了目标对同一接口的任何实现。

因此，`LockMixin`扩展`DelegatingIntroductionInterceptor`并实现`Lockable` 自己。超类会自动选择`Lockable`可以支持引入的超类，因此我们无需指定。我们可以通过这种方式引入任意数量的接口。

注意`locked`实例变量的使用。这有效地将附加状态添加到目标对象中保存的状态。

以下示例显示了示例`LockMixin`类：

爪哇

科特林

```java
public class LockMixin extends DelegatingIntroductionInterceptor implements Lockable {

    private boolean locked;

    public void lock() {
        this.locked = true;
    }

    public void unlock() {
        this.locked = false;
    }

    public boolean locked() {
        return this.locked;
    }

    public Object invoke(MethodInvocation invocation) throws Throwable {
        if (locked() && invocation.getMethod().getName().indexOf("set") == 0) {
            throw new LockedException();
        }
        return super.invoke(invocation);
    }

}
```

通常，您无需覆盖该`invoke()`方法。的 `DelegatingIntroductionInterceptor`实现（它调用`delegate`如果引入的方法的方法，否则进行指向连接点），通常就足够了。在当前情况下，我们需要添加一个检查：如果处于锁定模式，则不能调用任何setter方法。

所需的简介仅需要保存一个不同的 `LockMixin`实例并指定所引入的接口（在这种情况下，仅 `Lockable`）。一个更复杂的示例可能引用了引入拦截器（将被定义为原型）。在这种情况下，没有与a相关的配置`LockMixin`，因此我们使用来创建它`new`。以下示例显示了我们的`LockMixinAdvisor`课程：

爪哇

科特林

```java
public class LockMixinAdvisor extends DefaultIntroductionAdvisor {

    public LockMixinAdvisor() {
        super(new LockMixin(), Lockable.class);
    }
}
```

我们可以非常简单地应用此顾问程序，因为它不需要配置。（但是，如果`IntroductionInterceptor`不带 ，就无法使用`IntroductionAdvisor`。）像通常的介绍中一样，顾问程序必须是基于实例的，因为它是有状态的。对于每个建议的对象`LockMixinAdvisor`，我们需要一个实例，因此 需要一个实例`LockMixin`。顾问程序包含建议对象状态的一部分。

我们可以`Advised.addAdvisor()`像其他任何顾问一样，通过使用XML配置中的方法或（推荐方式）以编程方式应用此顾问。下面讨论的所有代理创建选择，包括“自动代理创建器”，都可以正确处理介绍和有状态的混合。

### 6.3。Spring的Advisor API

在Spring中，顾问程序是一个方面，仅包含与切入点表达式关联的单个建议对象。

除了介绍的特殊情况外，任何顾问都可以与任何建议一起使用。 `org.springframework.aop.support.DefaultPointcutAdvisor`是最常用的顾问类。它可以与使用`MethodInterceptor`，`BeforeAdvice`或 `ThrowsAdvice`。

可以在同一AOP代理中的Spring中混合使用顾问和建议类型。例如，您可以在一个代理配置中使用对建议的拦截，抛出建议以及在建议之前。Spring自动创建必要的拦截器链。

### 6.4。使用`ProxyFactoryBean`创建AOP代理

如果将Spring IoC容器（`ApplicationContext`或`BeanFactory`）用于业务对象（应该是！），则要使用Spring的AOP `FactoryBean`实现之一。（请记住，工厂bean引入了一个间接层，使它可以创建其他类型的对象。）

|      | Spring AOP支持还在后台使用了工厂bean。 |
| ---- | -------------------------------------- |
|      |                                        |

在Spring中创建AOP代理的基本方法是使用 `org.springframework.aop.framework.ProxyFactoryBean`。这样可以完全控制切入点，任何适用的建议及其顺序。但是，如果不需要这样的控制，则有一些更简单的选项是可取的。

#### 6.4.1。基本

的`ProxyFactoryBean`，像其它的`FactoryBean`实现中，引入了一个间接的水平。如果您定义一个`ProxyFactoryBean`named `foo`，则引用的对象将`foo`看不到`ProxyFactoryBean`实例本身，而是一个由实现中的`getObject()`方法创建的对象`ProxyFactoryBean`。此方法创建一个包装目标对象的AOP代理。

使用一个`ProxyFactoryBean`或另一个支持IoC的类来创建AOP代理的最重要好处之一是，建议和切入点也可以由IoC管理。这是一项强大的功能，可实现某些其他AOP框架难以实现的方法。例如，受益于依赖注入提供的所有可插入性，建议本身可以引用应用程序对象（目标之外，目标应该在任何AOP框架中可用）。

#### 6.4.2。JavaBean属性

`FactoryBean`与Spring提供的大多数实现相同， `ProxyFactoryBean`该类本身就是JavaBean。其属性用于：

- 指定您要代理的目标。
- 指定是否使用CGLIB（稍后介绍，另请参见[基于JDK和CGLIB的代理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-pfb-proxy-types)）。

继承了一些关键属性`org.springframework.aop.framework.ProxyConfig` （Spring中所有AOP代理工厂的超类）。这些关键属性包括：

- `proxyTargetClass`：`true`如果要代理目标类，而不是目标类的接口。如果将此属性值设置为`true`，则将创建CGLIB代理（另请参见[基于JDK和CGLIB的代理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-pfb-proxy-types)）。
- `optimize`：控制是否将积极的优化应用于通过CGLIB创建的代理。除非您完全了解相关的AOP代理如何处理优化，否则不要随意使用此设置。当前仅用于CGLIB代理。它对JDK动态代理无效。
- `frozen`：如果代理配置为`frozen`，则不再允许更改配置。这对于进行轻微的优化以及在不希望调用者`Advised` 在创建代理后希望调用者能够（通过接口）操纵代理的情况下都是有用的。此属性的默认值为 `false`，因此允许进行更改（例如添加其他建议）。
- `exposeProxy`：确定是否应将当前代理公开在中， `ThreadLocal`以便目标可以访问它。如果目标需要获取代理并将`exposeProxy`属性设置为`true`，则目标可以使用该 `AopContext.currentProxy()`方法。

其他特定的属性`ProxyFactoryBean`包括：

- `proxyInterfaces`：`String`接口名称数组。如果未提供，则使用目标类的CGLIB代理（另请参见[基于JDK和CGLIB的代理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-pfb-proxy-types)）。

- `interceptorNames`：要套用`String`的`Advisor`，拦截器或其他建议名称的数组。顺序很重要，先到先得。也就是说，列表中的第一个拦截器是第一个能够拦截调用的拦截器。

  名称是当前工厂中的Bean名称，包括祖先工厂中的Bean名称。您不能在此提及bean引用，因为这样做会导致 `ProxyFactoryBean`忽略建议的单例设置。

  您可以在拦截器名称后加上星号（`*`）。这样做会导致应用所有顾问Bean，其名称以要应用星号的部分开头。您可以在“ [使用“全局”顾问”中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-global-advisors)找到使用此功能的示例。

- 单例：无论`getObject()`调用该方法的频率如何，工厂是否应返回单个对象。几种`FactoryBean`实现提供了这种方法。默认值为`true`。如果要使用有状态的建议（例如，对于有状态的混合），请使用原型建议以及单例值 `false`。

#### 6.4.3。基于JDK和CGLIB的代理

本节是有关如何`ProxyFactoryBean` 选择为特定目标对象（将被代理）创建基于JDK的代理或基于CGLIB的代理的权威性文档。

|      | 在`ProxyFactoryBean`Spring的1.2.x版和2.0版之间，关于创建基于JDK或CGLIB的代理的行为发生了变化。在`ProxyFactoryBean`现在表现关于与上述的自动检测接口相似的语义 `TransactionProxyFactoryBean`类。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果要代理的目标对象的类（以下简称为目标类）未实现任何接口，则将创建基于CGLIB的代理。这是最简单的情况，因为JDK代理是基于接口的，并且没有接口意味着甚至无法进行JDK代理。您可以插入目标bean并通过设置`interceptorNames`属性来指定拦截器列表。请注意，即使的`proxyTargetClass`属性 `ProxyFactoryBean`已设置为，也会创建基于CGLIB的代理`false`。（这样做没有任何意义，最好将其从bean定义中删除，因为它充其量是多余的，并且在最糟的情况下会造成混淆。）

如果目标类实现一个（或多个）接口，则创建的代理类型取决于的配置`ProxyFactoryBean`。

如果的`proxyTargetClass`属性`ProxyFactoryBean`已设置为`true`，则会创建一个基于CGLIB的代理。这是有道理的，并且符合最小惊讶原则。即使的`proxyInterfaces`属性 `ProxyFactoryBean`已设置为一个或多个完全限定的接口名称，该`proxyTargetClass`属性设置为的事实也会`true`导致基于CGLIB的代理生效。

如果的`proxyInterfaces`属性`ProxyFactoryBean`已设置为一个或多个完全限定的接口名称，则会创建一个基于JDK的代理。创建的代理实现`proxyInterfaces` 属性中指定的所有接口。如果目标类恰好实现了比该`proxyInterfaces`属性中指定的接口更多的接口，那就很好了，但是这些附加接口不是由返回的代理实现的。

如果尚未设置的`proxyInterfaces`属性`ProxyFactoryBean`，但是目标类确实实现了一个（或多个）接口，则 `ProxyFactoryBean`自动检测到目标类实际上确实实现了至少一个接口，并创建了基于JDK的代理。实际代理的接口是目标类实现的所有接口。实际上，这与为目标类提供该`proxyInterfaces`属性实现的每个接口的列表相同。但是，它的工作量大大减少，而且不容易出现印刷错误。

#### 6.4.4。代理接口

考虑一个简单的实际例子`ProxyFactoryBean`。此示例涉及：

- 代理的目标bean。这是`personTarget`示例中的bean定义。
- 一个`Advisor`和`Interceptor`使用提供建议。
- 一个AOP代理bean定义，用于指定目标对象（`personTarget`bean），代理接口以及要应用的建议。

以下清单显示了示例：

```xml
<bean id="personTarget" class="com.mycompany.PersonImpl">
    <property name="name" value="Tony"/>
    <property name="age" value="51"/>
</bean>

<bean id="myAdvisor" class="com.mycompany.MyAdvisor">
    <property name="someProperty" value="Custom string property value"/>
</bean>

<bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor">
</bean>

<bean id="person"
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="com.mycompany.Person"/>

    <property name="target" ref="personTarget"/>
    <property name="interceptorNames">
        <list>
            <value>myAdvisor</value>
            <value>debugInterceptor</value>
        </list>
    </property>
</bean>
```

请注意，该`interceptorNames`属性采用的列表`String`，其中包含当前工厂中的拦截器或顾问程序的Bean名称。您可以在返回之前，之后使用顾问程序，拦截器并引发建议对象。顾问的顺序很重要。

|      | 您可能想知道为什么列表不包含bean引用。这样做的原因是，如果将的singleton属性`ProxyFactoryBean`设置为`false`，则它必须能够返回独立的代理实例。如果任何顾问本身就是原型，则需要返回一个独立的实例，因此必须能够从工厂获得原型的实例。保持引用是不够的。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`person`可以使用前面显示的bean定义代替`Person`实现，如下所示：

爪哇

科特林

```java
Person person = (Person) factory.getBean("person");
```

与普通Java对象一样，在同一IoC上下文中的其他bean可以表达对此的强类型依赖性。以下示例显示了如何执行此操作：

```xml
<bean id="personUser" class="com.mycompany.PersonUser">
    <property name="person"><ref bean="person"/></property>
</bean>
```

`PersonUser`本示例中的类公开了type属性`Person`。就其而言，可以透明地使用AOP代理代替“真实”的人实现。但是，其类将是动态代理类。可以将其强制转换为`Advised`接口（稍后讨论）。

您可以使用匿名内部bean隐藏目标和代理之间的区别。只有`ProxyFactoryBean`定义不同。该建议仅出于完整性考虑。以下示例显示如何使用匿名内部Bean：

```xml
<bean id="myAdvisor" class="com.mycompany.MyAdvisor">
    <property name="someProperty" value="Custom string property value"/>
</bean>

<bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor"/>

<bean id="person" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="com.mycompany.Person"/>
    <!-- Use inner bean, not local reference to target -->
    <property name="target">
        <bean class="com.mycompany.PersonImpl">
            <property name="name" value="Tony"/>
            <property name="age" value="51"/>
        </bean>
    </property>
    <property name="interceptorNames">
        <list>
            <value>myAdvisor</value>
            <value>debugInterceptor</value>
        </list>
    </property>
</bean>
```

使用匿名内部bean的优点是只有一个类型的对象`Person`。如果我们希望防止应用程序上下文的用户获取对未建议对象的引用，或者需要避免使用Spring IoC自动装配的任何歧义，这将非常有用。可以说，还有一个优点是`ProxyFactoryBean`定义是独立的。但是，有时能够从工厂获得未经建议的目标实际上可能是一个优势（例如，在某些测试方案中）。

#### 6.4.5。代理课程

如果您需要代理一类，而不是一个或多个接口，该怎么办？

想象一下，在我们之前的示例中，没有`Person`接口。我们需要建议一个`Person`没有实现任何业务接口的类。在这种情况下，您可以将Spring配置为使用CGLIB代理而不是动态代理。为此，请将前面显示的`proxyTargetClass`属性设置 `ProxyFactoryBean`为`true`。尽管最好对接口而不是对类进行编程，但是在处理遗留代码时，建议未实现接口的类的功能可能会很有用。（通常，Spring并不是规定性的。虽然可以轻松地应用良好实践，但可以避免强制采用特定方法。）

如果需要，即使您有接口，也可以在任何情况下强制使用CGLIB。

CGLIB代理通过在运行时生成目标类的子类来工作。Spring配置此生成的子类以将方法调用委托给原始目标。子类用于实现Decorator模式，并编织在建议中。

CGLIB代理通常应对用户透明。但是，有一些问题要考虑：

- `Final` 不能建议使用方法，因为它们不能被覆盖。
- 无需将CGLIB添加到您的类路径中。从Spring 3.2开始，CGLIB被重新打包并包含在spring-core JAR中。换句话说，基于CGLIB的AOP就像JDK动态代理一样“开箱即用”。

CGLIB代理和动态代理之间几乎没有性能差异。在这种情况下，性能不应作为决定性的考虑因素。

#### 6.4.6。使用“全球”顾问

通过在拦截器名称后附加一个星号，所有具有与该星号之前的部分匹配的Bean名称的顾问程序都将添加到顾问程序链中。如果您需要添加标准的“全局”顾问程序集，这可能会派上用场。以下示例定义了两个全局顾问程序：

```xml
<bean id="proxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="service"/>
    <property name="interceptorNames">
        <list>
            <value>global*</value>
        </list>
    </property>
</bean>

<bean id="global_debug" class="org.springframework.aop.interceptor.DebugInterceptor"/>
<bean id="global_performance" class="org.springframework.aop.interceptor.PerformanceMonitorInterceptor"/>
```

### 6.5。简洁的代理定义

特别是在定义事务代理时，您可能会得到许多类似的代理定义。使用父子bean定义和子bean定义以及内部bean定义可以使代理定义更加简洁明了。

首先，我们为代理创建父模板，bean定义，如下所示：

```xml
<bean id="txProxyTemplate" abstract="true"
        class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributes">
        <props>
            <prop key="*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```

它本身从未实例化，因此实际上可能是不完整的。然后，每个需要创建的代理都是一个子bean定义，它将代理的目标包装为内部bean定义，因为无论如何该目标都不会单独使用。以下示例显示了这样的子bean：

```xml
<bean id="myService" parent="txProxyTemplate">
    <property name="target">
        <bean class="org.springframework.samples.MyServiceImpl">
        </bean>
    </property>
</bean>
```

您可以从父模板覆盖属性。在以下示例中，我们将覆盖事务传播设置：

```xml
<bean id="mySpecialService" parent="txProxyTemplate">
    <property name="target">
        <bean class="org.springframework.samples.MySpecialServiceImpl">
        </bean>
    </property>
    <property name="transactionAttributes">
        <props>
            <prop key="get*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="find*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="load*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="store*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```

请注意，在父bean的示例中，我们通过将`abstract`属性设置为来将父bean定义显式标记为抽象`true`， [如前所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-child-bean-definitions)，因此实际上可能不会实例化它。默认情况下，应用程序上下文（但不是简单的bean工厂）会预先实例化所有单例。因此，重要的是（至少对于单例bean），如果您有一个（父）bean定义仅打算用作模板，并且此定义指定了一个类，则必须确保将`abstract` 属性设置为`true`。否则，应用程序上下文实际上会尝试对其进行实例化。

### 6.6。使用以下程序以编程方式创建AOP代理`ProxyFactory`

使用Spring以编程方式创建AOP代理很容易。这使您可以使用Spring AOP，而无需依赖Spring IoC。

由目标对象实现的接口将被自动代理。以下清单显示了使用一个拦截器和一个顾问程序为目标对象创建代理的过程：

爪哇

科特林

```java
ProxyFactory factory = new ProxyFactory(myBusinessInterfaceImpl);
factory.addAdvice(myMethodInterceptor);
factory.addAdvisor(myAdvisor);
MyBusinessInterface tb = (MyBusinessInterface) factory.getProxy();
```

第一步是构造类型的对象 `org.springframework.aop.framework.ProxyFactory`。您可以使用目标对象创建此对象，如前面的示例中所示，或指定要在备用构造函数中代理的接口。

您可以添加建议（使用拦截器作为一种特殊的建议），建议程序，或同时添加两者，并在的生命周期内对其进行操作`ProxyFactory`。如果添加 `IntroductionInterceptionAroundAdvisor`，则可以使代理实现其他接口。

`ProxyFactory`（继承自`AdvisedSupport`）上还有便捷的方法，可让您添加其他建议类型，例如before和throw通知。 `AdvisedSupport`既是超`ProxyFactory`和`ProxyFactoryBean`。

|      | 在大多数应用程序中，将AOP代理创建与IoC框架集成在一起是最佳实践。通常，我们建议您使用AOP从Java代码外部化配置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 6.7。操作建议对象

但是，无论创建AOP代理，都可以通过使用`org.springframework.aop.framework.Advised`界面来操作它们 。任何AOP代理都可以强制转换为该接口，无论它实现了哪个其他接口。该界面包括以下方法：

爪哇

科特林

```java
Advisor[] getAdvisors();

void addAdvice(Advice advice) throws AopConfigException;

void addAdvice(int pos, Advice advice) throws AopConfigException;

void addAdvisor(Advisor advisor) throws AopConfigException;

void addAdvisor(int pos, Advisor advisor) throws AopConfigException;

int indexOf(Advisor advisor);

boolean removeAdvisor(Advisor advisor) throws AopConfigException;

void removeAdvisor(int index) throws AopConfigException;

boolean replaceAdvisor(Advisor a, Advisor b) throws AopConfigException;

boolean isFrozen();
```

该`getAdvisors()`方法`Advisor`针对已添加到工厂的每个顾问程序，拦截器或其他建议类型返回一个。如果您添加`Advisor`，则在此索引处返回的顾问程序就是您添加的对象。如果您添加了拦截器或其他建议类型，Spring会将其包装在带有始终返回的切入点的顾问中`true`。因此，如果您添加了`MethodInterceptor`，则为此索引返回的顾问`DefaultPointcutAdvisor`程序将返回您的，`MethodInterceptor`并且返回 一个与所有类和方法都匹配的切入点。

该`addAdvisor()`方法可用于添加任何`Advisor`。通常，拥有切入点和建议的顾问是通用的`DefaultPointcutAdvisor`，您可以将其与任何建议或切入点一起使用（但不能用于介绍）。

默认情况下，即使已创建代理，也可以添加或删除顾问程序或拦截器。唯一的限制是不可能添加或删除介绍顾问，因为工厂中的现有代理不会显示界面更改。（您可以从工厂获取新的代理来避免此问题。）

以下示例显示了将AOP代理投射到`Advised`接口上并检查和处理其建议：

爪哇

科特林

```java
Advised advised = (Advised) myObject;
Advisor[] advisors = advised.getAdvisors();
int oldAdvisorCount = advisors.length;
System.out.println(oldAdvisorCount + " advisors");

// Add an advice like an interceptor without a pointcut
// Will match all proxied methods
// Can use for interceptors, before, after returning or throws advice
advised.addAdvice(new DebugInterceptor());

// Add selective advice using a pointcut
advised.addAdvisor(new DefaultPointcutAdvisor(mySpecialPointcut, myAdvice));

assertEquals("Added two advisors", oldAdvisorCount + 2, advised.getAdvisors().length);
```

|      | 尽管无疑存在合法的使用案例，但是否建议（无双关语）修改生产中的业务对象的建议值得怀疑。但是，它在开发中（例如在测试中）非常有用。有时我们发现能够以拦截器或其他建议的形式添加测试代码，并进入我们要测试的方法调用中，这非常有用。（例如，建议可以进入为该方法创建的事务内部，也许可以在将事务标记为回滚之前运行SQL以检查数据库是否已正确更新。） |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

根据创建代理的方式，通常可以设置一个`frozen`标志。在这种情况下，该`Advised` `isFrozen()`方法会返回`true`，并且通过添加或删除来修改建议的任何尝试都会导致`AopConfigException`。冻结建议对象状态的功能在某些情况下很有用（例如，防止调用代码删除安全拦截器）。

### 6.8。使用“自动代理”功能

到目前为止，我们已经考虑过使用一个`ProxyFactoryBean`或类似的工厂bean 显式创建AOP代理。

Spring还允许我们使用“自动代理” Bean定义，该定义可以自动代理选定的Bean定义。它建立在Spring的“ bean后处理器”基础结构上，该基础结构允许在容器加载时修改任何bean定义。

在此模型中，您在XML bean定义文件中设置了一些特殊的bean定义来配置自动代理基础结构。这使您可以声明有资格进行自动代理的目标。您不需要使用`ProxyFactoryBean`。

有两种方法可以做到这一点：

- 通过使用在当前上下文中引用特定bean的自动代理创建器。
- 自动代理创建的一种特殊情况，值得单独考虑：由源级元数据属性驱动的自动代理创建。

#### 6.8.1。自动代理Bean定义

本节介绍了`org.springframework.aop.framework.autoproxy`软件包提供的自动代理创建者 。

##### `BeanNameAutoProxyCreator`

该`BeanNameAutoProxyCreator`班是一个`BeanPostProcessor`能够自动创建一个用于匹配字符串或者通配符名称豆类AOP代理。以下示例显示了如何创建`BeanNameAutoProxyCreator`bean：

```xml
<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
    <property name="beanNames" value="jdk*,onlyJdk"/>
    <property name="interceptorNames">
        <list>
            <value>myInterceptor</value>
        </list>
    </property>
</bean>
```

与一样`ProxyFactoryBean`，有一个`interceptorNames`属性而不是拦截器列表，以允许原型顾问程序具有正确的行为。名为“拦截器”的可以是顾问或任何建议类型。

通常，与自动代理一样，使用的要点`BeanNameAutoProxyCreator`是将相同的配置一致地应用于多个对象，并且配置量最少。将声明式事务应用于多个对象是一种流行的选择。

名称匹配的Bean定义，例如`jdkMyBean`和`onlyJdk`在前面的示例中，是带有目标类的普通旧Bean定义。会自动创建AOP代理`BeanNameAutoProxyCreator`。相同的建议适用于所有匹配的bean。注意，如果使用了顾问程序（而不是前面的示例中的拦截器），则切入点可能会不同地应用于不同的bean。

##### `DefaultAdvisorAutoProxyCreator`

一个更通用，功能极其强大的自动代理创建者是 `DefaultAdvisorAutoProxyCreator`。这会自动在当前上下文中应用合格的顾问程序，而无需在自动代理顾问程序的Bean定义中包括特定的Bean名称。它具有与一致的配置和避免重复的优点`BeanNameAutoProxyCreator`。

使用此机制涉及：

- 指定`DefaultAdvisorAutoProxyCreator`bean定义。
- 在相同或相关的上下文中指定任意数量的顾问。请注意，这些必须是顾问，而不是拦截器或其他建议。这是必需的，因为必须有一个评估的切入点，以检查每个建议是否符合候选bean定义。

该`DefaultAdvisorAutoProxyCreator`自动评估包括在每个advisor中的切入点，看看有什么（如果有的话）的建议，应该适用于每个业务对象（比如`businessObject1`和`businessObject2`在本例中）。

这意味着可以将任意数量的顾问程序自动应用于每个业务对象。如果在任何顾问程序中没有切入点与业务对象中的任何方法匹配，则该对象不会被代理。当为新的业务对象添加Bean定义时，如有必要，它们会自动被代理。

通常，自动代理的优点是使调用者或依赖者无法获得不建议的对象。调用`getBean("businessObject1")`此方法 `ApplicationContext`将返回AOP代理，而不是目标业务对象。（前面显示的“ inner bean”成语也提供了这一好处。）

以下示例创建一个`DefaultAdvisorAutoProxyCreator`bean和本节中讨论的其他元素：

```xml
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="customAdvisor" class="com.mycompany.MyAdvisor"/>

<bean id="businessObject1" class="com.mycompany.BusinessObject1">
    <!-- Properties omitted -->
</bean>

<bean id="businessObject2" class="com.mycompany.BusinessObject2"/>
```

`DefaultAdvisorAutoProxyCreator`如果您希望将相同的建议一致地应用于许多业务对象，则很有用。基础结构定义到位后，您可以添加新的业务对象，而无需包括特定的代理配置。您还可以轻松地添加其他方面（例如，跟踪或性能监视方面），而对配置的更改最少。

该`DefaultAdvisorAutoProxyCreator`支持过滤（通过使用一种命名约定，使得只有特定的顾问进行评估，其允许使用多个不同配置，在同一个工厂AdvisorAutoProxyCreators）和订货。`org.springframework.core.Ordered`如果存在问题，顾问可以实现该接口以确保正确的排序。在`TransactionAttributeSourceAdvisor`前述实施例中使用具有可配置的顺序值。默认设置为无序。

### 6.9。使用`TargetSource`实施

Spring提供了`TargetSource`在`org.springframework.aop.TargetSource`界面中表达 的概念。该接口负责返回实现连接点的“目标对象”。在`TargetSource` 每一个AOP代理处理一个方法调用时实现请求一个目标实例。

使用Spring AOP的开发人员通常不需要直接与`TargetSource`实现一起工作，但这提供了一种强大的手段来支持池化，可热插拔和其他复杂的目标。例如，`TargetSource`通过使用池来管理实例，池可以为每个调用返回不同的目标实例。

如果未指定`TargetSource`，则使用默认实现包装本地对象。每次调用都返回相同的目标（与您期望的一样）。

本节的其余部分描述了Spring附带的标准目标源以及如何使用它们。

|      | 使用自定义目标源时，目标通常需要是原型而不是单例bean定义。这样，Spring可以在需要时创建一个新的目标实例。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 6.9.1。可热交换的目标源

的`org.springframework.aop.target.HotSwappableTargetSource`存在让一个AOP代理的目标进行切换，同时让来电者保持自己对它的引用。

更改目标源的目标会立即生效。该 `HotSwappableTargetSource`是线程安全的。

您可以使用`swap()`HotSwappableTargetSource上的方法更改目标，如以下示例所示：

爪哇

科特林

```java
HotSwappableTargetSource swapper = (HotSwappableTargetSource) beanFactory.getBean("swapper");
Object oldTarget = swapper.swap(newTarget);
```

以下示例显示了必需的XML定义：

```xml
<bean id="initialTarget" class="mycompany.OldTarget"/>

<bean id="swapper" class="org.springframework.aop.target.HotSwappableTargetSource">
    <constructor-arg ref="initialTarget"/>
</bean>

<bean id="swappable" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="targetSource" ref="swapper"/>
</bean>
```

前面的`swap()`调用更改了可交换bean的目标。拥有对该bean的引用的客户端不知道更改，但立即开始达到新目标。

尽管此示例未添加任何建议（不必添加建议来使用`TargetSource`），但任何建议`TargetSource`都可以与任意建议结合使用。

#### 6.9.2。汇集目标源

使用池目标源提供了与无状态会话EJB相似的编程模型，在无状态会话EJB中，维护了相同实例的池，并且方法调用将释放池中的对象。

Spring池和SLSB池之间的关键区别在于，Spring池可以应用于任何POJO。通常，与Spring一样，可以以非侵入性方式应用此服务。

Spring提供对Commons Pool 2.2的支持，该池提供了相当有效的池实现。您需要在`commons-pool`应用程序的类路径上使用Jar才能使用此功能。您也可以子类化 `org.springframework.aop.target.AbstractPoolingTargetSource`以支持任何其他池化API。

|      | 还支持Commons Pool 1.5+，但从Spring Framework 4.2开始不推荐使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下清单显示了一个示例配置：

```xml
<bean id="businessObjectTarget" class="com.mycompany.MyBusinessObject"
        scope="prototype">
    ... properties omitted
</bean>

<bean id="poolTargetSource" class="org.springframework.aop.target.CommonsPool2TargetSource">
    <property name="targetBeanName" value="businessObjectTarget"/>
    <property name="maxSize" value="25"/>
</bean>

<bean id="businessObject" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="targetSource" ref="poolTargetSource"/>
    <property name="interceptorNames" value="myInterceptor"/>
</bean>
```

请注意，目标对象（`businessObjectTarget`在前面的示例中）必须是原型。这使`PoolingTargetSource`实现可以创建目标的新实例，以根据需要扩展池。有关其属性的信息，请参见的[javadoc `AbstractPoolingTargetSource`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframeworkaop/target/AbstractPoolingTargetSource.html)和具体的子类。`maxSize`是最基本的，并且始终保证存在。

在这种情况下，`myInterceptor`是需要在同一IoC上下文中定义的拦截器的名称。但是，您无需指定拦截器即可使用池。如果只希望池化而没有其他建议，则根本不要设置该 `interceptorNames`属性。

您可以将Spring配置为能够将任何池对象强制转换为 `org.springframework.aop.target.PoolingConfig`接口，该接口通过介绍来公开有关池的配置和当前大小的信息。您需要定义类似于以下内容的顾问程序：

```xml
<bean id="poolConfigAdvisor" class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
    <property name="targetObject" ref="poolTargetSource"/>
    <property name="targetMethod" value="getPoolingConfigMixin"/>
</bean>
```

该顾问程序是通过在`AbstractPoolingTargetSource`类上调用便捷方法而获得的 ，因此可以使用`MethodInvokingFactoryBean`。该顾问的名称（`poolConfigAdvisor`，在此处）必须在`ProxyFactoryBean`公开池对象的拦截器名称列表中。

演员表的定义如下：

爪哇

科特林

```java
PoolingConfig conf = (PoolingConfig) beanFactory.getBean("businessObject");
System.out.println("Max pool size is " + conf.getMaxSize());
```

|      | 通常不需要合并无状态服务对象。我们不认为它应该是默认选择，因为大多数无状态对象自然是线程安全的，并且如果缓存了资源，实例池会成问题。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

通过使用自动代理，可以实现更简单的池化。您可以设置`TargetSource`任何自动代理创建者使用的实现。

#### 6.9.3。原型目标源

设置“原型”目标源类似于设置池`TargetSource`。在这种情况下，每次方法调用都会创建目标的新实例。尽管在现代JVM中创建新对象的成本并不高，但是连接新对象（满足其IoC依赖性）的成本可能会更高。因此，没有充分的理由就不应使用此方法。

为此，您可以`poolTargetSource`按如下所示修改前面显示的定义（为清楚起见，我们也更改了名称）：

```xml
<bean id="prototypeTargetSource" class="org.springframework.aop.target.PrototypeTargetSource">
    <property name="targetBeanName" ref="businessObjectTarget"/>
</bean>
```

唯一的属性是目标Bean的名称。在`TargetSource`实现中使用继承 以确保命名一致。与池化目标源一样，目标bean必须是原型bean定义。

#### 6.9.4。`ThreadLocal`目标来源

`ThreadLocal`如果需要为每个传入请求（每个线程）创建一个对象，则目标源很有用。的概念`ThreadLocal`提供了一个JDK范围的功能，可以透明地将资源与线程一起存储。设置a `ThreadLocalTargetSource`几乎与其他类型的目标源所说明的相同，如以下示例所示：

```xml
<bean id="threadlocalTargetSource" class="org.springframework.aop.target.ThreadLocalTargetSource">
    <property name="targetBeanName" value="businessObjectTarget"/>
</bean>
```

|      | `ThreadLocal`在多线程和多类加载器环境中错误使用实例时，实例会带来严重问题（可能导致内存泄漏）。您应该始终考虑在其他一些类中包装threadlocal，并且永远不要直接使用其`ThreadLocal`本身（包装类中除外）。另外，您应该始终记住正确设置和取消设置`ThreadLocal.set(null)`线程本地资源（在后者只是涉及到的调用 ）。在任何情况下都应进行取消设置，因为不取消设置可能会导致出现问题。Spring的 `ThreadLocal`支持为您做到了这一点，应该始终考虑使用 `ThreadLocal`不带其他适当处理代码的实例。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 6.10。定义新的建议类型

Spring AOP被设计为可扩展的。尽管目前在内部使用拦截实现策略，但是除了在建议周围，在建议之前，抛出建议和返回建议之后进行拦截之外，还可以支持任意建议类型。

该`org.springframework.aop.framework.adapter`软件包是一个SPI软件包，可在不更改核心框架的情况下添加对新的自定义建议类型的支持。对自定义`Advice`类型的唯一限制是它必须实现 `org.aopalliance.aop.Advice`标记接口。

有关[`org.springframework.aop.framework.adapter`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/aop/framework/adapter/package-frame.html) 更多信息，请参见javadoc。

## 7.空安全

尽管Java不允许您使用其类型系统来表示空安全性，但是Spring Framework现在在`org.springframework.lang`包中提供了以下注释，以使您可以声明API和字段的空性：

- [`@Nullable`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/lang/Nullable.html)：表示特定参数，返回值或字段可以为的注释`null`。
- [`@NonNull`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/lang/NonNull.html)：表示不能指定特定参数，返回值或字段的注释`null`（分别不需要在参数/返回值和字段`@NonNullApi`以及`@NonNullFields`应用的字段上）。
- [`@NonNullApi`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/lang/NonNullApi.html)：程序包级别的注释，它声明非null为参数和返回值的默认语义。
- [`@NonNullFields`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/lang/NonNullFields.html)：程序包级别的注释，它声明非null为字段的默认语义。

Spring框架本身利用了这些注释，但是它们也可以在任何基于Spring的Java项目中使用，以声明null安全的API和可选的null安全的字段。尚不支持泛型类型参数，varargs和数组元素的可空性，但应在即将发布的版本中使用它们，有关 最新信息，请参见[SPR-15942](https://jira.spring.io/browse/SPR-15942)。可空性声明预计将在Spring Framework版本之间进行微调，包括次要版本。在方法主体内部使用的类型的可空性超出了此功能的范围。

|      | 其他常见的库（例如Reactor和Spring Data）提供了使用类似可空性设置的空安全API，从而为Spring应用程序开发人员提供了一致的总体体验。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 7.1。用例

除了为Spring Framework API可空性提供显式声明外，IDE（例如IDEA或Eclipse）还可以使用这些注释来提供与空安全性相关的有用警告，从而避免`NullPointerException`在运行时出现警告。

由于Kotlin原生支持[null-safety](https://kotlinlang.org/docs/reference/null-safety.html)，因此它们还用于在Kotlin项目中使Spring API为null-safe 。[Kotlin支持文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/languages.html#kotlin-null-safety)中提供了更多详细信息。

### 7.2。JSR-305元注释

Spring注释使用[JSR 305](https://jcp.org/en/jsr/detail?id=305) 注释（休眠但广泛使用的JSR）进行元注释。JSR-305元注释使工具供应商（如IDEA或Kotlin）以通用方式提供了空安全支持，而无需对Spring注释进行硬编码支持。

既不需要也不建议向项目类路径中添加JSR-305依赖项以利用Spring空安全API。只有项目，如基于Spring的库，在他们的代码库使用空安全注解应该增加`com.google.code.findbugs:jsr305:3.0.2` 与`compileOnly`摇篮配置或Maven `provided`范围，以避免编译警告。

## 8.数据缓冲区和编解码器

Java NIO提供了，`ByteBuffer`但是许多库在顶部构建了自己的字节缓冲区API，特别是对于网络操作，其中重用缓冲区和/或使用直接缓冲区对性能有利。例如，Netty具有`ByteBuf`层次结构，Undertow使用XNIO，Jetty使用具有要释放的回调的池字节缓冲区，依此类推。该`spring-core`模块提供了一组抽象，可与各种字节缓冲区API配合使用，如下所示：

- [`DataBufferFactory`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#databuffers-factory) 抽象数据缓冲区的创建。
- [`DataBuffer`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#databuffers-buffer)表示一个字节缓冲区，可以将其 [缓冲](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#databuffers-buffer-pooled)。
- [`DataBufferUtils`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#databuffers-utils) 提供数据缓冲区的实用方法。
- [编](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#codecs)解码[器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#codecs)将流数据缓冲区流解码或编码为更高级别的对象。

### 8.1。 `DataBufferFactory`

`DataBufferFactory` 用于通过以下两种方式之一创建数据缓冲区：

1. 分配一个新的数据缓冲区，可以选择预先指定容量（如果已知），即使容量的实现`DataBuffer`可以按需增长和收缩，该容量也会更有效。
2. 包装一个现有的`byte[]`或`java.nio.ByteBuffer`，用一个`DataBuffer`实现装饰给定的数据，并且不涉及分配。

请注意，WebFlux应用程序不会`DataBufferFactory`直接创建一个而是通过客户端上的`ServerHttpResponse`或访问它`ClientHttpRequest`。工厂的类型取决于基础客户端或服务器，例如 `NettyDataBufferFactory`Reactor Netty `DefaultDataBufferFactory`等。

### 8.2。 `DataBuffer`

该`DataBuffer`界面提供的操作类似，`java.nio.ByteBuffer`但还带来了一些其他好处，其中一些是受Netty启发的`ByteBuf`。以下是部分好处清单：

- 具有独立位置的读取和写入，即不需要调用`flip()`在读取和写入之间交替。
- 与一样，容量可按需扩展`java.lang.StringBuilder`。
- 合并缓冲和引用计数[`PooledDataBuffer`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#databuffers-buffer-pooled)。
- 查看缓冲区`java.nio.ByteBuffer`，`InputStream`或`OutputStream`。
- 确定给定字节的索引或最后一个索引。

### 8.3。 `PooledDataBuffer`

如Javadoc中 [ByteBuffer所述](https://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html)，字节缓冲区可以是直接的也可以是非直接的。直接缓冲区可以驻留在Java堆之外，从而无需复制本机I / O操作。这使得直接缓冲区对于通过套接字接收和发送数据特别有用，但是它们的创建和释放也更昂贵，这导致了缓冲池的想法。

`PooledDataBuffer`是对`DataBuffer`引用计数的帮助，这对于字节缓冲区池至关重要。它是如何工作的？当`PooledDataBuffer`被分配的引用计数为1.呼吁以`retain()`增加计数，而调用`release()`递减它。只要计数大于0，就保证不会释放缓冲区。当计数减少到0时，可以释放池中的缓冲区，这实际上意味着将为缓冲区保留的内存返回到内存池。

请注意`PooledDataBuffer`，在大多数情况下，与其直接操作，不如在`DataBufferUtils`应用释放或保留到 `DataBuffer`的实例（仅当它是的实例）中使用便捷方法`PooledDataBuffer`。

### 8.4。 `DataBufferUtils`

`DataBufferUtils` 提供了许多实用的方法来对数据缓冲区进行操作：

- 如果底层字节缓冲区API支持，则可以通过复合缓冲区将数据缓冲区流连接到可能具有零副本的单个缓冲区中。
- 将`InputStream`或NIO `Channel`变成`Flux`，反之亦然 `Publisher`变成`OutputStream`或NIO `Channel`。
- `DataBuffer`如果缓冲区是的实例，则 释放或保留的方法`PooledDataBuffer`。
- 从字节流中跳过或获取，直到特定的字节数为止。

### 8.5。编解码器

该`org.springframework.core.codec`软件包提供以下策略接口：

- `Encoder`编码`Publisher`为数据缓冲区流。
- `Decoder`解码`Publisher`为更高级别的对象流。

该`spring-core`模块提供`byte[]`，`ByteBuffer`，`DataBuffer`，`Resource`，和 `String`编码器和解码器实现。该`spring-web`模块添加了Jackson JSON，Jackson Smile，JAXB2，协议缓冲区以及其他编码器和解码器。请参阅 WebFlux部分中的[编解码器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-codecs)。

### 8.6。使用`DataBuffer`

使用数据缓冲区时，必须特别小心以确保释放缓冲区，因为它们可能会被[合并](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#databuffers-buffer-pooled)。我们将使用编解码器来说明其工作原理，但是这些概念将更普遍地应用。让我们看看编解码器必须在内部执行哪些操作来管理数据缓冲区。

`Decoder`在创建更高级别的对象之前，A 是最后一个读取输入数据缓冲区的对象，因此，它必须按以下方式释放它们：

1. 如果`Decoder`只需读取每个输入缓冲区并准备立即释放它，则可以通过进行操作`DataBufferUtils.release(dataBuffer)`。
2. 如果`Decoder`是使用`Flux`或`Mono`运营商，如`flatMap`，`reduce`和其他该预取和高速缓存的数据项内部，或者是使用运算符，如 `filter`，`skip`即离开了物品和其他，然后 `doOnDiscard(PooledDataBuffer.class, DataBufferUtils::release)`必须被添加到该组合物链以确保这些缓冲器之前被释放被丢弃，可能还导致错误或取消信号。
3. 如果`Decoder`以任何其他方式保留一个或多个数据缓冲区，则必须确保在完全读取时释放它们，或者在读取和释放缓存的数据缓冲区之前发生错误或取消信号的情况下。

请注意，这`DataBufferUtils#join`提供了一种安全有效的方法来将数据缓冲区流聚合到单个数据缓冲区中。同样`skipUntilByteCount`， `takeUntilByteCount`它们是供解码器使用的其他安全方法。

一个`Encoder`分配其他人必须读取（和释放）的数据缓冲区。因此，`Encoder` 没有太多的工作要做。但是，`Encoder`如果在用数据填充缓冲区时发生序列化错误，则必须注意释放数据缓冲区。例如：

爪哇

科特林

```java
DataBuffer buffer = factory.allocateBuffer();
boolean release = true;
try {
    // serialize and populate buffer..
    release = false;
}
finally {
    if (release) {
        DataBufferUtils.release(buffer);
    }
}
return buffer;
```

an的使用者`Encoder`负责释放其接收的数据缓冲区。在WebFlux应用程序中，的输出`Encoder`用于写入HTTP服务器响应或客户端HTTP请求，在这种情况下，释放数据缓冲区是代码写入服务器响应或客户端请求的责任。 。

请注意，在Netty上运行时，有用于调试[缓冲区泄漏的](https://github.com/netty/netty/wiki/Reference-counted-objects#troubleshooting-buffer-leaks)调试选项 。

## 9.附录

### 9.1。XML模式

附录的此部分列出了与核心容器相关的XML模式。

#### 9.1.1。该`util`模式

顾名思义，`util`标记处理常见的实用程序配置问题，例如配置集合，引用常量等。要在`util`架构中使用标记，您需要在Spring XML配置文件的顶部具有以下前导（代码段中的文本引用了正确的架构，以便`util`您可以使用名称空间中的标记）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:util="http://www.springframework.org/schema/util"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd">

        <!-- bean definitions here -->

</beans>
```

##### 使用 ``

考虑以下bean定义：

```xml
<bean id="..." class="...">
    <property name="isolation">
        <bean id="java.sql.Connection.TRANSACTION_SERIALIZABLE"
                class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean" />
    </property>
</bean>
```

前面的配置使用Spring `FactoryBean`实现（ `FieldRetrievingFactoryBean`）将`isolation`Bean上的属性的值设置为`java.sql.Connection.TRANSACTION_SERIALIZABLE`常量的值。这一切都很好，但是很冗长，并且（不必要地）将Spring的内部管道暴露给最终用户。

以下基于XML Schema的版本更加简洁，清楚地表达了开发人员的意图（“注入此常数值”），并且读起来更好：

```xml
<bean id="..." class="...">
    <property name="isolation">
        <util:constant static-field="java.sql.Connection.TRANSACTION_SERIALIZABLE"/>
    </property>
</bean>
```

###### 从字段值设置Bean属性或构造函数参数

[`FieldRetrievingFactoryBean`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/FieldRetrievingFactoryBean.html) 是`FactoryBean`检索一个`static`或非静态字段值的。它通常用于检索`public` `static` `final`常量，然后可以将其用于为另一个bean设置属性值或构造函数参数。

下面的示例显示如何`static`通过使用[`staticField`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/FieldRetrievingFactoryBean.html#setStaticField(java.lang.String)) 属性来显示字段 ：

```xml
<bean id="myField"
        class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean">
    <property name="staticField" value="java.sql.Connection.TRANSACTION_SERIALIZABLE"/>
</bean>
```

还有一个便捷用法表格，其中将`static`字段指定为Bean名称，如以下示例所示：

```xml
<bean id="java.sql.Connection.TRANSACTION_SERIALIZABLE"
        class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean"/>
```

这的确意味着不再需要选择什么bean `id`（因此，引用它的任何其他bean也都必须使用这个较长的名称），但是这种形式的定义非常简洁，可以很方便地用作内部bean因为`id`不必为bean引用指定，如以下示例所示：

```xml
<bean id="..." class="...">
    <property name="isolation">
        <bean id="java.sql.Connection.TRANSACTION_SERIALIZABLE"
                class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean" />
    </property>
</bean>
```

您还可以访问另一个bean的非静态（实例）字段，如[`FieldRetrievingFactoryBean`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/FieldRetrievingFactoryBean.html) 该类的API文档中所述 。

在Spring中，很容易将枚举值作为属性或构造函数参数注入到bean中。实际上，您不必做任何事情或不了解任何有关Spring内部的知识（甚至不了解诸如之类的类`FieldRetrievingFactoryBean`）。以下示例枚举显示了注入枚举值的难易程度：

爪哇

科特林

```java
package javax.persistence;

public enum PersistenceContextType {

    TRANSACTION,
    EXTENDED
}
```

现在考虑以下类型的setter `PersistenceContextType`和相应的bean定义：

爪哇

科特林

```java
package example;

public class Client {

    private PersistenceContextType persistenceContextType;

    public void setPersistenceContextType(PersistenceContextType type) {
        this.persistenceContextType = type;
    }
}
```

```xml
<bean class="example.Client">
    <property name="persistenceContextType" value="TRANSACTION"/>
</bean>
```

##### 使用 ``

考虑以下示例：

```xml
<!-- target bean to be referenced by name -->
<bean id="testBean" class="org.springframework.beans.TestBean" scope="prototype">
    <property name="age" value="10"/>
    <property name="spouse">
        <bean class="org.springframework.beans.TestBean">
            <property name="age" value="11"/>
        </bean>
    </property>
</bean>

<!-- results in 10, which is the value of property 'age' of bean 'testBean' -->
<bean id="testBean.age" class="org.springframework.beans.factory.config.PropertyPathFactoryBean"/>
```

前面的配置使用Spring `FactoryBean`实现（ `PropertyPathFactoryBean`）创建一个`int`名为（类型为）的bean ，该bean `testBean.age`的值等于bean 的`age`属性`testBean`。

现在考虑以下示例，该示例添加了一个``元素：

```xml
<!-- target bean to be referenced by name -->
<bean id="testBean" class="org.springframework.beans.TestBean" scope="prototype">
    <property name="age" value="10"/>
    <property name="spouse">
        <bean class="org.springframework.beans.TestBean">
            <property name="age" value="11"/>
        </bean>
    </property>
</bean>

<!-- results in 10, which is the value of property 'age' of bean 'testBean' -->
<util:property-path id="name" path="testBean.age"/>
```

元素的`path`属性值``遵循的形式 `beanName.beanProperty`。在这种情况下，它将获取`age`名为的bean 的属性 `testBean`。该`age`属性的值为`10`。

###### 使用``设置bean属性或构造器参数

`PropertyPathFactoryBean`是`FactoryBean`，用于评估给定目标对象上的属性路径。可以直接指定目标对象，也可以通过bean名称指定目标对象。然后，您可以在另一个bean定义中将此值用作属性值或构造函数参数。

以下示例按名称显示了针对另一个bean的路径：

```xml
// target bean to be referenced by name
<bean id="person" class="org.springframework.beans.TestBean" scope="prototype">
    <property name="age" value="10"/>
    <property name="spouse">
        <bean class="org.springframework.beans.TestBean">
            <property name="age" value="11"/>
        </bean>
    </property>
</bean>

// results in 11, which is the value of property 'spouse.age' of bean 'person'
<bean id="theAge"
        class="org.springframework.beans.factory.config.PropertyPathFactoryBean">
    <property name="targetBeanName" value="person"/>
    <property name="propertyPath" value="spouse.age"/>
</bean>
```

在以下示例中，针对内部bean评估路径：

```xml
<!-- results in 12, which is the value of property 'age' of the inner bean -->
<bean id="theAge"
        class="org.springframework.beans.factory.config.PropertyPathFactoryBean">
    <property name="targetObject">
        <bean class="org.springframework.beans.TestBean">
            <property name="age" value="12"/>
        </bean>
    </property>
    <property name="propertyPath" value="age"/>
</bean>
```

还有一种快捷方式，其中Bean名称是属性路径。以下示例显示了快捷方式表格：

```xml
<!-- results in 10, which is the value of property 'age' of bean 'person' -->
<bean id="person.age"
        class="org.springframework.beans.factory.config.PropertyPathFactoryBean"/>
```

这种形式的确意味着在Bean名称中没有选择。对它的任何引用也必须使用same `id`，即path。如果用作内部bean，则根本不需要引用它，如以下示例所示：

```xml
<bean id="..." class="...">
    <property name="age">
        <bean id="person.age"
                class="org.springframework.beans.factory.config.PropertyPathFactoryBean"/>
    </property>
</bean>
```

您可以在实际定义中专门设置结果类型。对于大多数用例来说，这不是必需的，但有时可能会有用。有关此功能的更多信息，请参见javadoc。

##### 使用 ``

考虑以下示例：

```xml
<!-- creates a java.util.Properties instance with values loaded from the supplied location -->
<bean id="jdbcConfiguration" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
    <property name="location" value="classpath:com/foo/jdbc-production.properties"/>
</bean>
```

前面的配置使用Spring `FactoryBean`实现（ `PropertiesFactoryBean`）实例化一个`java.util.Properties`实例，该实例具有从提供的[`Resource`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources)位置加载的值。

下面的示例使用一个`util:properties`元素来进行更简洁的表示：

```xml
<!-- creates a java.util.Properties instance with values loaded from the supplied location -->
<util:properties id="jdbcConfiguration" location="classpath:com/foo/jdbc-production.properties"/>
```

##### 使用 ``

考虑以下示例：

```xml
<!-- creates a java.util.List instance with values loaded from the supplied 'sourceList' -->
<bean id="emails" class="org.springframework.beans.factory.config.ListFactoryBean">
    <property name="sourceList">
        <list>
            <value>pechorin@hero.org</value>
            <value>raskolnikov@slums.org</value>
            <value>stavrogin@gov.org</value>
            <value>porfiry@gov.org</value>
        </list>
    </property>
</bean>
```

前面的配置使用Spring `FactoryBean`实现（ `ListFactoryBean`）来创建`java.util.List`实例，并使用从提供的中获取的值对其进行初始化`sourceList`。

下面的示例使用一个``元素来进行更简洁的表示：

```xml
<!-- creates a java.util.List instance with the supplied values -->
<util:list id="emails">
    <value>pechorin@hero.org</value>
    <value>raskolnikov@slums.org</value>
    <value>stavrogin@gov.org</value>
    <value>porfiry@gov.org</value>
</util:list>
```

您还可以`List`通过使用元素`list-class`上的属性来显式控制实例化和填充的类型的确切类型``。例如，如果我们确实需要`java.util.LinkedList`实例化a，则可以使用以下配置：

```xml
<util:list id="emails" list-class="java.util.LinkedList">
    <value>jackshaftoe@vagabond.org</value>
    <value>eliza@thinkingmanscrumpet.org</value>
    <value>vanhoek@pirate.org</value>
    <value>d'Arcachon@nemesis.org</value>
</util:list>
```

如果没有`list-class`提供属性，则容器选择一个`List`实现。

##### 使用 ``

考虑以下示例：

```xml
<!-- creates a java.util.Map instance with values loaded from the supplied 'sourceMap' -->
<bean id="emails" class="org.springframework.beans.factory.config.MapFactoryBean">
    <property name="sourceMap">
        <map>
            <entry key="pechorin" value="pechorin@hero.org"/>
            <entry key="raskolnikov" value="raskolnikov@slums.org"/>
            <entry key="stavrogin" value="stavrogin@gov.org"/>
            <entry key="porfiry" value="porfiry@gov.org"/>
        </map>
    </property>
</bean>
```

前面的配置使用Spring `FactoryBean`实现（ `MapFactoryBean`）来创建一个`java.util.Map`实例，该实例使用从提供的中获取的键值对进行初始化`'sourceMap'`。

下面的示例使用一个``元素来进行更简洁的表示：

```xml
<!-- creates a java.util.Map instance with the supplied key-value pairs -->
<util:map id="emails">
    <entry key="pechorin" value="pechorin@hero.org"/>
    <entry key="raskolnikov" value="raskolnikov@slums.org"/>
    <entry key="stavrogin" value="stavrogin@gov.org"/>
    <entry key="porfiry" value="porfiry@gov.org"/>
</util:map>
```

您还可以`Map`通过使用元素`'map-class'`上的属性来显式控制实例化和填充的类型的确切类型``。例如，如果我们确实需要`java.util.TreeMap`实例化a，则可以使用以下配置：

```xml
<util:map id="emails" map-class="java.util.TreeMap">
    <entry key="pechorin" value="pechorin@hero.org"/>
    <entry key="raskolnikov" value="raskolnikov@slums.org"/>
    <entry key="stavrogin" value="stavrogin@gov.org"/>
    <entry key="porfiry" value="porfiry@gov.org"/>
</util:map>
```

如果没有`'map-class'`提供属性，则容器选择一个`Map`实现。

##### 使用 ``

考虑以下示例：

```xml
<!-- creates a java.util.Set instance with values loaded from the supplied 'sourceSet' -->
<bean id="emails" class="org.springframework.beans.factory.config.SetFactoryBean">
    <property name="sourceSet">
        <set>
            <value>pechorin@hero.org</value>
            <value>raskolnikov@slums.org</value>
            <value>stavrogin@gov.org</value>
            <value>porfiry@gov.org</value>
        </set>
    </property>
</bean>
```

前面的配置使用Spring `FactoryBean`实现（ `SetFactoryBean`）来创建`java.util.Set`实例，该实例使用从提供的中获取的值进行初始化`sourceSet`。

下面的示例使用一个``元素来进行更简洁的表示：

```xml
<!-- creates a java.util.Set instance with the supplied values -->
<util:set id="emails">
    <value>pechorin@hero.org</value>
    <value>raskolnikov@slums.org</value>
    <value>stavrogin@gov.org</value>
    <value>porfiry@gov.org</value>
</util:set>
```

您还可以`Set`通过使用元素`set-class`上的属性来显式控制实例化和填充的类型的确切类型``。例如，如果我们确实需要`java.util.TreeSet`实例化a，则可以使用以下配置：

```xml
<util:set id="emails" set-class="java.util.TreeSet">
    <value>pechorin@hero.org</value>
    <value>raskolnikov@slums.org</value>
    <value>stavrogin@gov.org</value>
    <value>porfiry@gov.org</value>
</util:set>
```

如果没有`set-class`提供属性，则容器选择一个`Set`实现。

#### 9.1.2。该`aop`模式

这些`aop`标记涉及在Spring中配置所有AOP，包括Spring自己的基于代理的AOP框架以及Spring与AspectJ AOP框架的集成。这些标签在标题为“ [面向方面的Spring编程”](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop)的一章中进行了全面介绍。

为了完整起见，要在`aop`架构中使用标记，您需要在Spring XML配置文件的顶部具有以下前导（代码段中的文本引用了正确的架构，以便`aop`命名空间中的标记可用于您）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- bean definitions here -->

</beans>
```

#### 9.1.3。该`context`模式

该`context`标签处理`ApplicationContext`的配置，涉及到管道-也就是通常不是做了很多的“咕噜”的工作在春季，如豆类是重要的终端用户，而是豆类`BeanfactoryPostProcessors`。以下代码段引用了正确的架构，以便`context`您可以使用命名空间中的元素：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- bean definitions here -->

</beans>
```

##### 使用 ``

此元素激活`${…}`占位符的替换，这些占位符针对指定的属性文件（作为[Spring资源location](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#resources)）解析。此元素是[`PropertySourcesPlaceholderConfigurer`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-placeholderconfigurer)为您设置的便捷机制。如果需要对特定`PropertySourcesPlaceholderConfigurer`设置进行更多控制，则 可以自己将其明确定义为Bean。

##### 使用 ``

此元素激活Spring基础结构以检测Bean类中的注释：

- 春天的[`@Configuration`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-metadata)模型
- [`@Autowired`/`@Inject`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-annotation-config)和`@Value`
- JSR-250的`@Resource`，`@PostConstruct`和`@PreDestroy`（如果可用）
- JPA `@PersistenceContext`和`@PersistenceUnit`（如果有）
- 春天的 [`@EventListener`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#context-functionality-events-annotation)

或者，您可以选择`BeanPostProcessors` 为这些注释显式激活个人。

|      | 这个元素不会激活Spring [`@Transactional`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#transaction-declarative-annotations)注释的处理 。您可以[``](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#tx-decl-explained) 为此目的使用元素。同样，还需要显式 [启用](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#cache-annotation-enable) Spring的 [缓存注释](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#cache-annotations)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 使用 ``

有关[基于注释的容器配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-annotation-config)的部分中详细介绍了此元素。

##### 使用 ``

[在Spring Framework中关于使用AspectJ](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-aj-ltw)进行[加载时编织](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-aj-ltw)的部分中详细介绍了此元素。

##### 使用 ``

在[使用AspectJ通过Spring依赖注入域对象](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-atconfigurable)的部分中详细介绍了此元素。

##### 使用 ``

有关[配置基于注释的MBean导出](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#jmx-context-mbeanexport)的部分中详细介绍了此元素。

#### 9.1.4。Bean模式

最后但并非最不重要的一点是，我们在`beans`架构中具有元素。自框架刚诞生以来，这些元素就一直出现在春季。`beans`此处未显示模式中各种元素的示例，因为它们在[依赖关系和配置中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-properties-detailed)非常全面地涵盖了它们 （实际上，在整[章中也是如此](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans)）。

请注意，您可以向``XML定义添加零个或多个键值对。使用此额外的元数据进行的操作（如果有的话）完全取决于您自己的自定义逻辑（因此，通常仅在您按照标题为[XML Schema Authoring](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xml-custom)的附录中所述编写自己的自定义元素时使用）。

以下示例显示``了周围环境中的元素`` （请注意，如果没有任何逻辑来解释它，则元数据实际上是毫无用处的）。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="foo" class="x.y.Foo">
        <meta key="cacheName" value="foo"/> 
        <property name="name" value="Rick"/>
    </bean>

</beans>
```

|      | 这是示例`meta`元素 |
| ---- | ------------------ |
|      |                    |

在前面的示例中，您可以假设存在一些逻辑，这些逻辑消耗了bean的定义并建立了一些使用提供的元数据的缓存基础结构。

### 9.2。XML模式创作

从2.0版开始，Spring提供了一种机制，该机制可将基于架构的扩展添加到基本Spring XML格式中，以定义和配置bean。本节介绍如何编写自己的自定义XML Bean定义解析器，以及如何将此类解析器集成到Spring IoC容器中。

为了方便使用架构感知的XML编辑器编写配置文件，Spring的可扩展XML配置机制基于XML Schema。如果您不熟悉标准Spring发行版随附的Spring当前的XML配置扩展，则应首先阅读标题为[appendix.html的附录](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/appendix.html#xsd-configuration)。

要创建新的XML配置扩展，请执行以下操作：

1. [编写](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-custom-schema) XML模式以描述您的自定义元素。
2. [编码](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-custom-namespacehandler)自定义`NamespaceHandler`实现。
3. [对](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-custom-parser)一个或多个`BeanDefinitionParser`实现进行[编码](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-custom-parser)（这是完成实际工作的地方）。
4. 向Spring [注册](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-custom-registration)您的新工件。

对于一个统一的示例，我们创建一个XML扩展（一个自定义XML元素），该扩展使我们可以`SimpleDateFormat`（从`java.text`包中）配置类型的对象 。完成后，我们将能够`SimpleDateFormat`如下定义bean类型的定义：

```xml
<myns:dateformat id="dateFormat"
    pattern="yyyy-MM-dd HH:mm"
    lenient="true"/>
```

（我们将在本附录后面提供更多详细的示例。第一个简单示例的目的是引导您完成制作自定义扩展程序的基本步骤。）

#### 9.2.1。编写架构

创建用于Spring的IoC容器的XML配置扩展首先要编写XML模式来描述扩展。对于我们的示例，我们使用以下架构来配置`SimpleDateFormat`对象：

```xml
<!-- myns.xsd (inside package org/springframework/samples/xml) -->

<?xml version="1.0" encoding="UTF-8"?>
<xsd:schema xmlns="http://www.mycompany.example/schema/myns"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        xmlns:beans="http://www.springframework.org/schema/beans"
        targetNamespace="http://www.mycompany.example/schema/myns"
        elementFormDefault="qualified"
        attributeFormDefault="unqualified">

    <xsd:import namespace="http://www.springframework.org/schema/beans"/>

    <xsd:element name="dateformat">
        <xsd:complexType>
            <xsd:complexContent>
                <xsd:extension base="beans:identifiedType"> 
                    <xsd:attribute name="lenient" type="xsd:boolean"/>
                    <xsd:attribute name="pattern" type="xsd:string" use="required"/>
                </xsd:extension>
            </xsd:complexContent>
        </xsd:complexType>
    </xsd:element>
</xsd:schema>
```

|      | 所指示的行包含所有可识别标签的扩展基础（这意味着它们具有`id`可用作容器中的bean标识符的属性）。我们可以使用此属性，因为我们导入了Spring提供的 `beans`名称空间。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

前面的架构使我们可以`SimpleDateFormat`使用``元素在XML应用程序上下文文件中直接配置对象，如以下示例所示：

```xml
<myns:dateformat id="dateFormat"
    pattern="yyyy-MM-dd HH:mm"
    lenient="true"/>
```

请注意，在创建基础结构类之后，前面的XML代码段基本上与以下XML代码段相同：

```xml
<bean id="dateFormat" class="java.text.SimpleDateFormat">
    <constructor-arg value="yyyy-HH-dd HH:mm"/>
    <property name="lenient" value="true"/>
</bean>
```

前面两个片段中的第二个片段在容器中创建了一个bean（由`dateFormat`type 的名称标识`SimpleDateFormat`），并设置了几个属性。

|      | 创建配置格式的基于模式的方法允许与具有模式识别XML编辑器的IDE紧密集成。通过使用正确编写的架构，您可以使用自动完成功能来让用户在枚举中定义的多个配置选项之间进行选择。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 9.2.2。编码a`NamespaceHandler`

除了架构之外，我们还需要a `NamespaceHandler`来解析Spring在解析配置文件时遇到的该特定名称空间的所有元素。对于此示例， `NamespaceHandler`应当注意`myns:dateformat` 元素的解析。

该`NamespaceHandler`界面具有三种方法：

- `init()`：允许使用初始化，`NamespaceHandler`并在使用处理程序之前由Spring调用。
- `BeanDefinition parse(Element, ParserContext)`：当Spring遇到顶级元素（未嵌套在bean定义或其他命名空间中）时调用。此方法本身可以注册Bean定义，返回Bean定义或两者。
- `BeanDefinitionHolder decorate(Node, BeanDefinitionHolder, ParserContext)`：当Spring遇到另一个名称空间的属性或嵌套元素时调用。一个或多个bean定义的修饰（例如）与[Spring支持](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes)的[范围](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes)一起使用 。我们首先突出显示一个简单的示例，而不使用装饰，然后在一个更高级的示例中显示装饰。

尽管您可以`NamespaceHandler`为整个名称空间编写自己的代码（并因此提供解析名称空间中每个元素的代码），但通常情况下，Spring XML配置文件中的每个顶级XML元素都会产生一个bean定义（在我们的示例中，单个`` 元素导致单个`SimpleDateFormat`bean定义）。Spring提供了许多支持这种情况的便利类。在下面的示例中，我们使用`NamespaceHandlerSupport`类：

爪哇

科特林

```java
package org.springframework.samples.xml;

import org.springframework.beans.factory.xml.NamespaceHandlerSupport;

public class MyNamespaceHandler extends NamespaceHandlerSupport {

    public void init() {
        registerBeanDefinitionParser("dateformat", new SimpleDateFormatBeanDefinitionParser());
    }
}
```

您可能会注意到，此类中实际上没有很多解析逻辑。实际上，`NamespaceHandlerSupport`该类具有内置的委托概念。它支持注册任何数量的`BeanDefinitionParser` 实例，在需要解析其名称空间中的元素时将委托该实例。关注点的这种清晰分离使`NamespaceHandler`处理者可以处理其命名空间中所有自定义元素的编排，同时委托进行`BeanDefinitionParsers`XML解析的繁琐工作。这意味着每个`BeanDefinitionParser`元素仅包含用于解析单个自定义元素的逻辑，正如我们在下一步中看到的那样。

#### 9.2.3。使用`BeanDefinitionParser`

`BeanDefinitionParser`如果`NamespaceHandler`遇到遇到已映射到特定bean定义解析器的类型的XML元素（`dateformat`在这种情况下），则使用A。换句话说，`BeanDefinitionParser`负责解析模式中定义的一个不同的顶级XML元素。在解析器中，我们可以访问XML元素（因此也可以访问其子元素），以便我们可以解析自定义XML内容，如以下示例所示：

爪哇

科特林

```java
package org.springframework.samples.xml;

import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.xml.AbstractSingleBeanDefinitionParser;
import org.springframework.util.StringUtils;
import org.w3c.dom.Element;

import java.text.SimpleDateFormat;

public class SimpleDateFormatBeanDefinitionParser extends AbstractSingleBeanDefinitionParser { 

    protected Class getBeanClass(Element element) {
        return SimpleDateFormat.class; 
    }

    protected void doParse(Element element, BeanDefinitionBuilder bean) {
        // this will never be null since the schema explicitly requires that a value be supplied
        String pattern = element.getAttribute("pattern");
        bean.addConstructorArgValue(pattern);

        // this however is an optional property
        String lenient = element.getAttribute("lenient");
        if (StringUtils.hasText(lenient)) {
            bean.addPropertyValue("lenient", Boolean.valueOf(lenient));
        }
    }

}
```

|      | 我们使用Spring提供的功能`AbstractSingleBeanDefinitionParser`来处理创建单个的许多基本工作`BeanDefinition`。 |
| ---- | ------------------------------------------------------------ |
|      | 我们为`AbstractSingleBeanDefinitionParser`超类提供单身`BeanDefinition`代表的类型。 |

在这种简单的情况下，这就是我们要做的。我们单曲的创建 `BeanDefinition`由`AbstractSingleBeanDefinitionParser`超类处理，bean定义的唯一标识符的提取和设置也是如此。

#### 9.2.4。注册处理程序和架构

编码完成。剩下要做的就是让Spring XML解析基础结构了解我们的自定义元素。`namespaceHandler`为此，我们在两个专用属性文件中注册了自定义 和自定义XSD文件。这些属性文件都放置在`META-INF`应用程序的目录中，例如，可以与二进制类一起分发到JAR文件中。Spring XML解析基础结构通过使用这些特殊的属性文件来自动选择您的新扩展，以下两部分将详细介绍其格式。

##### 写作 `META-INF/spring.handlers`

名为的属性文件`spring.handlers`包含XML Schema URI到名称空间处理程序类的映射。对于我们的示例，我们需要编写以下内容：

```
http \：//www.mycompany.example/schema/myns=org.springframework.samples.xml.MyNamespaceHandler
```

（`:`字符是Java属性格式的有效分隔符，因此 `:`URI中的字符需要用反斜杠转义。）

键值对的第一部分（键）是与自定义名称空间扩展关联的URI，并且需要与`targetNamespace` 自定义XSD架构中指定的属性值完全匹配。

##### 编写“ META-INF / spring.schemas”

名为的属性文件`spring.schemas`包含XML架构位置（与架构声明一起引用，在使用该架构作为`xsi:schemaLocation`属性一部分的XML文件中）到类路径资源的映射。需要使用该文件来防止Spring绝对使用默认值`EntityResolver`，该默认值需要Internet访问才能检索架构文件。如果在此属性文件中指定映射，Spring会在类路径上搜索架构（在这种情况下， `myns.xsd`在`org.springframework.samples.xml`包中）。以下代码段显示了我们需要为自定义架构添加的行：

```
http \：//www.mycompany.example/schema/myns/myns.xsd=org/springframework/samples/xml/myns.xsd
```

（请记住该`:`字符必须转义。）

鼓励您在类路径上的`NamespaceHandler`和`BeanDefinitionParser`类旁边部署XSD文件。

#### 9.2.5。在Spring XML配置中使用自定义扩展

使用您自己实现的自定义扩展与使用Spring提供的“自定义”扩展没有什么不同。以下示例``在Spring XML配置文件中使用前面步骤中开发的定制元素：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:myns="http://www.mycompany.example/schema/myns"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.mycompany.example/schema/myns http://www.mycompany.com/schema/myns/myns.xsd">

    <!-- as a top-level bean -->
    <myns:dateformat id="defaultDateFormat" pattern="yyyy-MM-dd HH:mm" lenient="true"/> 

    <bean id="jobDetailTemplate" abstract="true">
        <property name="dateFormat">
            <!-- as an inner bean -->
            <myns:dateformat pattern="HH:mm MM-dd-yyyy"/>
        </property>
    </bean>

</beans>
```

|      | 我们的定制豆。 |
| ---- | -------------- |
|      |                |

#### 9.2.6。更详细的例子

本节提供了一些更详细的自定义XML扩展示例。

##### 在自定义元素中嵌套自定义元素

本节中的示例显示如何编写满足以下配置目标所需的各种工件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:foo="http://www.foo.example/schema/component"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.foo.example/schema/component http://www.foo.example/schema/component/component.xsd">

    <foo:component id="bionic-family" name="Bionic-1">
        <foo:component name="Mother-1">
            <foo:component name="Karate-1"/>
            <foo:component name="Sport-1"/>
        </foo:component>
        <foo:component name="Rock-1"/>
    </foo:component>

</beans>
```

前面的配置在彼此之间嵌套了自定义扩展。``元素实际配置的`Component` 类是该类（在下一个示例中显示）。请注意，`Component`该类如何不公开该`components`属性的setter方法。这使得很难（或几乎不可能）`Component`通过使用setter注入来为该类配置bean定义。以下清单显示了`Component`该类：

爪哇

科特林

```java
package com.foo;

import java.util.ArrayList;
import java.util.List;

public class Component {

    private String name;
    private List<Component> components = new ArrayList<Component> ();

    // mmm, there is no setter method for the 'components'
    public void addComponent(Component component) {
        this.components.add(component);
    }

    public List<Component> getComponents() {
        return components;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

解决此问题的典型方法是创建一个自定义`FactoryBean`，以公开该属性的setter `components`属性。以下清单显示了这样的一个习惯 `FactoryBean`：

爪哇

科特林

```java
package com.foo;

import org.springframework.beans.factory.FactoryBean;

import java.util.List;

public class ComponentFactoryBean implements FactoryBean<Component> {

    private Component parent;
    private List<Component> children;

    public void setParent(Component parent) {
        this.parent = parent;
    }

    public void setChildren(List<Component> children) {
        this.children = children;
    }

    public Component getObject() throws Exception {
        if (this.children != null && this.children.size() > 0) {
            for (Component child : children) {
                this.parent.addComponent(child);
            }
        }
        return this.parent;
    }

    public Class<Component> getObjectType() {
        return Component.class;
    }

    public boolean isSingleton() {
        return true;
    }
}
```

这很好用，但是向最终用户暴露了很多Spring管道。我们要做的是编写一个自定义扩展名，以隐藏所有此Spring管道。如果我们坚持[前面描述的步骤](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-custom-introduction)，那么我们首先创建XSD模式来定义我们的自定义标签的结构，如下面的清单所示：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<xsd:schema xmlns="http://www.foo.example/schema/component"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        targetNamespace="http://www.foo.example/schema/component"
        elementFormDefault="qualified"
        attributeFormDefault="unqualified">

    <xsd:element name="component">
        <xsd:complexType>
            <xsd:choice minOccurs="0" maxOccurs="unbounded">
                <xsd:element ref="component"/>
            </xsd:choice>
            <xsd:attribute name="id" type="xsd:ID"/>
            <xsd:attribute name="name" use="required" type="xsd:string"/>
        </xsd:complexType>
    </xsd:element>

</xsd:schema>
```

再次按照[前面描述的过程](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#xsd-custom-introduction)，然后创建一个自定义`NamespaceHandler`：

爪哇

科特林

```java
package com.foo;

import org.springframework.beans.factory.xml.NamespaceHandlerSupport;

public class ComponentNamespaceHandler extends NamespaceHandlerSupport {

    public void init() {
        registerBeanDefinitionParser("component", new ComponentBeanDefinitionParser());
    }
}
```

接下来是风俗`BeanDefinitionParser`。请记住，我们正在创建一个`BeanDefinition`描述的`ComponentFactoryBean`。以下清单显示了我们的自定义`BeanDefinitionParser`实现：

爪哇

科特林

```java
package com.foo;

import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.ManagedList;
import org.springframework.beans.factory.xml.AbstractBeanDefinitionParser;
import org.springframework.beans.factory.xml.ParserContext;
import org.springframework.util.xml.DomUtils;
import org.w3c.dom.Element;

import java.util.List;

public class ComponentBeanDefinitionParser extends AbstractBeanDefinitionParser {

    protected AbstractBeanDefinition parseInternal(Element element, ParserContext parserContext) {
        return parseComponentElement(element);
    }

    private static AbstractBeanDefinition parseComponentElement(Element element) {
        BeanDefinitionBuilder factory = BeanDefinitionBuilder.rootBeanDefinition(ComponentFactoryBean.class);
        factory.addPropertyValue("parent", parseComponent(element));

        List<Element> childElements = DomUtils.getChildElementsByTagName(element, "component");
        if (childElements != null && childElements.size() > 0) {
            parseChildComponents(childElements, factory);
        }

        return factory.getBeanDefinition();
    }

    private static BeanDefinition parseComponent(Element element) {
        BeanDefinitionBuilder component = BeanDefinitionBuilder.rootBeanDefinition(Component.class);
        component.addPropertyValue("name", element.getAttribute("name"));
        return component.getBeanDefinition();
    }

    private static void parseChildComponents(List<Element> childElements, BeanDefinitionBuilder factory) {
        ManagedList<BeanDefinition> children = new ManagedList<BeanDefinition>(childElements.size());
        for (Element element : childElements) {
            children.add(parseComponentElement(element));
        }
        factory.addPropertyValue("children", children);
    }
}
```

最后，需要通过修改`META-INF/spring.handlers`和`META-INF/spring.schemas`文件，将各种工件注册到Spring XML基础结构中，如下所示：

```
＃在'META-INF / spring.handlers'中
http \：//www.foo.example/schema/component=com.foo.ComponentNamespaceHandler
```

```
＃在'META-INF / spring.schemas'中
http \：//www.foo.example/schema/component/component.xsd=com/foo/component.xsd
```

##### “普通”元素上的自定义属性

编写自己的自定义解析器和关联的工件并不难。但是，有时这不是正确的选择。考虑一个需要将元数据添加到已经存在的bean定义的场景。在这种情况下，您当然不需要编写自己的整个自定义扩展名。相反，您只想向现有的bean定义元素添加一个附加属性。

作为另一个示例，假设您为访问集群[JCache](https://jcp.org/en/jsr/detail?id=107)的服务对象（它不知道）定义了Bean定义 ，并且您想确保在周围的集群中急切启动命名的JCache实例。以下清单显示了这样的定义：

```xml
<bean id="checkingAccountService" class="com.foo.DefaultCheckingAccountService"
        jcache:cache-name="checking.account">
    <!-- other dependencies here... -->
</bean>
```

然后，我们可以创建另一个`BeanDefinition`当 `'jcache:cache-name'`属性被解析。这`BeanDefinition`则初始化命名的JCache我们。我们还可以修改的现有项`BeanDefinition`，以 `'checkingAccountService'`使其依赖于此新的JCache-initializing `BeanDefinition`。以下清单显示了我们的`JCacheInitializer`：

爪哇

科特林

```java
package com.foo;

public class JCacheInitializer {

    private String name;

    public JCacheInitializer(String name) {
        this.name = name;
    }

    public void initialize() {
        // lots of JCache API calls to initialize the named cache...
    }
}
```

现在我们可以进入自定义扩展了。首先，我们需要编写描述自定义属性的XSD架构，如下所示：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<xsd:schema xmlns="http://www.foo.example/schema/jcache"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        targetNamespace="http://www.foo.example/schema/jcache"
        elementFormDefault="qualified">

    <xsd:attribute name="cache-name" type="xsd:string"/>

</xsd:schema>
```

接下来，我们需要创建关联的`NamespaceHandler`，如下所示：

爪哇

科特林

```java
package com.foo;

import org.springframework.beans.factory.xml.NamespaceHandlerSupport;

public class JCacheNamespaceHandler extends NamespaceHandlerSupport {

    public void init() {
        super.registerBeanDefinitionDecoratorForAttribute("cache-name",
            new JCacheInitializingBeanDefinitionDecorator());
    }

}
```

接下来，我们需要创建解析器。请注意，在这种情况下，因为我们要解析XML属性，所以我们编写了a `BeanDefinitionDecorator`而不是a `BeanDefinitionParser`。以下清单显示了我们的`BeanDefinitionDecorator`实现：

爪哇

科特林

```java
package com.foo;

import org.springframework.beans.factory.config.BeanDefinitionHolder;
import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.xml.BeanDefinitionDecorator;
import org.springframework.beans.factory.xml.ParserContext;
import org.w3c.dom.Attr;
import org.w3c.dom.Node;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class JCacheInitializingBeanDefinitionDecorator implements BeanDefinitionDecorator {

    private static final String[] EMPTY_STRING_ARRAY = new String[0];

    public BeanDefinitionHolder decorate(Node source, BeanDefinitionHolder holder,
            ParserContext ctx) {
        String initializerBeanName = registerJCacheInitializer(source, ctx);
        createDependencyOnJCacheInitializer(holder, initializerBeanName);
        return holder;
    }

    private void createDependencyOnJCacheInitializer(BeanDefinitionHolder holder,
            String initializerBeanName) {
        AbstractBeanDefinition definition = ((AbstractBeanDefinition) holder.getBeanDefinition());
        String[] dependsOn = definition.getDependsOn();
        if (dependsOn == null) {
            dependsOn = new String[]{initializerBeanName};
        } else {
            List dependencies = new ArrayList(Arrays.asList(dependsOn));
            dependencies.add(initializerBeanName);
            dependsOn = (String[]) dependencies.toArray(EMPTY_STRING_ARRAY);
        }
        definition.setDependsOn(dependsOn);
    }

    private String registerJCacheInitializer(Node source, ParserContext ctx) {
        String cacheName = ((Attr) source).getValue();
        String beanName = cacheName + "-initializer";
        if (!ctx.getRegistry().containsBeanDefinition(beanName)) {
            BeanDefinitionBuilder initializer = BeanDefinitionBuilder.rootBeanDefinition(JCacheInitializer.class);
            initializer.addConstructorArg(cacheName);
            ctx.getRegistry().registerBeanDefinition(beanName, initializer.getBeanDefinition());
        }
        return beanName;
    }
}
```

最后，我们需要通过修改`META-INF/spring.handlers`和`META-INF/spring.schemas`文件，在Spring XML基础结构中注册各种工件，如下所示：

```
＃在'META-INF / spring.handlers'中
http \：//www.foo.example/schema/jcache=com.foo.JCacheNamespaceHandler
```

```
＃在'META-INF / spring.schemas'中
http \：//www.foo.example/schema/jcache/jcache.xsd=com/foo/jcache.xsd
```

版本5.2.6.RELEASE
最后更新时间2020-04-28 08:14:00 UTC