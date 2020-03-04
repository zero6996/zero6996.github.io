---
title: MySQL查询操作
date: 2019-5-19 23:59
categories: DataBase # 分类
tags: MySQL
description: MySQL的DQL，约束等
---



## 1. DQL查询语句

### 1.1 排序

- 语法：`select 字段名 from 表名 order by 字段名1 [ASC|DESC],字段名2 [ASC|DESC]...;`
- 排序方式：ASC(升序,默认值)；DESC(降序)。



<!--more-->




```sql
# 单列排序
SELECT * FROM student2 ORDER BY math; # 以math为条件升序排序，默认升序
SELECT * FROM student2 ORDER BY math DESC; # 以math为条件进行降序排序
# 组合排序
# 查询所有数据，在数学成绩升序排序的基础上，如果数学成绩相同在以英语成绩降序排序
SELECT * FROM student2 ORDER BY math ASC,english DESC;
```
> 注意：同时对多个字段进行排序，如果第1字段相等，则按第2字段排序，依次类推。

### 1.2 聚合函数

将一列数据作为一个整体，进行纵向的计算，返回一个结果值。

1. count：统计个数，一般选择非空的列(主键)
2. max：求最大值
3. min：求最小值
4. sum：求和
5. avg：求平均值

```sql
# 统计学生人数
SELECT COUNT(id) AS 总人数 FROM student2;
# 对english列统计个数
SELECT COUNT(english) FROM student2; -- 7
# 对english列统计个数,如果值为NULL，替换为0
SELECT COUNT(IFNULL(english,0)) FROM student2; -- 8
# 查询年龄大于20岁的人数
SELECT COUNT(*) FROM student2 WHERE age>20;
# 查询数学成绩总分
SELECT SUM(math) AS 数学成绩总分 FROM student2;
# 查询数学成绩平均分
SELECT AVG(math) AS 数学平均分 FROM student2;
# 查询数学成绩最高分
SELECT MAX(math) AS 数学最高分 FROM student2;
# 查询数学最低分
SELECT MIN(math) AS 数学最低分 FROM student2;
```

IFNULL(列名，默认值)：如果列名不为空，返回该列值。如果为NULL，则返回默认值。

> 聚合函数的计算排除了NULL值,可以选择非空列进行计算或者使用IFNULL函数

### 1.3 分组

分组查询是指使用`group by` 语句对查询信息进行分组，相同数据作为一组。

`select 字段1,字段2... from 表名 group by 分组字段[HAVING 条件];`

```sql
# 按性别进行分组，求男女生的数学平均分。
SELECT sex,AVG(math) FROM student2 GROUP BY sex; # 当我们使用某个字段分组，在查询时也需要将这个字段查询出来，否则看不大数据属于哪组的
/*
	group by 将分组字段结果中相同内容作为一组，并且返回每组的第一条数据。单独分组没用，分组的目的就是为了统计，所以一般分组会跟聚合函数一起使用。
*/
# 查询男女各有多少人
SELECT sex,COUNT(*)男女各有多少人 FROM student2 GROUP BY sex; # 分组函数会先查询所有数据，按性别分组，然后统计每组人数
# 查询年龄大于25岁的人，按照性别分组，统计每组人数
SELECT sex,COUNT(*)年龄大于25岁人数 FROM student2 WHERE age>25 GROUP BY sex;
# 查询年龄大于25岁的人，按性别分组，统计每组的人数，并只显示性别人数大于 2 的数据(使用having条件控制)
SELECT sex,COUNT(*)年龄大于25岁人数且人数大于2的 FROM student2 WHERE age>25 GROUP BY sex HAVING COUNT(*)>2; # 该SQL语句会先过滤掉年龄小于25岁的人，在按照性别分组，然后统计每组人数，最后显示性别人数大于2的数据
```

#### having与where的区别

| 子句       | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| where子句  | 对查询结果进行分组前，将不符合where条件的行去掉，即在分组之前过滤数据。where后面不可以使用聚合函数。 |
| having子句 | 该子句的作用是筛选满足条件的组，即在分组之后过滤数据。having后面可以使用聚合函数。 |



### 1.4 limit语句

limit是限制的意思，作用就是限制查询记录的条数。

语句：`select 字段列表 from 表名 [where子句][group by子句][limit offset(默认0),length];`

