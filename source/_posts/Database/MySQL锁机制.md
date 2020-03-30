---
title: MySQL锁机制
date: 2020-03-29 15:00:00
categories: DataBase
tags: MySQL Lock
urlname: Lock
---



# 1. MySQL锁机制

锁是计算机协调多个进程或线程并发访问某一资源的机制。

<!--more-->

在数据库中，除传统的计算资源(如CPU、RAM、I/O等)的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤为重要，也更加复杂。

## 1.1 锁的分类

### 1.1.1 按对数据操作的类型分

- 读锁(**共享锁**)：其他事务可以读，但不能写。
- 写锁(**排它锁**)：其他事务不能读取，也不能写。

### 1.1.2 按对数据操作的粒度分

MySQL不同存储引擎支持不同的锁机制。

- 表锁：就是一锁就锁一整张表，表锁定期间，其他事务不能对该表进行操作。
- 行锁：就是一锁就锁一行或多行记录。



# 2. 三锁

## 2.1 表锁(偏读)

偏向MyISAM存储引擎；表级锁开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。表级锁更适合于查询为主，并发用户小，只有少量按索引条件更新数据的应用，如Web应用。

关于锁操作基本命令：

- 查看表是否上锁(1代表上锁)：`show open tables;`
- 解锁：`unlock tables;`

### 2.1.1 建表

以下操作要在两个会话中操作，分别是会话1和会话2。

- 会话1建表

```sql
CREATE TABLE mylock(
  id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR(20)
)ENGINE MYISAM;

INSERT INTO mylock(NAME) VALUES
('a'),
('b'),
('c'),
('d'),
('e');
```

### 2.1.2 加读锁

- 会话1(当前)加读锁，开启会话2(其他)

```sql
LOCK TABLE mylock READ;
```

- 当前会话可以查询该表，其他会话也可查询该表

```sql
SELECT * FROM mylock;
```

- 当前会话无法查询其他未锁表，其他会话正常查询或更新其他未锁表

```sql
SELECT * FROM book; /*会话1查询未锁表会报错：ERROR 1100 (HY000): Table 'book' was not locked with LOCK TABLES*/
select * from book;/*会话2查询未锁表正常*/
```

- 当前会话无法更新该表，其他会话更新或插入操作会阻塞等待，直到会话1释放锁(`unlock tables;`)

```sql
UPDATE mylock SET NAME='a2' WHERE id=1; /*当前会话更新会报错：ERROR 1099 (HY000): Table 'mylock' was locked with a READ lock and can't be updated*/
UPDATE mylock SET NAME='a2' WHERE id=1; /*其他会话操作会一直阻塞等待会话1释放锁*/
```

### 2.1.3 加写锁

- 会话1加写锁，开启会话2

```sql
lock table mylock write;
```

- 当前会话正常查询、修改该表，不可查询其他未锁表

```sql
select * from mylock;
update mylock set name='a2' where id=1;
select * from book; /*报错*/
```

- 会话2查询或者修改操作都会阻塞等待释放锁。

```sql
select * from mylock; # 其他查询会阻塞等待释放锁。
update mylock set name='b2' where id=2; # 其他操作会阻塞
```



### 2.1.4 表锁小结

`MyISAM`在执行查询语句(SELECT)之前，会自动给涉及的所有表加读锁；在执行增删改操作前，会自动给涉及的表加写锁。

MySQL的表级锁有两种模式：

- 表共享读锁(Table Read Lock)
- 表独占写锁(Table Write Lock)

结合上述案例，对MyISAM表进行操作，有如下结论：

1. 对MyISAM表的读操作(加读锁)，不会阻塞其他进程对同一表的读请求，但会阻塞对同一表的写请求；只有当读锁释放后，才会执行其他进程的写操作。
2. 对MyISAM表的写操作(加写锁)，会阻塞其他进程对同一表的读和写操作；只有当写锁释放后，才会执行其他进程的读写操作。

> <font color=red>简言之：读锁可读不可写；写锁读写会阻塞。</font>

### 2.1.5 表锁分析

可以使用SQL：`show status like "table%";`来查看表锁定状态，分析系统上的表锁定。下面是两个较为重要的状态变量：

- `Table_locks_immediate`：产生表级锁定的次数，表示可以立即获得锁的查询次数，每立即获取锁值加1。
- `Table_locks_waitd`：出现表级锁定争用而发生等待的次数(不能立即获取锁的次数，每等待一次锁值加1)，此值高则说明存在着较严重的表级锁争用情况。

> MyISAM的读写锁调度是写优先的，因此MyISAM不适合做写为主的表的引擎。因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，造成永久堵塞。



## 2.2 行锁(偏写)

偏向InnoDB存储引擎；行级锁开销大，加锁慢；会出现死锁；锁定粒度最小，发送锁冲突概率最低，并发度最高。最大程度的支持并发，同时也带来最大的锁开销。

InnoDB与MyISAM最大的不同有两点：

1. InnoDB是支持事务的。
2. InnoDB采用了行级锁。

### 2.2.1 建表

