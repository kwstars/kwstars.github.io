---
title: "使用Wireshark抓取Redis协议包"
date: 2022-01-11T16:12:04+08:00
draft: false
tags: ["Wireshark", "Redis"]
categories: ["OPS"]
---



## 建立连接

```go
func main() {
	rdb := redis.NewClient(&redis.Options{
		Network:  "",
		Addr:     "192.168.0.100:6379",
		DB:       8,
		Password: "b62uHm3eFEUZkF4z",
	})
	defer rdb.Close()

	rdb.Ping(ctx)
}
```

![image-20220111161332727](https://i.imgur.com/95xzIo1.png)



## Pipeline

```go
func main() {
	rdb := redis.NewClient(&redis.Options{
		Network:  "",
		Addr:     "192.168.0.100:6379",
		DB:       8,
		Password: "b62uHm3eFEUZkF4z",
	})
	defer rdb.Close()

	rdb.HSet(ctx, "hash:kira", "a", 1)
	rdb.HSet(ctx, "hash:kira", "b", 1)
	rdb.HSet(ctx, "hash:kira", "c", 1)

	pipeline := rdb.Pipeline()
	defer pipeline.Close()
	pipeline.HIncrBy(ctx, "hash:kira", "a", 1)
	pipeline.HIncrBy(ctx, "hash:kira", "b", 1)
	pipeline.HIncrBy(ctx, "hash:kira", "c", 1)
	cmd, err := pipeline.Exec(ctx)
	if err != nil {
		log.Fatalln(err)
	}
	for _, v := range cmd {
		log.Println(v.String())
	}
}
```



TCP Stream

```bash
*2
$4
auth
$16
b62uHm3eFEUZkF4z
*2
$6
select
$1
8
+OK
+OK
*4
$4
hset
$9
hash:kira
$1
a
$1
1
:0
*4
$4
hset
$9
hash:kira
$1
b
$1
1
:0
*4
$4
hset
$9
hash:kira
$1
c
$1
1
:0
*4
$7
hincrby
$9
hash:kira
$1
a
$1
1
*4
$7
hincrby
$9
hash:kira
$1
b
$1
1
*4
$7
hincrby
$9
hash:kira
$1
c
$1
1
:2
:2
:2
```



## TxPipeline

```go
func main() {
	rdb := redis.NewClient(&redis.Options{
		Network:  "",
		Addr:     "192.168.0.100:6379",
		DB:       8,
		Password: "b62uHm3eFEUZkF4z",
	})
	defer rdb.Close()

	rdb.HSet(ctx, "hash:kira", "a", 1)
	rdb.HSet(ctx, "hash:kira", "b", 1)
	rdb.HSet(ctx, "hash:kira", "c", 1)

	pipeline := rdb.TxPipeline()
	defer pipeline.Close()
	pipeline.HIncrBy(ctx, "hash:kira", "a", 1)
	pipeline.HIncrBy(ctx, "hash:kira", "b", 1)
	pipeline.HIncrBy(ctx, "hash:kira", "c", 1)
	cmd, err := pipeline.Exec(ctx)
	if err != nil {
		log.Fatalln(err)
	}
	for _, v := range cmd {
		log.Println(v.String())
	}
}
```

 ![image-20220111171419208](https://i.imgur.com/ceeWdWa.png)

与pipeline的tcpstream对比，多了以上内容。pipelne在客户端缓存多个命令，再通过multi发送到服务端。



---

[Redis使用手册](http://redisguide.com/)
