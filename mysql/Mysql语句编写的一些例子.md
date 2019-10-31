全部来自牛客网





## 1、获取某列（col）倒数第三的人的信息

```sql
select * from table where
col = 
(select distinct(col) from table order by col desc limit 2,1) ;

-- desc 是倒序，distinct为了去除相同项目的值，保证倒数的唯一
-- limit m,n 从m+1开始取n个 ，2，1就是从3开始取1个
-- limit n 从0开始取n个
```



**null 不参与count计算，distinct会将所有null归为一个。**



## 2、一些聚合函数

```sql
IFNULL(v1,v2); --其中：如果 v1 不为 NULL，则 IFNULL 函数返回 v1; 否则返回 v2 的结果。

----------------------- 下面的都是在做判断聚合函数，为的就是符合业务逻辑的操作------------------------

sum(if(type=4,1,0)) --如果type=4,则if返回1，sun就可以+1，否则返回0，不加1 。这里就是累加的作用

count(distinct if(type =1, user_id, null)) --if的结果就是，如果type=1，就返回user_id,否则返回null。而distinct 如果是相同userid，就去重，如果是null， 就不理会。






```



## 3、分区间显示统计时间

```sql

------ PS： 一定要用 group by 区间名，否则只会查出第一个数据所在的区间的总数。

SELECT
	ELT(
		INTERVAL (cost_time,0, 100, 200, 500, 1000),
		'100毫秒以下',
		'100到200毫秒',
		'200到500毫秒',
		'500到1000毫秒',
		'一千毫秒以上的'
	) AS TIME,COUNT(*)
FROM
	ad_record_report
WHERE
	create_time >= '2019-10-01' AND create_time <= '2019-10-16' AND ad_slot_id = 111 GROUP BY TIME;
	
-----------------------------------------------------------
SELECT
	ELT(INTERVAL (ad_id,0,1300,1400,1500),'1300以内','1300到1400','1400到1500','1500以上') AS 区间,COUNT(*)
FROM ad_info GROUP BY 区间
```

