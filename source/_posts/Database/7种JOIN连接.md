---
title: MySQL7种JOIN连接
date: 2020-03-20 15:00:00
categories: DataBase
tags: MySQL Join
urlname: Join
---



### 1. MySQL逻辑架构简介

MySQL的架构可以在多种不同场景中应用并发挥良好作用，主要体现在存储引擎的架构上，插件式的存储引擎架构将查询处理和其他的系统任务以及数据的存储提取相分离，这种架构可以根据业务的需求和实际需要选择合适的存储引擎。

<!--more-->

![MySQL架构图](http://yanxuan.nosdn.127.net/2363bb5c93657e9c0e67f1d880868f1b.png)

#### 1.1. 连接层

最上层是一些客户端和连接服务，例如连接处理，身份验证，安全性等。

#### 1.2. 服务层

MySQL核心部分，主要完成权限判断，查询缓存，SQL解析及优化等功能。

#### 1.3. 引擎层

存储引擎层负责MySQL中数据的存储和提取，服务器通过API和存储引擎进行通信；不同的存储引擎具有不同的功能特性。

#### 1.4. 存储层

数据存储层主要是将数据存储在运行于裸系统的文件系统上，并完成与存储引擎的交互。

[补充文章](https://blog.csdn.net/weixin_42358062/article/details/80730694)

### 2. SQL语句执行流程

![SQL执行流程图](http://yanxuan.nosdn.127.net/75f839355a6df8d19c8dc89bed66dddb.png)



### 3. MyISAM和InnoDB

{% raw %}

<table>
    <tr>
    	<th>对比项</th>
        <th>MyISAM</th>
        <th>InnoDB</th>
    </tr>
    <tr>
    	<td>主外键</td>
        <td>不支持</td>
        <td>支持</td>
    </tr>
    <tr>
    	<td>事务</td>
        <td>不支持</td>
        <td>支持</td>
    </tr>
    <tr>
    	<td>行表锁</td>
        <td>表锁，即操作一条记录就锁住整个表，不适合高并发的操作</td>
        <td>行锁，即操作时只锁某一行，不对其他行有影响<br><font color=red>适合高并发的操作</font></td>
    </tr>
    <tr>
    	<td>缓存</td>
        <td>只缓存索引，不缓存真实数据(时间换空间)</td>
        <td>不仅缓存索引还缓存真实数据，对内存要求较高，且内存大小对性能有影响(空间换时间)</td>
    </tr>
    <tr>
    	<td>表空间</td>
        <td>小</td>
        <td>大</td>
    </tr>
    <tr>
    	<td>关注点</td>
        <td>性能</td>
        <td>事务</td>
    </tr>
</table>

{% endraw %}



### 4. SQL性能下降的原因

#### 4.1 查询语句没写好

避免`select *`全表扫描

#### 4.2 索引失效

建立好索引，且保证索引不失效，或为热点字段建立索引

- 单值索引：`create index idx_user_name on user(name);`
- 复合索引：`create index idx_user_nameEmail on user(name,email);`

#### 4.3 关联查询太多join

设计缺陷或不得已的必要需求

#### 4.4 服务器调优及参数设置

缓冲，线程数等。



### 5. Join连接

完整的SQL Join连接图示如下：

![SQL JOINS](http://yanxuan.nosdn.127.net/2a211fe3769fa933c5932424ad964b37.png)

#### 5.1 INNER JOIN(内连接)

![INNER JOIN](http://yanxuan.nosdn.127.net/00e6b1dff50539b0b4c637bd7298eae3.png)

- 内连接查询返回表A和表B中所有匹配行的结果，示例如下：

```sql
SELECT <select_list>
FROM Table_A a
INNER JOIN Table_B b
ON a.key = b.key
```

#### 5.2 LEFT JOIN(左连接)

![LEFT JOIN](http://yanxuan.nosdn.127.net/ed92f89d777d459dfaa8c0bfeeb4806e.png)

- 左连接查询返回所有表A中的记录，不管是否有匹配记录在表B中(匹配不到的标记为null)，示例如下：

```sql
SELECT <select_list>
FROM Table_A a
LEFT JOIN Table_B b
on a.key = b.key
```

#### 5.3 RIGHT JOIN(右连接)

![RIGHT JOIN](http://yanxuan.nosdn.127.net/fbfc22bf5858f1319856173a2a04a484.png)

- 和左连接相反，右连接查询会返回所有表B中的记录，不管是否有匹配记录在表A中(无匹配标记null)，示例如下：

```sql
SELECT <select_list>
FROM Table_A a
RIGHT JOIN Table_B b
on a.key = b.key
```

#### 5.4 LEFT Excluding JOIN

![LEFT Excluding JOIN](http://yanxuan.nosdn.127.net/aec850dd1fc1da519da079c69300526a.png)

- 左排除右连接，它会返回表A中所有不在表B中的行(返回表A独有部分)，示例：

```sql
SELECT <select_list>
FROM Table_A a
LEFT JOIN Table_B b
on a.key = b.key
WHERE b.key IS NULL
```

#### 5.5 RIGHT Excluding JOIN 

![RIGHT Excluding JOIN](http://yanxuan.nosdn.127.net/0877fe10b91eb3e06a76b475bd6f3cd9.png)

- 右排除左连接，它会返回表B中所有不在表A中的行，示例：

```sql
SELECT <select_list>
FROM Table_A a
RIGHT JOIN Table_B b
on a.key = b.key
WHERE a.key IS NULL
```

#### 5.6 OUTER Excluding JOIN

![OUTER Excluding JOIN](http://yanxuan.nosdn.127.net/9ed77593405efdf6142bdebebd18a359.png)

- 外连接排除内连接结果，它会返回所有表A和表B中没有匹配的行(返回各自独有部分)，示例：

```sql
SELECT <select_list>
FROM Table_A A
FULL OUTER JOIN Table_B B
ON A.Key = B.Key
WHERE A.Key IS NULL OR B.Key IS NULL
```

#### 5.7 FULL OUTER JOIN

![FULL JOIN](http://yanxuan.nosdn.127.net/7eeff04fbda89041e4fdf279991cd9bc.png)



- 全连接，返回表A和表B中的所有行(匹配不到标记null)，示例如下：

```sql
SELECT <select_list>
FROM Table_A a
FULL OUTER JOIN Table_B b
on a.key = b.key
```

> 注意：MySQL天生不支持全连接，你可以使用`UNION`函数连接两个查询(左连接+右连接)来实现全连接。
>
> > `UNION`函数用于合并两个或多个`SELECT`语句的结果集并去重。
> >
> > > [参考文章](https://www.cnblogs.com/xufeiyang/p/5818571.html)



#### 5.8 常规SQL查询语句模板

```sql
SELECT DISTINCT
	<select_list>
FROM
	<left_table> <join_type>
JOIN <right_table> on <join_condition>
WHERE
	<where_condition>
GROUP BY
	<group_by_list>
HAVING
	<having_condition>
ORDER BY
	<order_by_condition>
LIMIT <limit number>
```

