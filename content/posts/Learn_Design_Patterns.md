---
title: 设计模式总结
date: 2017-05-24 11:37:00
description: "设计模式总结"
categories: [Java]
tags: [Java, Design Patterns]
---

![image.png](http://upload-images.jianshu.io/upload_images/734456-32b3fb0c1a41aa71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 一、创建型模式(五种)
- 工厂方法模式(Factory)：工厂创建对象（经典实现：很多框架初始化时都会创建一个工厂对象，用来加载资源）
- 抽象工厂模式(Abstractfactory)：抽象工厂实例创建对象，工厂可修改，灵活度高（经典实现：Struts2插件机制的核心实现就是BeanFactory这个抽象工厂。Spring IOC加载Bean，AOP创建Proxy）
- 单例模式(Sington)：适用于只需要一个对象的情况（经典实现：Tomcat中StringManager的错误处理机制）
- 建造者模式(Builder)：一步一步创建一个复杂的对象（经典实现：MyBatis中的SQLSession就是结合了Configure，executor等对象，以此来实现SQLSession的复杂功能）
- 原型模式(Prototype)：复制对象，包括深度复制和浅度复制，深度复制重建引用对象，浅度复制不创建（经典实现：java序列化）

## 二、结构型模式(七种)
- 适配器模式(Adapter)：通过实现接口，依赖注入，继承等方式为不相关的实体建立关系（经典实现：Tomcat新版本连接器Coyote，就是通过为Connector适配建立了ProtocolHandler与Tomcat组件Connector的关联关系）
- 装饰器模式(Decorator)：创建包装对象修饰扩展被包装对象的功能（经典实现：IO家族中BufferedXxx）
- 代理模式(Proxy)：通过添加中间代理的方式限制，过滤，修改被代理类的某些行为（经典实现：Spring AOP核心实现，DataSource中为Connection创建代理对象，改变close方法的行为，使其从开始的关闭连接变成将连接还回连接池）
- 外观模式(Facade)：通过外观的包装，使应用程序只能看到外观对象，而不会看到具体的细节对象。（经典实现：Tomcat中创建外观类包装StandardContext传给Wrapper，创建外观类包装Wrapper以ServletConfiguration的形式传给Servlet，以此来屏蔽不想让Servlet可见的那些Tomcat容器参数）
- 桥接模式(Bridge)：将抽象部分与它的实现部分分离，使它们都可以独立地变化（经典实现：JDBC驱动）
- 组合模式(Composite)：部分与整体，常用于表示树形结构
- 享元模式(Flyweight)：维护资源集合（经典实现：数据库连接池，避免重新开启数据库链接的开销）
 
## 三、行为型模式(十一种)
 
- 策略模式(Strategy)：定义多个不同的实现类，这些类实现公共接口，通过调用接口调用不同实例得到不同结果（经典实现：Spring中Bean的定义与注入，Controller，Servcie，repository三层架构中只依赖上一层接口）
- 模板方法模式(Template)：父类定义公共方法，不同子类重写父类抽象方法，得到不同结果（经典实现：Tomcat生命周期中的init，SpringIOC上层类加载具体子类指定的配置文件）
- 观察者模式(Observer)：目标方法被调用，通知所有观察者（经典实现：Tomcat生命周期事件监听，Spring BeanPostProcessor实现 ）
- 迭代子模式(Interator)：提供一种方法顺序访问一个聚合对象中各个元素, 而又不需暴露该对象的内部表示。（经典实现：集合迭代器）
- 责任链模式(ChainOfResponsibility)：链式依赖，依次调用（经典实现：Tomcat Valve）
- 命令模式(Commond)：Action定义具体命令，拦截器Invocation回调执行命令（经典实现：Struts2）
- 备忘录模式(Memento)：建立原始对象副本，用于存储恢复原始对象数据
- 状态模式(Stage)：通过改变状态，改变行为（经典实现：切换装载着不同配置信息的配置文件对象）
- 访问者模式(Visitor)：结构与操作解耦。灵活的操作，放入固定的结构中执行（经典实现：在SpringAOP的实现过程中首先会有一个ProxyCreator去创建切入点，通知之类的，然后创建一个抽象工厂将这些参数对象传递给抽象工厂，抽象工厂调用createAopProxy(this)来创建对象，传入不同的抽象工厂创建出不同的实体对象）
- 中介者模式(Mediator)：MVC 框架，其中C（控制器）就是 M（模型）和 V（视图）的中介者
- 解释器模式(Iterpreter)：定义分别定义 + - * / 非终结符，组合不同的非终结符定义不同的表达式，维护繁琐

## 四、学习资源
- [调停者模式](http://www.cnblogs.com/java-my-life/archive/2012/06/20/2554024.html)
- [解释器模式](http://www.cnblogs.com/java-my-life/archive/2012/06/19/2552617.html)
- [访问者模式](http://www.cnblogs.com/java-my-life/archive/2012/06/14/2545381.html)
- [状态模式](http://www.cnblogs.com/java-my-life/archive/2012/06/08/2538146.html)
- [备忘录模式](http://www.cnblogs.com/java-my-life/archive/2012/06/06/2534942.html)
- [命令模式](http://www.cnblogs.com/java-my-life/archive/2012/06/01/2526972.html)
- [责任链模式](http://www.cnblogs.com/java-my-life/archive/2012/05/28/2516865.html)
- [迭代子模式](http://www.cnblogs.com/java-my-life/archive/2012/05/22/2511506.html)
- [观察者模式](http://www.cnblogs.com/java-my-life/archive/2012/05/16/2502279.html)
- [模板方法模式](http://www.cnblogs.com/java-my-life/archive/2012/05/14/2495235.html)
- [策略模式](http://www.cnblogs.com/java-my-life/archive/2012/05/10/2491891.html)
- [不变模式](http://www.cnblogs.com/java-my-life/archive/2012/05/08/2487757.html)
- [桥梁模式](http://www.cnblogs.com/java-my-life/archive/2012/05/07/2480938.html)
- [门面模式](http://www.cnblogs.com/java-my-life/archive/2012/05/02/2478101.html)
- [享元模式](http://www.cnblogs.com/java-my-life/archive/2012/04/26/2468499.html)
- [代理模式](http://www.cnblogs.com/java-my-life/archive/2012/04/23/2466712.html)
- [装饰模式](http://www.cnblogs.com/java-my-life/archive/2012/04/20/2455726.html)
- [合成模式](http://www.cnblogs.com/java-my-life/archive/2012/04/17/2453861.html)
- [适配器模式](http://www.cnblogs.com/java-my-life/archive/2012/04/13/2442795.html)
- [原型模式](http://www.cnblogs.com/java-my-life/archive/2012/04/11/2439387.html)
- [建造模式](http://www.cnblogs.com/java-my-life/archive/2012/04/07/2433939.html)
- [单例模式](http://www.cnblogs.com/java-my-life/archive/2012/03/31/2425631.html)
- [抽象工厂模式](http://www.cnblogs.com/java-my-life/archive/2012/03/28/2418836.html)
- [工厂方法模式](http://www.cnblogs.com/java-my-life/archive/2012/03/25/2416227.html)
- [简单工厂模式](http://www.cnblogs.com/java-my-life/archive/2012/03/22/2412308.html)

## 五、总结
> - 内容部分引用自http://smallbug-vip.iteye.com/blog/2276470
- 在学习设计模式的过程中最直观的感觉就是小demo感觉看着很爽，自己也能理解，但是在实际开发中想套用哪种设计模式却感觉有点心有余而力不足的意思，这可能就是设计模式太灵活的缘故，设计模式都是在大量的代码中积累的精髓所在，这篇总结也是给自己一个笔记，让自己在之后的开发中可以回过头来看看。
- 很多人对设计模式有点误解，认为只有GoF的设计模式才是设计模式，这种观点有点不正确，以下是引用一段解释：
    - 设计模式的不同侧重
      - 1、 侧重总体架构的设计模式，如MVC(Java web开发)，MVVM(WPF) 
      - 2、 侧重细节架构的设计模式，如GoF所描述的23种设计模式
      - 3、 侧重实现的模式，如生产消费队列的同步问题等等
- 设计模式是对oo原则的实现
- 再引用一段话：
    - 简单的功能或演示性的程序不要考虑设计模式
     - 关注问题而非解决方案。也就是说只有遇到问题时才去设计模式里找方法
    - 关注重用而非设计模式。也就是说设计模式是以重用为目的的，只要能做到重用，是否使用了某种设计模式并不重要
    - 在支持函数式编程的语言里避免使用设计模式。如前所述，设计模式是以重用的目的的，而高阶函数很多情况下是比设计模式更好的重用方法，这些语言包括常见的js, ruby, python等
能用简单的模式解决的问题不要引入复杂的模式
     - 已经有解决方案的情况下不要硬套设计模式，设计模式不是一切
    - 正确认识些暂时无法理解的模式，如Memento, Interpreter, Visitor等，这些模式代表着一类问题，很多时候使用第三方库是更好的选择
    - 看轻设计模式，它们只能解决一小部分问题