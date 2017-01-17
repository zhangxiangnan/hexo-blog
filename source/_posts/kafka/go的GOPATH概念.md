layout: title
title: guava的不可变集合
date: 2017-01-09 17:17:02
tags:
- kafka
categories:
- 译
- kafka
---

kafka的入门知识
<!-- more -->

### kafka是一个分布式流平台
分布式流平台应该具有的三个特征：
  - 可以发布和订阅流记录，这点类似于消息队列或者企业级消息系统
  - 可以存储流记录，且具有容错性
  - 可以当流记录发生时进行处理.

kafka的优势
kafka用于2大类应用程序：
  构建在系统或应用程序间可以可靠获取数据的流式数据管道，即流式数据在不同系统或程序间传输
  构建针对流式数据做出响应或者处理的实时流式应用程序，如监控日志进行报警或封禁等

几个概念：
Kafka部署在一台或多台机器上，作为一个集群运行。
Kafka集群按照分类存储流记录，这个分类称作主题（Topic）
每一个记录包含key、value、和一个时间戳timestamp

Kafka的四个核心API：
  - Producer API用来发布流记录到一个或多个Kafka topics
  - Consumer API用来消费一个或多个主题Topics，并处理接受到的流记录.
  - Steams API就好似一个流处理器，从一个或多个主题消费输入流，然后产生输出流到一个或多个输出主题topics，高效地转换输入流到输出流。
  - Connector API用来连接Kafka主题到已有的程序或者数据系统，来构建和运行可重复使用的生产者和消费者，如关联到一个关系数据库的连接器connector可以获取表的所有变更

Kafka中，客户端和服务端的通讯使用一个简单的，高性能的，语言无关的TCP协议。该协议有版本控制，并保持与旧版本的向后兼容性。同时，提供了很多种语言的客户端。

### 主题Topics和日志Logs
Kafka为流记录提供的核心抽象便是主题（topic）.
一个主题是一个类别或者feed名称，记录便发布在他们上面。Kafka的主题总是支持多个消费者的，即一个主题可以拥有0个、1个或者更多消费者来消费发送到该主题的数据。
对于每个主题，Kafka集群保持一个分区日志：

Each partition is an ordered, immutable sequence of records that is continually appended to—a structured commit log. The records in the partitions are each assigned a sequential id number called the offset that uniquely identifies each record within the partition.

The Kafka cluster retains all published records—whether or not they have been consumed—using a configurable retention period. For example, if the retention policy is set to two days, then for the two days after a record is published, it is available for consumption, after which it will be discarded to free up space. Kafka's performance is effectively constant with respect to data size so storing data for a long time is not a problem.


In fact, the only metadata retained on a per-consumer basis is the offset or position of that consumer in the log. This offset is controlled by the consumer: normally a consumer will advance its offset linearly as it reads records, but, in fact, since the position is controlled by the consumer it can consume records in any order it likes. For example a consumer can reset to an older offset to reprocess data from the past or skip ahead to the most recent record and start consuming from "now".

This combination of features means that Kafka consumers are very cheap—they can come and go without much impact on the cluster or on other consumers. For example, you can use our command line tools to "tail" the contents of any topic without changing what is consumed by any existing consumers.

The partitions in the log serve several purposes. First, they allow the log to scale beyond a size that will fit on a single server. Each individual partition must fit on the servers that host it, but a topic may have many partitions so it can handle an arbitrary amount of data. Second they act as the unit of parallelism—more on that in a bit.
