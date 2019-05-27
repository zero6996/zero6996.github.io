---
title: MySQL多表查询和事务
date: 2019-5-23 18:00
categorise: MySQL笔记
tags: [MySQL,DQL多表查询,事务]
description: MySQL多表查询和事务概念
---



## 1. MySQL多表查询

当我们需要查询的数据在多张表时，就需要连接多张表查询，这就是多表查询。


<!--more-->


### 1.1 表连接查询

- 数据准备

```sql
CREATE DATABASE IF NOT EXISTS `db5` CHARACTER SET gbk;

-- 创建部门表
CREATE TABLE dept(
id INT PRIMARY KEY AUTO_INCREMENT,
NAME VARCHAR(50)
);

INSERT INTO dept(NAME) VALUES('开发部'),('市场部'),('财务部');

-- 创建员工表
CREATE TABLE staff(
id INT PRIMARY KEY AUTO_INCREMENT,
NAME VARCHAR(30),
gender CHAR(1), -- 性别
salary DOUBLE, -- 工资
join_date DATE, -- 入职日期
dept_id INT,
FOREIGN KEY(dept_id) REFERENCES dept(id) -- 外键
);

INSERT INTO staff(NAME,gender,salary,join_date,dept_id) VALUES
('孙悟空','男',7200,'2013-02-24',1),
('猪八戒','男',3600,'2010-12-02',2),
('唐僧','男',9000,'2008-08-08',2),
('白骨精','女',5000,'2015-10-07',3),
('蜘蛛精','女',4500,'2011-03-14',1);
```

- 多表查询的作用

如果想查询孙悟空的名字和他所在的部门名字，就需要使用多表查询。

- 多表查询的分类
  - 内连接：隐式内连接和显示内连接
  - 外连接：左外连接和右外连接



### 1.2 笛卡尔积现象

- 笛卡尔积：有两个集合A、B，取这两个集合中元素的所有组成情况。

使用`select * from staff,dept;`就会出现该现象

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/22/CartesianProduct-1558535089208.jpg)

- 如何消除笛卡尔积现象的影响

查询结果中不是所有的数据组合都是有用的，只有**员工表.dept_id=部门表.id**的数据才是有用的。所有需要通过设置条件过滤掉无用数据。

```sql
SELECT * FROM staff,dept WHERE staff.`dept_id`=dept.`id`;
-- 查询员工和部门的名字
SELECT staff.`name`,dept.`name` FROM staff,dept WHERE staff.`dept_id`=dept.`id`;
```



### 1.3 内连接

用左边表的记录去匹配右边表的记录，如果符合条件的则显示。如：**从表.外键=主表.主键**

#### 1.3.1 隐式内连接

- 隐式内连接：不使用join关键字，条件使用where指定。
- 语法：`select 字段名 from 左表,右表 where 条件;`

```sql
-- 查询两张表中，所有员工表外键等于部门表主键的数据
SELECT * FROM staff,dept WHERE staff.`dept_id`=dept.`id`;
```

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/22/in-connection01-1558535127200.jpg)
#### 1.3.2 显式内连接

- 显式内连接：使用**[inner] JOIN...ON**语句, '[ ]'表示可省略
- 语法：`select 字段名 from 左表 [inner] join 右表 on 条件;`

```sql
-- 查询钢铁侠的信息，显示员工id、姓名、性别、工资和所在的部门名称
SELECT s.`id`,s.`name`,s.`gender`,s.`salary`,d.`name`部门名称 FROM staff s INNER JOIN dept d ON s.`dept_id`=d.`id` AND s.`name`='钢铁侠';
/*
内连接查询步骤：
1. 确定查询哪些表：员工表和部门表
2. 确定表连接条件：员工表.dept_id = 部门表.id 的数据才是有效的，即消除笛卡尔积现象
3. 确定查询条件：查询钢铁侠的信息，员工表.name='钢铁侠'
4. 确定显示字段：显示钢铁侠的id、姓名、性别、工资和所在的部门名称
5. 可以使用别名进行优化，如staff as s
*/
```


![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/22/in-connection02-1558535144642.jpg)


### 1.4 外连接

#### 1.4.1 左外连接

用左边表的记录去匹配右边表的记录，如果符合条件的则显示；否则，显示 NULL

- 语法：`select 字段名 fron 左表 left[outer] join 右表 on 条件;`

可以理解为：在内连接的基础上保证左表的数据全部显示(左表部门，右表员工情况下)

