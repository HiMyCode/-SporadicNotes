[toc]



参考文献：

https://blog.csdn.net/acmman/category_10080122.html

# 一、功能使用

## 1.1、读取配置文件

### 1.1.1、注入Environment

```java
@RestController
public class HelloController {

    @Autowired
    private Environment environment;

    @GetMapping("/hello")
    public String hello(){
        String port = environment.getProperty("server.port");
        return port;
    }
}
```

### 1.1.2、使用@Value注解

```java
@RestController
public class HelloController {

    @Value("${server.port}")
    private String port;

    @GetMapping("/hello")
    public String hello(){
        return port;
    }
}
```

### 1.1.3、使用@ConfigurationProperties注解

```java
// 须引入依赖：spring-boot-configuration-processor
@ConfigurationProperties(prefix = "com.example")
@Configuration
public class MyConfig {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

```java
@RestController
public class HelloController {

    @Autowired
    private MyConfig myConfig;

    @GetMapping("/hello")
    public String hello(){
        return myConfig.getName();
    }
}
```

## 1.2、profiles多环境配置

1. 定义多环境配置文件

   - application.properties				：通用配置，不区分环境
   - application-dev.properties        ：开发环境
   - application-test.properties        ：测试环境
   - application-prod.properties      ：生产环境

2. 激活指定环境配置

   - 在application.properties配置文件中添加参数

     spring.profiles.active=dev

   - 在启动命令后添加参数

     java -jar xxx.jar --spring.profiles.active=dev

## 1.3、热部署

1. 添加spring-boot-devtools依赖

```xml
 <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-devtools</artifactId>
   <scope>runtime</scope>
   <optional>true</optional>
</dependency>
```

2. 开启自动编译功能（Idea版本）

   perferences -> Complier -> 勾选 Build progect automatically

3. 快捷键打开Registry页面，勾选：complier automake allow when app running

   mac快捷键：Shift + option + command + /

   windows快捷键：Shift + Ctrl + Alt + /

## 1.4、actuator监控

### 1.4.1、使用actuator默认功能

1. 添加spring-boot-starter-actuator依赖

```xml
 <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2. 配置文件新增配置（springboot 2.4.4）

```properties
management.endpoints.web.exposure.include=*
```

3. 访问地址：http://localhost:8080/actuator/health

**其他Actuator默认启用的端点信息：**


| **ID**                   | **描述**                                                 |
| ------------------------ | -------------------------------------------------------- |
| /actuator/beans          | 显示应用程序中所有Spring bean的完整列表                  |
| /actuator/conditions     | 显示在配置和自动配置类上评估的条件以及它们是否匹配的原因 |
| /actuator/env            | 从Spring的`ConfigurableEnvironment`中公开属性            |
| /actuator/configprops    | 显示所有`@ConfigurationProperties`对照的列表             |
| /actuator/mappings       | 显示所有`@RequestMapping`路径对照的列表                  |
| /actuator/shutdown       | 关闭应用程序                                             |
| /actuator/scheduledtasks | 显示定时任务                                             |

全部端点信息说明请参照官网：

https://docs.spring.io/spring-boot/docs/current/actuator-api/htmlsingle/#overview

### 1.4.2、自定义actuator监控

#### 1.4.2.1、扩展health端点

```java
// 方式一：继承AbstractHealthIndicator
@Component
public class UserHealthIndicator1 extends AbstractHealthIndicator {
    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        builder.up().withDetail("userHealth", "yes");
    }
}
```

```java
// 方式二：实现HealthIndicator
@Component
public class UserHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        int result = checkHealth();
        if(result < 3){
            // 通过up()方法为应用添加健康状态参数
            return Health.up().withDetail("healthCount", result).withDetail("app-actuator", "up")
                    .withDetail("message", "服务正常").build();
        }

        // 通过dwon()方法为应用添加不健康状态参数
        return Health.down().withDetail("healthCount", result).withDetail("app-actuator", "down")
                .withDetail("message", "服务故障").build();
    }

    private int checkHealth(){
        Random rand = new Random();
        return rand.nextInt(10);
    }
}
```

