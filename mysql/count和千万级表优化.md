https://mp.weixin.qq.com/s?__biz=MzA5ODM5MDU3MA==&mid=2650864283&idx=1&sn=9d7ab5e80e61ca9dbcc8f15ef337e15b&chksm=8b661ddebc1194c840bd172fd314eff17a6e3611b6177a7aac7978ddf9dfd2f4153fe661dfa6&scene=0#rd

https://mp.weixin.qq.com/s/4atIDmiucqvkw0b2_u8itA



# 一、count系列

```
注意：

count(*) : 有null的也会被统计进去
count(id) : id为null的不会被统计进去
count(1) : 有null的也会被统计。

一般来说，count(*)比count(1)更高效：
	如果count(1)是聚集索引,id,那肯定是count(1)快。但是差的很小的。 因为count(*),自动会优化指定到那一个字段。所以没必要去count(1)，用count(*)，sql会帮你完成优化的 因此：count(1)和count(*)基本没有差别！ 


count(*) 和 count(1)和count(列名)区别  

执行效果上：  
count(*)包括了所有的列，相当于行数，在统计结果的时候，不会忽略列值为NULL  
count(1)包括了忽略所有列，用1代表代码行，在统计结果的时候，不会忽略列值为NULL  
count(列名)只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是只空字符串或者0，而是表示null）的计数，即某个字段值为NULL时，不统计。

执行效率上：  
列名为主键，count(列名)会比count(1)快  
列名不为主键，count(1)会比count(列名)快  
如果表多个列并且没有主键，则 count（1） 的执行效率优于 count（*）  
如果有主键，则 select count（主键）的执行效率是最优的  
如果表只有一个字段，则 select count（*）最优。

（一）结果区别
① COUNT(1)、COUNT(*)、COUNT(ROWID)、COUNT(常量)、COUNT(主键)、COUNT(非空列)这几种方式统计的行数是表中所有存在的行的总数，包括值为NULL的行和非空行。所以，这几种方式的执行结果相同。这里的常量可以为数字或字符串，例如，COUNT(2)、COUNT(333)、COUNT('x')、COUNT('xiaomaimiao')。需要注意的是：这里的COUNT(1)中的“1”并不表示表中的第一列，它其实是一个表达式，可以换成任意数字或字符或表达式。
② COUNT(允许为空列) 这种方式统计的行数不会包括字段值为NULL的行。
③ COUNT(DISTINCT 列名) 得到的结果是除去值为NULL和重复数据后的结果。
④ “SELECT COUNT(''),COUNT(NULL) FROM T_COUNT_LHR;”返回0行。

（二）效率、索引
① 如果存在主键或非空列上的索引，那么COUNT(1)、COUNT(*)、COUNT(ROWID)、COUNT(常量)、COUNT(主键)、COUNT(非空列)会首先选择主键上的索引快速全扫描（INDEX FAST FULL SCAN）。若主键不存在则会选择非空列上的索引。若非空列上没有索引则肯定走全表扫描（TABLE ACCESS FULL）。其中，COUNT(ROWID)在走索引的时候比其它几种方式要慢。通过10053事件可以看到这几种方式除了COUNT(ROWID)之外，其它最终都会转换成COUNT(*)的方式来执行。
② 对于COUNT(COL1)来说，只要列字段上有索引则会选择索引快速全扫描（INDEX FAST FULL SCAN）。而对于“SELECT COL1”来说，除非列上有NOT NULL约束，否则执行计划会选择全表扫描。
③ COUNT(DISTINCT 列名) 若列上有索引，且有非空约束或在WHERE子句中使用IS NOT NULL，则会选择索引快速全扫描。其余情况选择全表扫描。



PS：count(列名) : 有null走全表搜索，无null或者where is not null 则走索引。
```









# 二、分表原因

​	数据量千万条过大，查询效果缓慢，并发修改又回锁表，导致效率更低。

​	如现在单表数据量是5000w，那么可以拆成10张500w的表。

​	那么拆分就两种情况，一种就是业务设计的时候就想要拆分了，这种简单，对应插入表就好。如果是业务扩展过程，就需要设计拆表了。



## （一）数据库架构

### 1、主从复制

​	一主多从，写主读从。

问题：主无法扩展，复制延迟，数据量上升锁表情况明显（都写一张表）。表变大。所有类型的请求都到一个数据库。



### 2、垂直分区（业务拆分）

​	根据业务拆分， 将不同业务的数据放到不同数据库中。客户数据放客户数据库，独立部署，商品数据放商品数据库，独立部署。好处是不同业务不干扰。

​	问题是：不同DB间需要连接关联、并不能根本上解决数据量上升，该大还是大。



### 3、水平分片（数据拆分）

​	将业务数据根据某些依据拆分开，比如user_id，分别对应不同的表。就类似集群的分表，放在不同的服务器上。要对应的数据就去对应的server上找。	



