https://mp.weixin.qq.com/s/PAn5oTlvVmjMepmCRdBnkQ

https://www.cnblogs.com/waterystone/p/4920797.html

# AQS

​	java并发包下很多API都是基于AQS来实现的加锁和释放锁等功能的，AQS是java并发包的基础类。

ReentrantLock、ReentrantReadWriteLock都是基于AQS的。

![img](https://images2015.cnblogs.com/blog/721070/201705/721070-20170504110246211-10684485.png)

##  简介

​	AQS，AbstractQueuedSynchronizer 抽象队列同步器，是JUC包下众多同步类的核心实现，如ReentrantLock\Simaphone \CountdonwLatch正式底层含有一个AQS对象实现同步机制

## 核心

### 1 、 

​	AQS有维护了一个 volatile int state ，标志共享资源，也即锁状态。 以及一个FIFO的先进先出线程等待队列。多线程竞争资源阻塞会被放到这个队列中。

​	state 的修改 基于CAS操作。

​	AQS内部还有一个CLH双向队列，这是个虚拟的双向队列，将每个请求封装为其节点，节点间维护这相互的关系，用来竞争锁。

​	

### 2、AQS定义资源的两种方式：

​	Exclusive（独占，仅一个线程执行，如ReentrantLock）

​	Share（共享，多个线程可同时执行，如Semaphore \ CountDownLatch）

### 3、 自定义同步器

​	AQS采用的是模板方法模式，意味着，只需修改其模板方法，其他方法不必修改（其实AQS其他方法都是final也修改不了），就可以提供新的功能，而AQS的模板方法就是下面那几个

​	自定义同步器只需要去自定义 共享变量state 的获取、释放行为即可，其他的AQS已经实现。

​	自定义同步器 实现事主要有一下几种方法：	

```java
	isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。
	boolean tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
	boolean tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
	int tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩						    余可用资源；正数表示成功，且有剩余资源。
	int tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返						   回true，否则返回false。

	上面这几种方式就是 独占的、共享的锁的实现方式。
```



## 总结

​	**AQS就是一个并发包的基础组件，用来实现各种锁，各种同步组件的。**它包含了state变量、加锁线程、虚拟等待队列等并发中的核心组件。

​	通过AQS实现的同步功能，只需要自定义对state的修改规则，其他的入队、出队什么的细节，AQS都已经封装好，不需要再实现。

​	同步类在实现一般就是自定义一个同步器（sync）继承AQS作为内部类，供自己使用。

​	AQS底层也是主要靠CAS实现的。