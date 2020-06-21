---
title: "利用Telegram订阅RSS"
date: 2020-06-14T21:16:05+08:00
draft: false
toc: true
images:
tags:
  - RSS
  - TG
  - Bot
---


> 订阅优质的博客资源是学习精进的好方法，RSS是达成这一目的一个好工具。  

之前一直在用的是`Feedly`，但是用着用着`Feedly`打开的频率越来越低了，后来看到木子在用`Telegram`做了RSS，我也就比较趋向于这种方式了，这篇文章就不介绍RSS的一些概念了，就直接开始展示工作流以及如何使用吧，奥利给！

## 基础流程

![RSS基础流程](https://sail-blog.oss-cn-beijing.aliyuncs.com/uPic/6xQZHF.jpg)

- 其中带颜色的是需要自己部署的

> 流程分析  

- 在TG上新建一个Bot
- 将Bot加入自己的Channel或者是Group中
- 部署一个RSS服务
- 在Bot中输入想要订阅的网站订阅
- TG服务器收到消息
- RSS服务监听TG服务器消息，当你在TG客户端的Bot中加入了想要订阅的网站后，RSS服务是能得到信息的
- RSS服务每5分钟抓取一次目标网站，看是否有更新
- 如果有更新，则使用TG的OpenAPI给所在的Channel或者Group推送消息


## 搭建服务

### 准备Bot

- @BotFather，使用`/start`
- 输入Bot的name和userName
- BotFather会返回这个Bot的Token，一般是数字：字母，Token是极其重要的，是和TG的OpenAPI做交互的唯一凭证

### 将Bot加入Channel或者

- 将Bot作为Admin加入自己的Channel或者Group

> 测试Bot，获取Chat信息（不是必要操作），具体可以参考[Telegram Bot - how to get a group chat id? - Stack Overflow](https://stackoverflow.com/questions/32423837/telegram-bot-how-to-get-a-group-chat-id)  

- 按照这个格式输入到浏览器：`https://api.telegram.org/bot${token}/getUpdates`，例如：`https://api.telegram.org/bot12036334:A2zvFH8VYZ/getUpdates`
- 还可以按照这个格式以Bot的名义发送消息：`https://api.telegram.org/bot${token}/sendMessage?chat_id=${chat_id}&text=hello`

### 部署RSS服务

> 必要条件  

- 会使用Docker
- 有一台科学上网的VPS
- 有公网IP

在看了下RSS服务之后，有两个还不错的

- [GitHub - fengkx/NodeRSSBot: Another Telegram RSSBot  but in Node.js Telegram RSS 机器人](https://github.com/fengkx/NodeRSSBot)
- [GitHub - cbrgm/telegram-robot-rss: A clean and easy to use RSS Newsfeed Bot for fabulous Telegram Messenger App!](https://github.com/cbrgm/telegram-robot-rss)

因为有同学已经用了NeatRss了，所以就用了第一个，也有现成的Docker-compose脚本，直接用就OK啦，感谢。

```
version: '3.1'

services:
  rssbot:
    image: fengkx/node_rssbot
    container_name: node_rssbot
    restart: always
    volumes:
      - ./data:/app/data/
    ports:
      - 3306:3306
    environment:
      RSSBOT_TOKEN: 123456:ACEYzXbuuaxxxxxxxxxxxxxxtrvmdq8
    networks:
      - my-bridge

networks:
  my-bridge:
    driver: bridge
```

- `docker-compose up -d`

### 成果啦

![订阅成功](https://sail-blog.oss-cn-beijing.aliyuncs.com/uPic/41Lcvz.jpg)

### 最后

RSS的RSS Bot解决了如何抓取订阅的博客或者网站，其中还有一个比较重要的就是有些内容是不支持RSS的，这个时候可以配合`RSSHub`来使用，利用`RSSHub`获取到内容的Rss地址，再利用今天介绍的工具将其加入TG链中。

- 订阅一时爽，一直订阅一直爽

## 参考资料

- [Telegram 电报机器人](https://chanshiyu.com/#/post/108)
- [Telegram RSS 订阅频道](https://chanshiyu.com/#/post/111)