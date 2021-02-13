[TOC]



# 一、Maven简介

​	maven是apache软件基金会组织维护的一款自动化构建工具，专 注服务于java平台的项目构建和依赖管理。

- 快速寻找jar包
- 解决jar包依赖问题
- 解决jar包冲突问题
- 统一管理jar包
- 约定统一的项目结构
- 规范项目的生命周期

# 二、Maven命令格式

```shell
## mvn 插件名称:执行命令
mvn help:system
## mvn 插件名称:help
mvn clean:help
```

# 三、Maven配置

## 3.1、启动文件

运行```mvn```命令时会加载启动的配置文件```settings.xml    ```:

默认全局配置文件：M2_HOME/conf    

用户范围配置文件：~/.m2    

## 3.2、本地缓存目录    

settings.xml 可以设置本地缓存目录，用于保存maven从远程仓库下载的插件和jar包。

```xml
<localRepository>C:/Users/tom/.m1/repository</localRepository>
```

# 四、坐标和依赖详解    

## 3.1、Maven坐标

```xml
<groupId>com.javacode2018</groupId>
<artifactId>springboot-chat01</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>jar</packaging>
```

- goupId：定义当前构件所属的组，通常与域名反向一一对应。
- artifactId：项目组中构件的编号。 
- version：当前构件的版本号，每个构件可能会发布多个版本，通过版本号来区分不同版本的构 件。 
- package：定义该构件的打包方式，比如我们需要把项目打成jar包，采用 java -jar 去运行这个 jar包，那这个值为jar；若当前是一个web项目，需要打成war包部署到tomcat中，那这个值就是 war，可选（jar、war、ear、pom、maven-plugin。

**groupId、artifactId、version是必须要定义的，packeage可以省略，默认为jar。**   



## 3.2、Maven导入依赖的构件

```xml
<project>
    <dependencies>

        <dependency>
            <groupId></groupId>
            <artifactId></artifactId>
            <version></version>
            <type></type>
            <scope></scope>
            <optional></optional>
            <exclusions>
                <exclusion></exclusion>
                <exclusion></exclusion>
            </exclusions>
        </dependency>
        
    </dependencies>
</project>
```

- dependency 就表示当前项目需要依赖的 一个构件的信息  

- dependency中groupId、artifactId、version是定位一个构件的必要信息

- type 表示所要依赖的构件的类型，对应于被依赖的构件的packaging 。一般元素不被声明，默认值为jar

- scope：依赖的范围    

- option：标记依赖是否可选

  若 A->B  scope:compile 且 B->C  scope:compile ，则C会被A自动引入。

  若将B->C时的optional    设置为true，则A会根据实际需要决定是否依赖C。

  - 减少不必要的依赖传递
  - 减少jar包冲突

- exclusions：排除一个或多个依赖的传递，声明exclusion时只需要groupId、artifactId，version可省略 。   

  

## 3.3、maven依赖范围（scope )

- **compile** 编译依赖范围

  对于**编译源码、编译测试代码、测试、运行**4种 classpath都有效，默认该依赖范围。

- **test** 测试依赖范围

  对**编译测试、运行测试**的classpath有效，在编译主代码、运行项目时无法使用此类依赖。

- **provide** 已提供依赖范围。

  表示项目的运行环境中已经提供了所需要的构件。对于**编译源码、编译测试、运行测试**中classpath有效，运行时无效。

- **runtime** 运行时依赖范围，

  对于**编译测试、运行测试和运行项目**的classpath有 效，但在编译主代码时无效，比如jdbc驱动实现，运行时候才需要具体的jdbc驱动实现。 

- **system** 系统依赖范围，该依赖与3中classpath的关系，和provided依赖范围完全一致。但是，使用system范 围的依赖时必须通过systemPath元素显示第指定依赖文件的路径。这种依赖直接依赖于本地路径中的 构件。

**依赖范围与classpath的关系如下:** 

| 依赖范围 | 编译源码 | 编译测试代码 | 运行测试 | 运行项目 | 示例        |
| -------- | -------- | ------------ | -------- | -------- | ----------- |
| compile  | Y        | Y            | Y        | Y        | spring-web  |
| test     | -        | Y            | Y        | -        | junit       |
| provide  | Y        | Y            | Y        | -        | servlet-api |
| runtime  | -        | Y            | Y        | Y        | jdbc驱动    |
| system   | Y        | Y            | Y        | -        | 本地的jar包 |

 ## 3.4、依赖的传递性

|          | compile  | test | provided | runtime  |
| -------- | -------- | ---- | -------- | -------- |
| compile  | compile  | -    | -        | runtime  |
| test     | test     | -    | -        | test     |
| provided | provided | -    | provided | provided |
| runtime  | runtime  | -    | -        | runtime  |

 ## 3.5、maven依赖调解功能    

### 3.5.1、路径最近原则    

存在这样的情况，A->B->C->Y(1.0)，A->D->Y(2.0)，此时Y出现了2个版本：1.0和2.0 。

由于Y的2.0版本距离A更近一些，所以maven会选择2.0。

### 3.5.2、最先声明原则    

存在这样的情况，A->B->Y(1.0)，A->D->Y(2.0)   ，路径相同，版本不同。

此时会看pom.xml中A所依赖的B、D在 dependencies 中声明的位置，谁在前面，就以谁为主。    

# 五、Maven仓库

用于存放项目中依赖的第三方库 。采用引用的方式将依赖的jar引入项目，在打包时才会将jar拷贝到安装包中。

## 5.1、远程仓库

