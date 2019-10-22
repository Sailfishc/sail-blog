---
title: 消息队列之RabbitMQ基础
date: 2017-04-23 14:37:00
description: "消息队列之RabbitMQ基础"
categories: [MQ]
tags: [Java, MQ, RabbitMQ]
---


> MQ在工作中用途还是比较多的，RabbitMQ又是比较容易上手并且在企业中用的比较多的一种消息服务，本篇文章借鉴于ginobefun的文章和ConanLi的文章，一方面是加深理解，一方面也是补充自己在MQ的不足。

## 一、AMQP基础

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-845cc2702f14ced5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 二、RabbitMQ

###  基础概念
- 生产者、消费者
- 队列
- 交换器
- 绑定

### 生产者、消费者
- 生产者（producer）创建消息，然后发送到代理服务器（RabbitMQ）
- 消费者（consumer）连接到代理服务器上，并订阅到队列（queue）上；当消费者接收到消息时，它只得到消息的一部分：有效载荷（标签并没有随有效载荷一同传递）
- 信道（channel）建立在”真实的”TCP连接内的虚拟连接；不论是发布信息、订阅队列或是接收消息，都是通过信道完成的；不使用TCP连接主要是因为对于操作系统而言建立和销毁TCP会话非常昂贵的开销；在一条TCP连接上创建多少条信道是没有限制的
- 消息包含两部分：有效载荷（payload）和标签（label）；有效载荷就是你想要传输的数据（可以是任何格式的任何内容）；标签描述了有效载荷，并且RabbitMQ用它来决定谁将获得消息的拷贝（之后举例说明）

### 队列(queue)

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-f86bac7c48c13760.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- Queue（队列）是RabbitMQ的内部对象，用于存储消息  
- 主体流程： 
  - 队列类似一个broker角色，生产者将内容（消息）发送到队列
  - 队列进行存储，消费者将消息消费
  - 消费者确认消费消息（ack）
- 生产者和消费者都可以通过来创建队列：
```
channel.queueDeclare(QUEUE_NAME, durable, exclusive, autoDelete, arguments);
```
  - durable：队列名称，不指定则随机生成
  - exclusive：设置为true则为私有队列，只有当前消费者可以订阅；
  - autoDelete：设置为true时最后一个消费者取消订阅将自动移除队列；
  - arguments：参数

### 交换器&绑定

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-21d9bdcc89b8e154.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

RabbitMQ的消息不是直接从生产者发送到队列的，而是要经过交换器然后才可以到达队列：
- 生成者把消息发布到交换器上；
- 消息最终到达队列，并被消费者接收；
- 绑定决定了消息如何从交换器到特定的队列；

四种交换器类型：
- fanout：把所有发送到该Exchange的消息路由到所有与它绑定的Queue中
- direct：把消息路由到bindingKey与routingKey完全匹配的Queue中
- topic：把消息路由到bindingKey与routingKey模糊匹配的Queue中
- headers：headers类型的Exchange不依赖于routingKey与bindingKey的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配

### 1、fanout

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-258010ad115af8ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 生产者（P）发送到Exchange（X）的所有消息都会路由到图中的两个Queue，并最终被两个消费者（C1与C2）消费。

### 2、direct

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-9e5a006e5b703846.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- routingKey=”error”发送消息，则会同时路由到Queue1（amqp.gen-S9b…）和Queue2（amqp.gen-Agl…）
- routingKey=”info”或routingKey=”warning”发送消息，则只会路由到Queue2
- 以其他routingKey发送消息，则不会路由到这两个Queue中

### 3、topic

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-5a35611ee3132828.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- routingKey=”quick.orange.rabbit”发送信息，则会同时路由到Q1与Q2
- routingKey=”lazy.orange.fox”发送信息，则只会路由到Q1
- routingKey=”lazy.brown.fox”发送消息，则只会路由到Q2
- routingKey=”lazy.pink.rabbit”发送消息，则只会路由到Q2（只会投递给Q2一次，虽然这个routingKey与Q2的两个bindingKey都匹配）
- routingKey=”quick.brown.fox”、routingKey=”orange”、routingKey=”quick.orange.male.rabbit”发送消息，则会被丢弃，它们并没有匹配任何bindingKey

### 4、header
headers类型的Exchange不依赖于routingKey与bindingKey的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配。在绑定Queue与Exchange时指定一组键值对；当消息发送到Exchange时，RabbitMQ会取到该消息的headers（也是一个键值对的形式），对比其中的键值对是否完全匹配Queue与Exchange绑定时指定的键值对；如果完全匹配则消息会路由到该Queue，否则不会路由到该Queue。

## 三、SpringBoot+RabbitMQ实战
** Docker环境下安装RabbitMQ**
```
# 下载rabbitmq的docker镜像和managerment的镜像
docker pull rabbitmq:management
# 启动rabbitmq镜像
docker run -d --name rabbitmq --publish 5671:5671 \
 --publish 5672:5672 --publish 4369:4369 --publish 25672:25672 --publish 15671:15671 --publish 15672:15672 \
rabbitmq:management
```
- 端口解释：
  - 4369:epmd(Erlang Port Mapper Daemon)
  - 25672:Erlang distribution
  - 5672, 5671:AMQP 0-9-1 without and with TLS
  - 15672:if management plugin is enabled
  - 61613, 61614:if STOMP is enabled
  - 1883, 8883:if MQTT is enabled

- 默认访问路径：`http://localhost:15672`
- 默认用户名和密码：`guest/guest`

> 介于代码粘贴进来比较多，提供项目[GitHub](https://github.com/Sailfishc/learn-rabbitmq)地址，本文提取代码片段进行讲解

### 项目结构
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-8a6f1499f3aa21ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 关键参数
- @RabbitListener(queues = "xxx")
- RabbitTemplate
- TopicExchange
- Binding
- Queue

### 代码段讲解
- 创建队列
```
  @Bean
    public Queue ssdQueue(){
        return new Queue("hello");
    }
```
- 创建topic类型交换器
```
@Bean
    public TopicExchange topicExchange() {
        return new TopicExchange("topicExchange");
    }
```
- 创建fanout类型的交换器
```
@Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanoutExchange");
    }
```
- 绑定topic类型的交换器
```
@Bean
    public Binding bindingExchangeTopicA(Queue topicAQueue, TopicExchange topicExchange) {
        return BindingBuilder.bind(topicAQueue).to(topicExchange).with("topic.a");
    }
```
- 绑定fanout类型的交换器
```
@Bean
    public Binding bindingExchangeFanoutC(Queue fanoutCQueue, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutCQueue).to(fanoutExchange);
    }
```

## 四、总结
本片文章主要讲解了以下RabbitMQ的一些基础概念和使用SpringBoot和RabbitMQ整合的几个案例，也是笔者结合博客和官网写的一片总结，总结的也比较基础，没有包括一些高级的内容，例如事务，最终一致性，重复消息和顺序消息的处理等等。