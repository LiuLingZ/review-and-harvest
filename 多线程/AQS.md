https://mp.weixin.qq.com/s/PAn5oTlvVmjMepmCRdBnkQ

https://www.cnblogs.com/waterystone/p/4920797.html

# AQS

​	java并发包下很多API都是基于AQS来实现的加锁和释放锁等功能的，AQS是java并发包的基础类。

ReentrantLock、ReentrantReadWriteLock都是基于AQS的。

![img](https://images2015.cnblogs.com/blog/721070/201705/721070-20170504110246211-10684485.png)

##  简介

​	AQS，AbstractQueuedSynchronizer 抽象队列同步器，是JUC包下众多同步 类的核心实现，如ReentrantLock\Simaphone \CountdonwLatch正式底层含有一个AQS对象实现同步机制

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



# 面试回答

```java


ReentrantLock 基于 AQS ， JUC下多数并发类基于AQS 。

ReentrantLock 的内部类 Sync继承于抽象类AQS，实现了一些核心方法。从AQS说去

AQS的核心是： volatile 修饰的 int state , 代表锁的状态，0无锁，> 0 代表锁被占用
			FIFO的队列，队列的结点 封装的是竞争资源的线程。
			
AQS有两种模式：独占模式和共享模式，分别对应tryAquire和tryAquireShared

ReentrantLock 就是独占的，内部实现了公平独占和非公平独占。而诸如CountDownLatch就是共享的。

以ReentrantLock为例说一下lock和unlock过程

lock():
	- 首先会通过CAS尝试获取一次锁，获取成功(这是非公平的实现)，则该干嘛干嘛.
  	- 获取失败,就通过 addWaiter方式加入到队列的尾部，这里也是通过自旋+CAS实现的。
  	- 加入到尾部后，就开始判断这个线程是尝试获取资源还是暂时park等待了。
  	
  	这里涉及到一个节点的状态
  	> 0 ，则代表线程不参与竞争，可能就是中断了，取消了
    <= 0 ，代表线程有权利参与竞争
    SIGNAL -1 ,如果前一个节点是，当前节点就会等待前一个节点的唤醒。刚刚说到加入队尾可能就会park等待。
    
    - 继续，如果加入队列的节点的前一个节点是头结点，那么他不会等待，而是不断循环尝试获取锁，tryAquire。
    - 如果获取成功，则该该干嘛干嘛，失败不断自旋，同时判断是否需要park和中断过
    - 如果前一个节点不是头结点，并且状态不是<=0的， 就会一直往前找，直到找到状态 <=0的，并且把前一个节
      状态设置为SIGNAL， 然后park等待unpark .
    - 如果中断过，就调用 Thread.interrput()中断自己，这个线程也就不参与同步资源的获取了。
    
    
过程：
CAS获取锁 → 失败入队尾 → 无限循环中，往前找到合适的前一个节点(状态<=0),设置他为SIGNAL，然后park。或者中断响应 → 如果是老二节点，就不park了(中断还是要的，即即使拿了同步资源也要中断)，而是不断循环获取资源。 → 头结点释放资源就出队列了，拿到资源的节点变为 头结点。
    
    
    
unlock():
	底层调用tryRelease()，功能就是唤醒下一个等待的线程，可能是老二节点，如果老二因为中断什么的为空，就从尾结点开始往前找，知道找到合适的结点，unpark唤醒他。
     
  	
  	
  	

```











