#### 1.4.2.2、自定义actuator端点

```java
// 访问地址：http://localhost:8088/actuator/userEndpoint
@Component
@Endpoint(id = "userEndpoint")
public class UserEndpoint {

    @ReadOperation
    public List<Map<String, Object>> health(){
        List<Map<String, Object>> list = new ArrayList<>();
        Map<String, Object> map = new HashMap<>();
        map.put("userId", "001");
        map.put("userName", "kitty");
        list.add(map);
        return list;
    }
}
```

## 1.5、统一异常处理器（TODO）

1. 定义异常实体

```java
public class ResponseData {

    private Boolean status = true;
    private int code = 200;
    private String message;
    private Object data;

    // getter & setter
}
```

2. 定义一异常处理类

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    private Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public ResponseData defaultErrorHandler(HttpServletRequest req, Exception e) throws Exception {
        logger.error("", e);
        ResponseData r = new ResponseData();
        r.setMessage(e.getMessage());
        if (e instanceof org.springframework.web.servlet.NoHandlerFoundException) {
            r.setCode(404);
        } else {
            r.setCode(500);
        }
        r.setData(null);
        r.setStatus(false);
        return r;
    }
}
```

3. 添加spring boot配置

```properties
## 出现错误时, 直接抛出异常
spring.mvc.throw-exception-if-no-handler-found=true
## 不要为我们工程中的资源文件建立映射（废弃）
spring.resources.add-mappings=false
```

## 1.6、随机端口

1. 方式一

```properties
server.port=${random.int[2000,8000]}
```

缺点：不能保证生成的端口不重复，导致启动失败。

2. 方式二

```java
// 随机生成端口工具类
public class StartCommand {

    private Logger logger = LoggerFactory.getLogger(StartCommand.class);

    public StartCommand(String[] args){
        Boolean isServerPort = false;
        String serverPort = "";
        // 指定端口
        if (args != null){
            for (String arg : args){
                if (StringUtils.hasText(arg) && arg.startsWith("--server.port")){
                    isServerPort = true;
                    serverPort = arg;
                    break;
                }
            }
        }
        // 未指定端口
        if (!isServerPort) {
            int port = ServerPortUtils.getAvailablePort();
            logger.info("current server.port=" + port);
            System.setProperty("server.port", String.valueOf(port));
        } else {
            logger.info("current server.port=" + serverPort.split("=")[1]);
            System.setProperty("server.port", serverPort.split("=")[1]);
        }
    }
}

// 工具类中使用到的类
public class ServerPortUtils {

    public static int getAvailablePort(){
        int max = 65535;
        int min = 2000;
        Random random = new Random();
        int port = random.nextInt(max) % (max - min + 1) + min;
        boolean using = NetUtils.isLoclePortUsing(port);
        if (using){
            return getAvailablePort();
        }
        return port;
    }
}


public class NetUtils {

    public static boolean isLoclePortUsing(int port){
        boolean flag = true;
        try {
            flag = isPortUsing("127.0.0.1", port);
        } catch (Exception e) {
        }
        return flag;
    }

    public static boolean isPortUsing(String host,int port) throws UnknownHostException,IOException {
        boolean flag = false;
        InetAddress theAddress = InetAddress.getByName(host);
        Socket socket = new Socket(theAddress,port);
        flag = true;
        return flag;
    }
}
```

```java
// 使用方法
@SpringBootApplication
public class Springbootpplication {
    public static void main(String[] args) {
          new StartCommand(args);
          SpringApplication.run(SpringbootApplication.class, args);
    }
}
```

## 1.7、编译打包

1. 添加依赖

```xml
<build>
      <plugins>
        	<!-- 编译插件 -->
          <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-compiler-plugin</artifactId>
                  <configuration>
                      <source>1.8</source>
                      <target>1.8</target>
                  </configuration>
           </plugin>
        	 <!-- 打包插件 -->
           <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                  	<executable>true</executable>
                    <mainClass>com.example.springboot.SpringbootApplication</mainClass>
                </configuration>
            </plugin> 
      </plugins>
