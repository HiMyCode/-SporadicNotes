[TOC]



# 一、Spring Boot 入门



## 1.1、SpringBoot项目的启动

### 1.1.1、自动配置原理

1. 使用**@SpringBootApplication**标注SpringBoot主配置类

   - **@EnableAutoConfiguration**开启了自动配置    
     - **@AutoConfigurationPackage**

       添加该注解的类所在的package 作为自动配置package进行管理。

     - **@Import({AutoConfigurationImportSelector.class})**

     		扫描所有jar包类路径下 META‐INF/spring.factories；
	
     		把扫描到的文件解析成properties对象；
				
     		从properties中获取到EnableAutoConfiguration.class类（类名）的值，将其添加到容器。    

### 1.1.2、自动配置类原理

​	AutoConfigurationImportSelector.class 用于将所有的自动配置类添加到容器中，每一个自动配置类都会进行自动配置功能 。以HttpEncodingAutoConfiguration（Http编码自动配置）为例   ：

```java
// 标识一个配置类，可以给容器中添加组件
@Configuration
// 启动指定类的ConfigurationProperties功能：将配置文件中对应的值和HttpEncodingProperties绑定，并把HttpEncodingProperties加入到ioc容器中
@EnableConfigurationProperties(HttpEncodingProperties.class) 
// 为@Conditional注解扩展，判断当前应用是否是web应用，如果是，当前配置类生效
@ConditionalOnWebApplication 
// 为@Conditional注解扩展，判断当前项目有无CharacterEncodingFilter（SpringMVC中进行乱码解决的过滤器）；若无，则自动配置
@ConditionalOnClass(CharacterEncodingFilter.class) 
//判断配置文件中是否存在spring.http.encoding.enabled配置；若不存在，默认为:true
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing =
true) 
public class HttpEncodingAutoConfiguration {
    // SpringBoot配置文件属性映射类
    private final HttpEncodingProperties properties;
    // 只有一个有参构造器的情况下，参数的值就会从容器中拿
    public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
    	this.properties = properties;
    }
    
    // 判断容器中若没有CharacterEncodingFilter.class组件,则注册一个
    @Bean
    @ConditionalOnMissingBean(CharacterEncodingFilter.class) 
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
        return filter;
    }
}

//从配置文件中获取指定的值和bean的属性进行绑定
@ConfigurationProperties(prefix = "spring.http.encoding") 
public class HttpEncodingProperties {
	public static final Charset DEFAULT_CHARSET = Charset.forName("UTF‐8");
}
```



## 1.2、SpringBoot配置文件

### 1.2.1、配置文件的加载位置

#### 1.2.1.1、默认配置文件加载（项目内）

默认扫描路径，优先级由高到底，高优先级配置会覆盖低优先级配置：

- file:./config/ 
- file:./ 
- classpath:/config/ 
- classpath:/    

```xml
<!-- 在当前项目的父项目pom.xml文件中,默认配置文件路径：src/main/resources -->
<include>**/application*.yml</include>
<include>**/application*.yaml</include>
<include>**/application*.properties</include>
```

####1.2.1.2、默认配置文件加载（项目外）

- 指定启动时默认加载的配置文件

​       项目打包后，可使用命令行参数的形式，指定项目启动时加载配置文件位置。指定的配置文将和默认加载的配置文件形成互补配置，共同起作用。

```shell
java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.location=G:/application.properties
```

- 所有外部配置文件加载方式及优先级顺序：

1. 命令行参数 

   ```shell
   ## 所有的配置都可以在命令行上进行指定，多个配置用空格分开； --配置项=值。
   java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087 --server.context-path=/abc    
   ```

2. 来自java:comp/env的JNDI属性

3. Java系统属性

   System.getProperties() 

4. 操作系统环境变量 

5. RandomValuePropertySource配置的random.*属性值 

   **（6）-（9）由jar包外向jar包内进行寻找； 优先加载带profile** 

6. jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件 

7. jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件

8. jar包外部的application.properties或application.yml(不带spring.profile)配置文件 