```sql
-- 在部门表中增加一个销售部
INSERT INTO dept(NAME) VALUES('销售部');
-- 使用内连接查询数据
SELECT * FROM dept d INNER JOIN staff s ON d.`id`=s.`dept_id`;
-- 使用左外连接查询数据
SELECT * FROM dept d LEFT JOIN staff s ON d.`id`=s.`dept_id`;
```

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/22/LeftOuterJoin-1558535161078.jpg)

用左表记录去匹配右表记录，当不符合条件情况下，仍会显示左表数据，但右表数据会显示为NULL。即在内连接的基础上保证左表数据全部显示。

#### 1.4.2 右外连接

用右边表的记录去匹配左边表的记录，如果符合条件的则显示；否则，显示 NULL

- 语法：`select 字段名 from 左表 right[outer]join 右表 on 条件;`

可以理解为：在内连接的基础上保证右表的数据全部显示

```sql
-- 在员工表中增加一个员工
INSERT INTO staff VALUES(NULL,'蜘蛛侠','男',5880,'2014-4-1',NULL);
-- 使用内连接查询
SELECT * FROM dept d INNER JOIN staff s ON d.`id`=s.`dept_id`; -- 右表新添加的数据将查询不到，因为dept_id值为NULL。
-- 使用右外连接查询
SELECT * FROM dept d RIGHT JOIN staff s ON d.`id`=s.`dept_id`; -- 在内连接基础上，保证显示右表所有数据，即使条件不满足
```

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/22/RightOuterJoin-1558535183109.jpg)

## 2. 子查询

### 2.1 子查询概念

1. 一个查询的**结果**做为另一个查询的**条件**
2. 有查询的嵌套，内部的查询称为子查询
3. 子查询要**使用括号**

```sql
-- 查询开发部中有哪些员工
-- 通过两条语句查询
SELECT id FROM dept WHERE NAME='开发部'; -- 先获取开发部门id，1
SELECT * FROM staff WHERE dept_id=1; -- 通过id去查询对应员工
-- 使用子查询语句
SELECT * FROM staff WHERE dept_id = (SELECT id FROM dept WHERE NAME='开发部'); -- 注意子查询多行单列必须限定子查询字段列只有一个，不能使用* 或两个字段名
```

### 2.2 子查询结果的三种情况

1. 子查询的结果是单行单列
2. 子查询的结果是多行单列
3. 子查询的结果是多行多列

#### 2.2.1 子查询结果是一个值(单行单列)的时候

- 子查询结果只要是单行单列，肯定在 WHERE 后面作为条件，父查询使用：比较运算符，如：> 、<、<>、=等
- 语法：`select 查询显示字段 from 表 where 字段=(子查询);`

```sql
-- 查询工资最高的员工
SELECT staff.`name` FROM staff WHERE salary = (SELECT MAX(salary) FROM staff);
-- 查询工资小于平均工资的员工有哪些？
SELECT * FROM staff WHERE salary < (SELECT AVG(salary) FROM staff);
```

#### 2.2.2 子查询结果是多行单列的时候

- 子查询结果是多行单列，结果集类似于一个数组，父查询使用IN运算符
- 语法：`select 查询字段 form 表 where 字段 in (子查询);`

```sql
-- 查询工资大于5000的员工，来自于哪些部门的名字
-- 先查询大于5000的员工所在的部门id
SELECT dept_id FROM staff WHERE staff.`salary` > 5000;
-- 再查询这些部门id对应的部门名称,会报错
SELECT NAME FROM dept WHERE id = (SELECT dept_id FROM staff WHERE salary>5000); -- 错误代码： 1242 Subquery returns more than 1 row
-- 使用IN运算符
SELECT NAME FROM dept WHERE id IN(SELECT dept_id FROM staff WHERE salary>5000);

-- 查询开发部与财务部所有的员工信息
-- 先查询开发部和财务部的部门id
SELECT id FROM dept WHERE NAME IN ('开发部','财务部');
-- 通过部门id查询对应有哪些员工
SELECT * FROM staff WHERE dept_id IN (SELECT id FROM dept WHERE NAME IN ('开发部','财务部'));
```

#### 2.2.3 子查询结果是多行多列

子查询结果只要是多列，肯定在 FROM 后面作为表，子查询作为表需要**取别名**，否则这张表没有名称则无法访问表中的字段

- 语法：`select 查询字段 from (子查询) 表别名 where 条件;`

