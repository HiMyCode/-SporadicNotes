[toc]

# 一、Elasticsearch安装

## 1.1、环境说明

- Mac M1
- lasticsearch:7.13.0
- JDK 1.8（7.8+ 版本要求 JDK1.8+）

## 1.2、软件安装 & 启动

### 1.2.1、基于Docker

1. 拉取镜像

```shell
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.13.0
```

2. 单节点启动

```
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.13.0
```

3. 集群启动

（1）创建 **docker-compose.yml**

```yaml
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.13.0
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.13.0
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
    networks:
      - elastic
  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.13.0
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data03:/usr/share/elasticsearch/data
    networks:
      - elastic

volumes:
  data01:
    driver: local
  data02:
    driver: local
  data03:
    driver: local

networks:
  elastic:
    driver: bridge
```

（2）执行命令并测试启动状态

```shell
docker-compose up
## 命令行测试
curl -X GET "localhost:9200/_cat/nodes?v=true&pretty"
## 网页访问测试：http://localhost:9200
```

（3）关闭应用

```shell
docker-compose down
## 显示关闭详细日志
docker-compose down -v
```

**端口说明：**

- 9200：为浏览器访问的http协议RESTful端口
- 9300：为Elasticsearch集群间组件的通信端口

### 1.2.2、基于tar

1. 下载地址：https://www.elastic.co/cn/downloads/elasticsearch 

   下载版本：elasticsearch-7.13.0-darwin-x86_64.tar

2. 安装命令

   ```shell
   ## 解压
   tar zxvf ./elasticsearch-7.6.2-darwin-x86_64.tar ./elasticsearch-7.6.2
   ## 启动
   cd ./elasticsearch-7.6.2/bin
   ./elasticsearch
   ## 访问测试
   curl localhost:9200
   ```

   若Elastic正常运行，将范围如下包括当前节点、集群、版本等信息的结果。

   ```json
   {
     "name" : "atntrTf",
     "cluster_name" : "elasticsearch",
     "cluster_uuid" : "tf9250XhQ6ee4h7YI11anA",
     "version" : {
       "number" : "5.5.1",
       "build_hash" : "19c13d0",
       "build_date" : "2017-07-18T20:44:24.823Z",
       "build_snapshot" : false,
       "lucene_version" : "6.6.0"
     },
     "tagline" : "You Know, for Search"
   }
   ```

   安装分词器：

   ```shell
   ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.6.2/elasticsearch-analysis-ik-7.6.2.zip
   ```

## 1.3、安装 Q&A

1. 若启动报错："max virtual memory areas vm.max*map*count [65530] is too low"，则允许如下命令：

   ```shell
   sudo sysctl -w vm.max_map_count=262144
   ```

2. Elastic访问限制问题

   默认Elastic 只允许本机访问，若要远程访问，需修改 Elastic 配置文件：

   `./elasticsearch-7.6.2/config/elasticsearch.yml`

   去掉`network.host`的注释，将它的值改成`0.0.0.0`，然后重新启动 Elastic。

   ```shell
   ## 去掉 network.host 前的注释，并将IP改为 0.0.0.0（允许任何人访问），生产应设成为具体的 IP。
   network.host: 0.0.0.0
   ```


## 二、Elasticsearch基本概念

## 2.1、Node 与 Cluster

Elasticsearch 本质是一个分布式数据库，允许多台服务器协同工作，每台服务器可运行多个 Elastic 实例。

- Node：单个 Elasticsearch 实例称为一个节点
- Cluster：一组Elasticsearch节点构成一个集群

## 2.2、Document

Elasticsearch中的基本存储单位，每一条记录就称为Document（文档）；

多条 Document 构成了一个 Index。

同一 Index 中的 Document可以有不同的结构（scheme），但保持相同有利于提高搜索效率；

Elasticsearch的**Document**相当于Mysql数据库中表的 **行**。

## 2.3、Type

Type用于对Elasticsearch中的Document作虚拟的逻辑分组；

**Elasticsearch 6.x** 版只允许每个 Index 包含一个 Type，**7.x** 版将会彻底移除 Type；

Elasticsearch的**Type**相当于Mysql数据库中的 **表**。

## 2.4、Index

Elasticsearch 会索引所有字段，经过处理后保存一个反向索引（Inverted Index）。查找数据时将使用该索引；

Elasticsearch的**Index**相当于Mysql数据库中的 **库**。







# 参考资料

https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

