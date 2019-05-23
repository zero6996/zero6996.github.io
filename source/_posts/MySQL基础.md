---
title: MySQL基础
date: 2019-5-18 23:40
categories: MySQL基础 # 分类
tags: [MySQL]
description: MySQL基本使用
---



## 1. 数据库基本概念


数据库(DataBase)，简称DB，用于存储和管理数据的仓库。

数据库的特点：

1. 持久化存储数据。数据库其实就是一个文件系统。
2. 方便存储和管理数据。
3. 使用了统一的方式操作数据库。



<!--more-->



[安装MySQL](http://note.youdao.com/noteshare?id=3490275e3fa4e6d37fc8bedd3794aa1f&sub=7A387FC8255F4099AB24DCF98198C0E3)

[MySQL基本操作](http://note.youdao.com/noteshare?id=fe0558c8d90f7a4a831785455a30a8ac&sub=WEB2676b40c253e961e9a2574607287e12f)

> Tips：在MySQL中，数据库等于文件夹，表等于文件，数据就是数据。



### 2. SQL语句

Structured Query Language(结构化查询语句)，简称SQL。

其实就是定义了操作所有关系型数据库的规则。每一种数据库操作的方式存在异同的地方。

####  2.1 SQL通用语法

1. SQL语句可以单行或多行书写，以分号结尾。
2. 可以使用空格和缩进来增强语句的可读性。
3. MySQL数据库的SQL语句不区分大小写，关键字建议使用大写。
4. 注释方式：
   * 单行注释：-- 注释内容 或 # 注释内容(MySQL特有)
   * 多行注释：/\*  注释内容  \*/



### 2.2 SQL分类

1. 数据定义语言(Data Definition Language)，简称**DDL**。用来定义数据库对象：数据库、表、列等。关键字：create、drop、alter等。
2. 数据操作语言(Data Manipulation Language)，简称**DML**。用来对数据库中表的数据进行增删改。关键字：insert、delete、update等。
3. 数据查询语言(Data Query Language)，简称**DQL**。用以查询数据库中表的记录(数据)。关键字：select，where等。
4. 数据控制语言(Data Control Language)，简称**DCL**。用来定义数据库的访问权限和安全级别，以及创建用户。关键字：GRANT,REVOKE等。



#### DDL：操作数据库、表

**操作数据库： CRUD**

##### 1. C(Create):创建
   * 创建数据库：`create databases 数据库名称;`
   * 创建数据库，判断不存在才创建：`create database if not exists 数据库名称;`
   * 创建数据库，指定字符集：`create database 数据库名称 character set 字符集名;`
   * 例：创建db4数据库，判断是否存在，并指定字符集为gbk：`create database if not exists dbs character set gbk;`

##### 2. R(Retrieve):查询
   * 查询所有数据库名称：`show databases;`
   * 查询某个数据库的创建语句和字符集：`show create database 数据库名称;`

##### 3. U(Update):修改
   * 修改数据库的字符集:`alter database 数据库名称 character set 字符集名称;`

##### 4. D(Delete):删除
   * 删除数据库：`drop database 数据库名称;`
   * 判断数据库存在才删除：`drop database if exists 数据库名称;`

##### 5. 使用数据库
   * 查询当前正在使用的数据库名称：`select database();`
   * 使用数据库：`use 数据库名称;`

**操作表：CRUD**

##### 1. C(Create)：创建

```sql
create table 表名(
	列名1 数据类型1,
    列名2 数据类型2,
    ... ...
    列名n 数据类型n); # 注意：最后一列，不需要加逗号','
/*
    常用数据库类型:
    1. int：整数类型。例：`age int`
    2. double：小数类型。例：`score double(5,2)`
    3. date：日期(年月日)。例：`date yyyy-MM-dd`
    4. datetime：日期(年月日时分秒)。例：`date yyyy-MM-dd HH:mm:ss`
    5. timestamp：时间戳类型(年月日时分秒)。如不赋值，则默认使用当前系统时间，来自动赋值。
    6. varchar：字符串类型。例：`name`
*/
```
> Tips: 如果想复制一张表，可以使用`create table 表名 like 被复制表名;`

##### 2. R(Retrieve)：查询
   * 查询某个数据库中所有表的名称：`show tables;`
   * 查询表结构：`desc 表名;`

##### 3. U(Update)：修改
   1. 修改表名：`alter table 表名 rename to 新的表名;`
   2. 修改表的字符集：`alter table 表名 character set 字符集名称;`
   3. 添加一列：`alter table 表名 add 列名 数据类型;`
   4. 修改列名称：`alter table 表名 change 列名 新列名 新数据类型;`
   5. 修改列数据类型：`alter table 表名 modify 列名 新数据类型;`
   6. 删除列：`alter table 表名 dorp 列名;`

##### 4. D(Delete)：删除
   * 删除表：`drop table 表名;`
   * 判断表存在才删除：`drop table if exists 表名;`



#### DML：增删改表中数据

**1. 添加数据**
   * 语法：`insert into 表名(列名1,列名2,...列名n) values(值1,值2,...值n);`
   * 注意事项：列名和值要一一对应；如果表名后面不定义列名，则默认给所有列添加值；除了数字类型，其他类型需要使用单双引号引起来。

**2. 删除数据**
   * 语法：`delete from 表名 [where 条件]`
   * 注意事项：如果不指定条件，则删除表中所有记录；如果要删除表所有记录，推荐使用`TRUNCATE TABLE 表名;`

**3. 修改数据**
   * 语法：`update 表名 set 列名1=值1,列名2=值2,...[where 条件];`
   * 注意：如果不加任何条件，则会将表中所有记录全部修改。



#### DQL：查询表中的记录

**简单查询**：

1. 查询表所有行和列的数据：`select * from 表名;`
2. 查询指定列：`select 字段名1,字段名2,...from 表名;`
3. 指定列/表的别名进行查询：`select 字段名1 AS 别名1,字段名2 AS 别名2,... from 表名 AS 表别名;`  

> Tips: 表使用别名一般用于多表查询操作。

**清除重复值**：

查询指定列并且结果不出现重复数据：`select distinct 字段名 from 表名;`

**查询结果参与运算**：

1. 某列数据和固定值运算：`select 列名1+固定值 from 表名;`
2. 某列数据和其他列数据参与运算：`select 列名1+列名2 from 表名;`

> 注意：参与运算的必须是数值类型

SQL语句示例：

```sql
/*
	给student表添加数学、英语成绩列，给每条记录添加对应的数学和英语成绩，查询时将数学和英语的成绩相加显示。
*/
SELECT * FROM student;
ALTER TABLE student ADD math INT; # 添加数学列
ALTER TABLE student ADD english INT; # 添加英语列
-- 两位同学添加数学英语成绩
UPDATE `student` SET `math` = '86' , `english` = '94' WHERE `age` = '19' AND `name` = 'xiaoming' AND `math` IS NULL AND `english` IS NULL;
UPDATE `student` SET `math` = '79' , `english` = '88' WHERE `age` = '18' AND `name` = 'xiaohua' AND `math` IS NULL AND `english` IS NULL;
-- 给所有数学加5分
SELECT math+5 FROM student;
-- 查询math+english的和
SELECT math+english FROM student;
-- 查询总成绩，并使用别名显示
SELECT *,(math+english) AS 总成绩 FROM student;
```

**条件查询**：

根据指定查询条件，对信息进行过滤显示，语句：`select 字段名 from 表名 where 条件;`

- SQL语句示例

```sql
-- 创建一个学生表，包含如下列：
CREATE TABLE student2(
	id INT, -- 编号
	NAME VARCHAR(20), # 姓名
	age INT, -- 年龄
	sex VARCHAR(5), # 性别
	address VARCHAR(100), -- 地址
	math INT, # 数学
	english INT -- 英语
	);

INSERT INTO student2(id,NAME,age,sex,address,math,english) VALUES(1,'马云',55,'男','
杭州',66,78),(2,'马化腾',45,'女','深圳',98,87),(3,'马景涛',55,'男','香港',56,77),(4,'柳岩
',20,'女','湖南',76,65),(5,'柳青',20,'男','湖南',86,NULL),(6,'刘德华',57,'男','香港
',99,99),(7,'马德',22,'女','香港',99,99),(8,'德玛西亚',18,'男','南京',56,65);
```

* 运算符

| 比较运算符          | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| >、<、<=、>=、=、<> | <>在SQL中表示不等于，在sql中也可以使用!=，没有==           |
| BETWEEN...AND       | 在一个范围之内，如：between 100 and 200 相当于条件在100到200之间。 |
| IN(集合)            | 集合表示多个值，使用逗号分隔                                 |
| LIKE'张%'           | 模糊查询                                                     |
| IS NULL             | 查询某一列为NULL的值，注：不能写=NULL                        |

* 操作示例

```sql
# 查询math分数大于80分的学生
SELECT * FROM student2 WHERE math>80;
# 查询english分数小于或等于80分的学生
SELECT * FROM student2 WHERE english <=80;
# 查询age等于20岁的学生
SELECT * FROM student2 WHERE age=20;
# 查询age不等于20岁的学生，注：不等于有两种写法
SELECT * FROM student2 WHERE age<>20;
SELECT * FROM student2 WHERE age!=20;
```

* 逻辑运算符

| 逻辑运算符 | 说明                                |
| :--------- | ----------------------------------- |
| and 或 &&  | 与，SQL中建议使用前者，后者不通用。 |
| or 或 x | 或                                  |
| not 或 !   | 非                                  |

>  x ： `||`

* 操作示例

```sql
# 查询age大于35且性别为男的学生
SELECT * FROM student2 WHERE age>35 AND sex='男';
# 查询age大于35或性别为男的学生
SELECT * FROM student2 WHERE age>35 OR sex='男';
# 查询id是1或3或5的学生
SELECT * FROM student2 WHERE id=1 OR id=3 OR id=5;
```

* in 关键字

  in里面的每个数据都会作为一次条件，只要满足条件就会显示：`select 字段名 from 表名 where 字段 in(数据1，数据2,...);`

```sql
# 查询id是1或3或5的学生，使用in
SELECT * FROM student2 WHERE id IN(1,3,5);
# 查询id不是1或3或5的学生
SELECT * FROM student2 WHERE id NOT IN(1,3,5);
```

* 范围查询

表示从值1到值2范围，全部包含：`between 值1 and 值2`

```sql
# 查询english成绩大于等于75，且小于等于90的学生
SELECT * FROM student2 WHERE english BETWEEN 75 AND 90;
```

* like关键字

表示模糊查询:`select * from 表名 where 字段名 like '通配符字符串;'`

通配符：`%,匹配任意多个字符串`; `_,匹配单个字符`

```sql
# 查询姓马的学生
SELECT * FROM student2 WHERE NAME LIKE '马%';
# 查询姓名中包含'德'字的学生
SELECT * FROM student2 WHERE NAME LIKE '%德%';
# 查询姓马，且姓名有两个字的学生
SELECT * FROM student2 WHERE NAME LIKE '马_';
```

