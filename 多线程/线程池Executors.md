# 线程池

## 1、JDK对线程池的支持

​	JDK提供一套Executor框架，其中：

```java
Executors ：它扮演一个线程池工厂的角色。可以通过其获得特点功能的线程池（ThreadPoolExecutor）、
		   ThreadFactory(线程工厂)。 
ThreadPoolExecutor : 表示一个线程池 ，可以通过new构造一个自己的线程池 。

```

