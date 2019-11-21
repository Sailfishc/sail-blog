---
title: "这可能是最全的Redis分布式锁介绍"
date: 2019-11-22T07:01:41+08:00
draft: false
toc: true
tat:
    - Redis
    - Distributed Lock
---

## 目录
* 分布式锁介绍
* 分布式锁的应用场景
* 分布式锁几种实现方式介绍
	* Redis
	* ZK
	* 数据库
* Redis实现分布式锁的实现原理
* 易宝分布式锁分析
* *实现Redis分布式锁的正确操作
* 实现Redis分布式锁的错误案例
* Redis分布式锁的问题
* Redisson的原理及Redission的一些讨论
* 基于Redisson的分布式锁的使用

## 分布式锁介绍
### 定义

> A **distributed lock manager** (DLM) runs in every machine in a cluster, with an identical copy of a cluster-wide lock database. In this way a DLM provides  [software applications](https://en.wikipedia.org/wiki/Software_application)  which are  [distributed](https://en.wikipedia.org/wiki/Distributed_programming)  across a cluster on multiple machines with a means to synchronize their accesses to  [shared resources](https://en.wikipedia.org/wiki/Shared_resource) .  

维基百科上的定义是：在多台机器上同步对共享资源访问的方式。

共享资源包含：

- 数据库
- Table
- 文件
- 共享内存

### 特性

- 互斥性。在任意时刻，只有一个客户端能持有锁。
- 不会发生死锁。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
- 具有容错性。只要大部分的Redis节点正常运行，客户端就可以加锁和解锁。
- 解铃还须系铃人。加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。


## 分布式锁的应用场景
Redis在现在的项目中使用的很多，英国剑桥大学的分布式系统的研究员Martin Kleppmann总结了分布式锁的两个场景：

> * **Efficiency:** Taking a lock saves you from unnecessarily doing the same work twice (e.g. some expensive computation). If the lock fails and two nodes end up doing the same piece of work, the result is a minor increase in cost (you end up paying 5 cents more to AWS than you otherwise would have) or a minor inconvenience (e.g. a user ends up getting the same email notification twice).  
> *  **Correctness:** Taking a lock prevents concurrent processes from stepping on each others’ toes and messing up the state of your system. If the lock fails and two nodes concurrently work on the same piece of data, the result is a corrupted file, data loss, permanent inconsistency, the wrong dose of a drug administered to a patient, or some other serious problem.  

[How to do distributed locking — Martin Kleppmann’s blog](http://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html) 这篇文章中比较清楚的描述了分布式锁的两个场景：

- 效率：使用锁节省去做不必要的相同的事情，例如发送了相同的两封用户邮件，两台机器做相同事的事情。
- 正确性：使用分布式锁防止并发进程弄乱系统的状态，如果分布式锁失效，两个并发的节点操作了相同的数据，导致文件损坏，数据永久丢失，会造成严重的问题。

### 实际应用场景

> 场景一  

商户入网接口，一个身份证号只允许注册一个商户，为了避免多个相同的身份证号同时请求，需要使用分布式锁来保证同一时间只能有一个请求。

> 场景二  

一笔订单APP端用户在修改收货地址，管理后台端商家同时在修改订单备注，这二个请求同时到达的话，如果不借助db的事务，很容易造成行锁竞争，但用事务的话，db的性能显然比不上redis轻量

> 场景三：交易支付回调  

支付成功后支付公司回调消息A，交易系统接收到消息状态为A，进行处理，在处理过程中，支付公司由于系统1问题又发送了一条消息状态为B（相同订单号），B状态消息可能导致订单数据错误

> 场景四  

手机三要素验证的列子：(A渠道系统，业务B系统，外部厂商C系统)
(1)B业务系统调用A渠道系统，验证传入的手机、身份证、号码三要素是否一一致。
(2)A渠道系统再调用外部厂商C系统。
(3)A渠道系统将结果返回给B业务系统。
这3个过程中，(2)过程，外部厂商的调用时是需要计费的。当B业务系统并发量很高时，有100笔相同的三要素校验，由于是相同的三要素，A渠道只要调用一次厂商即可知道结果。那么A渠道系统如何控制不让100笔请求全部去访问外部厂商C系统呢？可以使用分布式锁


##  分布式锁几种实现方式介绍
> 分布式锁的操作  

- boolean lock()
- boolean tryLock()
- boolean tryLock(int timeout)
- void unlock

> 为了减少篇幅，下面几种实现方式只考虑lock和unlock的思路。  

### MySQL实现分布式锁

> 表结构设计  

- id
- key
- value
- retryNum
- createTime
- updateTime

> 加锁操作  

```
	// 使用排它锁
	Lock  = select * from tbl_lock where key = ‘’ for update;
	if ( Lock != null ){
		if ( value == Lock.value) { 
			// 相同的client
			retryNum++
		}else {
			return false;
		}
	}else {
		// 拿到锁
		insert lock..
		return true;
	}
	
	……

	lock.unlock()
```

> 分析  

- 实现分析
	- 在Innodb下使用`for update`加行锁（排它锁），其他client（事务）不能对其进行操作
- 优点
	- 理解简单，无需额外中间件
- 缺点
	- 性能比较低
	- 实现方式较为麻烦（不是复杂）

### ZK实现分布式锁

Zookeeper不太熟悉，但是从Zookeeper的客户端**Curator**官网中查看到实现分布锁的一段代码：

```
InterProcessMutex lock = new InterProcessMutex(client, lockPath);
if ( lock.acquire(maxWait, waitUnit) ) 
{
    try 
    {
        // do some work inside of the critical section here
    }
    finally
    {
        lock.release();
    }
}
```

ZK主要利用得失自己的临时有序节点+watch机制：

- ClientA和ClientB同时获取同一个锁
- 会生成两个有序节点
	- /00001
	- /00002
- 但是这时候只有clientA获取到锁（第一个节点）
- 等ClientA处理完成后进行release后唤醒clientB
- client获取到锁

![Zk获取分布式锁](https://raw.githubusercontent.com/Sailfishc/picRepo/master/iTz1U2.jpg)


> 分析  

- 优点
	- Zookeeper不需要配置锁超时，由于我们设置节点是临时节点，我们的每个机器维护着一个ZK的session，通过这个session，ZK可以判断机器是否宕机。如果我们的机器挂掉的话，那么这个临时节点对应的就会被删除，所以我们不需要关心锁超时。
	- ZK高可用程度比较高
- 缺点
	- ZK需要额外维护，增加维护成本，性能和Mysql相差不大，依然比较差。并且需要开发人员了解ZK是什么。

### Redis实现分布式锁

> 实现  

Redis官网是这么描述单机分布式的实现的：

> To acquire the lock, the way to go is the following: `    SET resource_name my_random_value NX PX 30000`The command will set the key only if it does not already exist (NX option), with an expire of 30000 milliseconds (PX option). The key is set to a value “my*random*value”. This value must be unique across all clients and all lock requests.  

获取锁的方式是：set 资源名称，随机值 NX PX 30000，这条命令只有在key不存在时才会设置，有效期是30000毫秒，这个key设置的value是一个随机值，这个值必须唯一贯穿在客户端加锁的请求中。

> 释放锁  

remove the key only if it exists and the value stored at the key is exactly the one I expect to be. This is accomplished by the following Lua script:

```
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call(“del”,KEYS[1])
else
    return 0
end
```

释放锁的条件是这个key存在并且value是我期望的，这是通过以下Lua脚本完成的。

Redis分布式锁使我们要重点分析的，这里先分简要分析下优缺点：
- 优点
	- 实现简单，性能高
- 缺点
	- 要配置超时时间
	- 单机问题，不能做到高可用

## 易宝分布式锁分析
### 源码分析

- 地址：com.yeepay.utils.lock.impl.RedisLock

> 简要分析Redis分布式锁步骤  

回头来分析下Redis官网给出的案例，加锁是的几个必要条件：

- value要唯一
- 需要在一个指令中使用setNX PX指令

Value唯一是要解决如果安全性，例如clientA删除了clientB的锁的这种情况，setNX PX要放在一条原子指令中是因为如果先设置了NX，在设置PX指令的时候客户端宕机，会发生死锁（锁释放不了的情况）

释放锁使用了Lua脚本，原因也是Redis目前不支持比对value的值，然后去删除锁的操作，所以使用了Lua脚本来达到一致性的效果，这类似MySQL中使用存储过程一样。

### 易宝分布式锁的一些问题

> expire seconds 单位不正确，不是second，而是millis  

```
09:45:53.985 [main] DEBUG com.yeepay.utils.lock.utils.RedisClientUtils - get lock success ,key=key, expire seconds=10000
```

> lock方法不太全，将lock方法实现为了tryLock的功能  

### 错误案例

> 错误案例一  

```
public static void wrongGetLock1(Jedis jedis, String lockKey, String requestId, int expireTime) {

    Long result = jedis.setnx(lockKey, requestId);
    if (result == 1) {
        // 若在这里程序突然崩溃，则无法设置过期时间，将发生死锁
        jedis.expire(lockKey, expireTime);
    }

}
```

> 错误案例二  

```
public static boolean wrongGetLock2(Jedis jedis, String lockKey, int expireTime) {

    long expires = System.currentTimeMillis() + expireTime;
    String expiresStr = String.valueOf(expires);

    // 如果当前锁不存在，返回加锁成功
    if (jedis.setnx(lockKey, expiresStr) == 1) {
        return true;
    }

    // 如果锁存在，获取锁的过期时间
    String currentValueStr = jedis.get(lockKey);
    if (currentValueStr != null && Long.parseLong(currentValueStr) < System.currentTimeMillis()) {
        // 锁已过期，获取上一个锁的过期时间，并设置现在锁的过期时间
        String oldValueStr = jedis.getSet(lockKey, expiresStr);
        if (oldValueStr != null && oldValueStr.equals(currentValueStr)) {
            // 考虑多线程并发的情况，只有一个线程的设置值和当前值相同，它才有权利加锁
            return true;
        }
    }
        
    // 其他情况，一律返回加锁失败
    return false;

}
```

### Redis分布式锁的问题分析

既然是分布式系统，那么有几个边界是需要考虑的：

- 网络是不可靠的，是有延迟的或者中断的
- 多台机器的时钟是有可能不一致的（向前或向后偏移）
- 系统会宕机

基于这些条件我们再分析下单节点Redis或者主从节点Redis实现分布式锁的一些问题：

> 场景一  

- clientA获取到锁，过期时间设置为3s
- clientA业务处理时间过长（或者GC以及其他网络问题），处理时间超过3s，Redis由于key过期释放了key
- clientB拿到了锁，clientA和clientB在同一时间访问了共享资源

> 场景二  

- clientA获取到锁
- Redis主节点挂了，从节点升级为Master，但是由于Redis主从同步是异步的，有可能Master没有将数据同步到Slave
- clientB获取到了锁，clientA和clientB在同一时间访问了共享资源


### Redisson的原理及如何实现分布式锁

Redis作者对于分布式锁从三个方面来描述如何有效使用分布式锁所需要的三个最低保证：

> 1. Safety property: Mutual exclusion. At any given moment, only one client can hold a lock.  
> 2. Liveness property A: Deadlock free. Eventually it is always possible to acquire a lock, even if the client that locked a resource crashes or gets partitioned.  
> 3. Liveness property B: Fault tolerance. As long as the majority of Redis nodes are up, clients are able to acquire and release locks.  

1. 安全属性：互斥，在任何给定的时间，有且仅有一个客户端持有锁
2. 存活属性：无死锁
3. 存活属性：只要大多数节点运行，客户端就可以获取和释放锁

> Redission的算法描述，RedLock的算法稍微有点复杂：  

- 获取当前Unix时间，以毫秒为单位。
- 依次尝试从5个实例，使用相同的key和**具有唯一性的value**（例如UUID）获取锁。当向Redis请求获取锁时，客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为10秒，则超时时间应该在5-50毫秒之间。这样可以避免服务器端Redis已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务器端没有在规定时间内响应，客户端应该尽快尝试去另外一个Redis实例请求获取锁。
- 客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。**当且仅当从大多数**（N/2+1，这里是3个节点）**的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功**。
- 如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）。
- 如果因为某些原因，获取锁失败（没有在至少N/2+1个Redis实例取到锁或者取锁时间已经超过了有效时间），客户端应该在**所有的Redis实例上进行解锁**（即便某些Redis实例根本就没有加锁成功，防止某些节点获取到锁但是客户端没有得到响应而导致接下来的一段时间不能被重新获取锁）。


> Redisson的使用：  

```
RLock lock1 = redissonInstance1.getLock("lock1");
RLock lock2 = redissonInstance2.getLock("lock2");
RLock lock3 = redissonInstance3.getLock("lock3");

RedissonRedLock lock = anyRedissonInstance.getRedLock(lock1, lock2, lock3);
// locks: lock1 lock2 lock3
lock.lock();
...
lock.unlock();
```

- 构造至少5个不同的RLock
- 对多个RLock对象进行分组并将它们作为一个锁处理


### 一些后续的话


即便是RedLock这么牛逼算法，还是不完善的，使用DDIA作者提出的问题分析下：

![Redis分布式锁的问题](https://raw.githubusercontent.com/Sailfishc/picRepo/master/XhLmcL.png)

还是之前的一个例子：

- ClientA获取锁，发生了GC，超过超时时间，Redis释放锁
- ClientB获取到锁，此时ClientA唤醒，两个客户端都获取到锁
- ClientB执行了Read-Write-Update模型之后ClientA再次覆盖了ClientB的数据，造成了数据错误

也有几个实例：

> 案例一  

- clientA获取到A，B，C三个节点，由于网络故障，无法访问D，E节点
- 由于C节点时钟向前偏移，导致锁过期
- clientB获取到C，D，E，由于网络故障，无法访问A，B节点
- 在此时，clientA和clientB都获取到了锁

并且他提出了几个观点，他认为RedLock是建立在这几个假设上的：

> * bounded network delay (you can guarantee that packets always arrive within some guaranteed maximum delay),  
> * bounded process pauses (in other words, hard real-time constraints, which you typically only find in car airbag systems and suchlike), and  
> * bounded clock error (cross your fingers that you don’t get your time from a  [bad NTP server](http://xenia.media.mit.edu/~nelson/research/ntp-survey99/) ).  

1. 有界的网络延迟
2. 有界的程序停顿
3. 有界的时钟异常

> 针对GC导致锁失效的问题  

Martin给出了一个解法，对于ZK这种他会生成一个自增的序列，那么我们真正进行对资源操作的时候，需要判断当前序列是否是最新，有点类似于我们乐观锁。当然这个解法Redis作者进行了反驳，你既然都能生成一个自增的序列了那么你完全不需要加锁了，也就是可以按照类似于Mysql乐观锁的解法去做。


> 针对时钟跳跃的问题  

Martin觉得RedLock不安全很大的原因也是因为时钟的跳跃，因为锁过期强依赖于时间，但是ZK不需要依赖时间，依赖每个节点的Session。Redis作者也给出了解答:对于时间跳跃分为人为调整和NTP自动调整。
* 人为调整:人为调整影响的那么完全可以人为不调整，这个是处于可控的。
* NTP自动调整:这个可以通过一定的优化，把跳跃时间控制的可控范围内，虽然会跳跃，但是是完全可以接受的。

最后Martin认为Redis说的也有道理，但是Redis的分布式锁实现方式还是有问题的，这可能也是TradeOff的魅力吧，Martin是学术流派，从他的书和博客中也可以看出他的参考资料偏Paper，Redis作者是实战派，虽然Redis理论上还是有些问题，但是在目前还是使用比较广泛，这可能就是虽然不完美，但是还是要接受她吧。

## 参考资料

- [Distributed locks with Redis – Redis](https://redis.io/topics/distlock)
- [GitHub - TaXueWWL/redis-distributed-lock: redis分布式锁工具包，提供纯Java方式调用，支持传统Spring工程，  为spring boot应用提供了starter，更方便快捷的调用。](https://github.com/TaXueWWL/redis-distributed-lock)
- [GitHub - AnthonyZero/distributed-lock-redis: Distributed lock based on redis](https://github.com/AnthonyZero/distributed-lock-redis)
- [How to do distributed locking — Martin Kleppmann’s blog](http://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)
- [Is Redlock safe? - <antirez>](http://antirez.com/news/101)
- [使用 Redis 实现分布式锁 - 技术翻译 - OSCHINA 社区](https://www.oschina.net/translate/redis-distlock?print)
- [死磕 java同步系列之redis分布式锁进化史 - 掘金](https://juejin.im/post/5d9699ace51d45781d5e4baa)
- [Redis分布式锁的正确实现方式（Java版） - 吴大山的博客 | Wudashan Blog](https://wudashan.cn/2017/10/23/Redis-Distributed-Lock-Implement/)
- [Redis cluster tutorial – Redis](https://redis.io/topics/cluster-tutorial#redis-cluster-consistency-guarantees)
- 分布式锁使用场景：https://blog.csdn.net/lemon89/article/details/52796775
- [ZooKeeper 笔记(6) 分布式锁 - 菩提树下的杨过 - 博客园](https://www.cnblogs.com/yjmyzz/p/distributed-lock-using-zookeeper.html)
- [基于redis的分布式锁二种应用场景 - 菩提树下的杨过 - 博客园](https://www.cnblogs.com/yjmyzz/p/distribution-lock-using-redis.html)
- [详解分布式协调服务 ZooKeeper](https://draveness.me/zookeeper-chubby)
- [再有人问你分布式锁，这篇文章扔给他 - 掘金](https://juejin.im/post/5bbb0d8df265da0abd3533a5)
- [redis 不可重入分布式锁(setNx()和getset()方法实现) - 断剑重铸之时 - 博客园](https://www.cnblogs.com/yzf666/p/10117890.html)
- [8. Distributed locks and synchronizers · redisson/redisson Wiki · GitHub](https://github.com/redisson/redisson/wiki/8.-distributed-locks-and-synchronizers)
- [Apache Curator – Distributed Lock](http://curator.apache.org/getting-started.html)
- [curator分布式锁](https://blog.csdn.net/xuefeng0707/article/details/80588855)
- [Redlock：Redis分布式锁最牛逼的实现 - 简书](https://www.jianshu.com/p/7e47a4503b87)
- [RedLock算法-使用redis实现分布式锁服务 - 简书](https://www.jianshu.com/p/fba7dd6dcef5)