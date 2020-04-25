## 索引作用，种类

提供类似书目录作用，优化查询

### 种类（算法）

B树索引，现在MySQL用B*Tree,重点考察B+Tree索引构建过程
从页节点开始构建，最后到根节点

### 聚簇（区）索引BTree结构

![Screen Shot 2020-04-21 at 8.24.19 pm](/Users/bowei/Documents/Note/Pic/Screen Shot 2020-04-21 at 8.24.19 pm.png)

构建前提：

1. 建表时，指定了主键列，MySQL InnoDB会将主键自动作为聚簇索引，如 not null primary key
2. 没有指定主键，自动选择唯一键列（unique），作为聚簇索引
3. 都没有，生成隐藏聚簇索引

作用：

1. 有了聚簇索引，降来插入的数据行，在同一个区内，按照ID值顺序有序存储数据

在存放数据时，会根据数据所在的页节点往上一级级升到枝节点和根节点，从而实现BTree，枝节点存放的下层节点的值，页节点的指针和存储范围。

### 辅助索引B+Tree结构

#### B—Tree构建

类似聚簇索引，但是辅助索引根据其他的值排列拍获取一个新的BTree，得到ID值之后，再回到聚簇索引，再一次查找ID（回表）。提高索引优化效果

```shell
(1). 索引是基于表中,列(索引键)的值生成的B树结构
(2). 首先提取此列所有的值,进行自动排序
(3). 将排好序的值,均匀的分布到索引树的叶子节点中(16K)
(4). 然后生成此索引键值所对应得后端数据页的指针
(5). 生成枝节点和根节点,根据数据量级和索引键长度,生成合适的索引树高度
id  name  age  gender
select  *  from  t1 where id=10;
问题: 基于索引键做where查询,对于id列是顺序IO,但是对于其他列的查询,可能是随机IO.
```

#### 聚簇索引和辅助索引构成区别

1. 聚集索引只能有一个,非空唯一,一般时主键
2. 辅助索引,可以有多个,时配合聚集索引使用的
3. 聚集索引叶子节点,就是磁盘的数据行存储的数据页
4. MySQL是根据聚集索引,组织存储数据,数据存储时就是按照聚集索引的顺序进行存储数据
5. 辅助索引,只会提取索引键值,进行自动排序生成B树结构

#### 辅助索引细分

1. 普通的单列辅助索引
2. 联合索引：多个列作为索引条件，生成索引树，理论上设计的好可以减少大量回表
3. 唯一索引：索引列的值都是唯一的

#### 索引树的高度影响

1. 数据量级，解决办法：分表，分库，分布式
2. 索引列值过长，解决办法：前缀索引
3. 数据类型：变长长度字符串，把char改成varchar
   enum类型的使用enum

## 索引的基本管理

#### 索引建立前

```ruby
db01 [world]>desc city;
+------------+----------+----+-----+------+----------------+
| Field   | Type   | Null | Key | Default | Extra          |
+------------+----------+----+-----+------+----------------+
| ID         | int(11)  | NO | PRI | NULL | auto_increment |
| Name       | char(35) | NO |     |      |                |
| CountryCode| char(3)  | NO | MUL |      |                |
| District   | char(20) | NO  |    |      |                |
| Population | int(11)  | NO  |    | 0    |                |
+------------+----------+-----+----+------+----------------+
Field :列名字
key  :有没有索引,索引类型
PRI: 主键索引
UNI: 唯一索引
MUL: 辅助索引(单列,联和,前缀)
```

#### 创建索引

```shell
# alter table用来创建普通索引，UNIQUE索引和PRIMARY KEY索引
ALTER TABLE table_name ADD INDEX index_name (column_list)
ALTER TABLE table_name ADD UNIQUE (column_list)
ALTER TABLE table_name ADD PRIMARY KEY (column_list)
#CREATE INDEX对表增加普通索引或UNIQUE索引
CREATE INDEX index_name ON table_name (column_list)
CREATE UNIQUE INDEX index_name ON table_name (column_list)
#删除索引
ALTER TABLE table_name DROP INDEX index_name
```

#### 覆盖索引（联合索引）

```shell
#索引包含所有满足查询所需要的数据的索引，即不需要回表，从非主键索引中就能查到记录
ALTER TABLE table_name add INDEX index_name(col1, col2);
```

#### 前缀索引

```shell
ALTER TABLE table_name ADD INDEX (index_name(4))
#以index——name的前四个字符来创建索引
```

#### 唯一索引

```shell
alter table city add unique index idx_uni1(name);
#单列主要是让该列在表中只能有唯一的一行，例如邮箱号，手机号等。多列则是多个加起来是唯一的。创建之后数据库进行存储操作时会判断库中是否已经有此数据，没有才进行插入操作
```

## 执行计划获取和分析

```csharp
(1)
获取到的是优化器选择完成的,他认为代价最小的执行计划.
作用: 语句执行前,先看执行计划信息,可以有效的防止性能较差的语句带来的性能问题.
如果业务中出现了慢语句，我们也需要借助此命令进行语句的评估，分析优化方案。
(2) select 获取数据的方法
1. 全表扫描(应当尽量避免,因为性能低)
2. 索引扫描
3. 获取不到数据
  
  执行计划获取
  desc select * from table_name where id=1;
explain select * from table_name where id=1;
```

