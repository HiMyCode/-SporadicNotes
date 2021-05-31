[TOC]

参考链接：https://blog.csdn.net/abc997995674/category_7616847.html



# 一、Spring MVC 请求原理

## 1.1、核心控制器（DispatcherServlet）

![SpringMVC原理图](C:\Users\pc\Documents\-SporadicNotes\图片\SpringMVC原理图.png)

DispatcherServlet 会拦截所有的请求，并且将这些请求发送给 Spring MVC 控制器 。

```xml
<!-- 在web.xml中配置核心控制器 -->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <!-- 默认加载配置文件：/WEB-INF/[servlet名字]-servlet.xml -->
        <param-value>classpath:springmvc-config.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```
## 1.2、处理器映射（HandlerMapping）

DispatcherServlet 会查询一个或多个处理器映射，处理器映射会**根据请求所携带的 URL 信息来进行决策** .

   ```xml
   <bean id="simpleUrlHandlerMapping"
         class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
       <property name="mappings">
           <props>
               <!-- /hello 路径的请求交给 id 为 helloController 的控制器处理-->
               <prop key="/hello">helloController</prop>
           </props>
       </property>
   </bean>
   <bean id="helloController" class="com.demo.controller.HelloController"></bean>
   ```

## 1.3、控制器返回处理结果给DispatcherServlet

DispatcherServlet 会将请求发送给匹配的控制器，并由控制器处理 用户提交的请求。

```java
public ModelAndView handleRequest(javax.servlet.http.HttpServletRequest request, javax.servlet.http.HttpServletResponse response, ModelAndView modelAndView) throws Exception {
    // TODO 处理请求
    ModelAndView modelAndView = new ModelAndView();
    // 1、将处理结果封装成模型（Model）
    modelAndView.addObject("userName","Tom");
    // 2、返回一个用于展示结果的视图（view）, 如：JSP
    modelAndView.setViewName("success");
    // 3、将 请求 、 模型 、 视图名（逻辑名称 ） 发送给 DispatcherServlet 
    return modelAndView;
}
```
## 1.4、视图解析器（view resolver）

DispatcherServlet 使用视图解析器将**逻辑视图名**匹配为一个特定的视图实现。步骤4，将返回到success.jsp

## 1.5、视图

视图将DispatcherServlet 传递模型数据进行渲染，并将结果传递给客户端。

##  1.6、主要组件介绍

### 1.6.1、HandlerMapping

​	**作用**：解析请求链接，根据请求链接匹配到执行此请求的Handler类。

​		1. 根据配置文件对url到controller的映射进行注册；

​		2. 根据具体的url请求找到执行该请求的controller。

### 1.6.2、HandlerAdapter

​	**作用**：调用具体的方法对用户发来的请求来进行处理。

SpringMVC默认处理适配器（DispatcherServlte会读取DispatcherServlte.properties文件获取）：

- AnnotationMethodHandlerAdapter：适配注解类处理器，适配标注了@Controller的处理器；
- HttpRequestHandlerAdapter：适配静态资源处理器，适配实现了HttpRequestHandler接口的处理器，用于处理通过SpringMVC来访问的静态资源的请求；
- SimpleControllerHandlerAdapter：Controller处理适配器，适配实现了Controller接口或
  Controller接口子类的，如继承MultiActionController；

Servlet处理适配器：

- SimpleServletHandlerAdapter：适配实现了Servlet接口或Servlet的子类的处理器。既可在web.xml中配置Servlet，也可用SpringMVC配置Servlet。

### 1.6.3、HandlerExceptionResolver

​	**作用**：用来捕获所有的异常。

​	使用`<mvc:annotation-driven/>`后，将会向Spring MVC容器中注入以下异常处理器：

- ExceptionHandlerExceptionResolver

  解析处理器类中注解的@ExceptionHandler的方法；

  使用@ControllerAdvice注解的类里的有@ExceptionHandler注解的全局异常处理方法

- ResponseStatusExceptionResolver

  解析有@ResponseStatus注解的异常

- DefaultHandlerExceptionResolver

  按照不同的类型分别对异常进行解析，Spring MVC内部使用。

其他异常处理器：

- SimpleMappingExceptionResolver

  通过配置的异常类和view的对应关系来解析异常。

  

# 二、Spring MVC 配置文件

##2.1、web.xml 配置核心控制器

```xml
    <!--配置springmvc 前端控制器 -->
    <servlet>
        <servlet-name>springMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <!--配置dispatcher-servlet.xml作为mvc配置文件，默认为：“/WEB-INF/[servlet名字]-servlet.xml -->
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/dispatcher-servlet.xml</param-value>
        </init-param>
        <!-- 值大于等于0时，该servlet在容器启动时被加载；空或负数时，servlet被使用时加载；值越小优先级越高 -->
        <load-on-startup>1</load-on-startup>
        <async-supported>true</async-supported>
    </servlet>

    <servlet-mapping>
        <servlet-name>springMVC</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```



