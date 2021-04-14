[toc]



# 一、权限管理简介

​		权限管理，一般指根据系统设置的安全规则或者安全策略，用户可以访问且只能访问自己被授权的资源。

**权限管理中的重要概念：**

​	**认证：**通过用户名和密码成功登陆系统后，让系统得到当前用户的角色身份。

​	**授权：**系统根据当前用户的角色，给其授予对应可以操作的权限资源。

**权限管理中的重要对象：**

​	**用户：**主要包含用户名，密码和当前用户的角色信息，用于实现认证操作。

​	**角色：**主要包含角色名称，角色描述和当前角色拥有的权限信息，用于实现授权操作。

​	**权限：**权限也可以称为菜单，主要包含当前权限名称，url地址等信息，可实现动态展示菜单。

# 二、Spring Security 简介

​		Spring Security 是一款采用Spring框架的AOP思想、基于servlet过滤器实现的、为Java应用程序提供高度可定制的身份认证和授权的安全框架。

**Spring Security特性：**

- 全面和可扩展的身份验证和授权支持
- 防止会话固定、点击劫持、跨网站请求伪造等攻击
- Servlet API集成
- 与Spring Web MVC的可选集成

# 三、Spring Security实战

## 3.1、默认Spring Security认证

1. 导入jar包

   - spring-security-core.jar

     Spring Security核心包，所有功能的基础。

   - spring-security-web.jar

     包含过滤器和Web相关的安全基础功能。

   - spring-security-config.jar

     用于解析Spring Security的xml配置文件。

   - spring-security-taglibs.jar

     为jsp页面提供动态标签库。

   由于：

   ​		spring-security-config.jar默认依赖spring-security-core.jar，

   ​		spring-security-taglibs.jar默认依赖spring-security-web.jar，

   所以，

   ​		若使用时需要导入全部jar，则只导入spring-security-config.jar和spring-security-taglibs.jar即可。

2. 配置web.xml

   ```xml
   <!--Spring Security过滤器链，过滤器名称必须为：springSecurityFilterChain-->
   <filter>
     <filter-name>springSecurityFilterChain</filter-name>
     <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
   </filter>
   <filter-mapping>
     <filter-name>springSecurityFilterChain</filter-name>
     <url-pattern>/*</url-pattern>
   </filter-mapping>
   ```

3. 配置spring-security.xml

   ```xml
   <!--设置可以用spring的el表达式配置Spring Security并自动生成对应配置组件（过滤器）-->
   <security:http auto-config="true" use-expressions="true">
   	<!--使用spring的el表达式来指定项目所有资源访问都必须有ROLE_USER或ROLE_ADMIN角色-->
   	<security:intercept-url pattern="/**" access="hasAnyRole('ROLE_USER','ROLE_ADMIN')"/>
   </security:http>
   
   <!--设置Spring Security认证用户信息的来源-->
   <security:authentication-manager>
     <security:authentication-provider>
       <security:user-service>
         <security:user name="user" password="{noop}user" authorities="ROLE_USER" />
         <security:user name="admin" password="{noop}admin" authorities="ROLE_ADMIN" />
       </security:user-service>
     </security:authentication-provider>
   </security:authentication-manager>
   ```

   将spring-security.xml文件引入到applicationContext.xml中：

   ```xml
   <!--引入SpringSecurity主配置文件-->
   <import resource="classpath:spring-security.xml"/>
   ```

4. 启动项目并访问，则跳转至Spring Security默认登录认证首页。

**以上配置存在以下问题：**

- 登录首页为Spring Security默认的首页；
- 认证用户名及密码为配置文件指定，并未访问数据库。

## 3.2、定制Spring Security认证

### 3.2.1、定制登录首页

