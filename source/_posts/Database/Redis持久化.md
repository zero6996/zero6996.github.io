---
title: Redis持久化
date: 2020-6-19 23:59
categories: DataBase
tags: [Redis]
description: RedisRDB&AOF
urlname: Redis-Persistence
---

## 1. Redis配置文件解析

### 1.1 Units单位

![units](http://yanxuan.nosdn.127.net/42152b12ec96c818814b3be7222721d0.png)

Redis配置文件中，开头定义了一些基本的度量单位，只支持bytes，不支持bit，且对大小写不敏感。

### 1.2 INCLUDES包含

![includes](http://yanxuan.nosdn.127.net/09b2d21d8a851a665019ac91e489d9a2.png)

可以将当前配置文件作为主配置，用于包含其他配置文件。

### 1.3 GENERAL通用

Redis的一些通用配置

- daemonize：默认情况下，Redis不作为守护进程运行，需要的话设置为yes
- pidfile：pid文件位置
- loglevel：显示的日志级别，Redis默认四个级别（debug、verbose、notice、warning）
- logfile：指定日志文件名，默认为空字符串表示标准输出，如果你使用标准输出但是守护进程，日志将被发送到`/dev/null`
- databases：配置默认数据库数量
- syslog-enabled：是否把日志输出到syslog中
- syslog-ident：指定syslog中的日志标识
- syslog-facility：指定syslog的设备，值可以是USER或LOCAL0-LOCAL7

### 1.4 NETWORK网络

Redis关于网络的一些配置

- bind：如果直接注释掉bind，将会监听所有ip，生产环境建议设置为固定ip
- port：接收指定端口上的连接，默认为6379
- tcp-backlog：设置tcp的连接队列，backlog队列总和=未完成三次握手队列+已完成三次握手队列；在高并发环境下你需要一个高backlog来避免慢客户端连接问题
- timeout：在客户端空闲N秒后关闭连接（0禁用）
- tcp-keepalive：心跳检测时间，单位秒，0表示不检测，官方建议设置为300s

### 1.5 SECURITY安全

Redis关于安全的配置，由于Redis非常快，所以你应该使用非常强的密码，否则会很容易被破解，这里仅做简单介绍和使用。

- config get requirepass：获取当前密码，默认为`""`
- config set requirepass "youpassword"：设置密码，后续操作需先验证权限
- auth youpassword：输入密码，验证权限

> 将密码重新修改为`""`后，需重启服务器生效。

### 1.6 LIMITS限制

主要是一些内存的限制，在vi命令模式下输入下面的配置参数即可找到。

- maxclients：设置redis同时可以连接多少客户端，默认10000
- maxmemory：设置redis可使用的最大内存量，当内存使用达到上限时，redis会实行缓存淘汰策略，根据maxmemory-policy设置的策略来移除数据。
- maxmemory-policy：最大内存的淘汰策略，有以下几种：
  - volatile-lru：加入键时如果过限，首先从设置了过期时间的键集合中移除最久没有使用的键；
  - allkeys-lru：加入键如过限，首先通过LRU算法移除最久没有使用的键；
  - volatile-lfu：从所有配置了过期时间的键中移除使用频率最少的键；
  - allkeys-lfu：从所有键中移除使用频率最少的键；
  - volatile-random：从过期键的集合中随机移除键
  - allkeys-random：从所有键集合中随机移除；
  - volatile-ttl：从配置了过期时间的键中移除将要过期的键；
  - noeviction：不进行移除，内存过限将返回错误信息；
- maxmemory-samples：设置样本数量，LRU算法和最小TTL算法都并非精确的算法，而是估算值，所以你可以设置样本的大小。

[相关文章参考](https://www.jianshu.com/p/c8aeb3eee6bc)

### 1.7 SNAPSHOTTING快照

Redis持久化中RDB持久化的相关配置区

- `save <秒数> <写操作次数>`：快照保存时间与改动次数的设置
- `stop-writes-on-bgsave-error`：如果后台出错是否停止写入或者忽略错误继续写入，默认yes停止写入
- rdbcompression：对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能。
- rdbchecksum：在存储快照后，可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能
- dbfilename：rdb存储文件的名称，默认`dump.rdb`
- dir：当前redis的工作路径，客户端中通过`config get dir`可获取路径

### 1.8 APPEND ONLY MODE追加

Redis持久化中AOF持久化的相关配置区

- appendonly：设置是否开启AOF，默认关闭
- appendfilename：设置aof文件名称，默认`appendonly.aof`
- appendfsync：配置数据写入到磁盘的模式，有以下三种
  - Always：同步持久化，即每次发生数据变更会被立即记录到磁盘，性能较差但能保证数据完整性
  - Everysec：默认推荐配置，异步操作，每秒记录一次，如果一秒内宕机会有数据丢失
  - No： 不同步
- no-appendfsync-on-rewrite：重写时是否可以运用appendfsync，默认on关闭，保证数据安全性
- auto-aof-rewrite-percentage：设置重写的百分比基准值
- auto-aof-rewrite-min-size：设置重写文件的最小基准值

### 1.9 常用参数配置介绍

1. `daemonize no`：Redis默认不是以守护进程方式运行的，可以通过该配置项修改，使用yes启动守护进程。
2. `pidfile /var/run/redis.pid`：当Redis以守护进程方式运行时，Redis会默认把pid写入到/var/run/redis.pid文件中，可以通过pidfile指定。
3. `port 6379`：指定Redis监听端口，默认端口为6379。
4. `bind 127.0.0.1`：绑定的主机地址
5. `timeout 300`：当客户端空闲多久后关闭连接，如指定为0则表示关闭该功能。
6. `loglevel verbose`：指定日志记录级别，Redis支持四个级别：debug、verbos、notice、warning，默认varbose。
7. `logfile stdout`：日志记录方式，默认标准输出，如果配置Redis为守护进程运行，而这里又配置为标准输出，则日志将会发送给/dev/null
8. `database 16`：设置数据库的数量
9. `save <seconds> <changes>`：指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可多个条件配合；Redis默认配置文件中提供了三个条件：`save 900 1`，`save 300 10 `，`save 60 10000`，分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有1w个更改。
10. `rdbcompression yes`：指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大。
11. `dbfilename dump.db`：指定本地数据库文件名。
12. `dir ./`：指定本地数据库存放目录。
13. `slaveof <masterip> <masterport>`：当本机为slave服务时，设置master服务的IP地址及端口号，在Redis启动时，它会自动从master进行数据同步。
14. `masterauth <master-password>`：当master服务设置了密码保护时，slave服务连接master的密码。
15. `requirepass foobared`：设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过`AUTH <password>`命令提供密码，默认关闭。
16. `maxclients 128`：设置同一时间最大客户端连接数，默认无限制。
17. `maxmemory <bytes>`：设置Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会启动缓存淘汰机制，尝试清除一些数据。
18. `appendonly no`：指定是否在每次更新操作后进行日志记录，Redis默认情况下是异步的把数据写入磁盘，如不开启，可能会在断电时导致数据丢失。
19. `appendfilename appendonly.aof`：指定更新日志文件名称。
20. `appendfsync everysec`：指定更新日志条件，三个可选值。
    1. `no`：表示等操作系统进行数据缓存同步到磁盘（快）
    2. `always`：表示每次更新操作后手动调用`fsync()`将数据写到磁盘（慢，安全）
    3. `everysec`：表示每秒同步一次（折中，默认值）
21. `vm-enabled no`：指定是否启动虚拟内容机制，默认no；VM机制将数据分页存放，由Redis将访问量较少的页（即冷数据）swap到磁盘上，访问多的页面由磁盘自动换出到内存中
22. `vm-swap-fiel /tmp/redis.swap`：虚拟内存文件存储路径，默认值为`/tmp/redis.swap`，不可与多个Redis实例共享。
23. `vm-max-memory 0`：将所有大于`vm-max-memory`的数据存入虚拟内存，无论该参数设置多小，所有索引数据都是内存存储的（Redis的索引数据就是keys），也即当`vm-max-memory`设置为0时，其实是所有value都存于磁盘。
24. `vm-page-size 32`：Redis swap文件分成了很多的page，一个对象可以保存在多个page上，但一个page不能被多个对象共享，该参数要根据存储的数据大小来设定，官方建议如果存储很多小对象，page大小最好设置为32或64bytes；如果存储大对象，则可以使用更大的page，如不确定就使用默认值即可。
25. `vm-pages 134217728`：设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，在磁盘上每8个pages将消耗1byte的内存。
26. `vm-max-threads 4`：设置访问swap文件的线程数，默认4；最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。
27. `glueoutputbuf yes`：设置在向客户端应答时，是否把较小的包合并为一个包发送，默认开启。
28. `hash-max-zipmap-entries 64`，`hash-max-zipmap-value 512`：指定在超过一定数量或最大元素超过某一临界值时，采用的一种特殊的哈希算法。
29. `activerehasing yes`：设置是否激活重置哈希，默认开启
30. `include /path/to/local.conf`：指定包含的其他配置文件的路径，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件。

## 2. Redis的持久化

由于Redis的数据都存放在内存中， 如果没有配置持久化，redis重启后数据就全丢失了，于是需要开启redis的持久化功能，将数据保存到磁盘上，当redis重启后，可以从磁盘中恢复数据。Redis的持久化主要有RDB和AOF两种。

- RDB（Redis DataBase）： 将Reids在内存中的数据库记录定时dump到磁盘上；
- AOF（Append Only File）： 将Reids的操作日志以追加的方式写入文件 

### 2.1 RDB

RDB是在指定的时间间隔内将内存中的数据集快照（Snapshot）写入磁盘。

#### 2.1.1 RDB具体过程

Redis会单独创建（Fork）一个子进程来进行持久化，将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程不会进行任何IO操作，如此便确保了极高的性能，如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那么RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。

> RDB保存的是`dump.rdb`文件

#### 2.1.2 配置和触发RDB快照

RDB配置位于Redis配置文件的`SNAPSHOTTING`配置区，配置`save <seconds> <changes>`参数，该参数根据给定秒数和给定写操作数量进行持久化操作，默认参数如下：

- `save 900 1`：15分钟内改动一次就保存
- `save 300 10`：5分钟内改动10次就保存
- `save 60 10000`：1分钟内改动1万次就保存

将save参数`save 300 10`改动为`save 120 10`，即2分钟内10次改动就持久化，然后重启服务，进入`redis-cli`，快速插入10条记录，然后exit；随后进入目录`/usr/local/bin`，等待2分钟到，刷新目录查看是否有`dump.rdb`文件，如果没有就查看配置文件是否配置成功，服务是否重启。现在使用`cp dump.rdb dump_bk.rdb`命令对文件进行备份，然后进入redis客户端，使用`flushall`清空数据，`shutdown`关闭服务器。然后重启服务器，发现数据并没有被恢复。原因是执行了`flushall`命令后，也会产生`dump.rdb`文件，但是该文件是空的，没有意义。接下来使用`cp dump_bk.rdb dump.rdb`，复制一份备份文件，重启服务，进入客户端就会发现，数据恢复成功了。

在客户端中也可以使用的RDB相关命令

- `save`：马上执行一次RDB，将数据写入到磁盘
- `bgsave`：redis会后台异步进行快照操作，同时还可以相应客户端请求
- `lastsave`：查看最后一次成功执行的快照时间

> 注意：dump.rdb文件可能是根据redis启动路径来存放的，客户端中可通过`config get dir`获取redis工作路径。

#### 2.1.3 RDB的优缺点

- 优点：
  - RDB是一个非常紧凑的文件；
  - 因为RDB保存文件时会fork一个子进程进行操作，因此不会影响父进程做IO操作，故最能发挥Redis的性能；
  - 适合大规模的数据集恢复；
  - 适用于对数据完整性和一致性要求不高的场景；
- 缺点：
  - 数据丢失风险大：在一定间隔时间做一次备份，如果redis意外down掉的话，会丢失最后一次快照的所有修改；
  - Fork时，内存中的数据被克隆了一份，大概2倍的膨胀性需要考虑到。

> 关于停止RDB，客户端中直接输入`config set save ""`即可，配置文件中将save参数配置为`save ""`，注释掉其他save也可。

### 2.2 AOF

AOF是**以日志的形式来记录每个写操作**，将Redis执行过的所有写指令记录下来（读操作不记录），只允许追加文件但不可用改写文件，Redis启动时会读取该文件重新构建数据；简单说redis重启时就会根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

#### 2.2.1 AOF的配置

AOF的配置位于Redis配置文件的APPEND ONLY MODE配置区，配置`appendonly`参数，默认关闭。默认保存文件是`appendonly.aof`，位于redis启动目录下。

#### 2.2.2 AOF的启动/修复/恢复

- 启动

开启AOF只需修改参数为`appendnoly yes`即可，然后启动redis，写入几条数据，刷新启动目录可以看到`appendonly.aof`文件，此时AOF已经将你的所有写操作备份了。

- 恢复

如果你现在执行`flushall`，在关闭服务，重启服务后发现数据并没有恢复，原因是AOF备份文件中将`flushall`命令也执行了一遍，打开aof文件，将`flushall`命令删除，重启服务即可恢复数据。

- 修复

那么如果AOF文件被损坏了怎么办呢？打开`appendonly.aof`文件，随便写入一点数据，以模拟一个aof文件损坏场景，重启服务会发现redis服务无法启动，也无法连接到服务器，说明redis服务器启动时会先加载aof文件，如果加载失败，服务不会启动。

这时就需要用到`redis-check-aof`工具，这是redis自带的一款AOF文件修复工具，会将所有不符合aof文件语法的数据全部清除，输入`redis-check-aof --fix appendonly.aof`对损坏的aof文件就行修复。

![aof](http://yanxuan.nosdn.127.net/d49215ec0cc3869d6dd83b7e25df6fff.png)

再次重启服务器，发现服务器正常启动了，连接后数据也恢复了。

#### 2.2.3 Rewrite

AOF采用的文件追加方式，文件会越来越大，为避免出现此种情况，Redis新增了重写机制，当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集，可使用命令`bgrewriteaof`手动启动。

- 重写机制的原理

AOF文件持续增长且文件过大时，会fork出一条新进程来将文件重写（也是先写临时文件最后再rename），遍历新进程的内存中数据，每条记录有一条的set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有些类似。

- 触发机制

Redis会记录上次重写时的AOF文件大小，默认配置是当AOF文件大小为上次rewrite后大小的一倍且文件大于64MB时触发。

#### 2.2.4 AOF的优缺点

- 优点：
  - 秒级同步和修改同步极大的保证了数据的完整性
  - Redis可以在aof文件体积过大时，自动后台对aof进行重写
- 缺点：
  - 相同数据集的数据来说，aof文件要远大于rdb文件，恢复速度也慢于rdb
  - aof的运行效率要慢于rdb，但每秒同步策略效率较好，不同步效率和rdb相同
  - aof文件有序地保存了对数据库执行的所有写操作，这些操作以Redis协议格式保存，因此aof文件内容非常容易被人读懂和修改。



### 2.3 小结

RDB持久化方式能够在指定的时间间隔内对你的数据进行快照存储；AOF持久化方式记录每次对服务器的写操作，当服务器重启时会重新执行这些命令来恢复原始数据，AOF命令以redis协议追加保存每次写操作到文件末尾。Redis还能对AOF文件进行重写保证其体积不过大。

如果只做缓存服务，即希望数据仅在服务器运行时存在，可以不使用任何持久化方式。

如果同时开启两种方式：这种情况下，当redis重启时会优先加载AOF文件来恢复原始数据，因为通常情况下AOF文件保存的数据集要比RDB文件保存的数据集完整。由于RDB数据集的不实时性，同时使用服务器重启时也只会找AOF文件，那么是否可以只使用AOF呢？官方建议不要这样，因为RDB更适合用于备份数据库（AOF不断变化不好备份），快速重启，且不会有AOF可能存在的bug，留一手以备后患。

#### 2.3.1 性能建议

因为RDB文件仅做后备用途，建议只在Slave上持久化RDB文件，且只需15分钟备份一次足够了，即配置文件中仅保留`save 900 1`这条配置。

如果开启AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本简单只加载自己的AOF文件即可。代价是带来了持续的IO；其次AOF重写的最后将重写过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘允许，应该尽量减少AOF rewrite的频率，同时AOF重写的基础大小默认值64m太小了，可以设置到5G以上，默认超过原大小100%的设定也可改到适当的数值。

如果不开启AOF，仅靠主从复制实现高可用性也可以；这样能省下一大笔AOF的IO性能损耗同时也减少了rewrite时带来的系统波动。代价是如果Master/Slave同时宕机，会丢失十几分钟数据，启动脚本也要比较两个Master/Slave中的RDB文件，加载较新的那个。