```sql
-- 查询出 2011 年以后入职的员工信息，包括部门名称
-- 在员工表中查询 2011-1-1 以后入职的员工
SELECT * FROM staff WHERE join_date >='2011-1-1';
-- 查询所有的部门信息，与上面的虚拟表中的信息组合，找出所有部门 id 等于的 dept_id
SELECT * FROM dept d, (SELECT * FROM staff WHERE join_date>='2011-1-1') e WHERE d.`id`=e.dept_id;
-- 也可以使用表连接查询
SELECT * FROM staff s INNER JOIN dept d ON  s.`dept_id`=d.`id` WHERE join_date>='2011-1-1';
SELECT * FROM staff s INNER JOIN dept d ON  s.`dept_id`=d.`id` AND join_date>='2011-1-1';
```

#### 2.2.4 子查询小结

- 子查询结果只要是单列，则在where后面作为条件
- 子查询结果只要是多列，则在from后面作为表进行二次查询



## 3. 事务

### 3.1 事务概述及应用场景

- **概念**：如果一个包含多个步骤的业务操作，被事务管理，那么这些操作要么同时成功，要么同时失败。
- 转账操作示例

```sql
-- 创建数据表
CREATE TABLE account(
id INT PRIMARY KEY AUTO_INCREMENT,
NAME VARCHAR(30),
balance DOUBLE);

-- 插入数据
INSERT INTO account(NAME,balance) VALUES('小张',1000),('小李',1000);
/*
	模拟小张给小李转500元钱，一个转账业务操作最少要执行下面两条语句：
	小张账户-500
	小李账户+500
*/
-- 小张账号-500
UPDATE account SET balance = balance-500 WHERE NAME='小张';
-- 小李账号+500
UPDATE account SET balance = balance+500 WHERE NAME='小李';
```

假设小张账户上-500后，服务器崩溃了。小李账户并没有+500元，如此数据就出现问题了。我们需要保证其中一条SQL语句出现问题，整个转账就算失败。因此需要使用事务。

### 3.2 手动提交事务

MySQL中可以有两种方式进行事务的操作：

1. 手动提交事务
2. 自动提交事务

#### 3.2.1 手动提交事务的SQL语句

| 功能     | SQL语句            |
| -------- | ------------------ |
| 开启事务 | start transaction; or begin;|
| 提交事务 | commit;            |
| 回滚事务 | rollback;          |

#### 3.2.2 手动提交事务使用过程

1. 执行成功的情况：开启事务--> 执行多条SQL语句--> 成功：提交事务
2. 执行失败的情况：开启事务--> 执行多条SQL语句--> 失败：事务的回滚

#### 3.2.3 案例演示

1. 模拟小张给小李转500元钱(成功情况)
   1. 使用DOS控制台进入MySQL
   2. 执行以下SQL语句：1. 开启事务;  2. 小张账户-500;  3. 小李账户+500
   3. 使用SQLyog查看数据库，发现数据未改变
   4. 在控制台执行commit;提交事务
   5. 示意SQLyog查看数据库，发现数据已改变

![transaction](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/transaction-1558592609612.jpg)

2. 模拟小张给小李转500元钱(失败情况)，两人账户都重置为1000元。
   1. 在控制台执行以下SQL语句：1. 开启事务;   2. 小张账户-500
   2. 使用SQLyog查看数据库，发现数据未改变
   3. 在控制台执行rollback回滚事务
   4. 使用SQLyog查看数据库，发现数据也未改变

![transaction-error](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/transaction-error-1558592664109.jpg)
#### 小结

如果事务中SQL语句没有问题，**commit提交事务**，会对数据库数据进行改变。如果事务中SQL语句有问题，**rollback回滚事务**，会回退到开启事务时的状态。



### 3.3 自动提交事务

MySQL 默认每一条 DML(增删改)语句都是一个单独的事务，每条语句都会自动开启一个事务，语句执行完毕自动提交事务，MySQL 默认开始自动提交事务。

![Auto_commit](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/Auto_commit-1558592689790.jpg)

#### 3.3.1 案例演示：自动提交事务

1. 将金额重置为1000
2. 更新其中某一个账户
3. 使用SQLyog查看数据库；发现数据已改变

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/demo01-1558592714256.jpg)

#### 3.3.2 取消自动提交

- 查看 MySQL 是否开启自动提交事务：`select @@autocommit;` @ 表示全局变量，1表示开启，0关闭。

![Auto_commit02](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/Auto_commit02-1558592732701.jpg)

- 取消自动提交事务

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/cancel_autocommit-1558592755217.jpg)

