

# 客户端/服务端模型

```shell
TCP/IP方式（远程、本地）：
mysql -uroot -poldboy123 -h 10.0.0.51 -P3306
Socket方式(仅本地)：
mysql -uroot -poldboy123 -S /tmp/mysql.sock

实例=mysqld后台守护进程+Master Thread +干活的Thread+预分配的内存
公司=      老 板   +      经理      +     员工   +  办公室
```

# MySQL程序运行原理

## 连阶层

```shell
（1）提供连接协议：TCP/IP 、SOCKET
（2）提供验证：用户、密码，IP，SOCKET
（3）提供专用连接线程：接收用户SQL，返回结果
通过以下语句可以查看到连接线程基本情况
mysql> show processlist;
```

## SQL层

```shell
（1）接收上层传送的SQL语句
（2）语法验证模块：验证语句语法,是否满足SQL_MODE
（3）语义检查：判断SQL语句的类型
DDL ：数据定义语言
DCL ：数据控制语言
DML ：数据操作语言
DQL： 数据查询语言
...
（4）权限检查：用户对库表有没有权限
（5）解析器：对语句执行前,进行预处理，生成解析树(执行计划),说白了就是生成多种执行方案.
（6）优化器：根据解析器得出的多种执行计划，进行判断，选择最优的执行计划
        代价模型：资源（CPU IO MEM）的耗损评估性能好坏
（7）执行器：根据最优执行计划，执行SQL语句，产生执行结果
执行结果：在磁盘的xxxx位置上
（8）提供查询缓存（默认是没开启的），会使用redis tair替代查询缓存功能
（9）提供日志记录（日志管理章节）：binlog，默认是没开启的。
```

## 存储层

```shell
负责根据SQL层执行的结果，从磁盘上拿数据。
将16进制的磁盘数据，交由SQL结构化化成表，
连接层的专用线程返回给用户。
```

## 表的段，区，页（16k）

```shell
页：最小的存储单元，默认16k
区：64个连续的页，共1M
段：一个表就是一个段，包含一个或多个区
```

# 基础管理

## 用户，权限管理

```shell
用户名@‘白名单’
例如
mytest@‘localhost’ :mytest用户能够通过本地登录MYSQL(socket方式)
mytest@‘10.0.0.10’:mytest用户能够通过10.0.0.10（远程登录）
mytest@‘10.0.0.%’:mytest用户能够通过10.0.0.x/24远程登录
```

## 用户管理操作

```shell
增：
mysql> create user bowie@'10.0.0.%' identified by '123';
查：
mysql> desc mysql.user;    ---->  authentication_string
mysql> select user,host,authentication_string from mysql.user
改:
mysql> alter user bowie@'10.0.0.%' identified by '456';
删：
mysql> drop user bowie@'10.0.0.%';
```

## 权限管理

```shell
# 权限管理操作
mysql>grant all on *.* to root@'10.0.0.%' identified by '123';
# 授权流程
#权限 -- 对谁操作 -- 从哪里来 -- 密码要求

##常用权限
ALL;   #包含以下所有
SELECT,INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE

#权限作用范围
*.* 								    --->管理员用户
wordpress.*/wordpress.t1 -->开发和应用用户

#查看授权
mysql> show grants for app@'10.0.0.%';
#收回权限
revoke  delete on app.*  from app@'10.0.0.%'；
```

## 连接管理

常用参数： -u#用户		-p#密码		-h#IP		-P#端口		-S#scoket		
					  -e#免交互执行命令			< #导入SQL脚本

## 常用SQL分类

```undefined
DDL：数据定义语言
DCL：数据控制语言
DML：数据操作语言
DQL：数据的查询语言
```

## 数据类型

### 数值类型

```shell
整数		TINYINT		0-255
整数		SMALLINT	-2^15 ~ 2^15 -1
整数		MEDIUMINT	中型整数数据类型
整数		INT				常规大小的整数数据类型(-2^31 ~ 2^31 -1)
整数		BIGINT		-2^63 ~ 2^63 -1 
浮点数		FLOAT			小型单精度浮点数（4byte）
浮点数		DOUBLE		常规双精度浮点数（8byte）
定点数		DECIMAL		包含整数部分，小数部分或同时包含的精确值数值
BIT			BIT				位字段值
```

### 字符类型

```42shell
文本		CHAR  固定长度字符串，maximum 255 character
文本		VARCHAR  可变长度字符串，maximum 65535 character
## char(11),不满11个由空格填充，如果是varchar则是最多11个
文本		TINYTEXT	可变长度字符串，maximum 255 character
文本		TEXT			可变长度字符串，macimum 65535 character
文本		MEDIUMTEXT	可变长度字符串，maximum 16777215 character
文本		LONGTEXT 		可变长度字符串，maximum 4294967295 character
整数		ENUM		由一组固定的合法值组成的枚举		enum('bj','tj');
整数		SET			由一组合法值组成的集
```