</build>
```

2. 启动命令

```shell
java -jar xxx.jar
```

## 1.8、自定义starter

### 1.8.1、项目启动时默认加载的starter

- **定义starter**

1. 引入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.6.RELEASE</version>
        <relativePath/> 
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo-spring-boot-starter</name>
  
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
				<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

2. 创建属性配置类，用于读取配置文件中的配置的属性值

```java
@ConfigurationProperties("spring.user")
public class UserProperties {
    private String name;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

3. 创建Client客户端，用于获取配置文件中的值

```java
public class UserClient {
    private UserProperties userProperties;
    public UserClient() { }
    public UserClient(UserProperties p) {
        this.userProperties = p;
    }
    public String getName() {
        return userProperties.getName();
    }
}
```

4. UserClient自动配置类

```java
@Configuration
@EnableConfigurationProperties(UserProperties.class)
public class UserAutoConfigure {
    @Bean
    @ConditionalOnProperty(prefix = "spring.user", value = "enabled", havingValue = "true")
    public UserClient userClient(UserProperties userProperties) {
        return new UserClient(userProperties);
    }
}
```

5. Springboot启动时，会扫描当前项目及其所有依赖包的： resources/META-INF/spring.factories，并读取文件中的org.springframework.boot.autoconfigure.EnableAutoConfiguration参数，加载器配置的类。

   基于此原理，若需要UserAutoConfigure在Springboot启动时被加载，则在当前项目的resources/META-INF/下新增文件spring.factories（**文件类型一定要选spring.factories**）：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.demospringbootstarter.demo.UserAutoConfigure
```

- **使用starter**

1. 在依赖的项目com.example.springboot中引用此starter

```xml
 <dependency>
   <groupId>com.example</groupId>
   <artifactId>demo-spring-boot-starter</artifactId>
   <version>0.0.1-SNAPSHOT</version>
</dependency>
```

2. 添加配置参数

```properties
## @ConditionalOnProperty(prefix = "spring.user", value = "enabled", havingValue = "true")
spring.user.enabled=true
spring.user.name=Jack
```

3. 启动项目后访问地址

http://localhost:8080/user/name

### 1.8.2、使用注解开启starter的加载

1.8.1示例中，SpringBoot启动时，将默认读取配置文件spring.factories，并加载UserAutoConfigure。无法配置。可通过注解方式，配置是否加载UserAutoConfigure。

1. 删除demo-spring-boot-starter项目中的spring.factories配置文件
2. 在依赖项目com.example.springboot中新建注解EnableUserClient

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({UserAutoConfigure.class})
public @interface EnableUserClient {
}
```

3. 开启UserAutoConfigure自动配置

```java
// 如果不需要自动配置UserAutoConfigure，去掉@EnableUserClient注解即可
@EnableUserClient
@SpringBootApplication
public class SpringbootApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootApplication.class, args);
    }
}
```

**注意：**

​	添加@EnableUserClient注解后，UserAutoConfigure配置类将被加载，但UserClient是否被添加到容器取决于当前环境配置是否满足@ConditionalOnProperty注解上的条件。

### 1.8.3、配置starter内容提示

	1. 在starter项目下创建目录/文件：resources/META-INF/spring-configuration-metadata.json
	2. 文件内容

```json
{
  "properties": [
      {
        "name": "spring.user.name",
        "type": "java.lang.String",
        "defaultValue": "Tom",
        "description": "默认用户名"
      },
      {
        "name": "spring.user.enabled",
        "type": "java.lang.Boolean",
        "defaultValue": "false",
        "description": "是否启用配置"
      },
}
```

# 二、组件实战

## 2.1、Eureka组件

### 2.1.1、Eureka的使用

#### 2.1.1.1、Eureka REST API

​	获取某个服务的注册信息：

​	http://localhost:8761/eureka/apps/eureka-client-user-service

​	其中**eureka-client-user-service**为服务名。

#### 2.1.1.3、Eureka元数据

- **标准元数据**

  标准元数据包括：主机名、IP、端口号、状态页、健康信息等。

- **自定义元数据**

