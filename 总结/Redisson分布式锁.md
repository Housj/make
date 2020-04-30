# Redis分布式锁（上）- 手写方案

分布式锁的使用场景
单体架构的应用可以直接使用本地锁（Synchronized）就可以解决多线程资源竞争的问题。如果公司业务发展较快，可以通过部署多个服务节点来提高系统的并行处理能力。由于本地锁的作用范围只限于当前应用的线程。高并发场景下，集群中某个应用的本地锁并不会对其它应用的资源访问产生互斥，就会产生数据不一致的问题，所以分布锁就派上了用场

常见的分布式锁应用场景：秒杀活动、优惠券抢购、接口幂等性校验等

分布式锁的三种实现方案
**1、数据库乐观锁**
**2、基于Redis的分布式锁**
**3、基于Zookeeper的分布式锁**

本篇博客主要介绍基于Redis实现的分布式锁

分布式锁需要满足的条件
**1、互斥性**
任何时候锁资源只能被一个线程持有
**2、死锁**
获取锁的客户端由于某些原因未能释放锁，也要保证其他客户端可以获取到锁
**3、可重入**
同一线程已经获得某个锁，可以再次获取锁而不会出现死锁
**4、安全性**
锁只能被持有该锁的客户端删除，不能由其它客户端删除

基于Redis手写的分布式锁
在高并发场景下，应用程序在执行过程中往往会受到网络、CPU、内存等因素的影响，所以实现一个线程安全的分布式组件，往往需要考虑很多case，下面我们通过手写的方式来探索一个完美的分布锁方案的复杂性

**使用setnx()和UUID**

```
/**
 * 获取锁
 */
public static boolean getLock(Jedis jedis, String key, int time) {
	Long flag = jedis.setnx(key, "object");
	if (flag == 1) {
		//如果程序突然宕机，则无法设置过期时间，将发生死锁
		jedis.expire(key, time);
		return true;
	}
	return false;
}

/**
 * 释放锁
 */
public static void releaseLock(Jedis jedis) {
	jedis.del("object");
}
```

上面的代码存在以下2个问题：
1、由于这是两条Redis命令，不具有原子性，如果程序在执行完setnx()之后突然崩溃，导致锁没有设置过期时间，那么锁资源将永远不会被释放
2、jedis.del(“object”)会导致操作超时的线程继续运行时解锁其它正在执行的线程锁资源，可以通过给value赋值为requestId，我们就知道这把锁是哪个请求加的，在解锁的时候就可以区分了

requestId可以使用UUID.randomUUID().toString()方法生成，修改代码如下：

```
/**
 * @param reqId 可用使用UUID生成
 */
public static boolean getLock(Jedis jedis, String reqId, String key, int time) {
	String result = jedis.set(key, reqId, "NX", "EX", time);
	// set操作成功时返回 OK
	if ("OK".equals(result)) {
		return true;
	}
	return false;
}
/**
 * 释放锁
 */
public static void releaseLock(Jedis jedis, String key, String reqId) {
	if(reqId.equals(jedis.get(key))) {
		// 若在此时，这把锁突然不是当前线程的，则会误解锁
		jedis.del(reqId);
	}
}

public static void main(String[] args) {
	Jedis jedis = new Jedis();
	String reqId = UUID.randomUUID().toString();
	try {
		if (getLock(jedis, reqId, "key", 10)) {
			// 数据库操作
		}
	} finally {
		releaseLock(jedis, "key", reqId);
	}
}

```

解锁好像还是有问题！
如果调用jedis.del()方法的时候，这把锁已经不属于当前线程的时候会解除其它线程加的锁。

那么是否真的有这种场景？

答案是肯定的，比如线程A在执行jedis.del()之前，锁突然过期了，此时线程B尝试加锁成功，然后线程A再执行del()方法，则将线程B的锁给删除了！ 原因是删除操作分两条命令去执行的，考虑使用lua脚本原子操作来解决。关于lua脚本的相关知识可以查看其它博文

修改如下：

```
public static boolean releaseLock(Jedis jedis, String reqId, String key) {
	String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
	Object result = jedis.eval(script, Collections.singletonList(key), Collections.singletonList(reqId));
	
	if ("1".equals(result)) {
		return true;
	}
	return false;
}

```

**使用setnx()和过期时间**

执行过程如下：
通过setnx()方法尝试加锁，如果当前锁不存在，返回加锁成功。如果锁已经存在(其它线程获取成功)则获取锁的过期时间和当前时间比较。如果锁已过期，则设置新的过期时间，返回加锁成功。

	public static boolean getLock(Jedis jedis, String key, int time) {
		// 过期时间
		long expires = System.currentTimeMillis() + time;
		String expiresValue = String.valueOf(expires);
	String result = jedis.set(key, expiresValue, "NX", "EX", time);
	// 加锁成功时返回 OK
	if ("OK".equals(result)) {
		return true;
	}
	
	// 锁存在，获取锁的过期时间
	String currentValue = jedis.get(key);
	if (currentValue != null && Long.parseLong(currentValue) < System.currentTimeMillis()) {
		// 锁已过期，获取上一个锁的过期时间，并设置当前锁的过期时间
		String oldValue = jedis.getSet(key, expiresValue);
		if (oldValue != null && oldValue.equals(currentValue)) {
			// 多线程并发的情况，只有一个线程的设置值和当前值相同时，加锁成功
			return true;
		}
	}
	
	return false;

以上方案有如下问题：

1、System.currentTimeMillis()是不同的客户端服务器的系统时间，分布式环境下每个客户端的时间必须同步。
2、锁过期时，如果多个线程同时执行jedis.getSet()方法，虽然只有一个线程可以加锁成功，但这个线程的锁过期时间可能被其他线程所覆盖
3、因为客户端系统时间不同步，根据系统时间删除锁时会误删其它系统的锁

手写分布式锁方案总结
手写方案看似好像很完美，遗憾的是，还是有问题！！！
1、不满足可重入性
2、A线程获锁成功，只是由于一些客观因素（如：数据库I/O偶尔过慢）操作延时，导致key失效，但是业务代码还会是继续往下执行。因为key失效的原因，其它线程可以继续获取到锁，从某种意义上讲还是没有达到从根本上解决资源互斥的效果！

有什么解决方案吗？有，超时时间续期！可以通过定时任务延长失效时间（参考Redisson的Watch Dog），但是实现起来有点麻烦。另外，可重入性问题也可以得到解决，比如：先将获锁成功的线程信息暂存起来，再次请求锁资源时判断一下即可。
（完）



# Redis分布式锁（下）- Redisson源码解析

在之前的文章谈谈基于Redis分布式锁（上）- 手写方案最后我们分析得出手写一个完美的分布式锁方案并不容易（比如可重入性），而且性能也没法得到保证。而Redisson可以解决上述的所有问题，但是还有些小缺陷，文章最后我们再做讨论

本章节我们将探索一下Redisson的底层现实原理，通过源码来分析一下Redisson的实现细节

Redisson简介
Redisson在基于NIO的Netty框架上，充分的利用了Redis键值数据库提供的一系列优势，在Java实用工具包中常用接口的基础上，为使用者提供了一系列具有分布式特性的常用工具类，支持 Redis 单实例、Redis 哨兵、Redis Cluster、Redis master-slave 等各种部署架构

redisson是目前redis分布式锁相对完美的实现，更多详情可以通过Redisson了解

Redisson简单使用

```
RLock lock = redisson.getLock("lockName");
try{
	//可以设置超时时间
	lock.lock();
	//业务逻辑
} finally {
	lock.unlock();
}

```

Redisson加解锁过程分析
详细流程图

通过一张图来看一看Redisson内部的锁操作流程，其内部实现主要用到3大技术栈（Lua脚本+Semaphore+异步线程），注：笔者使用的Redisson版本为3.12.1

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200308000130448.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZseWZoag==,size_16,color_FFFFFF,t_70)