1. 修改配置文件

   ```xml
   <!--直接释放无需经过SpringSecurity过滤器的静态资源-->
   <security:http pattern="/css/**" security="none"/>
   <security:http pattern="/img/**" security="none"/>
   <security:http pattern="/plugins/**" security="none"/>
   <security:http pattern="/failer.jsp" security="none"/>
   <security:http pattern="/favicon.ico" security="none"/>
   
   <!--设置可以用spring的el表达式配置Spring Security并自动生成对应配置组件（过滤器）-->
   <security:http auto-config="true" use-expressions="true">
     <!--指定login.jsp页面可以被匿名访问-->
     <security:intercept-url pattern="/login.jsp" access="permitAll()"/>
     <!--使用spring的el表达式来指定项目所有资源访问都必须有ROLE_USER或ROLE_ADMIN角色-->
     <security:intercept-url pattern="/**" access="hasAnyRole('ROLE_USER','ROLE_ADMIN')"/>
     <!--指定自定义的认证页面-->
     <security:form-login login-page="/login.jsp" login-processing-url="/login" 
               default-target-url="/index.jsp" authentication-failure-url="/failer.jsp"/>
     <!--指定退出登录后跳转的页面-->
     <security:logout logout-url="/logout" logout-success-url="/login.jsp"/>
   </security:http>
   
   <!--设置Spring Security认证用户信息的来源-->
   <security:authentication-manager>
     <security:authentication-provider>
       <security:user-service>
         <security:user name="user" password="{noop}user" authorities="ROLE_USER" />
         <security:user name="admin" password="{noop}admin" authorities="ROLE_ADMIN" />
       </security:user-service>
     </security:authentication-provider>
   </security:authentication-manager>
   ```

**注意：**

​	SpringSecurity 默认开启了csrf防护机制。

​	在SpringSecurity中CsrfFilter过滤器中，默认把请求方式分成两类：

 - GET、HEAD、TRACE、OPTIONS

   CsrfFilter过滤器不作拦截，直接放行。

 - 除了GET、HEAD、TRACE、OPTIONS的请求，如：POST、DELETE、PUT等

   **CsrfFilter过滤器将校验请求若携带token才放行**。

由于SpringSecurity的/login默认为POST请求，所以若要使用其/login逻辑，则提交的表单中须存在`_csrf`属性。可直接使用spring-security-taglibs.jar提供的`<security:csrfInput/>`标签。示例：

```jsp
<form action="${pageContext.request.contextPath}/login" method="post">
				<security:csrfInput/>
				<div class="form-group has-feedback">
					<input type="text" name="username" class="form-control"
						placeholder="用户名"> <span
						class="glyphicon glyphicon-envelope form-control-feedback"></span>
				</div>
				<div class="form-group has-feedback">
					<input type="password" name="password" class="form-control"
						placeholder="密码"> <span
						class="glyphicon glyphicon-lock form-control-feedback"></span>
				</div>
				<div class="row">
					<div class="col-xs-4">
						<button type="submit" class="btn btn-primary btn-block btn-flat">登录</button>
					</div>
				</div>
			</form>
```

​	上文中同时配置了登出逻辑：

```xml
<!--指定退出登录后跳转的页面-->
  <security:logout logout-url="/logout" logout-success-url="/login.jsp"/>
```

​	若使用默认的/logout方法且开启了csrf过滤，则该方法为POST请求。在请求该方法时，form表单也需要添加：`<security:csrfInput/>`。

```jsp
<form action="${pageContext.request.contextPath}/logout" method="post">
   <security:csrfInput/>
   <input type="submit" value="注销">
</form>
```

​	如需禁用csrf功能（生产不建议禁用）：

```xml
<security:csrf disabled="true"/>
```

### 3.2.2、定制数据库认证

