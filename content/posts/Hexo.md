---
title: 使用 Github 空间搭建 Hexo 技术博客--安装篇（基于 IntelliJ IDEA）
date: 2016-02-28 17:58:27
description: "本文将讲解基于 IntelliJ IDEA 如何使用 IDE 搭建 Hexo 博客、写博客！"
categories: [Hexo,IntelliJ IDEA]
tags: [Hexo,IntelliJ IDEA,Git,Github,Node.js]
---


<!-- more -->


## 部署前介绍 
- 这篇博客引自code.youmeek.com

### Hexo 是什么

- Hexo 的中文官网：<http://hexo.io/zh-cn/>
- 作者 Tommy Chen：<https://zespia.tw/>
- 在我的理解里面：Hexo 是一个基于 Node.js 快速、简洁且高效的博客框架，可以将 Markdown 文件快速的生成静态网页，托管在 GitHub Pages 上。
- 而官网自己是这样说的：

> Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。


### 为什么要用 Hexo

- 我：因为其他博客框架太烂了
- Tommy Chen：<https://zespia.tw/blog/2012/10/11/hexo-debut/>


### 适合人群

- 有 IntelliJ IDEA 基础的程序员（或者你使用的是 JetBrains 家的其他产品）
- 只想搭建一个技术博客的人（真心别搞太多，你没那么多精力）


### 文章要求

- 如果是 Git，Node.js 的新人，则这篇文章不要间断操作，要一气呵成，不然可能会遇到各种问题。


### 本文环境

- 系统：
    - Windows 10（64 位）
- 软件：
    - git：**2.7.0.2-64-bit**
    - IntelliJ IDEA：**15.0.4**
    - node.js：**v5.7.0-64-bit Stable**
- 账号：
    - Github：<https://github.com/>
    - DNSPOD：<https://www.dnspod.cn/>


### 搭建所需软件

- 各个软件官网：
    - git：<http://git-scm.com/>
    - IntelliJ IDEA：<https://www.jetbrains.com/idea/>
    - node.js：<http://nodejs.org/>
- 所需软件集合：
    - 上面三者集合包：<http://pan.baidu.com/s/1bvbo8e>


### 文章前提

