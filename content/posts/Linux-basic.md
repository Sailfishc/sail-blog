---
title: "基本的Linux命令"
date: 2020-06-21T21:06:09+08:00
draft: false
toc: false
images:
tags:
  - Linux
---
> 可能学习Linux最好的方式是自己搭建科学上网的时候去学习吧😑  

## 权限
- 创建用户：`sudo useradd -m -s /bin/bash username`
- 设置密码：`sodu passwd username`
- 将用户加入`sudo`组：`sudo usermod -G sudo username`
- 切换用户：`su -l username`

> 如果没有sudo组，可以使用  
> 1. 添加sudo组： `sudo groupadd sudo`  
> 2. 将sudo组加入`sudoer`：`sudo visudo`  
> 3. 配置：`%sudo   ALL=(ALL:ALL) ALL `  
> 4. 配置进行免密：`%sudo ALL=ALL NOPASSWD:ALL`  


- 给用户&用户组设置某个文件&文件夹的权限：`sudo chown -R acme:acme /etc/xx/live`
- 设置文件&文件夹的读写权限：`chmod -R 750 /etc/letsencrypt/live`

## 包管理

- 安装包：`sudo yum install -y curl`
- 服务管理：
	- 启动服务： `sudo systemctl start crond`
	- 设置自启动服务（如果服务器挂了重启之后再次启动应用）：`sudo systemctl enable crond`
	- 重启服务： `sudo systemctl restart nginx`

## 目录管理

- 创建多个文件夹：`sudo mkdir -p /etc/xx/live`

## 文本编辑&查看

- vi
- vim
- nano

## 服务信息

- 查看内核版本：`uname -a`

## 资料参考

- [写代码怎能不会这些 Linux 命令？ | 编程和酒](https://blog.biezhi.me/2017/08/write-code-must-linux-command.html)