9. jar包内部的application.properties或application.yml(不带spring.profile)配置文件 

10. @Configuration注解类上的@PropertySource 

11. 通过SpringApplication.setDefaultProperties指定的默认属性   

**配置文件加载官方参考：**

https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config

**配置文件参数配置官方参考：**

https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#common-application-properties

####1.2.1.3、加载指定位置的配置文件（xml）

@ImportResource：导入Spring的配置文件，让配置文件的内容生效    

```java

// classpath路径：src/main/resources
@ImportResource(locations = {"classpath:beans.xml"})
@Configuration
public class MyAppConfig {
	
}
```



### 1.2.2、从配置文件中获取属性

#### 1.2.2.1、从全局配置文件中获取属性

#####1.2.2.1.1、@ConfigurationProperties

```java
@Component
@ConfigurationProperties(prefix = "person")	// 将类中所有属性和配置文件中相关的配置进行绑定    
@Validated
public class Person {
    @Email
    private String lastName;
    private Integer age;
    private Boolean boss;
}
```

#####1.2.2.1.2、@Value

```java
@Component
public class Person {
    @Value("${person.last‐name}")	 // 从环境变量中获取 
    private String lastName;
    @Value("#{11*2}")				// 使用SEpl计算
    private Integer age;
    @Value("true")					// 字面量
    private Boolean boss;
}
```

#####1.2.2.1.3、@ConfigurationProperties && @Value对比

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

####1.2.2.2、从指定配置文件中获取属性

@PropertySource：加载指定的配置文件    

```xml
@Component
@PropertySource(value = {"classpath:person.properties"})
@ConfigurationProperties(prefix = "person")
public class Person {
    private String lastName;
    private Integer age;
    private Boolean boss;
}
```

### 1.2.3、配置文件占位符

#### 1.2.3.1、随机数

```yaml
randomUUID: ${random.uuid}
randomInt: ${random.int}
randomLong: ${random.long}
randomFixed: ${random.int(10)}
randomScope: ${random.int[1024,65536]}
```

####1.2.3.2、获取已配置的值，并指定默认值    

```yaml
person.name: Tom
persom.fullname: Tom Fork
persom.mark: ${person.name:Tom} say hello to you!
```

###1.2.4、多环境的配置文件

#### 1.2.4.1、多个profile文件

通过文件名去区分不同的profile环境，文件名格式：application-{profile}.properties.yml 。例如：

- application-dev.properties.yml  
- application-test.properties.yml  
- application-prod.properties.yml   

#### 1.2.4.2、单个文件多profile文档块

```yaml
server:
	port: 8081
spring:
	profiles:
		active: prod
‐‐‐
server:
	port: 8083
spring:
	profiles: dev
‐‐‐
server:
	port: 8084
spring:
	profiles: prod 
```

#### 1.2.4.3、profile的激活

1. 在配置文件中指定

   spring.profiles.active=dev    

2. 在启动命令行中指定

   java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev

3. 在虚拟机参数中指定

   Dspring.profiles.active=dev    



# 二、SpringBoot日志框架 

## 2.1、框架简介

- 日志门面

  - JCL（Jakarta Commons Logging） 

    Apache提供，最后更新时间2014年。

  - jboss-logging 

  - **SLF4j（Simple Logging Facade for Java）**

- 日志实现   

  - Log4j 

    支持LF4j，但存在性能问题。

  - Logback   

    Log4j 的升级版本，支持LF4j，且性能良好。

  - JUL（java.util.logging）

    Apache提供，功能一般。

  -  Log4j2  

    Apache提供的优秀框架，但尚未普遍使用。

**使用情况：**

Spring框架默认：**JCL**；

SpringBoot默认：**SLF4j + Logback**。

## 2.2、日志框架的原则