加锁操作

Redisson使用Lua脚本方式将多个非原子命令封装在一起，一起发送给服务端，保证操作的原子性，加锁操作比较复杂，我们以一个例子开始来逐步分析，假设多个客户端同时竞争key为lockName上的锁资源

客户端1

```
RLock lock1 = redisson.getLock(“lockName”);
lock1.lock();
System.out.println("客户端1获锁成功！");
lock1.lock();
System.out.println("客户端1重复获锁成功！");
lock1.unlock();
lock1.unlock();

```

客户端2

```
RLock lock2 = redisson.getLock(“lockName”);
lock2.lock();
```

加锁源代码

```
private void lock(long leaseTime, TimeUnit unit, boolean interruptibly) throws InterruptedException {
        long threadId = Thread.currentThread().getId();
        Long ttl = tryAcquire(leaseTime, unit, threadId);
        // ttl为空，则说明获锁成功
        if (ttl == null) {
            return;
        }
		// 使用redis->subscribe订阅channel，用于监听回调处理
        RFuture<RedissonLockEntry> future = subscribe(threadId);
        ...
        // 获锁失败，则使用while循环不断获取锁的剩余过期时间ttl，然后指定Park的时间为ttl，不断循环判断
        try {
            while (true) {
            	// 如果锁还存在，则返回的是当前锁的剩余过期时间ttl
                ttl = tryAcquire(leaseTime, unit, threadId);
				// 如果其它客户端释放了锁，当前客户端竞争锁成功
                if (ttl == null) {
                    break;
                }
                // 当前客户端线程阻塞，时间为指定的ttl
                if (ttl >= 0) {
                	//使用Semaphore->LockSupport.park()方法
					future.getNow().getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                    }
                } 
            }
        } finally {
            unsubscribe(future, threadId);
        }
    }

```

加锁lua脚本

```
<T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
	internalLockLeaseTime = unit.toMillis(leaseTime);
	return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
   	          //锁不存在，进行加锁操作，将锁资源保存在hash中
	          "if (redis.call('exists', KEYS[1]) == 0) then " +
	              "redis.call('hset', KEYS[1], ARGV[2], 1); " +
	              "redis.call('pexpire', KEYS[1], ARGV[1]); " +
	              "return nil; " +
	          "end; " +
			  //锁存在且为同一线程重复加锁，将该线程的锁重入次数加1
	          "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
	              "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
	              "redis.call('pexpire', KEYS[1], ARGV[1]); " +
	              "return nil; " +
	          "end; " +
	          //其它客户端已经竞争到锁，返回当前锁的剩余过期时间
	          "return redis.call('pttl', KEYS[1]);",
	            Collections.<Object>singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
	}

```


解释一下Lua脚本中的几个参数

KEYS[1]加锁的key的名称，比如RLock lock = redisson.getLock(“lockName”);则KEYS[1]就是lockName

ARGV[1]表示锁的过期时间，如果未设置默认为30秒

ARGV[2]表示加锁的客户端ID，格式为：uuid + “:” + threadid，例如：
11bb52bc-a764-4649-8b46-a61513d7fe44:1

获锁过程

分支一：锁不存在
使用"exists lockName"命令判断锁是否存在，不存在则使用"hset KEYS[1] ARGV[2] 1"命令进行加锁操作，假设客户端1加锁成功，使用hash数据结构存储客户端1的锁资源，结构为：

```
"lockName":{
	//客户端ID:重入次数
	"11bb52bc-a764-4649-8b46-a61513d7fe44:1":1
}
```

接着使用"pexpire KEYS[1] ARGV[1]"，即"pexpire lockName 30000"设置lockName锁的过期时间，然后返回null表示加锁成功!

分支二：锁存在且为同一客户端重复加锁
客户端在同一线程操作中是可以重复获得锁的，使用命令"hincrby KEYS[1] ARGV[2] 1"将同一客户端的可重入次数加1，并重新设置过期时间，返回null表示加锁成功!

客户端1重复加锁成功，此时hash结构如下：

```
"lockName":{
	//客户端ID:重入次数加了1
	"11bb52bc-a764-4649-8b46-a61513d7fe44:1":2
}
```

分支三：客户端锁竞争
在客户端获锁失败后，当前客户端会订阅(subscribe)名称为"redisson_lock__channel: {lockName}"的channel，用于监听回调处理，客户端释放锁时会在redisson_lock__channel:{lockName}的channel上发布(publish)UNLOCK_MESSAGE的解锁消息

如果此时另一个客户端2也尝试在lockName上加锁，exists判断lockName已存在且hash中lockName键已经存在客户端1的锁"11bb52bc-a764-4649-8b46-a61513d7fe44:1"，所以客户端2不能加锁了，怎么办？

客户端线程会使用"pttl KEYS[1]"命令返回当前锁的剩余过期时间ttl，然后使用J.U.C框架中的Semaphore根据返回的ttl时间调用LockSupport.parkNanos(ttl)来阻塞自己，在指定的等待时间结束后，则继续尝试加锁，不断循环，直到成功为止

