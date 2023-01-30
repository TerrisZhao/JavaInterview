# mysql

### acid 描述以及mysql如何实现以上特性

```
原子性 （atomicity）:强调事务的不可分割. 
一致性 （consistency）:事务的执行的前后数据的完整性保持一致. 
隔离性 （isolation）:一个事务执行的过程中,不应该受到其他事务的干扰 
持久性（durability） :事务一旦结束,数据就持久到数据库


实现：


原子性：
当事务对数据库进行修改时，InnoDB会生成对应的 undo log；如果事务执行失败或调用了 rollback，导致事务需要回滚，便可以利用 undo log 中的信息将数据回滚到修改之前的样子。undo log 属于逻辑日志，它记录的是sql执行相关的信息。当发生回滚时，InnoDB 会根据 undo log 的内容做与之前相反的工作：


对于每个 insert，回滚时会执行 delete；
对于每个 delete，回滚时会执行insert；
对于每个 update，回滚时会执行一个相反的 update，把数据改回去。
以update操作为例：当事务执行update时，其生成的undo log中会包含被修改行的主键(以便知道修改了哪些行)、修改了哪些列、这些列在修改前后的值等信息，回滚时便可以使用这些信息将数据还原到update之前的状态。


一致性：
一致性是事务追求的最终目标，原子性、持久性和隔离性，其实都是为了保证数据库状态的一致性。


隔离性：
四个隔离级别：
读未提交：未提交，被其它事物读到（脏读，不可重复读，幻读）
读已提交：提交后，才能被读到（不可重复读，幻读）
可重复读：一个事物中，无论是否被变更，读到都一样（幻读）
串行化：读写 串行化


持久性：Innnodb 持久性靠的是 redo log

```



### mvcc实现原理 结合具体场景

```
使用场景 在RR模式下， 两个事务（A、B），
先启动事物A，查询 select name from table where id=1,得到结果 a;
再启动事物B，修改 update table set name='new value' where id=1;
再在A事务中查询得到结果a`;
B事务提交，再次在A事务中查询得到结果a``;
三次结果分别是什么，为什么？引出下面的东西


MVCC 多版本并发控制，主要以row trx_id,undo log,read view来实现。
row trx_id
    InnoDB 里面每个事务有一个唯一的事务 ID，叫作 transaction id。它是在事务开始的时候向 InnoDB 的事务系统申请的，是按申请顺序严格递增的。而每行数据也都是有多个版本的。每次事务更新数据的时候，都会生成一个新的数据版本，并且把 transaction id 赋值给这个数据版本的事务 ID，记为 row trx_id。同时，旧的数据版本要保留，并且在新的数据版本中，能够有信息可以直接拿到它。
undo log 
    Undo log 就用于对两个版本的连接
read-view
    InnoDB 为每个事务构造了一个数组，用来保存这个事务启动瞬间，当前正在“活跃”的所有事务 ID。“活跃”指的就是，启动了但还没提交。数组里面事务 ID 的最小值记为低水位，当前系统里面已经创建过的事务 ID 的最大值加 1 记为高水位。这个视图数组和高水位，就组成了当前事务的一致性视图（read-view）。


引出undo log日志

```



### undo/redo/binlog区别

```
bin log
bin log中记录的是整个mysql数据库的操作内容，对所有的引擎都适用，包括执行的DDL、DML，可以用来进行数据库的恢复及复制。
redo log
redo log中记录的是要更新的数据，比如一条数据已提交成功，并不会立即同步到磁盘，而是先记录到redo log中，等待合适的时机再刷盘，为了实现事务的持久性
undo log
undo log中记录的是当前操作中的相反操作，一条insert语句在undo log中会对应一条delete语句，update语句会在undo log中对应相反的update语句，在事务回滚时会用到undo log，实现事务的原子性，同时会用在MVCC中，undo中会有一条记录的多个版本，用在快照读中；
redo log和binlog区别
	redo log是属于innoDB层面，binlog属于MySQL Server层面的，这样在数据库用别的存储引擎时可以达到一致性的要求。
	redo log是物理日志，记录该数据页更新的内容；binlog是逻辑日志，记录的是这个更新语句的原始逻辑
	edo log是循环写，日志空间大小固定；binlog是追加写，是指一份写到一定大小的时候会更换下一个文件，不会覆盖。
	binlog可以作为恢复数据使用，主从复制搭建，redo log作为异常宕机或者介质故障后的数据恢复使用。

```



### binlog 基础

```
binlog类型，使用场景
```



### 执行计划 type(const/eq_ref/ref/range/all)， extra：filesort, using temp

```
type：连接类型 最关键的一列 效率（const>eq_ref>ref>range>index>all）
找样例sql ，候选人说明type是什么类型 todo 


