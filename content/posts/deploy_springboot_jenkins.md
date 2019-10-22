---
title: Jenkins Maven Git SpringBoot Jar War Linux持续集成
date: 2017-03-24 15:12:00
description: "本文主要讲解使用Jenkins发布SrpringBoot项目，包括Jar包和War包"
categories: [后端学习]
tags: [后端学习，Jenkins, SpringBoot]
---

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-9d9aa000a2176fa1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 一、前言
公司一直使用的是Java语言进行开发，自然而然逐渐的使用SpringBoot替代原来的框架，特别是对于现在的spring cloud微服务来说，一个项目由多个小项目组成，每个小项目都独立部署，使用jenkins是最好的部署和管理工具了。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-727fcd1dd174863b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Jenkins大概就这么工作：
- 拉取GIT/SVN 等仓库的文件
- 然后使用Maven/Ant/Gradle等构件工具进行Build
- 构建成功之后会进行部署（deploy）

## 二、安装
- [Jenkins官网](https://jenkins.io/index.html)
- 下载War包或者Jar包都可以（推荐war包，适合新手，也比较稳定）

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-6ed5afc220bcf971.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 安装（略过）
- 安装条件
    - JDK
    - MAVEN
- 插件安装

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-31a2db10f939d233.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 必备插件
    - Git plugin
    - [Maven Integration plugin](http://wiki.jenkins-ci.org/display/JENKINS/Maven+Project+Plugin)
    - publish over ssh插件（用于上传打包好的项目到远程Linux）
- 插件列表（太多不一一列出）

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-ba5e79a14fc8f11f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 三、配置
> 此处省略jdk(请注意服务器上需要安装jdk，而不是jre)、maven、git的安装

### 1、系统配置

在系统管理中找到Global Tool Configurations,其中包含jdk、git、maven等工具的配置

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-6b6ad974687749b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-e6630d187141a908.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-c442bc89ef0430d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意： 请勾掉自动安装，自己手动安装以上工具后再进行配置

## 三、新建项目
这里我们选择创建Maven项目：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-f1f3c101baba2e12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果在源码管理中出现如下红色代码，说明是本机的用户没有配置Git用户授权


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-a3ff81beed0bd601.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这里使用了Git作为源码管理工具，先配置SSH Key，在Jenkins的证书管理中添加SSH。在Jenkins管理页面，选择“Credentials”，然后选择“Global credentials (unrestricted)”，点击“Add Credentials”，如下图所示，我们填写自己的SSH信息，然后点击“Save”，这样就把SSH添加到Jenkins的全局域中去了。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-d4812fcccba017d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 备注：Passphrase这里不用填值，这是自动生成的。
如何配置Git ssh解决上述ssh key问题，可参考以下教程链接
[http://www.linuxidc.com/Linux/2014-10/108080.htm](http://www.linuxidc.com/Linux/2014-10/108080.htm)

配置成功后选择配置的用户，就发现已经没有红色的提示，说明ssh key配置成功了
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-9aca3e863f2438bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1、构建

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-f8aad83a9637aca7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 备注：这里的Root Pom指的是根目录下的Pom文件

如果是图一结构，Root Pom为：pom.xml

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-5d60696a8822d328.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果为图二结构，Root Pom为：project_A/pom.xml

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-1fc8d3ad36650808.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2、授权服务器
打开jenkins首页，点击系统管理--系统设置，下拉找到找到publish over ssh,进行以下设置（请确保前面的步骤中publish over ssh插件已经安装成功，如果没有发现，那就是还没有安装成功，请返回去安装）。


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-91e845ed76874d9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3、部署Jar包
进入上面的已经创建好的jenkinsWeb项目，点击配置，下拉找到Post Steps进行配置。

１、点击

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-a4f2907d49e74db5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果没找到这项，证明publish over ssh没有安装成功。

2、接着进行下图配置

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-b95732df59e48d68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


参数说明：

Transfer SetSource files：表示要上传的本地的jar包及路径，可到工作空间去看。

Remove prefix:表示要上传时要去除的文件夹，即只上传jar包。

remote driectory:即表示执行时的路径，相当于把jar包上传到这里了。

exec commad:要执行的命令脚本。

> 构建脚本

```
# 将应用停止
#stop.sh
#!/bin/bash
echo "Stopping SpringBoot Application"
pid=`ps -ef | grep search-1.0-SNAPSHOT.jar | grep -v grep | awk '{print $2}'`
if [ -n "$pid" ]
then
   kill -9 $pid
fi
```

```
#replace.sh 用于将上次构建的结果备份，然后将新的构建结果移动到合适的位置
#!/bin/bash
# 先判断文件是否存在，如果存在，则备份
file="/home/app/mall-search/search-1.0-SNAPSHOT.jar"
if [ -f "$file" ]
then
   mv /home/app/mall-search/search-1.0-SNAPSHOT.jar /home/app/mall-search/backup/search-1.0-SNAPSHOT.jar.`date +%Y%m%d%H%M%S`
fi
mv /home/app/deploy/search-1.0-SNAPSHOT.jar /home/app/mall-search/search-1.0-SNAPSHOT.jar
```
```
# startup.sh 启动项目
#!/bin/sh
export JAVA_HOME=/home/shopin/jdk1.8.0_121
echo ${JAVA_HOME}
echo "授予当前用户权限"
chmod 777 /home/shopin/mall-search/search-1.0-SNAPSHOT.jar
echo "执行....."
cd /home/shopin/mall-search/
nohup ${JAVA_HOME}/bin/java -jar search-1.0-SNAPSHOT.jar > /dev/null &
echo "启动成功..."
```

这样就可以将jar包打到远程服务器了

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-7bee1d38208a4082.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4、部署War包容器（tomcat）中
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-c0f207c324028f18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果容器是tomcat，就需要在tomcat配置相应的用户名和密码：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-63057a4157bef0eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 以上就是Jenkins for SpringBoot，同样也可以发布到tomcat