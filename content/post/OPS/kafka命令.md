---
title: "Kafka命令"
date: 2021-07-13T09:41:26+08:00
draft: false
tags: ["kafka"]
categories: ["kafka"]
---



## Topic

### 创建 和修改 

```bash
# 创建一个名为 adb-test，分区为20, 分区副本为3 topic
kafka-topics --create --topic adb-test --partitions 20 --replication-factor 3 --bootstrap-server localhost:19091

# 创建topic
kafka-topics --create --topic my_topic_name --partitions 20 --replication-factor 3 --config x=y --bootstrap-server localhost:9092 

# 修改 partitions 为40
kafka-topics --alter --topic my_topic_name --partitions 40 --bootstrap-server localhost:9092

# 添加配置
kafka-configs --alter --entity-type topics --entity-name my_topic_name --add-config x=y  --bootstrap-server localhost:9092

# 移除配置
kafka-configs --alter --entity-type topics --entity-name my_topic_name --deleteConfig x  --bootstrap-server localhost:9092
```



### 查看

```bash
# 查看一个主题的分片和同步情况
kafka-topics --describe --topic adb-test --bootstrap-server localhost:19091

# 获取某个主题在zk上的偏移量
kafka-run-class kafka.tools.GetOffsetShell --broker-list "localhost:19091" --topic adb-test

# 获取上一时刻某个主题在zk上的偏移量
kafka-run-class kafka.tools.GetOffsetShell --broker-list "localhost:19091" --topic adb-test --time -1
```



### 删除 

```bash
kafka-topics --delete --topic adb-test --bootstrap-server localhost:19091
```



### 生产数据

```bash
kafka-console-producer --topic adb-test --broker-list localhost:19091
```



### 消费数据

```bash
kafka-console-consumer --topic adb-test --from-beginning --bootstrap-server localhost:19091
```



## 配置

### 修改 topic 配置

```bash
# 修改kafka分区的存储量为4GB,且保存时间为120小时
kafka-configs --zookeeper "zookeeper:2181" --entity-type topics --entity-name adb-test --alter --add-config retention.bytes="4294967296" log.retention.hours=120

# 修改完存储量后的查看
kafka-configs --zookeeper "zookeeper:2181" --entity-type topics --entity-name adb-test --describe
```



### 修改 brokers 配置

```bash
# 修改 Broker ID 0 的配置（例如，日志清理线程的数量）：
kafka-configs --bootstrap-server "localhost:9092" --entity-type brokers --entity-name 0 --alter --add-config log.cleaner.threads=2

# 查看brokers详情
kafka-configs --bootstrap-server localhost:9092 --entity-type brokers --entity-name 0 --describe

# 恢复到默认配置
kafka-configs --bootstrap-server localhost:9092 --entity-type brokers --entity-name 0 --alter --delete-config log.cleaner.threads

# 为集群中的所有brokers配置参数
kafka-configs --bootstrap-server localhost:9092 --entity-type brokers --entity-default --alter --add-config log.cleaner.threads=2

```



### 配置优先级

1. 存储在 zookeeper 中的每个Broken 配置
2. 存储在 zookeeper 中的默认配置
3. 本地配置文件 server.properties



## 性能测试

```bash
# 向kafka写入 2000000记录,每秒发送100000, 每个消息1000bytes
kafka-producer-perf-test --topic adb-test --num-records 2000000 --record-size 1000 --throughput 100000 --producer-props bootstrap.servers=localhost:19091
```

