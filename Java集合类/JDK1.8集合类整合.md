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

- 底层是一个Object[] elementData 数组，和ArrayList一样，随机访问快，删除慢，需要移动元素，是并发安全的，方法都加了synchronized修饰。
- s默认大小是10，可指定大小。每次扩容是2倍扩容
- 有个capacityIncrement ， 可以在初始化时指定，指定后，每次扩容都是扩 capacityIncrement 



## 3、CopyOnWriteArrayList

https://www.cnblogs.com/dolphin0520/p/3938914.html

**写时复制**；一种并发安全的ArrayList，其思想是：写的时候复制一份新的数组出来，在新的里面修改，修改完后，再用原来的引用去引用这个新的数组。这个过程中，读还是读原来的数组。

- 其add()方法是先加锁，再进行复制、新增的功能。加锁时为了防止并发copy出多个副本。
- 写时复制，适合读多写少的情况。比如黑名单、白名单。
- 写时复制的缺点：内存占用问题 ； 数据一致性问题。

内存占用：写的时候复制一份一模一样的大小的，在一个时间段内存驻留两个大对象，容易造成GC。

数据一致性：写的过程是复制，写；这个过程无法读到新的数据，所以要求立刻读到新的数据时，不要用CopyOnWriteArrayList。

```java
/*
	CopyOnWriteArrayList 的 add() ，通过可重入锁加锁。
*/

public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```





##3、LinkedList

https://www.cnblogs.com/leesf456/p/5308843.html

- 实现了Deque接口，这意味是一个双向链表。可从头和尾开始遍历。确实，每个方法通过索引确认节点位置的时候，也是判断索引靠前还是靠后，达到最多一半遍历即可访问到。

- 链表的每个节点类Node含三个实例，Node next \ Node prev \E item(泛型)

  链表本身含三个实例： int size \ Node first \ Node last ，全部都是transient修饰

- 需要重视 addAll(int index ,Collection《？extend E》 c) ， 他是在index+1前面的位置添加

  并且，先将 c.toArray，其作用主要是在并发修改的时候，一个线程占用c的时间短，并且列表访问快，提高并发效率。

  



## 4、HashMap

### 一、JDK1.7

#### HashMap(1.7)

详细介绍： https://mp.weixin.qq.com/s/dzNq50zBQ4iDrOAhM4a70A

https://mp.weixin.qq.com/s/KCxKKeJ28Yev5xsv4WzRog

https://mp.weixin.qq.com/s/dzNq50zBQ4iDrOAhM4a70A

https://www.cnblogs.com/chengxiao/p/6059914.html

- 底层是数组，每个数组元素是一个Entry对象（1.7；1.8为Node对象），有key\value\next三个实例，同时也是单向链表的头结点。
- 链表初始化长默认是16，并且长度必须是2的n次方，每次扩容扩容两倍长
- 扩容时机： 实际长度 ≥ 负载因子 * 总长 。 负载因子默认0.75，初始化时可修改。
- 扩容是就会涉及到 ReSize 和 ReHash过程。前者是扩容，后者是将元素从旧数组复制到新数组中。
- Resize 是先创建两倍长的数组，再从 旧数组Rehash到新的对应位置。
- HashMap在多线程是不安全的，并发插入元素后如果导致并发resize，可能出现带环的链表，在下次访问时出现死循环。https://coolshell.cn/articles/9606.html

- JDK1.7在put的时候判断有没有table，没有才真正初始化一个table。JDK1.8则是在put中先判断是否resize（）,因为一开始没有容量，所以触发第一次初始化。





#### ConcurrentHashMap(1.7)

https://mp.weixin.qq.com/s/1yWSfdz0j-PprGkDgOomhQ

https://blog.csdn.net/qq_33589510/article/details/79962152

- 线程安全有Hashtable 和 Collections.synchronizedMap，但是性能都不佳。无论读写操作，都只是给方法单纯加锁。 ConcurrentHashMap 则为优选

- 引入Segment的概念，每个Segment也是一个HashMap对象，整个ConcurrentHashMap是由多个Segment组成

- Entry变成HashEntry，且其value 和 next都变成 volatile修饰，为了可见性

- 这样设计：Segment高度自治，多个Segment间互不影响。同一个Segment可并发进行读写，不可并发写，并发写先自旋 获取锁，一直失败才阻塞挂起。降低锁粒度，提高效率

