https://www.jianshu.com/p/8f4f58b4b8ab

# LinkedHashMap



## 一、结构

​	LinkedHashMap底层是由 HashMap 和 一个由 HashMap的节点组成的**双向链表** 组成。看图：

![img](https://upload-images.jianshu.io/upload_images/4843132-7abca1abd714341d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/739/format/webp)



## 二、应用

​	HashMap本身是插入无序的，如果希望插入的键值有序，那么可以用 LinkedHashMap , 他能做到保存两种顺序：

​	1、插入顺序

​	2、访问顺序

​	其中的构造方法有个：

```java
public LinkedHashMap() {
        // 调用HashMap的构造方法，其实就是初始化Entry[] table
        super();
        // 这里是指是否基于访问排序，默认为false
        accessOrder = false;
 }

/*也可以指定*/
public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
}
```

### accessOrder

```
- 默认是false，代表的是按插入顺序排序。
- 如果是 true , 代表按 访问顺序排序，最近被访问的元素会放在队列最后面。（这就可以作为设计，设计lru算法）
```

### put \ get \扩容

```
- LinkedHashMap 继承与 HashMap，思想和它几乎是一样的
- put也是在set Entry的时候构造链表。get会根据accessOrder是否调整Entry位置。并且根据链表的顺序遍历。
- 扩容：触发 0.75 * length > capacity ，重写了transfer（）,会将值传到新的数组里面。
```





## 三、LinkedHashMap实现LRU

```java
/*
	通过继承LinkedHashMap ， 并且在构造器设置 accessOrder为true ; 
	然后重写其 removeEldestEntry（）即可。
*/
public class LRU extends LinkedHashMap{
    public LRU(int size,float loadFactory,boolean accessOrder){
        super(size,loadFactory,accessOrder);
    }
    
    /**
    	当插入的数目大于6时，自动触发，删除队列首部的元素，队尾是最近刚被用的。
    */
    public boolean removeEldestEntry(Map.Entry eldest){
        if(size()>6){
            return true ;
        }
        return false ;
    }
}


```









