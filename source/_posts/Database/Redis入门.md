---
title: Redis入门
date: 2020-6-16 23:59
categories: DataBase
tags: [Redis]
description: Redis命令大全
urlname: Redis-Introduction
---

## 1. Redis入门概述

### 1.1 是什么？

Redis全称`Remote Dictionary Server`（远程字典服务器），是一款**完全开源的用C语言编写的遵守BSD协议的高性能分布式内存数据库**。基于内存运行，并支持持久化的NoSQL数据库，是当前最热门的NoSQL数据库之一，也被人们称为数据结构服务器。

Redis主要有以下三个特点：

1. Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启时可以再次加载进行使用；
2. Redis支持丰富的数据类型，不仅支持k-v类型数据，还提供list，set，zset，hash等数据结构的存储；
3. Redis支持数据的备份，即master-slave模式的数据备份。

### 1.2 能干嘛？

1. 内存存储和持久化：Redis支持异步将内存中的数据写到硬盘中，同时不影响继续服务；
2. 取最新数据：例如可以将最新的10条评论的ID放在Redis的List集合中；
3. 模拟类似于HttpSession这种需要设定过期时间的功能；
4. 发布、订阅消息系统；
5. 定时器、计数器

### 1.3 怎么下？

中文官网[下载地址](http://www.redis.cn/download.html)

### 1.4 怎么用？

主要使用如下功能：

- 数据类型、基本操作和配置
- 持久化和复制，RDB/AOF
- 事务的控制
- 主从复制
- ... ...

### 1.5 Linux版安装

1. 进入`/opt`目录，该目录是用户软件的安装目录。

2. 获取Redis资源

   ```bash
   wget http://download.redis.io/redis-stable.tar.gz
   ```

3. 解压

   ```bash
   tar -xzvf redis-stable.tar.gz
   ```

4. 进入解压后的文件夹

   ```bash
   cd redis-stable
   ```

5. 安装，正常步骤

   ```bash
   make
   cd src
   make install 
   ```

6. 异常步骤：如果没有安装gcc环境，是无法使用`make`命令编译的，需安装gcc环境

   ```bash
   Ubuntu安装gcc
   sudo apt install build-essential
   CentOS安装gcc
   yum install gcc-c++
   验证gcc安装
   gcc -v
   make #再次make时如果提示没有那个文件或目录，运行make distclean清理一下文件夹
   cd src
   make install
   ```

> 安装参考[文章](https://www.cnblogs.com/john-xiong/p/12098827.html)

7. 将配置文件备份

   ```bash
   cd /opt/redis-stable
   mkdir /usr/local/redis/etc # 配置文件存放位置
   cp redis.conf /usr/local/redis/etc # 复制到该文件夹中
   ```

8. 配置redis后台启动

   ```bash
   vi /usr/local/redis/etc/redis.conf
   输入i进入编辑模式
   找到daemonize no
   将其改为daemonize yes
   输入ESC
   组合键Shift和:
   然后输入wq 回车
   ```

9. 开启远程访问：打开配置文件，注释70行的bind，将90行的`protected-mode`改为`no`

10. 重启redis

    ```bash
    # 查询redis的运行状态，找到pid
    ps -ef|grep redis
    root      5557     1  0 17:42 ?        00:00:00 redis-server 127.0.0.1:6380
    # 将其kill掉，然后重启
    kill 5557
    redis-server /usr/local/redis/etc/redis.conf
    # 连接,我自定义了端口6380，默认6379
    redis-cli -p 6380
    ```

### 1.6 其他知识点

- Redis是单进程的，默认端口号6379；
- Redis索引默认都是从0开始；
- 默认有16个数据库，初始使用0号库；

- `select`：切换数据库，例如`select 2`切换到3号库；
- `dbsize`：查看当前数据库的key数量；
- `flushdb`：清空当前数据库；
- `flushall`：清空全部数据库；

## 2. Redis数据类型

### 2.1 Redis五大数据类型简介

#### 2.1.1 String（字符串）

String是Redis最基本的类型，可以理解为与Memcached一模一样的类型，一个key对应一个value；String类型是二进制安全的，意思该类型可以包含任何数据，例如jpg图片或者序列化的对象；一个Redis中字符串value最大是512MB。

#### 2.1.2 Hash（哈希）

哈希，类似Java中的`Map<String,Object>`；Redis中的hash是一个键值对集合，是String类型的field和value的映射表，特别适合用于存储对象。

#### 2.1.3 List（列表）

Redis列表时简单的字符串列表，按照插入顺序排序，可以添加一个元素到列表的头部或者尾部，底层其实是个链表。

#### 2.1.4 Set（集合）

Redis的Set是String类型的无序集合，它是通过HashTable实现的，不允许重复。

#### 2.1.5 Zset（Sorted Set有序集合）

Redis的zset和set一样也是String类型元素的集合，且不允许重复；它与set不同的是每个元素都会关联一个double类型的分数，通过分数来为集合中的成员从小到大进行排序；zset的成员是唯一的，但分数却可以重复。

#### 2.1.6 Redis命令参考

[命令参考文档](http://redisdoc.com/)

### 2.2 Redis键（key）

命令表，常用加粗：

| 命令                                      |                       描述                        |
| :---------------------------------------- | :-----------------------------------------------: |
| DEL key                                   |                当key存在时删除key                 |
| DUMP key                                  |         序列化给定key，并返回被序列化的值         |
| **EXISTS key**                            |                检查给定key是否存在                |
| **EXPIPE key seconds**                    |           为给定key设置过期时间，单位秒           |
| EXPIREAT key timestamp                    |       为給定key设置过期时间，单位UNIX时间戳       |
| PEXPRRE key milliseconds                  |             设置key过期时间，单位毫秒             |
| PEXPIREAT key milliseconds-timestamp      |         设置key过期时间的时间戳以毫秒计算         |
| **KEYS pattern**                          |       查找所有符合给定模式（pattern）的key        |
| **MOVE key db**                           |       将当前数据库的key移动到给定数据库db中       |
| **PERSIST key**                           |         移除key的过期时间，key将持久保持          |
| PTTL key                                  |         以毫秒为单位返回key的剩余过期时间         |
| **TTL key**                               | 秒为单位返回key剩余过期时间，-1永不过期，-2已过期 |
| RANDOMKEY                                 |            从当前数据库随机返回一个key            |
| RENAME key newkey                         |                   修改key的名称                   |
| RENAMENX key newkey                       |     仅当newkey不存在时，才会将key改名为newkey     |
| SCAN cursor [MATCH pattern] [COUNT count] |              迭代数据库中的数据库键               |
| **TYPE key**                              |              返回key所存储的值的类型              |

[Redis key常用命令表详见](https://www.runoob.com/redis/redis-keys.html )

### 2.3 Redis字符串（String）

非常常用的一种数据类型，单值对应单value，命令表如下，常用加粗：

| 命令                                 |                           描述                           |
| :----------------------------------- | :------------------------------------------------------: |
| **SET key value**                    |                     设置指定key的值                      |
| **GET key**                          |                     获取指定key的值                      |
| GETRANGE key start end               |                返回key中字符串值的子字符                 |
| **GETSET key value**                 |         将给定key的值设为value，并返回key的旧值          |
| **MGET key1 [key2... ...]**          |                获取一个或多个给定key的值                 |
| SETBIT key offset value              |    对key所存储的字符串值，设置或清除指定偏移量上的位     |
| GETBIT key offset                    |       对key所存储的字符串值，获取指定偏移量上的位        |
| **SETEX key seconds value**          |   将值 value 关联到 key ，并设置key的过期时间，单位秒    |
| **SETNX key value**                  |                仅当key不存在时设置key的值                |
| SETRANGE key offset value            | 从偏移量offset开始，用value参数覆写给定key存储的字符串值 |
| **STRLEN key**                       |                  返回key字符串值的长度                   |
| **MSET key value [key value ...]**   |               同时设置一个或多个k-v键值对                |
| **MSETNX key value [key value ...]** |              仅当key不存在时，创建多个k-v对              |
| PSETEX key milliseconds value        |          创建k-v对，并设置key过期时间，单位毫秒          |
| **INCR key**                         |                  将key中存储的数字值+1                   |
| **INCRBY key increment**             |              将key中存储的值加上给定增量值               |
| **INCRBYFLOAT key increment**        |            将key中存储的值加上给定浮点增量值             |
| **DECR key**                         |                  将key中存储的数值减一                   |
| **DECRBY key decrement**             |              将key中存储的值减去给定减量值               |
| **APPEND key value**                 |    如果key存在且是字符串，将指定value追加到key值末尾     |

[命令表详见](https://www.runoob.com/redis/redis-strings.html)

### 2.4 Redis列表（List）

Redis列表是简单的字符串列表，单值对应多value，按照插入顺序排序；你可以添加一个元素到列表的头部（左边）或者尾部（右边）；一个列表最多可以包含 2^32 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。常用命令表如下：

| 命令                                                         |                             描述                             |
| :----------------------------------------------------------- | :----------------------------------------------------------: |
| **LPUSH key value1 [value2]**                                |                 从列表左侧将一个或多个值插入                 |
| LPUSHX key value                                             |                将一个值插入到已存在的列表头部                |
| BLPOP key1 [key2] timeout                                    | 移出列表第一个元素并返回，如果列表为空则阻塞等待到超时或可弹出元素为止 |
| BRPOP key1 [key2] timeout                                    | 移出列表最后一个元素并返回，如果没有元素会阻塞等待到超时或发现可弹出元素为止 |
| BRPOPLPUSH source destination timeout                        | 从列表弹出一个值，将弹出的元素插入到另一列表中并返回它，没有元素会阻塞超时或发现有元素可弹出为止 |
| **LINDEX key index**                                         |        返回指定索引位置的元素，从列表头部开始索引为0         |
| [LINSERT key BEFORE\|AFTER pivot value](https://www.runoob.com/redis/lists-linsert.html) |                  在列表的元素前或后插入元素                  |
| **LLEN key**                                                 |                         获取列表长度                         |
| **LPOP key**                                                 |                  移出并获取列表的第一个元素                  |
| **LRANGE key start end**                                     |                    获取列表指定范围的元素                    |
| **LREM key count vlaue**                                     |                         移除列表元素                         |
| **LSET key index value**                                     |                   通过索引设置列表元素的值                   |
| [LTRIM key start stop](https://www.runoob.com/redis/lists-ltrim.html) | 对一个列表进行修剪（trim），就是让列表只**保留指定区间内**的元素，不在区间内的都删除 |
| **RPOP key**                                                 |                  移除列表最后一个元素并返回                  |
| **RPOPLPUSH source destination**                             |      移除列表最后一个元素，将其添加到另一个列表中并返回      |
| **RPUSH key value1 [value2]**                                |                从右侧向列表中添加一个或多个值                |
| RPUSHX key value                                             |                         从右侧添加值                         |

[命令表详见](https://www.runoob.com/redis/redis-lists.html)

### 2.5 Redis集合（Set）

Redis的Set是String类型的无序集合，不允许重复数据；集合是通过哈希表实现的，所以效率非常好，常用命令表如下：

| 命令                                                         |                  描述                  |
| :----------------------------------------------------------- | :------------------------------------: |
| **SADD key member1 [member2 ...]**                           |        向集合添加一个或多个成员        |
| **SCARD key**                                                |            返回集合成员数量            |
| **SDIFF key1 [key2]**                                        |                返回差集                |
| SDIFFSTORE destination key1 [key2]                           |     将给定集合的差集存储到目的地中     |
| **SINTER key1 [key2]**                                       |                返回交集                |
| SINTERSTORE destination key1 [key2]                          |     将给定集合的交集存储到目的地中     |
| **SISMEMBER key member**                                     |     判断member是否是集合key的成员      |
| **SMEMBERS key**                                             |           返回集合中所有成员           |
| **SMOVE source destination member**                          | 将member元素从source集合移到目的地集合 |
| **SPOP key**                                                 |         随机移除一个元素并返回         |
| **SRANDMEMBER key [count]**                                  |            返回几个随机元素            |
| **SREM key member1 [member2]**                               |        移除集合中一个或多个成员        |
| **SUNION key1 [key2]**                                       |                返回并集                |
| SUNIONSTORE destionation key1 [key2]                         |        返回并集放到目的地集合中        |
| [SSCAN key cursor [MATCH pattern]  [COUNT count]](https://www.runoob.com/redis/sets-sscan.html) |            迭代集合中的元素            |

[命令表详见](https://www.runoob.com/redis/redis-sets.html)

### 2.6 Redis哈希（Hash）

Redis hash是一个string类型的field和value的映射表，特别适合用于存储对象，常见命令如下：

| 命令                                                         |                 描述                  |
| :----------------------------------------------------------- | :-----------------------------------: |
| **HSET key field value**                                     | 将哈希表key中的字段field的值设为value |
| **HMSET key field1 value1 [field2 value2]**                  | 同时设置多个field-value到哈希表key中  |
| **HDEL key field1 [field2]**                                 |          删除一个或多个字段           |
| **HEXISTS key field**                                        |    查看指定字段是否存在于哈希表中     |
| **HGET key field**                                           |    获取存储在哈希表中指定字段的值     |
| **HGETALL key**                                              |       获取哈希表中所有字段和值        |
| **HINCRBY key field increment**                              |   为指定字段的值加上指定整数增量值    |
| **HINCRBYFLOAT key field increment**                         |   为指定字段的值加上指定浮点增量值    |
| **HKEYS key**                                                |         获取哈希表中所有字段          |
| **HLEN key**                                                 |        获取哈希表中字段的数量         |
| **HMGET key field [field2]**                                 |         获取所有给定字段的值          |
| **HSETNX key field value**                                   |  仅当字段field不存在时，设置字段的值  |
| **HVALS key**                                                |          获取哈希表中所有值           |
| [HSCAN key cursor [MATCH pattern] [COUNT count]](https://www.runoob.com/redis/hashes-hscan.html) |         迭代哈希表中的键值对          |

[命令表详见](https://www.runoob.com/redis/redis-hashes.html)

### 2.7 Redis有序集合Zset（sorted set）

Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序；有序集合的成员是唯一的,但分数(score)却可以重复；常用命令如下：

| 命令                                                         |                     描述                     |
| :----------------------------------------------------------- | :------------------------------------------: |
| **ZADD key score1 member1 [score2 member2]**                 |  添加一个或多个成员，或更新已存在成员的分数  |
| **ZCARD key**                                                |             获取有序集合的成员数             |
| **ZCOUNT key min max**                                       |       计算集合中指定区间分数的成员数量       |
| ZINCRBY key increment member                                 |       对集合中指定成员的分数加上增量值       |
| [ZINTERSTORE destination numkeys key [key ...]](https://www.runoob.com/redis/sorted-sets-zinterstore.html) |   计算多个集合的交集并将结果存储到新集合中   |
| [ZLEXCOUNT key min max](https://www.runoob.com/redis/sorted-sets-zlexcount.html) |         返回指定字典区间内的成员数量         |
| **ZRANGE key start stop [WITHSCORES]**                       |           返回指定索引区间内的成员           |
| ZRANGEBYLEX key min max [LIMIT offset count]                 |           返回指定字典区间内的成员           |
| **ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT]**           |           返回指定分数区间内的成员           |
| **ZRANK key member**                                         |           返回集合中指定成员的索引           |
| **ZREM key member [member ...]**                             |           移除集合中一个或多个成员           |
| ZREMRANGEBYLEX key min max                                   |        移除指定**字典区间**的所有成员        |
| ZREMRANGEBYRANK key start stop                               |        移除指定**索引区间**的所有成员        |
| ZREMRANGEBYSCORE key min max                                 |        移除指定**分数区间**的所有成员        |
| **ZREVRANGE key start stop [WITHSCORES]**                    | 返回集合中指定**索引区间**内的成员，分数降序 |
| **ZREVRANGEBYSCORE key max min [WITHSCORES]**                | 返回集合中指定**分数区间**内的成员，分数降序 |
| **ZREVRANK key member**                                      |  按照分数降序排序，返回集合中指定成员的索引  |
| **ZSCORE key member**                                        |             返回指定成员的分数值             |
| ZUNIONSTORE destination numbers key [key ...]                |           计算并集，存储在新集合中           |
| ZSCAN key cursor [MATCH pattern] [COUNT count]               |        迭代有序集合中的元素，包括分数        |

[命令表详见](https://www.runoob.com/redis/redis-sorted-sets.html)