- 再次执行更新语句，然后使用SQLyog查看数据库，会发现数据并没有改变
- 在控制台执行commit提交任务


![transaction02](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/transaction02-1558592768439.jpg)


## 3.4 事务原理

事务开启之后，所有的操作都会临时保存到事务日志中，事务日志只有在得到commit命令才会同步到数据表中，其他任何情况都会情况事务日志(rollback,断开连接等)

#### 3.4.1 原理图

![theory](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/theory-1558592803406.jpg)

**事务的步骤：**

1. 客户端连接数据库服务器，创建连接时创建此用户临时日志文件
2. 开启事务后，所有的操作都会先写入到临时日志文件中
3. 所有的查询操作从表中查询，但会结果日志文件加工后才返回
4. 如果事务提交则将日志文件中的数据写入到表中，否则情况日志文件



### 3.5 回滚点

在某些成功的操作完成之后，后续的操作有可能成功有可能失败，但是不管成功还是失败，前面操作都已经成功，可以在当前成功的位置设置一个回滚点。可以供后续失败操作返回到该位置，而不是返回所有操作，这个点称之为**回滚点**。

#### 3.5.1 回滚点操作语句

| 作用       | 语句                 |
| ---------- | -------------------- |
| 设置回滚点 | savepoint 回滚点名字 |
| 回到回滚点 | rollback to 名字     |

#### 3.5.2 案例演示

1. 将数据重置为1000
2. 开启事务
3. 让小张账户减3次钱，每次10元
4. 设置回滚点：savepoint three_times;
5. 让小张减4次钱，每次10元
6. 回到回滚点：rollback to three_times;
7. 查看结果，分析执行过程

- 小结：设置回滚点可以让我们在失败的时候回到指定回滚点，而不是回到事务开启的时候。

### 3.6 事务的隔离级别

#### 3.6.1 事务四大特性ACID

| 事务特性            | 含义                                                         |
| ------------------- | ------------------------------------------------------------ |
| 原子性(Atomicity)   | 每个事务都是一个整体，不可再拆分，事务中所有的SQL语句要么都执行成功，要么都失败。 |
| 一致性(Consistency) | 事务在执行前数据库的状态与执行后数据库的状态保持一致。例：转账前两人的总金额是2000，转账后总金额也是2000 |
| 隔离性(Isolation)   | 事务与事务之间不应该相互影响，执行时保持隔离的状态           |
| 持久性(Durability)  | 一旦事务执行成功，对数据库的修改是持久的。就算关机，也是保存下来的。 |

#### 3.6.2 事务的隔离级别

事务在操作时的理想状态：所有的事务之间保持隔离，互不影响。因为并发操作，多个用户同时访问同一个数据，可以引发并发访问的问题

| 并发访问的问题 | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| 脏读           | 一个事务读取到了另一个事务中尚未提交的数据                   |
| 不可重复读     | 一个事务中两次读取的数据**内容不一致**，要求的是一个事务中多次读取时数据是一致的，这是事务update时引发的问题 |
| 幻读           | 一个事务中两次读取的数据的**数量不一致**，要求在一个事务多次读取的数据数量是一致的，这是insert或delete时引发的问题 |



#### 3.6.3 MySQL数据库有四种隔离级别

从上往下隔离级别依次递增。“是”表示会出现这种问题，“否”表示不会出现这种问题。

| 级别 | 名字     | 隔离级别         | 脏读 | 不可重复读 | 幻读 | 默认隔离级别                                       |
| ---- | -------- | ---------------- | ---- | ---------- | ---- | -------------------------------------------------- |
| 1    | 读未提交 | read uncommitted | 是   | 是         | 是   |                                                    |
| 2    | 读已提交 | read committed   | 否   | 是         | 是   | Oracle和SQL Server                                 |
| 3    | 可重复读 | repeatable read  | 否   | 否         | 是   | ![isolation](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/isolation-1558592857953.jpg) |
| 4    | 串行化   | serializable     | 否   | 否         | 否   |                                                    |

> 隔离级别越高，性能越差，安全性越高。


#### 3.6.4 MySQL事务隔离级别相关命令

- 查询全局事务隔离级别：`select @@ tx_isolation;`
- 设置事务隔离级别：`set global transaction isolation level 级别字符串;` 重新登录MySQL生效。

#### 3.6.5 脏读的演示

将余额数据重置为1000元

1. 打开A窗口登录MySQL，设置全局隔离级别为最低(read uncommitted)

