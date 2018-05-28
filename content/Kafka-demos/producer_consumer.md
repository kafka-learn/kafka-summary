---
title: "Kafka Producer & Consumer JAVA API简单的操作"
layout: page
date: 2018-05-28 08:00
---

[TOC]

## 项目环境

* unix platform

* zookeeper-3.4.12

* kafka_2.11-1.1.0

* JDK8 + idea + gradle

## Zookeeper & Kafka 操作

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
topic-test1
```

* 命令行 Produer & Consumer

```bash
$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic topic-test1
```

![](https://raw.githubusercontent.com/kafka-learn/imgs/master/src/console_producer.png)

```bash
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topic-test1 --from-beginning
```

![](https://raw.githubusercontent.com/kafka-learn/imgs/master/src/console_consumer.png)

* 关闭 kafka 服务

```bash
$ bin/kafka-server-stop.sh
```

### Java API Producer & Consumer的简单操作

参考类： <a href="http://kafka.apache.org/11/javadoc/org/apache/kafka/clients/producer/ProducerRecord.html" target="_blank">ProducerRecord类</a>

* producer 发送record

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
//        props.put("acks", "all");
//        props.put("retries", 0);
//        props.put("batch.size", 16384);
//        props.put("linger.ms", 1);
//        props.put("buffer.memory", 33554432);
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
//        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
//        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

//生产者发送消息
String topic = "topic-test1";
Producer<String, String> procuder = new KafkaProducer<String, String>(props);
for (int i = 1; i <= 10; i++) {
	//String value = "value_" + i;
	Scanner scanner = new Scanner(System.in);
	String value = scanner.next();
	//System.out.println(value);

	ProducerRecord<String, String> msg = new ProducerRecord<String, String>(topic, value);
	procuder.send(msg);
}
procuder.flush();
```

* cosumer 消费record

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "test_g1");
props.put("auto.offset.reset", "earliest");
// 是否自动确认offset
props.put("enable.auto.commit", "true");
//        props.put("acks", "all");
//        props.put("retries", 0);
//        props.put("batch.size", 16384);
//        props.put("linger.ms", 1);
//        props.put("buffer.memory", 33554432);
//        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
//        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

KafkaConsumer<String, String> consumer = new KafkaConsumer(props);
consumer.subscribe(Arrays.asList("topic-test1"));
while (true) {
	ConsumerRecords<String, String> records = consumer.poll(100);
	for (ConsumerRecord<String, String> record : records)
		System.out.printf("offset = %d, key = %s, value = %s%n",
				record.offset(), record.key(), record.value());
}
```

* 结果图

![](https://raw.githubusercontent.com/kafka-learn/imgs/master/src/api_producer_consumer.png)

## 源代码地址

<a href="https://github.com/kafka-learn/demos/tree/master/kafka-producer-consumer" target="_blank">点击查看</a>