##2.2、dispatcher-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/mvc
                            http://www.springframework.org/schema/mvc/spring-mvc.xsd
                            http://www.springframework.org/schema/context
                            http://www.springframework.org/schema/context/spring-context.xsd">

    <!--配置项目默认首页-->
    <mvc:view-controller path="/" view-name="index"/>
    
    <!--启用spring的一些annotation -->
    <context:annotation-config/>

    <!-- 开启Spring容器启动时自动扫描装配 -->
    <context:component-scan base-package="com.demo"/>

    <!-- SpringMVC三大组件：处理器映射器、处理器适配器、视图解析器-->
    <!-- 加载：RequestMappingHandlerMapping（处理器映射器） 和 RequestMappingHandlerAdapter（适配器）-->
    <!-- <mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven> -->
    <!-- 配置注解驱动：将request参数绑定到controller参数上 -->
    <!-- 若 org.springframework.context.support.ConversionServiceFactoryBean 的 bean名称不为默认的 conversionService ，则需要指定 conversion-service 参数-->
    <mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>

    <!-- 视图解析器-->
    <bean id="defaultViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- InternalResourceViewResolver默认使用InternalResourceView作为视图解析器；若要使用JSTL标签，需要用JstlView来替换InternalResourceView-->
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/views/"/><!--设置JSP文件的目录位置-->
        <property name="suffix" value=".jsp"/>
        <property name="exposeContextBeansAsAttributes" value="true"/>
    </bean>

    <!-- 自定义类型转换器-->
    <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="com.demo.springmvc.converter.StringToDateConverter"></bean>
            </set>
        </property>
    </bean>

    <!-- 文件上传转换器,id必须为：multipartResolver-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize" value="2000000000"/>
        <property name="defaultEncoding" value="utf-8"></property>
    </bean>

    <!--静态资源映射：webapp/static -->
    <mvc:resources mapping="/css/**" location="../static/css/"/>
    <mvc:resources mapping="/js/**" location="../static/js/"/>
    <mvc:resources mapping="/image/**" location="../static/images/"/>

    <!-- 区分请求为“静态资源的请求”还是“普通请求”。若是静态资源的请求，将请求转由Web应用服务器默认的Servlet处理；若不是静态资源的请求，由DispatcherServlet处理。
         方式一：<mvc:default-servlet-handler /> + <mvc:annotation-driven />
         方式二：<mvc:resources />
 	     方式三：如果不使用rest，可将Spring的全局拦截设置为 *.do 的拦截
     -->
    <!--参考地址：https://www.cnblogs.com/dflmg/p/6393416.html-->
    <mvc:default-servlet-handler />
    <mvc:annotation-driven />

    <!-- 配置自定义异常处理器 -->
    <bean id="handlerExceptionResolver" class="com.demo.springmvc.exception.CustomExceptionResolver" />

    <!-- 配置自定义异常拦截器 -->
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>
            <bean id="customInterceptor1" class="com.demo.springmvc.interceptor.CustomInterceptor1" ></bean>
        </mvc:interceptor>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>
            <bean id="customInterceptor2" class="com.demo.springmvc.interceptor.CustomInterceptor2" ></bean>
        </mvc:interceptor>
    </mvc:interceptors>

</beans>
```

- 关于SpringMvc包扫描

  ​	spring容器是springmvc容器的父容器。**父容器可以看到子容器的Bean,子容器中不能看到父容器中的Bean**。如果两个容器都配置了包扫描,会有同一个bean被创建两次的情况。

  ​	常用解决方案：在springmvc配置文件中配置只扫描@Controller注解，spring容器配置排除扫描@Controller注解。 

```java
<!-- springmvc中配置只扫描@Controller注解 -->
<context:component-scan base-package="com.springmvc" use-default-filters="false">
    <context:include-filter type="annotation"
        expression="org.springframework.stereotype.Controller" />
</context:component-scan>
```



#三、Spring MVC 参数绑定

## 3.1、基本数据类型绑定

```java
/**
 * 要求请求参数中 参数的名称 与 方法形参名 一致，常用类型Spring会使用默认加载的格式转换器进行类型转换
 * 例： /bind/findUserById?userId=001&age=11。
 */
@RequestMapping(value="/findUserByUserId")
public String findUserByUserId(String userId, int age) {
    return "success";
}

/**
 * 若请求参数中 参数的名称 与 方法形参名 不一致，可使用 @RequestParam 注解:
 * 	 @RequestParam(value = "id") 将参数中 id 的值映射为 userId 
 * 	 @RequestParam(required = true) 未传参数时，抛异常
 * 	 @RequestParam(defaultValue = "001") 未传参数时，设置默认值
 */
@RequestMapping(value="/findUserByUserId")
public String findUserByUserId(
    	@RequestParam( value = "id", required = true, defaultValue = "001") String userId, 
    	@RequestParam( value = "age", required = false, defaultValue = "18")int age ) {
    return "success";
}
```

## 3.2、POJO数据类型绑定

```java
/**
 * 绑定User对象：
 * 要求请求参数中 参数的名称 与 POJO中属性名称 一致，且方法参数类型为要绑定的 POJO 对象
 */
@RequestMapping(value="/findUserByVo")
 public String findUserByVo(User user) {
     return "success";
 }

/**
 * 绑定User对象，且User对象中 Address对象 属性：
 *     User    对象表单参数示例： <input type="text" name="userName" />
 *     Address 对象表单参数示例： <input type="text" name="address.addressName" />
 */
@RequestMapping(value="/findUserByVo")
 public String findUserByVo(User user) {
     return "success";
 }
```

## 3.3、Array 或 Map 类型参数

```java
/**
  * 数组或集合类型参数
  *      User    对象表单参数示例： <input type="text" name="userName" />
  *      Address 对象表单参数示例： <input type="text" name="address.addressName" />
  *      List<Account>  对象表单参数示例：
  *               <input type="text" name="accountsList[0].accountCode" />
  *               <input type="text" name="accountsList[1].accountCode" />
  *      Map<Account>   对象表单参数示例： 
  *               <input type="text" name="accountsMap['wechat'].accountCode" />
  *               <input type="text" name="accountsMap['qq'].accountCode" />
  */
