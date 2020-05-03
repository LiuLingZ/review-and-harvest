

# 快速链表



是 list 的底层原理



原先的 list  在元素少的时候使用 ziplist ， 在元素多时使用双向链表 linkedList 。

然而 linkedList 每个节点都要存储 前一个指针 和 下一个指针，浪费空间， 就延伸出了 ziplist + linkedList 的 quickList

![img](https://user-gold-cdn.xitu.io/2018/7/29/164e3b0b953f2fc7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





## quicklist

​	快速链表的每个节点都是 ziplist  ，然后通过前后指针指向前一个 或 下一个快速链表节点（ziplist）

```c
struct quicklistNode {
    quicklistNode* prev;
    quicklistNode* next;
    ziplist* zl; // 指向压缩列表 （每个节点本身就是一个压缩链表）
    int32 size; // ziplist 的字节总数
    int16 count; // ziplist 中的元素数量
    int2 encoding; // 存储形式 2bit，原生字节数组还是 LZF 压缩存储
    ...
}
struct quicklist {
    quicklistNode* head;
    quicklistNode* tail;
    long count; // 元素总数
    int nodes; // ziplist 节点的个数
    int compressDepth; // LZF 算法压缩深度
    ...
}


注：
quicklist 默认每个 压缩链表 长度不超过 8K，超出就新起一个 快速链表节点(ziplist) 。

```



## 压缩深度

​	压缩深度指从首尾第几个元素开始进行压缩。越深，越靠中间。

​	压缩深度为1时，首尾第一个节点不压缩，其他中间的全部压缩 。 为2则第一、二个节点不压缩，其他压缩。

​	**首尾节点不压缩主要是优化 插入删除首尾节点时的高效性。**