- 中央仓库   

  - 由maven官方社区提供    
  - maven内置的默认的远程仓库  
  - 使用时需要连接外网    
  - 默认位置：apache-maven-3.6.1\lib\maven-model-builder-3.6.1.jar\org\apache\maven\model\pom- 4.0.0.xml    

- 私服    

  - maven仓库管理软件 : archiva 、Artifactory   、Nexus    

- 其他公共远程仓库    

  中央仓库在国外，访问速度不快。一些大公司自己搭建了 maven公共仓库服务器，给开发者使用。

## 5.2、本地仓库

1. 本地仓库默认地址：~/.m2/respository 。可在 ~/.m2/settings.xml 文件中进行修改   ：

   ```java
   <localRepository>本地仓库地址</localRepository>
   ```



## 5.3、Maven中远程仓库的配置    

###5.3.1、pom.xml中配置远程仓库    

```xml
<project>
    <repositories>
        <repository>
        <id>aliyun-releases</id>
        <url>https://maven.aliyun.com/repository/public</url>
        <releases>
       		<enabled>true</enabled>
        </releases>
        <snapshots>
        	<enabled>false</enabled>
        </snapshots>
        </repository>
    </repositories>
</project>
```

- id：远程仓库的一个标识，中央仓库的id是 central 。添加远程仓库时，id不要和中央仓库id重复，会把中央仓库的覆盖掉 
- url：远程仓库地址 
- releases：用来配置是否需要从这个远程仓库下载稳定版本构件 
- snapshots：用来配置是否需要从这个远程仓库下载快照版本构件    

**pom中配置远程仓库的方式只对当前项目起效**。

### 5.3.2、镜像方式

采用镜像的方式在 settings.xml  中配置远程仓库，对所有使用该配置的maven项目起效。

```xml
<mirrors>
    <mirror>
        <id>mirrorId</id>
        <mirrorOf>repositoryId</mirrorOf>
        <name>Human Readable Name for this Mirror.</name>
        <url>http://my.repository.com/repo/path</url>
    </mirror>
</mirrors> 
```

- id：镜像的id，是一个标识 
- name：镜像的名称
- url：镜像对应的远程仓库的地址 
- mirrorOf：指定哪些远程仓库的id使用这个镜像，对应pom.xml中repository元素的id，表示这个镜像是给哪些pom.xml中的远程仓库使用，多个远程仓库的id之间用逗号隔开， * 表示给所有远程仓库做镜像  。
- 镜像仓库完全屏蔽了被镜像的仓库，所以当镜像仓库无法使用的时候，maven是无法自动切换 到被镜像的仓库的，此时下载构件会失败 。

**mirrorOf的用法：**

- 匹配所有远程仓库 

  <mirrorOf>*</mirrorOf>    

- 匹配指定的仓库    

  <mirrorOf>远程仓库1的id,远程仓库2的id</mirrorOf>    

- 匹配所有远程仓库，除了repo1

  <mirrorOf>*,! repo1</mirrorOf>    

## 5.4、Maven寻找依赖的jar过程    

1. 先查看本地仓库
   - 若存在，则直接使用；
   - 若不存在，则查看远程仓库
2. 在远程仓库中查找
   - 若找到了，则将其下载到本地仓库中进行使用
   - 若没找到，则maven报错

##5.5、发布本地构件发布到Maven私服    

### 5.5.1、使用maven部署构件    

1. 配置  pom.xml  

```java
<distributionManagement>
    <repository>
        <id>release-nexus</id>
        <url>http://localhost:8081/repository/maven-releases/</url>
        <name>nexus私服中宿主仓库->存放/下载稳定版本的构件</name>
    </repository>
    <snapshotRepository>
        <id>snapshot-nexus</id>
        <url>http://localhost:8081/repository/maven-snapshots/</url>
        <name>nexus私服中宿主仓库->存放/下载快照版本的构件</name>
    </snapshotRepository>
</distributionManagement>
```

2. 配置  settings.xml    

```xml
<server>
    <id>release-nexus</id>
    <username>admin</username>
    <password>admin123</password>
</server>
<server>
    <id>snapshot-nexus</id>
    <username>admin</username>
    <password>admin123</password>
</server>
```

3. 使用命令 mvn deploy 。对构件进行打包，然后上传到私服中 。

**注意：**

- snapshot属于快照版本，同一个snapshot版本的构件可以重复部署到私服，已存在会进行覆盖掉。 
- release是稳定版本，重复部署会报错    。

### 5.5.2、手动部署第三方构件    



# 六、生命周期和插件详解    

## 6.1、maven生命周期

1. clean 生命周期

| 生命周期阶段 | 描述                                  |
| ------------ | ------------------------------------- |
| pre-clean    | 执行一些需要在clean之前完成的工作     |
| clean        | 移除所有上一次构建生成的文件          |
| post-clean   | 执行一些需要在clean之后立刻完成的工作 |

2. default 生命周期