开发时，代码中应调用日志抽象层slf4j的方法； 同时需要给系统中导入 logback的实现jar 。

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(HelloWorld.class);
        logger.info("Hello World");
    }
}
```

   **日志配置：**

​	使用slf4j后，日志的配置文件还是要使用日志实现框架要求格式的配置文件。

## 2.3、日志框架适配

**适配图：**

![日志适配图](..\图片\日志框架适配图-1.png)

​    

![日志框架适配图-1](..\图片\日志框架适配图-1.png)



**问题:**

​	现新建项目使用日志框架：slf4j + logback

​	项目使用的 Spring框架使用日志框架：commons-logging

​			     Hibernate使用日志框架：jboss-logging

​			     MyBatis使用日志框架： log4j 等等

​	如何让系统中所有的日志都统一使用slf4j进行输出？    

**方案：**

1. 将系统中其他日志框架先排除出去；
2. 用中间包来替换原有的日志框架
3. 导入slf4j其他的实现    

**适配原则：**

​	SpringBoot底层使用slf4j+logback的方式记录日志，而且能自动适配所有的日志。引入其他框架的时，只需把引入框架依赖的日志框架排除掉即可 。 



## 2.4、SpringBoot 日志的使用

### 2.4.1、默认配置 

```java

Logger logger = LoggerFactory.getLogger(getClass());

@Test
public void contextLoads() {
    // SpringBoot默认使用info日志级别
    logger.trace("这是trace日志...");
    logger.debug("这是debug日志...");
    logger.info("这是info日志...");
    logger.warn("这是warn日志...");
    logger.error("这是error日志...");
}
```

### 2.4.1、自定义配置

```yaml
## 修改配置文件 application.properties

## 指定全局日志级别
logging.level.root=info

## 指定com.common包下的日志为error级别
## 其他未指定级别的包就用SpringBoot的root级别（所有级别）。
## logging.level.com.common=error

## 指定日志路径：日志输出到指定路径的 spring.log 文件中
# logging.path=
## 仅指定日志路径：日志输出到当前项目下的 springboot.log 文件中
## logging.file 也可以指定完整的路径
logging.file=G:/springboot.log

## 指定日志输出格式
# 在控制台输出的日志格式
logging.pattern.console=%d{yyyy‐MM‐dd} [%thread] %‐5level %logger{50} ‐ %msg%n
# 指定文件中日志输出格式
logging.pattern.file=%d{yyyy‐MM‐dd} === [%thread] === %‐5level === %logger{50} ==== %msg%n

```



1. 参数 **logging.file** 与 参数 **logging.path**    

| logging.file | logging.path | Example        | Description                        |
| ------------ | ------------ | -------------- | ---------------------------------- |
| (none)       | (none)       | 只在控制台输出 |                                    |
| 指定文件名   | (none)       | my.log         | 输出日志到当前项目下的my.log文件   |
| (none)       | 指定目录     | /var/log       | 输出到指定目录的 spring.log 文件中 |

2. 日志输出格式参数说明

	 %d	表示日期时间
- %thread  表示线程名
	 %‐5level	 级别从左显示5个字符宽度
	 %logger{50}	 表示logger名字最长50个字符，否则按照句点分割
- %msg  日志消息
- %n 是换行符

**示例：**
	%d{yyyy‐MM‐dd HH:mm:ss.SSS} [%thread] %‐5level %logger{50} ‐ %msg%n



3. 指定配置

   给类路径下放上每个日志框架自己的配置文件即可；SpringBoot就不使用他默认配置的了    

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | logback-spring.xml , logback-spring.groovy , logback.xml or logback.groovy |
| Log4j2                  | log4j2-spring.xml or log4j2.xml                              |
| JDK (Java Util Logging) | logging.properties                                           |

4. SpringBoot日志的profile特性

- logback.xml

  logback.xml 为上述列表中的文件，此配置将直接被日志框架识别 。SpringBoot的默认日志配置将不生效。

- logback-spring.xml

  logback-spring.xml 为上述列表中的文件添加了spring后缀，此配置将被SpringBoot加载，可使用SpringBoot 的高级Profile功能：指定某段配置只在某个环境下生效。

```xml
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
    <layout class="ch.qos.logback.classic.PatternLayout">
        <!‐‐ configuration to be enabled when the "dev" profile is active ‐‐>
        <springProfile name="dev">
            <pattern>
              %d{yyyy‐MM‐dd HH:mm:ss.SSS} ‐> [%thread] ‐> %‐5level %logger{50} -%msg%n
            </pattern>
        </springProfile>
        <springProfile name="dev,prod">
            <pattern>
              %d{yyyy‐MM‐dd HH:mm:ss.SSS} ‐> [%thread] ‐> %‐5level %logger{50} -%msg%n
            </pattern>
        </springProfile>
        <!‐‐ configuration to be enabled when the "dev" profile is not active ‐‐>
        <springProfile name="!dev">
            <pattern>
             %d{yyyy‐MM‐dd HH:mm:ss.SSS} == [%thread] == %‐5level %logger{50} ‐ %msg%n
            </pattern>
        </springProfile>
    </layout>
