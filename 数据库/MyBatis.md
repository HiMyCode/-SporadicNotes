参考资料：

​	https://www.w3cschool.cn/mybatis/mybatis-dyr53b5w.html

[toc]





# 一、MyBatis入门案例

## 1.1、原生版本

1. 导入mybatis依赖

   ```xml
   <dependency>
     <groupId>org.mybatis</groupId>
     <artifactId>mybatis</artifactId>
     <version>x.x.x</version>
   </dependency>
   ```

2. 新建表、实体类、mapper映射文件

   - 表（省略）
   - 实体类（省略）
   - mapper映射文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
     "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="org.mybatis.example.BlogMapper">
     <select id="selectBlog" resultType="Blog">
       select * from Blog where id = #{id}
     </select>
   </mapper>
   ```

3. 配置用于构建 SqlSessionFactory的mybatis全局配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
     "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
     <!-- 1.数据库配置 -->
     <environments default="development">
       <environment id="development">
         <transactionManager type="JDBC"/>
         <dataSource type="POOLED">
           <property name="driver" value="${driver}"/>
           <property name="url" value="${url}"/>
           <property name="username" value="${username}"/>
           <property name="password" value="${password}"/>
         </dataSource>
       </environment>
     </environments>
     <!-- 2.mapper映射文件 -->
     <mappers>
       <mapper resource="org/mybatis/example/BlogMapper.xml"/>
     </mappers>
   </configuration>
   ```

4. 构建SqlSessionFactory，调用mapper中的SQL语句

   ```java
   public void test() throws IOException{
     	String resource = "org/mybatis/example/mybatis-config.xml";
   		InputStream inputStream = Resources.getResourceAsStream(resource);
   		SqlSessionFactory sqlSessionFactory = 
         						new SqlSessionFactoryBuilder().build(inputStream);
       // SqlSession 代表与数据库的一次会话，非线程安全。每次使用前需获取，使用后需关闭。
       // sqlSessionFactory.openSession() ： 需要手动提交事务
       // sqlSessionFactory.openSession(true) ： 自动提交事务
       try (SqlSession session = sqlSessionFactory.openSession()) {
     			Blog blog = 
           (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
   		}
   } 
   ```

## 1.2、接口版本（常用）

1. 导入mybatis依赖（同《1.1、原生版本》）

2. 新建表、实体类、mapper映射文件（同《1.1、原生版本》）

3. 配置用于构建 SqlSessionFactory的mybatis全局配置文件（同《1.1、原生版本》）

4. 创建用于与mapper文件动态绑定的接口

   ```java
   package org.mybatis.example.BlogMapper;
   // 绑定前提条件一：接口类的<类路径名>与<mapper文件的namespace>一致
   public interface BlogMapper{
     // 绑定前提条件二：接口类的<方法名>与<mapper文件中的Id>一致
     public Blog getBlogById();
   }
   ```

   Mapper接口不需要实现类，mybatis会为此接口生成代理对象。

5. 构建SqlSessionFactory，调用接口类的方法

   ```java
   public void test() throws IOException{
     	String resource = "org/mybatis/example/mybatis-config.xml";
   		InputStream inputStream = Resources.getResourceAsStream(resource);
   		SqlSessionFactory sqlSessionFactory = 
         						new SqlSessionFactoryBuilder().build(inputStream);
       try (SqlSession session = sqlSessionFactory.openSession()) {
     			BlogMapper mapper = session.getMapper(BlogMapper.class);
     			Blog blog = mapper.getBlogById(101);
   		}
   } 
   ```

## 1.3、接口无Mapper版本

1. 导入mybatis依赖（同《1.1、原生版本》）

2. 新建表、实体类（同《1.1、原生版本》，但你不需要mapper文件）

3. 配置用于构建 SqlSessionFactory的mybatis全局配置文件（同《1.1、原生版本》）

4. 创建BlogMapper映射器类

   ```java
   package org.mybatis.example.BlogMapper;
   
   public interface BlogMapper {
     @Select("SELECT * FROM blog WHERE id = #{id}")
     Blog selectBlog(int id);
   }
   ```

   Mapper接口不需要实现类，mybatis会为此接口生成代理对象。

5. 构建SqlSessionFactory，调用接口类的方法（同《1.2、接口版本（常用）》

# 二、MyBatis配置文件详解

## 2.1、MyBatis全局配置文件

1. org/mybatis/example/dbconfig.properties

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/plan_springmvc
```

2. Mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  
  <!-- properties标签：用于引入外部配置文件 或 定义文件属性值。 
			 优先级：方法参数属性 > resource 文件属性 > properties 元素体内的属性。
	-->
  <properties resource="org/mybatis/example/dbconfig.properties">
  		<property name="username" value="dev_user"/>
 	 		<property name="password" value="F2Fa3!33TYyg"/>
	</properties>
  
  <!-- settings标签：用于指定MyBatis 运行时行为 -->
  <settings>
      <setting name="cacheEnabled" value="true"/>
      <setting name="lazyLoadingEnabled" value="true"/>
      <setting name="multipleResultSetsEnabled" value="true"/>
      <setting name="useColumnLabel" value="true"/>
      <setting name="useGeneratedKeys" value="false"/>
      <setting name="autoMappingBehavior" value="PARTIAL"/>
      <setting name="defaultExecutorType" value="SIMPLE"/>
      <setting name="defaultStatementTimeout" value="25"/>
      <setting name="safeRowBoundsEnabled" value="false"/>
      <!-- 是否开启驼峰命名规则映射。默认：false -->
      <setting name="mapUnderscoreToCamelCase" value="false"/>
      <setting name="localCacheScope" value="SESSION"/>
      <setting name="jdbcTypeForNull" value="OTHER"/>
      <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
	</settings>
  
  <!-- typeAliases标签：为常用的java类型起别名，用来减少类完全限定名的冗余 -->
  <!-- 别名冲突时，可以用  @Alias("author") 注解为类单独指定别名 -->
  <typeAliases>
      <typeAlias alias="Author" type="domain.blog.Author"/>
      <typeAlias alias="Blog" type="domain.blog.Blog"/>
      <typeAlias alias="Comment" type="domain.blog.Comment"/>
      <typeAlias alias="Post" type="domain.blog.Post"/>
      <typeAlias alias="Section" type="domain.blog.Section"/>
      <typeAlias alias="Tag" type="domain.blog.Tag"/>
    	<!-- 为domain.blog及其子包下的类设置别名，别名为类名首字母小写，但别名不区分大小写 -->
    	<!-- <package name="domain.blog"/> -->
  </typeAliases>
  
  <!-- typeHandlers标签：类型处理器 -->
  <typeHandlers>
    	
  </typeHandlers>
  
  <!-- interceptor标签：插件标签 -->
  <plugins>
      <plugin interceptor="org.mybatis.example.ExamplePlugin">
        <property name="someProperty" value="100"/>
      </plugin>
  </plugins> 
  
  <!-- environments标签：环境标签 -->
  <environments default="dev_mysql">
      <environment id="dev_mysql">
          <transactionManager type="JDBC"/>
          <dataSource type="POOLED">
            <property name="driver" value="${driver}"/>
            <property name="url" value="${url}"/>
            <property name="username" value="${username}"/>
            <property name="password" value="${password}"/>
          </dataSource>
      </environment>
    <environment id="dev_oracle">
          <transactionManager type="JDBC"/>
          <dataSource type="POOLED">
            <property name="driver" value="${driver}"/>
            <property name="url" value="${url}"/>
            <property name="username" value="${username}"/>
            <property name="password" value="${password}"/>
          </dataSource>
      </environment>
  </environments>
  
  <!-- databaseIdProvider标签：用于支持多数据库厂商 -->
  <databaseIdProvider type="DB_VENDOR">
    	 <!-- 为不同的厂商数据库起别名，可在sql标签中的 databaseId 属性中使用-->
      <property name="SQL Server" value="sqlserver"/>
      <property name="DB2" value="db2"/>        
      <property name="Oracle" value="oracle" />
    	<property name="Mysql" value="mysql" />
  </databaseIdProvider>
  
  <!-- 2.mapper映射文件 -->
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```



