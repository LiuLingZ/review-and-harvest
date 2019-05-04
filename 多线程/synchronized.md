# synchronized关键字



## 一、使用场景

### 1、同步方法

​	对于同一个实例，多个线程共享这个实例时，同一时间，只有一个线程可持有这个实例，并且进入同步方法中。

```java
/**
 * 用在普通方法
 */
private synchronized void synchronizedMethod() {
    System.out.println("synchronizedMethod");
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```



### 2、同步静态方法

​	静态方法属于类，这意味着，即使不同同一个实例，同一时间也只能有个一个线程携带着一个实例去执行同步方法。

```java
/**
 * 用在静态方法
 */
private synchronized static void synchronizedStaticMethod() {
    System.out.println("synchronizedStaticMethod");
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```



### 3、同步类（代码块）

​	这和同步静态方法是一样的，将类对象作为锁，这样同一时间只能有一个线程去执行同步代码块，因为只能有一个线程去持有类对象

```java
/**
 * 用在类
 */
private void synchronizedClass() {
    synchronized (TestSynchronized.class) {
        System.out.println("synchronizedClass");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

/**
 * 用在类
 */
private void synchronizedGetClass() {
    synchronized (this.getClass()) {
        System.out.println("synchronizedGetClass");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



### 4、同步this对象（代码块）

​	效果和同步方法是一样的，共享对象，同一时间仅一个线程能持有并执行

```java
/**
 * 用在this
 */
private void synchronizedThis() {
    synchronized (this) {
        System.out.println("synchronizedThis");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



### 5、同步对象（代码块）

​	和同步this对象思想是一样的，只是这里锁住的是另外一个共享的对象

```java
/**
 * 用在对象
 */
private void synchronizedInstance() {
    synchronized (LOCK) {
        System.out.println("synchronizedInstance");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```





## 二、synchronized底层优化

### 偏向锁

```
- 针对加锁优化
- 如果一个线程获得了锁，就会进入偏向锁模式，当线程再次申请锁时，无需做任何同步操作，省去大量有关锁申请的操作
- 竞争不大时，偏向锁有用，竞争大时失效，因为总是不同的线程申请锁。

简单理解：一个线程再次申请获得锁时，直接判断是否已经占有锁，是的话就直接获取锁。
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
  
 
 简单理解：就是CAS加锁过程。竞争不大才可
```

### 自旋

```
- 轻量级锁失败后，要膨胀为重量级锁，为了避免挂起线程，尝试自旋，看是否在一定时间内获得锁
- 因为有的加锁后很快释放锁，没必要挂起线程消耗资源。
- 自旋：进入一个空循环，循环一定次数还未获得锁，再挂起。


简单理解：线程阻塞挂起耗时可能比代码执行时间还长，为了避免短代码加锁，尝试等待一段时间（自旋），看是否释放锁。
```

### 锁消除

```
- JIT编译中，对上下文扫描，通过逃逸分析、去除不可能被竞争的资源上的锁，节省时间。
- 比如在栈帧中的局部变量，若不会逃逸，就是私有的，不需要加锁。
```



## 三、底层特点

### 同步方法

​	synchronized修饰的同步方法，在其所在的方法常量池中有一个 ACC_SYNCHRONIZED 标志修饰，当程序执行判断到有这个标识时，就需要获取同步监视器

### 同步代码块	

​	同步代码块则是通过 monitorenter 和 monitorexit 包围同步程序，同一时间只能有一个线程获取到同步资源并进入同步代码块。