2. 重写认证逻辑

   ```java
   import org.springframework.security.core.userdetails.UserDetailsService;
   
   // 继承UserDetailsService类，并重写loadUserByUsername方法（该方法默认读取配置文件中的用户）
   public class UserService extends UserDetailsService {
     @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
               //根据用户名做查询
               SysUser sysUser = userDao.findByName(username);
               if(sysUser==null){
                   return null;
               }
               List<SimpleGrantedAuthority> authorities = new ArrayList<>();
               List<SysRole> roles = sysUser.getRoles();
               for (SysRole role : roles) {
                   authorities.add(new SimpleGrantedAuthority(role.getRoleName()));
               }
               //{noop}后面的密码，springsecurity会认为是原文。
               return new User(sysUser.getUsername(), "{noop}"+ sysUser.getPassword(), authorities);
       }
   }
   ```

2. 修改配置文件，指定认证用户信息的来源

   ```xml
   <!-- 设置Spring Security认证用户信息的来源 -->
   <security:authentication-manager>
       <security:authentication-provider user-service-ref="userService">
       </security:authentication-provider>
   </security:authentication-manager>
   ```

   注意:**

   此处用户认证的密码并未被加密，而是使用的明文。

   **扩展：**UserDetails构造器：

   ```java
   // 此处的User对象为UserDetails的实现类
   UserDetails user = 
     new User(sysUser.getUsername(), "{noop}"+ sysUser.getPassword(), authorities);
   
   /**
    * User对象的另一个构造器，后四个参数必须同时为true才能通过认证
    * username：用户名
    * password：用户密码
    * enabled：是否可用
    * accountNonExpired：账户是否失效
    * credentialsNonExpired：秘密是否失效
    * accountNonLocked：账户是否锁定
    */
   public User(String username, String password, boolean enabled, boolean accountNonExpired,
   			boolean credentialsNonExpired, boolean accountNonLocked,
   			Collection<? extends GrantedAuthority> authorities) {
   }
   ```

### 3.2.3、用户密码加密认证

1. 在IOC容器中提供加密对象

   ```java
   <!--把加密对象放入的IOC容器中-->
   <bean id="passwordEncoder"
      		class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/>
   
   <!--设置Spring Security认证用户信息的来源，并指定加密策略-->
   <security:authentication-manager>
     <security:authentication-provider user-service-ref="userServiceImpl">
       <security:password-encoder ref="passwordEncoder"/>
     </security:authentication-provider>
   </security:authentication-manager>
   ```

2. 保存用户信息时，对密码加密

   ```java
   @Service
   @Transactional
   public class UserServiceImpl implements UserService {
       @Autowired
       private UserDao userDao;
   
       @Autowired
       private RoleService roleService;
   
       @Autowired
       private BCryptPasswordEncoder passwordEncoder;
   
       @Override
       public void save(SysUser user) {
           user.setPassword(passwordEncoder.encode(user.getPassword()));
           userDao.save(user);
       }
   }
   ```

3. 修改认证方法（取消密码前的"{noop}"）

   ```java
   import org.springframework.security.core.userdetails.UserDetailsService;
   
   // 继承UserDetailsService类，并重写loadUserByUsername方法（该方法默认读取配置文件中的用户）
   public class UserService extends UserDetailsService {
     @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
               //根据用户名做查询
               SysUser sysUser = userDao.findByName(username);
               if(sysUser==null){
                   return null;
               }
               List<SimpleGrantedAuthority> authorities = new ArrayList<>();
               List<SysRole> roles = sysUser.getRoles();
               for (SysRole role : roles) {
                   authorities.add(new SimpleGrantedAuthority(role.getRoleName()));
               }
               //{noop}后面的密码，springsecurity会认为是原文。
               return new User(sysUser.getUsername(), "{noop}"+ sysUser.getPassword(), authorities);
       }
   }
   ```

## 3.3、其他认证相关功能

### 3.3.1、Remember Me功能

​	Remember Me功能 关键代码：

