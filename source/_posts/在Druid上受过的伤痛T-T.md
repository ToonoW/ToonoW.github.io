---
title: 在Druid上受过的伤痛T-T
date: 2017-09-11 17:31:12
tags:
- Druid
---

## 利用Kafka给Druid导入数据报错`Provider io.druid.storage.hdfs.HdfsStorageDruidModule could not be instantiated`

外行人遇到下面一大段错误让人摸不着头脑，意思好像是说缺少了一些必要的类组件，但是Druid的开发者应该不会在稳定版忘了打包一些部件吧。硬是搜索了大半天的答案，实际上也没什么资料可以搜索到，所以同样的几条翻来覆去地看，最后把每条觉得不太可能成功的答案都尝试了，终于有一个奏效了。

错误信息如下。

```shell
...

2017-09-11 18:33:59,833 [KafkaConsumer-1] ERROR c.m.tranquility.kafka.KafkaConsumer - Exception:
java.util.ServiceConfigurationError: io.druid.initialization.DruidModule: Provider io.druid.storage.hdfs.HdfsStorageDruidModule could not be instantiated
	at java.util.ServiceLoader.fail(ServiceLoader.java:232) ~[na:1.8.0_111]
	at java.util.ServiceLoader.access$100(ServiceLoader.java:185) ~[na:1.8.0_111]
	at java.util.ServiceLoader$LazyIterator.nextService(ServiceLoader.java:384) ~[na:1.8.0_111]
	at java.util.ServiceLoader$LazyIterator.next(ServiceLoader.java:404) ~[na:1.8.0_111]
	at java.util.ServiceLoader$1.next(ServiceLoader.java:480) ~[na:1.8.0_111]
	at io.druid.initialization.Initialization.getFromExtensions(Initialization.java:157) ~[io.druid.druid-server-0.9.1.jar:0.9.1]
	at com.metamx.tranquility.druid.DruidGuicer.<init>(DruidGuicer.scala:124) ~[io.druid.tranquility-core-0.8.2.jar:0.8.2]
	at com.metamx.tranquility.druid.DruidGuicer$.<init>(DruidGuicer.scala:138) ~[io.druid.tranquility-core-0.8.2.jar:0.8.2]
	at com.metamx.tranquility.druid.DruidGuicer$.<clinit>(DruidGuicer.scala) ~[io.druid.tranquility-core-0.8.2.jar:0.8.2]
	at com.metamx.tranquility.druid.DruidBeams$.makeFireDepartment(DruidBeams.scala:433) ~[io.druid.tranquility-core-0.8.2.jar:0.8.2]
	at com.metamx.tranquility.druid.DruidBeams$.fromConfigInternal(DruidBeams.scala:299) ~[io.druid.tranquility-core-0.8.2.jar:0.8.2]
	at com.metamx.tranquility.druid.DruidBeams$.fromConfig(DruidBeams.scala:204) ~[io.druid.tranquility-core-0.8.2.jar:0.8.2]
	at com.metamx.tranquility.kafka.KafkaBeamUtils$.createTranquilizer(KafkaBeamUtils.scala:40) ~[io.druid.tranquility-kafka-0.8.2.jar:0.8.2]
	at com.metamx.tranquility.kafka.KafkaBeamUtils.createTranquilizer(KafkaBeamUtils.scala) ~[io.druid.tranquility-kafka-0.8.2.jar:0.8.2]
	at com.metamx.tranquility.kafka.writer.TranquilityEventWriter.<init>(TranquilityEventWriter.java:64) ~[io.druid.tranquility-kafka-0.8.2.jar:0.8.2]
	at com.metamx.tranquility.kafka.writer.WriterController.createWriter(WriterController.java:171) ~[io.druid.tranquility-kafka-0.8.2.jar:0.8.2]
	at com.metamx.tranquility.kafka.writer.WriterController.getWriter(WriterController.java:98) ~[io.druid.tranquility-kafka-0.8.2.jar:0.8.2]
	at com.metamx.tranquility.kafka.KafkaConsumer$2.run(KafkaConsumer.java:231) ~[io.druid.tranquility-kafka-0.8.2.jar:0.8.2]
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511) [na:1.8.0_111]
	at java.util.concurrent.FutureTask.run(FutureTask.java:266) [na:1.8.0_111]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142) [na:1.8.0_111]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617) [na:1.8.0_111]
	at java.lang.Thread.run(Thread.java:745) [na:1.8.0_111]
Caused by: java.lang.NoClassDefFoundError: io/druid/java/util/common/logger/Logger
	at io.druid.storage.hdfs.HdfsStorageDruidModule.<clinit>(HdfsStorageDruidModule.java:51) ~[na:na]
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method) ~[na:1.8.0_111]
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62) ~[na:1.8.0_111]
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45) ~[na:1.8.0_111]
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423) ~[na:1.8.0_111]
	at java.lang.Class.newInstance(Class.java:442) ~[na:1.8.0_111]
	at java.util.ServiceLoader$LazyIterator.nextService(ServiceLoader.java:380) ~[na:1.8.0_111]
	... 20 common frames omitted
Caused by: java.lang.ClassNotFoundException: io.druid.java.util.common.logger.Logger
	at java.net.URLClassLoader.findClass(URLClassLoader.java:381) ~[na:1.8.0_111]
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424) ~[na:1.8.0_111]
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357) ~[na:1.8.0_111]
	... 27 common frames omitted
2017-09-11 18:33:59,835 [KafkaConsumer-1] INFO  c.m.tranquility.kafka.KafkaConsumer - Shutting down - attempting to flush buffers and commit final offsets
2017-09-11 18:33:59,836 [Curator-Framework-0] INFO  o.a.c.f.imps.CuratorFrameworkImpl - backgroundOperationsLoop exiting
2017-09-11 18:33:59,851 [KafkaConsumer-1] INFO  org.apache.zookeeper.ZooKeeper - Session: 0x15e71a1da94001e closed
2017-09-11 18:33:59,852 [KafkaConsumer-1-EventThread] INFO  org.apache.zookeeper.ClientCnxn - EventThread shut down for session: 0x15e71a1da94001e
2017-09-11 18:33:59,864 [KafkaConsumer-1] INFO  k.c.ZookeeperConsumerConnector - [tranquility-kafka_localhost.localdomain-1505154826384-a1e5698a], ZKConsumerConnector shutting down
2017-09-11 18:33:59,880 [KafkaConsumer-1] INFO  k.c.ZookeeperTopicEventWatcher - Shutting down topic event watcher.
2017-09-11 18:33:59,880 [KafkaConsumer-1] INFO  k.consumer.ConsumerFetcherManager - [ConsumerFetcherManager-1505154826568] Stopping leader finder thread
2017-09-11 18:33:59,881 [KafkaConsumer-1] INFO  k.c.ConsumerFetcherManager$LeaderFinderThread - [tranquility-kafka_localhost.localdomain-1505154826384-a1e5698a-leader-finder-thread], Shutting down
2017-09-11 18:33:59,886 [tranquility-kafka_localhost.localdomain-1505154826384-a1e5698a-leader-finder-thread] INFO  k.c.ConsumerFetcherManager$LeaderFinderThread - [tranquility-kafka_localhost.localdomain-1505154826384-a1e5698a-leader-finder-thread], Stopped

...
```