| 生命周期阶段            | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| validate                | 校验：校验项目是否正确并且所有必要的信息可以完成项目的构建过 程。 |
| initialize              | 初始化：初始化构建状态，比如设置属性值。                     |
| generate-sources        | 生成源代码：生成包含在编译阶段中的任何源代码。               |
| process-sources         | 处理源代码：处理源代码，比如说，过滤任意值。                 |
| generate-resources      | 生成资源文件：生成将会包含在项目包中的资源文件。             |
| process-resources       | 编译：复制和处理资源到目标目录，为打包阶段最好准备。         |
| compile                 | 处理类文件：编译项目的源代码。                               |
| process-classes         | 处理类文件：处理编译生成的文件，比如说对Java class文件做字节码改 善优化。 |
| generate-test sources   | 生成测试源代码：生成包含在编译阶段中的任何测试源代码。       |
| process-test sources    | 处理测试源代码：处理测试源代码，比如说，过滤任意值。         |
| generate-test resources | 生成测试源文件：为测试创建资源文件。                         |
| process-test resources  | 处理测试源文件：复制和处理测试资源到目标目录。 ava           |
| test-compile            | 编译测试源码：编译测试源代码到测试目标目录. 甲               |
| process-test-classes    | 处理测试类文件：处理测试源码编译生成的文件。                 |
| test                    | 测试：使用合适的单元测试框架运行测试（Juint是其中之一）。 ： |
| prepare-package         | 准备打包：在实际打包之前，执行任何的必要的操作为打包做准备。 |
| package                 | 打包：将编译后的代码打包成可分发格式的文件，比如JAR、WAR或者 EAR文件。 |
| pre-integration-test    | 集成测试前：在执行集成测试前进行必要的动作。比如说，搭建需要的 环境。 |
| integration-test        | 集成测试：处理和部署项目到可以运行集成测试环境中。           |
| post-integration test   | 集成测试后：在执行集成测试完成后进行必要的动作。比如说，清理集 成测试环境。 |
| verify                  | 验证：运行任意的检查来验证项目包有效且达到质量标准。         |
| install                 | 安装：安装项目包到本地仓库，这样项目包可以用作其他本地项目的依 赖。 |
| deploy                  | 部署：将最终的项目包复制到远程仓库中与其他开发者和项目共享。 |
| pre-site                | 执行一些需要在生成站点文档之前完成的工作                     |
| site                    | 生成项目的站点文档                                           |
| post-site               | 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备   |
| site-deploy             | 将生成的站点文档部署到特定的服务器上                         |

3. site生命周期    

| 阶段        | 描述                                                       |
| ----------- | ---------------------------------------------------------- |
| pre-site    | 执行一些需要在生成站点文档之前完成的工作                   |
| site        | 生成项目的站点文档                                         |
| post-site   | 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备 |
| site-deploy | 将生成的站点文档部署到特定的服务器上                       |

## 6.2、maven插件

### 6.2.1、maven插件命令

1. **查看插件前缀（类似于简写）**

```shell
# 语法：mvn help:describe -Dplugin=插件goupId:插件artifactId[:插件version]
mvn help:describe -Dplugin=org.apache.maven.plugins:maven-surefire-plugin
## 输出如下
Name: Maven Surefire Plugin
Description: Surefire is a test framework project.
Group Id: org.apache.maven.plugins
Artifact Id: maven-surefire-plugin
Version: 2.12.4
Goal Prefix: surefire ##前缀为 surefire
```

maven通过插件前缀与插件groupId:artifactId的关系来解析插件前缀，此配置存储在仓库的元数据中：

```txt
~/.m2/repository/org/apache/maven/plugins/maven-metadata-central.xml
~/.m2/repository/org/codehaus/mojo/maven-metadata-central.xml    
```

也可以在`settings.xml`  文件中配置自定义插件得到元数据目录：

```xml
<settings>
    <pluginGroups>
    	<pluginGroup>com.your.plugins</pluginGroup>
    </pluginGroups>
</settings>
```

  

2. **列出插件目标详情**

- 方式一

```shell
# 列出插件目标
## 语法：mvn 插件goupId:插件artifactId[:插件version]:help 
mvn org.apache.maven.plugins:maven-surefire-plugin:help
mvn org.apache.maven.plugins:maven-surefire-plugin:help -Dgoal=test -Ddetail=true 
## 语法：mvn 插件前缀:help  
mvn surefire:help

# 列出插件目标及参数
## 语法：mvn 插件goupId:插件artifactId[:插件version]:help -Dgoal=目标名称 -Ddetail
mvn org.apache.maven.plugins:maven-clean-plugin:help -Dgoal=help -Ddetail
## 语法：mvn 插件前缀:help -Dgoal=目标名称 -Ddetail
mvn clean:help -Dgoal=help -Ddetail
```

- 方式二

```shell
# 调用help插件的 describe 目标
## 语法：mvn help:describe -Dplugin=插件goupId:插件artifactId[:插件version] -Dgoal=目标名称 -Ddetail
## 语法：mvn help:describe -Dplugin=插件前缀 -Dgoal=目标名称 -Ddetail
```

3. **运行目标插件**    

```shell
# 语法：mvn 插件goupId:插件artifactId[:插件version]:插件目标 [-D目标参数1] [-D目标参数2] [-D目标参数n]
## 查看插件test目标详情
mvn org.apache.maven.plugins:maven-surefire-plugin:help -Dgoal=test -Ddetail=true
## 运行插件
mvn org.apache.maven.plugins:maven-surefire-plugin:test 
mvn org.apache.maven.plugins:maven-surefire-plugin:test -Dmaven.test.skip=true

## 语法：mvn 插件前缀:插件目标 [-D目标参数1] [-D目标参数2] [-D目标参数n]
```

**查看插件的默认绑定**

```shell
# mvn help:describe -Dplugin=插件goupId:插件artifactId[:插件version] -Dgoal=目标名称 -Ddetail
# mvn help:describe -Dplugin=插件前缀 -Dgoal=目标名称 -Ddetail
```

​    

### 6.2.2、maven插件命令传参

####6.2.2.1、命令后跟 -D[属性名称]

```shell
mvn org.apache.maven.plugins:maven-surefire-plugin:test -Dmaven.test.skip=true
```

####6.2.2.2、在pom文件 \<properties\>元素中指定

```xml
<maven.test.skip>true</maven.test.skip>
```

#### 6.2.2.3、build中配置插件参数的方式 