### 时间类型

```shell
DATA 			YYYY-MM-DD.  2006-08-04
TIME			HH:MM:SS[.UUUUUU]  12:59:02.123456
DATETIME	YYYY-MM-DD					2006-08-04
					HH:MM:SS[.UUUUUU]  12:59:02.123456
TIMESTAMP	YYYY-MM-DD					2006-08-04 12:59:02.123456
					HH:MM:SS[.UUUUUU]  	会受到时区影响
YEAR			YYYY								2006
```

### 二进制类型

```shell
二进制		BINARY		类似CHAR类型，但是存储的是二进制字节字符串
二进制		VARBINARY	类似VARCHAR类型，存储的是二进制字节字符串
BLOB		TINYINT		最大长度为255字节的BLOB列
BLOB		BLOB			最大长度为65535字节
BLOB		MEDIUMBLOB	最大长度为16777215长度
BLOB		LONGBLOB	最大长度为4294967295
```

## 表属性

### 列属性

```shell
约束(一般建表时添加):
**primary key** ：主键约束
设置为主键的列，此列的值必须非空且唯一，主键在一个表中只能有一个，但是可以有多个列一起构成。
**not null**   ：非空约束
列值不能为空，也是表设计的规范，尽量将所有的列设置为非空。可以设默认值为0
**unique key** ：唯一键
列值不能重复
**unsigned** 	 ：无符号
针对数字列，非负数。

其他属性:
**key** :索引
可以在某列上建立索引，来优化查询,一般是根据需要后添加
**default**  : 默认值
列中，没有录入值时，会自动使用default的值填充
**auto_increment**:自增长
针对数字列，顺序的自动填充数据（默认是从1开始，将来可以设定起始点和偏移量）
**comment ** : 注释
```

### 表的属性

```shell
存储引擎:
InnoDB（默认的）
字符集:  utf8       utf8mb4
排序规则(校对规则)：小小写是否敏感
```

# DDL（数据定义语言）应用

## 数据库增删改查

```
create database xyz charset utf8mb4 collate utf8mb4_bin;
show create database xyz; #查看建库指令
规范：1。库名不能有大写字母
		 2。建库要加字符集
		 3.库名不能有数字开头
		 4.库名要和业务相关

删除##生产中禁止使用
drop database xyz;

改
show create database xyz;
alter database school charset utf8;
#修改字符集，修改后的必须是原来的严格超集

查询库相关信息（DQL）
show databases;
show create database xyz;
```

## 表定义

```shell
创建表
USE school;
CREATE TABLE stu(
id      INT NOT NULL PRIMARY KEY AUTO_INCREMENT COMMENT '学号',
sname   VARCHAR(255) NOT NULL COMMENT '姓名',
sage    TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '年龄',
sgender ENUM('m','f','n') NOT NULL DEFAULT 'n' COMMENT '性别' ,
sfz     CHAR(18) NOT NULL UNIQUE  COMMENT '身份证',
intime  TIMESTAMP NOT NULL DEFAULT NOW() COMMENT '入学时间'
) ENGINE=INNODB CHARSET=utf8 COMMENT '学生表';

建表规范
1. 表名小写
2. 不能是数字开头
3. 注意字符集和存储引擎
4. 表名和业务有关
5. 选择合适的数据类型
6. 每个列都要有注释
7. 每个列设置为非空，无法保证非空，用0来填充。

删除。drop table stu
```

### 修改表

```shell
1. 在stu表中添加qq列
desc stu;
alter table stu add qq varchar(20) not null unique comment'qq号‘;
2. 在sname后加微信列
alter table stu add wechat varchar(64) not null unique comment'微信号' after sname;
3. 在id列前加一个新列num
alter table stu add num int not null comment'数字' first;
4. 删除所有刚刚加的列
alter table stu drop qq; alter table stu drop wechat;
alter table stu drop num;
5. 修改sname数据类型的属性
alter table stu modify sname varchar(128) not null;
## 原为sname VARCHAR(255) NOT NULL COMMENT '姓名'
6. 将sgender改为sg，数据类型改为char
alter table stu change sgender sg char(1) not null defult 'n';
```

### 表属性查询(DQL)

```shell
use school;
show tables;
desc stu;
show create table stu;
create table ceshi like stu;
```

### DCL用户账号及权限管理

`grant		revoke	`

## DML应用

作用：对表中数据进行增，删，改

## insert

```shell
最标准——————
insert into stu(id, name, sage, sg, sfz, intime)
values
(1,'zs',18,'m','123456',now());
select * from stu;  ## 查看当前数据
省事————————
insert into stu
values
(1,'zs',18,'m','123456',now());
针对性录入数据————
insert into stu(sname, sfz)
values ('w5','33442244');
录入多行数据————————
insert into stu(sname, sfz)
values
('w55','3444578d8'),
('m6','1212313');
select * from stu;
```