  使用eureka.instance.metadata-map定义，例：

  ```properties
  ## eureka.instance.metadata-map.[key]=[value]
  eureka.instance.metadata-map.serviceType=provider
  ```

#### 2.1.1.3、Eureka客户端

​	项目集成Eureka后，可通过Eureka Client获取相关数据，如：

```java
@Qualifier("eurekaClient")
@Autowired
private EurekaClient eurekaClient;

@GetMapping("/article/getEurekaInfos")
public Object getEurekaInfos(){
  // eureka-client-user-service 为服务名
  return eurekaClient.getInstancesByVipAddress("eureka-client-user-service", false);
}
```

除了使用EurekaClient，还可以使用spring cloud封装的DiscoveryClient，如：

```java
@Autowired
private DiscoveryClient discoveryClient;

@GetMapping("/article/getEurekaInfos")
public Object getEurekaInfos(){
  return discoveryClient.getInstances("eureka-client-user-service");
}
```

#### 2.1.1.5、Eureka服务上下线监控

Eureka提供了事件监听方式来对服务进行上下线的监控，支持事件包括：

- EurekaInstanceCanceledEvent 服务下线事件
- EurekaInstanceRegisteredEvent 服务注册事件
- EurekaInstanceRenewedEvent 服务续约事件
- EurekaRegistryAvailableEvent Eureka注册中心启动事件
- EurekaServerStartedEvent Eureka Server启动事件

使用示例：

```java
import com.netflix.appinfo.InstanceInfo;
import org.springframework.cloud.netflix.eureka.server.event.*;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class EurekaStateChangeListener {
 
    @EventListener
    public void listen(EurekaInstanceCanceledEvent event) {
        System.err.println(event.getServerId() + "\t" + event.getAppName() + " 服务下线");
    }
    
    @EventListener
    public void listen(EurekaInstanceRegisteredEvent event) {
        InstanceInfo instanceInfo = event.getInstanceInfo();
        System.err.println(instanceInfo.getAppName() + "进行注册");
    }
    
    @EventListener
    public void listen(EurekaInstanceRenewedEvent event) {
        System.err.println(event.getServerId() + "\t" + event.getAppName() + " 服务进行续约");
    }
    
    @EventListener
    public void listen(EurekaRegistryAvailableEvent event) {
        System.err.println("注册中心 启动");
    }
    
    @EventListener
    public void listen(EurekaServerStartedEvent event) {
        System.err.println("Eureka Server 启动");
    }
```



### 2.1.2、Eureka配置

#### 2.1.2.1、开启Eureka认证

1. 添加依赖

   ```xml
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   ```

2. 添加配置

   ```properties
   spring.security.user.name=cookie
   spring.security.user.password=123456
   ```

   开启认证后，在Eureka集群中配置文件的**eureka.client.serviceUrl.defaultZone**选项须带上用户名/密码：

