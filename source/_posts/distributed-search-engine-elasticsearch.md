---
title: 分布式搜索引擎ElasticSearch初探
date: 2016-06-20 20:06:00
tags:
    - ElasticSearch
categories: 
    - 开发笔记
description: 分布式搜索引擎ElasticSearch初探
---

ElasticSearch是一个基于Lucene构建的开源，分布式，RESTful搜索引擎。


#### 任务点

* 在Ubuntu下安装
* 在Windows下安装
* 使用Restful API实现搜索引擎的CURD操作
* 在Python网站Flask下使用ElasticSearch实现文章搜索
* ElasticSearch的Python连接器：Pyes


#### 安装

Java (JVM) versionedit

Elasticsearch is built using Java, and requires at least Java 7 in order to run. Only Oracle’s Java and the OpenJDK are supported. The same JVM version should be used on all Elasticsearch nodes and clients.
We recommend installing the Java 8 update 20 or later, or Java 7 update 55 or later. Previous versions of Java 7 are known to have bugs that can cause index corruption and data loss. Elasticsearch will refuse to start if a known-bad version of Java is used.
The version of Java to use can be configured by setting the JAVA_HOME environment variable.


##### Linux(Ubuntu)下安装

###### Installing the oracle JDK

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
java -version
```

###### RPM based distributions

```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
sudo /bin/systemctl start elasticsearch.service
```

##### Windows下安装

从网站 [](https://www.elastic.co/downloads/elasticsearch) 下载windows下的msi安装包，
在cmd命令行进入安装目录，再进入 `bin` 目录，运行 `elasticsearch.bat` 。

*** 

#### Ubuntu下配置ElasticSearch环境实录

#### 安装Java环境

由于ElasticSearch的运行需要Java环境的支持，先安装java环境。
由于Ubuntu系统中由于授权问题，默认只安装了OpenJDK的包，通过`java`和`javac`可以看到：  
```
 * default-jdk
 * ecj
 * gcj-4.9-jdk
 * openjdk-8-jdk-headless
 * gcj-4.8-jdk
 * gcj-5-jdk
 * openjdk-9-jdk
```

根据ElasticSearch官方推荐安装方法：

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
java -version
```

如果出现如下提示信息，并有进度条显示，则说明运行顺利并正在下载所需文件：

```
HTTP request sent, awaiting response... 200 OK
Length: 181367942 (173M) [application/x-gzip]
Saving to: ‘jdk-8u91-linux-x64.tar.gz’

     0K ........ ........ ........ ........ ........ ........  1%  566K 5m8s
  3072K ........ ........ ........ ........ ........ ........  3% 1.15M 3m44s
  6144K ........ ........ ........ ........ ........ ........  5% 1.10M 3m16s
  9216K ........ ........ ........ ........ ........ ........  6% 1002K 3m5s
 12288K ........ ........ ........ ........ ........ ........  8%  715K 3m11s
 15360K ........ ........ ........ ........ ........ ........ 10%  624K 3m18s
 18432K ........ ........ ........ ........ ........ ........ 12%  417K 3m40s

```

否则可能由于网络原因或其他问题，会中断下载或报错。


##### 错误一：  

安装时出现错误：
```
sha256sum mismatch jdk-8u91-linux-x64.tar.gz
Oracle JDK 8 is NOT installed.
dpkg: 处理软件包 oracle-java8-installer (--configure)时出错：
 子进程 已安装 post-installation 脚本 返回错误状态 1
正在设置 gsfonts-x11 (0.24) ...
在处理时有错误发生：
 oracle-java8-installer
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

可以用该文章中介绍的方法解决，亲测有效：
http://www.miaoqiyuan.cn/p/ubuntu-e-sub-process-dpkg-returned-an-error-code

##### 错误二：

根据上面的方法执行完命令后，执行 `java -version` 时只会显示下列输出信息：

```
tiger@vbox:~$ java -version
程序 'java' 已包含在下列软件包中：
 * default-jre
 * gcj-4.9-jre-headless
 * gcj-5-jre-headless
 * openjdk-8-jre-headless
 * gcj-4.8-jre-headless
 * openjdk-9-jre-headless
