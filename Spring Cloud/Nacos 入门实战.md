

# 一、软件安装(Mac环境)

## 环境准备

Nacos 依赖 [Java](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/) 环境来运行，请确保是在以下版本环境中安装使用：

1. 64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac。
2. 64 bit JDK 1.8+；[下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) & [配置](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/)。
3. Maven 3.2.x+；[下载](https://maven.apache.org/download.cgi) & [配置](https://maven.apache.org/settings.html)。

## 软件下载

- 下载编译后的压缩包

  https://github.com/alibaba/nacos/releases

- 从 Github 上下载源码方式

  ```shell
  git clone https://github.com/alibaba/nacos.git
  cd nacos/
  mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U  
  ls -al distribution/target/
  
  ## change the $version to your actual path
  cd distribution/target/nacos-server-$version/nacos/bin
  ```






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

