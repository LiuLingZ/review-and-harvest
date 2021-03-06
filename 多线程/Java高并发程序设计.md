#### 临界区

#### 线程活跃度：

​	死锁	饥饿	活锁

#### 控制并发的策略，分类并发的级别 : 

​	阻塞 ： 有同步区

​	无饥饿 ：不论优先级，先来后到按顺序执行线程

​	无障碍 ：乐观锁态度管理多线程

​	无锁 ：乐观锁形式，至少有一个线程能执行完

​	无等待 ：乐观锁形式，所有线程在优先时间都能执行完

（主机张所等）

####JMM

​	原子性

​	可见性

​	有序性

​	内存规范，解决多线程通过主内存进行通信中产生的数据不一致、处理器乱序执行、指令重排序的问题，实现并发安全的可见性、有序性、原子性。

​	原子性中，synchronized 通过 monitorenter 和 monitorexit 保证。



#### 64位的long在并发读写是不安全的

#### happen-before原则/先行发生原则

​	不能发生指令重排序的情况

```
程序次序规则：
	在单线程中，程序的执行结果和代码块的顺序执行的结果是一样的。即可以理解单线程中上一句的代码应该在下一句代	码执行前执行。
管程锁定规则
	对同一个锁，上一个加锁的unlock先于下一个加锁的lock
volatile变量规则
	同一时间，对volatile的写操作先于volatile的读操作
线程启动规则
	一个线程的start优先于线程执行中的每一个动作
线程终止规则
	线程中的所有操作都先于发生于线程的终止检测
线程中断规则
	对线程的interrupt()调用优先发生于被中断线程的代码检测到中断事件的发生。
对象终结规则
	一个对象的初始化（<clinit>()）优先于对象finalize()的调用
传递性
	A先于B，B先于C，则A先于C
```



####

#### Thread.stop（）和 Thread.interrupt()

​	强行中断线程，不安全，已经废弃，会导致数据不一致 。

​	建议 Thread.interrupt() ， 发送一个通知给线程让其停，线程再在合适的时间停止。



#### wait() \ notify() \notifyAll()

​	都是Object的方法，必须在 synchronizd 中运行。作用是 等待、唤醒在某个资源上的线程。

