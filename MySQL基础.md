---
title: MySQL基础
date: 2018-06-07 18:09:00
toc: true
categories: 数据库
---

## 基本概念
所谓安装数据库服务器，只是在机器上安装了一个数据库管理系统程序，这个管理程序可以管理多个数据库，一般开发人员会针对每一个应用创建一个数据库。
为保存应用中实体的数据，一般会在数据库中创建多个表，以保存程序中实体的数据。
数据库服务器、数据库和表的关系如下：

![img](20181207131611230.png)

## MySQL安装

[《CentOS 6.5下编译安装MySQL 5.6.14》](http://blog.51cto.com/aiilive/2119890)
[《Windows下通过MySQL Installer安装MySQL服务》](http://blog.51cto.com/aiilive/2116476)
[《CentOS 7 通过 yum 安装 MariaDB》](https://zhuanlan.zhihu.com/p/49046496)
MariaDB数据库管理系统是MySQL的一个分支，主要由开源社区在维护，采用GPL授权许可 MariaDB的目的是完全兼容MySQL，包括API和命令行，使之能轻松成为MySQL的代替品，只是存储引擎不同！
## Mysql使用
### 连接Mysql服务
```sql
mysql -h 127.0.0.1 -P 3306 -u root -p
```
`-h` 选项默认是127.0.0.1 `-P`默认是3306 后面有没有空格都是可以的
很多时候我们要使用本地Mysql服务的话，直接简写为：
```sql
mysql -uroot -p
```
使用`services.msc`命令可以打开服务管理，来启动和关闭Mysql服务
[《MySql服务器的启动和关闭》](https://www.cnblogs.com/tv151579/p/3278056.html)这篇文章就讲述了如何通过命令其启动和关闭Mysql

## SQL分类
DDL数据定义语言，用来维护存储数据的结构
代表指令: `create`、 `drop`、`alter`

DML数据操纵语言，用来对数据进行行操作
代表指令： `insert`、`delete`、`update`

DML中又单独分了一个DQL，数据查询语言，代表指令： `select`

DCL数据控制语言，主要负责权限管理理和事务
代表指令： `grant` 、`revoke` 、`commit`

## MySQL架构
MySQL 是一个可移植的数据库，几乎能在当前所有的操作系统上运行，如 Unix/Linux、Windows、Mac 和 Solaris。各种系统在底层实现⽅方⾯面各有不不同，但是 MySQL 基本上能保证在各个平台上的物理体系结构的一致性：
![](https://s2.ax1x.com/2019/05/07/EsIEBd.png)
说说这张图：

* Client Connectors 是客户端链接，这个不用细说，就是应用程序与Mysql交互的接口，毕竟Mysql是要为程序提供数据存储服务的，所以必须将操作接口暴露出来，假如你是一个Java开发者，那么JDBC可以轻松链接上Mysql服务，就可以让你的Java程序使用上Mysql提供的服务
* Connection Pool这个是连接池，Mysql与外界可能不止有一个连接，多次链接和断开会造成非常大的性能消耗，于是用使用连接池来管理这些链接，这就如Java的线程池来管理线程一样，通过连接池来避免性能损耗
* Management Serveices & Utilities是管理服务和工具组件，例如备份恢复、Mysql复制、安全性验证、集群、分区工作台等，下面会演示一个Mysql备份的例子
* SQL Interface 就是SQL接口，存储过程、触发器、视图等，接受用户的SQL命令，并且返回用户需要查询的结果。接收DML(data manipulation language)数据操纵语言、DDL(data definition language数据库定义语言、比如select from就是调用SQL Interface
* Parser 是解析器，SQL命令传递到解析器的时候会被解析器验证和解析。解析器是由Lex和YACC实现的，是一个很长的脚本，将SQL语句分解成数据结构，并将这个结构传递到后续步骤，以后SQL语句的传递和处理就是基于这个结构的，如果在分解构成中遇到错误，那么就说明这个sql语句是不合理的 
* Optimizer 是查询优化器，SQL语句在查询之前会使用查询优化器对查询进行优化，这个不难理解，假如你有一张`info` 表中的字段是年龄(很显然这个额字段值是大于0的)，如果你在查询的时候的SQL语句是`select * form info where age=-10`，那么这条语句经过优化器之后不会再被执行，这就好像优化器知道不可能存在年龄小于0的条目
* Caches 是高速缓存， 查询缓存，如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。 通过LRU算法将数据的冷端溢出，未来得及时刷新到磁盘的数据页，叫脏页。 这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等 
* Pluggable Storage Engines 是存储引擎，图中的圆柱体都是存储引擎，Mysql默认的存储引擎是`InnoDB`，后面谈论存储引擎
* FileSystem 就是文件系统，Mysql数据库的数据最终还是要存放到文件中，所以我们可以理解为数据库就是一种帮我们管理数据的软件，处于文件系统的应用程序之间专门提供数据管理的软件，把数据的增删改查以及他的功能做了完美的封装，使用起来安全性更高，更方便我们队数据进行操作

## Mysql存储引擎
存储引擎是：数据库管理理系统如何存储数据、如何为存储的数据建立索引和如何更新、查询数据等技术的实现方法。
MySQL的核心就是插件式存储引擎，支持多种存储引擎，所以你可以看到在Mysql的架构图上存储引擎的小插头，存储引擎是插拔式的，默认是InnoDB（从MySQL5.5.8开始，之前是MyISAM），当然也可以选择其他的存储引擎
使用`show engines;`命令可以查看支持的存储引擎：
![](https://s2.ax1x.com/2019/05/07/EsImNt.png)

接下来说说他们的区别：

|特点|Myisam|BDB|Memory|InnoDB|Archive|
|:-|:-|:-|:-|:-|:-|
| 存储限制     |  没有  | 没有 |   有   |  64TB  |  没有   |
| 事物安全     |        | 支持 |        |  支持  |         |
| 锁机制       |  表锁  | 页锁 |  表锁  |  行锁  |  行锁   |
| B树索引      |  支持  | 支持 |  支持  |  支持  |         |
| 哈希索引     |        |      |  支持  |  支持  |         |
| 全文索引     |  支持  |      |        |        |         |
| 集群索引     |        |      |        |  支持  |         |
| 数据缓存     |        |      |  支持  |  支持  |         |
| 索引缓存     |  支持  |      |  支持  |  支持  |         |
| 数据可压缩   |  支持  |      |        |        |  支持   |
| 空间使用     |   低   |  低  |  中等  |   高   |   低    |
| 批量插入速度 |   高   |  高  |   高   |   低   | 非常高  |
| 支持外键     |        |      |        |  支持  |         |


### MyISAM存储引擎
MyISAM是MySQL官方提供默认的存储引擎，其特点是不支持事务、表锁和全文索引，对于一些OLAP系统(OLAP 系统强调数据分析，强调SQL执行市场，强调磁盘I/O，强调分区等)，操作速度快。关于[《OLAP、OLTP的介绍和比较》](https://www.cnblogs.com/hhandbibi/p/7118740.html)

每个MyISAM在磁盘上存储成三个文件。文件名都和表名相同，扩展名分别是.frm（存储表定义）、.MYD (MYData，存储数据)、.MYI (MYIndex，存储索引)。这里特别要注意的是MyISAM不缓存数据文件，只缓存索引文件。

### InnoDB存储引擎
InnoDB存储引擎支持事务，主要面向OLTP方面的应用，其特点是行锁设置、支持外键，并支持类似于Oracle的非锁定读，即默认情况下读不产生锁。InnoDB将数据放在一个逻辑表空间中。InnoDB通过多版本并发控制来获得高并发性，实现了ANSI标准的4种隔离级别，默认为Repeatable，使用一种被称为next-key locking的策略避免幻读。

对于表中数据的存储，InnoDB采用类似Oracle索引组织表Clustered的方式进行存储。

InnoDB 存储引擎提供了具有提交、回滚和崩溃恢复能力的事务安全。但是对比Myisam的存储引擎，InnoDB 写的处理效率差一些并且会占用更多的磁盘空间以保留数据和索引


### NDB存储引擎
NDB存储引擎是一个集群存储引擎，类似于Oracle的RAC，但它是Share Nothing的架构，因此能提供更高级别的高可用性和可扩展性。NDB的特点是数据全部放在内存中，因此通过主键查找非常快。

关于NDB，有一个问题需要注意，它的连接(join)操作是在MySQL数据库层完成，不是在存储引擎层完成，这意味着，复杂的join操作需要巨大的网络开销，查询速度会很慢。

### Memory (Heap) 存储引擎
Memory存储引擎（之前称为Heap）将表中数据存放在内存中，如果数据库重启或崩溃，数据丢失，因此它非常适合存储临时数据。


### Archive存储引擎
正如其名称所示，Archive非常适合存储归档数据，如日志信息。它只支持INSERT和SELECT操作，其设计的主要目的是提供高速的插入和压缩功能。

## Federated存储引擎
Federated存储引擎不存放数据，它至少指向一台远程MySQL数据库服务器上的表，非常类似于Oracle的透明网关。
### InnoDB与MyISAM应用场景
参考：[《InnoDB与MyISAM两者的区别》](https://www.cnblogs.com/kevingrace/p/5685355.html)
MyISAM管理非事务表。它提供高速存储和检索，以及全文搜索能力。如果应用中需要执行大量的SELECT查询，那么MyISAM是更好的选择。

InnoDB用于事务处理应用程序，具有众多特性，包括ACID事务支持。如果应用中需要执行大量的INSERT或UPDATE操作，则应该使用InnoDB，这样可以提高多用户并发操作的性能

## 存储引擎相关SQL
查看Mysql已提供存储引擎
```sql
show engines;
```
查看Mysql默认存储引擎:
```sql
show variables like 'storage_engine';
```
查看某个表的存储引擎:
```sql
show create table 表名;
```
修改表的存储引擎：
```sql
alter table 表名 engine=引擎
```
## 数据库操作
### 字符集与校验规则
当我们创建数据库没有指定字符集和校验规则时，系统使用默认字符集：`utf8`，校验规则
是：`utf8_ general_ ci` ，这个校验规则中的 `ci`就是Case insensitive意为不区分大小写

创建一个使用`utf8` 的字符集，并带校对规则为`utf8_general_ci`的数据库。
```sql
create database DBName charset=utf8 collate utf8_general_ci;
```
查看系统默认字符集、默认校验规则
```sql
show variables like 'character_set_database'; 
show variables like 'collation_database';
```
支持的字符集、支持的校验规则
```sql
show charset;
show collation;
```
### 创建数据库
```sql
mysql> create database myDB;
Query OK, 1 row affected (0.90 sec)

mysql> show create database myDB;
+----------+---------------------------------------------------------------+
| Database | Create Database                                               |
+----------+---------------------------------------------------------------+
| myDB     | CREATE DATABASE `myDB` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+---------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> create database `myDB2`;
Query OK, 1 row affected (0.00 sec)

mysql> show create database myDB2;
+----------+----------------------------------------------------------------+
| Database | Create Database                                                |
+----------+----------------------------------------------------------------+
| myDB2    | CREATE DATABASE `myDB2` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+----------------------------------------------------------------+
1 row in set (0.00 sec)
```
MySQL 建议我们关键字使用大写，但是不是必须的。
数据库名字的反引号,是为了防止使用的数据库名刚好是关键字
`/*!40100 DEFAULT CHARACTER SET utf8 */ `这个不是注释，表示当前Mysql版本大于4.01版本，就执行这句话

### 修改数据库
对数据库的修改主要指的是修改数据库的字符集，校验规则

例如把 mytest 数据库字符集改为 gbk
```sql
alter database mytest charset=gbk
```
### 删除数据库
不要随意删除数据库，否则还是容易**从删库到跑路的**
```sql
drop databse [if exists] 数据库名称
```
如果加上`if exists`那么删除一个不存在的数据库也不会出错，但是会有警告(查看警告就使用`show warnings;`)，如果不加`if exists`去删除一个不存在的数据库就会报错
```sql
mysql> drop database mydbs;
ERROR 1008 (HY000): Can't drop database 'mydbs'; database doesn't exist
mysql> drop database if exists mydbs;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> show warnings;
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                                   |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+
| Error | 1064 | You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'warning' at line 1 |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```
### 备份和恢复
```
mysqldump -P3306 -uroot -p -B 数据库名称 > 路径+数据库名称.sql
```
使用示例：
```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb1              |
| performance_schema |
| sakila             |
| test               |
| userdata           |
| world              |
+--------------------+
11 rows in set (0.00 sec)

mysql> exit;
Bye

C:\Users\15291\Desktop>mysqldump -P3306 -uroot -p -B mydb1 > ./mydb1.sql
Enter password: ****
```
备份后会生成mydb1.sql文件，通过执行这个sql脚本就会恢复数据库：
```sql
mysql> source C:/Users/15291/Desktop/mydb1.sql;
```
如果备份的不是整个数据库，而是其中的一张表:
```sql
mysqldump -u root -p 数据库名 表名1 表名2 > ./mytest.sql
```
同时备份多个数据库:
```sql
mysqldump -u root -p -B 数据库名1 数据库名2 ... > 数据库存放路路径
```
如果备份一个数据库时，没有带上-B参数， 在恢复数据库时，需要先创建空数据库，然后使用数据库，再使用source来还原
### 查看连接情况
可以告诉我们当前有哪些用户连接到我们的MySQL，如果查出某个用户不不是你正常登陆的，很有可能数据库被人入侵了。如果发现数据库比较慢时，可以用这个指令来查看数据库连接情况：
```sql
mysql> show processlist;
+----+------+-----------------+------+---------+------+-------+------------------+
| Id | User | Host            | db   | Command | Time | State | Info             |
+----+------+-----------------+------+---------+------+-------+------------------+
|  9 | root | localhost:55469 | NULL | Query   |    0 | init  | show processlist |
+----+------+-----------------+------+---------+------+-------+------------------+
1 row in set (0.04 sec)
```
## 表的操作
### 创建表
```sql
CREATE TABLE table_name (
	field1 datatype,
	field2 datatype,
	field3 datatype
) character set 字符集 collate 校验规则 engine 存储引擎;
```
不同的存储引擎，创建表的文件不一样，这个在`Myisam`存储引擎中说到过
假设users表存储引擎是`Myisam`，在数据目中有三个不同的文件，分别是：
users.frm：表结构
users.MYD：表数据
users.MYI：表索引

### 查看表结构
```sql
mysql> desc users;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | YES  |     | NULL    |       |
| name     | varchar(20) | YES  |     | NULL    |       |
| password | char(32)    | YES  |     | NULL    |       |
| birthday | date        | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
4 rows in set (0.03 sec)
```
### 修改表/表结构
```sql
ALTER TABLE tablename ADD (column datatype [DEFAULT expr][,column datatype]...);
ALTER TABLE tablename MODIfy (column datatype [DEFAULT expr][,column datatype]...);
ALTER TABLE tablename DROP (column);
```
使用示例：

在users表的password字段后添加assets字段，类型为varchar(50)，备注为图片路径
```sql
mysql> alter table users add assets varchar(50) comment '图片路径' after password;
Query OK, 0 rows affected (0.14 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc users;                                                            
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | YES  |     | NULL    |       |
| name     | varchar(20) | YES  |     | NULL    |       |
| password | char(32)    | YES  |     | NULL    |       |
| assets   | varchar(50) | YES  |     | NULL    |       |
| birthday | date        | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
5 rows in set (0.02 sec)
```
修改users表中name字段的类型为varchar(30)
```sql
mysql> alter table users modify name varchar(30);
Query OK, 0 rows affected (0.09 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc users;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | YES  |     | NULL    |       |
| name     | varchar(30) | YES  |     | NULL    |       |
| password | char(32)    | YES  |     | NULL    |       |
| assets   | varchar(50) | YES  |     | NULL    |       |
| birthday | date        | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
5 rows in set (0.02 sec)
```
删除users表中assets字段
```sql
mysql> alter table users drop assets;
Query OK, 0 rows affected (0.09 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc users;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | YES  |     | NULL    |       |
| name     | varchar(30) | YES  |     | NULL    |       |
| password | char(32)    | YES  |     | NULL    |       |
| birthday | date        | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
4 rows in set (0.01 sec)
```
对user表的名字进行重命名
```sql
mysql> alter table users rename to user;
Query OK, 0 rows affected (0.01 sec)
```
对user表中的name字段进行重命名并更改字段类型
```sql
mysql> alter table user change name xingming varchar(50);
Query OK, 0 rows affected (0.09 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc user;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | YES  |     | NULL    |       |
| xingming | varchar(50) | YES  |     | NULL    |       |
| password | char(32)    | YES  |     | NULL    |       |
| birthday | date        | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
4 rows in set (0.02 sec)
```
删除user表
```sql
mysql> drop table user;
Query OK, 0 rows affected (0.01 sec)

mysql> show tables;
+---------------+
| Tables_in_db1 |
+---------------+
| person        |
+---------------+
1 row in set (0.00 sec)
```
### Mysql数据类型
#### 数值类型
#### tinyint-bigint
这是有符号的范围，无符号的返回自行推导，和C的无符号是一致的

|类型|字节|Min|Max|
|:-|:-|:-|:-|
|  TINYINT  |  1   |         -128         |         127         |
| SMALLINT  |  2   |        -32768        |        32767        |
| MEDIUMINT |  3   |       -8388608       |       8388607       |
|    INT    |  4   |     -2147483648      |     2147483647      |
|  BIGINT   |  8   | -9223372036854775808 | 9223372036854775807 |

在MySQL中，整型可以指定是有符号的和无符号的，默认是有符号的。
可以通过UNSIGNED来说明某个字段是无符号的，但是一般遇到存储类型不足以存储数据的大小时，应该换成更大的类型，而不是换成对应的无符号类型！
#### bit
`bit[(M)] `: 位字段类型。M表示每个值的位数，范围从1到64。如果M被忽略，默认为1
注意bit类型，bit字段在显示时，是按照ASCII码对应的值显示！
如果我们有这样的值:只存放0或1，这时可以定义bit(1)，这样可以节省空间！
#### float
`float[(m, d)] [unsigned] `: M指定显示长度，d指定小数位数，占用空间4个字节！
#### decimal
浮点数float与定点数decimal的存储方式的不同决定了他们的用途不同：[《浮点数的存储方式》](https://blog.csdn.net/JoJoJo1234/article/details/79968358)
decimal用于保存必须为确切精度的值，很显然底层是用字符串来存储的
```sql
decimal(m, d) [unsigned] 
```
定点数m指定长度，d表示小数点的位数

float表示的精度大约是7位。decimal整数最大位数m为65。支持小数最大位数d是30。如果d被省略，默认为0.如果m被省略，默认是10。
建议：如果希望小数的精度高，推荐使用decimal(对于银行这种对小数要求非常高的业务，decimal还是大有用处的)
#### 字符串类型
##### char
```sql
char(L)
```
固定长度字符串，L是可以存储的长度，单位为字符，最大长度值可以为255，这里需要注意的是一个汉字和一个字母均被视为一个字符！
##### varchar
```sql
varchar(L)
```
可变长度字符串，L表示字符长度，最大长度65535个字节
关于varchar(len)，len到底是多大，这个len值，和表的编码密切相关：varchar长度可以指定为0到65535之间的值，但是有两个字节用于记录数据大小，所以说有效字节数65532。
当我们的表的编码是utf8时，varchar(n)的参数n最大值是65532/3=21844[因为utf中，一个字符占用3个字节]，如果编码是gbk，varchar(n)的参数n最大是65532/2=32766（因为gbk中，一个字符占用2字节）

##### char和varcahr比较

| 实际存储 | char(4) | varchar(4) | char占用字节 | varchar占用字节 |
| :------: | :-----: | :--------: | :----------: | :-------------: |
|   acbd   |  adcb   |    adcb    |    4*3=12    |    4*3+1=13     |
|    A     |    A    |     A      |    4*3=12    |      1*3+1      |
|  Abcde   |    ×    |     ×      |  数据超过度  |   数据超过度    |

两者如何选择？
如果数据确定长度都一样，就使用定长（char），比如：身份证，手机号，md5
如果数据长度有变化,就使用变长(varchar), 比如：名字，地址，但是你要保证最长的能存的进去
定长的磁盘空间比较浪费，但是效率高
变长的磁盘空间比较节省，但是效率

#### 日期和时间类型
常用的日期有如下三个：
* datetime 时间日期格式 'yyyy-mm-dd HH:ii:ss' 表示范围从1000到9999，占用八字节
* date:日期 'yyyy-mm-dd'，占用三字节
* timestamp：时间戳，从1970年开始的 yyyy-mm-dd HH:ii:ss格式和datetime完全一致，占用四字节

使用示例：
```sql
mysql> create table birthday (t1 date, t2 datetime, t3 timestamp);
Query OK, 0 rows affected (1.00 sec)

mysql> show tables;
+---------------+
| Tables_in_db1 |
+---------------+
| birthday      |
| person        |
+---------------+
2 rows in set (0.00 sec)

mysql> insert into birthday(t1,t2) values('1997-7-1','2008-8-8 12:1:1');
Query OK, 1 row affected (0.22 sec)

mysql> select * from birthday;
+------------+---------------------+---------------------+
| t1         | t2                  | t3                  |
+------------+---------------------+---------------------+
| 1997-07-01 | 2008-08-08 12:01:01 | 2018-12-07 18:57:58 |
+------------+---------------------+---------------------+
1 row in set (0.00 sec)
```
#### enum和set
语法：
enum：枚举，“单选”类型；
```sql
enum('选项1','选项2','选项3',...);
```
该设定只是提供了若干个选项的值，最终一个单元格中，实际只存储了其中一个值；而且出于效率考虑，这些值实际存储的是“数字”，因为这些选项的每个选项值依次对应如下数字：1,2,3,....最多65535个；当我们添加枚举值时，也可以添加对应的数字编号。

set：集合，“多选”类型；
```sql
set('选项值1','选项值2','选项值3', ...);
```
该设定只是提供了若干个选项的值，最终一个单元格中，设计可存储了其中任意多个值；而且出于效率考虑，这些值实际存储的是“数字”，因为这些选项的每个选项值依次对应如下数字：1,2,4,8,16,32，.... 最多64个。

不建议在添加枚举值，集合值的时候采用数字的方式，因为不利于阅读。
使用示例：
```sql
mysql> create table votes(
    -> username varchar(30),
    -> hobby set('登山','黎狗子吃粑粑','运动'),
    -> gender enum('男','女'));
Query OK, 0 rows affected (0.43 sec)

mysql> insert into votes values('LiLiNaNa','登山,黎狗子吃粑粑','男');
Query OK, 1 row affected (0.09 sec)

mysql> select * from votes;
+----------+-------------------+--------+
| username | hobby             | gender |
+----------+-------------------+--------+
| LiLiNaNa | 登山,黎狗子吃粑粑 | 男     |
+----------+-------------------+--------+
1 row in set (0.02 sec)
```
集合查询使用`find_ in_ set`函数：
```sql
find_in_set(sub,str_list) 
```
如果sub 在str_list 中，则返回下标；如果不在，返回0； str_list 用逗号分隔的字符串。
```sql
mysql> insert into votes values('Juse','登山',2);
Query OK, 1 row affected (0.07 sec)

mysql> select * from votes where find_in_set('登山', hobby);
+----------+-------------------+--------+
| username | hobby             | gender |
+----------+-------------------+--------+
| LiLiNaNa | 登山,黎狗子吃粑粑 | 男     |
| Juse     | 登山              | 女     |
+----------+-------------------+--------+
2 rows in set (0.09 sec)
```