- 如果你对上面要求的软件一个都不了解的话，建议你先看如下内容（只是让你们先了解下，当时别照着文章内容做）：
    - [如何搭建一个独立博客——简明Github Pages与Hexo教程](http://www.jianshu.com/p/05289a4bc8b2)
    - [通过Hexo在Github上搭建博客教程](http://www.jianshu.com/p/858ecf233db9)
    - [hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)
    - [手把手教你建github技术博客](http://www.jianshu.com/p/701b1095da11)
    - [如何搭建一个独立博客——简明Github Pages与Hexo教程](http://www.jianshu.com/p/05289a4bc8b2)
    - [windows下搭建hexo博客并将其部署到GitCafe终极教程](http://www.jianshu.com/p/e858a90d0a17)
    - [使用Hexo搭建博客（三），博客配置、主题和写作](http://www.jianshu.com/p/db7e64d86067)
    - [Hexo搭建WiKi](http://www.jianshu.com/p/e7413116e9d4)
    - [怎么用hexo上传一个README.md到github?](https://www.zhihu.com/question/28058973)


### 域名绑定

- 如果你一开始就打算要绑定域名，则下面教程中所有可以填写域名的地方你都填写上你要绑定的域名，如果你没独立域名，那就使用 Github 默认给你的：XXXXXX.github.io 域名即可。


------


## 部署开始

### git 安装

- 双击运行 **Git-2.7.0.2-64-bit.exe**，接下来一律下一步，不需要多余的选择，假设你安装的目录位置跟我一样：C:\Program Files\Git
- 打开 Git Bash（路径：C:\Program Files\Git\git-bash.exe），输入：`git --version`
    - 如下图，如果出现：**git version 2.7.0.windows.2**，这表示安装成功
    - ![验证 git 安装](http://img.youmeek.com/2016/hexo-start-a-1.jpg)


### Node.js 安装

- 双击运行 **node-v5.7.0-x64.msi**，接下来一律下一步，不需要多余的选择。
- 安装完之后，打开 Git Bash，输入：`npm -v`
    - 如下图，如果出现：**3.6.0**，则表示 Node.js 安装成功。如果你没有显示这个信息，我怀疑你需要重启电脑试试看，因为我在给我弟讲解这一步的时候发现有这个问题。
    - ![验证 node.js 安装](http://img.youmeek.com/2016/hexo-start-a-2.jpg)


### Node.js 源设置

- 在安装 Hexo 之前，先说一下 Node.js 的源，Node.js 官方源默认是：<http://registry.npmjs.org>，但是由于在国外，说不定你使用的时候就抽风无法下载任何软件。所以我们决定暂时使用淘宝提供的源，淘宝源官网：<http://npm.taobao.org/>
- 在 Git Bash 中我们执行下面这一句（这是一整句的）：

``` bash
alias cnpm="npm --registry=https://registry.npm.taobao.org \
--cache=$HOME/.npm/.cache/cnpm \
--disturl=https://npm.taobao.org/dist \
--userconfig=$HOME/.cnpmrc"
```

- 现在验证下是否可以使用淘宝的 cnpm 命令：`cnpm info express`
    - 如果能输出一大堆介绍，则说明成功了，以后每次要使用淘宝的源都需要这样来。现在除了默认的 **npm**，还多了一个 **cnpm** 可以使用，我们下面有关安装的讲解内容也都是基于此临时命令。
    - 如果输出：bash: cnpm: command not found，则表示没成功，需要你在排查下
    - 需要强调的是：**cnpm** 不是永久性命令，只是此时这个界面窗口下的临时命令，关掉窗口就没效果了。


### 安装 Hexo 框架

- 安装 Hexo（注意，现在是 cnpm 开头了，不是 npm 了）：`cnpm install -g hexo-cli`
    - 安装时间不一定很快，有可能需要等 3 ~ 5 分钟。
    - 安装过程中有 WARN 警告也没关系的，不用在意这些 WARN，继续等它安装完成。因为国内的网络问题，有时候安装异常慢花了大半个小时都没效果，那就 Ctrl + C 停掉这次命令，重新再执行一次。


### 创建 Hexo 项目

- 现在假设我要创建一个名为 hexo 的项目，项目目录就放在：E:\git_work_space 目录下，所以我们在 E:\git_work_space 目录下创建一个 hexo 目录。现在这个项目的全路径是：E:\git_work_space\hexo
- 打开 Git Bash：
    - 进入该目录：`cd e:/git_work_space/hexo`
    - 然后执行：`hexo init`，这个时间也会比较长，也有可能要等几分钟，有显示 WARN 也不用管
    - 最后执行：`cnpm install`，有显示 WARN 也不用管
    - 安装完成之后，E:\git_work_space\hexo 目录结构是这样的：
        - ![安装 hexo 后的目录结构](http://img.youmeek.com/2016/hexo-start-b-1.jpg)
    - 现在我们启动 hexo 本地服务，看下默认的博客是怎样的，命令：`hexo server`
    - 现在用浏览器访问：<http://localhost:4000/>，效果如下图
        - ![默认模板效果](http://img.youmeek.com/2016/hexo-start-b-2.jpg)
    - 如果要停止 hexo 服务：在 Git Bash 下按 `Ctrl + C` 即可


### 选用其他主题

- 由于默认主题太大众了，所以现在我们换个主题。
- 你可以去这里找主题：
    - hexo-theme：<https://hexo.io/themes/>
    - hexo-github-theme-list：<https://github.com/hexojs/hexo/wiki/Themes>
    - 有那些好看的hexo主题？：<http://www.zhihu.com/question/24422335>
- 我这里选择的 **yelee**：<https://github.com/MOxFIVE/hexo-theme-yelee>
    - 原因是能最大化衬托出：目录、文章内容、代码块。因为我对这个博客的定位就是用来放技术类内容，所以不会让它太杂或是太花。只是目前这个颜色偏浅，后续还需要调整。
- 现在假设你跟我要用的模板是一样：
    - 还是让 Git Bash 保持在 E:\git_work_space\hexo 目录下，然后输入命令：`git clone https://github.com/MOxFIVE/hexo-theme-yelee.git themes/yelee`
    - 这样就在 E:\git_work_space\hexo\themes 目录下生成了一个 yelle 文件夹，里面有我们刚刚 clone 下来的主题内容。
    - 如果以后你不自己修改这个主题的话，可以考虑经常更新下作者的更新内容：
        - `cd e:/git_work_space/hexo/themes/yelee`
        - `git pull origin master`
- 下载好主题文件之后，我们现在要修改 E:\git_work_space\hexo 目录下的项目配置文件：**\_config.yml**，把对应的主题目录名改下，编辑如下图。
    - ![修改主题目录](http://img.youmeek.com/2016/hexo-start-b-3.jpg)
- 更改主题目录名后，我们还要重新生成主题静态内容：
    - 继续在 Git Bash 中输入命令：
        - 重新生成静态博客的所有内容：`hexo generate`
        - 重启 hexo 本地服务：`hexo server`
        - 重新访问：<http://localhost:4000/>，效果如下图
        - ![新主题效果](http://img.youmeek.com/2016/hexo-start-b-4.jpg)


### 创建 Github pages 并 SSH 授权

- 现在假设你已经有一个 Gtihub 账号，你还需要一个特别的仓库，特别在仓库名就是你的 Github 账号登录名，比如我的用户名是：judasn，那我要创建的仓库名字完整滴填写是：judasn.github.io，具体效果如下图。
    - ![创建 github pages](http://img.youmeek.com/2016/hexo-start-c-1.jpg)
- 创建好仓库之后，要本地生成 SSH 秘钥，方便电脑上的 git 软件好提交内容到 Github 上
    - 在 Git Bash 中，输入：`ssh-keygen -t rsa -C "你的邮箱地址"`，然后回车，回车，再回车，一共 3 次回车，具体含义自己 Google。
    - 比如我的：`ssh-keygen -t rsa -C "jn3.141592654@gmail.com"`，生成后效果如下图：
    - ![生成 ssh 密钥](http://img.youmeek.com/2016/hexo-start-c-2.jpg)
    - 此时，生成密钥后，在你电脑目录：C:\Users\你的计算机用户名\\.ssh 下，会生成两个文件：
        - 私钥：**id_rsa**
        - 公钥：**id_rsa.pub**
        - 如果生成的不是这样的文件，那删除掉这两个生成的，重新执行上面的命令，让它再一次生成。
    - 现在用记事本打开 id_rsa.pub，复制文件中的所有内容。
        - 访问：<https://github.com/settings/ssh>，添加新秘钥，效果如下图
            - Title：自己随便取
            - Key：把刚刚复制的都粘贴进来
            - ![添加密钥](http://img.youmeek.com/2016/hexo-start-c-3.jpg)


### 把本地的博客内容同步到 Github 上

- 要把本地的静态博客同步到 Github，我们还需要先安装两个跟部署相关的 hexo 插件：
    - 继续在 Git Bash 中输入：
    - `cnpm install hexo -server --save`
    - `cnpm install hexo-deployer-git --save`
- 编辑全局 hexo 的配置文件：**\_config.yml**
    - 官网对此配置的介绍：<https://hexo.io/zh-cn/docs/configuration.html>
    - 我自己的编辑内容初稿（你需要认真看的是含有中文注释的内容）：
    
``` bash
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site，这一块区域主要是设置博客的主要说明，需要注意的是：每个冒号后面都是有一个空格，然后再书写自己的内容的
title: YouMeek Code
subtitle: 这里只有代码相关，要了解更多 >>> YouMeek.com
description: 这里是 YouMeek.com 一部分
author: Judas.n
email: 363379444@qq.com
language: zh-CN
timezone:

# URL，这一块一般可以设置的是 url 这个参数，比如我要设置绑定域名的，这里就需要填写我的域名信息
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://code.youmeek.com
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: yelee

# Deployment
## 这里是重点，这里是修改发布地址，因为我们前面已经加了 SSH 密钥信息在 Github 设置里面了，所以只要我们电脑里面持有那两个密钥文件就可以无需密码地跟 Github 做同步。
## 需要注意的是这里的 repo 采用的是 ssh 的地址，而不是 https 的。分支我们默认采用 master 分支，以后你翅膀硬了要换其他也无所谓。
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:judasn/judasn.github.io.git
  branch: master
  
```

- 编辑全局配置后我们需要重新部署：
    - 继续在 Git Bash 中输入命令：
    - 先清除掉已经生成的旧文件：`hexo clean`
    - 再生成一次静态文件：`hexo generate`
    - 在本地预览下：`hexo server`
    - 本地没问题之后，Ctrl + C 停掉本地预览。
    - 在部署到 Github 之前，需要先确定你是否已经用过 Git，如果你没用过，则此时你需要做如下设置，在 Git Bash 中依次输入下面两个命令：
        - `git config --global user.email "你的 Github 注册邮箱地址"`
        - `git config --global user.name "你的 Github 用户名"`
    - 使用部署命令部署到 Github 上：`hexo deploy`，有弹出下面提示框，请输入：`yes`
        - ![确认提交](http://img.youmeek.com/2016/hexo-start-d-1.jpg)
    - 提交成功效果如下：
        - ![提交成功](http://img.youmeek.com/2016/hexo-start-d-2.jpg)
    - 访问服务器地址进行检查：<http://judasn.github.io>，效果如下
        - ![服务器效果](http://img.youmeek.com/2016/hexo-start-d-3.jpg)
    - 但是，也不排除你 deploy 不了到 Github，报这个错误：`Host key verification failed`，那解决办法如下：
        - 还是在 Git Bash 界面中，输入如下命令：`ssh -T git@github.com`，根据界面提示，输入：`yes` 回车。之后你可以再试一下是否可以 deploy。
- 通过上面几次流程我们也就可以总结：以后，每次发表新文章要部署都按这样的流程来：
    - `hexo clean`
    - `hexo generate`
    - `hexo deploy`
- 也因为这几个命令太频繁了，所以又有了精简版的命令：
    - `hexo clean == hexo clean`
    - `hexo g == hexo generate`
    - `hexo s == hexo server`
    - `hexo d == hexo deploy`


### 绑定域名

- 绑定域名不排除会遇到很多网络问题或是七七八八，所以这里先提供先官网的一些说明：
    - <https://help.github.com/articles/setting-up-your-pages-site-repository/>
    - <https://help.github.com/articles/quick-start-setting-up-a-custom-domain/>
    - <https://help.github.com/articles/setting-up-an-apex-domain/>
    - <https://help.github.com/articles/troubleshooting-custom-domains/>
    - <https://help.github.com/articles/custom-domain-redirects-for-github-pages-sites/>
- 首先我们要一个 CNAME 文件（文件名叫 CNAME，没有文件后缀的），把该文件放在 E:\git_work_space\hexo\source 目录下，以后一些需要放在根目录的资源文件都可以放这里。如果你找不到这样的文件可以到我的项目上下载：<https://github.com/judasn/judasn.github.io>
    - CNAME 上的内容需要写你具体要绑定的域名信息，比如我是：**code.youmeek.com**，具体你可以参考下图：
        - ![设置 CNAME 文件](http://img.youmeek.com/2016/hexo-start-e-1.jpg)
- 接着我们需要到 DNSPOD 上设置域名解析：<https://www.dnspod.cn/>
    - ![设置域名解析](http://img.youmeek.com/2016/hexo-start-e-2.jpg)
    - ![设置域名解析](http://img.youmeek.com/2016/hexo-start-e-3.jpg)
- 设置好之后，等过几分钟域名解析好之后，我们访问：<http://code.youmeek.com>，效果如下：    
    - ![域名访问效果](http://img.youmeek.com/2016/hexo-start-e-4.jpg)
	- 2016-08-19 更新：Github 提示，建议我们使用 CNAME 方式来指向，别用 IP，所以建议你这样配置，还是以我的为例：
		- 主机记录：code
		- 记录类型：CNAME
		- 记录值：judasn.github.io.（后面的这个点别忘记了）
		- 还有，要记得把 CNAME 那个文件再 deploy 到 Github 哦，不然还是访问不了的。

### 整合 IntelliJ IDEA 提高效率

- 为了提交写作效率，我个人建议使用 IntelliJ IDEA 作为 Markdown 编辑工具
    - IntelliJ IDEA 有各种各样的快捷键支持你的操作
    - IntelliJ IDEA 可以快速地全文检索项目所有的文件
    - 对 JavaScript、CSS、HTML 等常见语言的良好支持，方便你修改你的主题
- 如果你还不会使用 IntelliJ IDEA 或是 JetBrains 家其他产品，你可以看下我写一整套教程：<http://wiki.jikexueyuan.com/project/intellij-idea-tutorial/>
- 现在我们用 IntelliJ IDEA 打开我们本地目录：E:\git_work_space\hexo，打开后效果如下图：
    - ![IntelliJ IDEA 打开项目](http://img.youmeek.com/2016/hexo-start-f-1.jpg)
- 由于 IntelliJ IDEA 在 Windows 下的默认终端是 `cmd` 不好用，我们现在需要重新修改下 IntelliJ IDEA 的终端工具，把它指向我们习惯的 Git Bash，这样方便操作，如下图 gif 演示。
    - ![IntelliJ IDEA 下操作 hexo](http://img.youmeek.com/2016/hexo-start-f-2.gif)
    - 如 gif 演示，我们可以 IntelliJ IDEA 里面安心写文章、做发布等操作。
- 为了更稳定地使用 IntelliJ IDEA，在不修改主题的情况下，我们还需要这样做：
    - ![IntelliJ IDEA 下操作 hexo](http://img.youmeek.com/2016/hexo-start-f-3.jpg)
- hexo 新文章内容的开头需要这样定义：
    - categories：表示文章所属分类
    - tags：表示文章所属标签

``` bash
---
title: 这是文章标题
date: 2016-02-28 17:58:27
categories: [Hexo,IntelliJ IDEA]
tags: [Hexo,IntelliJ IDEA,Git,Github,Node.js]
---
```


## 结束语

- 我希望从这一篇你也可以再一次了解到 IntelliJ IDEA 的美丽之处。
- 最后，欢迎再次来到 IntelliJ IDEA 的世界！