最终找到的[答案](https://groups.google.com/forum/#!topic/druid-user/y43-b1z3c3E)，里面说原因是加载了会引起错误的拓展。然后我们指定拓展外空，就可以避免错误了。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1fjfrno6ypij31gg0f4ad3.jpg)

所以我们在命令最后加上参数`-Ddruid.extensions.loadList=[] `即可。

完整命令：

`./tranquility-distribution-0.8.2/bin/tranquility kafka -configFile conf-quickstart/tranquility/kafka.json -Ddruid.extensions.loadList=[]`

​        

## 确认Kafka运行正常的情况下Message被Dropped

我是使用tranquility配合kafka进行数据流导入Druid。

```shell
2017-09-12 08:41:49,603 [KafkaConsumer-CommitThread] INFO  c.m.tranquility.kafka.KafkaConsumer - Flushed {crlog={receivedCount=1, sentCount=1, droppedCount=0, unparseableCount=0}} pending messages in 1ms and committed offsets in 0ms.
```

首先以上是一条理想情况的日志信息`receivedCount`等于`sentCount`，也就是消费者tranquility接收到的消息全部成功解析并且成功发送给Druid。

如果出现`unparseableCount`不为0的情况，那么就是你发过来的字符串无法解析为对应的数据格式（我选用的是json）。在正常情况下，我们的内容发送给kafka之前是需要经过编码的（我选用utf-8）。所以确保你发送的是经过编码，而且可以被解析成json的字符串。

如果出现`droppedCount`，那么就是解析出来的内容不合法了，情况很多种。我这里说我遇到的一种，是时间窗问题不在时间窗内的将会被dropped。还有那就是服务器时间问题。请确保服务器和开发机时间一致，并且是ISO8601格式的时间（2017-09-12T08:59:03Z）。这会影响EVENT是否在时间窗内。

下面再贴上我的项目用到的tranquility配置，以供参考。

```json
{
  "dataSources" : {
    "crlog-kafka" : {
      "spec" : {
        "dataSchema" : {
          "dataSource" : "crlog-kafka",
          "parser" : {
            "type" : "string",
            "parseSpec" : {
              "timestampSpec" : {
                "column" : "datetime",
                "format" : "auto"
              },
              "dimensionsSpec" : {
                "dimensions" : ["product_name", "event_name", "event", "username", "dept", "hospital", "job_title"],
                "dimensionExclusions" : []
              },
              "format" : "json"
            }
          },
          "granularitySpec" : {
            "type" : "uniform",
            "segmentGranularity" : "hour",
            "queryGranularity" : "none"
          },
          "metricsSpec" : [
            {
              "fieldName" : "spent_time",
              "name" : "spent_time",
              "type" : "longSum"
            }
          ]
        },
        "ioConfig" : {
          "type" : "realtime"
        },
        "tuningConfig" : {
          "type" : "realtime",
          "maxRowsInMemory" : "100000",
          "intermediatePersistPeriod" : "PT10M",
          "windowPeriod" : "PT10M"
        }
      },
      "properties" : {
        "task.partitions" : "1",
        "task.replicants" : "1",
        "topicPattern" : "crlog"
      }
    }
  },
  "properties" : {
    "zookeeper.connect" : "192.168.1.4",
    "druid.discovery.curator.path" : "/druid/discovery",
    "druid.selectors.indexing.serviceName" : "druid/overlord",
    "commit.periodMillis" : "15000",
    "consumer.numThreads" : "2",
    "kafka.zookeeper.connect" : "192.168.1.4",
    "kafka.group.id" : "tranquility-kafka"
  }
}
```

  

## 修改 Tranquility 配置中的 intermediatePersistPeriod 导致 Indexing Service 中任务过多

在配置文件的 tuningConfig 部分有关于时间窗口和存盘时间间隔的设置，我在试验性使用 Tranquility 进行实时数据的摄入的时候将时间窗口 windowPeriod 设置为 `PT10H` ，使其窗口接受十小时以内的数据。但是我也在不知道 intermediatePersistPeriod 含义的情况下，同样将其改为 `PT10H`，导致了 Indexing Service 中任务过多的情况。如下图：![](https://ws3.sinaimg.cn/large/006tNc79gy1flqzvsn8tkj31kw0wj486.jpg)

所以 intermediatePersistPeriod 属于优化用的配置，刚接触的情况下不建议修改。

### 相关

以上的说明中， `PT10H` 是[ ISO8601 规范](https://en.wikipedia.org/wiki/ISO_8601#Durations)中定义的格式。

## 时区设置

Druid 的时间默认是 UTC 事件，我们需要设置相应的时区才能按照我们当地的时间运行。否则就会出现一些因为需要导入的记录的时间戳不在时间窗口内而被丢弃的问题。所以遇到 droped 的问题，一定要检查时间的因素。

Druid 的配置文件在 `conf` 目录的各组件子目录下，需要修改的是 `jvm.config`文件，`conf` 下的目录结构如下，每个文件都需要改，否则时间不统一。

```shell
druid/
├── broker
│   ├── jvm.config
│   └── runtime.properties
├── _common
│   ├── common.runtime.properties
│   └── log4j2.xml
├── coordinator
│   ├── jvm.config
│   └── runtime.properties
├── historical
│   ├── jvm.config
│   └── runtime.properties
├── middleManager
│   ├── jvm.config
│   └── runtime.properties
└── overlord
    ├── jvm.config
    └── runtime.properties
```

我们需要将 `-Duser.timezone=UTC` 改为 `-Duser.timezone=UTC+0800`  。同时需要修改 `tranquility/conf/application.ini` 文件的 `-J-Duser.timezone=UTC` 为 `-J-Duser.timezone=UTC+0800` 。这样子就时间统一了。

## 各节点的默认端口

* Overlord - 8090
* Historical - 8083
* Coordinator - 8081
* MiddleManager - 8091
* Broker - 8082