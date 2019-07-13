https://www.cnblogs.com/itdragon/p/7906481.html



是什么 → 什么时候触发 → 过程 → 优点/适用时机 → 

# redis 持久化

​	即将内存中的数据保存到磁盘中，在redis的体现就是 AOF （Append Only File）和 RDB ( Redis Database)

​	**通过config get dir 获得持久化文件保存的位置，可在.conf中配置 dir 位置**



## RDB（Redis Database）



###1、使用save，bgsave指令立刻保存到dump.rdb中

- save 是立刻保存，不管其他，全部阻塞，不建议使用
- bgsave 后台异步进行快照操作，还可响应客户端请求，后台fork一个进程，这个过程是阻塞的
- flushall 也会立刻保存，但是清空，没意义
- shutdown也会触发bgsave

### 2、通过dump.rdb恢复

- 将正常时拷贝dump.rdb文件重命名 dump1.rdb保存
- 恢复时，先关闭redis，然后删除原来为dump.rdb，然后将 dump1.rdb 重命名为dump.rdb 到原来位置
- redis启动时，自动去dump.rdb获取，恢复成功

### 3、建议的dump保存策略（save参数）

1分钟改了 1w 次		60	10000

5分钟改了10次		300		10

15分钟改了1次		900		1

### 4、一些参数

- stop-writes-on-bgsave-error yes 保存过程发生异常停止写操作
- **rdbcompression yes 保存后的rdb文件进行LZF压缩算法压缩，会略微消耗cpu，但是如果不压缩日志文件很大**
- rdbchecksum yes 压缩后进行数据校验，会占用10％性能，默认开
- dbfilename dump.rdb 压缩文件名
- dir ./ 保存rdb文件的位置

### 5、优势

- 适合大规模的数据恢复，建议隔一段时间就bgsave
- 恢复速度比AOF快

### 6、劣势

- 没办法做到实时持久化/秒级持久化，存的也只是fork前的内容，势必会丢失数据
- RDB采用特定二进制保存，Redis演化了多个格式RDB，老版本Redis无法兼容新版本Redis的RDB

### 7、过程

```
bgsave
↓
父进程判断是否正在执行RDB备份， 是则直接返回
↓
否，则fork一个子进程，去生成RDB文件。父进程就去接着响应 。fork过程会阻塞父进程。
↓
RDB文件根据当前父进程的内存生成临时快照文件，完成后对原有的dump.rdb进行原子替换。
↓
子进程完成后通知父进程，父进程再去记录一些RDB备份信息（时间、次数等）

```









## AOF(Append Only File)

​	默认不开启，修改 appendonly yes 打开。为了弥补RDB最后一次持久化可能出现的 失败导致较多的数据丢失的缺点。提供实时/秒级持久化的功能。

​	aof内容是文本格式，记录的是数据的写入修改操作。恢复时重新执行一遍，相比RDB比较耗时。

​	重启时有AOF先AOF。

### 0、过程

https://www.cnblogs.com/iamsach/p/8490387.html

```
AOF流程：文件写入(append) 、文件同步（sync）、文件重写（rewrite）、重新加载（load）

持久化过程：
append （注意！！！是append到 aof_buf 缓存文件中，因为每次写到AOF，是在磁盘上，效率低；并且缓冲区可控，可以在性能和安全上做权衡）
↓
sync(存到磁盘，根据所选策略不同，这一步的时机也不同)
↓
load（更新完成后，重新加载AOF）

根据appendfsync策略的不同，期间sync执行的时机也不同

关于 fsync :
	就是将 aof_buf 的内容写到 appendonly.aof 文件中，真正的持久化


---------------------详细：--------------------------
AOF缓冲区同步文件策略（配置appendfsync参数）说明：

　　always:命令写入aof_buf后调用系统fsync操作同步到AOF文件，fsync完成后线程返回

　　everysec:命令写入aof_buf后调用系统write操作，write完成后线程返回。fsync同步文件操作由专门线程每秒调用一次

　　no:命令写入aof_buf后调用系统write操作，不对AOF文件做fsync同步，同步硬盘操作由操作系统负责，通常同步周期最长30s

　　配置为always时，每次写入都要同步AOF文件，在一般的SATA硬盘上，Redis只能支持大约几百TPS写入，显然跟Redis高性能特性背道而驰，不建议配置。

 　　配置为no，由于操作系统每次同步AOF文件的周期不可控，而且会加大每次同步硬盘的数据量，虽然提升了性能，但数据安全性无法保证。

 　　配置为everysec，是建议的同步策略，也是默认配置，做到兼顾性能和数据安全性。理论上只有在系统突然宕机的情况下丢失1秒的数据。

(策略选择根据业务不同进行选择)

系统调用write和fsync说明：

　　write操作会触发延迟写（delayed write）机制，Linux在内核提供页缓冲区用来提高硬盘IO性能，write操作在写入系统缓冲区后直接返回，同步硬盘操作依赖于系统调度机制，例如：

缓冲区页空间写满或达到特定时间周期。同步文件之前，如果此时系统故障宕机，缓冲区内数据将丢失。

 　　fsync针对单个文件操作（比如AOF文件），做强制硬盘同步，fsync将阻塞直到写入硬盘完成后返回，保证了数据持久化。　　


```