### update/delete

```shell
desc stu;# 查看表属性
select * from stu; #查看表内容
update stu set sname='w5' where id = 1;
## update语句一定要加where

delete from stu where id = 3;#删除一个数据
全表删除
delete from stu # DML操作，逻辑性质删除，逐行删除，速度慢
truncate table stu;#DDL操作，与表段中数据页进行清空，速度快
伪删除：新增status列，业务语句查询时只查询激活状态的列
alter table stu add state tinyint not null default 1;
update stu set state = 0 where id = 6;
select * from stu where state = 1;
```

## DQL应用（select）

```shell
单独使用
-- select @@xxx 查看系统参数
SELECT @@port;(basedir,datadir,socket,server_id)

--select 函数();
SELECT NOW();
SELECT DATABASE();
SELECT USER();
SELECT CONCAT("hello world");
SELECT CONCAT(USER,"@",HOST) FROM mysql.user;
SELECT GROUP_CONCAT(USER,"@",HOST) FROM mysql.user;
#将某一字段的值按照指定的字符进行累加
```

```shell
单表子句 -from
select 列1，列2 from 表/selct * from 表（全部）
select * from stu  # 查询stu中所有数据（不要对大表操作）
select sname, intime from stu# 查询学生姓名和入学时间
```

```shell
单表子句 -where
select col1,col2 from table where coln condition
select * from stu where sname = 'w5';
select * from stu where id < 10;#比较操作符
select * from stu where id < 10 and sname = 'w5';#逻辑运算符
select * from stu where id < 10 or sname = 'w5';
select * from stu where sname like 'w%';#模糊查询，%不能放在前面，因为不走索引
select * from stu where sname in ('w5','bowie')#w5或bowie信息
select * from stu where id between 2 and 5;#id大于2小于5的学生
```

```shell
group by + 常用聚合函数 #根据一个或多个列结果集进行分组
**max()**      ：最大值
**min()**      ：最小值
**avg()**      ：平均值
**sum()**      ：总和
**count()**    ：个数
group_concat() : 列转行

统计age总和
select country,sum(population) from city group by country;
#city是表名，统计每个country的population总和
```

```shell
having
where | group | having
#统计chn每个省的总人数，只打印总人口数小于100w的省
select district,sum(population)
from city where country = 'chn'
group by district
having sum(population) < 1000000;
```

```shell
order by + limit
实现先排序，by后添加条件列
# 查看chn所有城市，按照人口从大到小排序
select * from city where county = 'chn' order by population DESC; # 默认从小到大，desc为从大到小排序
#统计各省总人口数量，从大到小排序
select  district,sum(population) as 总人口 from city
where country = 'chn'
group by district
order by 总人口 desc;
#统计中国每个省总人口，找出大于500w的，按照从大到小排序，只显示前三名
select district,sum(population) as 总人口 from city
where country = 'chn'  #选择国家
group by 总人口				#统计总人口
having 总人口 > 5000000	#只打印大雨500W的district
order by 总人口 desc		#从大到小排列
limit 3;			# 只显示3行，  limit n,m 跳过n行，显示m行
```

```shell
distinct # 去重复
select county from city;
select distict(country) from city;
```

```shell
union all 联合查询，以下两条语句效果一样
select * from city where country in ('chn','usa');
select * from city where country = 'chn'
union all
select * from city where country = 'usa';
#一般情况下会吧in或者or改写成union all提高性能，
union 去重复，而union all不去重复
```

### join多表连接查询

```shell
#查询张三家地址
select A.name, B.address from
A join B
on A.id=B.id
where A.name='zhangsan';
#查询人口数量小于100的城市名和国际名
select b.name, a.name, a.population
from city AS a join country AS b
on b.code=a.country
where a.population<100;
#查询张三学习的课程，st.sname->sc.cno->co.cname
select st.sname,group_concat(co.cname)
from student as st join sc
on st.sno=sc.sno
join course as co
on sc.cno=co.cno
where s.sname='zhang3';
#查询oldguo老师教的学生
select te.sname group_concat(st.tname) #concat里才是数据来源
from student as st join sc
on st.sno=sc.sno
join course as co on sc.cno=co.cno
join teacher as te on co.tno=te.tno
where te.sname='oldguo';	#te.sname 和 select后面参数对应
#每位老师所教课程的平均分，并排序
select te.tname,avg(sc.score)# 教师的平均分
from teacher as te join course as co
on te.tno=co.tno
join sc on co.cno=sc.cno
group by te.tname
order by avg(sc.score) desc;
#查询所有老师所教学生不及格信息
select te.tname, st.sname, sc.score
from teacher as te join course as co
on te.tno=co.tno
join sc on sc.cno=co.cno
join student as st on st.sno=sc.sno
where sc.score<60

### 查询和想好每个表关联的项目，并且意义连接起来最后用where，group by和order by等进行显示
```