请尝试：sudo apt install <选定的软件包>
```

即使是重复执行上面的安装方法，结果仍是如此：

```
tiger@vbox:~$ sudo apt-get install oracle-java8-installer
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
oracle-java8-installer 已经是最新版 (8u92+8u91arm-2~really8u91~webupd8~0)。
升级了 0 个软件包，新安装了 0 个软件包，要卸载 0 个软件包，有 196 个软件包未被升级。
tiger@vbox:~$ sudo apt-get install oracle-java8-set-default
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
oracle-java8-set-default 已经是最新版 (8u92+8u91arm-2~really8u91~webupd8~0)。
升级了 0 个软件包，新安装了 0 个软件包，要卸载 0 个软件包，有 196 个软件包未被升级。
```

**根本问题是什么**

转到目录` /usr/lib/jvm/` 下，可以看到一个名为 `java-8-oracle` 的目录，但查看该目录却发现里面是空的。所以虽然提示是安装成功了，但却没有可执行的文件，可能是在下载文件时出错了。

由于直接安装错误，下面尝试手动进行安装。


##### 什么情况下表示安装成功，什么情况下表示安装失败

通过 `java -version` 来查看

如果输出如下，则说明 Orancel JDK 安装失败：

```
root@ubuntu:~# java -version
The program 'java' can be found in the following packages:
 * default-jre
 * gcj-4.8-jre-headless
 * openjdk-7-jre-headless
 * gcj-4.6-jre-headless
 * openjdk-6-jre-headless
Try: apt-get install <selected package>

```

如果输出如下，则说明安装成功：

```
tiger@vbox:/usr/lib/jvm$ java -version
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
```

#### Ubuntu下手动安装JDK

1. 到Oracle官网下载对应当前系统的jdk版本 [Java SE Development Kit 8 Downloads](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) ：

```
// 下载对应版本jdk压缩包，这里选择 `Linux64`
wget http://download.oracle.com/otn-pub/java/jdk/8u91-b14/jdk-8u91-linux-x64.tar.gz
// 等待下载完成后，执行：
tar zxvf jdk-8u91-linux-x64.tar.gz
// 查看  `ls`
jdk1.8.0_91 jdk-8u91-linux-x64.tar.gz
```

将解压后的目录 `jdk1.8.0_91` 复制到 `/usr/lib/jvm` 目录里，
```
sudo cp -r jdk1.8.0_91/ /usr/lib/jvm
```

配置环境变量：

```
sudo vim /etc/profile
```

在文件的末尾添加以下内容：(注意对应自己的目录路径和jdk的版本号)

```
JAVA_HOME=/usr/lib/jvm/jdk1.8.0_91
export JRE_HOME=/usr/lib/jvm/jdk1.8.0_91/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```

修改完后，执行 `:wq` 退出vim ，使用`source`刷新一下：

```
source /etc/profile
```

然后执行：

```
java -version
//看到如下信息表示安装成功：
tiger@vbox:/usr/lib/jvm$ java -version
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)

//执行 `java`

tiger@vbox:/usr/lib/jvm$ java
用法: java [-options] class [args...]
           (执行类)
   或  java [-options] -jar jarfile [args...]
           (执行 jar 文件)
其中选项包括:
    -d32      使用 32 位数据模型 (如果可用)
    -d64      使用 64 位数据模型 (如果可用)
    -server   选择 "server" VM
                  默认 VM 是 server.

    -cp <目录和 zip/jar 文件的类搜索路径>
    -classpath <目录和 zip/jar 文件的类搜索路径>
                  用 : 分隔的目录, JAR 档案
                  和 ZIP 档案列表, 用于搜索类文件。
    -D<名称>=<值>
                  设置系统属性
    -verbose:[class|gc|jni]
                  启用详细输出
    -version      输出产品版本并退出
....
....

//执行 `javac`