```shell
关注的信息
table: city                 				 ---->查询操作的表    **
possible_keys: CountryCode,idx_co_po ---->可能会走的索引  **
key: CountryCode  									 ---->真正走的索引    ***
type: ref   												 ---->索引类型       *****
Extra: Using index condition         ---->额外信息       *****
```

```csharp
Type详解
从上到下性能越来越好
  
ALL  :  全表扫描,不走索引
例子:
1. 查询条件列,没有索引
SELECT * FROM t_100w WHERE k2='780P';  
2. 查询条件出现以下语句(辅助索引列)
USE world 
DESC city;
DESC SELECT * FROM city WHERE countrycode <> 'CHN';
DESC SELECT * FROM city WHERE countrycode NOT IN ('CHN','USA');
DESC SELECT * FROM city WHERE countrycode LIKE '%CH%';
注意:对于聚集索引列,使用以上语句,依然会走索引
DESC SELECT * FROM city WHERE id <> 10;

INDEX  :	全索引扫描
1. 查询需要获取整个索引树种的值时:
DESC  SELECT countrycode  FROM city;

2. 联合索引中,任何一个非最左列作为查询条件时:
idx_a_b_c(a,b,c)  ---> a  ab  abc

SELECT * FROM t1 WHERE b 
SELECT * FROM t1 WHERE c    

RANGE :	索引范围扫描 
辅助索引> < >= <= LIKE IN OR 
主键 <>  NOT IN

例子:
1. DESC SELECT * FROM city WHERE id<5;
2. DESC SELECT * FROM city WHERE countrycode LIKE 'CH%';
3. DESC SELECT * FROM city WHERE countrycode IN ('CHN','USA');

注意: 
1和2例子中,可以享受到B+树的优势,但是3例子中是不能享受的.
所以,我们可以将3号列子改写:
DESC SELECT * FROM city WHERE countrycode='CHN'
UNION ALL 
SELECT * FROM city WHERE countrycode='USA';

ref: 	非唯一性索引,等值查询
DESC SELECT * FROM city WHERE countrycode='CHN';
eq_ref: 在多表连接时,连接条件使用了唯一索引(uk  pK)

DESC SELECT b.name,a.name FROM city AS a 
JOIN country AS b 
ON a.countrycode=b.code 
WHERE a.population <100;
DESC country
system,const :
唯一索引的等值查询
DESC SELECT * FROM city WHERE id=10;
```

#### explain和desc使用场景

```css
题目意思:  我们公司业务慢,请你从数据库的角度分析原因
1.mysql出现性能问题,我总结有两种情况:
（1）应急性的慢：突然夯住
应急情况:数据库hang(卡了,资源耗尽)
处理过程:
1.show processlist;  获取到导致数据库hang的语句
2. explain 分析SQL的执行计划,有没有走索引,索引的类型情况
3. 建索引,改语句
（2）一段时间慢(持续性的):
(1)记录慢日志slowlog,分析slowlog
(2)explain 分析SQL的执行计划,有没有走索引,索引的类型情况
(3)建索引,改语句
```

## 建立索引原则

为了提高索引使用效率，在创建索引时，就要考虑那些字段上创建索引和创建什么类型的索引。

1. 建表时一定要有主键，一般是个无关列

2. 选择唯一性索引

3. 为经常需要where，order by，group by，join on等操作的字段建立联合索引

4. 尽量使用前缀来索引

5. 限制索引条目

   ```shell
   (1) 每个索引都需要占用磁盘空间，索引越多，需要的磁盘空间就越大。
   (2) 修改表时，对索引的重构和更新很麻烦。越多的索引，会使更新表变得很浪费时间。
   (3) 优化器的负担会很重,有可能会影响到优化器的选择.
   ```

6. 删除不再使用或很少使用的索引
7. 大表加索引要在业务部繁忙的时候操作
8. 尽量少在经常更新值的列上建索引

## 不走索引的情况（开发规范）

1. 在没有查询条件，或者查询条件没有建立索引的情况下，换成有索引的列作为查询条件，或建立索引
2. 查询结果如果是原表中超过25%的数据，优化器就觉得没必要走索引了。如果允许可以用limit控制，或者放redis运行
3. 索引本身失效，统计数据不真实时，考虑删除索引重建
4. 查询条件使用函数在索引列上，尽量不要对索引进行运算(+-*/!等)
5. 隐式转换导致索引失效（操作符与不同类型操作数一起使用时，会发生类型转换使操作符兼容，从而可能会全表扫描）
6. <>,not in不走索引（辅助索引），单独的>,<,in可能走，尽量结合limit；or或in尽量改成union
7. 使用like，如果%在最前面不走索引

```shell
EXPLAIN SELECT * FROM teltab WHERE telnum LIKE '31%'  走range索引扫描
EXPLAIN SELECT * FROM teltab WHERE telnum LIKE '%110'  不走索引
```