![isolation-level1](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/isolation-level1-1558592887521.jpg)

2. 打开B窗口，AB窗口都开启事务

![AB](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/AB-1558592910534.jpg)

3. A窗口更新2个人的账户数据，模拟转账，未提交

![A-update](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/A-update-1558592932429.jpg)

4. B窗口查询账户数据，到账了

![B-select](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/B-select-1558592959485.jpg)

5. A窗口回滚：`rollback;`
6. B窗口查询账户数据，到账的钱没了

![B-select-2](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/B-select-2-1558592982344.jpg)

脏读问题非常危险。比如小张向小李购买商品，小张开启事务，向小李账户转入500块钱，然后打电话给小李说钱已经转了。小李查询看钱的确到账了，发货给小张。小张收到货后回滚事务，小李再次查看钱就没了。

**解决脏读问题**：将全局的隔离级别进行提升

1. 提高A窗口的隔离级别为read committed，并开启事务

![isolation-level2](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/isolation-level2-1558593001591.jpg)

2. 重新登录B窗口MySQL，进入数据库并开启事务

![B-usedb](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/B-usedb-1558593015378.jpg)

3. A窗口模拟转账操作，更新两个账户数据，未提交

![A-level2](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/A-level2-1558593029364.jpg)

4. B窗口查询数据，发现没有改变(没有读取到另一个事务未提交的内容)

![B-select-level2](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/B-select-level2-1558593046721.jpg)

5. A窗口`commit;`提交事务，B窗口查询，发现数据改变了(另一个事务提交后的数据才能读取到)

![A_commit_B_select](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/A_commit_B_select-1558593059422.jpg)

**小结**：read committed的方式可以避免脏读的发生

### 3.6.6 不可重复读的演示

将数据重置为1000

1. 开启A窗口，并设置全局隔离级别为read committed；

![A-set-readcommitted](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/A-set-readcommitted%3B-1558593079757.jpg)

2. 开启B窗口，B窗口开启事务并查询数据

![B-startAndselect](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/B-startAndselect-1558593106316.jpg)

3. A窗口开启事务，并更新数据

![A-update-commit](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/A-update-commit-1558593126721.jpg)

4. B窗口查询数据，发现两次结果不一致

![B-select-02](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/B-select-02-1558593149558.jpg)

两次查询输出的结果不同，无法确定哪次是对的，以哪次为准？

**解决不可重复读的问题**：将全局隔离级别提升为：**repeatable read**

将数据重置为1000

1. A窗口设置隔离级别为：**repeatable read**

![A_repeatable_read](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/A_repeatable_read-1558593166460.jpg)

2. B窗口重新登录MySQL，开启事务并查询数据

![B_select_level3](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/B_select_level3-1558593179684.jpg)

3. A窗口开启事务，更新数据

![A_update_level3](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/A_update_level3-1558593195882.jpg)

4. B窗口查询数据，两次查询数据一致

![B_select_level3_02](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/B_select_level3_02-1558593214294.jpg)

#### 小结

同一个事务中为了保证多次查询数据一致，必须使用**repeatable read**隔离级别

![level3](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/level3-1558593223717.jpg)


### 3.7 幻读的演示

- 幻读主要是说多次读取一个范围内的记录(包括直接查询所有记录结果或者做聚合统计)，发现结果不一致(标准档案一般指记录增多，记录的减少应该也算是幻读)。
- 其实对于 `幻读`, MySQL的InnoDB引擎默认的`RR`级别已经通过`MVCC自动帮我们解决了`, 所以该级别下, 也模拟不出幻读的场景; 退回到 `RC` 隔离级别的话,又容易把`幻读`和`不可重复读`搞混淆。理论上RR级别是无法解决幻读问题的，但是由于InnoDB引擎的RR级别还使用了MVCC, 所以也就避免了幻读的出现。

- 使用`隔离性`的最高隔离级别`SERIALIZABLE`也可以解决`幻读`, 在该隔离级别下，一个事务没有执行完，其他事务的SQL执行不了，可以挡住幻读。但该隔离级别在实际中很少使用！