```xml
<build>
    <plugins>
        <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.12.4</version>
        <!-- 插件参数配置，对插件中所有的目标起效 -->
        <configuration>
        	<skip>true</skip>
        </configuration>
        </plugin>
    </plugins>
</build>
```



## 6.3、maven插件与生命周期的绑定

### 6.3.1、内置绑定

- clean生命周期阶段与插件绑定关系    

| 生命周期阶段 | 插件:目标                |
| ------------ | ------------------------ |
| pre-clean    |                          |
| clean        | maven-clean-plugin:clean |
| post-clean   |                          |

- default生命周期阶段与插件绑定关系 

| 生命周期阶段           | 插件:目标                            | 执行任务                        |
| ---------------------- | ------------------------------------ | ------------------------------- |
| process-resources      | maven-resources-plugin:resources     | 复制主资源文件至主输出目录      |
| compile                | maven-compiler-plugin:compile        | 编译主代码至主输出目录          |
| process-test resources | maven-resources plugin:testResources | 复制测试资源文件至测试输出 目录 |
| test-compile           | maven-compiler plugin:testCompile    | 编译测试代码至测试输出目录      |
| test                   | maven-surefile-plugin:test           | 执行测试用例                    |
| package                | maven-jar-plugin:jar                 | 创建项目jar包                   |
| install                | maven-install-plugin:install         | 将输出构件安装到本地仓库        |
| deploy                 | maven-deploy-plugin:deploy           | 将输出的构件部署到远程仓库      |

- site生命周期阶段与插件绑定关系

| 生命周期阶段 | 插件:目标                |
| ------------ | ------------------------ |
| pre-site     |                          |
| site         | maven-site-plugin:site   |
| post-site    |                          |
| site-deploy  | maven-site-plugin:deploy |

### 6.3.2、自定义绑定

​	将插件 maven-source-plugin 的 jar-no-fork 目标绑定到 default 生命周期的 verify 阶段上，创建的项目源码jar包安装到仓库中。 

```xml
<build>
    <plugins>
        <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-source-plugin</artifactId>
        <version>3.2.0</version>
        <executions>
            <!-- 使用插件需要执行的任务 -->
            <execution>
                <!-- 任务id -->
                <id>attach-source</id>
                <!-- 任务中插件的目标，可以指定多个 -->
                <goals>
                     <goal>jar-no-fork</goal>
                </goals>
                <!-- 绑定的阶段 -->
                <phase>verify</phase>
            </execution>
        </executions>
        </plugin>
    </plugins>
</build>
```

​	在 default 生命周期的 validate 阶段绑定清理代码的插件，就不需要每次mvn compile 前 mvn clean：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-clean-plugin</artifactId>
    <version>2.5</version>
    <executions>
        <!-- 使用插件需要执行的任务 -->
        <execution>
            <!-- 任务中插件的目标，可以指定多个 -->
            <id>clean-target</id>
            <goals>
            	<goal>clean</goal>
            </goals>
            <!-- 绑定的阶段 -->
            <phase>validate</phase>
        </execution>
    </executions>
</plugin>
```



## 6.4、插件配置

###6.4.1、插件仓库配置

```xml
<pluginRepositories>
    <pluginRepository>
        <id>myplugin-repository</id>
        <url>http://repo1.maven.org/maven2/</url>
        <releases>
        	<enabled>true</enabled>
        </releases>
    </pluginRepository>
</pluginRepositories>
```

###6.4.2、导入插件

在pom.xml中若配置插件是官方插件时，可省略 groupId   ：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId> <!-- 可省略 -->
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
                <compilerVersion>1.8</compilerVersion>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

#七、聚合、继承、裁剪

## 7.1、聚合

用于实现多模块的快速构建。

```xml
<groupId>com.javacode</groupId>
<artifactId>javacode-aggregator</artifactId>
<version>1.0-SNAPSHOT</version>
<!-- 引用被聚合的模块 -->
<modules>
    <module>com.javacode-pc</module>
    <module>com.javacode-h5</module>
    <module>com.javacode-api</module>
</modules>
<!-- 此处必须为pom -->
<packaging>pom</packaging>
```

对于聚合来说，聚合模块是知道被聚合模块的存在的，而被聚合模块是感知不到聚合模块的存在。

## 7.2、继承

用于实现相同配置的重用。 

###7.2.1、依赖继承

1. 父工程

```xml
<groupId>com.javacode2018</groupId>
<artifactId>javacode2018-parent</artifactId>
<version>1.0-SNAPSHOT</version>

<packaging>pom</packaging>

<!-- 子构件默认继承父构件dependencies元素定义的所有依赖 -->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.2.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.3</version>
    </dependency>
</dependencies>
```

​	dependencies元素定义的依赖默认被子构件全部继承。

​	dependencyManagement元素声明的依赖不会默认不会被子构件引入实际的依赖。 但会对子构件 dependencies 中使用的依赖起到约束作用。待子构件使用dependencie 引用父构件声明的依赖后，依赖才会起效。

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>5.2.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.3</version>
        </dependency>
    </dependencies>
</dependencyManagement
```

2. 子工程

```xml
<modelVersion>4.0.0</modelVersion>

<parent>
    <groupId>com.javacode2018</groupId>
	<artifactId>javacode2018-parent</artifactId>
	<version>1.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
</parent>

<artifactId>javacode2018-pc</artifactId>

<!-- version 信息将从父构件继承-->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
    </dependency>
</dependencies>
```

**单继承存在的问题：**

​	使用继承时，  只有dependencyManagement中声明的依赖才可能被子pom.xml使用。若项目已存在父pom.xml ，但仍想使用另一项目dependencyManagement中声明的依赖，此时单继承无法解决。

