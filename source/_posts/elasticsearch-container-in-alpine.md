---
title: Docker下配置Elasticsearch容器过程记录
date: 2017-03-22 10:42:06
tags:
    - ElasticSearch
    - Docker
description: Docker下配置Elasticsearch容器过程记录
---

主要记录 Elasticsearch 容器创建流程

#### 目标

* Elasticsearch 2.4.x
* Elasticsearch 5.x


#### alpine 下配置java环境

参考自 `https://github.com/docker-library/openjdk/blob/master/8-jdk/alpine/Dockerfile`

后期创建一个java环境的基础容器。

***

#### 测试记录

下载 `elasticsearch.tar.gz` 文件并解压到指定目录：

```
$ curl -sSL https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.4.0/elasticsearch-2.4.0.tar.gz | tar -zxv -C /usr/share/elasticsearch --strip-components 1
```

#### 只在 `docker run` 时执行安装 `elasticsearch-analysis-ik` 插件

在 dockerfile 中的 `ENTRYPOINT` 命令会在 `docker run` 或 `docker restart` 时都执行，这样就导致重复安装插件的问题。

目前采用的方法是在 `shell script` 文件中 通过判断 `plugins` 目录下是否存在 `elasticsearch-analysis-ik-${IK_ANALYSIS_VERSION}.zip` 文件来判断是否已经添加该插件。

另一种方式: 考虑通过 `docker exec` 在容器启动后来安装。

使用 `docker exec` 命令执行 `shell script` 时，有一个疑惑：需要执行的 `shell script` 文件应该放在容器内部还是可以调用外部即宿主机上的。(后经测试，放在容器内部。)

***

#### Elasticsearch 集群


其中有一台被作为Master，其他为Slave。

配置 `elasticsearch.yml` （**注意配置文件冒号“：”后要有一个空格，否则报错**）

```
network.host: 10.0.0.1
cluster.name: elasticsearch    # 集群名称
node.name: es-node1    # 节点名称 （同一集群下多个节点名称不同）
bootstrap.mlockall: true  # 
node.master: true  # 是否作为主节点，每个节点都可以被配置成为主节点，默认值为true
node.data: true  # 是否存储数据，即存储索引片段，默认值为true


discovery.zen.ping.unicast.hosts: ["10.0.0.1:9300", "10.0.0.2:9300"]
discovery.zen.minimum_master_nodes: 2
```


```
network.host: ["127.0.0.1", "[::1]"]
cluster.name: elasticsearch
node.name: ${HOSTNAME} # es_node_1
discovery.zen.ping.unicast.hosts: ["10.0.0.1", "10.0.0.2", "10.0.0.3"]

node.master: true
node.data: false
```

***

#### 如何指定自定义配置文件

##### 自定义配置文件和默认配置文件的关系

指定 配置文件路径：  `./bin/elasticsearch -Des.path.conf=/my/conf.yaml`

注意： **Des.config=/path/to/config/file doesn't replace $ES_HOME/elasticsearch.conf, just appends to it**