@RequestMapping(value="/saveVos")
public String saveVos(List<User> users) {
    return "success";
}
```

##3.4、servletApi 参数绑定

```java
@RequestMapping(value="/servletAPIParameter")
public String servletAPIParameter(HttpRequest request, HttpResource response) {
    request.setAttribute("name", "Tom");
    return "success";
}
```

#四、Spring MVC 数据类型转换

## 4.1、Spring MVC内置转换器

Spring MVC内置转换器所在包位置：`package org.springframework.core.convert.support`。包括：

![SpringMVC标量转换器](C:\Users\pc\Documents\-SporadicNotes\图片\SpringMVC标量转换器.png)

![SpringMVC集合、数组相关转换器](C:\Users\pc\Documents\-SporadicNotes\图片\SpringMVC集合、数组相关转换器.png)

参考：https://blog.csdn.net/coldplay/article/details/102839502

## 4.2、Spring MVC内置时间转换器

1、在xml配置文件中配置：`<mvc:annotaton-driven/>` 

2、在参数上使用注解 @DateTimeFormat

~~~java
@RequestMapping("/dateFormat")
public String dateFormat(@DateTimeFormat(pattern = "yyyy-MM-dd") Date birthday){
    System.out.println(dateStr);
    return "success";
}
~~~

2、在实体上使用注解 @DateTimeFormat

~~~java
public class User {

    private String userName;

    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date birthday;
}
~~~

## 4.3、使用PropertyEditor进行数据转换

###4.3.1、局部属性编辑器 <a id="@InitBinder">@InitBinder</a> 

**（1）**使用系统默认提供的属性编辑器

~~~java
/**
 * 注册属性编辑器（字符串 -> Date），编辑器仅对方法所在controller有效；
 */
@InitBinder
public void initBinder(DataBinder bin){
    // 自定义时间属性编辑器
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    CustomDateEditor cust = new CustomDateEditor(sdf, true);
    // 绑定属性编辑器到Date类型上
    bin.registerCustomEditor(Date.class,cust);
}
~~~

​	Spring还提供了大量的编辑器实现类，如:  CustomDateEditor，CustomBooleanEditor，CustomNumberEditor都继承自PropertyEditorSupport类。

​	@InitBinder(value="person")

​		为属性名称为person的参数添加自定义的属性编辑器;

​		若不指定则为所有的相关类型参数进行添加自定义编辑器。

**（2）**使用自定义的属性编辑器

- 自定义属性编辑器 

~~~java
/**
 * 功能：将竖线分割的多个字符串转换为String[]
 **/
public class StringToListPropertyEditor extends PropertyEditorSupport {

	@Override
	public void setAsText(String text) throws IllegalArgumentException {
		String[] resultArr = null;
		if (!StringUtils.isEmpty(text)) {
			resultArr = text.split("_");
		}
         // 设置转换后的值
		setValue(resultArr);
	}
}
~~~

- 注册属性编辑器并使用

```java
@RequestMapping("/stringToListTest")
@Controller
public class StringToListController {

	@InitBinder
	public void stringToListBinderTest(WebDataBinder dataBinder) {
		dataBinder.registerCustomEditor(String[].class, new StringToListPropertyEditor());
	}

    // 访问参数：?strToListArr=11_22_33
	@RequestMapping(value = "/test", method = RequestMethod.GET)
	@ResponseBody
	public String myStringToListTest(String[] strToListArr, HttpServletResponse response) {
		response.setCharacterEncoding("UTF-8");
		if (strToListArr != null && strToListArr.length > 0) {
			return Arrays.asList(strToListArr).toString();
		}
		return "[]";
	}
}
```

 **（3）**处理带有前缀的form字段

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>$Title$</title>
    </head>
    <body>
        <form action="${pageContext.request.contextPath}/myInitBinder0954/test0942"  					 method="post" enctype="multipart/form-data">
            <input type="text" name="people.name" placeholder="人名"><br><br>
            <input type="text" name="people.age" placeholder="人年龄"><br><br>
            <input type="text" name="address.name" placeholder="地址名称"><br><br>
            <input type="text" name="address.city" placeholder="地址所在城市"><br><br>
            <input type="submit" value="提交"/>
        </form>
    </body>
</html>
```

```java
public class People {
	private String name;
	private String age;
}

public class Address {
	private String name;
    private String city;
}
```

```java
@RequestMapping("/initBinderTest")
@Controller
public class InitBinderTestController {
	@InitBinder(value = "people")
	public void initBinderSetDefaultPreifixPeople(WebDataBinder dataBinder) {
		dataBinder.setFieldDefaultPrefix("people.");
	}

	@InitBinder(value = "address")
	public void initBinderSetDefaultPreifixAddress(WebDataBinder dataBinder) {
		dataBinder.setFieldDefaultPrefix("address.");
	}

	@RequestMapping(value = "/testFromPrefix", method = RequestMethod.POST)
	@ResponseBody
	public String testFromPrefix(@ModelAttribute("people") People people, @ModelAttribute("address") Address address) {
		StringBuffer sb = new StringBuffer();
		sb.append(people.toString()).append("-")append(address.toString());
		return sb.toString();
	}
}
```

##4.4、使用Formatter进行数据转换

1. 自定义Formatter类型转换器 

```java
// 定义一个类实现Formatter<T>接口，Date 为转换后的类型
public class MyDateFormatter implements Formatter<Date> {

    /**
     * text : 接收的数据
     * return: 转换后的数据
     **/
    public Date parse(String text, Locale locale) throws ParseException {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        return sdf.parse(text);
    }

    public String print(Date object, Locale locale) {
        return null;
    }
}
```

2. 在配置文件中注册转换器

```java
<mvc:annotation-driven conversion-service="myDateFormatter"/>

    <bean id="myDateFormatter" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="formatters">
            <set>
                <bean class="com.test.common.MyDateFormatter"></bean>
            </set>
        </property>
    </bean>
```

**Formatter 只能对String类型的参数进行转化。** 

##4.5、使用Converter进行数据转换

1. 自定义Converter类型转换器 

```java
// 泛型S为接收的参数类型; 泛型T为转换后的类型
public class MyDateConverter implements Converter<String, Date> {

    @Override
    public Date convert(String source) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        try {
            return sdf.parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return null;
    }
```

2. 在配置文件中注册转换器

```jjava
<mvc:annotation-driven conversion-service="myDateConverter" />

<bean id="myDateConverter"
    class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="com.test.common.MyDateConverter"></bean>
            </set>
        </property>
    </bean>
```

**注册Converter服务后，所有使用了@RequestMapping注解的方法参数，将按规定的格式转换为Date类型。**

#四、Spring MVC 校验器

1. 实体类

```java
public class User {
	private String userName;
}
```

2. 实现接口 `org.springframework.validation.Validator` ，自定义校验器

```java
@Component
public class UserValidator implements Validator {

	@Override
	public boolean supports(Class<?> clazz) {
		// 只支持User类型对象的校验
		return User.class.equals(clazz);
	}

	@Override
	public void validate(Object target, Errors errors) {
		User user = (User) target;
		String userName = user.getUserName();
		if (StringUtils.isEmpty(userName) || userName.length() < 8) {
             // errors数据会被设置到 org.springframework.validation.BindingReuslt
			errors.rejectValue("userName", "valid.userNameLen",
					new Object[] { "minLength", 8 }, "用户名不能少于{1}位");
		}
	}
}
```