- [幻读的延伸](https://segmentfault.com/a/1190000012669504?utm_source=tag-newest)



## 4. DCL(Data Control Language)

- DDL：create/alter/drop
- DML：insert/ update/delete
- DQL：select/show
- DCL：grant/ revoke

默认使用的都是root用户，超级管理员，拥有全部的权限。但是，一个公司里面的数据库服务器上面可能同时运行着很多个项目的数据库。所以，我们应该可以根据不同的项目建立不同的用户，分配不同的权限来管理和维护数据库。

> mysqld是MySQL的主程序，服务器端。mysql是MySQL的命令行工具，客户端。


### 4.1 创建用户

- 语法：`create user '用户名'@'主机名' identified by '密码';`

- 关键字说明

| 关键字 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 用户名 | 将创建用户名称                                               |
| 主机名 | 指定该用户在哪个主机上可以登录，如果是本地用户可用localhost，如果想让该用户可以从任意远程主机登录，可以使用通配符% |
| 密码   | 该用户的登录密码，可以为空(则表示无需密码即可登录服务器)     |

- 操作示例

```sql
-- 创建 zero 用户，只能在 localhost 这个服务器登录 mysql 服务器，密码为 123
create user 'zero'@'%' identified by '123';
-- 创建 zero2用户可以在任何电脑上登录 mysql 服务器，密码为 123
CREATE USER 'zero2'@'%' IDENTIFIED BY '123';
-- 创建的用户名都在mysql数据库的user表中可以查看,密码经过了加密
select user.`Host`,user.`User`,user.`authentication_string`PASSWORD FROM USER;
```

![users](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/23/users-1558605540361.jpg)

### 4.2 用户授权

用户创建之后，什么权限都没有，需要管理员手动给用户授权

- 语法：`grant 权限1,权限2...on 数据库名.表名 to '用户名'@'主机名';`
- 关键字说明

| 关键字            | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| grant...on...to   | 授权关键字                                                   |
| 权限              | 授予用户的权限，如CREATE、ALTER、SELECT、INSERT、UPDATE等。如果要授予所有的权限则使用ALL |
| 数据库名.表名     | 该用户可以操作哪个数据库的哪些表。如果要授予该用户对**所有数据库和表**的相应操作权限则可以使用`*.*` |
| '用户名'@'主机名' | 给哪个用户授权。注意有单引号                                 |

- 操作示例

```sql
-- 给 zero 用户分配对 test 这个数据库操作的权限：创建表，修改表，插入记录，更新记录，查询
GRANT CREATE,ALTER,INSERT,UPDATE,SELECT ON db1.* TO 'zero'@'localhost';
-- 给 zero2 用户分配所有权限，对所有数据库的所有表
GRANT ALL ON *.* TO 'zero2'@'%';
```



### 4.3 撤销权限

- 语法：`revoke 权限1,权限2... on 数据库.表名 from '用户名'@'主机名';`

- 关键字

| 关键字             | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| revoke...on...from | 撤销授权的关键字                                             |
| 权限               | 用户的权限，如 CREATE、ALTER、SELECT、INSERT、UPDATE 等，所有的权限则使用 ALL |
| 数据库名.表名      | 对哪些数据库哪些表做操作。如果要撤销该用户对**所有数据库和表**的相应操作权限则可以使用`*.*` |
| '用户名'@'主机名'  | 给哪个用户撤销                                               |

- 操作示例

```sql
-- 撤销zero2用户对全部数据库的全部权限
REVOKE ALL ON *.* FROM 'zero2'@'%';
```

### 4.4 查看权限

- 语法：`show grants for '用户名'@'主机名';`
- 操作示例

```sql
-- 查看zero用户的权限
SHOW GRANTS FOR 'zero'@'localhost';
```

> usage是指连接(登录)权限，建立一个用户，就会自动授予其usage权限，无法revoke。



### 4.5 删除用户

- 语法：`drop user '用户名'@'主机名';`

- 操作示例

```sql
-- 删除zero用户
DROP USER 'zero'@'localhost';
```



### 4.6 修改管理员密码

- 语法：`mysqladmin -uroot -p password 新密码`

> 该命令在DOS控制台执行，新密码无需加上引号。
>
> Tips: 
>
> 1. 请在DBA允许陪同下使用该命令
> 2. 数据库内修改root密码需使用`flush privileges;`刷新权限才能生效

### 4.7 修改普通用户密码

- 语法：`set password for '用户名'@'主机名'=password('newpwd');`




## 今日目标

* [x] 能够使用内连接进行多表查询
* [x] 能够使用左外连接和右外连接进行多表查询
* [x] 能够使用子查询进行多表查询
* [x] 能够理解事务的概念
* [x] 能够说出事务的特点
* [x] 能够在MySQL中使用事务
* [x] 能够理解脏读、不可重复读、幻读的概念及解决方法
* [x] 能够使用DCL管理MySQL中的用户



