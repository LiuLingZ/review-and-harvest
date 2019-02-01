# 堆和Minor\Full GC



## 一、堆结构布局

1. 分新生代 、 老年代。大小 ： 1:2

   新生代又分 ： Eden 、From Survivor、 To Survivor = 8:1:1

   ![1](C:/Users/lingz/AppData/Local/YNote/data/m13229401809@163.com/ee2d7e664a114e0aa905e615ef7e13bb/921034534396.png)

2. Java每次都使用一个Eden 和 一个Survivor区进行对象服务。这和GC算法有关



## 二、Minor\Full GC

​	根据新生代和老年代步不同分区，堆的GC主要分两种：Minor GC 和 Full GC

####(一)、Minor GC

1. 发生在新生代。采用**复制算法**

2. 新生代是大部分Java对象诞生的地方，但是活的不长久，Minor GC发生过程：

   ​	对象在Eden和一个Survivor区中的某个诞生后，经过一次Minor GC，若对象还存活，则被放到另外一个Survivor中。复制算法，将存活的对象复制到Survivor中，然后清理Eden和另外一个Servivor 。 并将此对象年龄 +1 。

   ​	当到达某个年龄（默认15，可以通过 -XX：MaxTenuringThreashold设定）。这些对象就会成为老年代。

   ​	有例外，一些较大的对象（如需要占用较多连续空间）的集合直接进入老年代。

   ​

#### (二)、Full GC

1. 发生在老年代，采用 **标记-清除算法**

2. 老年代的对象并不会轻易死去，所有Full GC没有Minor GC频繁。并且Full-GC时间更长。

   并且标记-清除算法容易产生大量碎片空间，可能在下一次分配连续空间的大对象会提前出发一次Full GC 。

   ​

