​	**解决方案：**

```xml
<dependencyManagement>
    <dependencies>
        <!-- 相当于把javacode2018-parent中所有声明的依赖导入到dependencyManagement下 --> 
        <dependency>
            <groupId>com.javacode2018</groupId>
            <artifactId>javacode2018-parent</artifactId>
            <version>1.0-SNAPSHOT</version>
            <type>pom</type>		<!-- type 必须为 pom --> 
            <scope>import</scope>	 <!-- scope 必须为 import -->
        </dependency>
        <dependency>构件2</dependency>
        <dependency>构件n</dependency>
    </dependencies>
</dependencyManagement>
```

### 7.2.2、插件继承

父pom中 pluginManagement  元素声明的插件配置不会被子pom,xml继承，只有子pom.xml中使用plugins->plugin 元素时，插件才会生效。

- 父构件插件配置

```xml
<build>
    <pluginManagement>
         <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>3.2.0</version>
                <executions>
                    <execution>
                        <id>attach-source</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>jar-no-fork</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

- 子构建插件配置

  引用父构建插件：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

​	自定义引用父构件的插件（自定义插件目标将和父插件目标合并）：

```xml
<build>
    <plugins>
        <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-source-plugin</artifactId>
        <executions>
                <execution>
                <id>attach-source</id>
                <goals>
                	<goal>help</goal>
                </goals>
            </execution>
        </executions>
        </plugin>
    </plugins>
</build>
```

对于继承来说，父构件是感知不到子构件的存在，而子构件需要使用 parent 来引用父构件。    

## 7.3、反应堆

###7.3.1、反应堆完全构建

现有如下项目结构：

- b2b
  - b2b-account
    - b2b-account-api
    - b2b-account-service （依赖于 b2b-account-api）
  - b2b-order
    - b2b-order-api
    - b2b-order-service（依赖于 b2b-order-api、b2b-account-api）

各项目pom.xml文件如下：

**b2b/pom.xml**    

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.javacode2018</groupId>
    <artifactId>b2b</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>b2b-account</module>
        <module>b2b-order</module>
    </modules>
</project>
```

**b2b-account/pom.xml**    

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.javacode2018</groupId>
    <artifactId>b2b-account</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>b2b-account-api</module>
        <module>b2b-account-serivce</module>
    </modules>
</project>
```

**b2b-account-api/pom.xml**    

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
   <groupId>com.javacode2018</groupId>
    <artifactId>b2b-account-api</artifactId>
    <version>1.0-SNAPSHOT</version>
</project>
```

**b2b-account-serivce/pom.xml**    

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.javacode2018</groupId>
    <artifactId>b2b-account-service</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <dependencies>
        <dependency>
            <groupId>${project.groupId}</groupId>
            <artifactId>b2b-account-api</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

**b2b-order/pom.xml**    

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.javacode2018</groupId>
    <artifactId>b2b-order</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>b2b-order-api</module>
        <module>b2b-order-service</module>
    </modules>
</project>
```

**b2b-order-api/pom.xml**    

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.javacode2018</groupId>
    <artifactId>b2b-order-api</artifactId>
    <version>1.0-SNAPSHOT</version>
    
</project>
```

**b2b-order-service/pom.xml**    

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.javacode2018</groupId>
    <artifactId>b2b-order-service</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <dependencies>
        <dependency>
            <groupId>${project.groupId}</groupId>
            <artifactId>b2b-account-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>${project.groupId}</groupId>
            <artifactId>b2b-order-api</artifactId>
            <version>${project.version}</version>
        </dependency>
	</dependencies>
</project>
```

在b2b/pom.xml 所在 目录执行下面命令 ：mvn clean install ，则maven会根据模块之间的依赖关系，得到所有模块的构建顺序（由maven的反应堆reactor计算），并按照列出的顺序依次进行构建 ：

```xml
[INFO] Reactor Build Order:
[INFO]
[INFO] b2b-account-api [jar]
[INFO] b2b-account-service [jar]
[INFO] b2b-account [pom]
[INFO] b2b-order-api [jar]
[INFO] b2b-order-service [jar]
[INFO] b2b-order [pom]
[INFO] b2b
```

 ### 7.3.2、反应堆按需构建

**-pl,--projects \<arg\>**    

```shell
# 构件指定的模块，arg表示多个模块，之间用逗号分开，模块有两种写法
## -pl 模块1相对路径 [,模块2相对路径] [,模块n相对路径]
## -pl [模块1的groupId]:模块1的artifactId [,[模块2的groupId]:模块2的artifactId] [,[模块n的groupId]:模块n的artifactId]
mvn clean install -pl b2b-account
mvn clean install -pl b2b-account/b2b-account-api
mvn clean install -pl b2b-account/b2b-account-api,b2b-account/b2b-accountservice
mvn clean install -pl :b2b-account-api,b2b-order/b2b-order-api
mvn clean install -pl :b2b-account-api,:b2b-order-service
```

**-rf,--resume-from \<arg\>** 

```shell
# 从指定的模块产生反应堆 
mvn clean install -rf b2b-order/b2b-order-service
```

```shell
## mvn clean installd 的执行日志
[INFO] Reactor Build Order:
[INFO]
[INFO] b2b-account-api [jar]
[INFO] b2b-account-service [jar]
[INFO] b2b-account [pom]
[INFO] b2b-order-api [jar]
[INFO] b2b-order-service [jar]
[INFO] b2b-order [pom]
[INFO] b2b [pom]
## mvn clean install -rf b2b-order/b2b-order-service 的执行日志
[INFO] Reactor Build Order:
[INFO]
[INFO] b2b-order-service [jar]
[INFO] b2b-order [pom]
[INFO] b2b [pom]
```

**-am,--also-make** 

```shell
# 同时构建所列模块的依赖模块 
mvn clean install -pl b2b-account/b2b-order-service -am
```

**-amd,--also-make-dependents** 

```shell
# 同时构件依赖于所列模块的模块    
mvn clean install -pl b2b-account/b2b-account-api -amd
```



# 八、maven多环境构建

## 8.1、Maven属性    

### 8.1.1、自定义属性

```xml
<properties>
	<spring.verion>5.2.1.RELEASE</spring.verion>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.verion}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${spring.verion}</version>
    </dependency>
