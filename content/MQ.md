# Kafka
##  Install
### 环境
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
* 监听消息：
```shell
>./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic Hello-Kafka --from-beginning
```