```java
public abstract class AbstractRememberMeServices implements RememberMeServices,
InitializingBean, LogoutHandler {
		public final void loginSuccess(HttpServletRequest request, 
                                   HttpServletResponse response,
                                   Authentication successfulAuthentication) {
				// 判断是否勾选记住我。this.parameter为常量：private String parameter ="remember-me"
				if (!this.rememberMeRequested(request, this.parameter)) {
						this.logger.debug("Remember-me login not requested.");
				} else {
						this.onLoginSuccess(request, response, successfulAuthentication);
				}
		}
}

//若勾选则调用onLoginSuccess方法
protected boolean rememberMeRequested(HttpServletRequest request, String parameter) {
		if (this.alwaysRemember) {
  			return true;
    } else {
        String paramValue = request.getParameter(parameter);
        // 若"remember-me"的为：true || on || yes || 1 则返回 true
        if (paramValue != null && (paramValue.equalsIgnoreCase("true") ||
                                   paramValue.equalsIgnoreCase("on")   ||
                                   paramValue.equalsIgnoreCase("yes")   ||
                                   paramValue.equals("1"))) {
        		return true;
        } else {
        		if (this.logger.isDebugEnabled()) {
        				this.logger.debug("Did not send remember-me cookie " +
                                  "(principal did not set parameter '" + parameter + "')");
        		}
        		return false;
        }
    }
}

//若勾选了”Remember Me“选项，则调用如下onLoginSuccess方法
public class PersistentTokenBasedRememberMeServices extends AbstractRememberMeServices {
    protected void onLoginSuccess(HttpServletRequest request, HttpServletResponse response,
                                  Authentication successfulAuthentication) {
        String username = successfulAuthentication.getName();
        PersistentRememberMeToken persistentToken = new PersistentRememberMeToken(username,
        this.generateSeriesData(), this.generateTokenData(), new Date());
        try {
            //将token持久化到数据库
            this.tokenRepository.createNewToken(persistentToken);
            //将token写入到浏览器的Cookie中
            this.addCookie(persistentToken, request, response);
        } catch (Exception var7) {
            this.logger.error("Failed to save persistent token " , var7);
        }
    }
}
```
根据以上原理，若需要Remember Me功能，则配置如下：

1. Jsp页面添加”记住我“功能

   ```jsp
   <div class="checkbox icheck">
     <!-- 注意：name必须为：remember-me ，value必须为：true、on、yes、1 四选一 -->
     <label><input type="checkbox" name="remember-me" value="true"> 记住 下次自动登录
     </label>
   </div>
   ```

2. 开启过滤器RememberMeAuthenticationFilter，默认是不开启。

   ```xml
   <!--设置可以用spring的el表达式配置Spring Security并自动生成对应配置组件（过滤器）-->
   <security:http auto-config="true" use-expressions="true">
       <!--省略其余配置-->
       <!--开启remember me过滤器，设置token存储时间为60秒-->
       <security:remember-me token-validity-seconds="60"/>
   </security:http>
   ```

扩展：

​		由于Cookie的值与用户名、密码等敏感数据相，虽然加密了，但保存在客户端依然不太安全。

​		因此SpringSecurity提供了remember me的另一种相对更安全的实现机制 ：

​				在客户端cookie中，仅保存一个无意义的加密串，然后在db中保存该加密串-用户信息的对应关系，自动		登录时，用cookie中的加密串，到db中验证，如果通过，自动登录才算通过。

​		若开启此功能，由于需要持久化cookie的数据，则必选按照SpringSecurity的要求创建如下表：

```sql
CREATE TABLE `persistent_logins` (
  `username` varchar(64) NOT NULL,
  `series` varchar(64) NOT NULL,
  `token` varchar(64) NOT NULL,
  `last_used` timestamp NOT NULL,
  PRIMARY KEY (`series`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

​		同时在spring-security.xml中配置持久化需要的datasource：

```xml
<!--开启remember me过滤器，data-source-ref="dataSource" 指定数据库连接池;
												 token-validity-seconds="60" 设置token存储时间为60秒;
													remember-me-parameter="remember-me" 指定记住的参数名 -->
