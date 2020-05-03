# Spring + Mybatis

## userMapper 注入的是什么

采用jdk的动态代理，注入Mybatis的代理对象

```java
@Component
public class UserService{
@Autowired
UserMapper userMapper; // 对象？ Mybatis代理对象
@Autowired
User user;	
    public void test(){
   	 userMapper.selectById(1);
    }
}

```

## 数据库连接在哪里打开的

```
mapper.selectById(1);
```

真正查询的时候，不是在sqlSessionFactory.openSession()；



**类 -> beanDefinition -> bean工厂后置处理器 -> bean后置处理器-> 放入单例缓存map -> bean**

**BeanDefinition**

![image-20200503120038598](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200503120038598.png)

## 怎么把对象放入Spring容器

### @Component

```
@Component
public class UserA 
```

### @Configuration

```java
@Configuration
public class AppConfig {
	@Bean
    public UserA userA(){
      return new UserA();
    }
```

```java
@Configuration
@Import(UserA.class)
@Import(MyBeanDefinitionRegistry.class)
public class AppConfig {
```

通过导入ImportBeanDefinitionRegistrar的实现类，可以在它的registerBeanDefinitions方法里实现beanDefinition的注册，可以注册多个

```
public class MyBeanDefinitionRegistry implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        
    }
}
```



### 实现FactoryBean

FactoryBean工厂bean 也是一个bean，它的对象需要通过getObject方法返回

BeanFactory 是spring里IOC的容器对象，比较大的工厂

```java
@Component
public class MyFactoryBean implements FactoryBean {
    @Override
    public Object getObject() throws Exception {
        return new UserC();
    }

    @Override
    public Class<?> getObjectType() {
        return UserC.class;
    }
}
```



## MapperScan注解

扫描mapper路径

```java
@MapperScan("mapper")
public class AppConfig {
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({MapperScannerRegistrar.class})
@Repeatable(MapperScans.class)
public @interface MapperScan {
    String[] value() default {};

    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    Class<? extends Annotation> annotationClass() default Annotation.class;

    Class<?> markerInterface() default Class.class;

    String sqlSessionTemplateRef() default "";

    String sqlSessionFactoryRef() default "";

    Class<? extends MapperFactoryBean> factoryBean() default MapperFactoryBean.class;

    String lazyInitialization() default "";
}
```



## MapperScannerRegistrar 扫描映射并注册

ImportBeanDefinitionRegistrar允许MyBatis映射器扫描的注释配置

使用 @Enable批注允许通过@Component配置注册bean

而实现 BeanDefinitionRegistryPostProcessor 将适用于XML配置

### registerBeanDefinitions()

