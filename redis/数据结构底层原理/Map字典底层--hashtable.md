<https://juejin.im/book/5afc2e5f6fb9a07a9b362527/section/5b5bdbbd5188251ac22b5bf7>



# Hash字典底层



hmset myHash key1 value1 key2 value2 key3 value3....

hset myHash key1 value111

hget myHash key1

hgetall myhash



# 1、字典 dict



结构

```c
// dict就是字典结构，核心是包含了两个 dictht 数组
// dictht 就是类似hashmap，在redis称为 hashtable ,只不过他的数组存的是指向链表头元素的指针
struct dict{
    ···
    //内部包含两个hashtable，一次只用一个，另外一个当扩容时会使用，就是新旧替换这样
    dictht ht[2] ;
}

// dictht数组
struct dictht{
 	// 这个table就是二维的那个链表，hash碰撞的那个链表
    dictEntry** table ;
    // 一维数组的长度
    long size ;
    // hash表元素的个数
    long used ;
}

//二维的那个一个个节点
struct dictEntry {
    void* key;
    void* val;
    dictEntry* next; // 链接下一个 entry
}


```





## 2、渐进式rehash

​	这里提到的是字典的扩容

​	扩容是一个O（n）操作，考虑到性能，redis不会一次就执行完，而是采用渐进式，慢慢迁移。

​	迁移过程可以在 增删改过程中、 以及定时任务定时触发逐步完成。

​	原先知道 dict 结构存有两个 dictht ， 其实就是在扩容时，一个原来的，一个是扩容新的，从旧的到新的迁移。

​	扩容完了之后，把引用指向新的，旧的抛弃即可。

​	扩容为原来2倍。

​		





## 3、扩容与缩容



​	**普通扩容时机：元素个数等于数组长度时。**

​	

​	**强制扩容：**

​	当此时在bgsave持久化时，暂时不会进行扩容，防止频繁读写，但是当元素个数已经达到第一维数字长度的5倍时，就需要强制扩容了，太挤了。

​	而 缩容， 就是元素个数低于数组长度的10%，就会缩容，不考虑bgsave . 







## 4、关于Set

​	set和dict底层是一样的，仅仅是value为null而已。