tiger@vbox:/usr/lib/jvm$ javac
用法: javac <options> <source files>
其中, 可能的选项包括:
  -g                         生成所有调试信息
  -g:none                    不生成任何调试信息
  -g:{lines,vars,source}     只生成某些调试信息
  -nowarn                    不生成任何警告
  -verbose                   输出有关编译器正在执行的操作的消息
  -deprecation               输出使用已过时的 API 的源位置
  -classpath <路径>            指定查找用户类文件和注释处理程序的位置
  -cp <路径>                   指定查找用户类文件和注释处理程序的位置
  -sourcepath <路径>           指定查找输入源文件的位置
  -bootclasspath <路径>        覆盖引导类文件的位置
.... 
....
```

*** 

**下面的步骤未测试，仅供参考**

如果到这一步，任然不成功的话，则属于手动配置默认的JDK版本：

```
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java/bin/java 300  
sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java/bin/javac 300  
sudo update-alternatives --install /usr/bin/jar jar /usr/lib/jvm/java/bin/jar 300   
sudo update-alternatives --install /usr/bin/javah javah /usr/lib/jvm/java/bin/javah 300   
sudo update-alternatives --install /usr/bin/javap javap /usr/lib/jvm/java/bin/javap 300 

//然后执行
sudo update-alternatives --config java

// 在通过 java -version 来判断
java -version
```


参考 ：
* [Running as a Service on Linux        | Elasticsearch Reference [2.3]      | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-service.html)
* [ubuntu 13.04 安装 JDK - plinx - 博客园](http://www.cnblogs.com/plinx/archive/2013/06/01/3113106.html)
* [在 Ubuntu 系统上安装 Oracle Java 8 | 半瓶](http://www.orangeclk.com/2014/08/10/install-java-on-linux/)

***

#### 安装ElasticSearch

下载最新安装包：

从 [download Elasticsearch free](https://www.elastic.co/downloads/elasticsearch) 下载Linux的安装包：
```
// 下载
$ wget -c https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/zip/elasticsearch/2.3.3/elasticsearch-2.3.3.zip
// 解压安装
unzip elasticsearch-2.3.3.zip
```

进入解压后得到的目录 `elasticsearch-2.3.3` ，参考官网安装教程 [Setup](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html#setup-installation) 执行如下命令来运行:
```
$ bin/elasticsearch

```

如果上一步的java环境没有安装成功，执行该命令会报如下错误：

```
Could not find any executable java binary. Please install java in your PATH or set JAVA_HOME
```

如果没有出现错误，提示信息中输出 如 `[INFO] .....` 等信息，说明运行成功。

再新开一个命令行终端，执行：

```
tiger@vbox:~$ curl http://localhost:9200/
{
  "name" : "Rune",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.3.3",
    "build_hash" : "218bdf10790eef486ff2c41a3df5cfa32dadcfde",
    "build_timestamp" : "2016-05-17T15:40:04Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.0"
  },
  "tagline" : "You Know, for Search"
}
tiger@vbox:~$ 
```

出现上面的提示则说明ElasticSearch安装成功。


* [ElasticSearch - kid551 - 博客园](http://www.cnblogs.com/kid551/p/4273256.html)


*** 

默认情况下 `Elasticsearch` 的 `RESTful` 服务只有本机才能访问。也就是说无法从主机访问虚拟机中的服务。为了方便调试，可以修改 `/config/elasticsarch.yml` 文件，加入以下两行：

```
network.bind_host: "0.0.0.0"
network.publish_host: _non_loopback:ipv4_
```

但线上环境切忌不要这样配置，否则任何人都可以通过这个接口修改你的数据。

*** 

#### 安装 IK Analysis 处理中文搜索

Elasticsearch 自带的分词器会粗暴地把每个汉字直接分开，没有根据词库来分词。为了处理中文搜索，还需要安装中文分词插件。我使用的是 [elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik)，支持自定义词库。

首先，下载与 Elasticsearch 匹配的 elasticsearch-analysis-ik 插件：{2.3.3--1.9.3}

```
// 从github 下载内容
git clone https://github.com/medcl/elasticsearch-analysis-ik.git
cd elasticsearch-analysis-ik
// 进入指定的 tag v1.9.3
git checkout v1.9.3
// 当前目录
ls
config  LICENSE.txt  pom.xml  README.md  src
```

要编译 `elasticsearch-analysis-ik` ,需要安装 `Apache Maven` 工具：

```
sudo apt-get update
sudo apt-get install maven

