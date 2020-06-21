---
title: Redis事务和主从复制
date: 2020-6-21 18:40
categories: DataBase
tags: [Redis]
description: Redis事务、发布订阅和主从复制
urlname: Redis-TransactionAndReplication
---

## 1. Redis的事务

Redis的事务可以一次执行多个命令，本质是一组命令的集合；一个事务中的所有命令都会序列化，**按顺序地串行化执行而不会被其他命令插入**，不允许加塞。

> 简言之，Redis的事务就是：在一个队列中，一次性、顺序性、排他性的执行一系列命令

### 1.1 常用命令

常用命令表如下：

| 命令                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| DISCARD             | 取消事务，放弃执行事务块内的所有命令                         |
| EXEC                | 执行所有事务块内的命令                                       |
| MULTI               | 标记一个事务块的开始                                         |
| UNWATCH             | 取消WATCH命令对所有key的监视                                 |
| WATCH key [key ...] | 监视一个或多个key，如果在事务执行之前这些key被其他命令所改动，那么事务将被打断。 |

#### 1.1.1 Case1：全体连坐

发生一个错误，该事务块内所有命令都不执行： 

![连坐机制](http://yanxuan.nosdn.127.net/f606718797441c948e816847759a22db.png)

#### 1.1.2 Case2：冤头债主

对的执行错的抛出

![抛出错误](http://yanxuan.nosdn.127.net/5b40b8cd1f1650f98e23981e969c1ad4.png)

> 对于这种运行期才会报错的命令，是不会影响到其他命令继续执行的

#### 1.1.3 Case3：watch监控

- 悲观锁：就是很悲观，每次去拿数据时都认为别人会修改，所以每次拿数据时都会上锁，别人想拿数据就会阻塞等待。关系型数据库中很常用这种锁机制，如行锁、表锁等，都是在操作之前先上锁。
- 乐观锁：就是很乐观，每次去拿数据时都认为别人不会修改，所以不上锁，但是一般会使用到CAS（Check And Set）机制来进行判断，即在更新时会判断一下在此期间别人有没有去更新过这个数据，如使用到版本号机制，提交版本必须大于记录当前版本才能执行更新；乐观锁通常适用于多读的应用类型，可以提高吞吐量。

- 初始化余额和欠额，并使用事务保证两笔金额变动一致

![init](http://yanxuan.nosdn.127.net/f9ae4e95a7ece4d51b720bce6b42b9b3.png)

- 使用`watch`监控余额，如果balance被修改了，后一个事务的执行将会失效

![watch](http://yanxuan.nosdn.127.net/bbdd9f9c7096033e4a94185a0998bdc1.png)

- 使用`unwatch`取消监控，事务就能正常执行

![UTOOLS1592640014048.png](http://yanxuan.nosdn.127.net/b956290a78cedf4f427e2453c8407442.png)

> 执行了exec命令后之前加的监控锁都会被取消掉。

- 小结

Watch命令有点类似乐观锁，可同时监控多个keys，事务提交时，如果key值被别人修改过了，那么整个事务队列都不会被执行，返回一个`nil`提示事务执行失败。

- 事务的三个阶段
  - 开启：以`MULTI`开始一个事务；
  - 入队：将多个命令入队到事务中，这些命令并不会立即执行，而是放到等待执行的事务队列里面；
  - 执行：使用`EXEC`命令触发事务，依次执行事务队列中的命令。
- 事务的三个特性
  - 单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行；事务在执行过程中，不会被其他客户端发送来的命令请求所打断。
  - 没有隔离级别的操作：队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行；因此不存在事务内查询看到事务内的更新，事务外查询不能看到这类问题。
  - 不保证原子性：Redis同一个事务中如果有一条命令执行失败，其他的命令仍然会被执行，不会回滚。

## 2. Redis的发布订阅

Redis的发布订阅是一种进程间的消息通信模式：发送者（pub）发送消息，订阅者（sub）接收消息。

相关命令如下：

| 命令                             | 描述                             |
| -------------------------------- | -------------------------------- |
| PSUBSCRIBE pattern [pattern ...] | 订阅一个或多个符合给定模式的频道 |
| PUBSUB subcommand [argument ...] | 查看订阅与发布系统状态           |
| PUBLISH channel message          | 将消息发送到指定的频道           |
| PUNSUBSCRIBE [pattern ...]       | 退订所有给定模式的频道           |
| SUBSCRIBE channel ...            | 订阅给定的一个或多个频道的消息   |
| UNSUBSCRIBE channel...           | 退订指定频道                     |

- 案例
  - 客户端1输入`SUBSCRIBE c1 c2 c3`，同时订阅多个频道
  - 客户端2输入`PUBLISH c2 hello-redis`，对c2频道进行消息发布

> 不常用，了解即可。

## 3. Redis的主从复制（Master/Slave）

主从复制，即主机数据更新后根据配置和策略，自动同步到备机的master/slave机制，Master以写为主，Slave以读为主。主要功能就是做读写分离和容灾恢复。

### 3.1 从库配置

Redis的主从复制，**配从不配主**。

从库配置非常简单，使用`slaveof masterIP masterPort`即可，但每次与master断开之后，都要重新连接，除非将命令配置进redis.conf文件。

#### 3.1.1 修改配置文件细节

- 拷贝三份redis.conf文件

![cp](http://yanxuan.nosdn.127.net/9bfec101e8894da902eb81016148b0e2.png)

- 修改每个文件中的一些配置，修改为与文件端口名相同的信息。例如`redis6380.conf`配置文件，修改如下内容：
  - 开启`daemonize yes`
  - 修改`port 6380`
  - 修改`pidfile /var/run/redis_6380.pid`
  - 修改`logfile "6380.log"`
  - 修改`dbfilename dump_6380.rdb`

#### 3.1.2 常用一主二仆配置

- 开启3个不同的窗口，启动3台redis服务器并连接

```bash
窗口1：
redis-server /usr/local/redis/etc/redis6380.conf
redis-cli -p 6380
窗口2：
redis-server /usr/local/redis/etc/redis6381.conf
redis-cli -p 6381
窗口3：
redis-server /usr/local/redis/etc/redis6382.conf
redis-cli -p 6382
```

- 使用` info replication`命令查看当前库复制信息，默认情况下三台都是master角色
- 主机6380插入三条数据

![master](http://yanxuan.nosdn.127.net/634a7d8d0171f468beca1e89df3c6d15.png)

- 从机6381和6382连接主机，获取主机全部数据

![slave6381](http://yanxuan.nosdn.127.net/9c044b1fc3b5cdf2f47228991e6953d8.png)

![slave6382](http://yanxuan.nosdn.127.net/211a0d964c434c411f16ab40d71229ce.png)

- 此时主机6380输入命令`info replication`，信息如下

![master6380_info](http://yanxuan.nosdn.127.net/6b6d8c34c054c9887eb80526a8007c9e.png)

- 从机6381输入`info replication`，信息如下

![slave6381_info](http://yanxuan.nosdn.127.net/a833c590e33629da527e651a2eb496e7.png)

- 如果从机6381想执行写操作命令，是无法执行的，所谓的（从）读（主）写分离

![READONLY](http://yanxuan.nosdn.127.net/dc1681ee0d14cee181af8642ff67c084.png)

- 如果此时主机6380宕机了`shutdown`，那么从机的状态如下，从机会原地待命，等待主机上线。

![ifmasterdown](http://yanxuan.nosdn.127.net/f3245beb998a748d627c55204cccb718.png)

- 如果主机重新上线，那么从机会显示主机连接状态为上线状态，并且继续复制主机数据。
- 如果某一台从机掉线了，那么当它重启后就不是从机身份了，无法继续复制主机数据；想要获取主机数据需要重新连接。

![slave6381_down](http://yanxuan.nosdn.127.net/64c9fea74f79c8027eef01536f8e17e5.png)

#### 3.1.3 薪火相传配置

可以将一台Slave作为下一台slave的Master，Slave同样可以接收其他slaves的连接和同步请求，那么该Slave作为了链条中的下一个master，可以有效减轻主机的写压力。

- 将6382从机连接至6381从机

![slave6382](http://yanxuan.nosdn.127.net/954cb5078e7e8a69ecb8a6fa15eaf1c9.png)

- 6381从机信息

![6381slave](http://yanxuan.nosdn.127.net/fe85bbc83b9e7e53bba72d7cf9b82471.png)

- 6380主机信息，此时主机执行写操作，2台从机都可以获取

![UTOOLS1592727842027.png](http://yanxuan.nosdn.127.net/a29eaa5d5851141e7769ceae18093ab6.png)

#### 3.1.4 反客为主配置

当主机宕机时，从机输入`SLAVEOF on one`使当前数据库停止与其他数据库的同步，转为主数据库。其他从机需要重新连接至主机。

### 3.2 复制的原理

1. Slave启动成功连接到Master后会发送一个同步命令
2. Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕后，master将传送整个数据文件到slave，以完成一次完全同步
3. 全量复制：slave服务在接收到数据库文件数据后，将其存盘并加载到内存中
4. 增量复制：Master继续将新收集到的写操作命令依次传给slave，完成同步操作
5. slave只要重新连接到master，就会执行一次完全同步，进行全量复制

### 3.3 哨兵模式（sentinel）

哨兵模式就是反客为主的自动版，能够在后台监控主机是否故障，如果主机故障了会根据投票数自动选出一台从机将其转换为主库。

#### 3.3.1 配置

- 首先将3台服务器调整为一主二从配置，主机6380从机6381和6382。
- 在你的redis配置文件夹内新建`sentinel.conf`文件，文件名不能错。

![sentinel](http://yanxuan.nosdn.127.net/8efc5ca09ffc2ec2a488db11ea7381f6.png)

- 配置哨兵，填写内容，格式如下
  - `sentinel monitor 监控的数据库名称（自定义） IP port 投票数`

![conf](http://yanxuan.nosdn.127.net/56be22fd529df1eb1ad6342c59964a6d.png)

> 投票数1表示，主机宕机后，slave投票看看让谁接替成为主机，票数大于1成为主机

- 启动哨兵，输入`redis-sentinel+你的sentinel.conf文件位置`命令即可

![start](http://yanxuan.nosdn.127.net/0be9fa66a2e76fa8bf0339b135c0a48e.png)

- 此时如果主机6380宕机了，那么哨兵进程就会自动票选出master主机

![vote](http://yanxuan.nosdn.127.net/d5675d0e684414ba299be2fc722173fe.png)

- 如果以前的主机6380上线了，哨兵进程监控到就会将其连接到当前master主机，作为当前主机的从机。

![switch](http://yanxuan.nosdn.127.net/e162d9987d91809b58370d7b31c7372e.png)

### 3.4 复制的缺点

主从复制的一个重大缺点就是，复制延时性：由于所有的写操作都是先在Master上操作的，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统繁忙时，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重。



