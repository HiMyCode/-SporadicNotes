

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

    // 对应的bean名称的生成器 默认的是BeanNameGenerator
    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    // 处理检测到的bean的scope范围
    Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;

    // 是否为检测到的组件生成代理
    ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

    // 控制符合组件检测条件的类文件,默认是包扫描下的  **/*.class
    String resourcePattern() default "**/*.class";
	
    // 是否对带有@Component @Repository @Service @Controller注解的类开启检测,默认是开启的
    boolean useDefaultFilters() default true;

    // 指定某些定义Filter满足条件的组件 FilterType 配置的类
    // Filter类型：ANNOTATION, 注解类型 默认；ASSIGNABLE_TYPE,指定固定类；
    // 			  ASPECTJ， ASPECTJ类型；REGEX,正则表达式；CUSTOM,自定义类型
    ComponentScan.Filter[] includeFilters() default {};

    // 排除某些过滤器扫描到的类
    ComponentScan.Filter[] excludeFilters() default {};

    // 扫描到的类是都开启懒加载 ，默认是不开启
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

## 1.4、@Scope

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Scope {
    @AliasFor("scopeName")
    String value() default "";

    @AliasFor("value")
    String scopeName() default "";

    // 配置当前类的代理模式
    ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;
}

public enum ScopedProxyMode {
    DEFAULT, NO, 	// 不使用代理，需要就立刻创建
    INTERFACES, 	// 使用jdk实现动态代理
    TARGET_CLASS;	// 使用CGLib实现动态代理
    private ScopedProxyMode()
}
```

**Scope作用域：**

- `singleton`单例模式(**默认**):全局有且仅有一个实例
- `prototype`原型模式:每次获取Bean的时候会有一个新的实例
- `request`: 在Web应用中，为每个请求创建一个bean实例 
- `session` :在web应用中，为每个会话创建一个bean实例 
- `global session` : 类似于标准的HTTP Session作用域，仅在基于portlet的web应用中才有意义

## 1.5、@Lazy

```java
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.CONSTRUCTOR, ElementType.PARAMETER, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Lazy {
    boolean value() default true;
}
```

## 1.6、@Conditional

按照一定条件进行判断，满足条件则给容器中注册bean。

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
    Class<? extends Condition>[] value();
}
```

## 1.7、@Import

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {
    Class<?>[] value();
}
```

- 参数为：类 或 类数组

```java
@Configuration
@Import({Windows.class, Linux.class})
public class ImportConfig{
    
}
```

- 参数为：ImportSelector

```java
public class ImportSelectorConfig implements ImportSelector{
    @Override
    public String[] selectImport(AnnotationMetadata annotationMetadata){
        // 返回需要导入组件的全类名数组
        return new String[]{"com.demo.bean.Blue","com.demo.bean.Red"}
    }
}

@Configuration
@Import({ImportSelectorConfig.class})
public class ImportConfig{
    
}
```

- 参数为：ImportBeanDefinitionRegistrar

```java
public class ImportSelectorDefinitionRegistrarConfig implements ImportSelectorDefinitionRegistrar{
    @Override
    public String[] selectImport(AnnotationMetadata annotationMetadata, BeanDefinitionRegistrar registrar){
        // 将需要添加到容器的Bean手动注册到容器中
        RootBeanDefinition beanDefinition = new RootBeanDefinition(Windows.class);
        beanDefinition.registerBeanDefinition("window", beanDefinition);
    }
}

@Configuration
@Import({ImportSelectorDefinitionRegistrarConfig.class})
public class ImportConfig{
    
}
```



# 二、Spring容器注册Bean

## 2.1、@ComponentScan + @Controller、@Service、@Repository、@Component

## 2.2、@Bean

## 2.3、@Import

一般用于导入第三方包的组件。

## 2.4、FactoryBean

```java
public class ColorFactoryBean implements FactoryBean<Color> {
    @Override
    public Color getObject() throws Exception {
        return new Color();
    }