```sql
# 查询学生表中数据，从第三条开始，显示6条。
SELECT * FROM student2 LIMIT 2,6;
# 使用场景：分页，一般使用在类似淘宝商品信息分页。
```



## 2. 数据库的备份和还原

### 2.1 备份格式

在DOS下，使用语句：`mysqldump -u用户名 -p密码 数据库>文件路径`

```powershell
# 备份db4数据库中数据到本地db4.sql文件中
mysqldump -uroot -p123456 db4 > C:\Java\JavaWeb\db4.sql
```

### 2.2 还原格式

mysql中的命令，登录后使用：`use 数据库; source 导入文件的路径;`

```sql
/*还原步骤：
    1. 删除db4数据库中的所有表
    2. 登录mysql，选中数据库
    3. 使用source 命令还原数据
    4. 查看还原情况
*/
use db4;
source C:\Java\JavaWeb\db4.sql;
```



## 3. 数据库表的约束

### 3.1 概述

对表中的数据进行限制，保证数据的正确性、有效性和完整性。一个表如果添加了约束，不正确的数据将无法插入到表中。约束在创建表的时候添加比较合适。

### 3.2 约束种类

| 约束名   | 约束关键字             |
| -------- | ---------------------- |
| 主键     | primary key            |
| 唯一     | unique                 |
| 非空     | not null               |
| 外键     | foreign key            |
| 检查约束 | check，注：mysql不支持 |

### 3.3 主键

用来唯一标识数据库中的每一条记录。通常不用业务字段作为主键，而是单独给每张表设计一个id的字段，把id作为主键。

- 主键关键字：primary key
- 主键特点：非空(not null)且唯一

#### 创建主键

1. 在创建表时给字段添加主键：`字段名 字段类型 PRIMARY KEY;`
2. 在已有表中添加主键：`alter table 表名 add primary key(字段名);`

```sql
# 创建表时添加
create table student3(
	id int primary key; # id设为主键
    name varchar(20);
    age int);
# 删除主键
alter table student3 drop primary key;
# 在已有表中添加主键
alter table student3 add primary key(id);
```

#### 主键自增

可以使用`auto_increment`将主键字段设置为自增，数据库会自动生成主键字段值。

#### 修改自增长默认起始值

默认地 AUTO_INCREMENT 的开始值是 1，如果希望修改起始值,可以使用如下SQL语句。

- 创建表时指定起始值

```sql
create table 表名(
	列名 int primary key auto_increment,
)auto_increment=起始值;
```

- 创建表后修改起始值

```sql
alter table 表名 auto_increment=10;
```



####  delete和truncate对自增长的影响

- delete：删除所有的记录后，自增长没有影响。
- truncate：删除后，自增长初始值重新开始了。



### 3.4 唯一约束

用以约束表中某一列不能出现重复的值

- 语法：`字段名 字段类型 unique`

```sql
# 创建学生表4，包含字段(id, name),name 这一列设置唯一约束,不能出现同名的学生
CREATE TABLE student4(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(255) UNIQUE);

# 添加学生
INSERT INTO student4(NAME) VALUES('xiaohua');
INSERT INTO student4(NAME) VALUES('xiaohua'); # 错误代码： 1062 Duplicate entry 'xiaohua' for key 'name'
# 测试插入null
INSERT INTO student4(NAME) VALUES(NULL); # 不会报错，因为null无数据
```

### 3.5 非空约束

用于约束某一列不能为null

语法：`字段名 字段类型 not null`

```sql
-- 创建表学生表5, 包含字段(id,name,gender)其中 name 不能为 NULL
CREATE TABLE student5(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(100) NOT NULL,
	gender CHAR(1));

# 添加一条记录其中姓名不赋值
INSERT INTO student5(NAME,gender) VALUES(NULL,'man'); # 错误代码： 1048 Column 'name' cannot be null
```

#### 默认值

我们可以为字段指定默认值

语法：`字段名 字段类型 default 默认值`

```sql
-- 创建一个学生表6，包含字段(id,name,address)， 地址默认值是广州
CREATE TABLE student6(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20),
	address VARCHAR(50) DEFAULT '杭州');
	
-- 添加一条记录,使用默认地址
INSERT INTO student6(NAME,address) VALUES('xiaozhang',DEFAULT);
-- 添加一条记录,不使用默认地址
INSERT INTO student6(NAME,address) VALUES('xiaohua','上海');
```

