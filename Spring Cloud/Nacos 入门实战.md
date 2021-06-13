 

# 一、服务治理简介

​	服务治理是微服务架构中最核心最基本的模块。用于实现各个微服务的自动化注册与发现。

- **服务注册**

  ​	在服务治理框架中，都会构建一个注册中心，每个服务单元向注册中心登记自己提供服务的详细信息。并在注册中心形成一张服务的清单，服务注册中心需要以心跳的方式去监测清单中的服务是否可用，如果不可用，需要在服务清单中剔除不可用的服务。

- **服务发现**

  ​	服务调用方向服务注册中心咨询服务，并获取所有服务的实例清单，实现对具体服务实例的访问。

   **服务注册中心的作用：**

- 服务发现：
  - 服务注册：保存服务提供者和服务调用者的信息
  - 服务订阅：服务调用者订阅服务提供者的信息，注册中心向订阅者推送提供者的信息
- 服务配置：
  - 配置订阅：服务提供者和服务调用者订阅微服务相关的配置
  - 配置下发：主动将配置推送给服务提供者和服务调用者
3. 服务健康检测
- 检测服务提供者的健康情况，如果发现异常，执行服务剔除



# 二、常见的注册中心

- **Zookeeper**
  	Zookeeper是Apache Hadoop的子项目，分布式服务框架，主要用来解决分布式应用中的数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项管理等。

- **Eureka**
  Eureka是Springcloud Netflix中的组件，主要做服务注册和发现，2.x后闭源。

- **Consul**

  Consul基于GO语言开发的开源工具，主要面向分布式，服务化的系统提供服务注册、服务发现
  和配置管理的功能。Consul的功能都很实用，其中包括：服务注册/发现、健康检查、Key/Value
  存储、多数据中心和分布式一致性保证等特性。Consul本身只是一个二进制的可执行文件，所以
  安装和部署都非常简单，只需要从官网下载后，在执行对应的启动脚本即可。

- **Nacos**

  Nacos是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。它是 Spring
  Cloud Alibaba 组件之一，负责服务注册发现和服务配置，可以这样认为nacos=eureka+config。

# 三、软件安装

**1、环境准备**

Nacos 依赖 [Java](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/) 环境来运行，请确保是在以下版本环境中安装使用：

1. 64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac。

2. 64 bit JDK 1.8+；[下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) & [配置](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/)。

3. Maven 3.2.x+；[下载](https://maven.apache.org/download.cgi) & [配置](https://maven.apache.org/settings.html)。

   

**2、Nacos Docker 快速开始**

参考链接：https://nacos.io/zh-cn/docs/quick-start-docker.html

- Clone 项目

  ```shell
  git clone https://github.com/nacos-group/nacos-docker.git
  cd nacos-docker
  ```

- 单机模式 Derby

  ```shell
  docker-compose -f example/standalone-derby.yaml up
  ```

- 下载压缩包地址

  https://github.com/alibaba/nacos/releases

  https://github.com/alibaba/nacos/releases/download/1.4.1/nacos-server-1.4.1.tar.gz

  









```jsp
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>table</title>
<style>
  tr,th {
    padding: 30px;
  }
</style>
</head>
<body>
  <table id="myTable" border="1" >
    <thead>
      <tr>
        <th>序列</th>
        <th>名称</th>
        <th>单价</th>
        <th>数量</th>
      </tr>
    </thead>
    <tbody>
      <tr draggable="true">
        <td>1</td>
        <td>冰箱</td>
        <td>13</td>
        <td>14</td>
      </tr>
      <tr draggable="true">
        <td>2</td>
        <td>彩电</td>
        <td>113</td>
        <td>114</td>
      </tr>
      <tr draggable="true">
        <td>3</td>
        <td>空调</td>
        <td>12</td>
        <td>12</td>
      </tr>
      <tr draggable="true">
        <td>4</td>
        <td>洗衣机</td>
        <td>12</td>
        <td>12</td>
      </tr>
    </tbody>
  </table>
  
  <script>
    function drag_row_table() {
      var EventUtil = {  //跨浏览器的事件处理程序,视情况分别选择以下事件处理程序
        addHandler: function(element, type, handler) {
          if(element.addEventListener) {  //DOM2级
            element.addEventListener(type, handler, false);
          }
          else if (element.attachEvent) {  //IE方法
            element.attachEvent('on' + type, handler);
          }
          else {  //DOM1级
            element["on" + type] = handler;
          }
        },
        getTarget: function(event) {  //获取事件的实际目标（在这里是tr），不同于event.currentTarget和this指向的是绑定事件处理程序的目标，（tbody）
          return event.target || event.srcElement;
        },
        preventDefault: function(event) {  //取消事件默认方法
          if(event.preventDefault) {  
            event.preventDefault();
          }
          else {
            event.returnValue = false;  //默认为true，设为falsew为取消默认行为。
          }
        }
      }

      var myTable = document.querySelector('#myTable');
      var tbody = myTable.querySelector("tbody");  //这里监听的是tbody，而不是tr。目的是利用事件冒泡，减少监听的元素个数，提高性能
      var trs = tbody.querySelectorAll("tr");  

      EventUtil.addHandler(tbody, "dragover", function(event) { //默认元素不可放置，这里取消默认，将放置目标修改为可放置的
        EventUtil.preventDefault(event);  
      })

      EventUtil.addHandler(tbody, "dragstart", function(event) {  //监听点击的拖动的元素
      	event.dataTransfer.setData("drag_index", event.target.rowIndex); //将被拖元素的信息，传给放置位置
      })

      EventUtil.addHandler(tbody, "drop", function(event) {  //监听鼠标移动到可放置的放置目标上，松开鼠标的事件
	     EventUtil.preventDefault(event);  //取消在Firefox 3.5+中，放置事件的默认行为是打开被放置目标上的URL
	     let drag_index = parseInt(event.dataTransfer.getData("drag_index"));  //获取传过来的被拖元素的信息
	     let drop_index = EventUtil.getTarget(event).parentNode.rowIndex;  //this为当前触发drop的元素，即放置目标的行下标
	
	     tbody.insertBefore(trs[drag_index], EventUtil.getTarget(event).parentNode.nextSibling);   //将被拖元素放到放置目标的兄弟节点上
	     trs = myTable.querySelectorAll("tr");  //更新变换的tr行信息
      })
    }
    drag_row_table();
  </script>
</body>
</html>
```



























- 参考资料

  官方地址：https://nacos.io/zh-cn/docs/quick-start.html