3. 使用自定义校验器

```java
@Controller
@RequestMapping("/valid")
public class ValidatorController {

	@Autowired
	private UserValidator userValidator;

	@InitBinder
	private void initBinder(WebDataBinder binder) {
		binder.addValidators(userValidator);
	}

	@RequestMapping(value = { "/index", "" }, method = { RequestMethod.GET })
	public String index(ModelMap m) throws Exception {
		m.addAttribute("user", new User());
		return "initbinder/user.jsp";
	}

	@RequestMapping(value = { "/signup" }, method = { RequestMethod.POST })
	public String signup(@Validated User user, BindingResult br, RedirectAttributes ra) throws Exception {
		// 携带用户录入的信息方便回显
		ra.addFlashAttribute("user", user);
		return "initbinder/user.jsp";
	}
}
```

4、返回页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<html>
    <head>
        <title>validate user</title>
    </head>
    <body>
        <form:form modelAttribute="user" action="/valid/signup" method="post">
            <!-- 显示所有的错误信息 -->
            <form:errors path="*"></form:errors><br><br>
            用户名：<form:input path="userName"/><form:errors path="userName"/>
        </form:form>
    </body>
</html>
```



#四、Spring MVC 向页面返回数据

## 4.1、使用Model

```java
@RequestMapping("/sayHello")
public String sayHello(Model model){
    model.addAttribute("name", String.format("你好，%s!",name), "Tom");
    return "success";
}
```

## 4.2、使用ModelAndView

```java
// 页面使用EL表达式 ${name} 获取数据
@RequestMapping("/sayHello")
public ModelAndView sayHello(String name){
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.addObject("name", String.format("你好，%s!",name));
    modelAndView.setViewName("success");
    return modelAndView;
}
```

## 4.3、使用Map

```java
@RequestMapping("/sayHello")
public String sayHello(Map map){
    map.put("name", "Tom");
    map.put("sex","男");
    return "success";
}
```

## 4.4、使用ModelMap

```java
@RequestMapping("/sayHello")
public String sayHello(ModelMap modelMap){
    modelMap.addAttribute("name", "Tom");
    modelMap.addAttribute("sex","男");
    // modelMap.put("name", "Tom");
    // modelMap.put("sex","男");
    return "success";
}
```

## 4.5、使用request

```java

@RequestMapping("/sayHello.do")
public String sayHello(HttpServletRequest request){
    request.setAttribute("name", "Tom");
    return "success";
}
```

#五、重定向和转发

## 5.1、重定向

1. 重定向后浏览器 URL 发生变化，客户端重新通过URL 发送GET请求；

2. 重定向前Model中的数据不会绑定到重定向后方法的Model对象中（重定向Model不共享 ）；

   但重定向前Model中的数据会自动拼接到重定向的URL后；

   所以重定向前Model中的数据会自动绑定到方法的同名参数**上 或 使用request.getParameter获取；

3. 重定向前 Request 中的数据 无法再次获取。

   **请求重定向参数传递方式：**

```java
// 方式一：使用 Model （ModelAndView、ModelMap、Map 和 Model效果相同）
@RequestMapping("/index")
public String index(Model, model, String name, String age){
    model.addAttribute("name",name);
    model.addAttribute("age",age);
    return "redirect:success";
}

// 浏览器中的URL : http://localhost:8081/SpringMVC/redirect/success?name=Tom&age=18
@RequestMapping("/success")
public String success(HttpServletRequest request, Model model, String name, String age){

    System.out.println(request.getAttribute("name"));		 // 输出：null
    System.out.println(request.getAttribute("age"));		 // 输出：null
    
    System.out.println(request.getParameter("name"));		 // 输出：Tom
    System.out.println(request.getParameter("age"));		 // 输出：18
    
    System.out.println(model.containsAttribute("name"));	  // 输出：false
    System.out.println(model.containsAttribute("age"));		  // 输出：false
    
    System.out.println(name)								// 输出：Tom
    System.out.println(age);								// 输出：18

    return "success";
}
```

```java
/**
 * 方式二：使用 RedirectAttributes 的 addAttribute 方法 (SpringMVC 3.1添加)
 **/
@RequestMapping("/index")
public String index(Model, model, String name, String age){
    redirectAttributes.addAttribute("name", "Tom");
    redirectAttributes.addAttribute("age", "18");
    return "redirect:success";
}

// 参数传递效果同 方式一
```

前两种方式传递的参数会跟在URL后面 ，若传递参数敏感，如密码，可使用如下方式：

```java
/**
 * 方式三：使用 RedirectAttributes 的 addFlashAttribute 方法 (SpringMVC 3.1添加)
 *        addFlashAttribute 将参数放到session中，待重定向页面渲染完成后从session中移除，
 *        故再次刷新将无法获取跳转前的参数
 **/
@RequestMapping("/idenx")
public String idenx(HttpServletRequest request, RedirectAttributes redirectAttributes){
    redirectAttributes.addAttribute("name", request.addAttribute("name"));
    // 这种方式会将参数放到session中,如果刷新页面就会丢失
    redirectAttributes.addFlashAttribute("age", request.addAttribute("age"));
    return "redirect:success";
}

// 浏览器中的URL : http://localhost:8081/SpringMVC/redirect/success
// 再次请求此地址刷新后，页面将无法获取 name 和 age 属性
@RequestMapping("/success")
public String success(HttpServletRequest request, Model model){

    System.out.println(request.getParameter("name"));		 // 输出：tom
    System.out.println(request.getParameter("age"));		 // 输出：null
    
    System.out.println(model.containsAttribute("name"));	  // 输出：false
    System.out.println(model.containsAttribute("age"));		  // 输出：true

    return "success";
}
```

## 5.2、转发

1. 转发不会再经过客户端，所以浏览器 URL 保持不变；
2. 可以在转发后的方法获取转发前中request对象的数据

```java
// 浏览器请求地址：http://localhost:8080/SpringMVC/index
@RequestMapping(value="/index",method=RequestMethod.GET)
public String index(){
   return "forward:hello1";
}
// 服务器根据 转发请求("forward:hello1") 和 请求方式(method=RequestMethod.GET) 将请求转发给此方法
@RequestMapping("/hello1",method=RequestMethod.GET)
public String hello1(){
   return "hello1";
}