//查看是否正确安装
mvn -v
```

执行编译：

```
// 当前目录
ls
config  LICENSE.txt  pom.xml  README.md  src
// 编译
mvn package
```

等待直到出现如下提示时，表示编译成功：

```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 7:43.935s
[INFO] Finished at: Sun Jun 19 18:36:59 CST 2016
[INFO] Final Memory: 15M/56M
[INFO] ------------------------------------------------------------------------

//查看目录
ls
config  LICENSE.txt  pom.xml  README.md  src  target

```

发现会多出了一个 `target` 目录，`copy and unzip` 目录下的文件 `target/releases/elasticsearch-analysis-ik-{version}.zip`  到 上面安装的 `elasticsearch/plugins/ik` 目录中，需要新建 `ik` 目录：

```
// target 目录
$ pwd
elasticsearch-analysis-ik/target/releases
$ ls 
elasticsearch-analysis-ik-1.9.3.zip

```

在 `elasticsearch-2.3.3/plugins/` 下新建目录`ik` :

```
$ ls /elastic/elasticsearch-2.3.3/plugins
$ mkdir /elastic/elasticsearch-2.3.3/plugins/ik
$ ls /elastic/elasticsearch-2.3.3/plugins
ik
```

将上面编译完成的 `elasticsearch-analysis-ik-1.9.3.zip` 拷贝到 `ik` 目录并解压：

```
$ cp elasticsearch-analysis-ik-1.9.3.zip ~/elastic/elasticsearch-2.3.3/plugins/ik
$ ls /elastic/elasticsearch-2.3.3/plugins/ik
elasticsearch-analysis-ik-1.9.3.zip
// 解压
$ unzip elasticsearch-analysis-ik-1.9.3.zip
// 得到如下内容：
$ ls
commons-codec-1.9.jar                elasticsearch-analysis-ik-1.9.3.zip
commons-logging-1.2.jar              httpclient-4.4.1.jar
config                               httpcore-4.4.1.jar
elasticsearch-analysis-ik-1.9.3.jar  plugin-descriptor.properties

```

**注意：上面的操作目录请根据个人的目录和所下载的版本号进行修改**

*** 

如果你觉得上面的编译步骤太繁琐，也可以直接在github页面的`release`中下载已经编译好的zip文件，直接解压到 `ik` 目录下即可：

```
wget -c https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v1.9.3/elasticsearch-analysis-ik-1.9.3.zip

unzip elasticsearch-analysis-ik-1.9.3.zip
```

重启 `elasticsearch` 服务即可。

如果看到类似于下面的信息，说明 IK Analysis 插件已经装好了:

```
plugins [analysis-ik]
```

*** 

#### 如何以`root` 账户运行elasticSearch

默认情况下，ElasticSearch不允许以`root`账户运行，会报 `don't run elasticsearch as root.` 错误。
不过我们也可以强制让其以 `root` 账户来运行：

```
bin/elasticsearch -Des.insecure.allow.root=true
```

*** 

#### 配置同义词  (该部分待测试)

Elasticsearch 自带一个名为 `synonym` 的同义词 `filter`。为了能让 `IK` 和 `synonym` 同时工作，我们需要定义新的 `analyzer`，用 `IK` 做 `tokenizer`，`synonym` 做 `filter`。听上去很复杂，实际上要做的只是加一段配置。

打开 `~/elasticsearch-2.3.3/config/elasticsearch.yml` 文件，加入以下配置：

```
index:
  analysis:
    analyzer:
      ik_syno:
          type: custom
          tokenizer: ik_max_word
          filter: [my_synonym_filter]
      ik_syno_smart:
          type: custom
          tokenizer: ik_smart
          filter: [my_synonym_filter]
    filter:
      my_synonym_filter:
          type: synonym
          synonyms_path: analysis/synonym.txt
```

