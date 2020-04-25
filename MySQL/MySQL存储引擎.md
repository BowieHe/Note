MySQL存储引擎相当于Linux的文件系统，但是比文件系统更强大

功能：

1. 数据读写
2. 数据安全和一致性
3. 提高性能
4. 热备份
5. 自动故障恢复
6. 高可用方面支持等等

# 存储引擎种类

```shell
InnoDB
MyISAM
MEMORY
ARCHIVE
FEDERATED
EXAMPLE
BLACKHOLE
MERGE
NDBCLUSTER
CSV
```

通过 `show engines`可以查看引擎种类
存储引擎是作用在表上的，因此不同的表可以有不同的存储引擎类型
PerconaDB： 默认XtraDB
MaraDB： 默认InnoDB
TokuDB，RocksDB，MyRocks有共同点：压缩比较高，数据插入性能极高

## InnoDB

### 优点

```php
1、事务（Transaction）
2、MVCC（Multi-Version Concurrency Control多版本并发控制）
3、行级锁(Row-level Lock)
4、ACSR（Auto Crash Safey Recovery）自动的故障安全恢复
5、支持热备份(Hot Backup)
6、Replication: Group Commit , GTID (Global Transaction ID) ,多线程(Multi-Threads-SQL ) 
```

## 存储引擎查看

使用select查看 select @@default_storage_engine;
`set default_storage_engine=myisam`会话级别
`set global default_storage_engine=myisam`全局级别(影响新会话)
如果要永久生效，则在配置文件中修改

show确认每个表的存储引擎
`show create table City\G`
`show table status like 'countrylanguage'\G`

INFORMATION_SCHEMA确认每个表的存储引擎
`select table_schema,table_name ,engine from information_schema.tables where table_schema not in ('sys','mysql','information_schema','performance_schema');`

修改一个表的存储引擎
`alter table t1 engine innodb;`

## InnoDB存储引擎物理存储结构

### 最直观的存储方式：(/data/musql/data)

```undefined
ibdata1：系统数据字典信息(统计信息)，UNDO表空间等数据
ib_logfile0 ~ ib_logfile1: REDO日志文件，事务日志文件。
ibtmp1： 临时表空间磁盘位置，存储临时表
frm：存储表的列信息
ibd：表的数据行和索引
```

### 表空间(Tablespace)

#### 共享表空间：

InnoDB将所有数据存储在一个单独的表空间里，这个表空间可以由很多个文件组成，一个表可以跨多个文件存在，所以其大小限制不再是文件大小的限制，而是其自身的限制。一般当数据量比较小时使用。
**现在共享表空间只用来存储数据字典信息**

#### 独立表空间：

从5.6开始，默认表空间不再使用共享表空间，改为独立表空间。主要存储用户数据。
特点：一个表一个ibd文件，存储数据行和索引信息
基本表结构元数据存储： XXX.frm

```shell
元数据 	  =	数据行 + 索引
mysql表数据= (ibdataX + frm) + ibd(段，区，页)
DDL       = DML + DQL

mysql的存储引擎日志
Redo log：ib_logfile0 ,ib_logfile1  重做日志
Undo log：ibdata1 ，ibdata2(存储在共享表空间中)回滚日志
临时表：ibtmp1 -> 在join union操作产生的数据，用完自动清除
```

## 事务的ACID特性

- A(Atomic)所有语句作为一个单元全部成功执行或全部取消，不能出现中间状态
- C(Consistent)如果数据在事务开始时处于一致状态，则在执行该事务期间将保留一致状态
- I(Isolated)事务之间不相互影响
- D(Durable)事务成功之后，所做的所有更改都会准确记录在数据库中，所做的更改不会丢失

## 事务的生命周期

- 事务开始
  begin：在5.5以后，只要执行的是DML，会自动加上begin命令

- 事务结束
  commit：提交事务。一旦提交成功，就说明具备ACID特性了
  rollback：回滚事务。

- 自动提交策略

  ```csharp
  db01 [(none)]>select @@autocommit;
  db01 [(none)]>set autocommit=0;
  db01 [(none)]>set global autocommit=0;
  不管有无事务需求，都建议设置为0，提高数据库性能
  ```

- 隐式提交语句

  ```ruby
  导致提交的非事务语句：
  DDL语句： （ALTER、CREATE 和 DROP）
  DCL语句： （GRANT、REVOKE 和 SET PASSWORD）
  锁定语句：（LOCK TABLES 和 UNLOCK TABLES）
  导致隐式提交的语句示例：
  TRUNCATE TABLE
  LOAD DATA INFILE
  SELECT FOR UPDATE
  
  ```

- 事务流程

```csharp
1、检查autocommit是否为关闭状态
select @@autocommit;
或者： show variables like 'autocommit';
2、开启事务,并结束事务
begin
delete from student where name='alexsb';
update student set name='alexsb' where name='alex';
rollback;  或者 commit;
```