@RequestMapping("/hello2",method=RequestMethod.POST)
public String hello2(){
   return "hello2";
}
```

#五、Spring MVC 文件上传/下载

##5.1、文件上传

1. 上传文件原理 简介

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 设置文件编码格式 -->
    <property name="defaultEncoding" value="UTF-8"></property>
    <!-- 设置最大上传大小 -->
    <property name="maxUploadSize" value="10485760"></property>
</bean>
```

​	收到请求时，DispatcherServlet 的 checkMultipart() 方法会调用 MultipartResolver 的 isMultipart() 方法判断请求中是否包含文件。

​	若请求数据中包含文件，则调用 MultipartResolver 的 resolveMultipart() 方法对请求的数据进行解析，然后将文件数据解析成 MultipartFile 并封装在 MultipartHttpServletRequest (继承了 HttpServletRequest) 对象中，最后传递给 Controller。

​	在 MultipartResolver 接口中有如下方法： 

​		boolean isMultipart(HttpServletRequest request);	 // 是否是 multipart 

 		MultipartHttpServletRequest resolveMultipart(HttpServletRequest request); 	// 解析请求 
 		void cleanupMultipart(MultipartHttpServletRequest request);

​	**MultipartFile** 封装了请求数据中的文件，此文件存储在内存中或临时的磁盘文件中，请求结束后临时存储将被清空。在 MultipartFile 接口中有如下方法： 
 	String getName(); // 获取参数的名称 
 	String getOriginalFilename(); // 获取文件的原名称 
 	String getContentType(); // 文件内容的类型 
 	boolean isEmpty(); // 文件是否为空 
 	long getSize(); // 文件大小 
 	byte[] getBytes(); // 将文件内容以字节数组的形式返回 
 	InputStream getInputStream(); // 将文件内容以输入流的形式返回 
 	void transferTo(File dest); // 将文件内容传输到指定文件中

2. 上传文件实例

```jsp
<h2>使用form表单上传文件</h2>
<form action="uploadByForm.do" method="post" enctype="multipart/form-data">
    <input type="file" name="file">
    <button type="submit">upload</button>
</form>
```

```java
@RequestMapping("/uploadByForm.do")
public String uploadByForm(MultipartFile file,HttpServletRequest request,Model model) throws IllegalStateException, IOException{
    if(!file.isEmpty()){
        //原文件名
        String originalFileName = file.getOriginalFilename();
        System.out.println("文件名:"+file.getOriginalFilename()+",文件大小:"+file.getSize());

        //获取项目根目录文件夹下 fileupload 子文件夹
        String ctxPath = request.getServletContext().getRealPath("/fileupload/");
        System.out.println(ctxPath);

        //获取文件后缀
        String extName = originalFileName .substring(originalFileName .lastIndexOf("."));
        //上传后的文件名
        String fileName = new Date().getTime() + extName;
        //将新文件名传回前台
        model.addAttribute("fileName",fileName);
        //文件按路径
        File filePath = new File(ctxPath,fileName);

        if(!filePath.getParentFile().exists()){
            //如果文件夹不存在就新建一个
            filePath.getParentFile().mkdirs();
        }
        //将文件上传到指定文件夹下
        file.transferTo(filePath);
    }
    return "success";
}
```

```java
@RequestMapping("/uploadByForm.do")
public String uploadByForm(MultipartFile file,HttpServletRequest request,Model model) throws IllegalStateException, IOException{
    if(!file.isEmpty()){
        //原文件名
        String originalFileName = file.getOriginalFilename();
        System.out.println("文件名:"+file.getOriginalFilename()+",文件大小:"+file.getSize());

        //获取项目根目录文件夹下 fileupload 子文件夹
//          String ctxPath = request.getServletContext().getRealPath("/fileupload/");
//          System.out.println(ctxPath);
        //存放文件目录
        String path = "F:/upload/"; 

        //获取文件后缀
        String extName = originalFileName .substring(originalFileName .lastIndexOf("."));
        //上传后的文件名
        String fileName = new Date().getTime() + extName;
        //将新文件名传回前台
        model.addAttribute("fileName",fileName);
        //文件按路径
        File filePath = new File(path,fileName);

        if(!filePath.getParentFile().exists()){
            //如果文件夹不存在就新建一个
            filePath.getParentFile().mkdirs();
        }

        InputStream in = file.getInputStream();

        OutputStream out = new FileOutputStream(filePath);

        byte[] b = new byte[1024];
        int i = 0;
        while((i=in.read(b)) != -1){
            out.write(b, 0, i);//将字节数组b从下标为0到下标为i的内容写到输出流中
        }
        out.flush();
        out.close();
        in.close();
    }
    return "success";
}
```

3. 使用ajax上传文件

```jsp
<h2>使用ajax上传文件</h2>
<form enctype="multipart/form-data" id="uploadForm">
    <input type="file" id="file" name="file">
    <button type="button" id="upload">upload</button>
</form>
<script>
    	$(function() {
            $('#upload').click(function() {
                var form = new FormData($('#uploadForm')[0]);
                $.ajax({
                    type : "POST",
                    url : "uploadByAjax.do",
                    data : form,
                    // 下面三个参数要指定，如果不指定，会报一个JQuery的错误 
                    cache : false,
                    contentType : false,
                    processData : false,
                    async : false,
                    success : function(data) {
                        $('p').html('上传成功，文件名为:' + data);
                    }
                });
            });
        });
</script>
```

## 5.2、文件下载

1. 使用ResponseEntity 下载文件