   ```properties
   eureka.client.serviceUrl.defaultZone=http://cookie:123456@localhost:8762/eureka/
   ```

#### 2.1.2.2、配置Eureka实例Instance ID

​	默认Instance ID格式：

```properties
## eureka.instance.instance-id=主机名：服务名称：服务端口
eureka.instance.instance-id=${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}
```

​	默认Instance ID中无服务器IP，不利于问题排查，可更改为如下格式：

```properties
##eureka.instance.instance-id=服务名称：服务IP：服务端口
eureka.instance.instance-id=${spring.application.name}:${spring.cloud.client.ip-address}:${server.port}
```

#### 2.1.2.3、配置实例跳转连接

默认跳转地址：http://ip:port/actuator/info

自定义跳转地址：

```properties
eureka.instance.status-page-url=http://${spring.cloud.client.ip-address}:${server.port}/actuator/info
```



### 2.1.3、Eureka原理



#### 自我保护机制

进入Eureka首页后，会在Eureka页面发现如下红色字样：

```
EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.
```

**原理解释：**

- 默认情况下，如果Eureka Server在一定时间内（默认90秒）没有接收到某个微服务实例的心跳，Eureka Server将会移除该实例。但是当网络分区故障发生时，微服务与Eureka Server之间无法正常通信，而微服务本身是正常运行的，此时按道理不应该移除这个微服务，所以引入了自我保护机制。
- Eureka Server在运行期间会去统计心跳失败比例在 15 分钟之内是否低于 85%，如果低于 85%，Eureka Server 会将这些实例保护起来，让这些实例不会过期。
- 自我保护模式是一种针对网络异常波动的安全保护措施，增加了Eureka集群的健壮性、稳定性。

**相关配置：**

Eureka server 配置：

```properties
## 关闭自我保护（切记：生产不要关闭！）
# eureka.server.enable-self-preservation=false
## 超时后移除实例
# eureka.server.eviction-interval-timer-in-ms=2000
```

Eureka Client 配置：

```properties
## 开启健康检查，需要在pom.xml中引入 actuator 依赖
eureka.client.healthcheck.enabled=true
## 配置Eureka Server接受上一次心跳后，等待下一次心跳的最大时间，若超时则移除实例，默认：90s
eureka.instance.lease-expiration-duration-in-seconds=5
## 配置Eureka Client向Eureka Server发送心跳的时间间隔，默认：30s
eureka.instance.lease-renewal-interval-in-seconds=5 
```

## 2.2、Ribbon组件

### 2.2.1、简介

Ribbon是Netflix开源的一款用于客户端负载均衡的工具软件。

目前主流的负载方案分为两种：

- 服务端负载均衡（集中式负载均衡）：在消费者和服务提供方中间使用独立的代理方式进行负载。
  - 硬件负载：如F5；
  - 软件负载，如Nginx

- 客户端负载均衡：根据自身请求情况做负载，如Ribbon。

### 2.2.2、基本使用

#### 2.2.2.1、集成RestTemplate实现负载均衡

1. 创建springBoot项目，添加依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
   </dependency>
   ```

2. 添加注解

   ```java
   @Configuration
   public class RestTemplateConfig {
       
       @Bean
       @LoadBalanced
       public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
           return new RestTemplate(factory);
       }
    
       @Bean
       public ClientHttpRequestFactory simpleClientHttpRequestFactory() {
           SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
           //读取反馈超时时间5000ms
         	factory.setReadTimeout(5000);
         	//连接超时时间15000ms
           factory.setConnectTimeout(15000);
           return factory;
       }
   }
   ```

3. 使用RestTemplate进行API调用

   ```java
   @RestController
   public class HelloController {
   
       @Autowired
       private RestTemplate restTemplate;
   
       // 添加@LoadBalanced注解后，请求地址必须使用服务名。
       @GetMapping("/ribbon/callHello")
       public String callHello(){
           return restTemplate.getForObject("http://user-provider-service/user/hello", String.class);
       }
   }
   ```

#### 2.2.2.2、Ribbon API的使用

通过Ribbon API获取对应服务的数据信息：

```java
@Autowired
private LoadBalancerClient loadBalancerClient;