1、const:查询索引字段，并且表中最多只有一行匹配（好像只有主键查询只匹配一行才会是const，有些情况唯一索引匹配一行会是ref）
2、eq_ref 主键或者唯一索引
3、ref 非唯一索引（主键也是唯一索引）
4、range 索引的范围查询
5、index (type=index extra = using index 代表索引覆盖，即不需要回表)
6、all 全表扫描（通常没有建索引的列）


高级 
index merge @邵洋 todo


extra
1、using temporary(组合查询返回的数据量太大需要建立一个临时表存储数据,出现这个sql应该优化)
2、using where (where查询条件)
3、using index(判断是否仅使用索引查询，使用索引树并且不需要回表查询)
4、using filesort(order by 太占内存，使用文件排序) 什么场景下会用到文件排序
```



### 索引覆盖，回表，相关优化手段和经验

```
回表：如果索引的列在 select 所需获得的列中（因为在 mysql 中索引是根据索引列的值进行排序的，所以索引节点中存在该列中的部分值）或者根据一次索引查询就能获得记录就不需要回表，如果 select 所需获得列中有大量的非索引列，索引就需要到表中找到相应的列的信息，这就叫回表
索引覆盖：只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快
常见的方法是：将被查询的字段，建立到联合索引里去
基于非主键索引的查询需要多扫描一棵索引树。因此，我们在应用中应该尽量使用主键查询。


如何创建有效的索引
如果需要索引很长的字符串，此时需要考虑前缀索引
使用多列索引，尽量不要为多列上创建单列索引，因为这样的情况下最多只能使用一星索引
选择合适的索引列顺序 	经验是将选择性最高的列放到索引最前列，可以在查询的时候过滤出更少的结果集
覆盖索引 所谓覆盖索引就是指索引中包含了查询中的所有字段，这种情况下就不需要再进行回表查询了

```



### select * select id 区别

```
回表
```



### 使用不上索引的情况

```
1、like 百分号开头
2、where 带 or
3、索引列 做计算，函数
4、字符串 int 转型（隐式转换）
5、!= 有时候 全表扫描
6、类似是否，01的值，加索引不生效
7、有些使用全表扫描比索引快的时候
8、关联查询 编码不一致
9、联合索引 第一个字段不用
```



### select count(*) count(列) 实现原理区别，执行效率区别

```
count(1)包括了忽略所有列，用1代表代码行，在统计结果的时候，不会忽略列值为NULL
count(列名)只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是只空字符串或者0，而是表示null）的计数，即某个字段值为NULL时，不统计。列名为主键，count(列名)会比count(1)快 。列名不为主键，count(1)会比count(列名)快。如果表多个列并且没有主键，则 count（1） 的执行效率优于 count（*） 。如果有主键，则 select count（主键）的执行效率是最优的。如果表只有一个字段，则 select count（*）最优。


count(*)优化思路
建立类型小的二级索引
```



### 索引覆盖

```
name查age


select * from candidates where name = xx
name 普通索引


如何提升效率


1.改成select age


2.增加联合索引 name age


3.去掉name索引


这样age通过联合索引就能拿到，也就是索引覆盖

```



### 联合索引

```
select id from t where a=1 order by b 


a b 单列索引


初级 --
1 b索引是否能用得到 ？ 不能
2 如果进一步让排序使上索引 ？设置联合索引


中级--
3 a=1 改为 a>1 ，索引使用情况是什么样的 原因是什么？ 只能使上a的联合索引， 联合索引的存储结构导致
```



### 死锁case

```
1.Session1:
mysql> select * from t3 where id in (8,9) for update;
Session2:
select * from t3 where id in (10,8,5) for update;
锁等待中……




2.根据字段值查询（有索引），如果不存在，则插入；否则更新。
以id为主键为例，目前还没有id=22的行


Session1:
select * from t3 where id=22 for update;
Empty set (0.00 sec)
session2:
select * from t3 where id=23  for update;
Empty set (0.00 sec)
Session1:
insert into t3 values(22,'ac','a',now());
锁等待中……
当对存在的行进行锁的时候(主键)，mysql就只有行锁。
当对未存在的行进行锁的时候(即使条件为主键)，mysql是会锁住一段范围（有gap锁）
3
mysql> select * from t3 where id=9 for update;
+----+--------+------+---------------------+
| id | course | name | ctime               |
+----+--------+------+---------------------+
|  9 | JX     | f    | 2016-03-01 11:36:30 |
+----+--------+------+---------------------+