</appender>
```



##  2.5、SpringBoot 日志切换

原理、实现参照《2.4、日志框架适配》。





# 三、WEB开发

## 3.1、SpringBoot对SpringMVC的自动配置    

官方网址：https://docs.spring.io/spring-boot/docs/2.3.9.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-auto-configuration

### 3.1.1、Spring MVC 自动配置项    

SpringBoot配置类**WebMvcAutoConfiguration**对SpringMVC的默认配置 ：

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

  - 自动配置了视图解析器 
  - ContentNegotiatingViewResolver：组合所有的视图解析器的，一同其作用
  - 自定义视图解析器：向容器中添加一个`ViewResolver`类型的视图解析器    

- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.3.9.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-static-content))).

  - 静态资源文件夹路径,webjars

- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.

  - 注册 `Converter`转换器    

  - 注册 `Formatter` 转换器    

    ```java
    @Bean
    // 在文件中配置日期格式化的规则
    @ConditionalOnProperty(prefix = "spring.mvc", name = "date‐format")
    public Formatter<Date> dateFormatter() {
    	return new DateFormatter(this.mvcProperties.getDateFormat());
    }
    ```

- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.3.9.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-message-converters)).

  - 自定义转换器：向容器中添加一个`HttpMessageConverters`类型的视图解析器   

- Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.3.9.RELEASE/reference/htmlsingle/#boot-features-spring-message-codes)).

  - 定义错误代码生成规则    

- Static `index.html` support.

  - 配置静态首页访问    

- Custom `Favicon` support (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.3.9.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-favicon)).

  - 配置 favicon.ico    

- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.3.9.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-web-binding-initializer)).

  - 配置请求数据绑定
  - 自定义数据绑定：向容器中添加一个`ConfigurableWebBindingInitializer`类型的数据绑定器，替换默认数据绑定器   

### 3.1.2、Spring MVC 扩展

If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring/docs/5.2.13.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without**`@EnableWebMvc`.

```java
// 使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        super.addViewControllers(registry);
        registry.addViewController("/atguigu").setViewName("success");
    }
}
```

If you want to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, and still keep the Spring Boot MVC customizations, you can declare a bean of type `WebMvcRegistrations` and use it to provide custom instances of those components.

**自动扩展原理：**

1. 编写SpringMVC自动配置类：`WebMvcAutoConfiguration`    

   ```java
   @Configuration
   public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
       
       private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();
      
       //从容器中获取所有的WebMvcConfigurer
       @Autowired(required = false)
       public void setConfigurers(List<WebMvcConfigurer> configurers) {
           if (!CollectionUtils.isEmpty(configurers)) {
           this.configurers.addWebMvcConfigurers(configurers);
           //一个参考实现；将所有的WebMvcConfigurer相关配置都来一起调用；
           @Override
           // public void addViewControllers(ViewControllerRegistry registry) {
           // for (WebMvcConfigurer delegate : this.delegates) {
           // delegate.addViewControllers(registry);
           // }
           }
       }
   }
   ```

2. SpringBoot自动配置时会导入上述配置文件，@Import(EnableWebMvcConfiguration.class) ，然后和容器中的其他WebMvcConfigurer一起起作用。

### 3.1.3、Spring MVC 全面接管

If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`, or alternatively add your own `@Configuration`-annotated `DelegatingWebMvcConfiguration` as described in the Javadoc of `@EnableWebMvc`.

