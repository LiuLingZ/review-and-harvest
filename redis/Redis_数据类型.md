## Redis （一）

它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）

### 基础数据结构 

​	 不同类型的数据结构差异就在于value的结构不一样



#### string（字符串） 

- 唯一key对应value 。类似Java中的ArrayList。

- 常用：缓存用户信息，将用户信息结构体用JSON序列化成字符串，再将序列化后的字符串用redis缓存。相同的，取用户信息就是一次反序列化的过程。

- 关于长度和扩容

  - redis的字符串限定最长 512M 
  - redis的字符串是动态可变长的，有两个变量：length 和 capacity。length是已使用长度，capacity是实际长度，length <= capacity 
  - 当字符串长小于1M时，扩容是加倍的 ； 当长度大于1M时，每次扩容只增加1M。

- 操作（简略）

  - set name1 ling1   ;    get name1
  - 批量操作节省网络开销
    - **mset** name1 ling1 name2 ling2 name3 ling3 ;
    - **mget** name1 name2 name3

- 过期和set扩展

  - set name ling	

    expire name 5      #5s后过期

  - setnx name ling    #如果name不存在则添加

  - setex name 5 ling  #设置name=ling，5秒后过期。相当于 set + expire

- 计数（若value是整数，可以运算）

  - 自增：最大范围是 signed long的最大最小值，超过报错。

    - set age 20

      incr age # 21

      incrby age 5 # 26

      incrby age -5 #21

      set codes 9223372036854775807  # Long.Max

      incr codes   # 报错

- 字符由多个字节组成，一个字节由8个bit组成，所以一个字符串看成很多bit组合，是bitmap[位图]数据结构。

```
要点：
- 类似Java的ArrayList,由一个个字节组成，一个字节占8 bit 。
- 动态变更长度，可扩容。＜1M，则每次 * 2 ； ≥ 1M，每次扩容 +1M。


△
-底层是SDS结构，simple dynamic string , 和arraylist差不多，是基于数组，预先设定容量。
-通过debug object 键名 可以发现，SDS有两种结构 ： emb 和 raw 形式。【看encoding属性】
-小字符串（44字节内）是emb， 大于则是raw形式。


```

#### list（列表）

- 类似Java中的LinkedList，是链表不是数组，所以插入删除很快O(1)，但是索引定位很慢。

  列表弹出一个最后一个元素，此结构被删除，回收内存。

- 可以做异步队列，将延后执行的任务体序列化成字符串后存储到列表中，另一线程从列表轮询数据处理。

- 右进左出：队列

  - rpush books python java php

    llen books # 3

    lpop books # 弹出左边第一个python 

- 右进右出：栈

  - rpush books python java php

    rpop books # php

- lindex 类似Java 的 get(int index) ，O(n)慎用

   ltrim 跟的两个参数 start_index 和 end_index 定义了一个区间，在这个区间内的值，ltrim 要保留，区间之外统统砍掉。我们可以通过ltrim来实现一个定长的链表，这一点非常有用。

  - rpush books python java php

    lindex books 0  # php,右进左出

    lrange books 0 -1 #获取所有元素O(n)慎用，-1代表最后一位

    ltrim books 1 -1  #O(n) 慎用

    ltrim books 1 0    #这其实是清空了整个列表，因为区间范围长度为负，此时llen books

- 快速列表

   - redis的列表底层不仅仅只类似LinkedList，如果只是单纯的list,使用一个个内存块，容易造成内存的碎片化。并且一个结点就要额外存储pre和next两个指针，容易造成内存的浪费。因此，当元素比较少时，就使用一整块的连续空间，这空间里面就是个数组，此结构称为**ziplist(压缩列表)** ；当元素较多时，就分为多个ziplist , ziplist 本身保存了pre 和 next指向别的ziplist， 形成链，此链称为 **快速列表（quicklist）** 。这样就删除了大量的冗余内存，并且减少内存的碎片空间。（这倒和ConcurrentHashMap底层原理差不多，一个一个segment组成一个concurrentHashMap）

```
要点：
- 类似Java的LinkedList ，双向链表。
- 根据规定的出队入队顺序，可做 队列 或 栈 。 
- 底层其实是 快速链表 的形式。元素节点 → 压缩链表 → 快速链表 
```






#### hash

- 底层和Java的HashMap一样，无序字典，一维碰撞后，碰撞的元素形成链表拼接起来

- 与Java不同的是，Java的rehash是直接完成的，这期间该线程无法执行其他任务。而redis的hash则采用 **渐进式rehash** 策略。

  - 渐进式rehash：rehash时产生一个新的长度的hash，然而并不立即进行rehash操作，而是同时保留 新旧两个hash，同时操作，在后期的定时任务中将旧hash中的内容一点点迁移到新hash中，最终结束，回收旧hash的内存，使用新的hash。

- hash也可存储用户信息，不同于字符串一次性全部序列化整个对象，hash可以对用户结构中的每个字段单独存储，这样我们就能部分获取所需的属性，而不用一次性全部读耗费流量。

- hash缺点也是明显的， 相比字符串更占内存。到底使用hash还是字符串需要考虑。

- 操作

  - hset books java 'think in java'

    hset books  golang "concurrency in go"

    hgetall books  # entries()，key 和 value 间隔出现

    hlen books

    hget books java

    hset books golang "learning go programming"  # 因为是更新操作，所以返回 0

    hmset books java "effective java" python "learning python" golang "modern golang programming"  # 批量 set

  - 同字符对象一样，hash结构单个子key也可计数，对应指令hincrby 和 incr一样

    hincrby user-ling age 2 # age+2

```
注意：
- hash只能存字符串
- string虽然比hash省空间，但是读取效果不如hash，因为string需要完整取出再读取，hash可以取出部分属性直接读。
```



#### set（无序集合）

- 类似Java的HashSet，内部是无序键值对，是一个特殊字典，因为value都为null 。

- set具备去重功能，可以做ID去重。

- 操作：

  - sadd books python java

    smembers books

    sismember books rust  # 判断rust是否在books中

    scard books # 获取长度

    spop books # 弹出一个，任意的



#### zset（有序集合）

- 带权值的有序集合，类似Java的SortedSet和HashMap的结合，即是set又是key-value。**每个value都会赋予一个score**，代表排序的权重。内部采用“跳跃列表”数据结构。

- 操作

  - zadd books 9.0 "think in java"

    zadd books 8.9 "java concurrency"

    zadd books 8.6 "java cookbook"

    zrange books 0 -1  # 按 score 排序列出，参数区间为排名范围 ，**从低到高**

    zrevrange books 0 -1  # 按 score 逆序列出，参数区间为排名范围，**从高到低**

    zcard books  # 相当于 count()

    zscore books "java concurrency"  #获取指定 value 的 score，内部 score 使用 double 类			              

    ​							    .# 型进行存储，所以存在小数点精度问题	

    zrem books "java concurrency"  # 删除 value

```
要点：
- 带分数的集合
```





### 关于过期时间

​	Redis 所有的数据结构都可以设置过期时间，时间到了就删除对象。过期是以对象为单位，一个过期，对象中的所有值都过期。

​	**如果一个字符串已经设置过期时间，再set修改该字符串，则过期时间会消失**（其他结构则不会改变原来设置的）

​	set codehole yoyo

​	expire codehole 600

​	set codehole yoyos

​	ttl codehole # -1 已经永久有效了