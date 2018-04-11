---
title: 阿里云MQ与Spring整合
date: 2017-11-24
tags: [消息服务]
categories: [消息服务]
---

##### 导论
记录一下阿里云消息服务与Spring的整合，以及ProducerId与ConsumerId的管理，其他的消息服务也是类似（RocketMQ、Kafka），阿里云消息服务性能还是很可观的，虽然收费，单也推荐使用。

##### 整合
消息服务的概念就不想多说了，需要的可以去看官方文档，[参考文档](https://help.aliyun.com/product/29530.html?spm=5176.doc29532.3.1.GGkAkd)。

###### 创建topic

首先创建topic，如下图填好信息就OK了。

![图一](http://otqvaruzt.bkt.clouddn.com/%E5%9B%BE1.png)

创建成功之后是这样

![图二](http://otqvaruzt.bkt.clouddn.com/%E5%9B%BE2.png)

###### ProducerId的创建
![图三](http://otqvaruzt.bkt.clouddn.com/%E5%9B%BE3.png)

###### ConsumerId的创建
![图四](http://otqvaruzt.bkt.clouddn.com/%E5%9B%BE%E5%9B%9B.png)

###### 整合spring
上面那些步骤信息填完整之后topic、ProducerId、ConsumerId都创建好了就可以使用消息队列了

Producer的整合
```
	 <bean id="producer" class="com.aliyun.openservices.ons.api.bean.ProducerBean" init-method="start"
          destroy-method="shutdown">
        <property name="properties"> <!--生产者配置信息-->
            <props>
            	<!-- 生成者ID，需要提前在阿里云创建  -->
                <prop key="ProducerId">PID-SIT-TransitHub-NotifyUnbind</prop> <!--请替换为自己的账户信息-->
                <!-- AccessKey、SecretKey由阿里云分配 -->
                <prop key="AccessKey">LTAIqfzogBNFeohh11</prop>
                <prop key="SecretKey">zoahuhZKscEk5Q8Qtr</prop>
                <!-- 根据自己服务器选择不同的tcp接入url,此处选择公网 -->
                <prop key="ONSAddr">http://onsaddr-internet.aliyun.com/rocketmq/nsaddr4client-internet</prop>
            </props>
        </property>
    </bean>
```

Consumer的整合
```
<!-- 创建Listener将消费者处于阻塞状态，只要有自己topic订阅的消息发布消息马上就会订阅到-->
<bean id="tsmDeleteAidMsgListener" class="com.snowball.hub.msg.DataMessageListener" /> Listener配置
<bean id="consumer" class="com.aliyun.openservices.ons.api.bean.ConsumerBean"
	init-method="start" destroy-method="shutdown">
	<property name="properties">
		<props>
			<prop key="ConsumerId">CID-SIT-OPS-NotifyUnbind</prop> 
			<prop key="AccessKey">${access_key}</prop>
			<prop key="SecretKey">${secret_key}</prop>
			<!--将消费者线程数固定为50个,该线程不会和主业务线程耦合-->
			<prop key="ConsumeThreadNums">50</prop>
		</props>
	</property>
	<property name="subscriptionTable">
		<map>
			<entry value-ref="tsmDeleteAidMsgListener">
				<key>
					<bean class="com.aliyun.openservices.ons.api.bean.Subscription">
						<!-- 此处填将之前创建的topic -->
						<property name="topic" value="snb-test-topic4" />
						<property name="expression" value="*" />
						<!--
						expression即Tag，可以设置成具体的Tag，如 taga||tagb||tagc，也可设置成*。 *仅代表订阅所有Tag，不支持通配
						-->
					</bean>
				</key>
			</entry>
			更多的订阅添加entry节点即可
		</map>
	</property>
</bean>
```

Consumer的整合和Producer基本一致，不同的是需要创建一个Listener,作用已经在注释中说明。

##### 使用阿里云sdk发布和订阅消息

上面只是整合了普通消息，阿里云MQ消息分很四种，每一种的整合API都不一样，具体整合细节可以参考文章开始出的参考文档。

###### 发布消息

```
public class ProducerTest {
    
    //如果和spring整合了，那就直接注入就好了，本次使用传统的发布方式
    //@Autowired
	//private Producer producer;
	
	//topic的管理最好做成可配置，可以对应不同的环境管理不同的topic，本次还是使用传统的//方式发布
	//@Value("#{configProperties['send_unbind_topic']}")
	//private String send_unbind_topic;

 public static void main(String[] args) {
     Properties properties = new Properties();
     // 您在MQ控制台创建的Producer ID
     properties.put(PropertyKeyConst.ProducerId, "XXX");
     // 鉴权用AccessKey，在阿里云服务器管理控制台创建
     properties.put(PropertyKeyConst.AccessKey,"XXX");
     // 鉴权用SecretKey，在阿里云服务器管理控制台创建
     properties.put(PropertyKeyConst.SecretKey, "XXX");
     // 设置 TCP 接入域名（此处以公共云的公网接入为例）
     properties.put(PropertyKeyConst.ONSAddr,
       "http://onsaddr-internet.aliyun.com/rocketmq/nsaddr4client-internet");

     Producer producer = ONSFactory.createProducer(properties);
     // 在发送消息前，必须调用start方法来启动Producer，只需调用一次即可
     producer.start();

     //循环发送消息
     while(true){
         Message msg = new Message( //
             // 在控制台创建的Topic，即该消息所属的Topic名称
             "TopicTestMQ",
             // Message Tag,
             // 可理解为Gmail中的标签，对消息进行再归类，方便Consumer指定过滤条件在MQ服务器过滤
             "TagA",
             // Message Body
             // 任何二进制形式的数据， MQ不做任何干预，
             // 需要Producer与Consumer协商好一致的序列化和反序列化方式
             "Hello MQ".getBytes());
         // 设置代表消息的业务关键属性，请尽可能全局唯一，以方便您在无法正常收到消息情况下，可通过MQ控制台查询消息并补发
         // 注意：不设置也不会影响消息正常收发
         msg.setKey("ORDERID_100");
         // 发送消息，只要不抛异常就是成功
         // 打印Message ID，以便用于消息发送状态查询
         SendResult sendResult = producer.send(msg);
         System.out.println("Send Message success. Message ID is: " + sendResult.getMessageId());
     }

     // 在应用退出前，可以销毁Producer对象
     // 注意：如果不销毁也没有问题
     producer.shutdown();
 }
}
```
消息发布成功可以看到sendResult是这样的信息
```
{"messageId":"0200010546D011E87BD078ACF4180003","topic":"TPC-SIT-COM-TransitHub-NotifyUnbind"}
```
根据messageId可以定位这条消息的轨迹，可以很清晰的定位消息的消费轨迹。
![图五](http://otqvaruzt.bkt.clouddn.com/%E5%9B%BE%E4%BA%94.png)

###### 订阅消息

```
public class ConsumerTest {
    public static void main(String[] args) {
        Properties properties = new Properties();
        // 您在MQ控制台创建的Consumer ID
        properties.put(PropertyKeyConst.ConsumerId, "XXX");
        // 鉴权用AccessKey，在阿里云服务器管理控制台创建
        properties.put(PropertyKeyConst.AccessKey, "XXX");
        // 鉴权用SecretKey，在阿里云服务器管理控制台创建
        properties.put(PropertyKeyConst.SecretKey, "XXX");
        // 设置 TCP 接入域名（此处以公共云公网环境接入为例）
        properties.put(PropertyKeyConst.ONSAddr,
          "http://onsaddr-internet.aliyun.com/rocketmq/nsaddr4client-internet");

        Consumer consumer = ONSFactory.createConsumer(properties);
        //这个Listener如果之前已经在spring容器中注册过直接使用就好了，这里就不演示了
        consumer.subscribe("TopicTestMQ", "*", new MessageListener() {
            public Action consume(Message message, ConsumeContext context) {
                System.out.println("Receive: " + message);
                return Action.CommitMessage;
            }
        });
        consumer.start();
        System.out.println("Consumer Started");
    }
}

```

消息的发布与订阅就这么多，要使用消息服务总结起来就四步。
1. 开通服务
2. 申请资源
3. 发布消息
4. 订阅消息

##### 总结
消息的产品很多，阿里云的消息服务是目前互联网公司使用占比很大的，本次只是很简单介绍消息服务的使用，具体实现细节笔者也在学习中。




