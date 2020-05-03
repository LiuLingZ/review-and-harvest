



<https://zsr.github.io/2017/07/03/redis-zset%E5%86%85%E9%83%A8%E5%AE%9E%E7%8E%B0/>

# 跳跃链表



zset 的内部实现是一个 dict 字典 (dict { dictht }) +  一个跳跃列表 (skiplist)。

字典是 member映射到 score 的

跳跃链表是 score 映射到 member 的 （这里是redis内部优化过的 skiplist , 非经典 skiplist）





zset 元素少时，通过 ziplist 存储

 - 元素个数小于 128个
 - member长度 小于 64 



## skiplist

### 1 、经典 skiplist 

​	严格维护 ： 高一级的层级层数 ： 低一级的层级层数 = 1 ： 2

​	查找 ：logN

​	插入删除： 需要大量修改层级，O(N) ，**是redis改进的一个地方**

​	第一层 是 单向链表 

​	只存 score 。无法映射到 member

![两层跳跃指针](http://zhangtielei.com/assets/photos_redis/skiplist/skip2node_level3_linked_list.png)



### 2、redis的skiplist 

​	不严格维护 1:2 , 而是采用 随机的 层级 ， 晋升到高一级的层数的概率默认是 1 / 4  ，最高 32 层

​	查找 、插入、删除： **采用的是随机赋层，所以不需要可以维护，插入赋层即可 ， O（logN）**

​	第一层 是 双向链表，用于逆序获取score范围 

​	存 key : score ， value : member  ， 可以从 score 映射到 member



![skiplist插入形成过程](http://zhangtielei.com/assets/photos_redis/skiplist/skiplist_insertions.png)







## 比较 二叉平衡树 AVL

存的平均指针少。AVL 是 每个节点 2  ， skiplist 是 1.33 ( 1 / 1-P  ;  P = 1/4)

插入删除 维护难度低，AVL需要O(N)  , 这也是 为什么用 红黑树而不是 AVL的原因

查找简单，score直接定位最小的，范围查找。 而 AVL需要定位到最小，然后通过中序遍历找到比他大的第一个位置，才可。