> Tips: 如果一个字段设置了非空和唯一约束，那么该字段与主键有什么区别？
>
> 1. 主键在一个表中只能有一个。
> 2. 自增长只能用在主键上。



### 3.6 外键约束

**单表的缺点**：

```sql
# 创建一个员工表包含如下列(id, name, age, dep_name, dep_location),id 主键并自动增长,添加 5 条数据
CREATE TABLE emp (
    id INT PRIMARY KEY AUTO_INCREMENT,
    NAME VARCHAR(30),
    age INT,
    dep_name VARCHAR(30),
    dep_location VARCHAR(30)
);
# 添加数据
INSERT INTO emp (NAME, age, dep_name, dep_location) VALUES ('张三', 20, '研发部', '广州');
INSERT INTO emp (NAME, age, dep_name, dep_location) VALUES ('李四', 21, '研发部', '广州');
INSERT INTO emp (NAME, age, dep_name, dep_location) VALUES ('王五', 20, '研发部', '广州');

INSERT INTO emp (NAME, age, dep_name, dep_location) VALUES ('老王', 20, '销售部', '深圳');
INSERT INTO emp (NAME, age, dep_name, dep_location) VALUES ('大王', 22, '销售部', '深圳');
INSERT INTO emp (NAME, age, dep_name, dep_location) VALUES ('小王', 18, '销售部', '深圳');
```

- 上数据表的缺点: 一是数据冗余，二是后期可能出现的增删改问题。

#### 解决方案

```sql
# 将上述数据表分成两张表，部门表\员工表
-- 创建部门表(id,dep_name,dep_location)
-- 主表，一对多
CREATE TABLE department(
id INT PRIMARY KEY AUTO_INCREMENT,
dep_name VARCHAR(20),
dep_location VARCHAR(20));

-- 创建员工表(id,name,age,dep_id)
-- 从表,多对一
CREATE TABLE employee(
id INT PRIMARY KEY AUTO_INCREMENT,
NAME VARCHAR(20),
age INT,
dep_id INT -- 外键对应主表的主键
);

-- 添加两个部门
INSERT INTO department VALUES(NULL, '研发部','广州'),(NULL, '销售部', '深圳');
-- 添加员工，dep_id 表示员工所在部门
INSERT INTO employee (NAME, age, dep_id) VALUES ('张三', 20, 1);
INSERT INTO employee (NAME, age, dep_id) VALUES ('李四', 21, 1);
INSERT INTO employee (NAME, age, dep_id) VALUES ('王五', 20, 1);

INSERT INTO employee (NAME, age, dep_id) VALUES ('老王', 20, 2);
INSERT INTO employee (NAME, age, dep_id) VALUES ('大王', 22, 2);
INSERT INTO employee (NAME, age, dep_id) VALUES ('小王', 18, 2);
```

但是如果我们在employee的dep_id里面输入不存在的部门，数据仍然可以添加，所以要规范dep_id中的数据只能是department表中存在的id。故可以使用外键约束来解决该问题

#### 外键约束

- 外键：在从表中与主表主键对应的那一列，称为外键。例上从表中的dep_id
- 主表：一方，用来约束别人的表
- 从表：多方，被别人约束的表
- 创建约束语法：
  - 新建表时增加外键：`[CONSTRAINT] [外键约束名称] FOREIGN KEY(外键字段名) REFERENCES 主表名(主键字段名)`
  - 已有表增加外键：`ALTER TABLE 从表 ADD [CONSTRAINT] [外键约束名称] FOREIGN KEY (外键字段名) REFERENCES 主表(主键字段名);`

```sql
DROP TABLE employee;
-- 创建从表 employee 并添加外键约束 emp_depid_fk
-- 多方，从表
CREATE TABLE employee(
id INT PRIMARY KEY AUTO_INCREMENT,
NAME VARCHAR(20),
age INT,
dep_id INT, -- 外键对应主表的主键
-- 创建外键约束
CONSTRAINT emp_depid_fk FOREIGN KEY (dep_id) REFERENCES department(id)
);
-- 正常添加数据
INSERT INTO employee (NAME, age, dep_id) VALUES ('张三', 20, 1);
INSERT INTO employee (NAME, age, dep_id) VALUES ('李四', 21, 1);
INSERT INTO employee (NAME, age, dep_id) VALUES ('王五', 20, 1);
INSERT INTO employee (NAME, age, dep_id) VALUES ('老王', 20, 2);
INSERT INTO employee (NAME, age, dep_id) VALUES ('大王', 22, 2);
INSERT INTO employee (NAME, age, dep_id) VALUES ('小王', 18, 2);

-- 插入数据，指定一个不存在的部门值
INSERT INTO employee (NAME, age, dep_id) VALUES ('老板', 35, 6); # 错误代码： 1452 Cannot add or update a child row: a foreign key constraint fails (`db4`.`employee`, CONSTRAINT `emp_depid_fk` FOREIGN KEY (`dep_id`) REFERENCES `department` (`id`))
```

