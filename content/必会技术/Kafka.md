#  Install
## 环境
* mac OS
* Kafka
	* 版本：kafka_2.13-2.8.0.tgz
	* 下载地址：https://kafka.apache.org/downloads
## 使用
* 解压：
```shell
> tar -xzvf kafka_2.13-2.8.0.tgz
```
* 启动zookeeper：
```shell
>cd bin
>./zookeeper-server-start.sh ../config/zookeeper.properties
```
* 启动kafka：
```shell
>./kafka-server-start.sh ../config/server.properties
```
* 创建主题：
```shell
>./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic Hello-Kafka
```
* 查看主题：
```shell
>./kafka-topics.sh --list --zookeeper localhost:2181
```
* 发送消息：
```shell
>./kafka-console-producer.sh --broker-list localhost:9092 --topic Hello-Kafka
```
* 监听（消费）消息：
```shell
>./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic Hello-Kafka --from-beginning
```


# 模型
* 队列模型：是早期的消息模型
	* 使用队列作为消息通信载体，满足生产者-消费者模式，一条消息只能被一个消费者使用，未被消费的消息在队列中保留直到被消费或超时。
	* 存在的问题：当生产者产生的消息需要分发给多个消费者，并且每个消费者都能接受到完整的消息内容时，这种队列模型就不适合了。
* 发布-订阅模型（Pub-Sub）：Kafka消息模型
	* 发布订阅模型（Pub-Sub） 使用**主题（Topic）** 作为消息通信载体，类似于**广播模式**；发布者发布一条消息，该消息通过主题传递给所有的订阅者，**在一条消息广播之后才订阅的用户则是收不到该条消息的**。
	* 在发布 - 订阅模型中，如果只有一个订阅者，那它和队列模型就基本是一样的了。所以说，发布 - 订阅模型在功能层面上是可以兼容队列模型的。
# 核心概念
![](Kafka模型.png)

## Producer
> 生产消息的一方

## Consumer
> 消费消息的一方
## Broker
> 代理，可以看做是一个独立的Kafka实例，多个Kafka Broker组成一个Kafka Cluster。
## Topic
> 主题，Producer将消息发送到特点的主题，Consumer通过订阅特定的Topic（主题）来消费消息。
## Partition
> 分区，Partition属于Topic的一部分，一个Topic可以有多个Partition，并且同一个Topic下的Partition可以分布在不同的Broker上（一个Topic可以横跨多个Broker）。

## 多副本机制

## 多分区机制

## 消费组

## 重平衡机制

# Zookeeper 和 Kafka


# 消费顺序、消息丢失、重复消费问题