以上配置定义了 `ik_syno` 和 `ik_syno_smart` 这两个新的 `analyzer`，分别对应 `IK` 的 `ik_max_word` 和 `ik_smart` 两种分词策略。根据 `IK` 的文档，二者区别如下：

* ik_max_word：会将文本做最细粒度的拆分，例如「中华人民共和国国歌」会被拆分为「中华人民共和国、中华人民、中华、华人、人民共和国、人民、人、民、共和国、共和、和、国国、国歌」，会穷尽各种可能的组合；
* ik_smart：会将文本做最粗粒度的拆分，例如「中华人民共和国国歌」会被拆分为「中华人民共和国、国歌」；

ik_syno 和 ik_syno_smart 都会使用 synonym filter 实现同义词转换。为了方便后续测试，建议创建 `~/elasticsearch-2.3.3/config/analysis/synonym.txt` 文件，输入一些同义词并存为 `utf-8` 格式。例如：

```
ua,user-agent,userAgent
js,javascript
internet explore=>ie
```

*** 

#### 用Restful api 测试搜索

(1) 查看集群健康信息：

```
$ curl -X GET http://localhost:9200/_cat/health?v
```

返回结果为：

```
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent 
1466352115 00:01:55  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0% 
```

返回结果的主要字段意义：

* cluster：集群名，是在ES的配置文件中配置的cluster.name的值。
* status：集群状态。集群共有green、yellow或red中的三种状态。green代表一切正常（集群功能齐全），yellow意味着所有的数据都是可用的，但是某些复制没有被分配（集群功能齐全），red则代表因为某些原因，某些数据不可用。如果是red状态，则要引起高度注意，数据很有可能已经丢失。
* node.total：集群中的节点数。
* node.data：集群中的数据节点数。
* shards：集群中总的分片数量。
* pri：主分片数量，英文全称为private。
* relo：复制分片总数。
* unassign：未指定的分片数量，是应有分片数和现有的分片数的差值（包括主分片和复制分片）。

我们也可以在请求中添加help参数来查看每个操作返回结果字段的意义。

```
$ curl -X GET http://localhost:9200/_cat/health?help
```

```
epoch                 | t,time                                   | seconds since 1970-01-01 00:00:00  
timestamp             | ts,hms,hhmmss                            | time in HH:MM:SS                   
cluster               | cl                                       | cluster name                      
status                | st                                       | health status                      
node.total            | nt,nodeTotal                             | total number of nodes              
node.data             | nd,nodeData                              | number of nodes that can store data
shards                | t,sh,shards.total,shardsTotal            | total number of shards             
pri                   | p,shards.primary,shardsPrimary           | number of primary shards           
relo                  | r,shards.relocating,shardsRelocating     | number of relocating nodes         
init                  | i,shards.initializing,shardsInitializing | number of initializing nodes       
unassign              | u,shards.unassigned,shardsUnassigned     | number of unassigned shards        
pending_tasks         | pt,pendingTasks                          | number of pending tasks            
max_task_wait_time    | mtwt,maxTaskWaitTime                     | wait time of longest task pending  
active_shards_percent | asp,activeShardsPercent                  | active number of shards in percent 
```

有了这个东东，就可以减少看文档的时间。ES中许多API都可以添加help参数来显示字段含义，哪些可以这么做呢？每个API都试试就知道了。

当然，如果你觉得返回的东西太多，看着眼烦，我们也可以人为地指定返回的字段。

```
elasticsearch green 0
```

(2) 查看集群中的节点信息

```
tiger@vbox:~$ curl -XGET http://localhost:9200/_cat/nodes?v
```

返回节点的详细信息如下：

```
host      ip        heap.percent ram.percent load node.role master name     
10.0.2.15 10.0.2.15            9          90 0.15 d         *      Bullseye 
tiger@vbox:~$ 
```

(3) 查看集群中的索引信息

```
tiger@vbox:~$ curl -XGET http://localhost:9200/_cat/indices?v
health status index   pri rep docs.count docs.deleted store.size pri.store.size 
yellow open   twitter   5   1          1            0      4.1kb          4.1kb 
```

***

##### 索引（Index）相关API