```java
//使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
@EnableWebMvc
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/atguigu").setViewName("success");
    }
}
```

**`@EnableWebMvc`原理：**

```java
// 导入组件DelegatingWebMvcConfiguration
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc { 
}

// DelegatingWebMvcConfiguration 仅支持SpringMVC最基本的功能
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
}

@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
WebMvcConfigurerAdapter.class })
//容器中没有DelegatingWebMvcConfiguration组件时，自动配置类才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
    
}
```

### 3.1.4、Spring MVC 部分配置项  

#### 3.1.4.1、SpringBoot对静态资源的映射规则

1. 访问以jar包的方式引入静态资源

   默认在所有项目的classpath:/META-INF/resources/webjars/ 路径下找资源。

   ```xml
   <!‐‐ 引入jquery‐webjar -->
   <dependency>
       <groupId>org.webjars</groupId>
       <artifactId>jquery</artifactId>
       <version>3.3.1</version>
   </dependency>
   ```

   

2. 访问以jar包的方式引入静态资源，默认在以下静态资源文件夹查找

   - "classpath:/META‐INF/resources/"
   - "classpath:/resources/"
   - "classpath:/static/"
   - "classpath:/public/"
   - "/"：当前项目根路径    

3. 欢迎页

   在静态资源文件夹下的所有index.html页面找；被"/**"映射 。

4. 网站浏览器tab页图标

   在静态资源文件夹下的所有找 **/favicon.ico 。

### 3.1.5、总结

1. SpringBoot在自动配置大多组件时：

   先看容器中有无用户自定义配置（@Bean、@Component），如有则使用用户配置的；没有时才自动配置；如果有些组件可以有多个（ViewResolver），则将用户自定义配置的和自动默认配置组合起来，共同生效。

2. SpringBoot中有非常多的**xxxConfigurer**帮助我们进行扩展配置 

3. SpringBoot中会有很多的**xxxCustomizer**帮助我们进行定制配置    



## 3.2、thymeleaf

### 3.2.1、引入Thymeleaf

```xml
<!-- 切换thymeleaf版本 -->
<properties>
    <thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
    <thymeleaf‐layout‐dialect.version>2.2.2</thymeleaf‐layout‐dialect.version>
</properties>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring‐boot‐starter‐thymeleaf</artifactId>
</dependency>
```

- Thymeleaf 属性配置类

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {
    
    private static final Charset DEFAULT_ENCODING = Charset.forName("UTF‐8");
    private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");
    // 只需把HTML页面放在classpath:/templates/下，thymeleaf 就能自动渲染
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";

    // ...
}
```

- Thymeleaf 示例文件

```html
<!DOCTYPE html>
<!-- 导入thymeleaf的名称空间 -->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF‐8">
        <title>Title</title>
    </head>
    <body>
        <h1>成功！</h1>
        <!‐‐th:text 将div里面的文本内容设置为 ${hello} 的取值‐‐>
        <div th:text="${hello}">这是显示欢迎信息</div>
    </body>
