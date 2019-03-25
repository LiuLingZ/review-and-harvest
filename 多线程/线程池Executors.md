​       4项目例子：IDEA：IDEAProject\JavaSE\javaBasics\src\main\java\com\lingz\Thread\threadPool

# 线程池

## 一、JDK对线程池的支持

###1、JDK提供一套Executor框架，其中：

```java
Executors ：它扮演一个线程池工厂的角色。可以通过其获得特点功能的线程池（ThreadPoolExecutor）、
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
	public ScheduledFuture<?> scheduledAtFixedRate(Runnable,long initialDelay,long period,TimeUnit 
	unit);
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
```

### 2、线程执行流程

```
① 提交一个线程任务到线程池，首先比较 corePoolSize 是否还有剩余，有，则分配线程，执行。
② 如果无，则判断 阻塞队列 workQueue是否已经满了，没有，则加入队列中等候；满了，进入第③步。
③ 比较maximum，看是否已经达到最大，如果没有，则开辟一个新的线程，取队列首的任务进行，新的任务加入到队列中。
④ 若workQueue已满，则调用拒绝策略进行处理。
⑤ 由拒绝策略决定新加入的任务的去留问题。
```

### 3、参数详细

####   workQueue

```java
作用：用于存储提交的，但是未被执行的任务的任务队列，是一个BlockingQueue接口类型的对象。
ThreadPoolExecutor可以使用:
	- 直接提交队列
		SynchronousQueue,特殊的BQ，没有容量，一个任务的提交需要被立刻执行，没有空闲就创建线程，一旦达到最		  大值，就执行拒绝策略RejectedExecutorHandler。此时，需要设置很大的maximum，否则很容易拒绝。
		Executors.newCachedThreadPool就是采用这种任务队列。不保留，直接开线程去执行任务。
		
	- 有界任务队列
		ArrayBlockingQueue,其构造参数 public ArrayBlockingQueue(int capacity) 带了一个容量，表示队列
		的最大容量。注意，这里的执行流程就和【2、线程执行流程】一致。为的是尽量将线程数量保持在
		corePoolSize .
        
	- 无界任务队列
		LinkedBlockingQueue,除非资源耗尽，否则不会出现任务队列已满的情况。他和有界有点区别，即，先判断
		核心线程数是否满了，没有则创建线程执行任务，满了则如队列，一直入，不会达到线程池最大线程的情况。
		Executors.newSingleThreadExecutor、newFixedThreadFactory即是无界队列，因为corePoolSize =  		maximum .
		
	- 优先任务队列
		PriorityBlockingQueue,优先任务队列是带优先级的队列，可以控制任务的执行先后顺序，是特殊的，
		无界队列 。
```



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