#### 删除外键

- 语法：`ALTER TABLE 从表 drop foreign key 外键名称;`

```sql
-- 删除employee表的emp_depid_fk外键
ALTER TABLE employee DROP FOREIGN KEY emp_depid_fk;
-- 在 employee 表存在的情况下添加外键
ALTER TABLE employee ADD CONSTRAINT emp_depid_fk FOREIGN KEY(dep_id) REFERENCES department(id);
```

#### 外键的级联

- 级联操作：在修改和删除主表的主键时，同时更新或删除副表的外键值，称为级联操作。

| 级联操作语法      | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| ON UPDATE CASCADE | 级联更新，只能是创建表的时候创建级联关系。更新主表中的主键，从表中的外键列也自动同步更新 |
| ON DELETE CASCADE | 级联删除                                                     |

```sql
--  删除 employee 表，重新创建 employee 表，添加级联更新和级联删除
DROP TABLE employee;

CREATE TABLE employee(
id INT PRIMARY KEY AUTO_INCREMENT,
NAME VARCHAR(20),
age INT,
dep_id INT, -- 外键对应主表的主键
-- 创建外键约束
CONSTRAINT emp_depid_fk FOREIGN KEY (dep_id) REFERENCES
 department(id) ON UPDATE CASCADE ON DELETE CASCADE # 添加级联更新和级联删除
);

-- 再次添加数据到员工表和部门表
INSERT INTO employee (NAME, age, dep_id) VALUES ('张三', 20, 1);
INSERT INTO employee (NAME, age, dep_id) VALUES ('李四', 21, 1);
INSERT INTO employee (NAME, age, dep_id) VALUES ('王五', 20, 1);
INSERT INTO employee (NAME, age, dep_id) VALUES ('老王', 20, 2);
INSERT INTO employee (NAME, age, dep_id) VALUES ('大王', 22, 2);
INSERT INTO employee (NAME, age, dep_id) VALUES ('小王', 18, 2);


DROP TABLE department; # Cannot delete or update a parent row: a foreign key constraint fails

# 把部门表中 id 等于 1 的部门改成 id 等于 10
UPDATE department SET id=10 WHERE id=1; # 1 queries executed, 1 success, 0 errors, 0 warnings

-- 删除部门号是 2 的部门
DELETE FROM department WHERE id=2;
```

### 3.7 数据约束小结

| 约束名 | 关键字      | 说明                         |
| ------ | ----------- | ---------------------------- |
| 主键   | primary key | 唯一且非空                   |
| 默认   | default     | 如果一列没有值，使用默认值   |
| 非空   | not null    | 这一列必须有值               |
| 唯一   | unique      | 这一列不能有重复值           |
| 外键   | foreign key | 主表中主键列，在从表中外键列 |



## 4. 表与表之间的关系

| 表与表之间的三种关系                                         |
| ------------------------------------------------------------ |
| 一对多：最常用的关系 部门和员工                              |
| 多对多：学生选课表 和 学生表， 一门课程可以有多个学生选择，一个学生选择多门课程 |
| 一对一：相对使用比较少。员工表 简历表， 公民表 护照表        |

### 4.1 一对多

一对多(1:n) 例如：班级和学生，部门和员工，客户和订单，分类和商品

一对多建表原则: 在从表(多方)创建一个字段,字段作为外键指向主表(一方)的主键

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/20/oneToMany-1558281810598.jpg)



### 4.2 多对多

多对多(m:n) 例如：老师和学生，学生和课程，用户和角色

多对多关系建表原则: 需要创建第三张表，中间表中至少两个字段，这两个字段分别作为外键指向各自一方的主 键。

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/20/ManyToMany-1558281920649.jpg)

### 4.3 一对一

一对一（1:1） 可以在任意一方添加外键指向另一方的主键。

两种建表原则：

