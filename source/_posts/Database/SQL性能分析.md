---
title: SQL性能分析
date: 2020-03-23 15:00:00
categories: DataBase
tags: Explain
urlname: Explain
---



### 1. MySQL Query Optimizer

MySQL查询优化器，是专门负责优化SELECT语句的模块，主要功能：通过计算分析系统中收集到的统计信息，为客户端请求的Query提供它认为最优的执行计划(优化器认为最优的数据检索方式，不一定是DBA认为最优的)。

<!--more-->

- 查询优化器工作步骤

当客户端向MySQL请求一条Query时，命令解析器模块会完成请求分类，区别出是SELECT并转发给MySQL的Query Optimizer时，查询优化器首先会对整条Query进行优化，处理掉一些常量表达式的预算，直接换算成常量值。并对Query中的查询条件进行简化和转换，如去掉一些无用或显而易见的条件、结构调整等。然后分析Query中的Hint信息(如果有)，看显示Hint信息是否可以完全确定该Query的执行计划。如果没有Hint或Hint信息不足以完成确定执行计划，则会读取所涉及对象的统计信息，根据Query进行写相应的计算分析，然后再得出最后的执行计划。

### 2. MySQL常见瓶颈

1. CPU：CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据时。
2. IO：磁盘I/O瓶颈发生在装入数据远大于内存容量时。
3. 硬件瓶颈：top、free、iostat和vmstat来查看系统的性能状态。

### 3. Explain

Explain可以**查看执行计划**，使用该关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈。[官网链接](https://dev.mysql.com/doc/refman/5.7/en/execution-plan-information.html)

- 使用`Explain`+SQL语句，就可以查看执行计划。

![Explain](http://yanxuan.nosdn.127.net/b6ae9124e0ee1862aac15d572becbb40.png)

下面详解`Explain`输出语句各字段的作用。

#### 3.1 id

id是查询的序列号，包含一组数组，表示查询中**执行select子句或操作表的顺序。**分别有三种情况

- id相同，执行顺序由上至下。

![id相同](http://yanxuan.nosdn.127.net/7a531e982c87d19e94948db515c7cf48.png)

- id不同，如果是子查询，id的序号会递增，**id值越大优先级越高，越先被执行。**

![id不同](http://yanxuan.nosdn.127.net/b55c700308941757050ee7f6893150b0.png)

- id有相同有不同，id值大的先执行，然后id相同的顺序执行。

![id相同不同](http://yanxuan.nosdn.127.net/eb5e0c6f446958bac4b9bcedc405cbe2.png)

> `<derived2>`表示的是衍生表的意思，关联了下方id为2的t3表。

#### 3.2 select_type

查询类型，主要是用于区别普通查询、联合查询、子查询等复杂查询，共有6种查询类型。

1. SIMPLE：简单的select查询，查询中不包含子查询或者UNION。
2. PRIMARY：查询中若包含如何复杂的子部分，最外层查询则被标记为
3. SUBQUERY：在SELECT或WHERE列表中包含了子查询。
4. DERIVED：在FROM列表中包含的子查询被标记为DERIVED(衍生的)时，MySQL会递归执行这些子查询，把结果放在临时表里。
5. UNION：若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED。
6. UNION RESULT：从UNION表获取结果的SELECT。

#### 3.3 table

显示这一行的数据是关于哪张表的。

#### 3.4 type

type显示的是访问类型，显示查询使用了那种类型，判定sql好坏的重要指标；从最好到最差依次是：<font color=red>system>const>eq_ref>ref>range>index>ALL</font>。

- system：**表只有一行记录**(等于系统表)，这是const类型的特例，平时不会出现，可以忽略不计。
- const：表示通过**索引一次就找到**了，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快；例如将主键置于where列表中，MySQL就能将该查询转换为一个常量时。
- eq_ref：**唯一性索引扫描**，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描。
- ref：非唯一性索引扫描，**返回匹配某个单独值的所有行**。本质上也是一种索引访问，它返回所有匹配某个单独值的行；然而它可能会找到多个符合条件的行，所以它应该属于查找和扫描的混合体。
- range：**只检索给定范围的行**，使用一个索引来选择行；key列显示使用了哪个索引，一般是SQL语句where中出现了`between、<、>、in`等的查询；这种范围扫描索引比全表扫描好一点，因为它只需扫描一定范围内的索引，不用扫描全部索引。
- index：Full Index Scan，index与ALL区别就是index类型**只遍历索引树**；通常比ALL快，因为索引文件通常比数据文件小。(即虽然all和index都是读全表，但index是从索引中读取的，而all是从硬盘中读的。)
- all：Full Table Scan，将**遍历全表**以找到匹配的行。

> 一般来说，保证查询至少达到range级别，最好达到ref。

#### 3.5 possible_keys

显示SQL优化器认为在该表中**可能用到的索引**列表，查询涉及到的字段上若存在索引，则该索引将被列出，但不代表查询时实际使用到。

#### 3.6 key

**实际使用到的索引**，如果为NULL表示没有使用到索引。

#### 3.7 key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引长度。在不损失精确性的情况下，长度越短越好。

key_len显示的值为索引字段的最大可能长度，并非实际使用长度；即key_len是根据表定义计算而得，而非通过表内检索出的。

#### 3.8 ref

**显示索引的哪一列被使用**了，如果可能的话，是一个常数；哪些列或常量被用于查找索引列上的值。

#### 3.9 rows

根据表统计信息及索引选用情况，大致估算出找到所需记录所需读取的行数。

#### 3.10 Extra

包含不适合在其他列显示但十分重要的额外信息。主要信息如下

1. **Using filesort**：说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取；MySQL中无法利用索引完成的排序操作称为”**文件排序**“。
2. **Using temporary**：**使用了临时表**保存中间结果，MySQL在对查询结果排序时使用临时表；常见于排序order by和分组查询group by。
3. **Using index：**
   - 表示相应的select操作中使用了覆盖索引(Covering Index)，避免访问了表的数据行，效率不错。
   - 如果同时出现using where，表明索引被用来执行索引键值的查找。
   - 如果没有出现using where，表明索引用来读取数据而非执行查找动作。
   - 覆盖索引：就是select的数据列只用从索引中就能够取得，不必读取数据行，MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件；即查询列要被所建立的索引覆盖。
4. Using where：表示使用了where过滤。
5. Using join buffer：使用了连接缓存。
6. Impossible where：表示where子句的值总是false，不能用来获取任何元祖。