```sql
CREATE TABLE test_innodb_lock(
    a INT(11),
    b VARCHAR(16)
)ENGINE=INNODB;

INSERT INTO test_innodb_lock VALUES
(1,'b2'),
(3,'3'),
(4,'4000'),
(5,'5000'),
(6,'6000'),
(7,'7000'),
(8,'8000'),
(9,'9000'),
(1,'b1');

CREATE INDEX test_innodb_a_idx ON test_innodb_lock(a);
CREATE INDEX test_innodb_b_idx ON test_innodb_lock(b);
```

### 2.2.2 行锁演示

- 开启两个会话，会话1和会话2，分别设置关闭自动提交

```sql
SHOW VARIABLES LIKE "autocommit"
SET autocommit = 0;
```

- 会话1更新但不提交

```sql
UPDATE test_innodb_lock SET b='4001' WHERE a=4;
```

- 会话2更新同字段将被阻塞等待

```sql
UPDATE test_innodb_lock SET b='4001' WHERE a=4;
```

- 会话1提交，会话2更新操作才会执行。

```sql
commit;
```

- 如果会话1更新字段和会话2不同，那么不会阻塞操作请求

```sql
/*会话1操作*/
UPDATE test_innodb_lock SET b='5001' WHERE a=5;
/*会话2操作*/
UPDATE test_innodb_lock SET b='9001' WHERE a=9;
```

> 注意：操作更新完成后，必须提交才能使其他会话看到更新记录；这是因为默认的隔离级别，可重复读的。

### 2.2.3 行锁索引失效升级为表锁

- 会话1执行以下语句

```sql
 update test_innodb_lock set a=40 where b=4001; /*注意此处b字段应该加单引号，故意为加，MySQL将会做自动类型转换，导致索引失效*/
```

- 会话2执行非同字段更新操作，会发现被阻塞了

```sql
update test_innodb_lock set b='9002' where a=9；
```

- 会话1执行commit操作后，会话2才结束阻塞，正常执行。

```sql
commit;
```

- 可以发现，在索引失效或者没有索引的情况下，行锁是会升级为表锁的，导致其他进程阻塞无法操作。

### 2.2.4 间隙锁

当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做"**间隙(GAP)**"；InnoDB也会对这个“间隙”加锁，这种锁机制就是**间隙锁(Next-Key)锁**。下面开始间隙锁案例：

- 查表可知表中没有a=2的记录，会话1做范围更新操作

```sql
update test_innodb_lock set b='0329' where a>1 and a<6;
```

- 会话2做插入操作，插入a=2的数据。

```sql
insert into test_innodb_lock values(2,'2000');
```

可以看到会话2的插入操作被阻塞了，只有会话1提交后才能继续执行，这就是间隙锁。

因为Query在执行过程中通过范围查找的话，会锁定整个范围内的所有索引键值，即使这个键值并不存在；间隙锁的致命弱点就是，当锁定一个范围键值后，即使某些不存在的键值也会被无辜的锁定，从而造成在锁定的时候无法插入锁定键值范围内的任何数据；在某些场景下可能会对性能造成很大的危害。

### 2.2.5 面试题：如何锁定一行？

案例如下：

- 会话1操作

```sql
begin;
select * from test_innodb_lock where a = 8 for update;
```

- 会话2操作更新将会阻塞

```sql
update test_innodb_lock set b='8888' where a=8;
```

- 直到会话1做提交操作在能执行成功

```sql
commit;
```

如上使用`select xxx... for update`锁定某一行后，其他的操作会被阻塞，直到锁定行的会话提交commit。

### 2.2.6 行锁小结

InnoDB存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会更高一些，但是在整体并发处理能力方面要远远优于MyISAM的表级锁定的。

当系统并发量较高时，InnoDB的整体性能和MyISAM相比就会有比较明显的优势了。但是InnoDB的行级锁定同样有其劣势的一面，当我们使用不当时，可能会让InnoDB的整体性能表现反而比不上MyISAM。

### 2.2.7 行锁分析

通过`show status like "innodb_row_lock%";`命令检查`InnoDB_row_lock`状态变量来分析系统上的行锁争夺情况。各个状态量说明：

- `Innodb_row_lock_current_waits`：当前正在等待锁定的数量；
- `Innodb_row_lock_time`：从系统启动到现在锁定总时间长度；
- `Innodb_row_lock_time_avg`：每次等待所花平均时间；
- `Innodb_row_lock_time_max`：从系统启动到现在等待最长一次所花时间；
- `Innodb_row_lock_waits`：系统启动后到现在总共等待的次数。

上述5个状态变量中，比较重要的是这三个：

- `Innodb_row_lock_time`：等待总时长
- `Innodb_row_lock_time_avg`：平均等待时长
- `Innodb_row_lock_waits`：等待总次数

尤其是当等待次数很高，且每次等待时长也不小时，就需要分析系统中为什么有如此多的等待了，然后根据分析结果制定优化计划。

### 2.2.8 行锁优化建议

- 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁
- 合理设计索引，尽量缩小锁的范围
- 尽可能较少检索条件，避免间隙锁
- 尽量控制事务大小，减少锁定资源量和时间长度
- 尽可能低级别事务隔离



## 2.3 页锁

页面锁开销和加锁时间介于表锁和行锁之间；会出现死锁；锁定粒度介于表锁和行锁之间，并发度一般。

