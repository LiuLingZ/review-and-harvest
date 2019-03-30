

https://mp.weixin.qq.com/s/IJt-oYl5DVJIlc8uYwLj2w

# ThreadLocal



```
什么是ThreadLocal ？
	通过定义一个ThreadLocal,每个线程都往这个ThreadLocal存储属于自己的那份对共享变量的拷贝，各个线程对这个	  拷贝独立占有使用，不发生竞争，实现线程隔离机制。
	
使用场景：
	比如服务端可以创建一个ThreadLocal用来存储用户信息，当用户来访并在页面间进行多种业务时，可以直接获得属于	 自己的信息，ThreadLocal使线程隔离，使用属于自己的变量副本，多线程间互不干扰。

大致思路：
	Thread类中含了一个ThreadLocal.ThreadLocalMap的threadLocals实例，这意味每个线程都有自己的一个		threadLocals。ThreadLocalMap是个特殊的hashmap，是个环状的hashMap，每个节点是一个Entry。Entry里含	 key、value。可以简单认为，key是继承ThreadLocal的一个弱引用，value是存储的变量副本。	

```



## 1、存储结构

​	类似map，但不是map，只是概念上是Map，节点是一个个Entry实例，包含key \ value \ 前一个节点 \ 后一个节点 ，这样形成的是个环，大小是2的幂 ， 采用的是线性探测法来避免冲突。



## 2、弱引用和内存泄漏

​	Entry是一个继承WeakReference类的对象。弱引用的特点是，引用的对象活不到下一个GC 。 这样的好处是，一旦ThreadLocal被GC，那么引用这个ThreadLocal的Entry也就要被回收，方便内存回收。

​	但是其实也有问题，因为即使Entry被标记为可回收，但只是key断了，value还是有一个强引用，即备份还是通过一个Object对象引用，这一部分没有被标记为可回收。当然，这是和线程绑定的，这个线程结束，这部分的内存也被回收，但是在线程池的情况就不一定了，不是说不被回收，而是线程不会结束，如corePoolSize数目的保活，这部分对象没有被利用但也不会被回收，造成内存泄漏。其实也是概念问题，不会真的内存泄漏，只是按这种情况这么说，就可以理解为内存泄漏了。





## 3、类成员变量

​	负载因子 2/3 ；

​	Entry表， 必须是2的幂，为的是尽可能的均匀散列。可以参考hashMap的长度，原理是一样的。

```
这里解释下，为什么是2的幂 ？
	还要从ThreadLocal说起，因为Entry的key就是ThreadLocal，通过计算ThreadLocal的hashcode来确认位置。
	ThreadLocal本身含有一个魔数，相当于此ThreadLocal的ID，计算hashcode时需要用上， 计算出来的结果再
	与2的幂相与能较为均匀地散列在Entry[]上。
	即便存在冲突，也可以通过“线性探测法”寻找最近的下个空闲Entry,解决冲突。
	
“线性探测法”：
	类似停车。
```



## 4、关于回收

​	因为是弱引用，经过一次GC可能就不存在了，此时这个Entry实际就是没用的，需要更新，那么对于setEntry 、 getEntry 、 remove方法，都可能触发对Entry数组的全量清理。

​	同时为了防止上面的“内存泄漏”，建议就是删除ThreadLocal时，显示使用remove，会触发全量清理，清理没用的Entry。