```java
@RequestMapping("/downloadFile.do")
public ResponseEntity<byte[]> downloadFile(String fileName) throws IOException {
     //http请求头
       HttpHeaders headers = new HttpHeaders();
       //设置attachment(附件)，让浏览器识别为下载
       headers.add("Content-Disposition", "attachment;filename="+fileName);
       //文件路径
       String downLoadPath = "F:/upload/"+fileName;
       //获取文件流
       InputStream in = new BufferedInputStream(new FileInputStream(downLoadPath)); 
       //将文件读到字节数组中
       byte[] b = new byte[in.available()];
       in.read(b);
       in.close();
       //http状态码也可以使用 HttpStatus.CREATED
       HttpStatus status = HttpStatus.OK;
       ResponseEntity<byte[]> responseEntity = new ResponseEntity<byte[]>(b,headers,status);
       return responseEntity;
}
```

2. 使用原始方式下载文件

```java
@ResponseBody
@RequestMapping("/downloadFile")
public byte[] testDownload(HttpServletResponse response,String fileName) throws IOException{
    //设置请求头，否则浏览器不会识别这是下载
    response.setHeader("Content-Disposition", "attachment;filename="+fileName);
    //文件路径
    String downLoadPath = "F:/upload/"+fileName;
    //获取文件流
    InputStream in = new BufferedInputStream(new FileInputStream(downLoadPath)); 
    //将文件读到字节数组中
    byte[] b = new byte[in.available()];
    in.read(b);
    in.close();
    return b;
}
```



# 六、Spring MVC 标签



## 6.1、 `<context:annotation-config />`

​	用于激活已经在spring容器里注册过的bean上面的注解，向Spring注册如下Bean：

- AutowiredAnnotationBeanPostProcessor ： 处理@Autowired注解
- CommonAnnotationBeanPostProcessor ： 处理 等注解
- PersistenceAnnotationBeanPostProcessor ：处理 @PersistenceContext注解
- RequiredAnnotationBeanPostProcessor ：处理 @Required注解

**注意：**

​		单独使用< context:annotation-config/>对并不能激活@Component、@Controller、@Service等注解。

​		< context:annotation-config/>仅能对已经注册的bean起作用。对没有注册的bean，不能执行任何操作。	



## 6.2、 `<context:annotation-driven />`



作用：Spring 3.0.x 版本加入。启用注解驱动 ，自动注册默认处理请求，参数和返回值的类 。

|              |          处理器映射器           |          处理器适配器          |
| :----------: | :-----------------------------: | :----------------------------: |
|    抽象类    |         HandlerMapping          |         HandlerAdapter         |
| Spring 3.0.x | DefaultAnnotationHandlerMapping | AnnotationMethodHandlerAdapter |
| Spring 3.1.x |  RequestMappingHandlerMapping   |  RequestMappingHandlerAdapter  |
|     作用     | 建立请求与Controller的映射关系  |     调用具体的方法处理请求     |

加载 RequestMappingHandlerMapping（处理器映射器） 和RequestMappingHandlerAdapter（适配器）

 

## 6.3、 `<context:component-scan />`

​		通过对**base-package**配置，就可以扫描到controller包下 service包下 dao包下的@Component、@Controller、@Service等这些注解。

​		配置<context:component-scan />后，<context:annotation-config />将被自动配置。故可以省略<context:annotation-config />。即使显式配置了< context:annotation-config/>也会被忽略。





## 6.2、`<mvc: xxxxxx>`

### 6.2.1、`<mvc:default-servlet-handler />` 

- **用途一：**

		开启web容器提供的默认的servlet，所有请求将由默认servlet处理，配置的dispatcherServlet将失效。

- **用途二：**

  ```java
  <mvc:default-servlet-handler default-servlet-name="xxx" /> 
  ```

  web应用启动后，DefaultServletHttpRequestHandler 通过默认的名称”default” 找到Servlet。

  若Servlet名称不是默认名称，使用参数 **default-servlet-name**指定。

  ```text
  常用Web应用服务器的默认Servlet名称：
  - Tomcat, Jetty, JBoss, and GlassFish 	默认 Servlet的名字 – “default”
  - Google App Engine 				   默认 Servlet的名字 – “_ah_default”
  - Resin								  默认 Servlet的名字 – “resin-file”
  - WebLogic 							  默认 Servlet的名字 – “FileServlet”
  - WebSphere 						  默认 Servlet的名字 – “SimpleFileServlet”
  ```

**仅配置`<mvc:default-servlet-handler />`后组件导入情况**

- org.springframework.web.servlet.HandlerMapping
  - SimpleUrlHandlerMapping
  - BeanNameUrlHandlerMapping
- org.springframework.web.servlet.HandlerAdapter
  - HttpRequestHandlerAdapter
  - SimpleControllerHandlerAdapter
- org.springframework.web.servlet.HandlerExceptionResolver
  - AnnotationMethodHandlerExceptionResolver
  - ResponseStatusExceptionResolver
  - DefaultHandlerExceptionResolver

### 6.2.2、 `<mvc:annotation-driven />` 

​	启用注解驱动，支持@RequestMapping注解,这样我们就可以使用@RequestMapping来配置处理器

​	在开启默认处理器后，将默认servlet无法处理的请求（本该有dispatcherServlet处理的请求）交给dispatcherServlet处理 。