```java
 * @see MapperFactoryBean
 * @see ClassPathMapperScanner
public class MapperScannerRegistrar implements ImportBeanDefinitionRegistrar, ResourceLoaderAware {
    
public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        AnnotationAttributes mapperScanAttrs = AnnotationAttributes.fromMap(importingClassMetadata.getAnnotationAttributes(MapperScan.class.getName()));
        if (mapperScanAttrs != null) {
            this.registerBeanDefinitions(importingClassMetadata, mapperScanAttrs, registry, generateBaseBeanName(importingClassMetadata, 0));
        }

    }

    void registerBeanDefinitions(AnnotationMetadata annoMeta, AnnotationAttributes annoAttrs, BeanDefinitionRegistry registry, String beanName) {
        BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(MapperScannerConfigurer.class);
        builder.addPropertyValue("processPropertyPlaceHolders", true);
        Class<? extends Annotation> annotationClass = annoAttrs.getClass("annotationClass");
        if (!Annotation.class.equals(annotationClass)) {
            builder.addPropertyValue("annotationClass", annotationClass);
        }

        Class<?> markerInterface = annoAttrs.getClass("markerInterface");
        if (!Class.class.equals(markerInterface)) {
            builder.addPropertyValue("markerInterface", markerInterface);
        }

        Class<? extends BeanNameGenerator> generatorClass = annoAttrs.getClass("nameGenerator");
        if (!BeanNameGenerator.class.equals(generatorClass)) {
            builder.addPropertyValue("nameGenerator", BeanUtils.instantiateClass(generatorClass));
        }

        Class<? extends MapperFactoryBean> mapperFactoryBeanClass = annoAttrs.getClass("factoryBean");
        if (!MapperFactoryBean.class.equals(mapperFactoryBeanClass)) {
            builder.addPropertyValue("mapperFactoryBeanClass", mapperFactoryBeanClass);
        }

        String sqlSessionTemplateRef = annoAttrs.getString("sqlSessionTemplateRef");
        if (StringUtils.hasText(sqlSessionTemplateRef)) {
            builder.addPropertyValue("sqlSessionTemplateBeanName", annoAttrs.getString("sqlSessionTemplateRef"));
        }

        String sqlSessionFactoryRef = annoAttrs.getString("sqlSessionFactoryRef");
        if (StringUtils.hasText(sqlSessionFactoryRef)) {
            builder.addPropertyValue("sqlSessionFactoryBeanName", annoAttrs.getString("sqlSessionFactoryRef"));
        }

        List<String> basePackages = new ArrayList();
        basePackages.addAll((Collection)Arrays.stream(annoAttrs.getStringArray("value")).filter(StringUtils::hasText).collect(Collectors.toList()));
        basePackages.addAll((Collection)Arrays.stream(annoAttrs.getStringArray("basePackages")).filter(StringUtils::hasText).collect(Collectors.toList()));
        basePackages.addAll((Collection)Arrays.stream(annoAttrs.getClassArray("basePackageClasses")).map(ClassUtils::getPackageName).collect(Collectors.toList()));
        if (basePackages.isEmpty()) {
            basePackages.add(getDefaultBasePackage(annoMeta));
        }

        String lazyInitialization = annoAttrs.getString("lazyInitialization");
        if (StringUtils.hasText(lazyInitialization)) {
            builder.addPropertyValue("lazyInitialization", lazyInitialization);
        }

        builder.addPropertyValue("basePackage", StringUtils.collectionToCommaDelimitedString(basePackages));
        registry.registerBeanDefinition(beanName, builder.getBeanDefinition());
    }
    private static String generateBaseBeanName(AnnotationMetadata importingClassMetadata, int index) {
        return importingClassMetadata.getClassName() + "#" + MapperScannerRegistrar.class.getSimpleName() + "#" + index;
    }

    private static String getDefaultBasePackage(AnnotationMetadata importingClassMetadata) {
        return ClassUtils.getPackageName(importingClassMetadata.getClassName());
    }

    static class RepeatingRegistrar extends MapperScannerRegistrar {
        RepeatingRegistrar() {
        }

        public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
            AnnotationAttributes mapperScansAttrs = AnnotationAttributes.fromMap(importingClassMetadata.getAnnotationAttributes(MapperScans.class.getName()));
            if (mapperScansAttrs != null) {
                AnnotationAttributes[] annotations = mapperScansAttrs.getAnnotationArray("value");

                for(int i = 0; i < annotations.length; ++i) {
                    this.registerBeanDefinitions(importingClassMetadata, annotations[i], registry, MapperScannerRegistrar.generateBaseBeanName(importingClassMetadata, i));
                }
            }

        }
    }
```



## ClassPathMapperScanner

一个{@link ClassPathBeanDefinitionScanner}，它通过{@code basePackage}，{@ code注记类}或* {@code markerInterface}注册映射器。如果指定了{@code注记类}和/或{@code markerInterface}，将仅搜索*指定的类型（将禁用搜索所有接口）。

### doScan(String... basePackages)

调用父级搜索，该搜索将搜索并注册所有候选者。然后对注册的对象进行后期处理以将其设置为MapperFactoryBeans

```java
public class ClassPathMapperScanner extends ClassPathBeanDefinitionScanner {
    
public Set<BeanDefinitionHolder> doScan(String... basePackages) {
   //调用Spring的ClassPathBeanDefinitionScanner在指定的基本包中执行扫描，返回注册的bean定义。
  Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);

  if (beanDefinitions.isEmpty()) {
    LOGGER.warn(() -> "No MyBatis mapper was found in '" + Arrays.toString(basePackages)
        + "' package. Please check your configuration.");
  } else {
    processBeanDefinitions(beanDefinitions);
  }

  return beanDefinitions;
}
```