</html>
```



## 3.3、





## 3.4、SpringBoot异常处理机制

### 3.4.1、SpringBoot默认异常处理机制策略

(1) 浏览器访问

​	根据请求头中有**Accept:text/html**确定。返回错误页面。

(2) 客户端访问

​	根据请求头中无**Accept:text/html**确定。返回错误JSON数据。

### 3.4.2、SpringBoot默认异常处理原理

​	SpringBoot使用**ErrorMvcAutoConfiguration**进行错误处理的自动配置。该类为容器中添加一下组件：

1. **ErrorPageCustomizer**    

   当系统出现**4xx**或**5xx**错误时，ErrorPageCustomizer就会生效（定制错误的响应规则）。从配置文件中取出配置错误页面的解析路径：@Value("${error.path:/error}")

2. **BasicErrorController**   

   处理默认/error请求：@RequestMapping({"${server.error.path:${error.path:/error}}"})。

   - 根据请求头中有无**Accept:text/html**确定。返回

     - 错误页面

       调用DefaultErrorViewResolver进行解析。

     - 错误JSON数据

3. **DefaultErrorAttributes**    

   返回错误具体信息：

   - timestamp：时间戳 
   - status：状态码 
   - error：错误提示  
   - exception：异常对象 
   - message：异常消息 
   - errors：JSR303数据校验的错误都在这里    

4. **DefaultErrorViewResolver**

   根据错误异常码httpStatus进行解析 ：

   - 有模板引擎

     在templates目录下查找视图：error/httpStatus.html

   - 无模板引擎

     在静态资源目录下查找视图：error/httpStatus.html

   若无精确匹配的httpStatus.html页面。将根据是否匹配默认的**4xx**或**5xx**错误进行匹配。若仍不匹配，则返回

SpringBoot默认的错误提示页面。









## 3.5、配置嵌入式Servlet容器    

### 3.5.1、修改Servlet容器相关配置

#### 3.5.1.1、配置ServerProperties   

```properties
## 通用Servlet容器设置：server.xxx
server.port=8081
server.context‐path=/crud

## Tomcat容器设置：server.tomcat.xxx
server.tomcat.uri‐encoding=UTF‐8
```

#### 3.5.1.2、配置EmbeddedServletContainerCustomizer    

```java
// 通过定制嵌入式Servlet容器，修改Servlet容器的配置
@Bean 
public EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer(){
    return new EmbeddedServletContainerCustomizer() {
    	//定制嵌入式的Servlet容器相关的规则
        @Override
        public void customize(ConfigurableEmbeddedServletContainer container) {
        	container.setPort(8083);
   	 	}
    }
}
```



### 3.5.2、注册Servlet三大组件

#### 3.5.2.1、Servlet

```java
@Bean
public ServletRegistrationBean myServlet(){
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(new
    MyServlet(),"/myServlet");
    return registrationBean;
}
```

#### 3.5.2.2、Filter

```java
@Bean
public FilterRegistrationBean myFilter(){
    FilterRegistrationBean registrationBean = new FilterRegistrationBean();
    registrationBean.setFilter(new MyFilter());
    registrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
    return registrationBean;
}
```

#### 3.5.2.3、Listener

```java
@Bean
public ServletListenerRegistrationBean myListener(){
    ServletListenerRegistrationBean<MyListener> registrationBean = new
        ServletListenerRegistrationBean<>(new MyListener());
    return registrationBean;
}
```

### 3.5.3、指定嵌入式Servlet容器    

#### 3.5.3.1、Tomcat（默认使用）    

```xml
<!-- 引入的web模块默认使用的就是嵌入式的Tomcat作为Servlet容器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring‐boot‐starter‐web</artifactId>
</dependency>
```

#### 3.5.3.2、Jetty    

```xml
<!‐‐ 引入web模块：排除默认使用的Tomcat容器 ‐‐>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring‐boot‐starter‐web</artifactId>
        <exclusions>
            <exclusion>
            <artifactId>spring‐boot‐starter‐tomcat</artifactId>
            <groupId>org.springframework.boot</groupId>
            </exclusion>
        </exclusions>
</dependency>

<!‐‐引入其他的Servlet容器‐‐>
<dependency>
    <artifactId>spring‐boot‐starter‐jetty</artifactId>
    <groupId>org.springframework.boot</groupId>
</dependency>
```

#### 3.5.3.3、Undertow    

```xml
<!‐‐ 引入web模块：排除默认使用的Tomcat容器 ‐‐>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring‐boot‐starter‐web</artifactId>
        <exclusions>
            <exclusion>
            <artifactId>spring‐boot‐starter‐tomcat</artifactId>
            <groupId>org.springframework.boot</groupId>
            </exclusion>
        </exclusions>
</dependency>

