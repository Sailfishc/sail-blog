---
title: 通过Hugo+Github快速搭建博客
date: 2019-10-24T15:05:56+08:00
draft: false
toc: false
images:
tags:
  - Hugo
---


> 写在前面  

程序员总想有自己的一个博客，厌倦了CSDN，简书，掘金这些博客网站，那就自己搭建一个吧，这篇文章包含了从0开始搭建到部署，可以没有自己的域名，自己的服务器，但是面包总会有的。

先罗列下常见的博客网站：

- Hexo
- WordPress
- JekyII
- Ghost
- Hugo

Hexo是我之前用的博客网站，但是在使用过程中遇到了一些问题：

- node环境问题较多，本身对Node不熟悉，解决成本高
- node_modules过大，网络问题下载依赖过慢
- 编译为html过程越来越慢

后来发现了Hugo，发现Github已经3w多star了，Hugo官网对于Hugo的介绍是：`The world’s fastest framework for building websites`，总结起来就一句话，那就是快！！！

## 快速开始
- [Quick Start | Hugo](https://gohugo.io/getting-started/quick-start/)

Hugo安装很简单，就以Mac为例快速开始：

- 安装Hugo：`brew install Hugo`
- 查看Hugo版本：`Hugo version`
- 新建一个网站，名字为`quickstart`：`Hugo new site quickstart`
- 安装主题
	- 进入目录：`cd quickstart`
	- 使用Git Init创建Git仓库：`git init`
	- 下载主题，放入theme目录（名字为`ananke`）：`git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke`
	- 使用主题：`echo ‘theme = “ananke”’ >> config.toml`
- 新建MarkDown文章（也可以用现有的放入content/posts目录）：`Hugo new posts/my-first-post.md`
- 本地预览：`Hugo server -D`
- 打开浏览器：`http://localhost:1313`
- 搞定🌹

## 部署
本地搭建好了网站要部署到外部服务上，常见的方式有三种：

- 部署Github Pages
- 部署到自己服务器，用静态服务器（例如Nginx）上
- 使用一些云服务（推荐）

先简要介绍下流程，Hugo将Markdown转为静态资源（Html，Css，Js）等资源，放在public目录下，将public发布到静态服务器或者CDN下达到外部可以访问的效果。

对于自己购买服务器部署的方式目前是不推荐的，因为刚开始博客没有自己的品牌，浪费一些资源在服务器上的话是不划算的（也增加维护成本），所以就只考虑基于Github Pages和云服务这两种方式，对于Github Pages，简要说明下，Github Pages提供了3种方式：

- 新建Github用户名（例如`sailfishc`）的仓库：`sailfishc.github.io`
- 新建项目，在`master`分支下新建`docs`目录
- 新建项目，添加`gh-pages`分支

最开始本来想尝试第一种，但是还有一个博客已经使用这种模式了，暂时是用不了，因为第二种会将源代码和静态网站在一个分支中，也就不考虑，就采用了第三种，对于第三种方式的部署，最开始采用了Travis部署的方式，但是经过一些尝试，效果并不好（主要是对于Travis的脚本不熟悉，找了一些脚本效果比较差），最后也放弃了，有兴趣的小伙伴们可以看参考文章中对于Travis部署的一些博客，之后一段时间采用了本地执行脚本的方式，等文章写完后，执行脚本[push hugo site to Github gh-pages branch · GitHub](https://gist.github.com/Sailfishc/5c8861ace0469aec69f834d972da64f3) ，脚本会自动将生成静态public文件，然后加入`gh-pages`分支，然后将源代码push到`master`分支，public下的文件push到`gh-pages`分支，就可以使用`https://sailfishc.github.io/sail-blog`访问博客站点了。

大功告成了吗？没有，因为没使用第一种Github pages方案，这个链接包含了`项目这名`这一级路径，用着很难受，于是就有了之后的改版，也就是`Host On Render`的方案。

- [Host on Render | Hugo](https://gohugo.io/hosting-and-deployment/hosting-on-render/)

> 将网站部署到Render上，享受全球CDN服务，通过`Cloudflare`自定义域名  

- 打开Render
- 注册
- 创建`Webservice`
- 授予仓库权限
- 设置参数
	- Environment
	- Build Command
	- Publish Directory
- 将自定义域名加入Render，在Cloudflare配置DNS转发
- 搞定🌹

## 一些总结
> 中间走了很多弯路，到最后发现官方文档已经有很多成熟的方案，只是自己没认真看，借`左耳朵耗子的话`：花时间学习基础知识，花时间度文档，只要把基础打扎实，认真读下文档，就会生出很多时间。  


> 参考文章  

- [利用Travis CI和Hugo將Blog自動部署到Github Pages - AxdLog](https://axdlog.com/zh/2018/using-hugo-and-travis-ci-to-deploy-blog-to-github-pages-automatically/)
- [我是如何用 Hugo、Travis CI 和 GitHub Pages 搭建博客的? - 独行的蚂蚁 - 博客](https://zyfdegh.github.io/post/201705-how-i-setup-hugo/)
- [Blog自动部署实践: Hugo + Travis CI -> GitHub Pages // Yuantops’ Blog](https://blog.yuantops.com/tech/hugo-travis-ci-auto-deploy-to-gh-pages/)
- [Hosting a Hugo blog on GitHub Pages with Travis CI - The Startup - Medium](https://medium.com/swlh/hosting-a-hugo-blog-on-github-pages-with-travis-ci-e74a1d686f10)
- [Host on GitHub | Hugo](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
- [Hosting on GitHub Pages - Hugo中文文档](https://www.gohugo.org/doc/tutorials/github-pages-blog/)
- [Host on Render | Hugo](https://gohugo.io/hosting-and-deployment/hosting-on-render/)