- ConcurrentHashMap的size（）采用乐观锁的实现。即统计长度时，先从每个Segment统计Entry个数，最后合并。期间还会统计所有Segment修改次数，如果和统计数目前的修改次数一致，则统计成功，如果不一致，重新统计（因为统计过程有其他线程并发增加、删除了元素）。当失败次数达到一定次数，就直接锁住全部Segment，再重新统计。

  ```java
  https://www.cnblogs.com/study-everyday/p/6430462.html
  1.7时底层是 HashEntry[]
  
  初始化：
  - concurrentHashMap通过位运算进行初始化segment的大小，sszie表示，默认是16，最大可以是65536个段（因为 
    最大规定只能向左移16位,属性是并发级别设置为16）
  - HashEntry的也是通过左移确定大小的。所以只能是2的n次方大小，这也与hash相与确定位置有关。
  
  put()
  - put操作，需要两次定位，Segment是继承于ReentrantLock的，所以Segment具备锁能力 ；
  - 第一次定位到Segment，如果Segment还没有初始化，就通过CAS进行初始化，成功则下一步，不成功可能就初始化 
    好了。
  - 定位到Segment后，接下来就在要插入的位置，通过继承得到的tryLock()尝试加锁，加锁成功则直接插入到链表尾
    部，如果tryLock失败说明有正在加锁插入的，则自旋等待释放锁，如果自旋次数超过阈值就挂起阻塞等待。
  （CAS初始化Segment，tryLock加对应位置的锁，成功则插入，失败自旋等待，超过阈值阻塞挂起。）
  
  
  get()
  - 两次定位，第一次hash定位到Segment，第二次才定位到准确位置，然后遍历链表比较匹配，匹配成功返回，失败null.
      
  
  size()
  - 是通过统计segment的段的size得到的。
  - segment继承于ReentrantLock,他是先不加锁，尝试直接对每个段进行统计，相加，然后再进行重复统计，相加，   比较两次结果，如果相同，就说明没有在统计时修改元素，返回。而如果不同，则重试，直到次数小于      
    RETRIES_BEFORE_LOCK.默认是失败重复一次，如果失败就直接用ReentrantLock加锁统计，再求和比较，这时候就   一样了，就返回。
  ```

  

  

  

### 二、JDK1.8

#### HashMap(1.8)

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

- 转红黑树过程：

  如果链表节点数大于等于8个，就调用TreeBin这个内部树的工具的方法，根据每个链表的节点，创建出新的树节点，然后构造红黑树。





####ConcurrentHashMap(1.8)

http://www.codeceo.com/article/java-hashmap-concurrenthashmap.html

- 放弃了原来Segment的概念，改用CAS+synchronized
- 原来的HashEntry变为Node，但是作用相同，其中value 和 next都是volatile修饰保证可见性。
- put（）：
  1. 根据key计算hashcode
  2. 判断是否需要初始化
  3. 通过key定位，为空、根据链表或红黑树的规则，通过CAS尝试，失败则synchronized
  4. 判断是否需要变为红黑树
  5. 判断是否需要扩容

```
https://www.cnblogs.com/study-everyday/p/6430462.html
JDK1.8的实现已经摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来实现，并发控制使用Synchronized和CAS来操作，整个看起来就像是优化过且线程安全的HashMap，虽然在JDK1.8中还能看到Segment的数据结构，但是已经简化了属性，只是为了兼容旧版本

Node内部类
- 是ConcurrentHashMap的基本存储单元，继承与Map的Entry，用于存储数据，提供的方法是只能读不能修改。
- 其中的find方法就是比较这个node节点用


TreeNode内部类
- 用于红黑树，当链表长度>8，转红黑树，这个TreeNode就是存红黑树节点用的。继承与上面的Node内部类。

TreeBin内部类
- 理解为存储树形结构的容器，就是用来存储TreeNode的，封装了链表转为红黑树的方法和红黑树锁控制的方法、节点查
  找方法，相当于功能集合。
 
初始化
- ConcurrentHashMap的构造器是空的，初始化操作在put中。

put操作
- 如果没有初始化，就调用initTable()进行初始化。
- 初始化后，尝试用CAS插入，没有冲突则插入成功。
- put时可能在扩容，那就先扩容；
- 如果有冲突，CAS插入失败，就通过加锁方式synchronized同步代码块，按照链表或红黑树结构，链表插在尾部，红黑树
  则按照其结构插入。
- 如果插入后链表长度>8，则调用TreeBin方法转换为红黑树
- 添加成功后addCount()，增加size长度，看是否要扩容
- 如果扩容后，重新hash，有的红黑树可能就需要变为链表 <=6 时。

1.7 ReentrantLock的tryLock()+自旋+阻塞
1.8 CAS一次，失败Synchronized



扩容
- 扩容刚开始调用的是helpTransfer方法，通过多线程协助扩容，提高效率，transfer真正扩容。
- 有个 sizeCtl 的属性值，-1代表当前一个线程在操作，-N代表N个线程在操作，通过CAS协助扩容，而不是阻塞等待扩容 
  完成。



get（）
- 计算hash值，定位到具体Node位置
- 判断首节点，同返回，不同则向下遍历
- 如果遇到正在扩容，就通过正在扩容的Node节点的find方法比较，匹配就返回。


size()
- 1.8时的size在扩容和put的时候已经计算好了，直接返回就行。
- 1.7则是在size通过比较两次，失败加锁统计的方式。


```





