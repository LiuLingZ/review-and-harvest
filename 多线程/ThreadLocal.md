

https://mp.weixin.qq.com/s/IJt-oYl5DVJIlc8uYwLj2w

# ThreadLocal



```
什么是ThreadLocal ？
	通过定义一个ThreadLocal,每个线程都往这个ThreadLocal存储属于自己的那份对共享变量的拷贝，各个线程对这个	  拷贝独立占有使用，不发生竞争，实现线程隔离机制。
	
使用场景：
	比如服务端可以创建一个ThreadLocal用来存储用户信息，当用户来访并在页面间进行多种业务时，可以直接获得属于	 自己的信息，ThreadLocal使线程隔离，使用属于自己的变量副本，多线程间互不干扰。

大致思路：
每个线程Thread都有一个 
ThreadLocal.ThreadLocalMap 对象，
Entry : key value 
key 就是指向那个ThreadLocal的弱引用，value就是我们的真实值
所以说，真正存储值的，是各自的每个线程，通过这个ThreadLocal识别存的是哪个Key

get时，获得当前线程的ThreadLocalMap，然后通过ThreadLocal找到key value（通过get方法就可看到这个过程。）
set时，通过线性探测法插入。


```



## 1、存储结构

​	类似map，但不是map，只是概念上是Map，节点是一个个Entry实例，大小是2的幂 ， 采用的是线性探测法来避免冲突。

```java
ThreadLocalMap含有一个Entry类型的数组， Entry[] ， 查看这个Entry，是实现了WeakReferance的类
翻看 Entry :
	static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
从这个 super(k) ，进入看源码，可以看到，最终，引用ThreadLocal的弱引用被作为key
这就解释了，为什么key被回收后，value还活着，因为是entry的一个字段（key）被回收，而不是整个entry！！！！
所以：
	Thread → ThreadLocalMap → Entry[] → entry.object 是一个强可达的。 
```







## 2、弱引用和内存泄漏 

​	Entry是一个继承WeakReference类的对象。弱引用的特点是，引用的对象活不到下一个GC 。 这样的好处是，Entry不会和该线程强绑定，就不至于跟着线程的生命周期存活，避免一直占用内存。

​	这里要注意，实际上是ThreadLocal如果被回收，即为null，那么 Entry 的key == null 了，虽然是弱引用类型，但是，此时还没有被回收，所以，还是有一条引用 ： Thread -> ThreadLocalMap -> Entry -> value ，所以此时内存被占用，但是不会被使用，只要GC一定会被回收，但是没GC就没法回收了，所以，所谓内存泄漏，看时机。

​	Entry的key本身也是一个弱引用，指向ThreadLocal 。GC时，这个key会断掉，key==null 。Entry本身也会被回收。







## 3、类成员变量

​	负载因子 2/3 ；（len * 2/3）

​	Entry表， 必须是2的幂，为的是尽可能的均匀散列。可以参考hashMap的长度，原理是一样的。

```
这里解释下，为什么是2的幂 ？
	还要从ThreadLocal说起，因为Entry的key就是ThreadLocal，通过计算ThreadLocal的hashcode来确认位置。
	ThreadLocal本身含有一个魔数，相当于此ThreadLocal的ID，计算hashcode时需要用上， 计算出来的结果再
	与2的幂相与能较为均匀地散列在Entry[]上。
	即便存在冲突，也可以通过“线性探测法”寻找最近的下个空闲Entry,解决冲突。

Entry[]是逻辑上的一个环，提供了往前或者往后的索引。

	
“线性探测法”：
	类似停车。
```





## 4、关于key的定位

```
	类似于HashMap的定位，长度为2的幂，而为了均匀散列，ThreadLocal这样计算自己的hashcode :
	
	每个ThreadLocal初始化时都有一个“魔数”成员变量，通过这个魔数计算出TnreadLocal自己的hashcode，相当于这个ThreadLocal的ID，在于长度-1相与，就可以均匀散列了。
	
	并且对于hash冲突，采用线性探测法。
	
	如果获得索引后去取，不是想要的（即在上一次可能通过线性探测法被别的元素占用了），就向下遍历，遇到key==null的，即ThreadLocal已经被清除了，就会调用清除方法，把这个无效的Entry给清除。然后再向下遍历。直到找到对应的Entry
```



## 5、关于扩容

```
阈值定于 长度的 2/3 ： threshold = len * 2/3 ， 如果到达，就会触发一次rehash，rehash的同时会触发一次全量清除，清除那些已经失效的Entry,然后rehash，之后得到的数组长度 , 如果还超过数组容量的 1/2 ： 
threshold-threshold * 1/4 ， 就会彻底触发扩容。扩容2倍。
```



## 6、关于remove

```
直接清除了对应位置的值，完全清空，所以建议当一个ThreadLocal弃用的时候，显示调用一次 threadLocal.remove()
```