@GetMapping("/article/getLoadBalancerInfos")
public Object getLoadBalancerInfos(){
  return loadBalancerClient.choose("eureka-client-user-service");
}
```

### 2.2.3、相关配置

#### 2.2.3.1、默认配置

Spring Cloud Ribbon默认配置类：**RibbonClientConfiguration**。包括但不限于如下配置：

- ILoadBalancer：负载均衡器
- RestClient：Rest服务的客户端
- ServerListFilter：服务列表过滤机制

#### 2.2.3.2、通过文件配置

1. **指定全局负载策略**

   ```java
   
   ```

   

2. **指定单个服务的负载策略**

   ```
   
   ```

#### 2.2.3.3、通过代码配置

1. **指定全局负载策略**

   如需指定全局的负载策略，只需添加一个配置类，确保该配置类可被Application Context组件扫描到，则该配置类中的配置将被所有的Ribbon客户端共享。

   ```java
   @Configuration
   public class RibbonConfiguration {
       @Bean
       public IRule ribbonRule() {
           return new RandomRule();
       }
   }
   ```

2. **指定单个服务的负载策略**

   ```java
   @Configuration
   protected static class UserRibbonConfiguration {
       @Bean
       public ZonePreferenceServerListFilter serverListFilter() {
           ZonePreferenceServerListFilter filter = new ZonePreferenceServerListFilter();
           filter.setZone("userZone");
           return filter;
       }
       @Bean
       public IPing ribbonPing() {
           return new PingUrl();
       }
   }
   ```

   ```java
   @Configuration
   @RibbonClient(name="user-service",configuration= UserRibbonConfiguration.class)
   public class TestConfiguration{
    
   }
   ```

   **注意：**

   需要确保UserRibbonConfiguration配置类不被Application Context组件扫描到，否则该配置类中的配置将被所有的Ribbon客户端共享。

   **参考方案：**

3. 

### 2.2.4、原理相关

#### 2.2.4.1、负载策略

1. **负责策略分类**

- RandomRule：随机策略

- RoundRobinRule：轮询策略

- RetryRule：重试策略。

  默认使用轮询策略，在配置时间内，若选择server失败，则一直尝试直到选择一个可用的server。

- BestAvailableRule：最低并发策略

  逐个考察server，如果server断路器打开，则跳过，再选择ActiveRequestCount最低的server。

- AvailabilityFilteringRule：可用过滤策略

  过滤掉一直失败并被标记为circuit tripped的server，过滤掉那些高并发链接的server（active connections超过配置的阈值）

- ResponseTimeWeightedRule：响应时间加权重策略，后改名为：WeightedResponsTimeRule

  综合网络，磁盘，io等因素，根据server的响应时间分配权重：

  ​		响应时间越长，权重越低，被选择到的概率就低；

  ​		响应时间越短，权重越高，被选中的概率就高。

- ZoneAvoidanceRule：区域权重策略

  判断server所在区域的性能和可用性，轮询选择server并且判断一个AWS Zone的运行性能是否可用，剔除不可用的Zone中的所有server。

2. **指定全局负载策略**

   如需指定全局的负载策略，只需添加一个配置类。

   ```java
   @Configuration
   public class RibbonConfiguration {
       @Bean
       public IRule ribbonRule() {
           return new RandomRule();
       }
   }
   ```

3. **指定单个服务的负载策略**

   ```java
   
   ```

   

4. 









1. 第一步：选择Eureka Server。优先选择同个Zone且负载较少的Server；

2. 第二步：根据指定策略，从Server对应的服务注册列表中选择一个地址。

### 2.2.5、工具对比



## 2.3、Feign组件

### 2.3.1、简介

​	Feign是Netflix开发的声明式、模板化的HTTP客户端，使REST调用更加方便简洁。

​	Spring Cloud 对Feign进行了增强，使其支持SpringMVC注解，并整合了Ribbon和Eureka以支持负载。

### 2.3.2、基本使用

1. 创建springBoot项目，添加依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. 开启Feign client功能

   ```java
   @SpringBootApplication
   @EnableFeignClients
   public class DemoApplication {
       public static void main(String[] args) {
           SpringApplication.run(DemoApplication.class, args);
       }
   }
   ```

   上述方式默认扫描启动类DemoApplication所在包下@FeignClient标注的类，如需要，可指定扫描基础包：

   ```java
   @EnableFeignClients(basePackages= "com.demo.feign.client")
   ```

3. 创建Feign接口

   ```java
   // user-provider-service 为服务名
   @FeignClient(name="user-provider-service")
   public interface UserFeignClient {
       @GetMapping("/user/{id}")
       User getById(@PathVariable("id") Integer id);
   }
   ```

   默认集成了Ribbon和Eureka。Ribbon负责将服务名解析为Eureka Server服务注册表中的服务。

4. 调用Feign接口

   ```java
   @RestController
   public class UserController {
     
       @Autowired
       private UserFeignClient userFeignClient;
   
       @GetMapping("/user/{id}")
       public User findById(@PathVariable Integer id) {
           return userFeignClient.getById(id);
       }
   }
   ```

### 2.3.3、相关配置

​	Spring Cloud中Feign的默认配置类是FeignClientsConfiguration。可用于配置Feign的编码器、解码器、日志打印、添加拦截器等。

​	Spring Cloud允许通过@FeignClient注解的Configuration属性自定义Feign的配置。自定义配置的优先级高于FeignClientsConfiguration。

#### 2.3.3.1、全局配置

- **Java Config**

```java
@Configuration
public class DefaultFeignConfig{
  
}
```

```java
@SpringBootApplication
@EnableFeignClients(defaultConfiguration = DefaultFeignConfig.class)
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