1. 外键唯一：主表的主键和从表的外键（唯一），形成主外键关系，外键唯一 UNIQUE
2. 外键是主键：主表的主键和从表的主键，形成主外键关系

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/20/OneToOne-1558282085899.jpg)

### 4.4 综合案例

根据途牛网的旅游分类、旅游航线等信息设计数据库表结构

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/20/tuniu-1558320997129.jpg)

#### SQL语句实现

```sql
/*
1. 创建旅游线路分类表，tab_category
cid 主键，自增
cname 分类名称，唯一，
*/
CREATE TABLE tab_category(
cid INT PRIMARY KEY AUTO_INCREMENT,
cname VARCHAR(100) UNIQUE
);
/*
2. 创建航线route表
rid ，旅游线路主键，自增
rname 旅游线路名称，非空唯一，
price 价格
rdate 上架时间，
cid 外键
*/
CREATE TABLE tab_route(
rid INT PRIMARY KEY AUTO_INCREMENT,
rname VARCHAR(100) UNIQUE NOT NULL,
price DOUBLE,
rdate DATE,
cid INT,
FOREIGN KEY(cid) REFERENCES tab_category(cid) -- 外键指向主表主键
);

/*
3. 创建用户表 tab_user
uid 用户主键，自增长
username 用户名长度 100，唯一，非空
password 密码长度 30，非空
name 真实姓名长度 100
birthday 生日
sex 性别，定长字符串 1
telephone 手机号，字符串 11
email 邮箱，字符串长度 100
*/
CREATE TABLE tab_user(
uid INT PRIMARY KEY AUTO_INCREMENT,
username VARCHAR(100) UNIQUE NOT NULL,
PASSWORD VARCHAR(30) NOT NULL,
NAME VARCHAR(100),
birthday DATE,
sex CHAR(1) DEFAULT '男',
telephone VARCHAR(11),
email VARCHAR(100)
);

/*
4. 创建用户和航线的中间表，收藏表:
创建收藏表 tab_favorite
rid 旅游线路id,外键
date 收藏时间
uid 用户 id,外键
rid 和 uid 不能重复,设置复合主键,同一个用户不能收藏同一个线路两次
*/
CREATE TABLE tab_favorite(
rid INT, -- 旅游线路id
favtime DATETIME,
uid INT, -- 用户id
-- 创建复合主键
PRIMARY KEY(rid,uid),
FOREIGN KEY(rid) REFERENCES tab_route(rid),
FOREIGN KEY(uid) REFERENCES tab_user(uid)
);
```

#### 架构图
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/20/ExerciseCase-1558321667308.jpg)

### 4.5 表与表之间的关系小结

| 表与表的关系 | 关系的维护                                           |
| ------------ | ---------------------------------------------------- |
| 一对多       | 主外键的关系                                         |
| 多对多       | 中间表，两个一对多                                   |
| 一对一       | 特殊一对多，从表中的外键设为唯一；从表的主键又是外键 |



## 6. 数据库设计

- 范式：好的数据库设计对数据的存储性能和后期的程序开发，都会产生重要的影响。建立科学的，规范的数据库就需要满足一些规则来优化数据的设计和存储，这些规则就称为范式。



### 6.1 三大范式

