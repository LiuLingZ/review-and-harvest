       项目例子：IDEA：IDEAProject\JavaSE\javaBasics\src\main\java\com\lingz\Thread\threadPool

参考：

http://www.cnblogs.com/dolphin0520/p/3932921.html

Java高并发程序设计



# 线程池

## 一、JDK对线程池的支持

###1、JDK提供一套Executor框架，其中：

```java
Executors ：它扮演一个线程池工厂的角色。可以通过其获得特定功能的线程池（ThreadPoolExecutor）、
		   ThreadFactory(线程工厂)。 
ThreadPoolExecutor : 表示一个线程池 ，可以通过new构造一个自己的线程池 。
```

###2、Executors提供各种类型的线程池，主要线程池工厂如下：

```java
public static ExecutorService newFixedThreadPool(int nThread)   //固定线程数的线程池
public static ExecutorService newSingleThreadExecutor() 	//仅一个线程数的线程池
public static ExecutorService newCachedThreadPool()		//自适应大小的线程池
public static ScheduledExecutorService newSingleThreadScheduledExecutor() //单线程的计时任务线程池
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) //自定义大小的计时·· 
```

#### 2.1关于ScheduledExecutorService

```java
内容：返回的ScheduledExecutorService可以根据时间需要对线程进行调度。
含主要方法：
	public ScheduledFuture<?> scheduledAtFixedRate(Runnable,long initialDelay,long period,TimeUnit unit);
			//按固定的延迟速率，第一个任务一开始，就计算延迟时间，到了就开始下一个
	public ScheduledFuture<?> scheduledWithFixedDelay(Runnable,long initialDelay,long 				period,TimeUnit unit) ;	//按固定延迟时段。上一个任务结束后，才开始计时延迟，到了才开始下一个任务。
```



## 二、线程池内部实现

### 1、newThreadPoolExecutor构造器

```java
public ThreadPoolExecutor(
    	int corePoolSize,
    	int maximum,
    	long keepAliveTime,
    	TimeUnit unit,
    	BlockingQueue<Runnable> workQueue, //任务队列
    	ThreadFactory threadFactory, //创建线程的工厂
    	RejectedExecutionHandler handler //拒绝策略
)
    
//记录任务数量
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```



### 2、线程执行流程

```
① 提交一个线程任务到线程池，首先比较 corePoolSize 是否还有剩余，有，则分配线程，执行。
② 如果无，则判断 阻塞队列 workQueue是否已经满了，没有，则加入队列中等候；满了，进入第③步。
③ 比较maximum，看是否已经达到最大，如果没有，则开辟一个新的线程，直接去执行任务。
④ 若workQueue已满，则调用拒绝策略进行处理。
⑤ 由拒绝策略决定新加入的任务的去留问题。
```

### 3、参数详细

####   workQueue

```java
作用：用于存储提交的，但是未被执行的任务的任务队列，是一个BlockingQueue接口类型的对象。
ThreadPoolExecutor可以使用:
	- 直接提交队列
		SynchronousQueue,特殊的BQ，没有容量，一个任务的提交需要被立刻执行，没有空闲就创建线程，一旦达		到最大值，就执行拒绝策略RejectedExecutorHandler。此时，需要设置很大的maximum，否则很容易拒
		绝。Executors.newCachedThreadPool就是采用这种任务队列。不保留，直接开线程去执行任务。
		
	- 有界任务队列
		ArrayBlockingQueue,其构造参数 public ArrayBlockingQueue(int capacity) 带了一个容量，表
		示队列的最大容量。注意，这里的执行流程就和【2、线程执行流程】一致。为的是尽量将线程数量保持在
		corePoolSize .
        
	- 无界任务队列
		LinkedBlockingQueue,除非资源耗尽，否则不会出现任务队列已满的情况。他和有界有点区别，即，先判断
		核心线程数是否满了，没有则创建线程执行任务，满了则如队列，一直入，不会达到线程池最大线程的情况。
		Executors.newSingleThreadExecutor、newFixedThreadFactory即是无界队列，因为	
		corePoolSize = maximum .
		
	- 优先任务队列
		PriorityBlockingQueue,优先任务队列是带优先级的队列，可以控制任务的执行先后顺序，是特殊的，
		无界队列 。
```

​	

#### RejectedExecutionHandler 拒绝策略

```java
内容：如果超出规定范围的任务，当作何处理

JDK内置四种拒绝策略：
	- AbortPolicy: 该策略直接抛异常，阻止程序运行。（显然不妥）
	- CallerRunsPolicy:让提交任务的线程去执行任务，但是这样对提交任务的线程的性能影响很大。
	- DiscardOldestPolicy:丢弃最老的一个等待任务，即丢弃排在队列最前头的任务，然后插入新任务。(不妥)
	- DiscardPolicy:直接丢弃（显然能够接受）

自定义拒绝策略：
	拒绝策略都是实现了 RejectedExecutionHandler 接口，重写方法：
	void rejectedExecution(Runnable r,ThreadPoolExecutor executor)  即可。
		
```

### 4、自定义线程池实现例子：

```java
//自定义的线程池
        ExecutorService executorService = new ThreadPoolExecutor(5, 5, 0, TimeUnit.MILLISECONDS,
                new LinkedBlockingDeque<Runnable>(10), Executors.defaultThreadFactory(),
                new RejectedExecutionHandler() {
                    @Override
                    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                        System.out.println(r.toString() + " is discard");
                    }  
                });
```





### 5、线程池的状态

https://blog.csdn.net/GoGleTech/article/details/79728522

状态图：

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20180328153220367?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvR29HbGVUZWNo/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```java
ThreadPoolExecutor定义了5中状态：
private static final int RUNNING  = -1 << COUNT_BITS;
private static final int SHUTDOWN = 0 << COUNT_BITS;
private static final int STOP = 1 << COUNT_BITS;
private static final int TIDYING = 2 << COUNT_BITS;
private static final int TERMINATED = 3 << COUNT_BITS;

1、RUNNING
	线程池被创建，就处于RUNNING状态，可以接受新的任务，此时线程数量 ctl 为 0；
2、SHUTDOWN
	RUNNING调用shutdown()方法， 此时不会再接收新的任务，但是会让正在执行的任务和任务队列里的任务都执行完。
	全部执行完之后，即ctl为0时，状态转为 TIDYING 。
3、STOP
	RUNNING调用shutdownnow()方法，立刻停止线程池，中断正在执行的，抛弃等待队列的。当ctl为0时，进入TIDYING	  状态。
4、TIDYING
	ctl为0了，此时会调用terminated(）方法进入TERMINATED状态，这是个空方法，可以重写添加额外操作
5、TERMINATED
	线程池彻底终止。
```



### 6、停止线程池

shutdown() ：不会立即终止线程池，而是等所有等待中的任务运行完后停止，期间拒绝新任务

shutdownNow()：立即停止线程池，并且尝试中断正在执行的线程，期间拒绝新任务，清空缓存队列。返回未执行的任务。



### 7、关于submit() 和 execute()

```
execute() ：
	Executor声明的方法，在ThreadPoolExecutor提供了具体实现，是ThreadPoolExecutor的核心方法，作用是提交一个任务给线程池去执行。
	
	
submit() :
	在ExecutorService中声明的方法，在它的实现类 AbstractExecutorService 提供了实现 ， 它的作用和execute()差别不大，都是提交一个任务给线程池执行。不同的是，他能接收返回结果。
	submit()底层还是调用execute()，并且通过Future去执行线程。
```