### 1、 指定更新条件

```redis
# appendfsync always
appendfsync everysec
# appendfsync no

always : 每次append到aof_buf时，都调用fsync，同步到aof文件中
everysec ： append到 aof_buf中，每隔1s就调用一次 fsync .理论丢失1s数据，突然宕机的话。
no : 写入到aof_buf中后，调用系统的write,让系统去决定什么时候 同步到aof中，通常同步周期30s

write ： 触发延迟写 机制，将aof_buf的内容写到系统缓冲区中，当满了或者一段时间后，再写入aof文件中。

```

always :同步持久化，每次数据变化都会立刻写入磁盘，性能较差但是数据完整性好。

no：同步周期不可控，虽提升性能，但安全性也降低了。



### 2、配置 重写 触发机制

auto-aof-rewrite-percentage 100   # 是上一次的一倍
auto-aof-rewrite-min-size 64mb	# 文件大小超过64M

当AOF文件大小是上次rewrite后大小的一倍（100％）且文件大于64M时触发。一般都设置为3G，64M太小了。



### 3、AOF重写机制

​	AOF文件是通过记录修改来进行持久化，如此日积月累占用内存大。所以redis提供重写机制，当.aof文件到达一个阈值，redis就会对aof文件的内容压缩。

​	

```
触发机制：
	当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发。这里的“一倍”和“64M” 可以通过配置文件修改。

手动触发：bgrewriteaof
自动触发：根据auto-aof-rewrite-xx参数

过程：
	bgrewriteaof
		↓
	fork一个父进程，此过程阻塞，但速度快
		↓
	fork后，父进程继续处理请求，但是此时，父进程分两路保存：
		1、继续写到 aof_buf中，并根据策略，fsync到 旧 的aof文件中
		2、写到一个 aof_rewrite_buf中，这个缓冲区，是在新的aof取代旧aof时，父进程还需要响应，先写到这
		↓
	当子进程重写完后通知父进程，替换旧的aof，按后再和aof_rewritwe_buf整合，得到最新的
		↓
	重启，重新load



父接收写 aof_buf同步到 旧aof，子rewrite
子告诉父，用新aof取代旧aof，这个过程父还在响应
取代后，整合 aof_rewrite_buf 和 新aof，形成最终版

```



### 4、AOF优缺点

优点：最多仅丢失1s的数据，数据完整性和一致性比RDB好

缺点：AOF记录的数据多，文件也会越来越大，数据恢复慢



### 5、文件校验

加载坏的aof 会拒绝启动，可以用 redis-check-aof --fix 修复，RDB也可以 redis-check-rdb --fix 



## 总结

1. Redis 默认开启RDB持久化方式，在指定的时间间隔内，执行指定次数的写操作，则将内存中的数据写入到磁盘中。
2. RDB 持久化适合大规模的数据恢复但它的数据一致性和完整性较差。
3. Redis 需要手动开启AOF持久化方式，默认是每秒将写操作日志追加到AOF文件中。
4. AOF 的数据完整性比RDB高，但记录内容多了，会影响数据恢复的效率。
5. Redis 针对 AOF文件大的问题，提供重写的瘦身机制。
6. 若只打算用Redis 做缓存，可以关闭持久化。
7. 若打算使用Redis 的持久化。建议RDB和AOF都开启。其实RDB更适合做数据的备份，留一后手。AOF出问题了，还有RDB。