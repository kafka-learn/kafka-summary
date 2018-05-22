---
title: "Kafka Topic的JAVA API操作"
layout: page
date: 2018-05-25 00:00
---

## 项目环境

* unix platform

* zookeeper-3.4.12

* kafka_2.11-1.1.0

* JDK8 + idea + gradle

## 介绍

### zookeeper 启动（单节点）

* zoo.cfg

修改下zoo.cfg, 没有则新建，主要是如下的三个设置

```
dataDir=/Users/mubi/soft/zookeeper-3.4.12/data
dataLogDir=/Users/mubi/soft/zookeeper-3.4.12/logs
clientPort=2181
```

* 启动zookeeper

```
$ bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /Users/mubi/soft/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

* 关闭 zookeeper

```bash
$ bin/zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /Users/mubi/soft/zookeeper-3.4.12/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
```

### Kafak 启动 与 topic操作相关命令

* conf/server.properties

主要是 ```zookeeper.connect```, ```log.dirs```, ```broker.id```等配置

* 开启 kafka server

```bash
$ bin/kafka-server-start.sh config/server.properties
```

* 查看所有topic

```bash
$ bin/kafka-topics.sh --list --zookeeper localhost:2181
__consumer_offsets
connect-test
my-replicated-topic
test
```

* 查看具体的topic

```bash
$ bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test	PartitionCount:1	ReplicationFactor:1	Configs:
	Topic: test	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
```

* 修改topic 分区(partion)数量

```bash
$ bin/kafka-topics.sh --zookeeper localhost:2181  --alter --topic test --partitions 2
```

![](https://raw.githubusercontent.com/kafka-learn/imgs/master/src/alter_partion.png)

* 关闭 kafka 服务

```bash
$ bin/kafka-server-stop.sh
```

### Java API 操作 topic

代码见 code/demo-topic 项目

* 查看所有的topic

```
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
//        props.put("group.id","test");
//        props.put("auto.offset.reset","earliest");
//        props.put("acks", "all");
//        props.put("retries", 0);
//        props.put("batch.size", 16384);
//        props.put("linger.ms", 1);
//        props.put("buffer.memory", 33554432);
//        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
//        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");


AdminClient adminClient = AdminClient.create(props);

ListTopicsResult listTopicsResult = adminClient.listTopics();
KafkaFuture<java.util.Map<java.lang.String,TopicListing>> kafkaFuture
		= listTopicsResult.namesToListings();
try {
	Map<String,TopicListing> mp = kafkaFuture.get();

	mp.forEach((String topic_name, TopicListing topicListing) -> {
		System.out.println(topicListing.toString());

	});
} catch (InterruptedException e) {
	e.printStackTrace();
} catch (ExecutionException e) {
	e.printStackTrace();
}finally {
	adminClient.close();
}
```

* topic 创建

![](https://raw.githubusercontent.com/kafka-learn/imgs/master/src/create_topic.png)

* describe topic

![](https://raw.githubusercontent.com/kafka-learn/imgs/master/src/describe_topic.png)

## 参考

http://kafka.apache.org/11/javadoc/overview-summary.html

注：具体的类说明，需要看API中的介绍