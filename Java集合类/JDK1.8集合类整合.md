#总概

![img](https://images2015.cnblogs.com/blog/249993/201611/249993-20161122113410534-705560500.jpg)

![img](https://pic002.cnblogs.com/images/2012/80896/2012053020261738.gif)

## 1、ArrayList

https://www.cnblogs.com/leesf456/p/5308358.html

- 核心是transient Object[] elementData ; 实例，真正存值。transient代表持久化时会被忽略。


- 如果创建一个ArrayList数组，没有指定长度，则添加元素时，或扩容到长度为10.

  如果创建时有指定长度，不会进行扩容。

- 除了未定义初始化的长度的第一次扩容（扩为10），其他每次扩容长度为1.5倍。

- 每次操作的第一件事，就是判断索引是否合理.

- ArrayList中是可以放null元素的。可以查找到null元素。

ArrayList有其特殊的应用场景，与LinkedList相对应。其优点是随机读取，缺点是插入元素时需要移动大量元素，效率不太高。

 

## 2、Vector

https://www.cnblogs.com/zhangzongle/p/5432212.html

- 底层是一个Object[] elementData 数组，和ArrayList一样，随机访问快，删除慢，需要移动元素
- 默认大小是10，可指定大小。每次扩容是2倍扩容
- 有个capacityIncrement ， 可以在初始化时指定，指定后，每次扩容都是扩 capacityIncrement 



## 3、CopyOnWriteArrayList

https://www.cnblogs.com/dolphin0520/p/3938914.html

**写时复制**；一种并发安全的ArrayList，其思想是：写的时候复制一份新的数组出来，在新的里面修改，修改完后，再用原来的引用去引用这个新的数组。这个过程中，读还是读原来的数组。

- 其add()方法是先加锁，再进行复制、新增的功能。加锁时为了防止并发copy出多个副本。
- 写时复制，适合读多写少的情况。比如黑名单、白名单。
- 写时复制的缺点：内存占用问题 ； 数据一致性问题。

内存占用：写的时候复制一份一模一样的大小的，在一个时间段内存驻留两个大对象，容易造成GC。

数据一致性：写的过程是复制，写；这个过程无法读到新的数据，所以要求立刻读到新的数据时，不要用CopyOnWriteArrayList。





##3、LinkedList

https://www.cnblogs.com/leesf456/p/5308843.html

- 实现了Deque接口，这意味是一个双向链表。可从头和尾开始遍历。确实，每个方法通过索引确认节点位置的时候，也是判断索引靠前还是靠后，达到最多一半遍历即可访问到。

- 链表的每个节点类Node含三个实例，Node next \ Node prev \E item(泛型)

  链表本身含三个实例： int size \ Node first \ Node last ，全部都是transient修饰

- 需要重视 addAll(int index ,Collection《？extend E》 c) ， 他是在index+1前面的位置添加

  并且，先将 c.toArray，其作用主要是在并发修改的时候，一个线程占用c的时间短，并且列表访问快，提高并发效率。

  



## 4、HashMap

### 一、JDK1.7

#### HashMap

详细介绍： https://mp.weixin.qq.com/s/dzNq50zBQ4iDrOAhM4a70A

https://mp.weixin.qq.com/s/KCxKKeJ28Yev5xsv4WzRog

https://mp.weixin.qq.com/s/dzNq50zBQ4iDrOAhM4a70A

https://www.cnblogs.com/chengxiao/p/6059914.html

- 底层是数组，每个数组元素是一个Entry对象（1.7；1.8为Node对象），有key\value\next三个实例，同时也是单向链表的头结点。
- 链表初始化长默认是16，并且长度必须是2的n次方，每次扩容扩容两倍长
- 扩容时机： 实际长度 ≥ 负载因子 * 总长 。 负载因子默认0.75，初始化时可修改。
- 扩容是就会涉及到 ReSize 和 ReHash过程。前者是扩容，后者是将元素从旧数组复制到新数组中。
- Resize 是先创建两倍长的数组，再从 旧数组Rehash到新的对应位置。
- HashMap在多线程是不安全的，并发插入元素后可能出现带环的链表，在下次访问时出现死循环。

- JDK1.7在put的时候判断有没有table，没有才真正初始化一个table。JDK1.8则是在put中先判断是否resize（）,因为一开始没有容量，所以触发第一次初始化。





#### ConcurrentHashMap

https://mp.weixin.qq.com/s/1yWSfdz0j-PprGkDgOomhQ

- 线程安全有Hashtable 和 Collections.synchronizedMap，但是性能都不佳。无论读写操作，都只是给方法单纯加锁。 ConcurrentHashMap 则为优选

- 引入Segment的概念，每个Segment也是一个HashMap对象，整个ConcurrentHashMap是由多个Segment组成

- Entry变成HashEntry，且其value 和 next都变成 volatile修饰，为了可见性

- 这样设计：Segment高度自治，多个Segment间互不影响。同一个Segment可并发进行读写，不可并发写，并发写先自旋 获取锁，一直失败才阻塞挂起。降低锁粒度，提高效率

- ConcurrentHashMap的size（）采用乐观锁的实现。即统计长度时，先从每个Segment统计Entry个数，最后合并。期间还会统计所有Segment修改次数，如果和统计数目前的修改次数一致，则统计成功，如果不一致，重新统计（因为统计过程有其他线程并发增加、删除了元素）。当失败次数达到一定次数，就直接锁住全部Segment，再重新统计。

- 关于put（），get()

  第一次hash，定位到Segment,获得此Segment的可重入锁，再次hash，定位到具体位置，覆盖或插入。释放锁。

  第一次hash，定位到Segment,再次hash，定位到具体位置。

  两次hash是为了精确定位到Segment中的位置。

  

  

  

### 二、JDK1.8

#### HashMap

http://www.codeceo.com/article/java-hashmap-concurrenthashmap.html

- 为了解决频繁hash碰撞导致桶中形成过长的链表，遍历失去优势O(n)。所以设定一个阈值，达到阈值后自动转换为红黑树。
- 红黑树阈值：TREEIFY_THRESHOLD = 8; 大于等于则转换为红黑树
- 其余初始化、负载因子什么的和1.7差别不大
- put()过程：
  1. 判断当前桶是否为空，空的就需要初始化（resize 中会判断是否进行初始化）。
  2. 根据当前 key 的 hashcode 定位到具体的桶中并判断是否为空，为空表明没有 Hash 冲突就直接在当前位置创建一个**新桶**即可。
  3. 判断桶中是红黑树还是链表，遍历比较是否有相同的key，若无相同
  4. 如果当前桶为红黑树，那就要按照红黑树的方式写入数据。
  5. 如果是个链表，就需要将当前的 key、value 封装成一个新节点写入到当前桶的后面（形成链表）。
  6. 接着判断当前链表的大小是否大于预设的阈值，大于时就要转换为红黑树。
  7. 如果在遍历过程中找到 key 相同时直接退出遍历。那就需要将值覆盖。
  8. 最后判断是否需要进行扩容。
- get()过程：
  1. 计算hashcode，然后找到对应桶，判断第一个是否相同，若是返回，若不是
  2. 判断桶是链表还是红黑树，按照其规则遍历
  3. 匹配返回
- 修改之后，查找优化，修改为红黑树之后查询效率直接提高到了 `O(logn)`
- 但是1.7出现的并发增加导致的循环链表仍可能出现。

```
1.8put()过程：实际是putVal()方法
- 判断table是否为空，是就创建一个，并且获得长度
- 计算key的索引位置，如果该位置没有元素，直接放入
- 如果有元素，判断是否是同一元素，是的就覆盖
- 不是的话，判断是否为红黑树，是红黑树就直接插入。
- 不是红黑树，遍历节点，并且判断链表长度，如果链表长度大于8，转为红黑树
- 遍历链表或红黑树，覆盖或者新增节点。并改变相关的值，长度啊，modCount之类。

1.8get()
- 判断表或key是否是null，如果是直接返回null
- 判断索引处第一个key与传入key是否相等，如果相等直接返回
- 如果不相等，判断链表是否是红黑二叉树，如果是，直接从树中取值
- 如果不是树，就遍历链表查找
```





####ConcurrentHashMap

http://www.codeceo.com/article/java-hashmap-concurrenthashmap.html

- 放弃了原来Segment的概念，改用CAS+synchronized
- 原来的HashEntry变为Node，但是作用相同，其中value 和 next都是volatile修饰保证可见性。
- put（）：
  1. 根据key计算hashcode
  2. 判断是否需要初始化
  3. 通过key定位，为空、根据链表或红黑树的规则，通过CAS尝试，失败则synchronized
  4. 判断是否需要变为红黑树
  5. 判断是否需要扩容



#### TreeMap

- 也是一个map，支持key-value;
- 底层根据红黑树实现，get \ put \ containKey \ remove 等节点操作时间复杂度都是 O(logn)
- 并不是线程安全的，支持 fast-fail
- 底层key是有序的，可以返回有序的keys集合，因为红黑树。
- 默认key是根据自然排序（数从小到大，字母从a到z），也可以在初始化的时候提供一个compare，即提供一个排序规则。









#### 红黑树（自平衡的二叉搜索树）

```
二叉搜索树（BST，Binary Search Tree）
	特点：
		1.左子树上所有结点的值均小于或等于它的根结点的值。
		2.右子树上所有结点的值均大于或等于它的根结点的值。
		3.左、右子树也分别为二叉排序树。
	缺点：
		可能变成一棵瘦高的树，失去高效
		
红黑树（Red Black Tree）
	特点：
		1.节点是红色或黑色。
		2.根节点是黑色。
		3.每个叶子节点都是黑色的空节点（NIL节点）。
		4 每个红色节点的两个子节点都是黑色。(从每个叶子到根的所有路径上不能有两个连续的红色节点)
		5.从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。
	
	变色
		为了重新符合红黑树的规则，尝试把红色节点变为黑色，或者把黑色节点变为红色。
	自旋
		左自旋：逆时针旋转红黑树的两个节点，使得父节点被自己的右孩子取代，而自己成为自己的左孩子。
		右自旋：顺时针旋转红黑树的两个节点，使得父节点被自己的左孩子取代，而自己成为自己的右孩子。
	
	
```





# 关于Map

https://www.cnblogs.com/xiaoming0601/p/5864106.html



## 1、HashMap当key为null时的处理：

```
JDK1.6
	如果key为null，直接遍历第一个桶，看有没有key为null的键值，有就覆盖替换，无就作为头节点存放。
JDK1.8
	如果key==null，就将其hashcode置为 0 ,再进行普通key的定位、存放。
```

注:

```
HashMap可以接受key为null ，而 HashTableb不可，因为，HashTable是需要用到equals的
HashTable就是直接在get \ put方法加 synchronized，效率低的。
```

## 2 、减少hash碰撞

​	尽量让不同对象产生不同的hashcode， 并且较为均匀地散列在数组上，减少链表的长度。

​	使用不可变类如String 、 Integer作为key的好处：自身重写了equals和hashCode方法，类不可变保证每次获取的hashcode都是一样的，确保存储的key是一致的。



##3 、为什么使用红黑树而不是二叉搜索树、B+树

```
二叉搜索树：
	- 若有左子树，则左子树的节点的任何值都小于等于根节点；若有右子树，也是如此
	- 一些极端情况下，二叉搜索树可能退化成普通的链表

红黑树：
	根节点黑色
	子节点可为红色或黑色
	红节点的子节点必须是黑色，这意味着两个红色节点不会相邻
	插入数据需要旋转、变色等等。
	黑色节点都是null
	叶子节点都是黑色节点
	从根节点到每个叶子节点，所经路径上的黑色节点数是一样多的。
	
B+树：
	相对而言，B+树就有个问题了，所有因为B+树存储过多的冗余数据，子节点存了父节点的值，这是没必要的。
	
B树：
B树的使用场景
B树多用于做文件系统的索引。

那么问题来了：为什么要用B树，红黑树不是就挺好的么？
原因：
B树和二叉树、红黑树相比较，子节点中一个节点存储的数更多，数越多表示数的高度越低，搜索效率越高，

回到正题：正因为文件系统和数据库一般都是存在电脑硬盘上的，如果数据量太大的话不一定能一次性加载到内存中。（一棵树不能一次性加载完怎么查找对吧？）但是B树可以多路存储。也正因为B树的这一个优点，可以在文件查找的时候每次只加载一个节点的内容存入内存来查找。而红黑树在内存中查找非常块，但是如果在数据库和文件系统中，显然B树更优。

而这里不用B树的原因，主要是B树本身的节点还是存了很多值的，因为要进行多路查找，那么在节点中查找，也是个O（k）的过程，可能出现要连续多次O（k），不划算。



所以，使用红黑树，搜索的时间复杂度就变为O（logN），即树的深度。

```





## 4、Hashtable vs ConcurrentHashMap

```
- HashTable 是线程安全的，put\get直接加synchronized实现，效率较低
- HashTable不允许 key为null ，因为她需要用到 equals方法比较key
- HashTable初始长度为11，负载因子0.75 ， 扩容是2*capacity + 1 （因为可能是质数）
- 建议初始化长度为质数，，因为！！HashTable确定位置是通过 hashcode对长度取模，而对质数取模可以完整均匀地分布在长度中。


两者比较：
- 两者都是通过加锁保证线程安全，但是加锁程度不同
- ConcurrentHashMap采用局部加锁的特点，1.7对某个Segment分段加锁 ； 1.8开始在链表（红黑树）中采用CAS修改，不行再加synchronized。1.8的变量都是volatile修饰，保证可见性。
- 虽然插入数据，红黑树需要变色、旋转等调整策略，但是和搜索效率比起来还是可以接受的


CAS：
	特点：比较-交换，乐观锁的实现
	缺点：ABA问题，失败重试带来的延时





```



 	