RedissonLock类中的lock()方法代码片段如下：

	while (true) {
		//尝试加锁
		ttl = tryAcquire(leaseTime, unit, threadId);
		//获取成功
		if (ttl == null) {
			break;
		}
		
		if (ttl >= 0) {
			// 调用Semaphore的tryAcquire()方法
			future.getNow().getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
		}
	}

Semaphore类中的tryACquire代码片段如下：

```
//nanosTimeout时间为客户端需要等待的时间ttl
LockSupport.parkNanos(this, nanosTimeout);
```

场景举例：A正在上厕所，B发现A已经在厕所了，于是B就问A什么时候可以出来，A说等10分钟就好了，这时B就使用手表开始计时等待(Semaphore)，10分钟后发现A还在里面，于是B继续问A还要多久，A不好意思说再等我5分钟吧，B又开始计时等待。。。

**WatchDog延期机制**

为什么要使用WatchDog？
Redisson提供的获锁api中有一个leaseTime选项，该值为-1时表明获锁成功的客户端可以一直持有该锁，释放锁之前，其他客户端线程将一直等待下去。我们知道当在Redis中设置一个key时，往往需要指定expireTime，防止其长期占用内存空间。在种场景下，锁最终还是会过期，所以在key过期之前，必须提供一种机制(WatchDog)来保证key继续有效

Redisson分布式锁中WatchDog实现机制
客户端加锁(lock)成功后，会启用一个watch dog后台线程，使用netty时间轮HashedWheelTimer算法，每隔delay=10秒检查如果客户端还持有锁，则重新设置锁的过期时间为lockWatchdogTimeout=30秒（默认)，其中delay = lockWatchdogTimeout/3

```
private <T> RFuture<Long> tryAcquireAsync(long leaseTime, TimeUnit unit, long threadId) {
	//当leaseTime为-1时启用watchdog
	if (leaseTime != -1) {
	    return tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_LONG);
	}
	RFuture<Long> ttlRemainingFuture = tryLockInnerAsync(commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_LONG);
	ttlRemainingFuture.onComplete((ttlRemaining, e) -> {
	    if (e != null) {
	        return;
	    }
	    // 客户端获锁成功，延期操作
	    if (ttlRemaining == null) {
	        scheduleExpirationRenewal(threadId);
	    }
	});
	return ttlRemainingFuture;
}

```

对于同一客户端重复获锁且成功时，Redisson是怎么保证WatchDog的延期操作只执行一次？答案是：本地缓存

```
private void scheduleExpirationRenewal(long threadId) {
    ExpirationEntry entry = new ExpirationEntry();
    ExpirationEntry oldEntry = EXPIRATION_RENEWAL_MAP.putIfAbsent(getEntryName(), entry);
    //第2次以后再获取锁，不用再使用时间轮算法延期了
    if (oldEntry != null) {
        oldEntry.addThreadId(threadId);
    } else {
    	//第1次获取成功时，进行延期操作
        entry.addThreadId(threadId);
        renewExpiration();
    }
}

```

RedissonLock类中的renewExpiration()方法代码片段如下：

```
//延期操作
private void renewExpiration() {
	Timeout task = commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
		@Override
		public void run(Timeout timeout) throws Exception {
			//Lua脚本延期锁的过期时间
			RFuture<Boolean> future = renewExpirationAsync(threadId);
			future.onComplete((res, e) -> {
				//延期成功
				if (res) {
					// 继续循环延期操作
					renewExpiration();
				}
			});
		}
	//每隔10秒检查一次
	}, internalLockLeaseTime / 3, TimeUnit.MILLISECONDS);
}

```

调用renewExpirationAsync()方法设置锁的过期时间，Lua脚本如下：

```
protected RFuture<Boolean> renewExpirationAsync(long threadId) {
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
            "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                "return 1; " +
            "end; " +
            "return 0;",
        Collections.<Object>singletonList(getName()), 
        internalLockLeaseTime, getLockName(threadId));
}

```

**哪些获锁方法会使用WatchDog?**
大家应该注意，不是所有的获锁成功操作都会开启WatchDog功能，还需要leaseTime为-1的条件成立时，才会启用WatchDog。

下面将罗列出Redisson提供的部分获锁操作：
定义RLock rlock = client.getLock(“lockName”);

获锁方法	是否开启WatchDog
rlock.lock()	启用
rlock.tryLock()	启用
tryLock(long waitTime, TimeUnit unit)	启用
tryLock(long waitTime, long leaseTime != -1, TImeUnit unit)	关闭
————————————————
注：leaseTime为-1会开启watchDao功能



**释放锁操作**

释放锁的操作相对简单，也比较容易理解，大概就四步：
删除key -> 设置过期时间 ->删除本地缓存 -> 发布解锁消息

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020030814390340.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZseWZoag==,size_16,color_FFFFFF,t_70)

解锁操作lua脚本

```
protected RFuture<Boolean> unlockInnerAsync(long threadId) {
	return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
			//不存在就直接返回null
	        "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then " +
	            "return nil;" +
	        "end; " +
	        //将当前客户端的锁重入次数-1
	        "local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); " +
	        //如果当前客户端还持了锁,则重新设置过期时间
	        "if (counter > 0) then " +
	            "redis.call('pexpire', KEYS[1], ARGV[2]); " +
	            "return 0; " +
	         //客户端已经释放了锁，删除key，发布解锁消息
	        "else " +
	            "redis.call('del', KEYS[1]); " +
	            "redis.call('publish', KEYS[2], ARGV[1]); " +
	            "return 1; "+
	        "end; " +
	        "return nil;",
	        Arrays.<Object>asList(getName(), getChannelName()), LockPubSub.UNLOCK_MESSAGE, internalLockLeaseTime, getLockName(threadId));
    }

```

删除本地缓存代码

```
void cancelExpirationRenewal(Long threadId) {
    ExpirationEntry task = EXPIRATION_RENEWAL_MAP.get(getEntryName());
    if (task == null) {
        return;
    }
    if (threadId != null) {
        task.removeThreadId(threadId);
    }
    if (threadId == null || task.hasNoThreads()) {
        Timeout timeout = task.getTimeout();
        if (timeout != null) {
            timeout.cancel();
        }
        EXPIRATION_RENEWAL_MAP.remove(getEntryName());
    }
}

```


