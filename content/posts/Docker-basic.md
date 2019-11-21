---
title: "开发人员必备的Docker基础"
date: 2019-11-10T16:01:41+08:00
draft: false
toc: true
tat:
    - Docker
---

## Docker简介
说到Docker，我们就来谈谈业务应用从旧时代到Docker的发展历程。

### 旧时代

旧时代是基于Application运转的，一台服务器（无虚拟机时代）部署多个Application，不管是Windows还是Linux其实是无法保证在一台服务器上稳定安全的运行多个应用的，CPU，内存等公共资源的竞争导致应用极为不稳定。

### VM阶段

VM让世界变得美好了，VM算是划时代意义的技术，但是VM不是完美的，VM会占用宿主机额外的CPU,RAM和存储（VM不够轻量级），启动慢。

### Linux容器

容器技术出现的比较早，只是Docker将Linux容器技术广泛应用了，Docker解决了VM的问题以外，还有如下优点：

- 容易上手
- 解决了运维中的环境问题及服务调度的痛点

## Docker安装

Docker提供了桌面安装和服务器安装，对于开发来说，我们可以归类为：

- Windows平台
- Mac平台
- Linux平台：Ubuntu/CentOS

### Windows平台

硬件要求：

- win10 64位
- 启用Hyper-V和容器特性

安装步骤：[Install Docker Desktop on Windows | Docker Documentation](https://docs.docker.com/docker-for-windows/install/)

- 下载Docker for windows安装包： [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop) 
- 根据安装向导安装
- 成功后再CMD中输入`docker version`查看版本，如果有版本信息表示安装成功

### Mac平台

Mac版本和Windows桌面版安装类似，找到下载包，静默安装即可：
- 下载地址：下载Docker for windows安装包： [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop) 

### Ubuntu安装

- Ubuntu安装：[Get Docker Engine - Community for Ubuntu | Docker Documentation](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

Ubuntu有几种方式安装Docker，我们使用便捷脚本来安装Docker

- 在Linux打开Shell
- 执行：`curl -fsSL https://get.docker.com -o get-docker.sh`
- 执行：`sudo sh get-docker.sh`
- 执行：`sudo usermod -aG docker your-user`
- 使用docker —version查看docker版本

> 备注：
> 1. 不要使用root用户使用Docker
> 2. 需要将非root用户到本地Docker Unix组当中：`sudo usermod -aG docker your-user` , your-user为执行Docker的用户名

卸载Docker-CE：

- `sudo apt-get purge docker-ce`
- `sudo rm -rf /var/lib/docker`


### CentOS安装

- CentOS安装： [https://docs.docker.com/install/linux/docker-ce/centos/](https://docs.docker.com/install/linux/docker-ce/centos/) 
- 安装方式同Ubuntu

卸载Docker-ce
- `sudo yum remove docker-ce`
- `sudo rm -rf /var/lib/docker`


## Docker镜像与容器
镜像和容器是Docker的重要概念，在本小节主要会有一下内容：

- 镜像和容器简介
- 搜索镜像
- 下载镜像
- 删除镜像
- 运行容器
- 停止容器
- 删除容器
- 命令合集

### 镜像和容器简介

对于开发来说，简单理解一下镜像：镜像就是Class，容器就是实例对象，一个Class可以有多个实例，每个实例即使内容一致，但是也不是相等的（不同的容器ID和名称）

### 搜索镜像

Docker安装后并没有镜像，需要从仓库中拉取（仓库可以类比Maven仓库，分为本地仓库，私有仓库和中央仓库），使用命令docker  search [image-name]，就可以看到一个redis仓库列表，列表中包含了以下元数据：

- 名称
- 描述
- star数
- 是否是官方
- 是否自动化

![搜索镜像](http://img.sailfishc.cn/20190825204152.png?imageslim)

### 下载镜像

在找到需要的镜像后，可以使用`docker pull [image-name]`来下载镜像

### 运行容器

基础运行容器： `docker run -it ubuntu:latest /bin/bash`

上面的命令是以交互形式运行了一个最后版本的ubuntu系统，默认进入的目录是/bin/bash，其中的一些参数介绍：

- run：运行容器的命令
- -i -t（可以合并为-it）：交互形式，还有如-d,&等形式
- ubuntu:latest：镜像名称（版本）
- /bin/bash：交互形式进入后的目录

> 如果当前没有该镜像，那么在运行容器时会先执行pull命令下载，然后再执行run命令，如果以-it参数启动容器后再bash中执行`ctrl-c`或者是执行exit命令会终止容器，原因是容器不存在任何进程则无法存在，可以使用`ctrl-pq`断开shell与容器终端的链接（容器不会终止）

![运行容器](http://img.sailfishc.cn/20190825205542.png?imageslim)

完整的一个docker run命令示例：`docker run -d --name my-redis --restart always redis`

解释如下：

- -d：后台运行模式
- —name：命名一个容器名称为my-redis
- —restart：always是一种简单的自我修复策略，表明除非明确停止（例如执行了stop命令），否则会一直尝试重启


### 镜像和容器命令合集

- 搜索镜像：docker search [image-name]
- 下载镜像：docker pull [image-name]
- 查看本地镜像列表：docker images
- 删除镜像：docker rm [docker-name]:[tag]
- 运行容器: docker run [image-name]
- 重新进入容器：docker exec -it container-id /bin/bash
- 查看容器列表:docker ps
- 停止容器：docker stop container-id
- 删除容器：docker rm container-id
- 启动容器: docker start container-id

> 备注：
> 1. image-name也可以使用image-id（唯一）
> 2. 删除镜像时如果镜像存在关联的容器，并且容器处于运行或者是停止状态，不允许删除镜像

### 实例：运行一个Web服务器

启动Nginx容器：`docker run --name some-nginx -d -p 8080:80 nginx`

> 以后台模式运行了nginx，名字为some-nginx，-p参数留到下一小节讲解，下图是启动后在本机（localhost）访问nginx容器，可以看到已经可以正常访问到nginx服务了。

![Nginx容器](http://img.sailfishc.cn/20190825211847.png?imageslim)


## 参考资料

- 深入浅出Docker
- Docker官网