row in set (0.00 sec)
Session2:
mysql> select * from t3 where id<20 for update;
锁等待中
```



### 排查死锁步骤

```
死锁排查步骤
1. 线上错误日志报警发现死锁异常
2. 查看错误日志的堆栈信息
3. 查看 MySQL 死锁相关的日志
4. 根据 binlog 查看死锁相关事务的执行内容
5. 根据上述信息找出两个相互死锁的事务执行的 SQL 操作，根据本系列介绍的锁相关理论知识，进行分析推断死锁原因
6. 修改业务代码


1、查询是否锁表
 show OPEN TABLES where In_use > 0;
2、查询进程
 show processlist;
查询到相对应的进程,然后
kill pid 或者找到被锁的表：UNLOCK TABLES; 查看正在锁的事务
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS; 
查看等待锁的事务
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
锁表 锁定数据表，避免在备份过程中，表被更新
LOCK TABLES tbl_name READ;
为表增加一个写锁定：
LOCK TABLES tbl_name WRITE;
```



### gap锁锁定范围举例（唯一索引等值查询，非唯一索引等值查询）

```
1、唯一索引 等值查询 如果查询的值存在，升级为行锁，如果查询的值不存在，会锁住数据前后的范围锁（左开右开）（如果两个事物更新的值都不存在，会造成死锁）
2、非唯一索引 等值查询 会锁住数据前后的范围锁（左开右开）
3、唯一索引 范围查询 会锁住查询精确范围（左闭右闭）
4、非唯一索引 范围查询 会锁住数据前后的范围（左开右开）
```



### gap锁锁定的操作是啥（update/delete/insert）

```
1.insert：阻塞插入到间隙的操作
2.update：以更新的方式set数据到间隙的操作
```



### 包含gap锁的死锁例子

```
例子：table trole, 两个属性，(id,level), id主键，level普通索引，表中数据
(10，1),（20，2），（30，4），（60，5），（80，5），（100，5），（130，11）
Session1: start transaction;
Session1: select * from trole where level=4 for update;
Session2: start transaction;
Session2: select * from trole where level=11 for update;
Session1: update trole set level = 6 where level = 1;--阻塞
Session2: insert into trole value(21,2);--死锁


本地测试结果：
Session1:1 row(s) affected Rows matched: 1  Changed: 1  Warnings: 0；
Session2:Error Code: 1213. Deadlock found when trying to get lock; try restarting transaction
```



### 乐观悲观结合一个场景

```
伪代码  并发场景下存在的问题


Order order = orderService.getById(100);


if (order.getStatus().equals("待支付")) {
	order.setStatus("已支付");
}


orderService.update(order);




乐观锁：


订单表 id = 100  status = '待支付' 如何通过乐观锁的方式 修改为 已支付 并发场景下 


有些候选人 想到 加版本号的方式 ，实际上 不用加版本号 status 就相当于版本号


update order set status = '待支付' where status = '已支付' and id = 100




悲观锁：


select * from order where id = 100 for update 

```



### 行锁，表锁触发场景

```
行锁触发索引，表锁未触发索引， 举场景todo
```



### for update排他锁产生的条件

```
1、只根据主键进行查询，并且查询到数据，主键字段产生行锁。
2、只根据主键进行查询，没有查询到数据，不产生锁。
3、根据主键、非主键含索引进行查询，并且查询到数据，主键字段产生行锁，查询字段产生行锁。
4、根据主键、非主键含索引进行查询，没有查询到数据，不产生锁。
5、根据主键、非主键不含索引进行查询，并且查询到数据，如果其他线程按主键字段进行再次查询，则主键字段产生行锁，如果其他线程按非主键不含索引字段进行查询，则非主键不含索引字段产生表锁，如果其他线程按非主键含索引字段进行查询，则非主键含索引字段产生行锁，如果索引值是枚举类型，mysql也会进行表锁，这段话有点拗口，大家仔细理解一下。
6、根据主键、非主键不含索引进行查询，没有查询到数据，不产生锁。
7、根据非主键含索引进行查询，并且查询到数据，查询字段产生行锁。
8、根据非主键含索引进行查询，没有查询到数据，不产生锁。
9、根据非主键不含索引进行查询，并且查询到数据，查询字段产生表锁。
10、根据非主键不含索引进行查询，没有查询到数据，查询字段产生表锁。
11、只根据主键进行查询，查询条件为不等于，并且查询到数据，主键字段产生表锁。
12、只根据主键进行查询，查询条件为不等于，没有查询到数据，主键字段产生表锁。
13、只根据主键进行查询，查询条件为 like，并且查询到数据，主键字段产生表锁。
14、只根据主键进行查询，查询条件为 like，没有查询到数据，主键字段产生表锁。
```



