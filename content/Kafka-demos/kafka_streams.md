---
title: "Kafka Streams的简单JAVA API操作"
layout: page
date: 2018-06-09 16:05
---

[TOC]

## 项目环境

* Win10 platform

* Kafka自带的Zookeeper

* kafka_2.12-1.1.0

* JDK8 + IDEA + Maven

## 项目搭建

使用IDEA建立一个Maven项目，并添加依赖：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>1.1.0</version>
</dependency>
```

## 第一个Streams应用程序：pipe

* 通过对象来添加配置项

```java
Properties props = new Properties();
props.put(StreamsConfig.APPLICATION_ID_CONFIG, "streams-pipe"); //该Streams应用程序的唯一ID
props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092"); 
// 假设正在与该应用程序对话的Kafka代理在端口为9092的本地机器上运行
props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());
```

* 定义拓扑结构

```java
final StreamsBuilder builder = new StreamsBuilder();
```

* 指定源主题和目的主题

```java
builder.stream("streams-plaintext-input").to("streams-pipe-output");
```

* 构建拓扑结构

```java
final Topology topology = builder.build();
```

* 检查拓扑结构的类型

```java
System.out.println(topology.describe());
```

此时运行的话，可以看到：

```bash
Sub-topologies:
  Sub-topology: 0
    Source: KSTREAM-SOURCE-0000000000(topics: streams-plaintext-input) --> KSTREAM-SINK-0000000001
    Sink: KSTREAM-SINK-0000000001(topic: streams-pipe-output) <-- KSTREAM-SOURCE-0000000000
Global Stores:
  none
```

上述输出结果指明了Kafka内部节点的数据传输情况。

* 通过配置文件和拓扑结构来构建KafkaStreams

```java
final KafkaStreams streams = new KafkaStreams(topology, props);
```

通过调用它的`start()`函数，我们可以触发这个客户端的执行。在此客户端被调用`close()`之前，执行不会停止。例如，我们可以添加一个带有`CountDownLatch`的关闭钩子来捕获用户中断，并在终止该程序时关闭客户端：

```java
final CountDownLatch latch = new CountDownLatch(1);

// attach shutdown handler to catch control-c
// 附加关闭处理程序来捕获control-c
Runtime.getRuntime().addShutdownHook(new Thread("streams-shutdown-hook") {
    @Override
    public void run() {
        streams.close();
        latch.countDown();
    }
});

try {
    streams.start();
    latch.await();
} catch (Throwable e) {
    System.exit(1);
}
System.exit(0);
```

* 接下来便要启动Kafka服务并创建相应的topic