</dependencies>
```

### 8.1.2、默认属性

- 内置属性

```xml
${basedir}：表示项目根目录，即包含pom.xml文件的目录
${version}：表示项目的版本号
```

- POM属性

```xml
${pom.build.sourceDirectory}：项目的主源码目录，默认为src/main/java/
${project.build.testSourceDirectory}：项目的测试源码目录，默认为src/test/java/
${project.build.directory}：项目构建输出目录，默认为target/
${project.build.outputDirectory}：项目主代码编译输出目录，默认为target/classes
${project.build.testOutputDirectory}：项目测试代码编译输出目录，默认为target/testclasses
${project.groupId}：项目的groupId
${project.artifactId}：项目的artifactId
${project.version}：项目的version，与${version}等价
${project.build.finalName}：项目打包输出文件的名称，默认为
${project.artifactId}-${project.version}
```

- Settings属性    

```xml
${settings.localRepository} : 指向用户本地仓库的地址
```

- java系统属性    

  ​	所有java系统属性都可以使用maven属性来引用，如 ${user.home} 指向了当前用户目录。 java系统属性可以通过 mvn help:system 命令看到 。

- 环境变量属性    

  ​	所有的环境变量都可以使用env.开头的方式来进行引用。使用 mvn help:system 查看所有环境变量。

```xml
${env.JAVA_HOME}
```

## 8.2、profiles处理多环境构建   

关键设置：

1. 开启文件内容动态替换 && 设置需要替换资源的文件

```xml
<build>
    
    <resources>
	    <!-- 仅复制jdbc.properties，且内容替换 -->       
        <resource>
            <!-- 指定资源文件的目录 -->
            <directory>${project.basedir}/src/main/resources</directory>
            <!-- 是否开启过滤替换配置，默认是不开启的 -->
            <filtering>true</filtering>
            <!-- 通配符写法：**匹配任意文件路径，*匹配任意个字符 -->
            <includes>
            	<include>**/jdbc.properties</include>
            </includes>
            <excludes>
                <exclude>**/const.properties</exclude>
            </excludes>
        </resource>
        <!-- const.properties，但不执行内容替换 -->       
        <resource>
            <directory>${project.basedir}/src/main/resources</directory>
            <filtering>false</filtering>
            <includes>
                <include>**/const.properties</include>
            </includes>
        </resource>
    </resources>

    <testResources>
        <testResource>
            <!-- 指定资源文件的目录 -->
            <directory>${project.basedir}/src/test/resources</directory>
            <!-- 是否开启过滤替换配置，默认是不开启的 -->
            <filtering>true</filtering>
        </testResource>
    </testResources>

</build>
```

2. 可自定义替换的分隔符（默认${*}和@）

若分隔符形式为 #*# , 其中 * 表示属性名称，前后分隔符都一样，可简写为 # 。即 #*# 和 # 写法一样。

```xml
<plugins>
    <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <!-- 是否使用默认的分隔符，默认分隔符是${*}和@ ,这个地方设置为false，表示不启
        用默认分隔符配置-->
        <useDefaultDelimiters>false</useDefaultDelimiters>
        <!-- 自定义分隔符 -->
        <delimiters>
            <delimiter>$*$</delimiter>
            <delimiter>##</delimiter>
        </delimiters>
    </configuration>
    </plugin>
</plugins>
```

3. 完整pom示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
        <modelVersion>4.0.0</modelVersion>
    
        <groupId>com.javacode2018</groupId>
        <artifactId>b2b-account-service</artifactId>
        <version>1.0-SNAPSHOT</version>
    
        <properties>
            <!-- 指定资源文件复制过程中采用的编码方式 -->
            <encoding>UTF-8</encoding>
        </properties>
    
        <!-- 配置多套环境 -->
        <profiles>
            <!-- 开发环境使用的配置 -->
            <profile>
                <id>dev</id>
                <activation>
                	<activeByDefault>true</activeByDefault> <!-- 默认开启此环境配置 -->
                </activation>
                <properties>
                    <jdbc.url>dev jdbc url</jdbc.url>
                    <jdbc.username>dev jdbc username</jdbc.username>
                    <jdbc.password>dev jdbc password</jdbc.password>
                </properties>
            </profile>
            <!-- 测试环境使用的配置 -->
            <profile>
                <id>test</id>
                <properties>
                    <jdbc.url>test jdbc url</jdbc.url>
                    <jdbc.username>test jdbc username</jdbc.username>
                    <jdbc.password>test jdbc password</jdbc.password>
                </properties>
            </profile>
            <!-- 线上环境使用的配置 -->
            <profile>
                <id>prod</id>
                <properties>
                    <jdbc.url>prod jdbc url</jdbc.url>
                    <jdbc.username>prod jdbc username</jdbc.username>
                    <jdbc.password>prod jdbc password</jdbc.password>
                </properties>
            </profile>
        </profiles>
    
        <dependencies>
            <dependency>
                <groupId>com.javacode2018</groupId>
                <artifactId>b2b-account-api</artifactId>
                <version>${project.version}</version>
            </dependency>
        </dependencies>
    
        <build>
            <resources>
                <resource>
                    <directory>${project.basedir}/src/main/resources</directory>
                    <filtering>true</filtering>
                    <includes>
                    	<include>**/jdbc.properties</include>
                    </includes>
                </resource>
                <resource>
                    <directory>${project.basedir}/src/main/resources</directory>
                    <includes>
                    	<include>**/const.properties</include>
                    </includes>
                </resource>
            </resources>
            <testResources>
                <testResource>
                    <directory>${project.basedir}/src/test/resources</directory>
                    <filtering>true</filtering>
                </testResource>
            </testResources>
            
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>2.6</version>
                    <configuration>
                        <!-- 自定义分隔符 -->
                        <useDefaultDelimiters>false</useDefaultDelimiters>
                        <delimiters>
                            <delimiter>$*$</delimiter>
                            <delimiter>##</delimiter>
                        </delimiters>
                    </configuration>
                </plugin>
            </plugins>
        </build>
</project>
```

