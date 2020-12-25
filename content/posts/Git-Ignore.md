---
title: "解决添加到GitIgnore的文件没被忽略的问题"
date: 2019-10-30T09:56:18+08:00
draft: false
toc: false
images:
tags:
  - Git
  - GitIgnore
---

> 问题简述  

在开发中有时候会遇到把某文件添加到Git Ignore文件中，但是在Commit的时候发现这个文件还是可以被Commit，并没有被忽略掉，在查看了Git的官方文档，发现官网文档已经做了如下解释：


```
The purpose of gitignore files is to ensure that certain files not tracked by Git remain untracked.
To stop tracking a file that is currently tracked, use：git rm —cached.
```

意思是：gitignore文件的目的是确保未被Git跟踪的某些文件保持未跟踪状态。要停止跟踪当前跟踪的文件，请使用`git rm --cached`。

> 原因  

一般来说出现这种问题的原因是使用了IDE，在创建文件的时候IDE自动给add进去了，被Git追踪管理了。

> 具体执行  

- 如果是单个文件：git rm —cached  path/filename
- 如果是文件夹：git rm -r —cached path

> 参考文档  

- [Git - gitignore Documentation](https://git-scm.com/docs/gitignore)
- [Git - git-rm Documentation](https://git-scm.com/docs/git-rm)