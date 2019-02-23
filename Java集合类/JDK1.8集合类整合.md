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