---
title: "Kafka参数"
date: 2021-07-13T09:12:20+08:00
draft: true
tags: ["kafka"]
categories: ["kafka"]
---



## Broker 端参数

### log.dirs

指定了 Broker 需要使用的若干个文件目录路径。

```
/home/kafka1,/home/kafka2,/home/kafka3
```



### zookeeper.connect

kafka链接zk的参数

```bash
# 单kafka使用zk
zk1:2181,zk2:2181,zk3:2181

# 多kafka使用zk
zk1:2181,zk2:2181,zk3:2181/kafka1
zk1:2181,zk2:2181,zk3:2181/kafka2
```



### listeners

告诉外部连接者要通过什么协议访问指定主机名和端口开放的 Kafka 服务



### advertised.listeners

Advertised 的含义表示宣称的、公布的，就是说这组监听器是 Broker 用于对外发布的



### listener.security.protocol.map

```bash
# 表示CONTROLLER这个自定义协议底层使用明文不加密传输数据。
listener.security.protocol.map=CONTROLLER:PLAINTEXT
```



### auto.create.topics.enable

是否允许自动创建 Topic。建议false

### unclean.leader.election.enable

是否允许 Unclean Leader 选举。建议false

### auto.leader.rebalance.enable

是否允许定期进行 Leader 选举。建议false

### log.retention.{hour|minutes|ms}

控制一条消息数据被保存多长时间

### log.retention.bytes

指定 Broker 为消息保存的总磁盘容量大小

### message.max.bytes

控制 Broker 能够接收的最大消息大小



## Topic级别参数

### retention.ms

该 Topic 消息被保存的时长，默认7天

### retention.bytes

Topic 预留多大的磁盘空间，-1为无限

###  topic参数设置

```bash
# 创建 Topic 时进行设置
bin/kafka-topics.sh --bootstrap-serverlocalhost:9092 --create--topictransaction --partitions1 --replication-factor1 --configretention.ms=15552000000 --configmax.message.bytes=5242880

# 修改 Topic 时设置
bin/kafka-configs.sh --zookeeperlocalhost:2181 --entity-typetopics --entity-nametransaction --alter--add-configmax.message.bytes=10485760

```