- 范式概念：设计关系数据库时，遵从不同的规范要求，设计出合理的关系型数据库，这些不同的规范要求被称为不同的范式。各种范式呈递次规范，越高的范式数据库冗余越小。
- 六种范式：第一范式（1NF）、第二范式（2NF）、第三范式（3NF）、巴斯-科德范式（BCNF）、第四范式(4NF）和第五范式（5NF，又称完美范式）。

满足最低要求的范式是第一范式（1NF）。在第一范式的基础上进一步满足更多规范要求的称为第二范式（2NF），其余范式以次类推。一般说来，数据库只需满足第三范式(3NF）就行了

### 6.2  第一范式:1NF

数据库表的每一列都是不可分割的原子数据项，不能是集合、数组等非原子数据项。即表中的某个列有多个值时，必须拆分为不同的列。简而言之，第一范式每一列不可再拆分，称为原子性。

#### 示例：班级表

| 学号 | 姓名 | 班级  |
| ---- | ---- | ----- |
| 1    | 小张 | 103班 |
| 2    | 小李 | 102班 |
| 3    | 小王 | 203班 |

### 6.3 第二范式：2NF

在INF的前提下，非码属性必须完全依赖于码(在1NF基础上消除非主属性对主码的部分函数依赖)，即让表中的非主属性字段都**完全依赖**于主键。
所谓完全依赖是指不能存在仅依赖主键一部分的列。简而言之，第二范式就是在第一范式的基础上所有列完全依赖于主键列。

#### 概念

| 学号  | 姓名   | 系名   | 系主任 | 课程名称   | 分数 |
| ----- | ------ | ------ | ------ | ---------- | ---- |
| 10010 | 张无忌 | 经济系 | 张三丰 | 高等数学   | 95   |
| 10010 | 张无忌 | 经济系 | 张三丰 | 大学英语   | 87   |
| 10010 | 张无忌 | 经济系 | 张三丰 | 计算机基础 | 65   |
| 10011 | 令狐冲 | 法律系 | 任我行 | 法理学     | 77   |
| 10011 | 令狐冲 | 法律系 | 任我行 | 大学英语   | 87   |
| 10011 | 令狐冲 | 法律系 | 任我行 | 法律社会学 | 65   |
| 10012 | 杨过   | 法律系 | 任我行 | 法律社会学 | 95   |
| 10012 | 杨过   | 法律系 | 任我行 | 法理学     | 97   |
| 10012 | 杨过   | 法律系 | 任我行 | 大学英语   | 99   |

1. 函数依赖：A-->B,如果通过A属性(属性组)的值，可以确定唯一B属性的值。则称B依赖于A。例：学号-->姓名，通过学号可以确定唯一姓名值，故称姓名依赖于学号 ；(学号，课程名称) --> 分数，通过学号+课程名称可以确定唯一分数值，故分数依赖于学号+课程名称。
2. 完全函数依赖：A-->B,如果A是一个属性组，则B属性值的确定需要依赖于A属性组中所有的属性值。例：(学号，课程名称) --> 分数，确定分数的值只能通过(学号+课程名称)属性组确定，故分数完全依赖于(学号+课程名称)。
3. 部分函数依赖：A-->B, 如果A是一个属性组，则B属性值的确定只需要依赖于A属性组中某一些值即可。例如：(学号，课程名称) -->姓名，姓名属性值可以通过学号查询，故称姓名属性值部分依赖于(学号，课程名称)属性组。
4. 传递函数依赖：A--->B, B--->C, 如果通过A属性(属性组)的值，可以确定唯一的B属性值，在通过B属性值可以确定唯一C属性值，则称C传递函数依赖于A。例如：学号-->系名，系名-->系主任，通过学号属性值可以确定唯一系名，通过系名可以确定唯一系主任，那么就可以通过学号确定系主任，故称系主任传递依赖于学号。
5. 码：如果在一张表中，一个属性或属性组，被其他所有属性完全依赖，则称这一属性或属性组为该表的码。例如：上表中的码为：(学号，课程名称)，通过该属性组可确定所有其他属性值，所有其他属性完全依赖于该属性组，故称为码。
   * 主属性：码属性组中的所有属性
   * 非主属性：除码属性组外的属性

**第二范式的特点**：

1. 一张表只描述一件事情。
2. 表中的每一列都完全依赖于主键



###  6.4 第三范式：3NF

在2NF基础上，任何非主属性不依赖于其他非主属性(在2NF基础上消除传递依赖)，即表中每一列都直接依赖于主键，而不是通过其他的列来间接依赖于主键，任何非主列不得传递依赖于主键。

**示例**：

有如下学生信息表

| 学号 | 姓名 | 所在学院 | 学院地点 |
| ---- | ---- | -------- | -------- |
| 1    | 小明 | 计算机系 | 杭州校区 |

- 存在传递依赖关系：学号-->所在学院-->学院地点
- 消除依赖，拆分成两张表

学生表：
| 学号 | 姓名 | 所在学院编号(外键) |
| ---- | ---- | ------------------ |
| 1    | 小明 | 1                  |

学院表：
| 学院编号 | 学院地点 |
| -------- | -------- |
| 1        | 杭州校区 |

### 6.5 三大范式总结

| 范式 | 特点                                                         |
| ---- | ------------------------------------------------------------ |
| 1NF  | 原子性：表中每列不可再拆分                                   |
| 2NF  | 不产生局部依赖，一张表只描述一件事情                         |
| 3NF  | 不产生传递依赖，表中每一列都直接依赖于主键。而不是通过其他列间接依赖于主键。 |

