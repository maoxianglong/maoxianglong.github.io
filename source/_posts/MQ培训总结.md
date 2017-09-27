---
title: MQ培训总结
date: 2017-09-06
tags: [MQ,消息中间件]
categories: [java]
---
## 一、导论
公司最近进行了MQ的培训，总结一下培训的内容。
## 二、MQ简介
### 2.1、什么是消息队列MQ
消息队列（Message queue）是在消息的传输过程中保存消息的容器。消息队列管理器在将消息从它的源中继到它的目标时充当中间人。队列的主要目的是提供路由并保证消息的传递；如果发送消息时接收者不可用，消息队列会保留消息，直到可以成功地传递它。
### 2.2、MQ通讯模式
- 点对点通讯 (p2p)
- 发布/订阅 (pub/sub)


### 2.3、常见的MQ队列
- ActiveMQ
- RabbitMQ
- ZeroMQ
- Kafka
- RocketMQ
- NSQ


## 三、MQ 队列使用场景
### 3.1、应用解耦
场景说明：用户下单后，订单系统需要通知库存系统。传统的做法是，订单系统调用库存系统的接口。如下图：

![image](http://otqvaruzt.bkt.clouddn.com/_01.png)

传统模式的缺点：假如库存系统无法访问，则订单减库存将失败，从而导致订单失败，订单系统与库存系统耦合
如何解决以上问题呢？引入应用消息队列后的方案，如下图：

![image](http://otqvaruzt.bkt.clouddn.com/_02.png)

订单系统：用户下单后，订单系统完成持久化处理，将消息写入消息队列，返回用户订单下单成功
库存系统：订阅下单的消息，采用拉/推的方式，获取下单信息，库存系统根据下单信息，进行库存操作
假如：在下单时库存系统不能正常使用。也不影响正常下单，因为下单后，订单系统写入消息队列就不再关心其他的后续操作了。实现订单系统与库存系统的应用解耦

### 3.2、异步通信
场景说明：用户注册后，需要发注册邮件和注册短信。传统的做法有两种。<font color="#FF0000">**串行和并行**</font>

a、串行方式：将注册信息写入数据库成功后，发送注册邮件，再发送注册短信。以上三个任务全部完成后，返回给客户端。

![image](http://otqvaruzt.bkt.clouddn.com/_03.png)

b、并行方式：将注册信息写入数据库成功后，发送注册邮件的同时，发送注册短信。以上三个任务完成后，返回给客户端。与串行的差别是，并行的方式可以提高处理的时间

![image](http://otqvaruzt.bkt.clouddn.com/_04.png)

假设三个业务节点每个使用50毫秒钟，不考虑网络等其他开销，则串行方式的时间是150毫秒，并行的时间可能是100毫秒。
因为CPU在单位时间内处理的请求数是一定的，假设CPU1秒内吞吐量是100次。则串行方式1秒内CPU可处理的请求量是7次（1000/150）。并行方式处理的请求量是10次（1000/100）
小结：如以上案例描述，传统的方式系统的性能（并发量，吞吐量，响应时间）会有瓶颈。如何解决这个问题呢？

引入消息队列，将不是必须的业务逻辑，异步处理。改造后的架构如下：

![image](http://otqvaruzt.bkt.clouddn.com/_05.png)

按照以上约定，用户的响应时间相当于是注册信息写入数据库的时间，也就是50毫秒。注册邮件，发送短信写入消息队列后，直接返回，因此写入消息队列的速度很快，基本可以忽略，因此用户的响应时间可能是50毫秒。因此架构改变后，系统的吞吐量提高到每秒20 QPS。比串行提高了3倍，比并行提高了两倍。

### 3.3、流量削峰
流量削锋也是消息队列中的常用场景，一般在秒杀或团抢活动中使用广泛。
应用场景：秒杀活动，一般会因为流量过大，导致流量暴增，应用挂掉。为解决这个问题，一般需要在应用前端加入消息队列。
1. 可以控制活动的人数。
2. 可以缓解短时间内高流量压垮应用。

![image](http://otqvaruzt.bkt.clouddn.com/_06.png)

用户的请求，服务器接收后，首先写入消息队列。假如消息队列长度超过最大数量，则直接抛弃用户请求或跳转到错误页面。
秒杀业务根据消息队列中的请求信息，再做后续处理

## 四、阿里云
### 4.1、阿里MQ简介
1. 提供了TCP、HTTP、MQTT 三种协议层面的接入方式
2. 支持 Java、C++ 以及 .NET 不同语言
推荐大家直接看这个吧， 阿里云MQ文档链接： https://help.aliyun.com/product/29530.html?spm=5176.doc29533.3.1.zGwE4m

### 4.2、阿里MQ 消息类型	
#### 4.2.1、定时消息
Producer 将消息发送到 MQ 服务端，但并不期望这条消息立马投递，而是推迟到在当前时间点之后的某一个时间投递到 Consumer 进行消费，该消息即定时消息

```
Message msg = new Message(…)
long timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").parse("2016-03-07 16:21:00").getTime();
msg.setStartDeliverTime(timeStamp);
```

#### 4.2.2、延时消息
Producer 将消息发送到 MQ 服务端，但并不期望这条消息立马投递，而是延迟一定时间后才投递到 Consumer 进行消费，该消息即延时消息。

```
Message msg = new Message(…);
long delayTime = 3000;
msg.setStartDeliverTime(System.currentTimeMillis() + delayTime);

```

#### 4.2.3、顺序消息
顺序消息是 MQ 提供的一种按照顺序进行发布和消费的消息类型。顺序消息由两个部分组成：顺序发布和顺序消费。

1. 顺序发布：对于指定的一个 Topic，客户端将按照一定的先后顺序进行发送消息。
2. 顺序消费：对于指定的一个 Topic，按照一定的先后顺序进行接收消息，即先发送的消息一定会先被客户端接收到。

顺序消息类型分为两种：全局顺序和分区顺序。

#### 4.2.3、事物消息
事务消息的 Producer ID 不能与其他类型消息的 Producer ID 共用。
通过 ONSFactory.createTransactionProducer 创建事务消息的 Producer 时必须指定 LocalTransactionChecker 的实现类，处理异常情况下事务消息的回查。
事务消息发送完成本地事务后，可在 execute 方法中返回如下三种状态：
- TransactionStatus.CommitTransaction 提交事务，允许订阅方消费该消息。
- TransactionStatus.RollbackTransaction 回滚事务，消息将被丢弃不允许消费。
- TransactionStatus.Unknow 暂时无法判断状态，期待固定时间以后 MQ Server 向发送方进行消息回查。

## 五、NSQ
### 5.1、NSQ简介
NSQ是一个基于Go语言的开源的分布式实时消息平台。
NSQ可用于大规模系统的实时消息服务，它的设计目标是为在分布式环境下提供一个强大的去除中心化的分布式服务架构，可以每天处理数以亿计的实时消息。

### 5.2、NSQ四大组件
1. nsqlookupd：管理nsqd节点拓扑信息并提供最终一致性的发现服务的守护进程。
2. nsqd：负责接收、排队、转发消息到客户端的守护进程，并且定时向nsqlookupd服务发送心跳。
3. nsqadmin：nsq的web统计界面，可实时查看集群的统计数据和执行一些管理任务。
4. utilities：常见基础功能、数据流处理工具，如nsq_stat、nsq_tail、nsq_to_file、nsq_to_http、nsq_to_nsq、to_nsq。

### 5.3、主要功能
-	具有分布式且无单点故障的拓扑结构 支持水平扩展，在无中断情况下能够无缝地添加集群节点
-	低延迟的消息推送，参见官方提供的性能说明文档
-	具有组合式的负载均衡和多播形式的消息路由
-	既擅长处理面向流（高吞吐量）的工作负载，也擅长处理面向Job的（低吞吐量）工作负载
-	消息数据既可以存储于内存中，也可以存储在磁盘中
-	实现了生产者、消费者自动发现和消费者自动连接生产者，参见nsqlookupd
-	支持安全传输层协议（TLS），从而确保了消息传递的安全性
-	具有与数据格式无关的消息结构，支持JSON、Protocol Buffers、MsgPacek等消息格式
-	非常易于部署（几乎没有依赖）和配置（所有参数都可以通过命令行进行配置）
-	使用了简单的TCP协议且具有多种语言的客户端功能库
-	具有用于信息统计、管理员操作和实现生产者等的HTTP接口
-	为实时检测集成了统计数据收集器StatsD
-	具有强大的集群管理界面，参见nsqadmin

### 5.4、NSQ的安装
1. 	选择版本： nsq-1.0.0-compat.windows-amd64.go1.8.tar.gz。
2. 	运行 nsqlookupd   (TCP 监听端口： 4160,  HTTP 监听端口：4161)。
3. 	运行 nsqd --lookupd-tcp-address=127.0.0.1:4160   (TCP 监听端口：4150, HTTP监听端口：4151,  启动后会与 nsqlookupd 之间发送心跳)。
4. 	运行 nsqadmin --lookupd-http-address=127.0.0.1:4161  (HTTP监听端口：4171,  访问 http://ip:4171 可以看到监控UI)。
5. 	运行 curl -d 'hello world 1' 'http://127.0.0.1:4151/pub?topic=test' , 创建一个topic：test 且向test中发送 "hello world 1"。
6. 	运行 nsq_to_file --topic=test --output-dir=F:/tmp --lookupd-http-address=127.0.0.1:4161  将消息落盘
7. 	继续发布更多的消息： curl -d 'hello world 2' 'http://127.0.0.1:4151/pub?topic=test'。
8. 	启动后的视图如下：

![image](http://otqvaruzt.bkt.clouddn.com/_07.gif)

### 5.5、NSQ的架构

![image](http://otqvaruzt.bkt.clouddn.com/_08.png)

## 六、消息系统对比图

![image](http://otqvaruzt.bkt.clouddn.com/_09.png)