[为什么wait，notify，notifyAll这些方法不在thread类里面？](https://www.cnblogs.com/pzyin/p/11414077.html)

​	Java提供的锁是对象级的而不是线程级的，每个对象都有锁，通过线程获得。简单的说，由于wait，notify，notifyAll都是锁级别的操作，所以把他们定义在object类中因为锁属于对象。

#### suspend() 和 resume()

​	线程挂起和继续执行。废弃。会出现死锁：suspend() 挂起线程后不会释放已经占用的资源。需要等待 resume() 唤醒如果 resume() 在并发情况先于 suspend() 执行，那么就没有机会再解锁了，资源永远被占用。



suspend被弃用的原因是因为它会造成死锁。suspend方法和stop方法不一样，它不会破换对象和强制释放锁，相反它会一直保持对锁的占有，一直到其他的线程调用resume方法，它才能继续向下执行。

假如有A，B两个线程，A线程在获得某个锁之后被suspend阻塞，这时A不能继续执行，线程B在或者相同的锁之后才能调用resume方法将A唤醒，但是此时的锁被A占有，B不能继续执行，也就不能及时的唤醒A，此时A，B两个线程都不能继续向下执行而形成了死锁。这就是suspend被弃用的原因。



#### join() 和 yield()

​	等待线程结束和谦让

​	join ：等调用这个方法的线程执行完成后，调用了这个 调用这个方法的线程 的线程才继续执行，本身会占用锁。

​	yied ：释放自己的资源，重新进入就绪状态竞争资源。



### volatile

​	https://mp.weixin.qq.com/s/mJVkLn2I1O7V8jvxc_Z4zw

​	保证可见性、有序性，不保证原子性。 eg: 单例模式的双锁校验



#### 缓存一致性问题

​	CPU和主存之间增加缓存，在并发时，多核CPU中，每个核的自己的缓存中，关于同一个数据的缓存内容可能不一致（保留着主存的备份）。

#### 处理器优化

​	硬件问题。为了使处理器内部的运算单元能够尽量的被充分利用，处理器可能会对输入代码进行乱序执行处理。这就是**处理器优化**。

#### 指令重排序

​	除了现在很多流行的处理器会对代码进行优化乱序处理，很多编程语言的编译器也会有类似的优化，比如Java虚拟机的即时编译器（JIT）也会做**指令重排**。

#### 并发问题

​	可见性 、有序性 、原子性

​	**缓存一致性问题**其实就是**可见性问题**。而**处理器优化**是可以导致**原子性问题**的。**指令重排**即会导致**有序性问题**。

#### 守护进程



###ReentrantLock

```java
- synchronized 的扩展 ： 重入锁 ；
- 相比之下可以显式加锁、释放锁 ；
- 加锁一定要带释放锁，否则其他线程无法再进入同步代码块中 ；
- 为什么可重入：
	如果不可重入，那么同一个ReentrantLock对象连续两个加锁过程必然导致死锁，等自己释放，卡在第二次加锁
- 特点：
	· 可响应中断 （https://blog.csdn.net/cby1516/article/details/80674703）	
	· 可尝试加锁，失败放弃
	· 有时间限制的尝试加锁
	· 公平锁（synchronized是非公平的）

- Condition 条件
	· wait、notify 是配合 synchronized 
	· Condition的 await \ signal 是配合 ReentrantLock
	
- 例子 ： 
	ArrayBlockingQueue   //适合做生产者消费者的阻塞队列

```



### Semaphore 信号量

```
- 和ReentrantLock差不多，也是现实 获得信号量（加锁） 、 释放信号量（解锁）过程
- Semaphore ，信号量， 如果一个线程只申请一个信号量，那么创建多少个信号量，就有规定了只能有多少个线程可以同	时进入同一资源
- aquire \ release 获取、释放信号量
- 一定记得释放信号量，否则越来越少的线程可以进入资源区中。
- 例子 P84
```



### ReentrantReadWriteLock 读写锁

```java
- JDK5提供读写锁，普通的synchronized 、ReentrantLock 都是直接阻塞，不论读写
- 读线程间不会破坏数据完整性，所以不应该等待
- 读写锁特性： 读读不阻塞、读写阻塞、写写阻塞
```



### CountDownLatch 倒计时门闩

```
- 作用是让一定数量的线程都完成后才能接下去的操作，如同火箭发射，需要多个检查都完成后，才能发射
- CountDownLatch调用 await() ，即可开始等待定义的线程数完成。 
- 例子 P87
```



### CyclicBarrier 循环栅栏

```java
- 和CountDownLatch很像，不同的是，在规定数量的线程数完成后，可以开始执行下一个任务，而不是结束
- 下个任务是Runnable接口类型的
- 有两个异常：InterruptedException 、BrokenBarrierException
	第一个是其他类都有的，毕竟要响应中断
	第二个是此类特有的，作用是，当有某个线程出问题停止，其他线程不会等候，直接BrokenBarrierException，表
	达此CyclicBrrier已经损坏，就地解散。
```

### LockSupport 线程阻塞工具类

```
- 作用和suspend\resume是一样的，不同的是，更安全，不会发生死锁
- park() 、 unpark(Thread thread) 两个方法
- 底层是通过信号量实现，park消耗信号量，变为不可用，unpark则将信号量变为可用，提供消费。信号量只有一个，不会增加，所以，即使unpark发生在park之前，也会被park消耗掉，不会死锁。（park是在unpark之前的，因为逻辑上，只有park阻塞了，才会有unpark解锁，才有锁的意义，否则都不需要了）
```



## 锁优化

### “锁”性能优化建议

``` 
- 减少锁持有时间
- 减少锁粒度
	比如JDK1.8之前的ConcurrentHashMap，通过一个个Segment段减少加锁范围
- 根据操作类型进行锁分离
	如读写锁，读写分离，而不是独占锁
- 锁粗化
```

## Synchronized底层优化

### 偏向锁

```
- 针对加锁优化
- 如果一个线程获得了锁，就会进入偏向锁模式，当线程再次申请锁时，无需做任何同步操作，省去大量有关锁申请的操作
- 竞争不大时，偏向锁有用，竞争大时失效，因为总是不同的线程申请锁。
```

### 轻量级锁

```
- https://www.cnblogs.com/paddix/p/5405678.html
- 偏向锁失败，不会立即挂起线程，而是采用轻量级锁
- 轻量级锁只是简单地将对象头部作为指针指向持有锁的线程堆栈的内部，来判断一个对象是否持有对象锁，如果有则获得	 轻量级锁成功，进入临界区，否则表示已经被其他线程抢占，就要膨胀为重量级锁
	（用对象头作为指针指向“持有锁”的线程，看其是否获得锁）
- 轻量级锁并不是用来代替重量级锁的，它的本意是在没有多线程竞争的前提下，减少传统的重量级锁使用产生的性能消	
  耗。在解释轻量级锁的执行过程之前，先明白一点，轻量级锁所适应的场景是线程交替执行同步块的情况，如果存在同一 
  时间访问同一锁的情况，就会导致轻量级锁膨胀为重量级锁。
```

### 自旋

```
- 轻量级锁失败后，要膨胀为重量级锁，为了避免挂起线程，尝试自旋，看是否在一定时间内获得锁
- 因为有的加锁后很快释放锁，没必要挂起线程消耗资源。
- 自旋：进入一个空循环，循环一定次数还未获得锁，再挂起。
```

### 锁消除

```
- JIT编译中，对上下文扫描，通过逃逸分析、去除不可能被竞争的资源上的锁，节省时间。
- 比如在栈帧中的局部变量，若不会逃逸，就是私有的，不需要加锁。
```



##ThreadLocal 人手一支笔

```
剖析发现：
	Thread类含一个ThreadLocalMap的对象threadLocals，这个ThreadLocalMap是ThreadLocal的内部类。通过ThreadLocal创建的副本是存储在每个线程自己的threadLocals中的。
	是以ThreadLocal作为键，副本作为值。
	为什么要ThreadLocal作为键？因为每个线程都可以有多个ThreadLocal对象。
	
使用场景：
	想数据库的connection、Session管理（从工厂拿Session）等。

```



## 无锁

### 乐观锁悲观锁

```
乐观锁：冲突检测、数据更新两步
```



### CAS

```
https://www.cnblogs.com/lingz/p/9522616.html

- 是乐观锁的一种实现
- CAS（compare and swap），比较交换，有三个值： 内存地址V,旧值A ,新值B。比较V 和 A 的值，如果相等，就用B 	去替换A。 否则重试。
- JDK1.5后J.U.C的类是基于CAS的，如AtomicXxx类。非阻塞，不会产生死锁

- CAS问题：
	ABA问题。即改为A→B又改回A，看起来值相同但是版本不同，可以用版本号解决。
	未知的等待时长，这也是乐观锁的通病。如果竞争激烈，可能需要循环多次。
	
JDK1.8开始一个升级版的AtomicInteger，采用了分段CAS: LongAdder类
https://mp.weixin.qq.com/s?__biz=MzU0OTk3ODQ3Ng==&mid=2247484070&idx=1&sn=c1d49bce3c9da7fcc7e057d858e21d69&chksm=fba6eaa5ccd163b3a935303f10a54a38f15f3c8364c7c1d489f0b1aa1b2ef293a35c565d2fda&scene=21#wechat_redirect

```



### AtomicInteger

​	无锁的线程安全整数类，基于CAS操作。

### Unsafe类

​	底层和硬件交互的类，不让动，一改可能导致程序全部出错。

### AtomicReference

```
- 无锁的对象引用
- 和AtomicInteger很像，只是前者是对整数封装，后者是普通的对象引用。保证在修改对象引用时的线程安全
- 仍可能出现ABA问题，在商务赠款中需要防范P164。解决：带时间戳的对象引用
```



###AtomicStampedReference

```
- 带时间戳的无锁对象引用
- 每次更新对象时，必须同时更新合理的时间戳。（也不叫时间戳啦，就是一个标记）
```





## 线程同步

https://www.cnblogs.com/paddix/p/5381958.html 

### wait / notify

```
- 属于Object的native方法，可以直接调用
- 依赖同步锁，他们需要出现在同步代码中，依附于这个同步方法的同步监视器上。
- wait 释放锁并释放资源
```



### sleep

```
- Thread类的静态方法
- 不会释放锁，仅仅只是释放CPU资源 ； sleep一直持有同步监视器
```



### yield

```
- Thread的静态方法
- 目的是让当前线程从 Running 状态 → Runnable ，让出资源给其他线程
- yield之后线程可以重新竞争锁，但是不能保证一定竞争得到。
```



### join

```
- 调用此方法的线程将进入执行，而外层的这个线程将等待此线程执行完才能执行，当然可以设定时间
- 比如A线程中，调用了B.join() ， 则A线程必须等B执行完后才能执行.
- 不会释放已经占有的同步资源。
```



### sleep 和 wait区别

```
- 一个是Thread , 一个是Object
- sleep释放cpu但是不释放锁， wait全部释放
```



**如果线程A希望立即结束线程B，则可以对线程B对应的Thread实例调用interrupt方法。如果此刻线程B正在wait/sleep/join，则线程B会立刻抛出InterruptedException，在catch() {} 中直接return即可安全地结束线程。**

**需要注意的是，InterruptedException是线程自己从内部抛出的，并不是interrupt()方法抛出的。对某一线程调用interrupt()时，如果该线程正在执行普通的代码，那么该线程根本就不会抛出InterruptedException。但是，一旦该线程进入到wait()/sleep()/join()后，就会立刻抛出InterruptedException。**



**waite()和notify()因为会对对象的“锁标志”进行操作，所以它们必须在synchronized函数或synchronized block中进行调用。如果在non-synchronized函数或non-synchronizedblock中进行调用，虽然能编译通过，但在运行时会发生illegalMonitorStateException的异常。**



## 停止线程的方式

https://www.cnblogs.com/skywang12345/p/3479949.html

### 一、通过异常+return

​	此时线程可能 sleep() \ join() \ wait()  ，那么这时，直接调用线程的 interrupt() ，线程会产生一个 InterruptedException，捕获，然后return 即可。

​	运行过程调用interrupt（）不会强制终止线程，只是标了终止记号，我们可以通过 while(isInterrupted())判断，如果中断标记为true 就可以停下了。当然可能需要考虑 interrput() 时线程正处于阻塞状态，那么就需要捕获异常。

​	还有一种方式，通过自己定义的一个变量flag ， 但是为了保证线程的可见性，需要volatile修饰。



### 二、暴力停止

​	stop（）方法，容易导致数据不一致。

	其实stop方法天生就不安全，因为它在终止一个线程时会强制中断线程的执行，不管run方法是否执行完了，并且还会释放这个线程所持有的所有的锁对象。这一现象会被其它因为请求锁而阻塞的线程看到，使他们继续向下执行。这就会造成数据的不一致，我们还是拿银行转账作为例子，我们还是从A账户向B账户转账500元，我们之前讨论过，这一过程分为三步，第一步是从A账户中减去500元，假如到这时线程就被stop了，那么这个线程就会释放它所取得锁，然后其他的线程继续执行，这样A账户就莫名其妙的少了500元而B账户也没有收到钱。这就是stop方法的不安全性。


## 关于死锁

### 1、死锁小demo

```java
public class DeathLockDemo{
    public void method1() throws InterruptedException{
        synchronized (String.class){
            Thread.sleep(1000);
            synchronized (Integer.class){}
        }
    }
    public void method2() throws InterruptedException{
        synchronized (Integer.class){
            Thread.sleep(100);
            synchronized (String.class){}
        }
    }
    public static void main(String[] args){
        final DeathLockDemo deathLockDemo = new DeathLockDemo() ;
        Thread thread1 = new Thread(){
            @Override
            public void run() {
                try {
                    deathLockDemo.method1();
                    deathLockDemo.method2();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        };
        Thread thread2 = new Thread(){
            @Override
            public void run() {
                try {
                    deathLockDemo.method2();
                    deathLockDemo.method1();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        };
        thread1.start();
        thread2.start();
    }
}

```



### 2、死锁的四个必要条件

```
- 互斥条件：多个线程竞争一个互斥条件。
- 请求保持：线程请求资源未得时，能够保持原有的资源。
- 不可剥夺：线程或进程资源未用完时不能被其他线程或进程夺走，除非主动释放。
- 循环等待：多个线程首尾相接等待其他线程释放资源。
```



### 3、 如何预防死锁

```
- 一次性分配
- 顺序获取资源
- 银行家算法避免死锁。
```





## 线程同步的方式

```
临界区
	- 多线程串行化访问公共资源或一段代码，同一时刻仅一个线程能访问到。通过一抢占临界资源运行。
互斥量
	- 采用互斥对象机制，只有拥有互斥对象的线程才能访问公共资源。互斥对象仅有一个。
信号量
	- 信号量数量不止一个，所以允许多个规定数目的线程访问统一资源。是有数量限制的互斥量。
事件
	- 通过通知的方式保证线程同步，还可以方便对多个线程的优先级进行比较操作。
```



## 线程状态

https://blog.csdn.net/pange1991/article/details/53860651

```
New、 Runnable、 Running、 Wait、Blocked 、Terminated
```

### 1、Wait 和 Blocked区别

```
wait : 指当线程遇到wait、sleep、join方法时发生,wait状态指的是线程让出资源，等待另外的线程执行。可等待有  		限、无限时间、唤醒动作唤醒后醒来。
blocked : 指当线程等待同步监视器时所处状态，可以理解为等待monitor的状态，获取不到，一直blocked,如获取锁、		在等待进入同步代码块时，会进入blocked状态。
```

### 2、生产者消费者

```java
/**     
    * 生产者生产出来的产品交给店员     
    */    
    public synchronized void produce()    
    {
        if(this.product >= MAX_PRODUCT)
        { 
            try 
            {
                wait(); 
                System.out.println("产品已满,请稍候再生产");
            }
            catch(InterruptedException e) 
            {
                e.printStackTrace () ; 
            } 
            return; 
        } 

        this.product++;
        System.out.println("生产者生产第" + this.product + "个产品.");
        notifyAll();   //通知等待区的消费者可以取出产品了
    }
  
    /**      
     * 消费者从店员取产品
    */    
    public synchronized void consume()     
    {        
        if(this.product <= MIN_PRODUCT)        
        {           
            try              
            {                
                wait(); 
                System.out.println("缺货,稍候再取");
            }              
            catch (InterruptedException e)              
            {                 
                e.printStackTrace();             
            }            
            return;
        }                  
        
            System.out.println("消费者取走了第" + this.product + "个产品.");
            this.product--;
            notifyAll();   //通知等待去的生产者可以生产产品了
    }
```













