需要一个索引 `index` 和 `type`

(1) 创建一个新的索引

* 使用自定义id索引文档

使用`PUT`请求创建一个索引为`twitter`类型为`tweet`的文档。其文档编号为`1`，文档内容包含`title`和`content` :

```
tiger@tiger-vbox:~$ curl -XPUT 'http://localhost:9200/twitter/tweet/1?pretty' -d '{
    "title":"三星无线充电技术，值得国产手机去学习吗",
    "content":"三星无线充电器为圆行造型"
}'
```

返回信息

```
{
  "_index" : "twitter",
  "_type" : "tweet",
  "_id" : "1",
  "_version" : 1,
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "created" : true
}
```

再添加一篇：

```
$ curl -XPUT 'http://localhost:9200/twitter/tweet/2?pretty' -d '{
    "title":"范冰冰着花裙亮相青岛，侧颜精致",
    "content":"6月19日，范冰冰现身青岛某活动，一身花裙甜美亮相，精致侧颜秒杀菲林。"
}'
```

返回结果：

```
tiger@vbox:~$ curl -XPUT 'http://localhost:9200/twitter/tweet/2?pretty' -d '{"title":"范冰冰着花裙亮相青岛，侧颜精致","content":"6月19日，范冰 冰现身青岛某活动，一身花裙甜美亮相，精致侧颜秒杀菲林。"}'
{
  "_index" : "twitter",
  "_type" : "tweet",
  "_id" : "2",
  "_version" : 2,
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "created" : false
}
```

查看所有文章：

```
$ curl -XGET 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '{"query":{"match_all":{}}}'
```

返回结果：

```
{
  "took" : 147,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "twitter",
      "_type" : "tweet",
      "_id" : "2",
      "_score" : 1.0,
      "_source" : {
        "title" : "范冰冰着花裙亮相青岛，侧颜精致",
        "content" : "6月19日，范冰冰现身青岛某活动，一身花裙甜美亮相，精致侧颜秒杀菲林。"
      }
    }, {
      "_index" : "twitter",
      "_type" : "tweet",
      "_id" : "1",
      "_score" : 1.0,
      "_source" : {
        "title" : "三星无线充电技术，值得国产手机去学习吗",
        "content" : "三星无线充电器为圆行造型"
      }
    } ]
  }
}
```

这样会把刚才添加的文章都列出来。

搜索关键词“无线”：

```
tiger@vbox:~$ curl -XGET "http://localhost:9200/twitter/tweet/_search?pretty=true" -d '{"query":{"query_string":{"query":"无线"}}}'
```

```
{
  "took" : 49,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.0958915,
    "hits" : [ {
      "_index" : "twitter",
      "_type" : "tweet",
      "_id" : "1",
      "_score" : 0.0958915,
      "_source" : {
        "title" : "三星无线充电技术，值得国产手机去学习吗",
        "content" : "三星无线充电器为圆行造型"
      }
    } ]
  }
}
```

检查ik的切词效果，可以执行：

```
tiger@tiger-vbox:~$ curl 'http://localhost:9200/twitter/_analyze?analyzer=ik_max_word&pretty=true' -d '{"text":"中华人民共和国国歌"}'
```

返回结果：

```
{
  "tokens" : [ {
    "token" : "中华人民共和国",
    "start_offset" : 0,
    "end_offset" : 7,
    "type" : "CN_WORD",
    "position" : 0
  }, {
    "token" : "中华人民",
    "start_offset" : 0,
    "end_offset" : 4,
    "type" : "CN_WORD",
    "position" : 1
  }, {
    "token" : "中华",
    "start_offset" : 0,
    "end_offset" : 2,
    "type" : "CN_WORD",
    "position" : 2
  }, {
    "token" : "华人",
    "start_offset" : 1,
    "end_offset" : 3,
    "type" : "CN_WORD",
    "position" : 3
  }, {
    "token" : "人民共和国",
    "start_offset" : 2,
    "end_offset" : 7,
    "type" : "CN_WORD",
    "position" : 4
  }, {
    "token" : "人民",
    "start_offset" : 2,
    "end_offset" : 4,
    "type" : "CN_WORD",
    "position" : 5
  }, {
    "token" : "共和国",
    "start_offset" : 4,
    "end_offset" : 7,
    "type" : "CN_WORD",
    "position" : 6
  }, {
    "token" : "共和",
    "start_offset" : 4,
    "end_offset" : 6,
    "type" : "CN_WORD",
    "position" : 7
  }, {
    "token" : "国",
    "start_offset" : 6,
    "end_offset" : 7,
    "type" : "CN_CHAR",
    "position" : 8
  }, {
    "token" : "国歌",
    "start_offset" : 7,
    "end_offset" : 9,
    "type" : "CN_WORD",
    "position" : 9
  } ]
}
```