- **Yaml Config：**

```yaml
## 配置全局参数
feign:
  client:
   config:
     default:
       connectTimeout: 3000
       readTimeout: 3000
       loggerLevel: full
       encoder: feign.form.spring.SpringFormEncoder
       requestInterceptors:
         - com.demo.feign.common.interceptor.RequestHeaderInterceptor
```

#### 2.3.3.2、自定义配置

- **Java Config：**

```java

```

- **Yaml Config：**

```yaml
## 配置 user-provider-service 服务参数
feign:
  client:
    config:
      user-provider:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: full
        encoder: feign.form.spring.SpringFormEncoder
        requestInterceptors:
          - com.demo.feign.common.interceptor.RequestHeaderInterceptor
```

#### 2.3.3.1、Feign日志

​	Feign允许为每个Feign客户端指定不同的日志记录策略。

1. 定义Feign日志配置类

   ```java
   @Configuration
   public class FeignLogConfiguration {
     @Bean
     Logger.Level feignLoggerLevel() {
       return Logger.Level.BASIC;
     }
   }
   ```

   - NONE： 不记录任何日志（默认值）
   - BASIC： 仅记录请求方法、URL、响应状态代码以及执行时间。
   - HEADERS:  记录basic级别的基础上，记录请求和响应的header。
   - FULL： 记录请求和响应的header、body和元数据。

2. 添加配置参数

   ```properties
   ## 指定某一客户端的日志级别
   ## logging.level.[全类名]:[LOG_LEVEL]
   logging.level.com.demo.feign.client.UserClientFeign: DEBUG
   
   ## 指定某一服务的日志级别
   ## feign.client.config.[服务名].loggerLevel:[LOG_LEVEL]
   # feign.client.config.user-provider-service.loggerLevel: HEADERS
   ```

3. 使用Feign接口时，添加指定配置类

   ```java
   @FeignClient(name = “user-provider-service", configuration = FeignLogConfiguration.class)
   public interface UserFeignClient {
     @RequestMapping(value = "/{id}", method = RequestMethod.GET)
     public User findById(@PathVariable("id") Long id);
   }
   ```

**知识点：**

- 每个Feign客户端都会创建一个logger。logger名称默认为Feign接口全类名；
- Feign日志打印只会对DEBUG级别做出响应。

#### 2.3.3.、配置文件示例

```properties
## 配置请求GZIP压缩,默认：false.
feign.compression.request.enabled=true
## 配置响应GZIP压缩,默认：false.
feign.compression.response.enabled=true
## 配置压缩支持的MIME TYPE
feign.compression.request.mime-types=text/xml,application/xml,application/json
## 配置压缩数据大小的下限
feign.compression.request.min-request-size=2048
```






### 2.3.4、工具对比

#### 其他HttpClient工具：

- Httpclient
- Okhttp
- HttpURLConnection
- RestTemplate

# 三、项目实战

## 3.1、Eureka使用

### 3.1.1、单实例Eureka的使用

### 3.1.2、搭建Eureka集群

1. 新建项目&引入依赖

   ```xml
   <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.4.3</version>
           <relativePath/> <!-- lookup parent from repository -->
       </parent>
      
       <properties>
           <java.version>1.8</java.version>
           <spring-cloud.version>2020.0.2</spring-cloud.version>
       </properties>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-dependencies</artifactId>
               <version>Hoxton.SR6</version>
               <scope>import</scope>
               <type>pom</type>
           </dependency>    
       </dependencies>
   
       <dependencyManagement>
           <dependencies>
               <dependency>
                   <groupId>org.springframework.cloud</groupId>
                   <artifactId>spring-cloud-dependencies</artifactId>
                   <version>${spring-cloud.version}</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
           </dependencies>
       </dependencyManagement>
   ```

