---
title: "RSS背后的逻辑"
date: 2019-12-01T18:09:12+08:00
draft: false
---

## 目录
- 前言
- 介绍RSS
- RSS的背后的逻辑论述
- RSS实践
- 总结
- 参考资源

## 前言
其实知道RSS也挺久了，但是一直没应用起来，今天正式的了解了一下，发现RSS其实是很能创造生产力的一种工具。

## 介绍RSS

> **RSS** (originally **  Site Summary** [RDF](https://en.wikipedia.org/wiki/Resource_Description_Framework) ; later, two competing approaches emerged, which used the  [backronyms](https://en.wikipedia.org/wiki/Backronym)  **Rich Site Summary** and **Really Simple Syndication** respectively) [[2]](https://en.wikipedia.org/wiki/RSS#cite_note-powers-2003-1-2)  is a type of  [web feed](https://en.wikipedia.org/wiki/Web_feed)  [[3]](https://en.wikipedia.org/wiki/RSS#cite_note-Netsc99-3)  which allows users and applications to access updates to websites in a standardized, computer-readable format  

简单来说：RSS指的是允许以标准化计算机可读的方式让用户和应用程序访问网站更新。

## RSS的背后的逻辑论述
RSS的机制和软件设计中的一些方式很像：`publish/subscribe`发布订阅模式，用户作为服务的消费者，是采用主动去拉取（pull）源信息，还是被动的接收（push）信息，也是一个TradeOff的过程，很明显，对于用户来说，人不能像机器，定时的去（pull）数据下来，因为这样太浪费精力，所以RSS订阅的方式出现，提高了生产力。

可能大家都被这种情况困扰过，接收信息的渠道太多，技术博客，新闻网站，某个关注的人的网站，YouTube频道，某个关注的Twitter动态，关注的UP主的直播，购物网站的商品优惠活动，在现在这样信息满天飞，贩卖焦虑的时期，我们要学会过滤信息，只关注自己关心的优质的内容，并且是可学习的，随着时间的变化，我们只保留了这些最优质的的，和自己最匹配的内容，我们每个人在一天的精力都是有限的，不能在每一天都挨个打开各类APP，去浏览信息，去让我们的碎片时间消耗掉，到最后接受了太多的垃圾信息，没有将信息转化为知识，还消耗了一天的精力。

## RSS实践
> RSS订阅的过程是对自己信息源梳理的过程，我分析了下我自己的信息输入源：  

- 个人技术博客
- 公共技术博客网站的某个人的动态
- 关注的YouTube频道
- 关注的Github人和仓库
- 关注的Twitter人的时间线

不是所有的网站都有RSS订阅的，如果我们想订阅的话需要借助一些工具：

- RssHub：RSSHub 是一个开源、简单易用、易于扩展的 RSS 生成器，可以给任何奇奇怪怪的内容生成 RSS 订阅源
- Feedly：Keep up with all the topics that matter to you. All in one place.
- Reeder：本地阅读器

> RSS实践起来其实也比较简单，我现在的WorkFlow是这样的：  

- 安装RssHub的插件（收集信息的客户端，可以将没有RSS功能的网站添加RSS订阅源）
- 使用Feedly订阅对应的RSS源
- 每天使用Reeder或者是Feedly阅读（Reeder专注于阅读，阅读体验比Feedly要好）


## 总结

工具只是实现目标的一个桥梁，最重要的其实是接触的信息，以及将信息转化为知识的能力，有些人没有这些工具，依然很厉害，不要做那种懂得了很多了道理，但是依然过不好一生的人。

## 参考资源
- [RSS - Wikipedia](https://en.wikipedia.org/wiki/RSS)
- - [介绍 | RSSHub](https://docs.rsshub.app/)
- [通过 RSSHub 订阅不支持 RSS 的网站 - 少数派](https://sspai.com/post/47100)
- [全网最详细的Reeder 3使用攻略 - 知乎](https://zhuanlan.zhihu.com/p/44215137)
- [收下阅读神器 Reeder，一篇文章带你轻松入门 RSS - 知乎](https://zhuanlan.zhihu.com/p/43752773)