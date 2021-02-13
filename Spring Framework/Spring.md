# Spring简介

1. Spring核心概念

   - 控制反转（IOC）

     对象的创建和组装由spring容器完成，使用者只需要去spring容器中查找需要的对象。

     IOC降低了系统代码的耦合度，让系统利于维护和扩展 。  

   - 依赖注入（DI）

     依赖注入是spring容器在创建对象时给其设置依赖对象的方式。

   - 面向切面编程（AOP）

2. Spring容器

   负责容器中对象的创建、组装、对象查找、对象生命周期的管理等操作。   

3. Spring容器的使用

   1. 引入spring相关的maven配置 ；
   2. 创建bean配置文件，比如bean.xml、Config类；
   3. 在bean.xml文件中定义要spring容器管理的bean对象 ；
   4. 创建spring容器，指定容器需要装载的bean配置文件。spring容器启动后，会加载配置文件，并创建好配置文件中定义的bean对象，将其放在容器中；
   5. 通过容器提供的方法获取容器中的对象，使用  。

4. Spring 容器对象

   1. BeanFactory接口  

      spring容器的顶层接口，提供了容器最 基本的功能  。

      ```java
      // org.springframework.beans.factory.BeanFactory
      
      //按bean的id或者别名查找容器中的bean
      Object getBean(String name) throws BeansException
      //这个是一个泛型方法，按照bean的id或者别名查找指定类型的bean，返回指定类型的bean对象
      <T> T getBean(String name, Class<T> requiredType) throws BeansException;
      //返回容器中指定类型的bean对象
      <T> T getBean(Class<T> requiredType) throws BeansException;
      //获取指定类型bean对象的获取器，这个方法比较特别，以后会专门来讲
      <T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
      ```

      

Spring