2. 添加注解@EnableEurekaServer

   ```java
   @EnableEurekaServer
   @SpringBootApplication
   public class EurekaServerApplication {
       public static void main(String[] args) {
           SpringApplication.run(EurekaServerApplication.class, args);
       }
   }
   ```

3. 添加配置文件

   ```properties
   spring.application.name=eureka-server
   
   ## 是否将自己注册到Eureka Server
   eureka.client.register-with-eureka=false
   ## 是否从Eureka Server检索服务（服务提供者提供的服务）
   eureka.client.fetch-registry=false
   ```

#### 3.1.2.1、新建项目&引入依赖

```xml
 		<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.3</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
   
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>2020.0.2</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR6</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>    
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

#### 3.1.2.2、添加注解@EnableEurekaServer

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

#### 3.1.2.3、添加配置文件

- 主配置文件：application.properties

```properties
spring.application.name=eureka-server-cluster

## 是否将自己注册到Eureka Server
eureka.client.register-with-eureka=true
## 是否从Eureka Server检索服务（服务提供者提供的服务）
eureka.client.fetch-registry=true

spring.profiles.active=@spring.profiles.active@
```

- Eureka Server主节点配置：application-master.properties

```properties
server.port=8761
eureka.client.serviceUrl.defaultZone=http://localhost:8762/eureka/,http://localhost:8763/eureka/
```

- Eureka Server从节点1配置：application-slaver.properties

```properties
server.port=8762
eureka.client.servceUrl.defaultZone=http://localhost:8761/eureka/,http://localhost:8763/eureka/
```

- Eureka Server从节点2配置：application-slaver.properties

```properties
server.port=8763
eureka.client.servceUrl.defaultZone=http://localhost:8761/eureka/,http://localhost:8762/eureka/
```

#### 3.1.2.4、项目启动

1. 方式一：使用idea启动多实例

   Edit Configurations -> Add New Configuration(选择Spring Boot) ->填写Name等信息并应用（如下图）；

   依次创建EurekaMaster、EurekaSlaver1、EurekaSlaver2服务，并指定正确的Active profiles；

   最后启动EurekaMaster、EurekaSlaver1、EurekaSlaver2服务。

   ![Eureka服务配置](../图片/Eureka服务配置.png)

2. 方式二：使用jar包方式

   将项目package后，通过jar命令方式启动：

   ```shell
   java -jar eureka-server-0.0.1-SNAPSHOT.jar --spring.profiles.active=master
   java -jar eureka-server-0.0.1-SNAPSHOT.jar --spring.profiles.active=slaver1
   java -jar eureka-server-0.0.1-SNAPSHOT.jar --spring.profiles.active=slaver2
   ```

## 3.2、Ribbon的使用





# 四、功能扩展



# 五、Q&A

## 3.1、项目构建相关

### 3.1.1、maven导入自定义jar包后，仍无法使用jar包中的类

自定义jar构建配置：

```xml
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

引入jar包后，jar包结构异常：

![maven导入jar异常图](../图片/maven导入jar异常图.png)

**解决方法：**

添加skip参数

```xml
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
  <configuration>
    <skip>true</skip>
  </configuration>
</plugin>
```

引入jar包后的正确结构：

![maven导入jar后正确结构图](../图片/maven导入jar后正确结构图.png)

## 3.1、项目启动相关

### 3.1.1、resources/META-INF/spring.factories 文件未被加载

- 问题描述：

  按照约定路径及文件名创建了resources/META-INF/spring.factories 文件；

  按照约定的key-value格式，配合了需要被自动加载类；

  但项目在启动时，spring.factories文件并未被加载读取。

- 错误原因：

  创建spring.factories 文件时，类型选择错误。一定要选择**spring.factories** ，如图：

  ![spring.factories文件类型](../图片/spring.factories文件类型.png)

  错误图例：

  ![spring.properties异常类型](../图片/spring.properties异常类型.png)