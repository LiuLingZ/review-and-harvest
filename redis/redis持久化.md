https://www.cnblogs.com/itdragon/p/7906481.html

# redis 持久化

​	即将内存中的数据保存到磁盘中，在redis的体现就是 AOF （Append Only File）和 RDB ( Redis Database)

​	**通过config get dir 获得持久化文件保存的位置，可在.conf中配置 dir 位置**



## RDB（Redis Database）

### 0、原理

​	进行rdb持久化时，redis的原进程fork一份与原进程一模一样的子进程进行持久化操作，子进程先将数据写到一个临时文件，待持久化结束，再替代上一个持久化的文件。这期间原进程不进行任何IO操作，为的就是高效地进行持久化操作。对于大数据量的恢复以及对数据完整性不是非常敏感，RDB比AOF适用，但是RDB的缺点也是，可能丢失最后一次持久化数据（持久化中断失败。）

###1、使用save，bgsave指令立刻保存到dump.rdb中

- save 是立刻保存，不管其他，全部阻塞
- bgsave 后台异步进行快照操作，还可响应客户端请求
- flushall 也会立刻保存，但是清空，没意义

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
- rdbcompression yes 保存后的rdb文件进行LZF压缩算法压缩，会略微消耗cpu，但是如果不压缩日志文件很大
- rdbchecksum yes 压缩后进行数据校验，会占用10％性能，默认开
- dbfilename dump.rdb 压缩文件名
- dir ./ 保存rdb文件的位置

### 5、优势

- 适合大规模的数据恢复
- 对数据的完整性和一致性要求不高（因为是隔一段时间保存，有可能突然中断，一段时间内的数据消失）

### 6、劣势

- 保存的完整性和一致性不高
- 直接的冷拷贝，导致2倍的备份文件大小



## AOF(Append Only File)

​	默认不开启，修改 appendonly yes 打开。为了弥补RDB最后一次持久化可能出现的失败导致较多的数据丢失的缺点。

### 1、 指定更新条件

```redis
# appendfsync always
appendfsync everysec
# appendfsync no
```

always :同步持久化，每次数据变化都会立刻写入磁盘，性能较差但是数据完整性好。

### 2、配置 重写 触发机制

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

当AOF文件大小是上次rewrite后大小的一倍（100％）且文件大于64M时触发。一般都设置为3G，64M太小了。



### 3、AOF重写机制

​	AOF文件是通过记录修改来进行持久化，如此日积月累占用内存大。所以redis提供重写机制，当.aof文件到达一个阈值，redis就会对aof文件的内容压缩。

​	重写原理：fork一个新的进程，读取内存的内容（不是读取旧文件的，太大），重写写到一个临时文件，最后替换掉原来的旧aof文件。

​	触发机制：当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发。这里的“一倍”和“64M” 可以通过配置文件修改。



### 4、AOF优缺点

优点：最多仅丢失1s的数据，数据完整性和一致性比RDB好

缺点：AOF记录的数据多，文件也会越来越大，数据恢复慢





## 总结

1. Redis 默认开启RDB持久化方式，在指定的时间间隔内，执行指定次数的写操作，则将内存中的数据写入到磁盘中。
2. RDB 持久化适合大规模的数据恢复但它的数据一致性和完整性较差。
3. Redis 需要手动开启AOF持久化方式，默认是每秒将写操作日志追加到AOF文件中。
4. AOF 的数据完整性比RDB高，但记录内容多了，会影响数据恢复的效率。
5. Redis 针对 AOF文件大的问题，提供重写的瘦身机制。
6. 若只打算用Redis 做缓存，可以关闭持久化。
7. 若打算使用Redis 的持久化。建议RDB和AOF都开启。其实RDB更适合做数据的备份，留一后手。AOF出问题了，还有RDB。