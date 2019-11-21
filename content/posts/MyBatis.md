---
title: 那些书本中没告诉你的MyBatis
date: 2019-01-07 21:37:00
description: "MyBatis"
categories: [MyBatis]
tags: [MyBatis]
---

 不管是《MyBatis从入门到精通》，还是21天精通MyBatis，都告诉了你怎么做？但是没告诉你为什么以及是什么。  
> 本篇将会讲解MyBatis（Spring-MyBatis）的一些关键类，以及关于DB操作的内容  


## 关键类
- SqlSessionFactory
- SqlSessionFactoryBean
- SqlSession
- MapperFactoryBean
- SqlSessionDaoSupport
- SqlSessionTemplate
- MapperScannerConfigurer

## 如何访问数据库
`mysql -uadmin -pxxx -h58.87.87.129 -P32007`

当我们需要连接MySQL时，需要用到几个属性：

- 用户名
- 密码
- 地址
- 端口

这在MySQL中叫一次会话（session），在JDBC中JDBC定义了规范，其实现类具体去操作，在MyBatis中，MyBatis的关键类`SqlSession`定义了如何去操作数据库，例如：

- selectOne
- selectAll
- selectList
- insert
- …..

具体的实现类有：

- DefaultSqlSession
- SqlSessionTemplate
- SqlSessionManager

当然我们都不会直接使用这几个实现类，我们一般是这样的：

```
public class UserService {
	
	@Resource
	private UserMapper userMapper;
}
```


那这其中究竟是怎么做到这样简化的呢？

## 实现细节

### SqlSessionFactoryBean&SqlSessionFactory

上面介绍了关键类`SqlSession`, 那如何创建`SqlSession`呢？MyBatis使用了工厂方法来创建`SqlSession`，由于工厂和SqlSession是一对多的关系，也就是说一个Factory可以创建多个SqlSession，MyBatis-Spring将SqlSessionFactory放入了容器的生命周期中，也就出现了SqlSessionFactoryBean，我们一般这样配置：

```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
</bean>
```

- DataSource（必须配置）： SqlSessionFactory 需要一个（也可以是多个） DataSource

### MapperFactoryBean

大家都知道MyBatis是基于Mapper的，一般情况MyBatis的一个Mapper就对应一张DB的Table，就有了

- UserMapper
- PersonMapper
- AddressMapper
- …..

那MapperFactoryBean是干什么用的呢？

- 告诉容器（可以是Spring容器）：我的这个Mapper可以访问这个数据，所以MapperFactoryBean需要配置`Mapper`和`SqlSessionFactory`

```
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
  <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```


> 到目前为止，我们没有定义mapper.xml文件，但是我们可以这样用了  

```
public interface UserMapper {
  @Select("SELECT * FROM users WHERE id = #{userId}")
  User getUser(@Param("userId") String userId);
} 
```

AND 

```
public class FooServiceImpl implements FooService {

private UserMapper userMapper;

public void setUserMapper(UserMapper userMapper) {
  this.userMapper = userMapper;
}

public User doSomeBusinessStuff(String userId) {
  return this.userMapper.getUser(userId);
}
}
```

但是只用注解的话面对繁杂的SQL，会显得代码凌乱，就出现了使用xml来描述SQL（当然其他的ORM甚至可以使Markdown），MyBatis要知道SQL文件在哪里有两种方式：

- 和mapper接口在同一个包路径下
- mapperLocations：将目录地址设置到mapperLocations中

```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
  <property name="mapperLocations" value="classpath*:sample/config/mappers/**/*.xml" />
</bean>
```


### MapperScannerConfigurer

在上面的方法中，我们需要把Mapper当做Service的Bean中的属性设置进去，这样显得太麻烦了，就有了`MapperScannerConfigurer`

```
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <property name="basePackage" value="org.mybatis.spring.sample.mapper" />
</bean>
```

- basePackage 属性是让你为映射器接口文件设置基本的包路径
- MapperScannerConfigurer , 它 将 会 查 找 类 路 径 下 的 映 射 器 并 自 动 将 它 们 创 建 成 MapperFactoryBean