#### 1.7 和 1.8 ConcurrentHashMap比较

```
- 1.7通过CAS+Segment+ReentrantLock
- 1.8则开始和HashMap的结构相同，使用CAS+synchronized+HashEntry+红黑树

- 1.8降低了锁粒度，1.7粒度为Segment，1.8是当个HashEntry节点

- 关于Synchronized代替了ReentrantLock :
  - Synchronized在1.6之后性能并不比ReentrantLock差，并且synchronized是基于JVM的不是API，有提升空间
  - 粒度变小，本来ReentrantLock对segment这种粒度较大的情况，Condition等待唤醒比较灵活，现在没了。
  - API的，自然带来更多开销，创建对象什么的。虽然不是瓶颈，但是有的。
```











#### TreeMap

https://www.cnblogs.com/chenssy/p/3746600.html

- 也是一个map，支持key-value;
- 底层根据红黑树实现，get \ put \ containKey \ remove 等节点操作时间复杂度都是 O(logn)
- 并不是线程安全的，支持 fast-fail
- 底层key是有序的，可以返回有序的keys集合，因为红黑树。
- 默认key是根据自然排序（数从小到大，字母从a到z），也可以在初始化的时候提供一个compare，即提供一个排序规则。

![2014051700002](https://images0.cnblogs.com/blog/381060/201405/222222278405105.png)







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
		PS：这样规定后，最长的子树长度不会超过最短的一倍，自平衡。
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



##  5、关于null值

​	**一定注意，ConcurrentHashMap的key\value都不能为null !!!**

| Hashtable         | 不允许为 null | 不允许为 null | Dictionary  | 线程安全               |
| ----------------- | ------------- | ------------- | ----------- | ---------------------- |
| ConcurrentHashMap | 不允许为 null | 不允许为 null | AbstractMap | 锁分段技术（JDK8:CAS） |
| TreeMap           | 不允许为 null | 允许为 null   | AbstractMap | 线程不安全             |
| HashMap           | 允许为 null   | 允许为 null   | AbstractMap | 线程不安全             |

HashMap 的 key-value 都可以为null， 但是ConcurrentHashMap不行。







## 6、关于红黑树和AVL树

https://www.jianshu.com/p/37436ed14cc6

​	为什么HashMap用红黑而不是AVL树？

- 红黑树的查询性能略微逊色于AVL树，因为其比AVL树会稍微不平衡最多一层，也就是说红黑树的查询性能只比相同内容的AVL树最多多一次比较，但是，红黑树在插入和删除上优于AVL树，AVL树每次插入删除会进行大量的平衡度计算，而红黑树为了维持红黑性质所做的红黑变换和旋转的开销，相较于AVL树为了维持平衡的开销要小得多。
- 红黑树不追求"完全平衡"，即不像AVL那样要求节点的 `|balFact| <= 1`，它只要求部分达到平衡，但是提出了为节点增加颜色，红黑是用非严格的平衡来换取增删节点时候旋转次数的降低，任何不平衡都会在三次旋转之内解决，而AVL是严格平衡树，因此在增加或者删除节点的时候，根据不同情况，旋转的次数比红黑树要多。
- 就插入节点导致树失衡的情况，AVL和RB-Tree都是最多两次树旋转来实现复衡rebalance，旋转的量级是O(1)
   删除节点导致失衡，AVL需要维护从被删除节点到根节点root这条路径上所有节点的平衡，旋转的量级为O(logN)，而RB-Tree最多只需要旋转3次实现复衡，只需O(1)，所以说RB-Tree删除节点的rebalance的效率更高，开销更小！
- AVL的结构相较于RB-Tree更为平衡，插入和删除引起失衡，如2所述，RB-Tree复衡效率更高；当然，由于AVL高度平衡，因此AVL的Search效率更高啦。
- 