​	同时使用 `<mvc:default-servlet-handler />` 和 ``<mvc:annotation-driven />`将静态资源的教由Web应用服务器处理，其他请求交由dispatcherServlet处理。

**仅配置`<mvc:annotation-driven />`后组件导入情况：**

- org.springframework.web.servlet.HandlerMapping
  - RequestMappingHandlerMapping
  - BeanNameUrlHandlerMapping
- org.springframework.web.servlet.HandlerAdapter
  - RequestMappingHandlerAdapter
  - HttpRequestHandlerAdapter
  - SimpleControllerHandlerAdapter
- org.springframework.web.servlet.HandlerExceptionResolver
  - ExceptionHandlerExceptionResolver
  - ResponseStatusExceptionResolver
  - DefaultHandlerExceptionResolver

**同时配置`<mvc:default-servlet-handler />`和`<mvc:annotation-driven />`后：**

- org.springframework.web.servlet.HandlerMapping

  - **RequestMappingHandlerMapping**	：支持@RequestMapping注解
  - BeanNameUrlHandlerMapping ：将controller类的名字映射为请求url
  - SimpleUrlHandlerMapping

- org.springframework.web.servlet.HandlerAdapter

  - **RequestMappingHandlerAdapter** ： 处理@Controller和@RequestMapping注解的处理器 
  - HttpRequestHandlerAdapter ： 处理继承了HttpRequestHandler创建的处理器 
  - SimpleControllerHandlerAdapter ： 处理继承自Controller接口的处理器

- org.springframework.web.servlet.HandlerExceptionResolver

  - ExceptionHandlerExceptionResolver

  - ResponseStatusExceptionResolver

  - DefaultHandlerExceptionResolver

    

**RequestMappingHandlerMapping**：

​	在容器启动时，扫描容器内的bean，解析带有@RequestMapping 注解的方法，并将其解析为url和handlerMethod键值对方式注册到请求映射表中。

### 6.2.3、 `<mvc:resources />` 

`<mvc:default-servlet-handler />`将静态资源交由Web应用服务器处理。

`<mvc:resources />`将静态资源交由Spring  MVC框架处理。

​	默认Web容器只能访问放在Web容器根路径下的静态资源，WEB-INF目录下的资源只有服务器才能访问到,客户端是访问不了的 。

```xml
<mvc:resources location="/resources,/WEB-INF/resources/" mapping="/resources/**" cache-period="31536000"/> 
```

- location：表示webapp目录static包下的所有文件；
- mapping：表示以resources 开头的所有请求路径；
- cache-period：表示静态资源在浏览器端的缓存时间 

作用：**mapping** 配置的请求`/resources/**`不会被当作静态资源请求，不会被 DispatcherServle拦截。

### 6.2.4、`<mvc:view-controller />`

```xml
<mvc:view-controller path="/index" view-name="index"></mvc:view-controller>
<!-- 重定向写法 -->
<mvc:view-controller path="/index" view-name="redirect:index"></mvc:view-controller>
```

将发送的`/index`请求直接跳转到目标页面`index.jsp`，不经过controller方法的处理，等效于如下代码：

```java
@RequestMapping(value="/index")
public String index(){
    return "index";
}

// 重定向写法 
@RequestMapping(value="/index")
public String index(){
    return "redirect:index";
}
```

注意：

- 使用标签`<mvc:view-controller>`前，必须使`<mvc:annotation-driven />`  ,否则会使`@Controller`注解无法解析，导致404错误 ;
- 如果存在请求处理器`@RequestMapping(value="/index")`,则标签`<mvc:view-controller />`对应的请求处理将不起作用 。应为请求先会被找到的处理器处理 ，若找不到则使用`<mvc:view-controller>`标签配置处理。



# 四、Spring MVC 注解

## 4.1、@RequestMapping

作用：处理请求地址映射，可用于类或方法上 。

## 4.2、请求参数相关注解

###4.2.1、@RequestParam

```java
/**
 * 若请求参数中 参数的名称 与 方法形参名 不一致，可使用 @RequestParam 注解:
 * @RequestParam( value = "id", required = true, defaultValue = "001")
 * 	 	@RequestParam(value = "id") 将参数中 id 的值映射为 userId ;
 * 	 	@RequestParam(required = true) 未传参数时，抛异常;
 * 	 	@RequestParam(defaultValue = "001") 未传参数时，设置默认值;
 */
@RequestMapping(value="/findUserByUserId")
public String findUserByUserId(
    	@RequestParam( value = "id", required = true, defaultValue = "001") String userId, 
    	@RequestParam( value = "age", required = false, defaultValue = "18")int age ) {
    return "success";
}
```

###4.2.2、@RequestBody

```java
 /**
  *  用于 将请求body中所有的json数据传到后端 和 指定请求参数是否必须;
  *  用于处理非 Content-Type: application/x-www-form-urlencoded 编码的数据，
  *  如：application/json、application/xml等类 。           
  *  @RequestBody String parameters
  *      若请求参数为：name=tom&age=11
  *          默认情况下，springMVC自动将 name 和 age 的数据的值 tom 和 11 绑定到 方法相同的形参上；
  *          若使用 @RequestBody 注解后，会将整个请求参数 name=tom&age=11 的字符串绑定到方法形参
  *          parameters 上。
  *  参考地址：https://blog.csdn.net/weixin_38004638/article/details/99655322
  */
@RequestMapping(value="/findUser")
public String findUser(@RequestBody String parameters) {
    return "success";
}
```

### 4.2.3、@PathVariable

```java
    /**
     *  @PathVariable： 接收请求路径中占位符的值(3.0后提供)
     *  用法：@PathVariable(value="userId", required = true) String parameters
     *      REST风格URL简介：
     *          新增 - 传统：http://localhost:8080/saveUser               post
     *                REST：http://localhost:8080/user                   post
     *          修改 - 传统：http://localhost:8080/updateUser             post
     *                REST：http://localhost:8080/user/001               put
     *          删除 - 传统：http://localhost:8080/deleteUser?id=001      get
     *                REST：http://localhost:8080/user/001               delete
     *          查询 - 传统：http://localhost:8080/findUser?id=001        get
     *                REST：http://localhost:8080/user/001               get
     *  若不使用 @PathVariable(value="id") 进行 参数id 与 形参userId 的绑定,
     *  则形参无法获取数据(即使形参与参数名称同为 id) 。
     */
    @RequestMapping(value="/user/{id}/{name}",method = {RequestMethod.GET})
    public String user(@PathVariable(value="id") String userId, 
                       @PathVariable(value="name", required = false) String userName) {
        return "success";
    }
```

###4.2.4、@RequestHeader

```java
/**
 *  @RequestHeader： 获取请求头中的数据，通过指定参数 value 的值来获取请求头中指定的参数值
 *  用法：@RequestHeader(value="User-Agent") String userAgent
 */