其中删除本地缓存map是在异步线程中执行的，WatchDog对客户端的锁进行缓期操作后，将该客户端线程信息保存在本地缓存map中，保证同一客户端重复获锁成功时，锁延期操作只执行一次

总结
至此，Redisson加解锁的详细过程分析完毕！回到开篇，我们说Redisson还有些小缺陷，比如在Mast-Slave架构下，主从同步通常是异步的

在这种场景（主从结构）中存在明显的竞态:
1、客户端A从master获取到锁
2、在master将锁同步到slave之前，master宕掉了
3、slave节点被晋级为master节点
4、客户端B取得了同一个资源被客户端A已经获取到的另外一个，锁安全失效！

官方给出的解决方案是使用Redlock算法，如果读者想进一步了解更多关于Redlock的内容，请参考官网Redis之RedLock算法

原文链接：https://blog.csdn.net/flyfhj/article/details/104715607



# Redisson实现分布式锁(1)---原理

![img](http://imgconvert.csdnimg.cn/aHR0cHM6Ly9pLmxvbGkubmV0LzIwMjAvMDMvMTMvUHR3UWtINXNBUzEybUxXLnBuZw?x-oss-process=image/format,png)

有关Redisson作为实现分布式锁，总的分3大模块来讲。

```
1、Redisson实现分布式锁原理
2、Redisson实现分布式锁的源码解析
3、Redisson实现分布式锁的项目代码（可以用于实际项目中）
```

本文只介绍Redisson如何实现分布式锁的原理。其它的会在接下来的博客讲，最后有关`Redisson实现分布式锁的项目代码`的博客中会放上项目源码到GitHub上。

## 一、高效分布式锁

当我们在设计分布式锁的时候，我们应该考虑分布式锁至少要满足的一些条件，同时考虑如何高效的设计分布式锁，这里我认为以下几点是必须要考虑的。

#### 1、互斥

在分布式高并发的条件下，我们最需要保证，同一时刻只能有一个线程获得锁，这是最基本的一点。

#### 2、防止死锁

在分布式高并发的条件下，比如有个线程获得锁的同时，还没有来得及去释放锁，就因为系统故障或者其它原因使它无法执行释放锁的命令,导致其它线程都无法获得锁，造成死锁。

所以分布式非常有必要设置锁的`有效时间`，确保系统出现故障后，在一定时间内能够主动去释放锁，避免造成死锁的情况。

#### 3、性能

对于访问量大的共享资源，需要考虑减少锁等待的时间，避免导致大量线程阻塞。

所以在锁的设计时，需要考虑两点。

1、`锁的颗粒度要尽量小`。比如你要通过锁来减库存，那这个锁的名称你可以设置成是商品的ID,而不是任取名称。这样这个锁只对当前商品有效,锁的颗粒度小。

2、`锁的范围尽量要小`。比如只要锁2行代码就可以解决问题的，那就不要去锁10行代码了。

#### 4、重入

我们知道ReentrantLock是可重入锁，那它的特点就是：同一个线程可以重复拿到同一个资源的锁。重入锁非常有利于资源的高效利用。关于这点之后会做演示。

针对以上Redisson都能很好的满足，下面就来分析下它。



## 二、Redisson原理分析

为了更好的理解分布式锁的原理，我这边自己画张图通过这张图来分析。

![img](https://img2018.cnblogs.com/blog/1090617/201906/1090617-20190618183025891-1248337684.jpg)

#### 1、加锁机制

线程去获取锁，获取成功: 执行lua脚本，保存数据到redis数据库。

线程去获取锁，获取失败: 一直通过while循环尝试获取锁，获取成功后，执行lua脚本，保存数据到redis数据库。

#### 2、watch dog自动延期机制

这个比较难理解，找了些许资料感觉也并没有解释的很清楚。这里我自己的理解就是:

在一个分布式环境下，假如一个线程获得锁后，突然服务器宕机了，那么这个时候在一定时间后这个锁会自动释放，你也可以设置锁的有效时间(不设置默认30秒），这样的目的主要是防止死锁的发生。

但在实际开发中会有下面一种情况:

```java
      //设置锁1秒过去
        redissonLock.lock("redisson", 1);
        /**
         * 业务逻辑需要咨询2秒
         */
        redissonLock.release("redisson");

      /**
       * 线程1 进来获得锁后，线程一切正常并没有宕机，但它的业务逻辑需要执行2秒，这就会有个问题，在 线程1 执行1秒后，这个锁就自动过期了，
       * 那么这个时候 线程2 进来了。那么就存在 线程1和线程2 同时在这段业务逻辑里执行代码，这当然是不合理的。
       * 而且如果是这种情况，那么在解锁时系统会抛异常，因为解锁和加锁已经不是同一线程了，具体后面代码演示。
       */
```

所以这个时候`看门狗`就出现了，它的作用就是 线程1 业务还没有执行完，时间就过了，线程1 还想持有锁的话，就会启动一个watch dog后台线程，不断的延长锁key的生存时间。

`注意` 正常这个看门狗线程是不启动的，还有就是这个看门狗启动后对整体性能也会有一定影响，所以不建议开启看门狗。

#### 3、为啥要用lua脚本呢？

这个不用多说，主要是如果你的业务逻辑复杂的话，通过封装在lua脚本中发送给redis，而且redis是单线程的，这样就保证这段复杂业务逻辑执行的**原子性**。

#### 4、可重入加锁机制

Redisson可以实现可重入加锁机制的原因，我觉得跟两点有关：

```
1、Redis存储锁的数据类型是 Hash类型
2、Hash数据类型的key值包含了当前线程信息。
```

下面是redis存储的数据
![img](https://img2018.cnblogs.com/blog/1090617/201906/1090617-20190618183037704-975536201.png)

这里表面数据类型是Hash类型,Hash类型相当于我们java的 `>` 类型,这里key是指 'redisson'

它的有效期还有9秒，我们再来看里们的key1值为`078e44a3-5f95-4e24-b6aa-80684655a15a:45`它的组成是:

​		guid + 当前线程的ID。后面的value是就和可重入加锁有关。

**举图说明**

![img](https://img2018.cnblogs.com/blog/1090617/201906/1090617-20190618183046827-1994396879.png)

上面这图的意思就是可重入锁的机制，它最大的优点就是相同线程不需要在等待锁，而是可以直接进行相应操作。

#### 5、Redis分布式锁的缺点

Redis分布式锁会有个缺陷，就是在Redis哨兵模式下:

`客户端1` 对某个`master节点`写入了redisson锁，此时会异步复制给对应的 slave节点。但是这个过程中一旦发生 master节点宕机，主备切换，slave节点从变为了 master节点。

这时`客户端2` 来尝试加锁的时候，在新的master节点上也能加锁，此时就会导致多个客户端对同一个分布式锁完成了加锁。

这时系统在业务语义上一定会出现问题，**导致各种脏数据的产生**。

`缺陷`在哨兵模式或者主从模式下，如果 master实例宕机的时候，可能导致多个客户端同时完成加锁。



# Redisson实现分布式锁(2)—RedissonLock

有关Redisson实现分布式锁上一篇博客讲了分布式的锁原理：[Redisson实现分布式锁---原理](https://www.cnblogs.com/qdhxhz/p/11046905.html)

这篇主要讲RedissonLock和RLock。Redisson分布式锁的实现是基于RLock接口，RedissonLock实现RLock接口。

## 一、RLock接口

#### 1、概念

```java
public interface RLock extends Lock, RExpirable, RLockAsync
```

很明显RLock是继承Lock锁，所以他有Lock锁的所有特性，比如lock、unlock、trylock等特性,同时它还有很多新特性：强制锁释放，带有效期的锁,。

#### 2、RLock锁API

这里针对上面做个整理，这里列举几个常用的接口说明

```java
public interface RRLock {
    //----------------------Lock接口方法-----------------------

    /**
     * 加锁 锁的有效期默认30秒
     */
    void lock();
    /**
     * tryLock()方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回false .
     */
    boolean tryLock();
    /**
     * tryLock(long time, TimeUnit unit)方法和tryLock()方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，
     * 在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。
     *
     * @param time 等待时间
     * @param unit 时间单位 小时、分、秒、毫秒等
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    /**
     * 解锁
     */
    void unlock();
    /**
     * 中断锁 表示该锁可以被中断 假如A和B同时调这个方法，A获取锁，B为获取锁，那么B线程可以通过
     * Thread.currentThread().interrupt(); 方法真正中断该线程
     */
    void lockInterruptibly();

    //----------------------RLock接口方法-----------------------
    /**
     * 加锁 上面是默认30秒这里可以手动设置锁的有效时间
     *
     * @param leaseTime 锁有效时间
     * @param unit      时间单位 小时、分、秒、毫秒等
     */
    void lock(long leaseTime, TimeUnit unit);
    /**
     * 这里比上面多一个参数，多添加一个锁的有效时间
     *
     * @param waitTime  等待时间
     * @param leaseTime 锁有效时间
     * @param unit      时间单位 小时、分、秒、毫秒等
     */
    boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException;
    /**
     * 检验该锁是否被线程使用，如果被使用返回True
     */
    boolean isLocked();
    /**
     * 检查当前线程是否获得此锁（这个和上面的区别就是该方法可以判断是否当前线程获得此锁，而不是此锁是否被线程占有）
     * 这个比上面那个实用
     */
    boolean isHeldByCurrentThread();
    /**
     * 中断锁 和上面中断锁差不多，只是这里如果获得锁成功,添加锁的有效时间
     * @param leaseTime  锁有效时间
     * @param unit       时间单位 小时、分、秒、毫秒等
     */
    void lockInterruptibly(long leaseTime, TimeUnit unit);  
}
```

RLock相关接口，主要是新添加了 `leaseTime` 属性字段，主要是用来设置锁的过期时间,避免死锁。



## 二、RedissonLock实现类

```java
public class RedissonLock extends RedissonExpirable implements RLock
```

RedissonLock实现了RLock接口，所以实现了接口的具体方法。这里我列举几个方法说明下

#### 1、void lock()方法

```java
    @Override
    public void lock() {
        try {
            lockInterruptibly();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
```

发现lock锁里面进去其实用的是`lockInterruptibly`（中断锁，表示可以被中断）,而且捕获异常后用 Thread.currentThread().interrupt()来真正中断当前线程，其实它们是搭配一起使用的。

具体有关lockInterruptibly()方法讲解推荐一个博客。`博客`：[Lock的lockInterruptibly()](https://blog.csdn.net/zengmingen/article/details/53260650)

接下来执行流程,这里理下关键几步

```java
   /**
     * 1、带上默认值调另一个中断锁方法
     */
    @Override
    public void lockInterruptibly() throws InterruptedException {
        lockInterruptibly(-1, null);
    }
    /**
     * 2、另一个中断锁的方法
     */
    void lockInterruptibly(long leaseTime, TimeUnit unit) throws InterruptedException 
    /**
     * 3、这里已经设置了锁的有效时间默认为30秒  （commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout()=30）
     */
    RFuture<Long> ttlRemainingFuture = tryLockInnerAsync(commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_LONG);
    /**
     * 4、最后通过lua脚本访问Redis,保证操作的原子性
     */
    <T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
        internalLockLeaseTime = unit.toMillis(leaseTime);

        return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
                "if (redis.call('exists', KEYS[1]) == 0) then " +
                        "redis.call('hset', KEYS[1], ARGV[2], 1); " +
                        "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                        "return nil; " +
                        "end; " +
                        "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                        "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                        "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                        "return nil; " +
                        "end; " +
                        "return redis.call('pttl', KEYS[1]);",
                Collections.<Object>singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
    }
```

那么void lock(long leaseTime, TimeUnit unit)方法其实和上面很相似了，就是从上面第二步开始的。

#### 2、tryLock(long waitTime, long leaseTime, TimeUnit unit)

接口的参数和含义上面已经说过了，现在我们开看下源码，这里只显示一些重要逻辑。

```java
 @Override
    public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
        long time = unit.toMillis(waitTime);
        long current = System.currentTimeMillis();
        long threadId = Thread.currentThread().getId();
        Long ttl = tryAcquire(leaseTime, unit, threadId);
        //1、 获取锁同时获取成功的情况下，和lock(...)方法是一样的 直接返回True，获取锁False再往下走
        if (ttl == null) {
            return true;
        }
        //2、如果超过了尝试获取锁的等待时间,当然返回false 了。
        time -= System.currentTimeMillis() - current;
        if (time <= 0) {
            acquireFailed(threadId);
            return false;
        }

        // 3、订阅监听redis消息，并且创建RedissonLockEntry，其中RedissonLockEntry中比较关键的是一个 Semaphore属性对象,用来控制本地的锁请求的信号量同步，返回的是netty框架的Future实现。
        final RFuture<RedissonLockEntry> subscribeFuture = subscribe(threadId);
        //  阻塞等待subscribe的future的结果对象，如果subscribe方法调用超过了time，说明已经超过了客户端设置的最大wait time，则直接返回false，取消订阅，不再继续申请锁了。
        //  只有await返回true，才进入循环尝试获取锁
        if (!await(subscribeFuture, time, TimeUnit.MILLISECONDS)) {
            if (!subscribeFuture.cancel(false)) {
                subscribeFuture.addListener(new FutureListener<RedissonLockEntry>() {
                    @Override
                    public void operationComplete(Future<RedissonLockEntry> future) throws Exception {
                        if (subscribeFuture.isSuccess()) {
                            unsubscribe(subscribeFuture, threadId);
                        }
                    }
                });
            }
            acquireFailed(threadId);
            return false;
        }

       //4、如果没有超过尝试获取锁的等待时间，那么通过While一直获取锁。最终只会有两种结果
        //1)、在等待时间内获取锁成功 返回true。2）等待时间结束了还没有获取到锁那么返回false。
        while (true) {
            long currentTime = System.currentTimeMillis();
            ttl = tryAcquire(leaseTime, unit, threadId);
            // 获取锁成功
            if (ttl == null) {
                return true;
            }
           //   获取锁失败
            time -= System.currentTimeMillis() - currentTime;
            if (time <= 0) {
                acquireFailed(threadId);
                return false;
            }
        }
    }
```

`重点` tryLock一般用于特定满足需求的场合，但不建议作为一般需求的分布式锁，一般分布式锁建议用void lock(long leaseTime, TimeUnit unit)。因为从性能上考虑，在高并发情况下后者效率是前者的好几倍

#### 3、unlock()

解锁的逻辑很简单。

```java
@Override
    public void unlock() {
        // 1.通过 Lua 脚本执行 Redis 命令释放锁
        Boolean opStatus = commandExecutor.evalWrite(getName(), LongCodec.INSTANCE,
                RedisCommands.EVAL_BOOLEAN,
                "if (redis.call('exists', KEYS[1]) == 0) then " +
                        "redis.call('publish', KEYS[2], ARGV[1]); " +
                        "return 1; " +
                        "end;" +
                        "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then " +
                        "return nil;" +
                        "end; " +
                        "local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); " +
                        "if (counter > 0) then " +
                        "redis.call('pexpire', KEYS[1], ARGV[2]); " +
                        "return 0; " +
                        "else " +
                        "redis.call('del', KEYS[1]); " +
                        "redis.call('publish', KEYS[2], ARGV[1]); " +
                        "return 1; "+
                        "end; " +
                        "return nil;",
                Arrays.<Object>asList(getName(), getChannelName()),
                LockPubSub.unlockMessage, internalLockLeaseTime,
                getLockName(Thread.currentThread().getId()));
        // 2.非锁的持有者释放锁时抛出异常
        if (opStatus == null) {
            throw new IllegalMonitorStateException(
                    "attempt to unlock lock, not locked by current thread by node id: "
                            + id + " thread-id: " + Thread.currentThread().getId());
        }
        // 3.释放锁后取消刷新锁失效时间的调度任务
        if (opStatus) {
            cancelExpirationRenewal();
        }
    }
```

使用 EVAL 命令执行 Lua 脚本来释放锁：

1. key 不存在，说明锁已释放，直接执行 `publish` 命令发布释放锁消息并返回 `1`。
2. key 存在，但是 field 在 Hash 中不存在，说明自己不是锁持有者，无权释放锁，返回 `nil`。
3. 因为锁可重入，所以释放锁时不能把所有已获取的锁全都释放掉，一次只能释放一把锁，因此执行 `hincrby` 对锁的值**减一**。
4. 释放一把锁后，如果还有剩余的锁，则刷新锁的失效时间并返回 `0`；如果刚才释放的已经是最后一把锁，则执行 `del` 命令删除锁的 key，并发布锁释放消息，返回 `1`。

`注意`这里有个实际开发过程中，容易出现很容易出现上面第二步异常，非锁的持有者释放锁时抛出异常。比如下面这种情况

```java
      //设置锁1秒过去
        redissonLock.lock("redisson", 1);
        /**
         * 业务逻辑需要咨询2秒
         */
        redissonLock.release("redisson");
      /**
       * 线程1 进来获得锁后，线程一切正常并没有宕机，但它的业务逻辑需要执行2秒，这就会有个问题，在 线程1 执行1秒后，这个锁就自动过期了，
       * 那么这个时候 线程2 进来了。在线程1去解锁就会抛上面这个异常（因为解锁和当前锁已经不是同一线程了）
       */
```



# Redisson实现分布式锁(3)—项目落地实现

有关Redisson实现分布式锁前面写了两篇博客作为该项目落地的铺垫。

1、[Redisson实现分布式锁(1)---原理](https://www.cnblogs.com/qdhxhz/p/11046905.html)

2、[Redisson实现分布式锁(2)—RedissonLock](https://www.cnblogs.com/qdhxhz/p/11055426.html)

这篇讲下通过Redisson实现分布式锁的项目实现，我会把项目放到GitHub,该项目可以直接运用于实际开发中，作为分布式锁使用。

## 一、项目概述

#### 1、技术架构

项目总体技术选型

```
SpringBoot2.1.5 + Maven3.5.4 + Redisson3.5.4 + lombok(插件)
```

#### 2、加锁方式

该项目支持 `自定义注解加锁` 和 `常规加锁` 两种模式

**自定义注解加锁**

```java
 @DistributedLock(value="goods", leaseTime=5)
  public String lockDecreaseStock(){
    //业务逻辑
  }
```

**常规加锁**

```java
 //1、加锁
 redissonLock.lock("redisson", 10);
 //2、业务逻辑
 //3、解锁
 redissonLock.unlock("redisson");
```

#### 3、Redis部署方式

该项目支持四种Redis部署方式

```
1、单机模式部署
2、集群模式部署
3、主从模式部署
4、哨兵模式部署
```

该项目已经实现支持上面四种模式，你要采用哪种只需要修改配置文件`application.properties`，项目代码不需要做任何修改。

#### 4、项目整体结构

```makefile
redis-distributed-lock-core # 核心实现
|
---src
      |
      ---com.jincou.redisson
                           |# 通过注解方式 实现分布式锁
                           ---annotation
                           |# 配置类实例化RedissonLock
                           ---config
                           |# 放置常量信息
                           ---constant
                           |# 读取application.properties信息后，封装到实体
                           ---entity    
                           |# 支持单机、集群、主从、哨兵 代码实现
                           ---strategy

redis-distributed-lock-web-test # 针对上面实现类的测试类
|
---src
      |
      ---java
            |
            ---com.jincou.controller
                                 |# 测试 基于注解方式实现分布式锁
                                 ---AnnotatinLockController.java
                                 |# 测试 基于常规方式实现分布式锁
                                 ---LockController.java
      ---resources                
           | # 配置端口号 连接redis信息(如果确定部署类型，那么将连接信息放到core项目中)
            ---application.properties
```



## 二、测试

模拟`1秒内100个线程`请求接口，来测试结果是否正确。同时测试3中不同的锁:lock锁、trylock锁、注解锁。

#### 1、lock锁

```java
   /**
     * 模拟这个是商品库存
     */
    public static volatile Integer TOTAL = 10;

    @GetMapping("lock-decrease-stock")
    public String lockDecreaseStock() throws InterruptedException {
        redissonLock.lock("lock", 10);
        if (TOTAL > 0) {
            TOTAL--;
        }
        Thread.sleep(50);
        log.info("======减完库存后,当前库存===" + TOTAL);
        //如果该线程还持有该锁，那么释放该锁。如果该线程不持有该锁，说明该线程的锁已到过期时间，自动释放锁
        if (redissonLock.isHeldByCurrentThread("lock")) {
           redissonLock.unlock("lock");
        }
        return "=================================";
    }
```

压测结果

![img](https://img2018.cnblogs.com/blog/1090617/201906/1090617-20190620160457530-219069000.png)

没问题，不会超卖！

#### 2、tryLock锁

```java
   /**
     * 模拟这个是商品库存
     */
    public static volatile Integer TOTAL = 10;

    @GetMapping("trylock-decrease-stock")
    public String trylockDecreaseStock() throws InterruptedException {
        if (redissonLock.tryLock("trylock", 10, 5)) {
            if (TOTAL > 0) {
                TOTAL--;
            }
            Thread.sleep(50);
            redissonLock.unlock("trylock");
            log.info("====tryLock===减完库存后,当前库存===" + TOTAL);
        } else {
            log.info("[ExecutorRedisson]获取锁失败");
        }
        return "===================================";
    }
```

测试结果

![img](https://img2018.cnblogs.com/blog/1090617/201906/1090617-20190620160509805-373780908.png)

没有问题 ，不会超卖！

#### 3、注解锁

```java
/**
     * 模拟这个是商品库存
     */
    public static volatile Integer TOTAL = 10;

    @GetMapping("annotatin-lock-decrease-stock")
    @DistributedLock(value="goods", leaseTime=5)
    public String lockDecreaseStock() throws InterruptedException {
        if (TOTAL > 0) {
            TOTAL--;
        }
        log.info("===注解模式=== 减完库存后,当前库存===" + TOTAL);
        return "=================================";
    }
```

测试结果

![img](https://img2018.cnblogs.com/blog/1090617/201906/1090617-20190620160520050-657011722.png)

没有问题 ，不会超卖！

通过实验可以看出，通过这三种模式都可以实现分布式锁，然后呢？哪个最优。



## 三、三种锁的锁选择

`观点` 最完美的就是lock锁，因为

```
1、tryLock锁是可能会跳过减库存的操作，因为当过了等待时间还没有获取锁，就会返回false,这显然很致命！

2、注解锁只能用于方法上，颗粒度太大，满足不了方法内加锁。
```

#### 1、lock PK tryLock 性能的比较

模拟`5秒内1000个线程`分别去压测这两个接口，看报告结果！

**1）lock锁**

压测结果 1000个线程平均响应时间为31324。吞吐量 14.7/sec

![img](https://img2018.cnblogs.com/blog/1090617/201906/1090617-20190620160532510-1165289045.png)

#### 2）tryLock锁

压测结果 1000个线程平均响应时间为28628。吞吐量 16.1/sec

![img](https://img2018.cnblogs.com/blog/1090617/201906/1090617-20190620160542186-124172156.png)

这里只是单次测试，有很大的随机性。从当前环境单次测试来看，tryLock稍微高点。

#### 2、常见异常 attempt to unlock lock, not ······

在使用RedissonLock锁时，很容易报这类异常，比如如下操作

```java
       //设置锁1秒过去
        redissonLock.lock("redisson", 1);
        /**
         * 业务逻辑需要咨询2秒
         */
        redissonLock.release("redisson");
```

上面在并发情况下就会这样

![img](https://img2018.cnblogs.com/blog/1090617/201906/1090617-20190620160552028-1884674135.png)

造成异常原因：

```
线程1 进来获得锁后，但它的业务逻辑需要执行2秒，在 线程1 执行1秒后，这个锁就自动过期了，那么这个时候 
线程2 进来了获得了锁。在线程1去解锁就会抛上面这个异常（因为解锁和当前锁已经不是同一线程了）
```

所以我们需要注意，设置锁的过期时间不能设置太小，一定要合理，宁愿设置大点。

正对上面的异常，可以通过isHeldByCurrentThread()方法，

```java
  //如果为false就说明该线程的锁已经自动释放，无需解锁
  if (redissonLock.isHeldByCurrentThread("lock")) {
            redissonLock.unlock("lock");
        }
```

好了，这篇博客就到这了！

至于完整的项目地址见GitHub。

如果对您能有帮助,就给个星星吧，哈哈！

`GitHub地址` https://github.com/yudiandemingzi/spring-boot-distributed-redisson



# redisson分布式锁源码和原理浅析

在redisson之前，很多人可能已经自己实现过基于redis的分布式锁，本身原理也比较简单，redis自身就是一个单线程处理器，具备互斥的特性，通过setNx，exist等命令就可以完成简单的分布式锁，处理好超时释放锁的逻辑即可。

redisson在此基础上，加上了更多的逻辑控制和功能，譬如公平锁等。这一篇我们就来看看redisson是如何完成分布式锁的。

![img](https://img-blog.csdnimg.cn/20190716142334579.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aWFueWFsZWkuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

先是使用锁的地方，在上一篇里已经用了。先获取RLock对象，调用lock、tryLock方法来完成加锁的功能。之后执行完毕需要被加锁的逻辑后，释放锁。

![img](https://img-blog.csdnimg.cn/20190716142533530.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aWFueWFsZWkuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

lock方法是直接加锁，如果锁已被占用，则直接线程阻塞，进行等待，直到锁被占用方释放。

tryLock方法则是设定了waitTime（等待时间），在这个等待时间没到前，也是线程阻塞并反复去获取锁，直到取到锁或等待时间超时，则返回false。

这里就以tryLock的源码为例来看看。pom里依赖的redisson版本是

![img](https://img-blog.csdnimg.cn/20190716144854741.png)



```java
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
        long time = unit.toMillis(waitTime);
        long current = System.currentTimeMillis();
        long threadId = Thread.currentThread().getId();
        //尝试获取锁，如果没取到锁，则获取锁的剩余超时时间
        Long ttl = tryAcquire(leaseTime, unit, threadId);
        // lock acquired
        if (ttl == null) {
            return true;
        }
        //如果waitTime已经超时了，就返回false
        time -= System.currentTimeMillis() - current;
        if (time <= 0) {
            acquireFailed(threadId);
            return false;
        }
       
    current = System.currentTimeMillis();
    RFuture<RedissonLockEntry> subscribeFuture = subscribe(threadId);
    if (!await(subscribeFuture, time, TimeUnit.MILLISECONDS)) {
        if (!subscribeFuture.cancel(false)) {
            subscribeFuture.onComplete((res, e) -> {
                if (e == null) {
                    unsubscribe(subscribeFuture, threadId);
                }
            });
        }
        acquireFailed(threadId);
        return false;
    }
 
    try {
        time -= System.currentTimeMillis() - current;
        if (time <= 0) {
            acquireFailed(threadId);
            return false;
        }
        //进入死循环，反复去调用tryAcquire尝试获取锁，ttl为null时就是别的线程已经unlock了
        while (true) {
            long currentTime = System.currentTimeMillis();
            ttl = tryAcquire(leaseTime, unit, threadId);
            // lock acquired
            if (ttl == null) {
                return true;
            }
 
            time -= System.currentTimeMillis() - currentTime;
            if (time <= 0) {
                acquireFailed(threadId);
                return false;
            }
 
            // waiting for message
            currentTime = System.currentTimeMillis();
            if (ttl >= 0 && ttl < time) {
                getEntry(threadId).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
            } else {
                getEntry(threadId).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
            }
 
            time -= System.currentTimeMillis() - currentTime;
            if (time <= 0) {
                acquireFailed(threadId);
                return false;
            }
        }
    } finally {
        unsubscribe(subscribeFuture, threadId);
    }
 //        return get(tryLockAsync(waitTime, leaseTime, unit));
    }
```

可以看到，其中主要的逻辑就是尝试加锁，成功了就返回true，失败了就进入死循环反复去尝试加锁。中途还有一些超时的判断。逻辑还是比较简单的。

再看看tryAcquire方法

![img](https://img-blog.csdnimg.cn/20190716145957212.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aWFueWFsZWkuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

这个方法的调用栈也是比较多，之后会进入下面这个方法

![img](https://img-blog.csdnimg.cn/20190716150043356.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aWFueWFsZWkuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

该方法就是与redis通信的地方，通过exists key的方法来判断是否已经上锁，如果没锁，则会返回null，锁了则返回超时时间。

回到那个死循环的地方：

![img](https://img-blog.csdnimg.cn/20190716150834658.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aWFueWFsZWkuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

这里有一个针对waitTime和redis锁住的key的超时时间大小的比较，取到二者中比较小的那个值，然后用java的Semaphore信号量的tryAcquire方法来阻塞线程。

那么Semaphore信号量又是由谁控制呢，何时才能release呢。这里又需要回到上面来看。

![img](https://img-blog.csdnimg.cn/2019071615263560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aWFueWFsZWkuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

注意这个subscribe方法，它会进入到PublishSubscribe.java中

![img](https://img-blog.csdnimg.cn/20190716152725759.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aWFueWFsZWkuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

在这里可以看到，将当前的threadId添加到一个AsyncSemaphore中，并且设置一个redis的监听器，这个监听器是通过redis的发布、订阅功能实现的。如果不了解redis发布订阅的可以去百度查查。一旦监听器收到redis发来的消息，就从中获取与当前thread相关的，如果是锁被释放的消息，就立马通过操作Semaphore来让刚才阻塞的地方释放。

![img](https://img-blog.csdnimg.cn/20190716153917631.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aWFueWFsZWkuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

释放后，线程继续执行，仍旧是判断是否已经超时。如果还没超时，就进入下一次循环。

![img](https://img-blog.csdnimg.cn/20190716154008309.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aWFueWFsZWkuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

再次去获取锁，取到了就返回true，取不到，继续刚才的步骤。

所以，总体流程应该是这样的。

![img](https://img-blog.csdnimg.cn/2019071616025376.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aWFueWFsZWkuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

最后还有个知识点，就是可重入加锁机制：

假如客户端1已经持有这个锁了，再次加锁时会怎样

![img](https://img-blog.csdnimg.cn/20190716160455229.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aWFueWFsZWkuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

这时我们来分析一下上面那段lua脚本。

第一个if判断肯定不成立，“exists myLock”会显示锁key已经存在了。

第二个if判断会成立，因为myLock的hash数据结构中包含的那个ID，就是客户端1的那个ID，也就是“8743c9c0-0795-4907-87fd-6c719a6b4586:1”

此时就会执行可重入加锁的逻辑，他会用：

incrby myLock 

 8743c9c0-0795-4907-87fd-6c71a6b4586:1 1

通过这个命令，对客户端1的加锁次数，累加1。

此时myLock数据结构变为下面这样：

大家看到了吧，那个myLock的hash数据结构中的那个客户端ID，就对应着加锁的次数。相应的，释放锁时，如果执行lock.unlock()，就可以释放分布式锁，此时的业务逻辑也是非常简单的。

其实说白了，就是每次都对myLock数据结构中的那个加锁次数减1。

如果发现加锁次数是0了，说明这个客户端已经不再持有锁了，此时就会用：

“del myLock”命令，从redis里删除这个key。

然后呢，另外的客户端2就可以尝试完成加锁了。

这就是所谓的分布式锁的开源Redisson框架的实现机制。

一般我们在生产系统中，可以用Redisson框架提供的这个类库来基于redis进行分布式锁的加锁与释放锁。



参考文章：https://www.cnblogs.com/AnXinliang/p/10019389.html