详见：[-Des.config=/path/to/config/file doesn&#39;t replace $ES_HOME/elasticsearch.conf, just appends to it · Issue #588 · elastic/elasticsearch · GitHub](https://github.com/elastic/elasticsearch/issues/588)


ES 2.4.0 

使用如下方式启动 `CMD gosu elasticsearch elasticsearch -Epath.conf="${ELASTICSEARCH_DATA}/config/elastic.yml"` 报下面的错误：

```
ERROR: Parameter [-Epath.conf="${ELASTICSEARCH_DATA}/config/elastic.yml"]does not start with --
```


```
-Des.path.conf

bin/elasticsearch -Des.path.conf=/etc/elasticsearch/node1
```

**总结** ：使用 `--path.conf=configdir` 来指定配置文件所在目录，并且配置文件必需命名为 `elasticsearch.yml` 才可以。

```
$ ./bin/elasticsearch --path.conf=/path/to/conf/dir
```

详细分析过程如下：

##### 报错分析

启动时报如下错误：

```
/ # gosu elasticsearch elasticsearch -Des.path.conf=/app/config/elastic.yml
log4j:WARN No appenders could be found for logger (bootstrap).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
Exception in thread "main" java.lang.IllegalStateException: Unable to access 'path.conf' (/app/config/elastic.yml)
Likely root cause: java.nio.file.NotDirectoryException: /app/config/elastic.yml
	at org.elasticsearch.bootstrap.Security.ensureDirectoryExists(Security.java:340)
	at org.elasticsearch.bootstrap.Security.addPath(Security.java:314)
	at org.elasticsearch.bootstrap.Security.addFilePermissions(Security.java:247)
	at org.elasticsearch.bootstrap.Security.createPermissions(Security.java:212)
	at org.elasticsearch.bootstrap.Security.configure(Security.java:118)
	at org.elasticsearch.bootstrap.Bootstrap.setupSecurity(Bootstrap.java:212)
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:183)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:286)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:35)
Refer to the log for complete error details.
```

解决方法：

找到一个讨论中的回答：

```
It is no longer possible to specify a custom config file with the CONF_FILE environment variable, or the -Des.config, -Des.default.config, or -Delasticsearch.config parameters.

Instead, the config file must be named elasticsearch.yml and must be located in the default config/ directory, unless a custom config directory is specified.
```

* [Problem in starting a node in ES 2.2.1 - Elasticsearch - Discuss the Elastic Stack](https://discuss.elastic.co/t/problem-in-starting-a-node-in-es-2-2-1/45082/5)
* [Setting changes        | Elasticsearch Reference 2.2 | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/breaking_20_setting_changes.html#_custom_config_file)

找到解决方案：

从文章 [Setting changes | Elasticsearch Reference 2.4](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/breaking_20_setting_changes.html#_custom_config_file) 中这段：

> Custom config fileedit
> It is no longer possible to specify a custom config file with the CONF_FILE environment variable, or the -Des.config, -Des.default.config, or -Delasticsearch.config parameters.
> Instead, the config file must be named elasticsearch.yml and must be located in the default config/ directory, unless a custom config directory is specified.
> The location of a custom config directory may be specified as follows:
> ./bin/elasticsearch --path.conf=/path/to/conf/dir
> ./bin/plugin -Des.path.conf=/path/to/conf/dir install analysis-icu
> When using the RPM or debian packages, the plugin script and the init/service scripts will consult > the CONF_DIR environment variable to check for a custom config location. The value of the CONF_DIR > variable can be set in the environment config file which is located either in /etc/default/elasticsearch or /etc/sysconfig/elasticsearch.


得知，之前的自定义配置文件方法 `-Des.path.conf` 改为了这种方式 `--path.conf` ，而且，只需要指定到配置文件所在目录即可。另外，配置文件的名称必须为 `elasticsearch.yml` 。

```
$ gosu elasticsearch elasticsearch --path.conf=/app/config
```

***

#### network.host 同时支持 IPV4 和 IPV6 报错

报如下错误：

```
Exception in thread "main" java.lang.IllegalArgumentException: bind address: {0.0.0.0} is wildcard, but multiple addresses specified: this makes no sense
	at org.elasticsearch.common.network.NetworkService.resolveBindHostAddresses(NetworkService.java:132)
	at org.elasticsearch.transport.netty.NettyTransport.bindServerBootstrap(NettyTransport.java:435)
	at org.elasticsearch.transport.netty.NettyTransport.doStart(NettyTransport.java:332)
	at org.elasticsearch.common.component.AbstractLifecycleComponent.start(AbstractLifecycleComponent.java:68)
	at org.elasticsearch.transport.TransportService.doStart(TransportService.java:182)
	at org.elasticsearch.common.component.AbstractLifecycleComponent.start(AbstractLifecycleComponent.java:68)
	at org.elasticsearch.node.Node.start(Node.java:278)
	at org.elasticsearch.bootstrap.Bootstrap.start(Bootstrap.java:222)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:288)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:35)
```

配置文件内容：

```
# default
network.host: ["0.0.0.0", "[::1]"]
path.data: /app/data
path.logs: /app/logs

# cluster
cluster.name: elasticsearch
node.name: ${HOSTNAME}
node.master: true
node.data: true
# bootstrap.mlockall: true
discovery.zen.ping.unicast.hosts: ["localhost:9300", "localhost:9301"]
```

改为 `network.host: 0.0.0.0` 后 启动成功。

可查看该讨论: [Binding to the wildcard of one address family should not prevent binding to addresses in another family · Issue #20703 · elastic/elasticsearch · GitHub](https://github.com/elastic/elasticsearch/issues/20703)

***

#### 当加上 `bootstrap.mlockall: true` 这行时，也会报错。 待探究原因！！！


***

#### elasticsearch cluster 配置 unicast.hosts

##### 探究

上面的配置文件中，设置的 `unicast.hosts` 值如下 ：

```
discovery.zen.ping.unicast.hosts: ["localhost:9300", "localhost:9301"]


discovery.zen.ping.unicast.hosts: ["172.17.0.4:9300", "172.17.0.5:9301"]
```

经测试，使用 `["localhost:9300", "localhost:9301"]` 无效。必须使用明确的IP地址 `["172.17.0.4:9300", "172.17.0.5:9301"]` 后才能互相发现。

通过 访问 `http://59.188.76.112:9200/_cluster/health?pretty` 能看到是否组成集群模式：

```
{
    "cluster_name": "elasticsearch",
    "status": "green",
    "timed_out": false,
    "number_of_nodes": 2,
    "number_of_data_nodes": 2,
    "active_primary_shards": 0,
    "active_shards": 0,
    "relocating_shards": 0,
    "initializing_shards": 0,
    "unassigned_shards": 0,
    "delayed_unassigned_shards": 0,
    "number_of_pending_tasks": 0,
    "number_of_in_flight_fetch": 0,
    "task_max_waiting_in_queue_millis": 0,
    "active_shards_percent_as_number": 100
}
```

考虑到Docker下容器的ip地址是随机的，容器重启后可能会改变，需要考虑其他确定的方式。

设置成 `localhost` 参考自 [如何使用一个IP搭建ES集群——Docker如你所愿 - 简书](http://www.jianshu.com/p/265db8b05d85#)


可能的一种方法是在 `docker run` 时 指定 `--add-host=[]` 参数  ，待研究！！！


经测试，使用 `localhsot`  `127.0.0.1`  `0.0.0.0`  `docker name` 均无效。

经测试，可以设置为相应容器的IP 如 `172.17.0.5:9300` 。通过命令 :

```
docker inspect --format='{{.NetworkSettings.IPAddress}}' $CONTAINER_ID
```

可以获得指定容器的ip地址。但这里的问题是 `容器的ip每次重启后都会改变，知道容器ip没什么意义。`
所以这种方式也失败。

如果不指定容器的IP,但可以指定容器所在宿主机的IP `120.121.xx.xx:9300` ，这样能发现集群。

***

##### 以下issure中，可以通过docker-compores 来指定容器ip ：待研究！！！

* [Cannot set up an elasticsearch:2 cluster in docker any more · Issue #68 · docker-library/elasticsearch · GitHub](https://github.com/docker-library/elasticsearch/issues/68)
* [Problems in getting single node cluster working as per your docs · Issue #19 · spujadas/elk-docker · GitHub](https://github.com/spujadas/elk-docker/issues/19)


***

##### 另外一种方式，通过 `--link` 来设置：

参考自：[dockerfiles/elasticsearch at master · itzg/dockerfiles · GitHub](https://github.com/itzg/dockerfiles/tree/master/elasticsearch)

```
docker run --name es5 -d -p 9230:9200 -p 9330:9300 e2
docker run --name es6 -d --link es5 e2
docker run --name es7 -d --link es5 e2

docker cp elasticsearch.yml es5:/app/config/elasticsearch.yml
docker cp elasticsearch.yml es6:/app/config/elasticsearch.yml
docker cp elasticsearch.yml es7:/app/config/elasticsearch.yml

docker restart es5 es6 es7
```

配置文件：

```
# default
network.host: 0.0.0.0
path.data: /app/data
path.logs: /app/logs

# cluster
cluster.name: elasticsearch
node.name: ${HOSTNAME}
node.master: true
node.data: true
discovery.zen.ping.unicast.hosts: ["es5:9330"]
```

**这种方式经测试，无效。** 报如下错误：

```
java.net.NoRouteToHostException: Host is unreachable
	at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
	at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:717)
	at org.jboss.netty.channel.socket.nio.NioClientBoss.connect(NioClientBoss.java:152)
	at org.jboss.netty.channel.socket.nio.NioClientBoss.processSelectedKeys(NioClientBoss.java:105)
	at org.jboss.netty.channel.socket.nio.NioClientBoss.process(NioClientBoss.java:79)
	at org.jboss.netty.channel.socket.nio.AbstractNioSelector.run(AbstractNioSelector.java:337)
	at org.jboss.netty.channel.socket.nio.NioClientBoss.run(NioClientBoss.java:42)
	at org.jboss.netty.util.ThreadRenamingRunnable.run(ThreadRenamingRunnable.java:108)
	at org.jboss.netty.util.internal.DeadLockProofWorker$1.run(DeadLockProofWorker.java:42)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
```

***

#### Docker elasticsearch 集群

* https://hub.docker.com/r/itzg/elasticsearch/
* https://github.com/Khezen/docker-elasticsearch/tree/2.4


看到网上大部分人都是将集群配置参数通过环境变量的形式在 `docker run` 时配置，考虑将配置文件暴露在主机目录下是否安全？


***

#### 查看集群状态

查看集群状态:  `curl -XGET 'http://172.16.212.102:9200/_cluster/health?pretty' `

***

#### 相关参考

##### Elasticsearch集群搭建相关参考

* [ElasticSearch集群部署文档](http://www.wklken.me/posts/2016/06/29/deploy-es.html)
* [elasticsearch2.3.1 集群安装 - 尚浩宇的博客](https://my.oschina.net/shyloveliyi/blog/653751) **五角星 配置**
* [How To Set Up a Production Elasticsearch Cluster on Ubuntu 14.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-production-elasticsearch-cluster-on-ubuntu-14-04)
* [Easy Elasticsearch cluster with docker 1.12 swarm? - General - Docker Forums](https://forums.docker.com/t/easy-elasticsearch-cluster-with-docker-1-12-swarm/19648/12)
* [Running an Elasticsearch cluster with Docker](https://stefanprodan.com/2016/elasticsearch-cluster-with-docker/)
* [如何使用一个IP搭建ES集群——Docker如你所愿 - 简书](http://www.jianshu.com/p/265db8b05d85) **五角星**
* [docker-elasticsearch-cn/docker-start at master · hangxin1940/docker-elasticsearch-cn · GitHub](https://github.com/hangxin1940/docker-elasticsearch-cn/blob/master/docker-start)
* [elasticsearch 集群](http://forkday.com/elastic-search-ji-qun/)
* [宿主机上如何获得 docker container 容器的 ip 地址？ - SegmentFault](https://segmentfault.com/q/1010000001637726)

***

##### Elasticsearch容器创建相关参考

* [GitHub - soldair/docker-alpine-elasticsearch: 130mb ish Elasticsearch container based on alpine linux.](https://github.com/soldair/docker-alpine-elasticsearch)
* [GitHub - docker-library/elasticsearch: Docker Official Image packaging for elasticsearch](https://github.com/docker-library/elasticsearch)
* [GitHub - kiasaki/docker-alpine-elasticsearch: ElasticSearch container based on Alpine Linux](https://github.com/kiasaki/docker-alpine-elasticsearch)
* [GitHub - docker-library/openjdk: Docker Official Image packaging for Java (openJDK)](https://github.com/docker-library/openjdk)
* [docker elasticsearch](https://github.com/Khezen/docker-elasticsearch)
* [GitHub - itzg/dockerfiles: Contains the various Dockerfile definitions I&#39;m maintaining.](https://github.com/itzg/dockerfiles)

***

#### 成果

源码见：[GitHub - Leafney/alpine-elasticsearch: Docker + Alpine + Elasticsearch](https://github.com/Leafney/alpine-elasticsearch)