<security:remember-me data-source-ref="dataSource"
											token-validity-seconds="60" remember-me-parameter="remember-me"/>
```

​	  `remember-me-parameter="remember-me"`用于指定记住我`input`标签的`name`属性。若`name`属性不为`remember-me`，可在此处指定。

### 3.3.2、显示当前认证用户名

**方案一：**java获取，页面使用EL表达式获取

```java
String username = SecurityContextHolder.getContext().getAuthentication().getName();
```

或

```java
String username = ((SysUser) SecurityContextHolder.getContext().getAuthentication().getPrincipal()).getUsername();
```

**方案二：**直接使用security-taglibs标签：

```jsp
<%@taglib uri="http://www.springframework.org/security/tags" prefix="security"%>
<span class="hidden-xs">
		<security:authentication property="name" />
</span>
```

或者

```jsp
<%@taglib uri="http://www.springframework.org/security/tags" prefix="security"%>
<span class="hidden-xs">
		<security:authentication property="principal.username" />
</span>
```

## 3.4、Spring Security授权

### 3.4.1、动态展示菜单

1. 添加授权的注解支持

   ```xml
   <!-- 开启权限控制的注解支持 -->
   <!-- springSecurity内部的权限控制注解开关 -->
   <security:global-method-security secured-annotations="enabled" />
   <!-- spring指定的权限控制的注解开关 -->
   <!-- <security:global-method-security pre-post-annotations="enabled" /> -->
   <!-- 开启java250注解支持 -->
   <!-- <security:global-method-security jsr250-annotations="enabled" /> -->
   ```

   **说明：**secured-annotations、pre-post-annotations、jsr250-annotations可同时开启。开启可使用相应的			注解。一般视情况使用一种即可。

2. 在类或方法上添加权限注解

   ```java
   @Controller
   @RequestMapping("/product")
   public class ProductController {
       //@PreAuthorize("hasAnyAuthority('ROLE_PRODUCT','ROLE_ADMIN')")//pre-post-annotations
       // @RolesAllowed({"ROLE_PRODUCT","ROLE_ADMIN"})	// jsr250-annotations
       // 表示findAll方法需要ROLE_ADMIN或ROLE_PRODUCT才能访问
       @Secured({"ROLE_PRODUCT","ROLE_ADMIN"}) // springSecurity注解
       @RequestMapping("/findAll")
       public String findAll(){
           return "product-list";
       }
   }
   ```
   
```java
   //表示当前类中所有方法都需要ROLE_ADMIN或者ROLE_PRODUCT才能访问
   @Controller
   @RequestMapping("/product")
   @RolesAllowed({"ROLE_ADMIN","ROLE_PRODUCT"})
   public class ProductController {
       @RequestMapping("/findAll")
       public String findAll(){
       		return "product-list";
       }
   }
   ```
   
3. 在jsp页面菜单上添加权限控制标签

   ```jsp
   <!-- 关键代码 -->
   <%@taglib uri="http://www.springframework.org/security/tags" prefix="security"%>
   <ul class="sidebar-menu">
     <li class="header">菜单</li>
     <li class="treeview">
       <ul class="treeview-menu">
         	<!-- 拥有'ROLE_PRODUCT', 'ROLE_ADMIN'角色的用户可见 -->
         	<security:authorize access="hasAnyRole('ROLE_PRODUCT', 'ROLE_ADMIN')">
              <li id="system-setting">
               <a href="${pageContext.request.contextPath}/product/findAll">
                   产品管理
               </a>
              </li>
         	</security:authorize>
           <!-- 拥有'ROLE_ORDER', 'ROLE_ADMIN'角色的用户可见 -->
           <security:authorize access="hasAnyRole('ROLE_ORDER', 'ROLE_ADMIN')">
               <li id="system-setting">
                 <a href="${pageContext.request.contextPath}/order/findAll">
                     订单管理
                 </a>
               </li>
         	</security:authorize>
       </ul>
     </li>
   </ul>
   ```

### 3.4.2、权限异常处理

**方式一：**在spring-security.xml配置文件中处理

```xml
<!--设置可以用spring的el表达式配置Spring Security并自动生成对应配置组件（过滤器）-->
<security:http auto-config="true" use-expressions="true">
    <!--省略其它配置-->
    <!--403异常处理，仅仅能处理403异常-->
    <security:access-denied-handler error-page="/403.jsp"/>
