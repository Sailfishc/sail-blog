---
title: Mac下Elasticsearch5.X和Head插件的安装
date: 2017-03-25 14:37:00
description: "本文主要讲解ES5.0的安装和Head插件的安装配置，将自己的坑给大家放出来！"
categories: [elasticsearch]
tags: [elasticsearch, Head]
---

> Elasticsearch5.x的安装：在ELK升级到5.0之后，有些特性发生了变化，例如全部使用JDK8，插件变化和环境校验等。

### 一、安装Elasticsearch
- 在官网下载tar包，[下载地址](https://www.elastic.co/downloads/elasticsearch)
- 下载之后解压
- 进入bin目录下启动
> 备注：如果是root用户启动会出现异常，因为es出去安全性考虑，禁止以root用户启动，解决办法是新建一个用户，详情请看这篇博客：[CENTOS安装ElasticSearch](https://my.oschina.net/topeagle/blog/591451?fromerr=mzOr2qzZlogstash)。

- 进入es的conf目录修改配置文件
```
cluster.name: es-cluster
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
```

- 启动ES，命令如下
    - ./elasticsearch
    - ./elasticsearch -d (这是后台启动）

- 看到如下日志信息：
```
[2017-03-25T14:30:45,189][INFO ][o.e.n.Node               ] [node-1] initializing ...
[2017-03-25T14:30:45,272][INFO ][o.e.e.NodeEnvironment    ] [node-1] using [1] data paths, mounts [[/ (/dev/disk1)]], net usable_space [278.8gb], net total_space [464.6gb], spins? [unknown], types [hfs]
[2017-03-25T14:30:45,272][INFO ][o.e.e.NodeEnvironment    ] [node-1] heap size [1.9gb], compressed ordinary object pointers [true]
[2017-03-25T14:30:45,273][INFO ][o.e.n.Node               ] [node-1] node name [node-1], node ID [n1HFjO-TQlSs4Ncw0HD34A]
[2017-03-25T14:30:45,276][INFO ][o.e.n.Node               ] [node-1] version[5.2.2], pid[3763], build[f9d9b74/2017-02-24T17:26:45.835Z], OS[Mac OS X/10.12.3/x86_64], JVM[Oracle Corporation/Java HotSpot(TM) 64-Bit Server VM/1.8.0_111/25.111-b14]
[2017-03-25T14:30:45,971][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [aggs-matrix-stats]
[2017-03-25T14:30:45,971][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [ingest-common]
[2017-03-25T14:30:45,971][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [lang-expression]
[2017-03-25T14:30:45,971][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [lang-groovy]
[2017-03-25T14:30:45,971][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [lang-mustache]
[2017-03-25T14:30:45,971][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [lang-painless]
[2017-03-25T14:30:45,971][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [percolator]
[2017-03-25T14:30:45,971][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [reindex]
[2017-03-25T14:30:45,972][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [transport-netty3]
[2017-03-25T14:30:45,972][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [transport-netty4]
[2017-03-25T14:30:45,972][INFO ][o.e.p.PluginsService     ] [node-1] no plugins loaded
[2017-03-25T14:30:47,936][INFO ][o.e.n.Node               ] [node-1] initialized
[2017-03-25T14:30:47,936][INFO ][o.e.n.Node               ] [node-1] starting ...
[2017-03-25T14:30:48,119][INFO ][o.e.t.TransportService   ] [node-1] publish_address {192.168.31.110:9300}, bound_addresses {[::]:9300}
[2017-03-25T14:30:48,123][INFO ][o.e.b.BootstrapChecks    ] [node-1] bound or publishing to a non-loopback or non-link-local address, enforcing bootstrap checks
[2017-03-25T14:30:51,226][INFO ][o.e.c.s.ClusterService   ] [node-1] new_master {node-1}{n1HFjO-TQlSs4Ncw0HD34A}{gn6UwArmTG-DEwYmzh1U9g}{192.168.31.110}{192.168.31.110:9300}, reason: zen-disco-elected-as-master ([0] nodes joined)
[2017-03-25T14:30:51,240][INFO ][o.e.h.HttpServer         ] [node-1] publish_address {192.168.31.110:9200}, bound_addresses {[::]:9200}
[2017-03-25T14:30:51,240][INFO ][o.e.n.Node               ] [node-1] started
[2017-03-25T14:30:51,246][INFO ][o.e.g.GatewayService     ] [node-1] recovered [0] indices into cluster_state
```
- 通过IP访问，例如我的IP是192.168.31.110，那么访问地址就是192.168.31.110:9200，可以看到如下信息
```
{
name: "n1HFjO-",
cluster_name: "elasticsearch",
cluster_uuid: "njH6T6eMS-mDf3tloN5THg",
version: {
number: "5.2.2",
build_hash: "f9d9b74",
build_date: "2017-02-24T17:26:45.835Z",
build_snapshot: false,
lucene_version: "6.4.1"
},
tagline: "You Know, for Search"
}
```

### 二、安装Head插件
> Head插件是我们常用的插件，但是在ELK5.0以后按照之前的插件安装方式不能使用了，看了head官网之后，要单独启动一个服务才可以。

- [Head-GitHub官网](https://github.com/mobz/elasticsearch-head)
- 官网简易教程
```
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install
grunt server
```

**这里依赖Node环境，我们需要做一下工作：**
- Git
- [Node官网](https://nodejs.org/en/download/)

#### 1、Head插件安装
- 下载并安装对应的Node安装包
- 安装Git环境
- 下载Head项目
- cd elasticsearch-head
- 修改服务器监听地址，地址目录：head/Gruntfile.js
```
connect: {
    server: {
        options: {
            port: 9100,
            hostname: '*',
            base: '.',
            keepalive: true
        }
    }
}
```
- 修改连接地址，目录：head/_site/app.js

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-38c88f015bd861e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- npm install
> 备注：如果npm源特别慢的话，可以参考这篇文档：[npm install 无响应解决方案
](http://www.uedbox.com/npm-install-slow-solution/)
- grunt是一个很方便的构建工具，可以进行打包压缩、测试、执行等等的工作，5.0里的head插件就是通过grunt启动的。因此需要安装一下grunt：`npm install grunt-cli`
- grunt server：保证es服务已经启动

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-633cc8658676a775.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 打开浏览器输入：IP:9100

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-411df93d94cc0cb4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 正常的话可以看到已经连接了ES，但是ES社区中很多人反映都配置好了，但是看不到连接信息，这时候需要在在 es 的 elasticsearch.ym 里添加如下配置：
```
http.cors.enabled: true
http.cors.allow-origin: "*" 
```

> 2017-03-26补充：在安装x-pack之后head无法连接es，只能将es的xpack认证给关闭，关闭方式：
- 在Es的conf文件中配置xpack.security.enabled: false