@RequestMapping(value="testRequestHeader")
public String testRequestHeader(@RequestHeader(value="User-Agent") String userAgent, 
                                @RequestHeader(value="accept-language") String acceptLanguage) {
    return "success";
}
```

###4.2.5、@CookieValue

```java
/**
 *  @CookieValue： 获取请求cookie的数据并映射到方法参数上
 *  用法：@CookieValue(value="JSESSIONID") String sessionId
 */
@RequestMapping(value="testCookie")
public String testCookie(@CookieValue(value="JSESSIONID") String sessionId) {
    return "success";
}
```

###4.2.6、@ModelAttribute

```java
/**
 *  @ModelAttribute 有返回值
 *  用在方法上：
 *      在此controller的每个方法执行前被执行，如果有返回值，则自动将该返回值加入到ModelMap中
 *  用在入参上：
 *      将客户端传递的参数按名称注入到指定对象中，并将此对象自动加入ModelMap中，便于View层使用。
 *  注解属性：
 *      value = "id" : 将请求参数中 id 属性的值 赋值给 入参userId。
 *  参考：https://blog.csdn.net/beidaol/article/details/87086124
 */
@ModelAttribute(value = "id")
public User testModelAttributeBeforeController(String userId ) {
    // 模拟从数据库中查询数据
    User user = new User();
    if(!StringUtils.isEmpty(userId)){
        user.setUserId("001");
        user.setUserName("Tom");
        user.setAge(18);
        return user;
    }
    return user;
}
```



##4.3、参数类型转换相关注解

###4.3.1、@InitBinder

<a href="#md1">@InitBinder用法详解</a>

### 4.4、处理异常相关的注解

参考文档：https://blog.csdn.net/abc997995674/article/details/80454221

###4.4.1、@ExceptionHandler

​		用于局部异常处理 ：只能处理当前方法所在的类的异常。由**ExceptionHandlerExceptionResolver**

```java
// 指定 arithmeticExceptionHandler 方法为 ArithmeticException 异常的处理方法
@ExceptionHandler({ArithmeticException.class})
public String arithmeticExceptionHandler(Exception e){
    // TODO 异常处理
    return "error";
}
```

​	大致流程：产生的 ArithmeticException 异常将被前置处理器捕获，然后交给ExceptionHandlerExceptionResolver进行解析/分派，根据异常类型转交给处理ArithmeticException 异常类型的 arithmeticExceptionHandler 方法处理。

​	@ExceptionHandler注解定义的方法优先级问题：例如发生的是NullPointerException，但是声明的异常有RuntimeException和Exception，这时候会根据异常的最近继承关系找到继承深度最浅的那个@ExceptionHandler注解方法，即标记了RuntimeException 的方法。

###4.4.2、@ControllerAdvice 

​		用于全局异常处理 。由**ExceptionHandlerExceptionResolver**负责解析。

```java
// 标记当前类是为异常处理类
@ControllerAdvice
public class HandleForException {

    @ExceptionHandler({ArithmeticException.class})
    public String testArithmeticException(Exception e){
        // TODO 异常处理
        return "error";
    }
}
```

- 发生异常时，先去异常产生类的找被 @ExceptionHandler  注解的方法 ，若为找到，或无法处理此异常，则去找被 @ControllerAdvice 注解标注的类中的 @ExceptionHandler 注解方法 。
- **被 @ExceptionHandler标记的异常处理方法，不能在方法中设置其它形参。但可以使用ModelAndView向前台传递数据** 

### 4.4.3、@ResponseStatus 

​	自定义异常类。由**ResponseStatusExceptionResolver**负责解析。

```java
// 自定义异常类,标注 @ResponseStatus 
@ResponseStatus(value = HttpStatus.FORBIDDEN,reason = "用户名和密码不匹配!")  
public class UserNameNotMatchPasswordException extends RuntimeException{  

}
```

```java
@RequestMapping("/testResponseStatusExceptionResolver")  
public String testResponseStatusExceptionResolver(@RequestParam("i") int i){  
    if (i==13){  
        // 抛出自定义异常 @1
        throw new UserNameNotMatchPasswordException();  
    }  
    return "success";  
}  
```

- @1处跑的自定义异常，会被ResponseStatusExceptionResolver解析到。最后响应HttpStatus.xx状态码和reason信息给客户端 。
- 若使用@ResponseStatus修饰方法，则所有对该方法的请求都会报错。
- 另外SpringMvc还定义了和UserNameNotMatchPasswordException相似的默认异常处理类，如：`NoSuchRequestHandlingMethodException`、`HttpRequestMethodNotSupportedException`、`HttpMediaTypeNotSupportedException`、`HttpMediaTypeNotAcceptableException`等。只不过这些类会被**DefaultHandlerExceptionResolver**  进行解析。

### 4.4.4、其它异常处理方法：SimpleMappingExceptionResolver

​	用于对所有异常进行统一处理 。它将异常类名映射为视图名，即发生异常时使用对应的视图报告异常 。

- 配置文件

  ```xml
  <!-- 配置使用 SimpleMappingExceptionResolver 来映射异常 -->
   <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
       <!-- 配置异常的属性值为ex，那么在错误页面中可以通过 ${ex} 来获取异常的信息
       如果不配置这个属性，它的默认值为exception -->
       <property name="exceptionAttribute" value="ex"></property>
       <property name="exceptionMappings">
           <props>
               <!-- 映射ArrayIndexOutOfBoundsException异常对应error.jsp这个页面 -->
               <prop key="java.lang.ArrayIndexOutOfBoundsException">error</prop>
           </props>
       </property>
   </bean> 
  
  ```

- 异常示例

  ```java
  
  ```

​        访问此方法会产生ArrayIndexOutOfBoundsException异常，而上面配置了发生这个异常由error页面处理,并且将异常信息通过 ex 进行传递，可在error.jsp页面使用 ${ex}  获取错误信息。





在web.xml中配置核心控制器dispatcherServlet ，拦截请求 ；

在springmvc配置文件中配置包扫描，处理请求；

在springmvc配置文件中配置页面解析器，根据处理器的返回值找到资源页面,然后装配页面数据 ；



赵天冲;计雪峰;董冬明; 

peter.chai@oracle.com