</security:http>
```

**方式二：**在web.xml中处理

```xml
<error-page>
  	<!-- 也可以配置其他异常，不过需要分别处理 -->
    <error-code>500</error-code>
    <location>/500.jsp</location>
    <error-code>403</error-code>
    <location>/403.jsp</location>
</error-page
```

**方式三**：编写异常处理器（推荐）

```java
@ControllerAdvice
public class ControllerExceptionAdvice {
    // 拦截所有AccessDeniedException异常并处理
    @ExceptionHandler(AccessDeniedException.class)
    public String exceptionAdvice(){
    		return "forward:/403.jsp";
    }
}
```

# 四、Spring Boot 整合 Spring Security

## 4.1、单体架构应用整合

1. 新建项目导入依赖

   ```xml
   <!-- 主要依赖 -->
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

   导入**spring-boot-starter-security**依赖后，SpringBoot会默认对security进行配置。如：登入、登出、用户认证方式、默认用户user（密码在启动日志中）等。

2. 添加配置类

   ```java
   @Configuration
   @EnableWebSecurity
   @EnableGlobalMethodSecurity(securedEnabled=true)	// 开启方法级别的授权
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
   
       @Autowired
       @Qualifier("userDetailsServiceImpl")
       private UserDetailsService userDetailsService;
   
       @Bean
       public BCryptPasswordEncoder passwordEncoder(){
           return new BCryptPasswordEncoder();
       }
   
       // 指定认证对象的来源、加密策略
       public void configure(AuthenticationManagerBuilder auth) throws Exception {
           auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
       }
       //SpringSecurity配置信息
       public void configure(HttpSecurity http) throws Exception {
           http.authorizeRequests()
                   .anyRequest().authenticated()
                   .and()
                   .formLogin().permitAll()
                   .and()
                   .logout().permitAll()
                   .and()
                   .csrf().disable();
       }
   }
   ```

   spring security默认会在指定的角色`ADMIN`和USER前添加前缀`ROLE_`。若此处指定角色为`ROLE_ADMIN`或`ROLE_USER`将报错：

   ```java
   org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'springSecurityFilterChain' defined in class path resource [org/springframework/security/config/annotation/web/configuration/WebSecurityConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [javax.servlet.Filter]: Factory method 'springSecurityFilterChain' threw exception; nested exception is java.lang.IllegalArgumentException: role should not start with 'ROLE_' since it is automatically inserted. Got 'ROLE_ADMIN'
   ```

3. 添加认证逻辑

   ```java
   @Configuration(value = "userDetailsServiceImpl") 
   public class UserDetailsServiceImpl implements UserDetailsService {
   
       @Autowired
       private UserServiceImpl userServiceImpl;
   
       @Autowired
       private RoleServiceImpl roleServiceImpl;
   
       /**
        * 返回一个包括用户名，密码，授权信息等数据的UserDetails对象
        */
       @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
   
           // 根据username从数据库中获取对应User对象
           User user = userServiceImpl.getUserByName(username);
           // 判断获取用户是否为null，若用户不存在，则返回null。Spring Security会抛出异常，并自动处理此异常
           if (user == null) { return null; }
   
           // 根据username从数据库中获取对应User的角色
           List<Role> roles = roleServiceImpl.getRoleByName(username);
   
           List<SimpleGrantedAuthority> authorityList = new ArrayList<>();
           for(Role role : roles){
               // 此处的角色名称必须要 ROLE_ 前缀!!!
               SimpleGrantedAuthority authority = new SimpleGrantedAuthority(role.getRoleName());
               authorityList.add(authority);
           }
   
           // 密码前添加{noop}，表示不需要对表单密码进行加密，直接和{noop}后的密码作对比。
           return new org.springframework.security.core.userdetails.User(user.getName(), "{noop}" + user.getPassword(), authorityList);
       }
   }
   ```