##### 说明一下

`pretty` 参数就是让返回的json有换行和缩进，容易阅读，调试时可以加上，开发到程序里就可以去掉了。

***

#### 相关链接

##### 依赖包

* [GitHub - medcl/elasticsearch-analysis-ik: The IK Analysis plugin integrates Lucene IK analyzer into elasticsearch, support customized dictionary.](https://github.com/medcl/elasticsearch-analysis-ik/)

##### Linux下安装

* [使用 Elasticsearch 实现博客站内搜索 | JerryQu 的小站](https://imququ.com/post/elasticsearch.html) **☆**
* [教你成为全栈工程师(Full Stack Developer) 二十四-ES(elasticsearch)搜索引擎安装和使用 - SharEDITor - 关注大数据技术](http://www.shareditor.com/blogshow/?blogId=36)
* [Running as a Service on Linux        | Elasticsearch Reference [2.3]      | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-service.html)
* [The Elastic Stack Download · Get Started in Minutes  | Elastic](https://www.elastic.co/downloads)


##### Windows下安装

* [windows下安装elasticsearch-1.7.1 | 教程网](http://www.fromwww.net/36206.html)
* [Running as a Service on Windows        | Elasticsearch Reference [2.3]      | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-service-win.html)
* [Windows下安装Maven参考](http://jingyan.baidu.com/article/1709ad808ad49f4634c4f00d.html)


##### Restful Api

* [Python Elasticsearch api - letong - 博客园](http://www.cnblogs.com/letong/p/4749234.html)
* [ElasticSearch教程（4）——ElasticSearch基于REST的CRUD API - 为程序员服务](http://ju.outofmemory.cn/entry/50617)
* [实时搜索引擎Elasticsearch（2）——Rest API的使用 - HinyLover的专栏        - 博客频道 - CSDN.NET](http://blog.csdn.net/xialei199023/article/details/48085125)
* [Document APIs        | Elasticsearch Reference [2.3]      | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html)
* [elasticsearch rest api 快速上手 · Issue #5 · sxyx2008/elasticsearch · GitHub](https://github.com/sxyx2008/elasticsearch/issues/5)


##### 文档

* [Elasticsearch - 随笔分类 - xingoo - 博客园](http://www.cnblogs.com/xing901022/category/642865.html)
* [使用Python进行Elasticsearch数据索引 | Silent River](http://www.justinablog.com/archives/967)
* [安装和使用 Elasticsearch | vpsee.com](http://www.vpsee.com/2014/05/install-and-play-with-elasticsearch/)
* [实时搜索引擎Elasticsearch（1）——基础概念、安装和运行 - HinyLover的专栏        - 博客频道 - CSDN.NET](http://blog.csdn.net/xialei199023/article/details/47680401) **☆**
* [Introduction | Elasticsearch权威指南（中文版）](http://es.xiaoleilu.com/) **☆**
* [AAAAAAA](http://www.ttlsa.com/bigdata/elasticsearch-analysis-ik-chinese/)
* https://github.com/medcl/elasticsearch-analysis-ik
* http://blog.csdn.net/xialei199023/article/details/48085125
* http://blog.csdn.net/xialei199023/article/details/48227247
* https://github.com/elastic/elasticsearch
* https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html
* https://github.com/sxyx2008/elasticsearch/issues/5
* http://www.shareditor.com/blogshow/?blogId=36