    @Override
    public Class<?> getObjectType() {
        return Color.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

使用applicationContext.getBean("colorFactoryBean")方法获取的是getObject()方法创建的对象。

使用 `&`符号获取 FactoryBean 本身对象：applicationContext.getBean("&colorFactoryBean")。此前缀在BeanFactory的FACTORY_BEAN_PREFIX中定义。

# 三、Spring Bean 生命周期

## 3.1、指定初始化方法

- xml <bean> 元素配置参数：init-method = ""

- 注解 @Bean 配置参数：initMethod = ""

- 实现接口 InitializingBean，重写 afterPropertiesSet方法

  ​	在Bean创建完并属性赋值后调用

- 使用JSR250提供注解标注在方法上 ：@PostConstruct 

  ​	在Bean创建完并属性赋值后调用

- 使用Spring容器提供的后置处理器接口：BeanPostProcessor

  ​	 postProcessBeforeInitialization：在属性数值后，其他初始化执行之前

  ​	 postProcessAfterInitialization：在其他初始化执行之后

  ​	

## 3.2、指定销毁方法

- xml <bean> 元素配置参数：destory-method = ""

- 注解 @Bean 配置参数：destoryMethod = ""

- 实现接口 DisposableBean，重写 destory 方法

- 使用JSR250提供注解标注在方法上 ：@PreDestory

  ​	在容器销毁bean之前调用



# 四、Spring 自动装配

## 4.1、@Value

```java
// 1.基本数值
@Value("Tom")
private String name;
// 2.SpEL ：#{}
@Value("#{20-2}")
private Integer age;
// 3.${person.nickName}
// 需要在配置类上添加： @PropertySource(value={"classpath:/allpication.properties"})
@Value("Tom")
private String nickName;
```

## 4.2、自动装配注解

### 4.2.1、@Autowired（Spring）

**查找过程：**	

​	按类型找->通过限定符@Qualifier过滤->@Primary->@Priority->根据名称找（字段名称或者参数名称）。

**标注位置：**

​	构造器、属性、方法、方法参数。

​	@Bean标注的方法参数也会自动从容器中获取。方法参数上的@Autowired可省略。

### 4.2.2、@Resource （JSR250）

​	先按Resource的name值作为bean名称找->按名称（字段名称、方法名称、set属性名称）找->按类型找-> 通过限定符@Qualifier过滤->@Primary->@Priority->根据名称找（字段名称或者方法参数名称）。

### 4.2.3、@Inject（JSR330 ）

​	@Inject，无任何可指定参数，但可以配合spring的@Primary、@Qualifier 注解。注入顺序同@Autowired。

​	使用时需要导入依赖：

        ``` xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
        ```

##4.3、@Profile

```java
@Configuration
public class DataSourceConfig {
	// 开发环境下的数据源
	@Bean("dataSource")
	@Profile("dev")
	public DataSource dataSource() {
	    return new EmbeddedDatabaseBuilder()
	        .setType(EmbeddedDatabaseType.H2)
	        .addScript("classpath:/com/hy/java/spring/environment/profile/my-schema.sql")
	        .addScript("classpath:/com/hy/java/spring/environment/profile/my-test-data.sql")
	        .build();
	}
	
	// 生产环境下的数据源
	@Bean("dataSource")
	@Profile("prod")
	public DataSource mySqlDataSource() {
		MysqlDataSource mysqlDataSource = new MysqlDataSource();
		mysqlDataSource.setUser("root");
		mysqlDataSource.setPassword("root");
		mysqlDataSource.setUrl("jdbc:mysql://localhost:3306/test?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC");
	    return mysqlDataSource;
	}
}
```

**要点：**

- @Profile 可使用在类、方法上。
- 加了@Profile的类或方法只有在相应环境被激活时才有效。若不激活任何Profile，默认激活：default。

激活Profile:

- 添加VM arguments参数： -Dspring.profile.actice=test
- 在ApplicationContext容器refresh前设置：application.getEnviroment().setActiveProfiles("test","dev");



# 五、Spring AOP

## 5.1、



## 5.2、



## 5.3、





## 5.4、





## 5.5、

