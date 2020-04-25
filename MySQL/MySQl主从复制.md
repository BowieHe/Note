### 可用架构方案：

- 负载均衡：有一定的高可用性
  LVS， Nginx
- 主备系统：有高可用性，但是需要切换，是单活的架构
  KA， MHA， MMM
- 真正高可用（多活系统）：
  NDB， Cluster， Oracle， RAC， Sysbase cluster， InnoDB Cluster，PXC， MGC

## 主从复制简介：

```css
1.1. 基于二进制日志复制的
1.2. 主库的修改操作会记录二进制日志
1.3. 从库会请求新的二进制日志并回放,最终达到主从数据同步
1.4. 主从复制核心功能:
辅助备份,处理物理损坏                   
扩展新型的架构:高可用,高性能,分布式架构等
1.5. 使用三个线程实现，一个在主服务器上，两个在从服务器上
```

## 主从复制搭建

1. 清理主库数据
   `rm -rf /data/3307/data/*`
2. 重新初始化主库
   `mysqld --initialize-insecure --user=mysql --basedir=/app/mysql --datadir=/data/3307/data`
3. 修改my.cnf，开启二进制日志功能
   修改语句：`log_bin=/data/3307/data/mysql-bin`
4. 启动所有节点
   `systemctl start mysqld3307`和3308，3309
5. 主库创建复制用户
   `mysql -S /data/3307/mysql.sock `#创建
   `grant replication slave on *.* to repl@'10.0.0.%' identified by '123';`
   `select user,host from mysql.user;`
6. 备份主库并恢复到从库

```shell
[root@db01 3307]# mysqldump -S /data/3307/mysql.sock -A --master-data=2 --single-transaction  -R --triggers >/backup/full.sql
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=653;
[root@db01 3307]# mysql -S /data/3308/mysql.sock
db01 [(none)]>source /backup/full.sql
```

7. 告知从库关键复制信息
8. 开启主从专用线程
   `start slave`
9. 检查复制状态
   `slave status \G`

## 主从复制原理

1. 1.change master to 时，ip pot user password binlog position写入到master.info进行记录
2. start slave 时，从库会启动IO线程和SQL线程
  3.IO_T，读取master.info信息，获取主库信息连接主库
3. 主库会生成一个准备binlog DUMP线程，来响应从库
4. IO_T根据master.info记录的binlog文件名和position号，请求主库DUMP最新日志
5. DUMP线程检查主库的binlog日志，如果有新的，TP(传送)给从从库的IO_T
6. IO_T将收到的日志存储到了TCP/IP 缓存，立即返回ACK给主库 ，主库工作完成
  8.IO_T将缓存中的数据，存储到relay-log日志文件,更新master.info文件binlog 文件名和postion，IO_T工作完成
  9.SQL_T读取relay-log.info文件，获取到上次执行到的relay-log的位置，作为起点，回放relay-log
  10.SQL_T回放完成之后，会更新relay-log.info文件。
7. relay-log会有自动清理的功能。
   细节：
   1.主库一旦有新的日志生成，会发送“信号”给binlog dump ，IO线程再请求

## 主从故障监控、分析、处理

主库：`show full processlist`

从库：`show slave status \G`

## IO故障处理：

### 连接主库错误：

原因可能有：密码错误，用户错误，skip_nane_resolve,地址错误，端口错误

解决办法：

1. stop slave
2. reset slave all
3. change master to
4. start slave

### 主库连接数上线或者主库太繁忙

```dart
1.  show slave  staus \G 
Last_IO_Errno: 1040
Last_IO_Error: error reconnecting to master 'repl@10.0.0.51:3307' - retry-time: 10  retries: 7
处理思路:
拿复制用户,手工连接一下

[root@db01 ~]# mysql -urepl -p123 -h 10.0.0.51 -P 3307 
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1040 (HY000): Too many connections
处理方法:
db01 [(none)]>set global max_connections=300;

(3) 防火墙,网络不通
```

### 请求二进制日志

```csharp
主库缺失日志
从库方面,二进制日志位置点不对
Last_IO_Error: Got fatal error 1236 from master when reading data from binary log: 'could not find next log; the first event 'mysql-bin.000001' at 154, the last event read from '/data/3307/data/mysql-bin.000002' at 154, the last byte read from '/data/3307/data/mysql-bin.000002' at 154.'
  
解决：重新构建主从
在主从环境中，禁止主库中reset master；可以选择expire进行定期清理出库二进制日志
```

## SQL线程故障

功能：

1. 读写relay-log.info 
2. relay-log损坏,断节,找不到
3. 接收到的SQL无法执行

导致SQL线程故障原因分析：**都是由于从库发生了写入操作**

1. 版本差异，参数设定不同。比如数据类型差异，SQL_MODE影响
2. 要创建的数据库对象已经存在
3. 要删除或修改的对象不存在
4. DML语句不符合表定义及约束时

解决方法：

```
方法一：
stop slave; 
set global sql_slave_skip_counter = 1;
#将同步指针向下移动一个，如果多次不同步，可以重复操作。
start slave;
方法二：
/etc/my.cnf
slave-skip-errors = 1032,1062,1007
常见错误代码:
1007:对象已存在
1032:无法执行DML
1062:主键冲突,或约束冲突

但是，以上操作有时是有风险的，最安全的做法就是重新构建主从。把握一个原则,一切以主库为主.

或者设置从库只读
加上中间件，实现读写分离
```

## 主从延时监控和原因：

主库做了修改操作，从库需要比较长的时间才能更新上

### 外在原因：

网络， 主从硬件差异较大，版本差异，参数因素

### 主库延时

1. 二进制日志写入不及时
2. CR的主从复制中，binlog_dump线程，事件为单元，串行传送二进制日志

主库并发事务量较大，主库可以并行，但是数据传送时串行
主库发生了较大事务，由于是串行传送，会产生阻塞后续事务的情况

解决办法：

1. 开启GTID，实现GC（group commit）机制，并行传输日志给从库IO
2. 大事务拆分成多个小事务，减少主从延时

### 从库延时

SQL线程导致主从延时
在CR复制情况下，从库默认情况下只有一个SQL，只能串行回放事务SQL

解决方案：

1. 5.6版本开启GTID之后，加入了SQL多线程特性，但是只针对不同的库下的事务进行并发回放
2. 5.7版本开启GTID之后，提供了基于逻辑时钟(binary_clock)，binlog加入seq_no机制，真正实现了基于事务级别的并发回放，为MTS(multi_thread slave)技术
3. 大事务拆分成多个小事务