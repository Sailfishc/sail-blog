---
title: 解决Mysql存储Emoji乱码问题
date: 2017-03-31 17:58:27
description: "本文主要解决Emoji表情存储在MySql中乱码的问题"
categories: [Java]
tags: [Java, Emoji]
---

> 最近开发的小伙伴们在开发一个社区模块的时候发现目前数据库存储Emoji表情有问题，会出现乱码的情况，之后是这么解决的：UTF-8转为utf8mb4，但是这种操作数据库的方式很不好，然后就找到了这种方式解决。

在解决之前，得先说明一下为什么会出现乱码，Emoji表情占用4个字节，但是MySQL数据库UTF-8编码最多只能存储3个字节，就会导致存储不进去，在读取的时候读取不完整，导致乱码，那Unicode和UTF-8有什么区别呢？
- [字符编码笔记](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)

总的就一句话，UTF-8是Unicode的一种实现。

## 一、如何解决Emoji存储问题
- [Github地址解决方案](https://github.com/Sailfishc/emoji-demo)

再来摘抄一遍自己的笔记：mysql 的 utf8编码的一个字符最多3个字节，但是一个emoji表情为4个字节，所以utf8不支持存储emoji表情。但是utf8的超集utf8mb4一个字符最多能有4字节，所以能支持emoji表情的存储。但是修改这个配置太繁琐了，容易出错，emoji-java这个库可以在代码段解决这个问题，解决思路：
- 页面有一个表情😁，在经过处理之后可以是😄,将这个字符存入数据库
- 读取的时候可以将😄这个字符转为😁

例如： 😁 我可以存储为:smile:，😭存储为:cry:，等等，可以这样映射起来。

### 1、引入依赖
```
<dependency>
     <groupId>com.vdurmont</groupId>
     <artifactId>emoji-java</artifactId>
    <version>3.2.0</version>
</dependency>
```

### 2、常用API
```
    @RequestMapping("/add/content")
    public ResponseEntity insertContent(@RequestBody Content content) {
        String title = content.getTitle();

        String titles = EmojiParser.parseToAliases(title);
        content.setTitle(titles);
        Integer integer = emojiMapper.insertContent(content);
        if (integer == 1) {
           return ResponseEntity.ok().build();
        }
        return  ResponseEntity.badRequest().build();

    }

    @RequestMapping("/get/{id}")
    public Content getById(@PathVariable("id") Integer id) {

        Content content = emojiMapper.selectById(id);
        String title = EmojiParser.parseToUnicode(content.getTitle());
        content.setTitle(title);
        if (content != null) {
            return content;
        }
        return  null;
    }
```
- EmojiParser.parseToAliases(string); 将表情符号转为字符
- EmojiParser.parseToUnicode(string); 将字符转为表情符号

### 3、案例
- 存储图片

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-7ca27396c4a6b58e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 数据库存储记录

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-61a5a894b926af27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 查询记录

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/734456-a97bc9194fbe4ee8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

