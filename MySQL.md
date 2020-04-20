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
mysql> create user oldboy@'10.0.0.%' identified by '123';
查：
mysql> desc mysql.user;    ---->  authentication_string
mysql> select user,host,authentication_string from mysql.user
改:
mysql> alter user oldboy@'10.0.0.%' identified by '456';
删：
mysql> drop user oldboy@'10.0.0.%';
```

## 权限管理