<!‐‐引入其他的Servlet容器‐‐>
<dependency>
    <artifactId>spring‐boot‐starter‐undertow</artifactId>
    <groupId>org.springframework.boot</groupId>
</dependency>
```





# 四、SpringBoot与数据访问

## 4.1、SpringBoot默认支持数据源

**SpringBoot 2.0以前：**

- TomcatDataSource（默认数据源）
- HikariDataSource
- BasicDataSource    

**SpringBoot 2.0以后：**

- GenericDataSource    
- Dbcp2DataSource 
- HikariDataSource（默认数据源） 
- TomcatDataSource

## 4.2、SpringBoot配置默认数据源

使用默认数据源时，只需配置数据源信息即可：

```yaml
spring:
    datasource:
        username: root
        password: 123456
        url: jdbc:mysql://192.168.15.22:3306/jdbc
        driver‐class‐name: com.mysql.jdbc.Driver
        schema:
          - classpath:user.sql
```

## 4.3、SpringBoot数据源自动配置原理

### 4.3.1、DataSourceProperties ：

​	数据源属性相关配置类。

### 4.3.2、DataSourceConfiguration：

- 使用 DataSource

```java
@ConditionalOnMissingBean({DataSource.class})
@ConditionalOnProperty(
	name = {"spring.datasource.type"}
)
```

- 使用 Dbcp2

```java
@Configuration
@ConditionalOnClass({BasicDataSource.class})
@ConditionalOnMissingBean({DataSource.class})
@ConditionalOnProperty(
    name = {"spring.datasource.type"},
    havingValue = "org.apache.commons.dbcp2.BasicDataSource",
    matchIfMissing = true
)
  
@ConfigurationProperties( prefix = "spring.datasource.dbcp2")
```

- 使用 Hikari

```java
@Configuration
@ConditionalOnClass({HikariDataSource.class})
@ConditionalOnMissingBean({DataSource.class})
@ConditionalOnProperty(
    name = {"spring.datasource.type"},
    havingValue = "com.zaxxer.hikari.HikariDataSource",
    matchIfMissing = true
)

@ConfigurationProperties( prefix = "spring.datasource.hikari" )
```

- 使用 Tomcat

```java
@Configuration   @ConditionalOnClass({org.apache.tomcat.jdbc.pool.DataSource.class})
@ConditionalOnMissingBean({DataSource.class})
@ConditionalOnProperty(
    name = {"spring.datasource.type"},
    havingValue = "org.apache.tomcat.jdbc.pool.DataSource",
    matchIfMissing = true
)

@ConfigurationProperties(  prefix = "spring.datasource.tomcat" )
```



### 4.3.3、DataSourceAutoConfiguration:

- ```
  DataSourcePoolMetadataProvidersConfiguration
  ```

- ```
  DataSourceInitializationConfiguration
  ```



## 4.4、整合Druid数据源

- 导入依赖

	<!--引入druid数据源-->
	<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
	<dependency>
	    <groupId>com.alibaba</groupId>
	    <artifactId>druid</artifactId>
	    <version>1.1.8</version>
	</dependency>
- 编写配置文件

```yaml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.15.22:3306/jdbc
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

	## 以下属性SpringBoot无法自动绑定，需要自定义DataSource并进行绑定
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
	# 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

- 自定义数据源

```java
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
       return new DruidDataSource();
    }

    //配置Druid的监控
    //1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams = new HashMap<>();
        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        initParams.put("allow","");//默认就是允许所有访问
        initParams.put("deny","192.168.15.21");
        bean.setInitParameters(initParams);
        return bean;
    }

    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");
        bean.setInitParameters(initParams);
        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }
}
```



## 4.5、SpringBoot整合MyBatis

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis‐spring‐boot‐starter</artifactId>
    <version>1.3.1</version>
