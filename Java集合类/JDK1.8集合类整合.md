#总概







## 1、ArrayList

https://www.cnblogs.com/leesf456/p/5308358.html

- 核心是transient Object[] elementData ; 实例，真正存值。transient代表持久化时会被忽略。


- 如果创建一个ArrayList数组，没有指定长度，则添加元素时，或扩容到长度为10.

  如果创建时有指定长度，不会进行扩容。

- 除了未定义初始化的长度的第一次扩容（扩为10），其他每次扩容长度为1.5倍。

- 每次操作的第一件事，就是判断索引是否合理.

- ArrayList中是可以放null元素的。可以查找到null元素。

ArrayList有其特殊的应用场景，与LinkedList相对应。其优点是随机读取，缺点是插入元素时需要移动大量元素，效率不太高。



##2、LinkedList

https://www.cnblogs.com/leesf456/p/5308843.html

- 实现了Deque接口，这意味是一个双向链表。可从头和尾开始遍历。确实，每个方法通过索引确认节点位置的时候，也是判断索引靠前还是靠后，达到最多一半遍历即可访问到。

- 链表的每个节点类Node含三个实例，Node next \ Node prev \E item(泛型)

  链表本身含三个实例： int size \ Node first \ Node last ，全部都是transient修饰

- 需要重视 addAll(int index ,Collection《？extend E》 c) ， 他是在index+1前面的位置添加

  并且，先将 c.toArray，其作用主要是在并发修改的时候，一个线程占用c的时间短，并且列表访问快，提高并发效率。

  ​



## 3、HashMap

### 一、JDK1.7

#### HashMap

https://mp.weixin.qq.com/s/KCxKKeJ28Yev5xsv4WzRog

https://mp.weixin.qq.com/s/dzNq50zBQ4iDrOAhM4a70A

- 底层是数组，每个数组元素是一个Entry对象，有key\value\next三个实例，同时也是单向链表的头结点。
- 链表初始化长默认是16，并且长度必须是2的n次方，每次扩容扩容两倍长
- 扩容时机： 实际长度 ≥ 负载因子 * 总长 。 负载因子默认0.75，初始化时可修改。
- 扩容是就会设计到 ReSize 和 ReHash过程。前者是扩容，后者是将元素从旧数组复制到新数组中。
- Resize 是先创建两倍长的数组，再从 旧数组Rehash到新的对应位置。
- HashMap在多线程是不安全的，并发插入元素后可能出现带环的链表，在下次访问时出现死循环。



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

  ​

  ​

  ​

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





####ConcurrentHashMap

http://www.codeceo.com/article/java-hashmap-concurrenthashmap.html

- 放弃了原来Segment的概念，改用CAS+synchronized
- 原来的HashEntry变为Node，但是作用相同，其中value 和 next都是volatile修饰保证可见性。
- put（）：
  1. 根据key计算hashcode
  2. 判断是否需要初始化
  3. 通过key定位，为空、根据链表或红黑树的规则，通过CAS尝试，失败则自旋，到达一定次数就直接加synchronized
  4. 判断是否需要变为红黑树
  5. 判断是否需要扩容



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