4. 添加异常处理

   ```java
   @ControllerAdvice
   public class HandleControllerException {
       @ExceptionHandler(RuntimeException.class)
       public Object exceptionHandler(RuntimeException e){
         	// TODO：403异常处理逻辑
           if(e instanceof AccessDeniedException){ 
             return null;
           }
           // TODO：其他异常处理逻辑
           return null;
       }
   }
   ```

5. 测试Controller

   ```java
   @RestController
   public class TestController {
       
       @GetMapping("/visitor/test")
       public String visitor(){
           return "hello visitor";
       }
   
       @Secured("ROLE_USER")
       @GetMapping("/user/test")
       public String user(){
           return "hello user";
       }
   
       @Secured("ROLE_ADMIN")
       @GetMapping("/admin/test")
       public String admin(){
           return "hello admin";
       }
   }
   ```

   

## 4.2、分布式应用整合





# Spring security进阶

## Spring Security默认过滤器

1. org.springframework.security.web.context.SecurityContextPersistenceFilter
  首当其冲的一个过滤器，作用之重要，自不必多言。
  SecurityContextPersistenceFilter主要是使用SecurityContextRepository在session中保存或更新一个
  SecurityContext，并将SecurityContext给以后的过滤器使用，来为后续filter建立所需的上下文。
  SecurityContext中存储了当前用户的认证以及权限信息。

2. org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter
  此过滤器用于集成SecurityContext到Spring异步执行机制中的WebAsyncManager

3. org.springframework.security.web.header.HeaderWriterFilter
  向请求的Header中添加相应的信息,可在http标签内部使用security:headers来控制

4. org.springframework.security.web.csrf.CsrfFilter
  csrf又称跨域请求伪造，SpringSecurity会对所有post请求验证是否包含系统生成的csrf的token信息，
  如果不包含，则报错。起到防止csrf攻击的效果。

5. org.springframework.security.web.authentication.logout.LogoutFilter

   匹配URL为/logout的请求，实现用户退出,清除认证信息。
6. org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter
  认证操作全靠这个过滤器，默认匹配URL为/login且必须为POST请求。
7. org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter
  如果没有在配置文件中指定认证页面，则由该过滤器生成一个默认认证页面。

8. org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter
  由此过滤器可以生产一个默认的退出登录页面

9. org.springframework.security.web.authentication.www.BasicAuthenticationFilter
  此过滤器会自动解析HTTP请求中头部名字为Authentication，且以Basic开头的头信息。

10. org.springframework.security.web.savedrequest.RequestCacheAwareFilter
    通过HttpSessionRequestCache内部维护了一个RequestCache，用于缓存HttpServletRequest

11. org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter
    针对ServletRequest进行了一次包装，使得request具有更加丰富的API

12. org.springframework.security.web.authentication.AnonymousAuthenticationFilter

    当SecurityContextHolder中认证信息为空,则会创建一个匿名用户存入到SecurityContextHolder中。
    spring security为了兼容未登录的访问，也走了一套认证流程，只不过是一个匿名的身份。

13. org.springframework.security.web.session.SessionManagementFilter
    SecurityContextRepository限制同一用户开启多个会话的数量
14. org.springframework.security.web.access.ExceptionTranslationFilter
    异常转换过滤器位于整个springSecurityFilterChain的后方，用来转换整个链路中出现的异常
15. org.springframework.security.web.access.intercept.FilterSecurityInterceptor
    获取所配置资源访问的授权信息，根据SecurityContextHolder中存储的用户信息来决定其是否有权
    限。