</dependency>
```

### 4.5.1、MyBatis注解版

- 自定义ConfigurationCustomizer，指定MyBatis配置规则 

```java
@Configuration
public class MyBatisConfig {
    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        return new ConfigurationCustomizer(){
            @Override
            public void customize(Configuration configuration) {
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```

- 指定Mapper扫描路径

```java
// 使用MapperScan批量扫描所有的Mapper接口
@MapperScan(value = "com.demo.springboot.mapper")
@Configuration
public class MyBatisConfig {

}
```

- DepartmentMapper  示例

```java
public interface DepartmentMapper {
    @Select("select * from department where id=#{id}")
    public Department getDeptById(Integer id);
    
    @Delete("delete from department where id=#{id}")
    public int deleteDeptById(Integer id);
    
    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into department(departmentName) values(#{departmentName})")
    public int insertDept(Department department);
    
    @Update("update department set departmentName=#{departmentName} where id=#{id}")
    public int updateDept(Department department);
}
```



### 4.5.2、MyBatis配置文件版

- 配置mybatis配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

- 全局配置文件中指定mybatis配置文件和mybatis的Mapper文件

```yaml
mybatis:
  # 指定全局配置文件位置
  config-location: classpath:mybatis/mybatis-config.xml
  # 指定sql映射文件位置
  mapper-locations: classpath:mybatis/mapper/*.xml
```

- 编写Mapper class文件

```java
// @Mapper或者@MapperScan将接口扫描装配到容器中
@Mapper
public interface EmployeeMapper {

    public Employee getEmpById(Integer id);

    public void insertEmp(Employee employee);
}
```

- 编写Mapper xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.springboot.mapper.EmployeeMapper">
   <!-- public Employee getEmpById(Integer id) -->
    <select id="getEmpById" resultType="com.atguigu.springboot.bean.Employee">
        SELECT * FROM employee WHERE id = #{id}
    </select>

     <!-- public void insertEmp(Employee employee) -->
    <insert id="insertEmp">
        INSERT INTO employee
        	(lastName,email,gender,d_id) 
        VALUES 
        	(#{lastName},#{email},#{gender},#{dId})
    </insert>
</mapper>
```



## 4.6、SpringBoot整合SpringData JPA    

1. 编写配置文件

```yaml
spring:
    jpa:
    hibernate:
        # 更新或者创建数据表结构
        ddl‐auto: update
        # 控制台显示SQL
        show‐sql: true
```

2. 编写实体类

```java
// 标注JPA实体类（和数据表映射的类）
@Entity 
//@Table指定对应数据表（省略则默认为类名首字母小写：user）
@Table(name = "tbl_user")
public class User {
    @Id // 主键
    @GeneratedValue(strategy = GenerationType.IDENTITY) //自增
    private Integer id;
    
    @Column(name = "last_name",length = 50) // 数据表列
    private String lastName;
    @Column 	// 省略默认列名为属性名
    private String email;
}
```

3. 编写实体类对应的Dao接口

```java
// 继承JpaRepository
public interface UserRepository extends JpaRepository<User,Integer> { }
```







# 五、自定义starter





- application.properties

```yaml
## 开启springBoot的debug模式，控制台会打印自动配置报告
debug=true
```





- 配置文件处理器    

```xml
<!‐‐导入配置文件处理器，配置文件进行绑定就会有提示‐‐>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring‐boot‐configuration‐processor</artifactId>
    <optional>true</optional>
</dependency>
```



# 六、Spring Boot实战

## 6.1、Spring Boot使用jsp

1. 导入SpringBoot的tomcat启动插件jar包

   ```xml
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-tomcat</artifactId>
   </dependency>
   <dependency>
     <groupId>org.apache.tomcat.embed</groupId>
     <artifactId>tomcat-embed-jasper</artifactId>
   </dependency>
   ```

2. 加入jsp页面等静态资源

   （1）将pom.xml中项目的打包方式改为war：`<packaging>war</packaging>`

   （2）在src/main目录下创建webapp目录

   （3）将WebContent下的静态资源copy到webapp目录下

3. 修改配置文件

   ```yaml
   spring:
   	mvc:
   		view:
   			prefix: /pages/
   			suffix: .jsp
   ```

4. 使用tomcat插件启动项目。