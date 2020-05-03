# limit 



## 一、分析



```sql

explain SELECT * FROM message ORDER BY id DESC LIMIT 10000, 20 \G;
***************** 1. row **************
id: 1
select_type: SIMPLE
table: message
type: index
possible_keys: NULL
key: PRIMARY
key_len: 4
ref: NULL
rows: 10020
Extra:
1 row in set (0.00 sec)

--从上可看到，where\order by xxx limit x,y 是 扫面x+y条记录，然后抛弃x之前的。
--这么看来，就是一次性要加载x+y条（10020）到内存，自然慢，也可能OOM

优化：
select * from message where id > (select id from message where message_id > 10000 limit 10000,1) limit 20

-- 首先，最好不要 * ，越多的列，加载越多。
-- 其次，建议多加索引，如（id，message_id）这个复合索引。
-- 通过子查询和索引，首先定位到要找的那条记录的位置(limit 10000,1)，再在索引树定位 id > xxx ,速度就快了。



```