## show命令

```shell
show databases;                          #查看所有数据库
show tables;                             #查看当前库的所有表
SHOW TABLES FROM                         #查看某个指定库下的表
show create database world               #查看建库语句
show create table world.city             #查看建表语句
show grants for  root@'localhost'        #查看用户的权限信息
show charset；                           #查看字符集
show collation                           #查看校对规则
show processlist;                        #查看数据库连接情况
show index from                          #表的索引情况
show status                              #数据库状态查看
SHOW STATUS LIKE '%lock%';               #模糊查询数据库某些状态
SHOW VARIABLES                           #查看所有配置信息
SHOW variables LIKE '%lock%';            #查看部分配置信息
show engines                             #查看支持的所有的存储引擎
show engine innodb status\G          #查看InnoDB引擎相关的状态信息
show binary logs                         #列举所有的二进制日志
show master status                       #查看数据库的日志位置信息
show binlog evnets in                    #查看二进制日志事件
show slave status \G                     #查看从库状态
SHOW RELAYLOG EVENTS                  #查看从库relaylog事件信息
desc (show colums from city)             #查看表的列定义信息
http://dev.mysql.com/doc/refman/5.7/en/show.html
```

## information_schema视图

information_schema是信息数据库，保存着关于MySQL服务器维护的所有其他数据库信息，只有视图，不是基本表

```shell
DESC information_schema.TABLES #保存了所有表的数据字典信息
TABLE_SCHEMA    ---->库名
TABLE_NAME      ---->表名
ENGINE          ---->引擎
TABLE_ROWS      ---->表的行数（不是很准确）
AVG_ROW_LENGTH  ---->表中行的平均行（字节）
DATA_LENGTH     ---->表使用的存储空间大小（不是实时）
INDEX_LENGTH    ---->索引的占用空间大小（字节）
```

```shell
#数据库统计，每个库，所有表的个数和表名
SELECT table_schema,COUNT(table_name),GROUP_CONCAT(name)
FROM  information_schema.tables
GROUP BY table_schema;
#统计每个库占用空间大小
SELECT table_schema,SUM(AVG_ROW_LENGTH*TABLE_RWOS+INDEX_LENGTH )/1024/1024
FORM information_schema.tables
GROUP BY table_schema;
#查询所有innodb引擎的表所在的库，除系统表
SELECT table_schema,table_name,ENGINE FROM information_schema.`TABLES`
WHERE ENGINE='innodb';
and table_schema not in ('sys','performance_schema', 'information_schema', 'mysql');
#生成整个数据库下的所有表的单独备份语句
模板语句：
mysqldump -uroot -p123 world city >/tmp/world_city.sql
SELECT CONCAT("mysqldump -uroot -p123 ",table_schema," ",table_name," >/tmp/",table_schema,"_",table_name,".sql" )
FROM information_schema.tables
WHERE table_schema NOT IN('information_schema','performance_schema','sys')
INTO OUTFILE '/tmp/bak.sh' ;

CONCAT("mysqldump -uroot -p123 ",table_schema," ",table_name," >/tmp/",table_schema,"_",table_name,".sql" )
```

MySQL一个逻辑表包含以下几个部分

数据字典（表中列的定义信息），数据行记录，索引，数据库状态，权限，日志

通过`information_schema视图`可以查看数据字典，数据库状态和权限
每次数据库启动都会自动在内存中生成I_S，生成查询MySQL部分元数据信息视图

```shell
视图？
select语句的执行方法，不保存数据本生
I_S中的视图，保存的就是查询元数据的方法，
例如create view 视图 as 语句
select * from 视图## 就可以直接执行语句了
```

拼接语句，通过`select concat("alter table", table_schema,".","table_name")`可以拼接输出一条语句。
如果想直接做成脚本可以在执行语句末尾加上一行
 `into outfile '/tmp/alter.sql'`把输出存入一个sql文件
可能会报错，需要前往cnf配置文件添加`secure-file-prive=/tmp`
然后输入 ``mysql -uroot -p </tmp/alter.sql`来执行脚本

### case语句

`case when 判断 then 结果 end`
比如得出所有老师上课的及格率：

```shell
select concat(teacher.tname,"_",teacher.tno), concat( count(case when sc.score>60 then 1 end)/count(sc.sno)*100,"%") as 及格率
from teacher join course
on teacher.tno=course.tno
join sc on course.cno=sc.cno
group by teacher.tno, teacher.tname;
#输出结果是左边教师名字，右边及格率
```

