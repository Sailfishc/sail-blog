---
title: 如何使用Unit testing健壮Java代码
date: 2019-12-03T23:57:32+08:00
draft: false
toc: true
images:
tags:
  - TDD
---

> 前言：[测试的道理](http://www.yinwang.org/blog-cn/2016/09/14/tests)，推荐下王垠的博客，这篇博客不会提高你的测试水平，但是有一些理论和方法上的指导。  

在之前对于测试一直处于写Test Case的状态，使用JUnit结合Spring Test跑下接口，使用断言或者格式化的JSON结果比对下结果，这样也平安无事的写了一段时间的代码，随着对代码的一些要求，去学习了一些书籍和资料，有了一些总结，开始使用Unit testing来健壮我们的Java代码吧。


## 目录
* 聊聊我为什么要写这篇Unit Test的文章
* Unit testing的目的和要解决的问题
* Unit testing的基础
* Unit testing的设计原则
* Unit testing实践中的问题
* 使用Groovy来编写测试

## 我为什么要写这篇Unit testing的文章
> 写好代码应该是一个有追求的程序员对自己的基本要求  

在初入Java时沉浸在学习各种框架的谜团里，在工作中开始关注自己的代码质量，期间看了《重构》、《重构与模式》，之后接触到了敏捷开发，去了解TDD，就开始实践TDD了，当然，这篇文章不是写TDD（Test Driven Development）的。我也一直在找一本适合我的Test书籍，一直没合适的，期间找到了几本：
* 《程序开发人员测试指南》
* 《Junit实战》

之后白衣推荐了《有效的单元测试》，立即入了这本书，发现这本书很符合我现在的情况，目前书还没看完，先大致的浏览了一遍，想结合耗哥（左耳朵耗子）在专栏中关于高效学习的模板写这篇文章，这也是写本文的初衷。

## Unit testing的目的和要解决的问题
维基百科定义：[Unit testing - Wikipedia](https://en.wikipedia.org/wiki/Unit_testing)

> In computer programming, unit testing is a software testing method by which individual units of source code, sets of one or more computer program modules together with associated control data, usage procedures, and operating procedures, are tested to determine whether they are fit for use.[1]  

我自己的看法是：
* 验证Source Code的正确性及可用性
* 提高开发效率
* 写出更好的代码

## Unit testing的基础
为了更好地描述出一些概念，我使用Junit作为Unit testing的工具结合一些案例来说明。

### 断言（assert）

> 有一个函数`add`,实现一个两个数相加  

```
public int add(int a, int b) {
    return a + b;
}
```

> 使用Junit进行测试  

```
@Test
public void add() {
    int sum = service.add(1, 2);
    assertEquals(3, sum);
}
```

这里的assertEquals就是断言，断定`add`方法的输出结果是我们期望(expect)的3.


> 断言的好处：  

咱们先说下如果没有断言，那么我们会将sum的结果打印到控制台，然后看下是不是3，如果是3那说明是符合我们的要求的，如果不是，那么不符合要求，这里涉及到了人手工的去比对结果，这里的结果只有一个，还好比对，如果有很多，那么大大的增加了比对中出现的问题。
断言的好处：减少人工处理，使用计算机帮我们验证结果


> 断言类型：  

* assertEquals：断言两个对象（或基本对象）是否相等
* assertArrayEquals：断言两个数组包含相同的元素
* assertTrue：断言语句为真
* assertFalse：断言语句为假
* assertNull：断言对象引用为空
* assertNotNull：断言对象引用不为空
* assertSame：断言两个对象引用相同的实例
* assertNotSame：断言两个对象引用不同的实例
* assertThat：断言对象满足条件

### Mock


## 参考资源

- [TDD编码实战讲义 | 斑斓](http://zhangyi.xyz/handout-tdd-code-kata/)
- [深度解读测试驱动开发（TDD）](https://gitbook.cn/books/58c2ac77306e988f12697f16/index.html)
- [让我们再聊聊TDD – ThoughtWorks洞见](https://insights.thoughtworks.cn/talk-about-tdd-again/)
- TDD并不是看上去的那么美：https://coolshell.cn/articles/3649.html
- [JUnit -     About](https://junit.org/junit4/)
- [Mocks Aren’t Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
- [How to write good tests · mockito/mockito Wiki · GitHub](https://github.com/mockito/mockito/wiki/How-to-write-good-tests)
- [Mockito - mockito-core 3.1.0 javadoc](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)