执行构建命令时，需添加参数：-Pdev    

```shell
mvn clean package -pl :b2b-account-service -Pdev
```

​	若不使用 -Pdev 参数，则使用默认开启的环境配置。若无默认环境配置，则不执行文件替换。

​	可在 -P 参数后跟多个环境id，逗号隔开。当使用多套环境时，maven属性会进行合并，若多套环境中属性有一样的，后面的会覆盖前面的。命令：

```shell
mvn clean package -pl :b2b-account-service -Pdev,test,prod 
```



**通过maven属性来控制环境的开启：**    

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
        <modelVersion>4.0.0</modelVersion>
    
        <groupId>com.javacode2018</groupId>
        <artifactId>b2b-account-service</artifactId>
        <version>1.0-SNAPSHOT</version>
    
        <properties>
            <!-- 指定资源文件复制过程中采用的编码方式 -->
            <encoding>UTF-8</encoding>
        </properties>
    
        <!-- 配置多套环境 -->
        <profiles>
            <!-- 开发环境使用的配置 -->
            <profile>
                <id>dev</id>
                <activation>
                	<activeByDefault>true</activeByDefault>
                    <property>
                        <name>env</name>
                        <value>env_dev</value>
                    </property>
                </activation>
                <properties>
                    <jdbc.url>dev jdbc url</jdbc.url>
                    <jdbc.username>dev jdbc username</jdbc.username>
                    <jdbc.password>dev jdbc password</jdbc.password>
                </properties>
            </profile>
            <!-- 测试环境使用的配置 -->
            <profile>
                <id>test</id>
                <activation>
                    <property>
                        <name>env</name>
                        <value>env_test</value>
                    </property>
                </activation>
                <properties>
                    <jdbc.url>test jdbc url</jdbc.url>
                    <jdbc.username>test jdbc username</jdbc.username>
                    <jdbc.password>test jdbc password</jdbc.password>
                </properties>
            </profile>
            <!-- 线上环境使用的配置 -->
            <profile>
                <id>prod</id>
                <activation>
                    <property>
                        <name>env</name>
                        <value>env_prod</value>
                    </property>
                </activation>
                <properties>
                    <jdbc.url>prod jdbc url</jdbc.url>
                    <jdbc.username>prod jdbc username</jdbc.username>
                    <jdbc.password>prod jdbc password</jdbc.password>
                </properties>
            </profile>
        </profiles>
</project>
```

​	执行构建命令时：

```shell
mvn clean package -pl :b2b-account-service -Denv=env_prod  
```

## 8.3、集中参数配置到统一配置文件

1. 新建配置文件

**b2b/config/dev.properties**

```properties
# b2b-account-service jdbc配置信息
b2b-account-service.jdbc.url=dev_account_jdbc_url
b2b-account-service.jdbc.username=dev_account_jdbc_username
b2b-account-service.jdbc.password=dev_account_jdbc_password
# b2b-order-service jdbc配置信息
b2b-order-service.jdbc.url=dev_order_jdbc_url
b2b-order-service.jdbc.username=dev_order_jdbc_username
b2b-order-service.jdbc.password=dev_order_jdbc_password
```

**b2b/config/test.properties**

```properties
# b2b-account-service jdbc配置信息
b2b-account-service.jdbc.url=test_account_jdbc_url
b2b-account-service.jdbc.username=test_account_jdbc_username
b2b-account-service.jdbc.password=test_account_jdbc_password
# b2b-order-service jdbc配置信息
b2b-order-service.jdbc.url=test_order_jdbc_url
b2b-order-service.jdbc.username=test_order_jdbc_username
b2b-order-service.jdbc.password=test_order_jdbc_password
```

**b2b/config/prod.properties**

```properties
# b2b-account-service jdbc配置信息
b2b-account-service.jdbc.url=prod_account_jdbc_url
b2b-account-service.jdbc.username=prod_account_jdbc_username
b2b-account-service.jdbc.password=prod_account_jdbc_password
# b2b-order-service jdbc配置信息
b2b-order-service.jdbc.url=prod_order_jdbc_url
b2b-order-service.jdbc.username=prod_order_jdbc_username
b2b-order-service.jdbc.password=prod_order_jdbc_password
```

2. 配置pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.javacode2018</groupId>
    <artifactId>b2b-order-service</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <dependencies>
    	<dependency>
            <groupId>${project.groupId}</groupId>
            <artifactId>b2b-account-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>${project.groupId}</groupId>
            <artifactId>b2b-order-api</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
    
    <properties>
        <encoding>UTF-8</encoding>
    </properties>
    
    <!-- 配置多套环境 -->
    <profiles>
        <!-- 开发环境使用的配置 -->
        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
                <property>
                    <name>env</name>
                    <value>env_dev</value>
                </property>
            </activation>
            <build>
                <filters>
                    <filter>../../config/dev.properties</filter>
                </filters>
            </build>
        </profile>
        
        <!-- 测试环境使用的配置 -->
        <profile>
            <id>test</id>
            <activation>
                <property>
                    <name>env</name>
                    <value>env_test</value>
                </property>
            </activation>
            <build>
                <filters>
                	<filter>../../config/test.properties</filter>
                </filters>
            </build>
        </profile>
        
        <!-- 线上环境使用的配置 -->
        <profile>
            <id>prod</id>
            <activation>
                <property>
                    <name>env</name>
                    <value>env_prod</value>
                </property>
            </activation>
            <build>
                <filters>
                    <filter>../../config/prod.properties</filter>
                </filters>
            </build>
        </profile>
    </profiles>
    
    <build>
        <resources>
            <resource>
                <directory>${project.basedir}/src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
        <testResources>
            <testResource>
                <directory>${project.basedir}/src/test/resources</directory>
                <filtering>true</filtering>
            </testResource>
        </testResources>
    </build>
</project>
```

