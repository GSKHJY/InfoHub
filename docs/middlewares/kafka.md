# Kafka

以下介绍基于 Kafka 2.x 版本

- 启动命令：bin/kafka-server-start.sh config/server.properties 
- 停止命令：bin/kafka-server-stop.sh config/server.properties

## Kafka 简介

Kafka 是一个分布式流式平台，它有三个关键能力：
1. 订阅发布记录流，它类似于企业中的消息队列 或 企业消息传递系统
2. 以容错的方式存储记录流
3. 实时记录流

Kafka 使用场景：
1. 用户追踪，记录用户操作，用于离线分析或数据挖掘；
2. 日志整合、收集；
3. 消息队列；
4. 记录运营指标

## Zookeeper

Kafka 底层使用 Zookeeper 进行管理。Zookeeper 负责管理 Kafka 集群元数据，协调系统工作。

- 启动命令：./zkServer.sh start
- 关闭命令：./zkServer.sh stop
- 查看状态：./zkServer.sh status

## 基本组件

- Producer : 发布消息的客户端，生产者在默认情况下把消息均衡地分布到主题的所有分区上，而并不关心特定消息会被写到哪个分区。
- Broker：一个从生产者接受并存储消息的客户端，broker 接收来自生产者的消息，为消息设置偏移量，并提交消息到磁盘保存。broker 为消费者提供服务，对读取分区的请求作出响应，返回已经提交到磁盘上的消息。
- Consumer : 消费者从 Broker 中读取消息，一个消费者可以消费多个 topic 的消息，对于某一个 topic 的消息，其只会消费同一个 partition 中的消息

## 基本概念

### topic

Topic 被称为主题，在 kafka 中，使用一个类别属性来划分消息的所属类，划分消息的这个类称为 topic。topic 相当于消息的分配标签，是一个逻辑概念。主题好比是数据库的表，或者文件系统中的文件夹。

### partition

partition 译为分区，topic 中的消息被分割为一个或多个的 partition，它是一个物理概念，对应到系统上的就是一个或若干个目录，一个分区就是一个 提交日志。消息以追加的形式写入分区，先后以顺序的方式读取。

分区可以分布在不同的服务器上，也就是说，一个主题可以跨越多个服务器，以此来提供比单个服务器更强大的性能。

### segment

Segment 被译为段，将 Partition 进一步细分为若干个 segment，每个 segment 文件的大小相等。

## 集群角色

- Leader 负责给定分区的所有读取和写入的节点，每个节点都会通过随机选择成为 leader。
- Replicas 是为该分区复制日志的节点列表，无论它们是 Leader 还是当前处于活动状态。
- Isr 是同步副本的集合。它是副本列表的子集，当前仍处于活动状态并追随Leader。


每个集群中都会有一个 broker 同时充当了 集群控制器(Leader)的角色，它是由集群中的活跃成员选举出来的。每个集群中的成员都有可能充当 Leader，Leader 负责管理工作，包括将分区分配给 broker 和监控 broker。集群中，一个分区从属于一个 Leader，但是一个分区可以分配给多个 broker（非Leader），这时候会发生分区复制。这种复制的机制为分区提供了消息冗余，如果一个 broker 失效，那么其他活跃用户会重新选举一个 Leader 接管。

## Kafka 重启顺序

先关闭 Kafka，后关闭 Zookeeper；先启动 Zookeeper，后启动 Kafka。

如果在关闭 Kafka时，先关闭了 zookeeper。在 Kafka 下一次启动时，会报节点已存在的错误。此时需要把 Zookeeper 中的 zkdata/version-2 文件夹删除。

## 参考资料

