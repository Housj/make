# Spring Boot自动配置原理

​	自动配置就是帮你把 所有 JavaConfig 类提前写好 并且装载到Spring 容器中

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
**@SpringBootConfiguration**就是 @Configuration，它是Spring框架的注解，标明该类是一个 JavaConfig配置类。
**@ComponentScan**启用组件扫描。
**@EnableAutoConfiguration**注解表示开启Spring Boot自动配置功能，Spring Boot会根据应用的依赖、自定义的bean、classpath下有没有某个类 等等因素来猜测你需要的bean，然后注册到IOC容器中。那 @EnableAutoConfiguration是如何推算出你的需求？首先看下它的定义：

**@AutoConfigurationPackage** 自动扫描当前路径下的包，所以启动类一般都在最外层

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
    Class<?>[] exclude() default {};
    String[] excludeName() default {};
}
```

关注点应该在 **@Import(EnableAutoConfigurationImportSelector.class)**，@Import注解用于导入类，并将这个类作为一个bean的定义注册到容器中，这里它将把 **EnableAutoConfigurationImportSelector**作为bean注入到容器中，而这个类会将所有符合条件的**@Configuration**配置都加载到容器中，看看它的代码：

```java
//DeferredImportSelector处理 EnableAutoConfiguration 自动配置。
//如果需要 @EnableAutoConfiguration 的自定义变体，则也可以将该类作为子类
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
         //返回所以的配置类 configurations
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    
// 这句代码被封装在AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
List<String> factories = newArrayList<String>(newLinkedHashSet<String>(
SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class, this.beanClassLoader)));
}
```

这个类会扫描所有的jar包，将所有符合条件的**@Configuration**配置类注入的容器中，何为符合条件，看看 **META-INF/spring.factories**的文件内容：

```xml
// 来自 org.springframework.boot.autoconfigure下的META-INF/spring.factories
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

整个流程很清晰，EnableAutoConfigurationImportSelector.selectImports()是在容器启动过程中执行：**AbstractApplicationContext.refresh()**。