​	profile元素可用于对不同环境的构建进行配置，project中包含的元素，在profile元素中基本上都有， profile可以定制更复杂的构建过程，不同的环境依赖的构件、插件、build过程、测试过程都是不一样的，这些都可以在profile中进行指定。

## 8.4、maven环境相关命令

```shell
## 查看目前有哪些环境
mvn help:all-profiles
## 查看目前激活的是哪些环境
mvn help:active-profiles
```



# 九、自定义插件

## 9.1、自定义插件步骤

1. 新建一个maven项目A，pom.xml中的packageing元素值为pom
2. 新建一个maven模块A-1，pom.xml中的packageing元素值为maven-plugin

```xml
<packaging>maven-plugin</packaging>
```

3. 为模块A-1导入maven插件依赖    

```xml
<dependency>
    <groupId>org.apache.maven</groupId>
    <artifactId>maven-plugin-api</artifactId>
    <version>3.0</version>
</dependency>

<dependency>
    <groupId>org.apache.maven.plugin-tools</groupId>
    <artifactId>maven-plugin-annotations</artifactId>
    <version>3.4</version>
    <scope>provided</scope>
</dependency>
```

4. 创建一个目标类，在类上添加@Mojo注解 ；继承 org.apache.maven.plugin.AbstractMojo，重写 execute 方法：

```java
import org.apache.maven.plugin.AbstractMojo;
import org.apache.maven.plugin.MojoExecutionException;
import org.apache.maven.plugin.MojoFailureException;
import org.apache.maven.plugins.annotations.Mojo;

@Mojo(name = "demo1")
public class Demo1Mojo extends AbstractMojo {
    
    /**
     * 通过mvn命令的-D传入参数，如：mvn ... -Dsayhi.greeting=Tom
     * 通过<execution> - <configuration>标签引入参数，如：<sayhi.greeting>Tom</sayhi.greeting>
     **/
    @Parameter( property = "sayhi.greeting", defaultValue = "Hello World!" )
	private String greeting；
        
    // <myBoolean>true</myBoolean>
    @Parameter
    private boolean myBoolean;
    
    // <myInteger>10</myInteger>
    @Parameter
    private Integer myInteger;
    
    // <myFile>c:\temp</myFile>
    @Parameter
    private File myFile;
    
    public void execute() throws MojoExecutionException, MojoFailureException {
        this.getLog().info("hello my first maven plugin!");
    }
}
```

6. 安装插件到本地仓库：在插件的pom.xml所在目录执行下面命令 

```shell
mvn clean install -pl :demo1-maven-plugin
```

## 9.2、自定义插件前缀  

​	若自定义插件artifactId满足格式：xxx-maven-plugin ，则maven会自动将 xxx 指定为插件的前缀。此格式为最常用合适，其他格式也可以 。

​	maven默认会在仓库 "org.apache.maven.plugins" 和 "org.codehaus.mojo" 2个位置查找插件。

​	若自定义插件也能被maven找到，则需要 **在 ~/.m2/settings.xml 中配置自定义插件组** :

        ```xml	
<pluginGroups>
    <pluginGroup>com.javacode2018</pluginGroup>
</pluginGroups>
        ```



#十、pomxml文件相关

## 10.1、项目默认pom.xml的父类

​	pom.xml默认会继承maven顶级的一个父类pom.xml，顶级的pom.xml中指定了很多默认的配 置，如生命周期中的阶段和很多插件的绑定。

```shell
# 查看完整的pom.xml文件
mvn help:effective-pom
# 将完整的pom.xml文件输入到指定文件
mvn help:effective-pom > tmp_pom.xml
```

 ## 10.2、pom.xml示例

```xml
<!-- 设置编译代码时，资源、测试资源文件的拷贝时，使用的编码  -->
<properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

#十一、其他

- 设置maven编译时文件拷贝编码

```shell
# pom.xml中2种：
<encoding>UTF-8</encoding>
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

# mvn命令中2种：
mvn compile -Dencoding=UTF-8
mvn compile -Dproject.build.sourceEncoding=UTF-8
```

- mvn运行测试用例时，对测试用例类名的要求：

```java
// org.apache.maven.plugin.surefire.SurefirePlugin    
protected String[] getDefaultIncludes()  {
    return new String[]{ "**/Test*.java", "**/*Test.java", "**/*TestCase.java"
}
```

