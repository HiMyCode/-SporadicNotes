

[TOC]



# 一、Spring常用注解

## 1.1、@Configuration

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
    
    boolean proxyBeanMethods() default true;
}
```

- value 参数

  指定配置类 名称。

- proxyBeanMethods  参数

  - proxyBeanMethods = true

    Full 全模式 ：配置类会被代理 ，IOC容器拦截@Bean标注的方法 ，保证多次调用方法返回同一个对象；

  - proxyBeanMethods = false

    Lite 轻量级模式 ：配置类不会被代理，每次调用@Bean标注的方法 ，返回的是不同的对象；

```java
@Configuration
public class MainConfig {
}
```

## 1.2、@Bean

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {
    // 指定返回Bean的 id ；默认为 @Bean标注的方法名。
    @AliasFor("name")
    String[] value() default {};

    // 与value属性互为别名，指定返回Bean的 id。
    @AliasFor("value")
    String[] name() default {};

	// 在spring5.1之后该属性不推荐使用。设置Autowire自动装配时首选的装配方式。 
    @Deprecated
    Autowire autowire() default Autowire.NO;

    // 容器按类型（byType）自动注入时，若该类满足类型条件，是否将该类加入符合条件的bean列表。
    // ture ：作为按类型注入时的候选者; false ：不作为按类型注入时的候选者。
    boolean autowireCandidate() default true;

    // 指定bean创建的初始化方法
    String initMethod() default "";

    // 指定bean的销毁回调方法。
    String destroyMethod() default "(inferred)";
}
```

```java
@Configuration
public class DaoConfig {

    @Bean
    public DruidDataSource dataSource() throws IOException {
        DruidDataSource dataSource = new DruidDataSource();
        // 加载配置文件，作为数据源的初始化属性
        Properties properties = PropertiesLoaderUtils.loadAllProperties("daoconfig/datasource-config.properties");
        dataSource.setConnectProperties(properties);
        // 返回dataSource，spring会把他注册到IOC容器中。
        return dataSource;
    }
}
```



## 1.3、@ComponentScan

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {
    
    // 配置spring扫描的基础包，扫描该基础包及其子包下满足条件的类生成bean组件
    // 默认扫描该配置类所在的包及其子包下的组件
    @AliasFor("basePackages")
    String[] value() default {};

    // 与 value 互为别名
    @AliasFor("value")
    String[] basePackages() default {};

    // 配置要扫描的类的class对象，默认会扫描该类所在的包及其子包下的组件。
    Class<?>[] basePackageClasses() default {};

    、、 
    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;

    ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

    String resourcePattern() default "**/*.class";

    boolean useDefaultFilters() default true;

    ComponentScan.Filter[] includeFilters() default {};

    ComponentScan.Filter[] excludeFilters() default {};

    boolean lazyInit() default false;

    @Retention(RetentionPolicy.RUNTIME)
    @Target({})
    public @interface Filter {
        FilterType type() default FilterType.ANNOTATION;

        @AliasFor("classes")
        Class<?>[] value() default {};

        @AliasFor("value")
        Class<?>[] classes() default {};

        String[] pattern() default {};
    }
}
```









- value 
- useDefaultFilters
- excludeFilters
  - @Filter
    - type = FilterType.ANNOTATION
    - type = FilterType.ASSIGNABLE_TYPE
    - type = FilterType.CUSTOM
- includeFilters

## 1.4、@Scope

## 1.5、@Lazy

## 1.6、@Conditional

## 1.7、@Import

- value 
  - 参数为：类 或 类数组
  - 参数为：ImportSelector
  - 参数为：ImportBeanDefinitionRegistrar



# 二、Spring容器注册Bean

## 2.1、@ComponentScan + @Controller、@Service、@Repository、@Component

## 2.2、@Bean

## 2.3、@Import

## 2.4、FactoryBean

使用 `&`符号获取 FactoryBean 本身对象。



# 三、Spring Bean 生命周期

## 3.1、指定初始化方法

- xml <bean> 元素配置参数：init-method = ""

- 注解 @Bean 配置参数：initMethod = ""

- 实现接口 InitializingBean，重写 afterPropertiesSet方法

  ​	在Bean创建完并属性赋值后调用

- 使用JSR250提供注解标注在方法上 ：@PostConstruct 

  ​	在Bean创建完并属性赋值后调用

- 使用Spring容器提供的后置处理器接口：BeanPostProcessor

  ​	 postProcessBeforeInitialization：在初始化之前

  ​	 postProcessAfterInitialization：在初始化之后

  ​	

## 3.2、指定销毁方法

- xml <bean> 元素配置参数：destory-method = ""

- 注解 @Bean 配置参数：destoryMethod = ""

- 实现接口 DisposableBean，重写 destory 方法

- 使用JSR250提供注解标注在方法上 ：@RreDestory

  ​	在容器销毁bean之前调用