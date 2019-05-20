参考：https://www.cnblogs.com/gl-developer/p/6170423.html（原文有部分细节错误）

# mysql主备（master-slave）

## 一、模型

​	多台业务服务器同时写master，读slave ； 

​	master 通过 异步 sync slave ；

![img](https://images2015.cnblogs.com/blog/1043616/201612/1043616-20161213151157558-1150305350.jpg)



## 二、一主一从同步机制

```mysql
	mysql的主备是通过 二进制日志文件 实现的。master将修改通过 “事件” 写入日志文件 ；其他slave通过一个 I/O线程去监控 master 的日志文件的变化，当发生变化，就去读取，写到自己的 中继日志 中，然后slave的SQL线程就会把相关的“事件”执行，即同步到自己数据库中。实现主从复制。
```



## 三、实现步骤

```
主服务器 ：
    - 开启二进制日志
    - 配置唯一的server-id（必须唯一）
    - 获取master二进制日志文件名及位置
    - 创建一个用于slave和master通信的用户账号
    
从服务器 ：
	- 配置唯一的server-id（必须唯一）
	- 使用master分配的用户账号读取master二进制日志
	- 启动slave服务
```



## 四、实践（一主一从）

master主机  192.168.88.100  ； slave主机  192.168.88.101



### 主服务器

#### 1、修改mysql配置

​	找到 mysql的配置文件，（linux为my.cnf , mysql为my.ini ； linux一般在 /etc/my.cnf）在[mysqld]添加如下内容：

```mysql
[mysqld]
log-bin=mysql-bin #开启二进制日志
server-id=1 #设置server-id，唯一
```

#### 2、重启mysql，创建用于同步的账号

```mysql
进入mysql的shell会话：mysql -uroot -p
创建用户并授权： 用户：sla1 ； 密码：sla1

mysql> CREATE USER 'slal'@'192.168.88.100' IDENTIFIED BY 'sla1';#创建用户,在主服务器上，作为凭据
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.88.100';#分配权限，允许所有权限
mysql>flush privileges;   #刷新权限
```



#### 3、查看master状态，记录二进制名（File）和位置（73）

```mysql
mysql > SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000003 | 73       | test         | manual,mysql     |
+------------------+----------+--------------+------------------+
```



### 从服务器

#### 1、修改mysql配置

​	在my.cnf修改，添加server-id

```mysql
[mysqld]
server-id=2 #设置server-id，必须唯一
```

#### 2、重启mysql，执行同步sql语句

​	**需要主服务主机名，登录凭据（master创建的同步账号），二进制文件的名称和位置**

```mysql
mysql> CHANGE MASTER TO
    ->     MASTER_HOST='192.168.88.100',
    ->     MASTER_USER='sla1',
    ->     MASTER_PASSWORD='sla1',
    ->     MASTER_LOG_FILE='mysql-bin.000003',
    ->     MASTER_LOG_POS=73;
```

#### 3、启动slave同步进程

```mysql
mysql>start slave ;
```

#### 4、查看slave状态

```mysql
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.88.100
                  Master_User: sla1
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000013
          Read_Master_Log_Pos: 11662
               Relay_Log_File: mysqld-relay-bin.000022
                Relay_Log_Pos: 11765
        Relay_Master_Log_File: mysql-bin.000013
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
          ···
```

当Slave_IO_Running和Slave_SQL_Running都为YES的时候就表示主从同步设置成功了。





## 五、测试和额外的配置

接下来就可以进行一些验证了，比如在主master数据库的test数据库的一张表中插入一条数据，在slave的test库的相同数据表中查看是否有新增的数据即可验证主从复制功能是否有效，还可以关闭slave（mysql>stop slave;）,然后再修改master，看slave是否也相应修改（停止slave后，master的修改不会同步到slave），就可以完成主从复制功能的验证了。

还可以用到的其他相关参数：

master开启二进制日志后默认记录所有库所有表的操作，可以通过配置来指定只记录指定的数据库甚至指定的表的操作，具体在mysql配置文件的[mysqld]可添加修改如下选项：

```mysql
# 不同步哪些数据库  
binlog-ignore-db = mysql  
binlog-ignore-db = test  
binlog-ignore-db = information_schema  
  
# 只同步哪些数据库，除此之外，其他不同步  
binlog-do-db = game  
```