## 2.2、Mapper映射文件

### 2.2.1、CRUD

- Select 

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select> 
```

- Insert && Update && Delete

```xml
<insert id="insertAuthor">
  insert into Author (id,username,password,email,bio)
  values (#{id},#{username},#{password},#{email},#{bio})
</insert>

<update id="updateAuthor">
  update Author set username = #{username}, password = #{password}, email = #{email}, bio = #{bio} where id = #{id}
</update>

<delete id="deleteAuthor">
  delete from Author where id = #{id}
</delete>
```

对于Insert && Update && Delete，mybatis允许其有如下的返回值类型：**Integer**、**Long**、**Boolean**、**void** 。

### 2.2.2、获取自增key

#### 2.2.2.1、Mysql获取自增key

```xml
<!-- parameterType : 此参数可省略；
 		 useGeneratedKeys="true" : 使用自增主键获取主键值策略，底层原理：statement.getGeneratedKeys();
		 keyProperty="id" : 指定主键对应属性，用于将获取到的主键值存储到javaBean对象中指定的属性。
		 databaseId="mysql" : 指定databaseId,需配置databaseIdProvider属性。
-->
<insert id="insertAuthor" parameterType="com.demo.vo.Author" 
        useGeneratedKeys="true" keyProperty="id" databaseId="mysql">
  insert into Author (id,username,password) values (#{id},#{username},#{password})
</insert>
```

#### 2.2.2.2、Oracle获取自增key

由于Oracle不支持自增，因此使用Oracle序列来实现。

- 在insert语句运行前获取自增主键key

```xml
<insert id="insertAuthor" parameterType="com.demo.vo.Author" databaseId="oracle">
  <!-- keyProperty="id" : 指定主键对应属性，用于将获取到的主键值存储到javaBean对象中指定的属性；
			 order="before" : 在insert语句运行前获取自增主键key;
			 resultType="Integer" : 查出主键的返回值类型
	-->
  <selectKey keyProperty="id" order="before" resultType="Integer">
    	select AUTHOR_SEQ.nextval from dual
  </selectKey>
  insert into Author (id,username,password) values (#{id},#{username},#{password})
</insert>
```

- 在insert语句运行后获取自增主键key

```xml
<insert id="insertAuthor" parameterType="com.demo.vo.Author" databaseId="oracle">
  <selectKey keyProperty="id" order="after">
    	select AUTHOR_SEQ.currval from dual
  </selectKey>
  insert into Author (id,username,password) 
  values (AUTHOR_SEQ.nextval,#{username},#{password})
</insert>
```

注意After存在如下问题：

​	当批量插入时，返回的自增主键key获取到的是最后一次插入时的主键。

### 2.2.3、mapper参数映射

#### 2.2.3.1、参数映射原理

```java
public interface BlogMapper {
  Blog selectBlogById(@Param("id") int id, String name);
}
```

ParamNameResolver解析参数封装Map过程：

1. 将传入的参数封装为数组：params **[ 001, "tom"]**

2. 遍历方法获取参数的names的Map集合。

​	key 为参数索引；

​	value 为值： 

​			若参数标注@Param("")注解，则value为@Param("")中的值；

​			若全局配置中isUseActualParamName(jdk1.8后)配置为true，则value为参数名；

​			否则，value为map.size()。

结果示例：map **{ 0=id, 1=1 }**

3. 循环map：

   ​	获取map的value，作为参数paramMap的key，

   ​	通过map的key作为数组索引从params中获取参数值，作为paramMap的value

   ​	结果示例：paramMap **{ id = 001, 1 = tom}**

4. 循环params为参数值取名param1 、param2 ... 添加至paramMap：

   ​	结果示例：paramMap **{ id = 001, 1 = tom, param1 = 001, param2 = tom}**

#### 2.2.3.2、简单参数

1. **单个简单参数**

- 参数前无注解	

  mybatis不会做任何特殊处理。可以在mapper文件中使用任何合法字符串获取变量。

  ```java
  public interface BlogMapper {
    Blog selectBlogById(int id);
  }
  ```

  ```xml
  <!-- #{id} 也可以换成 #{value} #{mm} 等任何合法字符串获取变量均可取出参数值 -->
  <select id="selectBlogById" parameterType="int" resultType="hashMap">
    SELECT * FROM PERSON WHERE ID = #{id}
  </select> 
  ```

- 参数前有注解

  ​	mybatis会做特殊处理：将多个参数封装成一个map，#{}就是通过map的key获取变量的value值。

  ```java
  public interface BlogMapper {
    Blog selectBlogById(@Param("id") int id);
  }
  ```

  ```xml
  <!-- 可以通过 #{id} 或 #{param1} 获取 -->
  <select id="selectBlogById" parameterType="int" resultType="hashMap">
    SELECT * FROM PERSON WHERE ID = #{id}
  </select> 
  ```

2. **多个简单参数**

   mybatis会做特殊处理：将多个参数封装成一个map，#{}就是通过map的key获取变量的value值。

- 未明确指定封装参数时Map的key

  ```java
  public interface BlogMapper {
    Blog selectBlogById(int id, String name);
  }
  ```

  错误获取参数方式：

  ​	将抛异常： Parameter 'id' not found. Avaliable parameters are [1, 0, param1, param2].

  ```xml
  <select id="selectBlogById" parameterType="int" resultType="hashMap">
    SELECT * FROM PERSON WHERE ID = #{id} and NAME = #{name}
  </select> 
  ```

  正确获取参数方式一：使用默认的参数索引 0、1 ...

  ```xml
  <select id="selectBlogById" parameterType="int" resultType="hashMap">
    SELECT * FROM PERSON WHERE ID = #{0} and NAME = #{1}
  </select> 
  ```

  正确获取参数方式二：使用默认的key param1、param2 ...

  ```xml
  <select id="selectBlogById" parameterType="int" resultType="hashMap">
    SELECT * FROM PERSON WHERE ID = #{param1} and NAME = #{param2}
  </select> 
  ```

- 明确指定封装参数时Map的key

  ```java
  public interface BlogMapper {
    // 此时封装参数Map时将使用@Param()指定的数据作为key
    Blog selectBlogById(@Param("id") int id, @Param("name") String name);
  }
  ```

  ```xml
  <select id="selectBlogById" parameterType="int" resultType="hashMap">
    SELECT * FROM PERSON WHERE ID = #{id} and NAME = #{name }
  </select> 
  ```

#### 2.2.3.3、对象参数

​		对于多个简单参数，可以将其封装为Pojo或者Map对象。

- Pojo对象

  ```java
  public interface BlogMapper {
    Blog selectBlogById(Person person);
  }
  ```

  ```xml
  <!-- 通过 #{属性值} 获取 Person 对象中的属性值-->
  <select id="selectBlogById" parameterType="int" resultType="hashMap">
    SELECT * FROM PERSON WHERE ID = #{id} and NAME = #{name }
  </select> 
  ```

- Map对象

  ```java
  /**
   *	Map<String, Object> map = new HashMap<>();
   *	map.put("id", 1);
   *  map.put("name", "tom");
   **/
  public interface BlogMapper {
    Blog selectBlogById(Map<String, Object> map);
  }
  ```

  ```xml
  <!-- 通过 #{key} 获取 Map 对象中的属性值-->
  <select id="selectBlogById" parameterType="int" resultType="hashMap">
    SELECT * FROM PERSON WHERE ID = #{id} and NAME = #{name }
  </select> 
  ```

#### 2.2.3.4、Collection（ list、set）、Array对象参数

​	mybatis不会做任何特殊处理。将传入的list获取数据封装在map中，且map中默认的key也有所区别：

		1. Collection对应的key ： collection
		2. List对应的key ： list
		3. Array对应的key ： array

- Collection获取传参方式

  ```java
  <select id="selectBlogById" parameterType="int" resultType="hashMap">
    SELECT * FROM PERSON WHERE ID = #{collection[0]} 
  </select> 
  ```

- List获取传参方式

  ```xml
  <select id="selectBlogById" parameterType="int" resultType="hashMap">
    SELECT * FROM PERSON WHERE ID = #{list[0]} 
  </select> 
  ```

- Array获取传参方式

  ```xml
  <select id="selectBlogById" parameterType="int" resultType="hashMap">
    SELECT * FROM PERSON WHERE ID = #{array[0]} 
  </select> 
  ```

#### 2.2.3.5、其他特殊情况下的参数取值

- 情况一：

  ```java
  public interface BlogMapper {
    Blog selectBlogById(@Param("id") int id, String name);
  }
  ```

  此时获取参数id的方式：#{id} 或 #{param1}

   获取参数name的方式：#{param2}

- 情况二：

  ```java
  public interface BlogMapper {
    Blog selectBlogById(int id, @Param("person") Person person);
  }
  ```

  此时获取参数id的方式： #{param1}

  获取参数person的name方式：#{person.name}  或 #{param2.name} 

### 2.2.4、参数获取方式

- #{}

  #{}表示一个占位符号，通过预编译的方式将参数设置到sql中。可有效方式SQL注入。

  **#{}相关使用参数：**

  - javaType

  - jdbcType

    数据为null时，oralce不识别mybatis对null的处理，会抛异常无效的jdbcType OTHER 异常。

    解放方法：

    - 指定参数的jdbcType：#{param1, jdbcType=OTHER}
    - 在全局配置文件中指定参数：`<setting name="jdbcTypeForNull" value="Null" />`

  - mode

  - numericScale

  - resultMap

  - typeHandler

  - jdbcTypeName

  - expression

  

- ${}

  #{}表示SQL拼接，直接取出参数值拼接到sql中。存在SQL注入问题。

  使用场景：

  ​	在原生jdbc不支持占位符的地方就可以使用${}进行取值，如字段名、表名。 

### 2.2.5、resultType返回值类型（select）

​	为实现查询sql查询结果集与restType返回对象字段的自动映射，需进行setting设置：

```xml
<settings>
  	<!-- 默认值为PARTINA，开启自动映射功能。要求列名与java属性名一致 -->
		<setting name="autoMappingBehavior" value="PARTINA" />
  
  	<!-- 若数据库字段规范命名方式：A_COLUMN, POJO使用驼峰命名：aColumn，则需开启自动驼峰命名规则映射 -->
		<setting name="mapUnderscoreToCamelCase" value="true"/>
</settings> 
```

#### 2.2.5.1、简单数据类型

```java
String getEmpNameById(Integer id);
```

```xml
<!-- string 在这里是mybatis默认定义的 java.lang.String 的别名 -->
<select id="getEmpNameById" resultType="string">
  	select username from t_employee where id = #{id}
</select>
```

#### 2.2.5.2、java对象

```java
Employee getEmpById(Integer id);
```

```xml
<!-- 通过 resultType 指定查询的结果是 Employee 类型的数据 -->
<select id="getEmpById" resultType="com.example.vo.Employee">
  	select * from t_employee where id = #{id}
</select>
```

#### 2.2.5.3、List

```java
 List<Employee> getAllEmps();
```

```xml
<!-- 通过 resultType 指定的是查询结果集合内存储数据的JavaBean类型为Employee -->
<select id="getEmpById" resultType="com.example.vo.Employee">
  	select * from t_employee where id = #{id}
</select>
```

#### 2.2.5.4、Map

- Map结果数据格式：{表字段名， 对应的值}

  ```java
  Map<String, Object> getEmpAsMapById(Integer id);
  ```

  ```xml
  <select id="getEmpAsMapById" resultType="map">
    select * from t_employee where id = #{id}
  </select>
  ```

- Map结果数据格式：{表中某一字段名, JavaBean}

  ```java
  // 查询所有员工的信息，把数据库中的 'id' 字段作为 key, 对应的 value 封装成 Employee 对象
  // @MapKey 中的值表示用数据库中的哪个字段名作 key
  @MapKey("id")
  Map<Integer, Employee> getAllEmpsAsMap();
  ```

  ```java
  <!-- 注意 resultType 指定的是 Map 的 value 对应的 JavaBean 类型 -->
  <select id="getAllEmpsAsMap" resultType="employee">
    	select * from t_employee
  </select>
  ```

### 2.2.6、resultMap返回值类型（select）

​		若resultType自动映射无法满足业务需求，可使用resultMap自定义自动映射规则。

#### 2.2.6.1、简单VO映射

```xml
<resultMap id="emps" type="com.example.vo.Employee" >
  	<!-- 其他不指定的列将不会自动封装 -->
    <id column="id" property="id" />
    <result column="last_name" property="lastName" />
    <result column="email" property="email" />
    <result column="gender" property="gender"/>
</resultMap>

<select id="getResultMap"  resultMap="emps" databaseId="mysql"> 
  	select last_name, email,id,gender from employee where id = #{id} 
</select> 
```

#### 2.2.6.2、VO含VO属性映射

- property映射

  ```java
  <resultMap id="emps" type="com.example.vo.Employee" >
      <id column="id" property="id" />
      <result column="last_name" property="lastName" />
      <result column="email" property="email" />
      <result column="gender" property="gender"/>
      <result column="id" property="dept.id"/>
      <result column="dept_name" property="dept.departmentName" />
  </resultMap>
  
  <select id="getEmpAndDep" resultMap="emps" databaseId="mysql"> 
     	select * from employee e,department d where e.d_id=d.id and e.id=#{id}
  </select> 
  ```

- association映射（一次性查询）

  ```xml
  <resultMap type="com.example.vo.Employee" id="emps">
      <id column="id" property="id" />
      <result column="last_name" property="lastName" />
      <result column="email" property="email" />
      <result column="gender" property="gender"/>
      <!-- property 属性名 -->
      <association property="dept" javaType="com.example.vo.Department">
          <id column="did" property="id" />
          <result column="dept_name" property="departmentName" />
      </association>
  </resultMap>
  
  <select id="getEmpAndDep"  resultMap="emps" databaseId="mysql"> 
     select e.last_name last_name, e.email email, e.id,e.gender gender,e.d_id ,d.id did,
    			  d.dept_name dept_name 
    	 from employee e,department d 
      where e.d_id=d.id and e.id=#{id}
  </select> 
  ```

- association映射（分步性查询）

  ```xml
  <resultMap type="com.example.vo.Employee" id="emps">
      <id column="id" property="id" />
      <result column="last_name" property="lastName" />
      <result column="email" property="email" />
      <result column="gender" property="gender"/>
      <association property="departMent" 
       						 select="com.dao.DepartMentMapper.getDeptById" column="d_id">
      </association>
  </resultMap>
  
  <select id="getEmpAndDep"  resultMap="emps" databaseId="mysql"> 
     select last_name,email,id,gender,d_id from employee e where id=#{id}
  </select> 
  ```

  ```xml
  <mapper namespace="com.dao.DepartMentMapper">
    	<select id="getDeptById" resultType="com.example.vo.DepartMent" databaseId="mysql"> 
         select e.* from DepartMent e where id=#{id}
      </select> 
  </mapper>
  ```

- association映射（分步性查询配置延迟性加载功能）

  在全局配置文件中新增配置：

  ```xml
  <settings>
    	<!-- lazyLoadingEnabled：是否开启延迟加载。默认false，不开启 -->
  		<setting name="lazyLoadingEnabled" value="true"/>
    	<!-- aggressiveLazyLoading：侵入懒加载。指当需要某个对象的级联属性时，其它属性也会加载，
  				 设置为false后只加载当前属性。 -->
      <setting name="aggressiveLazyLoading" value="false"/>
  </settings> 
  ```

  指定association为延迟加载：

  ```xml
  <!-- 默认开启延迟加载 fetchType="lazy"，若fetchType="eager"，则表示立即加载，即使开启了延迟加载配置-->
  <association property="departMent" fetchType="lazy"
               select="com.dao.DepartMentMapper.getDeptById" column="d_id">
  </association>
  ```

#### 2.2.6.3、VO含集合属性映射

- collection嵌套查询

```xml
<resultMap type="com.example.vo.Department" id="deps">
    <id column="did" property="id"/>
    <result column="dept_name" property="departmentName"/>
    <collection property="emps" ofType="com.example.vo.Employee">
        <id column="eid" property="id"/>
        <result column="last_name" property="lastName" />
        <result column="email" property="email" />
        <result column="gender" property="gender"/>
    </collection>
 </resultMap>

<select id="getDepartMentBylist"  resultMap="depls" databaseId="mysql"> 
     select 	d.id did,d.dept_name,e.id eid,e.last_name,e.email,e.gender 
       from 	department d 
  left join 	employee e  
         on   d.id=e.d_id 
  		where   d.id=#{id}
</select> 
```

- collection嵌套查询（分步）

```xml
<resultMap type="dep" id="depl">
    <id column="id" property="id"/>
    <result column="dept_name" property="departmentName"/>
    <!-- 开启延迟加载 fetchType="lazy"，若fetchType="eager"，表示立即加载，即使开启了延迟加载配置-->
    <collection property="emps" fetchType="lazy"
                select="com.dao.EmployeeMapper.getDeptByDid" column="id">
  	</collection>
 </resultMap>
```

```xml
<select id="getDeptByDid" resultMap="depl">
  	select id, dept_name from department where id=#{id}
</select>
```

**扩展：**`<association>` 和 `<collection>`标签column字段传递多个参数：

```xml
<association property="departMent" select="com.dao.DepartMentMapper.getDeptById"
              column="{d_id=d_id,d_name=d_name}">
</association>

<collection property="emps" fetchType="lazy" select="com.dao.EmployeeMapper.getDeptByDid"
             column="{id=dept_id,name=dept_name">
</collection>
```

### 2.2.7、mybatis内置参数

- _parameter : 代表整个参数

  - 单个参数：_parameter就是这个参数

  - 多个参数：由于参数会被封装为map，所以_parameter是这个map

- _databaseId：如果配置了databaseIdProviderbiao标签

  ​	_databaseId就代表当前数据库的别名。

  ```xml
  <databaseIdProvider type="DB_VENDOR">
        <property name="Oracle" value="oracle"/>
        <property name="MySQL" value="mysql"/>
        <property name="DB2" value="d2"/>
  </databaseIdProvider>
  ```

### 2.2.8、\<bind>参数绑定标签

可在\<bind>中将OGNL表达式的值绑定到一个变量中，并在后面的SQL中引用。

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  <bind name="_lastName" value="'%' + #{lastName} + '%'"></bind>
  SELECT * FROM PERSON WHERE last_name like #{lastName}
</select> 
```

### 2.2.9、\<sql>标签

```xml
<!-- 抽取SQL片段 -->
<sql id="query_user_where">
	<if test="id!=null and id!=''">
		and id=#{id}
	</if>
	<if test="username!=null and username!=''">
		and username like '%${username}%'
	</if>
</sql>
 
<select id="findUserList" parameterType="user" resultType="user">
	select * from user
		<where>
			<include refid="query_user_where"/> <!-- 使用SQL片段 -->
		</where>
</select>
 
<!-- 引用其它mapper.xml的sql片段 -->
<include refid="namespace.sql片段"/>
```



### 2.2.10、discriminator 监视器属性（select）

​	discriminator	会根据某个字段值的情况，动态选择不同的映射。

```xml
<resultMap type="emp" id="empd">
    <id column="id" property="id" />
    <result column="last_name" property="lastName" />
    <result column="email" property="email" />
    <result column="gender" property="gender"/>

    <discriminator javaType="int" column="gender">
        <!-- resultType注明返回的类型，不能缺少 -->
        <case value="1" resultType="emp">
            <association property="departMent" javaType="dep">
                <id column="did" property="id" />
                <result column="dept_name" property="departmentName" />
            </association>
        </case>
        <case value="0" resultType="emp">
            <id column="id" property="id" />
            <result column="email" property="lastName" />
            <result column="email" property="email" />
            <result column="gender" property="gender"/>
        </case>
    </discriminator>
</resultMap>
```



# 三、MyBatis动态SQL

## 3.1、if标签

可使用OGNL表达式进行判断。

```xml
<select id="select" resultType="Blog">
  SELECT * FROM BLOG
  WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
<if test="name!= null">
    AND name like #{title}
  </if>
</select>
```

## 3.2、where标签

用于将查询条件包含在内，会自动去除第一个多出来的 and 或or 关键字。

```xml
<select id="select" resultType="Blog">
  SELECT * FROM BLOG
  <where>
    	<!-- AND符号必须写在字段前 -->
      <if test="title != null">
        AND title like #{title}
      </if>
      <if test="name!= null">
        AND name like #{title}
      </if>
  <where>
</select>
```

## 3.3、choose标签（when、otherwise） 

```xml
<select id="findActiveBlogLike"resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select> 
```

## 3.4、trim标签

用于向trim标签包含的sql片段前后添加/覆盖字符串。

```xml
select * from test
<!-- prefix="WHERE" : 给标签内拼接好的字符串拼接 WHERE 前缀；
		 prefixoverride="AND丨OR" ： 去掉标签内接好的字符串前多余的 AND 或 OR 前缀；
		 suffix="AND" : 给标签内拼接好的字符串拼接 WHERE 后缀；
		 prefixoverride="AND丨OR" ： 去掉标签内接好的字符串前多余的 AND 或 OR 后缀；
-->
<trim prefix="WHERE" prefixoverride="AND丨OR">
      <if test="a!=null and a!=' '">AND a=#{a}<if>
      <if test="b!=null and b!=' '">AND a=#{a}<if>
</trim>
```

3.5、set标签  

set 元素用在更新操作，如果包含语句以逗号结束，逗号将会被忽略(也可使用trim)。

若set包含内容为空则会出错。

```xml
<update id="dynamicSetTest" parameterType="Blog">  
    update t_blog  
    <set>  
        <if test="title != null">  
            title = #{title},  
        </if>  
        <if test="content != null">  
            content = #{content},  
        </if>  
        <if test="owner != null">  
            owner = #{owner}  
        </if>  
    </set>  
    where id = #{id}  
</update> 
```

## 3.5、foreach标签

### 3.5.1、用于构建in条件

```xml
<select id="dynamicForeachTest" resultType="Blog">  
        select * from t_blog where id in  
        <foreach collection="list" index="index" item="item" open="(" separator="," close=")">  
            #{item}  
        </foreach>  
    </select>  
```

```xml
<select id="dynamicForeach3Test" resultType="Blog">  
        select * from t_blog where title like "%"#{title}"%" and id in  
        <foreach collection="ids" index="index" item="item" open="(" separator="," close=")">  
            #{item}  
        </foreach>  
</select>  
```

**index="index" item="item" :**

- 遍历list时：

  index是索引；item是当前值。

- 遍历map时： 

  index是map中的key；item是map中的value。

### 3.5.2、用于批量插入

#### 3.5.2.1、Mysql版本

- 方式一：

mysql支持语法： **inset into table (*) values (), (), ()**

```xml
<insert id="insertAuthor">
  insert into Author (id,username,password,email,bio) values
  <foreach collection="emps" item="emp" separator="," >
    	 (#{emp.id},#{emp.username},#{emp.password},#{emp.email},#{emp.bio})
  </foreach>
</insert>
```

- 方式二：

需要设置参数：allowMultiQueries=true ；设置在一条语句中允许使用 ；来分隔多条语句。

```xml
<insert id="insertAuthor">
  <foreach collection="emps" item="emp" separator=";" >
    	insert into Author (id,username,password,email,bio) values
    	(#{emp.id},#{emp.username},#{emp.password},#{emp.email},#{emp.bio})
  </foreach>
</insert>
```

#### 3.5.2.2、Oracle版本

Oracle不支持语法： **inset into table (*) values (), (), ()**

- 方式一：使用 begin / end 语法

```sql
begin
  insert into Author (id,username,password,email,bio) values
  (emp_seq.nextval,#{emp.username},#{emp.password},#{emp.email},#{emp.bio});
  insert into Author (id,username,password,email,bio) values
  (emp_seq.nextval,#{emp.username},#{emp.password},#{emp.email},#{emp.bio});
end;
```

```xml
<insert id="insertAuthor">
  <!-- 注意两个分号不能省 -->
  <foreach collection="emps" item="emp" separator=";" open="begin" close="end;">
    	insert into Author (id,username,password,email,bio) values
    	(emp_seq.nextval,#{emp.username},#{emp.password},#{emp.email},#{emp.bio});
  </foreach>
</insert>
```

- 方式二：使用 insert into select 语法

```sql
insert into Author (id,username,password,) 
		select id,username,password from (
      	select '1' id, 'tom' username, 'tom' password from dual
      	union 
      	select '2' id, 'ted' username, 'ted' password from dual
      	union 
      	select '3' id, 'lucy' username, 'lucy' password from dual
    )
```

```xml
<insert id="insertAuthor">
  <!-- 注意两个分号不能省 -->
  insert into Author (id,username,password,) 
		select id,username,password from (
  <foreach collection="emps" item="emp" separator="union" >
    	select emp_seq.nextval, #{emp.username},#{emp.password} from dual
  </foreach>
</insert>
```

# 四、Mybatis其他使用

## 4.1、PageHelper分页

参考：https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/en/HowToUse.md

1. 导入jar

   ```xml
   <dependency>
     <groupId>com.github.pagehelper</groupId>
     <artifactId>pagehelper</artifactId>
     <version>5.2.0</version>
   </dependency>
   <dependency>
     <groupId>com.github.jsqlparser</groupId>
     <artifactId>jsqlparser</artifactId>
     <version>3.2</version>
   </dependency>
   ```

2. 配置分页插件

   方式一：在MyBatis全局配置文件中配置

   ```xml
    <plugins>
      <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <!-- <property name="offsetAsPageNum" value="true"/>-->
        <!-- <property name="helperDialect" value="mysql"/>-->
        <property name="helperDialect" value="mysql"/>
        <!-- <property name="rowBoundsWithCount" value="true"/>-->
        <property name="reasonable" value="false"/>
      </plugin>
   </plugins>
   ```

   方式二：在spring配置文件中配置

   ```xml
   <bean id="sqlSessionFactory" name="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
           <property name="dataSource" ref="dataSource"></property>
           <!-- 指定mybatis全局配置文件路径 -->
           <property name="configLocation" value="classpath:mybatis/mybatis-config.xml"></property>
           <!-- 指定mapper配置文件路径 -->
           <property name="mapperLocations">
               <list> <value>classpath*:mybatis/mapper/*.xml</value> </list>
           </property>
           <!-- 配置Mybatis插件 -->
           <property name="plugins">
               <array>
                   <!-- 配置PageHelper分页插件（方式二） -->
                   <bean class="com.github.pagehelper.PageInterceptor">
                       <property name="properties">
                           <value>helperDialect=mysql</value>
                           <value>reasonable=reasonable=false</value>
                       </property>
                   </bean>
               </array>
           </property>
       </bean>
   ```

3. 使用分页查询

   ```java
   		SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
   		SqlSession openSession = sqlSessionFactory.openSession();
   		try {
   			EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
         // 使用PageHelper分页
   			Page<Object> page = PageHelper.startPage(5, 1);
   			// 查询数据集合
   			List<Employee> emps = mapper.getEmps();
   			// 使用PageInfo封装返回结果
   			PageInfo<Employee> info = new PageInfo<>(emps, 5);
   			// 获取 当前页码 info.getPageNum()
         // 获取 总记录数 info.getTotal()
         // 获取 每页的记录数 info.getPageSize()
         // 获取 总页码 info.getPages()
         // 获取 是否第一页 info.isIsFirstPage()
         // 获取 连续显示的页码 info.getNavigatepageNums()
   		} finally {
   			openSession.close();
   		}
   ```

## 4.2、调用存储过程

1. 编写存储过程

   ```sql
   create or replace procedure
   get_emps(p_start in int, p_end in int, p_count out int, p_cursor out sys_refcursor) as
   BEGIN
   	select count(*) into p_count from emp;
   	open p_cursor for
   		select * from 
   				(select e.*, rownum as r1 from emp e where e.rownum <= p_end)
   		where r1 > p_start;
   END get_emps;
   ```

2. 调用存储过程

   ```xml
   <!--  statementType="CALLABLE":表示要调用存储过程 -->
   <select id="getPageByProcedure" statementType="CALLABLE" databaseId="oracle">
       {call get_emps(
       #{start,mode=IN,jdbcType=INTEGER},
       #{end,mode=IN,jdbcType=INTEGER},
       #{count,mode=OUT,jdbcType=INTEGER},
       <!-- jdbcType指定返回值类型为游标 -->
       #{emps,mode=OUT,jdbcType=CURSOR,javaType=ResultSet,resultMap=PageEmp}
       )}
   </select>
   
   <!-- 返回结果集映射 -->
   <resultMap type="com.atguigu.mybatis.bean.Employee" id="PageEmp">
       <id column="EMPLOYEE_ID" property="id"/>
       <result column="LAST_NAME" property="email"/>
       <result column="EMAIL" property="email"/>
   </resultMap>
   ```

## 4.3、批量操作

### 4.3.1、每次获取批处理执行器

```java
public void testBatch() throws IOException{
		SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
    // TypSIMPLE 为每个sql语句创建新的预处理语句
		SqlSession openSession = sqlSessionFactory.openSession(ExecutorType.SIMPLE);
		try{
			EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
			for (int i = 0; i < 10000; i++) {
				mapper.addEmp(new Employee(UUID.randomUUID().toString().substring(0, 5), "b", "1"));
			}
			openSession.commit();
		}finally{
			openSession.close();
		}		
}
```

以上方式，每次**mapper.addEmp()**都会查询一次数据库，以下方式只会查询一次数据库：

```java
public void testBatch() throws IOException{
		SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
		// BATCH 将批量执行所有更新语句
		SqlSession openSession = sqlSessionFactory.openSession(ExecutorType.BATCH);
		try{
			EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
			for (int i = 0; i < 10000; i++) {
				mapper.addEmp(new Employee(UUID.randomUUID().toString().substring(0, 5), "b", "1"));
			}
      // 批量操作时，openSession.commit()后才一次将sql语句发送至数据库
			openSession.commit();
		}finally{
			openSession.close();
		}	
}
```

**开启SqlSession时指定ExecutorType的类型：**

- ExecutorType.SIMPLE：默认装配，未对执行mybatis执行流程有任何干扰
- ExecutorType.REUSE：mybatis会复用预处理语句
- ExecutorType.BATCH：mybatis会批量执行所有更新语句

### 4.3.2、配置批量的SqlSession

与Spring整合时，可以配置一个专门执行批量操作的是SqlSession。

```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"/>
  <constructor-arg name="executorType" value="BATCH"/>
</bean>
```

**注意：**

​	开启批量操作时，在执行**sqlSession.commit()**后才一次将sql语句发送至数据库。若需要提前执行，可使用**sqlSession.flushStatements()**方法将其冲刷到数据库。

## 4.4、自定义TypeHandler处理枚举

​	    通过自定义TypeHandler可在**ParameterHandler**设置参数或**ResultSetHandler**处理结果集时自定义参数封装策略。

1. 自定义枚举类

   ```java
   // 数据库保存 100,200
   public enum EmpStatus {
   	LOGIN(100,"用户登录"),LOGOUT(200,"用户登出"),REMOVE(300,"用户不存在");
   	
   	private Integer code;
   	private String msg;
   	private EmpStatus(Integer code,String msg){
   		this.code = code;
   		this.msg = msg;
   	}
   	public Integer getCode() {
   		return code;
   	}
   	public void setCode(Integer code) {
   		this.code = code;
   	}
   	public String getMsg() {
   		return msg;
   	}
   	public void setMsg(String msg) {
   		this.msg = msg;
   	}
   	
   	//按照状态码返回枚举对象
   	public static EmpStatus getEmpStatusByCode(Integer code){
   		switch (code) {
   			case 100:
   				return LOGIN;
   			case 200:
   				return LOGOUT;	
   			case 300:
   				return REMOVE;
   			default:
   				return LOGOUT;
   		}
   	}	
   }
   ```

2. 自定义TypeHandler

   实现TypeHandler接口或者继承BaseTypeHandler：

   ```java
   public class MyEnumEmpStatusTypeHandler implements TypeHandler<EmpStatus> {
   
   	@Override
   	public void setParameter(PreparedStatement ps, int i, EmpStatus parameter,
   			JdbcType jdbcType) throws SQLException {
   		ps.setString(i, parameter.getCode().toString());
   	}
   
   	@Override
   	public EmpStatus getResult(ResultSet rs, String columnName) throws SQLException {
   		int code = rs.getInt(columnName);
   		return EmpStatus.getEmpStatusByCode(code);
   	}
   
   	@Override
   	public EmpStatus getResult(ResultSet rs, int columnIndex) throws SQLException {
   		int code = rs.getInt(columnIndex);
   		return EmpStatus.getEmpStatusByCode(code);
   	}
   
   	@Override
   	public EmpStatus getResult(CallableStatement cs, int columnIndex) throws SQLException {
   		int code = cs.getInt(columnIndex);
   		return EmpStatus.getEmpStatusByCode(code);
   	}
   }
   ```

   





1. TypeHandler


我们可以通过自定义TypHdl的形式来

# 五、Mybatis缓存

缓存机制减轻数据库压力，提高数据库性能。mybatis的缓存分为两级：一级缓存、二级缓存。

## .1、一级缓存（本地缓存）

​	一级缓存为 `sqlsesson` 缓存，缓存的数据只在 SqlSession 内有效。 mybatis 默认开启一级缓存，无需配置。

​	`SqlSession session = sqlSessionFactory.openSession();`

​	操作数据库时需先创建 SqlSession ，在SqlSession中有一个私有有 HashMap 用于存储缓存数据。

**具体流程：**

​	第一次执行 select 完毕会将查到的数据写入 SqlSession 内的 HashMap 中缓存起来

​	第二次执行 select 会从缓存中查数据，如果 select 同传参数一样，那么就能从缓存中返回数据，不用去数据库了，从而提高了效率

**`sqlsesson` 缓存失效情况：**

- 若同一SqlSession执行两次查询的查询条件不同，则缓存失效。因为 mybatis 缓存基于 [namespace:sql语句:参数]（作为hashmap的key） 。
- 若 SqlSession 执行了 DML 操作（insert、update、delete），并 commit ，则 mybatis 会清空当前 SqlSession 中缓存的所有数据，保证缓存数据与数据库数据的一致；
- 手动清除了一级缓存，清除方法：`sqlSession.clearCache()`；
- 若 SqlSession 结束，则其私有的HashMap一级缓存也就不存在了。

**开启一级缓存步骤：**

1. 在全局配置文件中开启**一级缓存**

   3.3版本前，一级缓存默认开启，无法关闭。3.3版本后，可使用localCacheScope关闭一级缓存。

   ```xml
   <settings>
     	<!-- value="STATEMENT"：关闭一级缓存 -->
       <setting name="localCacheScope" value="SESSION"/> 
   <settings>
   ```

## 5.2、二级缓存（全局缓存）

二级缓存是` mapper` 级别的缓存，一个 namespace 对应一个二级缓存。

**原理：**

​	当多个 SqlSession 使用同一个 Mapper 操作数据库的时候，得到的数据会缓存在同一个二级缓存区域

二级缓存默认是没有开启的。需要在 setting 全局参数中配置开启二级缓存

**具体流程：**

- 当一个` sqlseesion `执行了一次` select` 后，在关闭此` session` 时，会将查询结果缓存到二级缓存；

- 当另一个` sqlsession `执行` select` 时，会先在他自己的一级缓存中找；若没找到，就去二级缓存中找；若找到了就返回，否则就去查询数据库。

**开启二级缓存步骤：**

1. 在全局配置文件中开启**二级缓存**

   ```xml
   <settings>
       <setting name="cacheEnabled" value="true"/> <!-- 默认是false：关闭二级缓存 -->
   <settings>
   ```

2. 在userMapper.xml中开启**二级缓存**

   ```xml
   <!-- 当前mapper下所有语句开启二级缓存 -->
   <cache eviction="LRU" flushInterval="60000" size="512" readOnly="true"/>
   ```

   - **eviction** ：回收策略

     - LRU(最近最少使用的)：移除最长时间不被使用的对象，默认值；
     - FIFO(先进先出):按对象进入缓存的顺序来移除它们；
     - SOFT(软引用)：移除基于垃圾回收器状态和软引用规则的对象；
     - WEAK(弱引用)：更积极地移除基于垃圾收集器状态和弱引用规则的对象。

   - **flushlnterval** ： 缓存刷新(清空)间隔（毫秒）

     可设置为任意的正整数。默认不设置，即没有刷新间隔。

   - **size**：**引用数目**

     设置存放缓存的对象个数。默认值是1024 。

   - **readOnly**：是否只读

     - true：mybatis将认为从缓存中获取的数据都是只读的不会被修改，所以直接将缓存中数据的索引返回。存在安全隐患，但速度快；

     - fasle(默认)：mybatis会会通过序列化返回缓存对象的拷贝。速度慢，但安全。

       注意POJO需要实现序列化接口。 

   - **Type** : 用于指定自定义缓存的全类名。

3. 被缓存的POJO对象需要实现序列化接口

4. 在mapper中具体的select查询标签里开启**二级缓存**：userCache="true"，默认true

   ```xml
   <select id="dynamicForeachTest" resultType="Blog" userCache="true">  
           select * from t_blog where id = #{item}  
   </select>  
   ```

   **另外：**

   在insert、delete、update标签中，默认设置参数：flushCache="true"。当sql执行后会清空一、二级缓存。

   在selete标签中，默认设置参数：flushCache="false"。当sql执行后不会清空一、二级缓存。



# 六、MyBatis插件

## .1、插件原理

MyBatis执行流程中会创建四大对象：

- StatementHandler ： 处理sql语句预编译，设置参数等相关工作
- ParameterHandler ： 设置预编译参数
- Executor ：执行sql语句
- ResultSetHandler：处理结果集

四大对象在创建的时候都会调用：interceptorChain.pluginAll(parameterHandler)，因此可以在此处添加拦截器。

**插件拦截顺序：**

​	插件拦截顺序与配置文件中插件注册顺序一致。多个插件依次拦截对目标对象包裹，先声明的先包裹。

**插件执行顺序：**

​	插件执行顺序与配置文件中插件注册顺序相反，后注册的插件最先执行。

## 6.2、插件开发

1. 编写插件实现Interceptor接口，并使用编写插件实现Itpt

   ```java
   package com.demo.mybatis.plugin;
   
   // type指定拦截对象类型，method指定方法名；args指定方法参数
   @Intercepts({ @Signature(type=StatementHandler.class,method="parameterize",
                            args=java.sql.Statement.class)})
   public class PluginOne implements Interceptor{
   
     // 拦截目标对象的目标方法的执行
   	@Override
   	public Object intercept(Invocation invocation) throws Throwable {
   		Object target = invocation.getTarget();
   		// 获取target元数据：StatementHandler==>ParameterHandler===>parameterObject
   		MetaObject metaObject = SystemMetaObject.forObject(target);
   		//修改sql语句参数
   		metaObject.setValue("parameterHandler.parameterObject", 11);
   		return invocation.proceed();
   	}
   
     // 使用Plugin的wrap方法用于为目标对象创建一个代理对象
   	@Override
   	public Object plugin(Object target) {
   		return Plugin.wrap(target, this);
   	}
   
   	@Override
   	public void setProperties(Properties properties) {
   		// TODO 插件注册时，会将<property>属性封装成Properties对象
   	}
   }
   ```

2. 配置文件中注册插件

   ```xml
   <plugins>
   		<plugin interceptor="com.demo.mybatis.plugin.PluginOne">
   			<property name="username" value="root"/>
   			<property name="password" value="123456"/>
   		</plugin>
   		<!-- <plugin interceptor="com.atguigu.mybatis.dao.PluginSecode"></plugin> -->
   	</plugins>
   ```

   

# MyBatis原理简介

## Mybatis查询流程

<img src="/Users/yangying/devtools/SporadicNotes/图片/mybatis查询流程.png" alt="mybatis查询流程" style="zoom:50%;" />

```java
public void test01() throws IOException {
		// 1、获取sqlSessionFactory对象
		SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
		// 2、获取sqlSession对象
		SqlSession openSession = sqlSessionFactory.openSession();
		try {
			// 3、获取接口的实现类对象
			EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
      // 4、使用代理对象去执行增删改查方法
			Employee employee = mapper.getEmpById(1);
		} finally {
			openSession.close();
		}
}
```

1. SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();

   解析mybatis全局配置文件，将信息保存在**Configuration**中；

   解析mapper.xml文件，将每一个CRUD标签封装成**MappedStatement**对象，并放在**Configuration**中；

   返回包含**Configuration**的**DefaultSqlSession**；

   ![mybatis原理-1](/Users/yangying/devtools/SporadicNotes/图片/mybatis原理-1.png)

2. SqlSession openSession = sqlSessionFactory.openSession();

   创建**Executor**对象；返回一个包含了**Executor**和**Configuration**的**DefaultSQlSession**对象；

   ![mybatis原理-2](/Users/yangying/devtools/SporadicNotes/图片/mybatis原理-2.png)

3. EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);

   使用**MapperProxyFactory**创建一个包含**DefaultSqlSession**（**Executor**）的代理对象**MapperProxy**；

   ![mybatis原理-3](/Users/yangying/devtools/SporadicNotes/图片/mybatis原理-3.png)

4. mapper.getEmpById(1);

   调用**DefaultSqlSession**的增删改查（Executor）；

   *         创建一个StatementHandler对象；
   *            同时创建出ParameterHandler和ResultSetHandler对象；
   *         调用StatementHandler预编译参数以及设置参数值;
   *            使用ParameterHandler来给sql设置参数
   *         调用StatementHandler的增删改查方法；
   *         ResultSetHandler封装结果

![mybatis原理-4](/Users/yangying/devtools/SporadicNotes/图片/mybatis原理-4.png)



