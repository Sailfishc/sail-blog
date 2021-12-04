---
title: "个人博客自建图床"
date: 2021-12-04T09:56:18+08:00
draft: false
toc: false
images:
tags:
  - 博客
  - 图床
---

![image-20211204091353741](https://static.sailfishc.cn/uPic/image-20211204091353741.png)

## 前言

写博客时间断断续续也有几年了，从最开始的CSDN，到简书，后面自己搭建博客，让我认识到了数据属于自己的重要性，自己建立博客的选择太多了，但是最终回归到博客输出的本质，就是文字和图片，其中图片需要额外的存储，CSDN和简书的图片是在博客托管的平台，有些链接也已经失效了，早期自己的博客图片依赖的一些图床（微博图床等）不再提供服务，让我意识到自己应该管理自己的数据，虽然增加了一些成本，但是是值得的，这篇文章总结了自己写博客的一些流程，不算最佳实践，但是用的也算是比较舒服，记录一下。

## 准备

博客托管的方式太多了，我就不一一展开了，不管是动态博客，还是静态博客，都有一些必须的东西，我已我的流程为例：

- 域名
- 阿里云账号
- Cloudflare账号

### 域名

域名我推荐使用Cloudflare申请，[Cloudflare Domain Register](https://blog.cloudflare.com/cloudflare-registrar/) 的博客里边承诺：**we promise to never charge you anything more than the wholesale price each TLD charges**. 这一句话我就被圈粉，现在的Domain注册商第一年的价格很便宜，但是后续续费的时候就会变得很贵，另外一个原因是我目前使用的是Cloudflare Pages来托管博客，用的也是Cloudflare提供的Https服务，所以就在Cloudflare购买域名或者是NS服务器使用的是Cloudflare的

域名为什么是必须的：我觉得域名是对于博客的一层外部抽象，有了域名，如果你要更换托管站，例如Github提供的域名是（sailfishc.github.io)迁移到Cloudflare（sailfishc.cloudflare.com)，这种情况下访问入口发生了变化，RSS地址也会发生变化，所以有自己的域名我认为是自己建立博客必须的一个步骤



## 详细步骤



### 创建OSS Bucket

<img src="https://static.sailfishc.cn/uPic/image-20211204095625172.png" alt="创建OSS" style="zoom:70%;" />

> 什么是OSS

这篇 [What is Object Storage and Why Should You Care?](https://cloudian.com/blog/object-storage-care/) 讲的挺好的，通俗的来说，就是用来存储图片，可以使用Http请求来读取照片



> 注意点

对于OSS创建需要注意的就是权限，其他的都跟着默认走就可以，如果准备直接使用OSS来作为图片访问的话，这里需要设置为公共读（但是不建议），建议设置为私有，配合CDN使用



### 创建CDN



### 添加域名信息



<img src="https://static.sailfishc.cn/uPic/image-20211204132131274.png" alt="image-20211204132131274" style="zoom:70%;" />

<img src="https://static.sailfishc.cn/uPic/image-20211204132036466.png" alt="源站信息" style="zoom:80%;" />

- 加速域名：通俗的来说就是用来加速博客图片访问速度的域名，比如阿里云OSS默认有一个bucket.oss-xxx.com的内部域名，我们要加速OSS内容访问的话，就可以用一个自己的域名image.sailfishc.cn来作为加速域名
- 源站: 指运行业务的网站服务器，是加速分发数据的来源。源站可用来处理和响应用户请求，当边缘节点没有缓存用户请求的内容时，节点会返回源站获取资源数据并返回给用户。阿里云CDN的源站可以是对象存储OSS、函数计算、自有源站（IP、源站域名）

> 如果网站是https的，源站的端口选择使用443，否则使用80



### CNAME配置



![image-20211204132703345](https://static.sailfishc.cn/uPic/image-20211204132703345.png)

CNAME的配置之后基本上就完成了最后一步，由于请求是「客户端」--》 「DNS」服务器 ，解析image.sailfishc.cn，如果要用CDN的话需要讲域名指向到阿里云CDN，所以这一步完成之后需要在域名服务商（例如我的是Cloudflare）设置CNAME的值



### 其他配置



![image-20211204133105025](https://static.sailfishc.cn/uPic/image-20211204133105025.png)

最后的一步就是配置https和用于防盗链的配置，对于防盗链的作用是如果你的图片或者是资源，被其他人复制地址到其他地方来使用，由于CDN是按流量付费， 所以如果你不希望其他人复制你的链接过去的话可以设置防盗链，只有特定的站点或者域名可以看到图片



### 权限配置



![image-20211204134358461](https://static.sailfishc.cn/uPic/image-20211204134358461.png)

- 在 [阿里云RAM控制台](https://ram.console.aliyun.com/users/sailfishc) 新增用户，配置权限（AliyunOSSFullAccess）

## 配置uPic



![image-20211204133421040](https://static.sailfishc.cn/uPic/image-20211204133421040.png)

![image-20211204133453546](https://static.sailfishc.cn/uPic/image-20211204133453546.png)

- 第一个图片是配置uPic的阿里云OSS配置
- 第二个图片是配置Typora的图片配置，可以基于uPic实现写文章的时候方便的插入图片



## 总结

写博客是一件很容易出发的事情，但是由于很多问题，大家把乐趣放在了如何搭建一个博客上面，也出现了今天使用的是Hugo，明天是Ghost，后天又是Hexo，博客作为自己思想自由表达的地方，不管张草多久，还记得回来就好，就像我秉承的健身观念一样，先找到自己舒服的工具，打造适合自己的流程，让自己先动起来，战胜惰性，后面形成习惯，虽然博客没有写很多，但是我一直在记录，这篇文章主要解决下博客图片的问题，让博客图片404消失！

## Reference

- [国内自建图床指南](https://lutaonan.com/blog/aliyun-cdn-tutorial/)
- [Typlog 自定义域名](https://docs.typlog.com/en/article/domain/) ：对于DNS的一些知识有讲解，例如Root Domain，A、CNAME等



