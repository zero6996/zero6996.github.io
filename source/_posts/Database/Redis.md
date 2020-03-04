---
title: Redis数据库
date: 2019-7-14 23:59
categories: DataBase
tags: Redis
description: Redis概念及基本使用
---



## Redis

`Redis`是一款高性能的NOSQL系列的非关系型数据库，完全开源免费的数据库，key-value型数据库，遵守BSD协议。



<!--more-->



## 1. 基本概念

### 1.1 什么是NOSQL

`NoSQL(NoSQL = Not Only SQL)`，意即“不仅仅是SQL”，是一项全新的数据库理念，泛指非关系型的数据库。

随着互联网web2.0网站的兴起，传统的关系数据库在应付web2.0网站，特别是超大规模和高并发的SNS类型的web2.0纯动态网站已经显得力不从心，暴露了很多难以克服的问题，而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。NoSQL数据库的产生就是为了解决大规模数据集合多重数据种类带来的挑战，尤其是大数据应用难题。



#### 1.1.1 NoSQL和关系型数据库比较

- 优点：
  - 成本：nosql数据库简单易部署，基本都是开源软件，不需要像使用oracle那样花费大量成本购买使用，相比关系型数据库价格便宜。
  - 查询速度：nosql数据库将数据存储在缓存中，关系型数据库将数据存储在硬盘中，自然查询速度远不及nosql数据库。
  - 存储数据的格式： nosql存储格式是key-value形式、文档形式，图片形式等等，所以可以存储基础类型以及对象或是集合等各种格式，而数据库则只支持基础类型。
  - 扩展性：关系型数据库有类似join这样的多表查询机制的限制导致扩展很难。
- 缺点：
  - 维护的工具和资料有限，因为nosql是属于新的技术，不能和关系型数据库十几年的技术同日而语。
  - 不提供对sql的支持，如果不支持sql这样的工业标准，将产生一定的用户学习成本和使用成本。
  - 不提供关系型数据库对事务的处理。

#### 1.1.2 关系型数据库的优势

1. 复杂查询可以用SQL语句方便的在单表或多表之间做非常**复杂的数据查询**。
2. **事务支持**使得对于安全性能很高的数据访问要求得以实现。对于这两类数据库，对方的优势就是自己的弱势，反之亦然。

#### 1.1.3 非关系型数据库的优势

1. NoSQL是基于键值对的，可以想象成表中的主键和值的对应关系，而且不需要经过SQL层的解析，所有**性能非常高**。
2. 同样也是因为基于键值对，数据之间没有耦合性，所以非常容易水平**扩展**。

#### 1.1.4 总结

1. 关系型数据库与NoSQL数据库并非对立而是互补的关系，即通常情况下使用关系型数据库，在适合使用NoSQL时使用NoSQL数据库。
2. 使用NoSQL数据库对关系型数据库的不足进行弥补。
3. 一般会将数据存储在关系型数据库中，在nosql数据库中备份存储关系型数据库的数据。



### 1.2 主流的NoSQL产品

- 键值存储数据库
  - 相关产品：Tokyo Cabinet/Tyrant、Redis、Voldemort、Berkeley DB
  - 典型应用：内容缓存，主要用于处理大量数据的高访问负载。
  - 数据模型：一系列键值对
  - 优势：快速查询
  - 劣势：存储的数据缺少结构化
- 列存储数据库
  - 相关产品：Cassandra, HBase, Riak
  - 典型应用：分布式的文件系统
  - 数据模型：以列簇式存储，将同一列数据存在一起
  - 优势：查找速度快，可扩展性强，更容易进行分布式扩展
  - 劣势：功能相对局限
- 文档性数据库
  - 相关产品：CouchDB、MongoDB
  - 典型应用：Web应用（与Key-Value类似，Value是结构化的）
  - 数据模型：一系列键值对
  - 优势：数据结构要求不严格
  - 劣势：查询性能不高，且缺乏统一的查询语法
- 图形(Graph)数据库
  - 相关数据库：Neo4J、InfoGrid、Infinite Graph
  - 典型应用：社交网络
  - 数据模型：图结构
  - 优势：利用图结构相关算法
  - 劣势：需要对整个图做计算才能得出结果，不容易做分布式的集群方案。



### 1.3 什么是Redis

Redis是用C语言开发的一个开源的高性能键值对（key-value）数据库，官方提供测试数据，50个并发执行100000个请求,读的速度是110000次/s,写的速度是81000次/s ，且Redis通过提供多种键值数据类型来适应不同场景下的存储需求，目前为止Redis支持的键值数据类型如下：

1. 字符串类型：string
2. 哈希类型：hash
3. 列表类型：list
4. 集合类型：set
5. 有序集合类型：sortedset



#### 1.3.1 redis的应用场景

缓存（数据查询、短连接、新闻内容、商品内容等等）
- 聊天室的在线好友列表
- 任务队列。（秒杀、抢购、12306等等）
- 应用排行榜
- 网站访问统计
- 数据过期处理（可以精确到毫秒
- 分布式集群架构中的session分离



## 2. 下载与安装

1. 官网：[https://redis.io/](https://redis.io/)
2. 中文网：[https://www.redis.net.cn/](https://www.redis.net.cn/)
3. windows下载地址：[https://github.com/dmajkic/redis/downloads](https://github.com/dmajkic/redis/downloads)
4. 安装与使用：解压即可直接使用
   - `redis.windows.conf`：配置文件
   - `redis-cli.exe`：redis的客户端
   - `redis-server.exe`：redis服务器端
5. Redis桌面管理工具：[下载地址](https://redisdesktop.com/download
   )
6. 安装为windows服务：`redis-server.exe --service-install redis.windows.conf`
   - 启动服务：`redis-server --service-start`
   - 停止服务：`redis-server --service-stop`
   - 卸载服务：`redis-server --service-uninstall`
   - 连接客户端：`redis-cli`&`redis-cli -h [服务器地址] -p [指定端口号] -a [连接数据库的密码,在conf中配置，默认无]`

安装与部署服务参考[文章](https://www.cnblogs.com/dingguofeng/p/8709476.html)

## 3. 命令操作

### 3.1 redis的数据结构

- redis存储的是：`key-value`格式的数据，其中key都是字符串，value有5种不同的数据结构。

#### value的五种数据结构

1. 字符串类型：`string`
2. 哈希类型` hash`：`ma`p格式
3. 列表类型 `list`：`linkedlis`t格式。支持重复元素
4. 集合类型 `set`：不允许重复元素
5. 有序集合类型` sortedset`：不允许重复元素，且元素有顺序



### 3.2 字符串类型:`String`

- 存储：`set key value`
- 获取：`get key`
- 删除：`del key`

```powershell
127.0.0.1:6379> set ts teststring
OK
127.0.0.1:6379> get ts
"teststring"
127.0.0.1:6379> del ts
(integer) 1
```

### 3.3 哈希类型:`Hash`

- 存储：`hset key field value`
- 获取：
  - `hget key field`：获取指定的`field`对应的值
  - `hgetall key`：获取所有的`field`和`value`
- 删除：`hdel key field`

```powershell
127.0.0.1:6379> hset myhash username shangsan
(integer) 1
127.0.0.1:6379> hset myhash password 123
(integer) 1
127.0.0.1:6379> hget myhash username
"shangsan"
127.0.0.1:6379> hgetall myhash
1) "username"
2) "shangsan"
3) "password"
4) "123"
127.0.0.1:6379> hdel myhash username
(integer) 1
127.0.0.1:6379> hdel myhash password
(integer) 1
```

### 3.4 列表类型:`List`

可以添加一个元素到列表的头部(左边)或尾部(右边)

- 添加：
  - `lpush key value`：将元素加入列表左边。
  - `rpush key value`：将元素加入列表右边
- 获取：`lrange key start end`，获取指定范围的元素数据
- 删除：
  - `lpop key`：删除列表最左边的元素，并将元素返回。从左出栈
  - `rpop key`：删除列表最右边的元素，并返回。从右出栈

```powershell
127.0.0.1:6379> lpush mylist c
(integer) 9998
127.0.0.1:6379> lpush mylist b
(integer) 9999
127.0.0.1:6379> lpush mylist a
(integer) 10000
127.0.0.1:6379> lrange mylist 0 2
1) "a"
2) "b"
3) "c"
127.0.0.1:6379> lpop mylist
"a"
127.0.0.1:6379> lpop mylist
"b"
127.0.0.1:6379> lpop mylist
"c"
```

### 3.5 集合类型:`set`

不允许重复元素

- 存储：`add key value`
- 获取：`smembers key`，获取集合中所有元素
- 删除：`srem key value`，删除集合中的某个元素



```powershell
127.0.0.1:6379> sadd myset a
(integer) 1
127.0.0.1:6379> sadd myset a
(integer) 0
127.0.0.1:6379> smembers myset
1) "a"
127.0.0.1:6379> srem myset a
(integer) 1
```



### 3.6 有序集合类型:`sortedset`

不允许重复元素，且元素有顺序。每个元素都会关联一个`double`类型的分数。`redis`正是通过分数来为集合中的成员进行从小到大的排序。

- 存储：`zadd key score vlaue`
- 获取：`zrange key start end [withscores]`，withscores可以在获取时同时获取分数。
- 删除：`zrem key value`



```powershell
127.0.0.1:6379> zadd mysort 50 zhangsan
(integer) 1
127.0.0.1:6379> zadd mysort 36 lisi
(integer) 1
127.0.0.1:6379> zadd mysort 88 wangwu
(integer) 1
127.0.0.1:6379> zrange mysort 0 -1
1) "lisi"
2) "zhangsan"
3) "wangwu"
127.0.0.1:6379> zrem mysort lisi
(integer) 1
127.0.0.1:6379> zrange mysort 0 -1
1) "zhangsan"
2) "wangwu"
```



### 3.7 通用命令

1. `keys *`：查询所有的键
2. `type key`：获取键对应的`value`类型
3. `del key`：删除指定的`key valu`



```powershell
127.0.0.1:6379> keys *
1) "counter:__rand_int__"
2) "mylist"
3) "key:__rand_int__"
4) "mysort"
127.0.0.1:6379> type mylist
list
127.0.0.1:6379> type mysort
zset
127.0.0.1:6379> del mylist
(integer) 1
```



## 4. 持久化

`redis`是一个内存数据库，当`redis`服务器重启或电脑重启时，数据会丢失。我们可以将`redis`内存中的数据持久化保存到硬盘的文件中。

#### 4.1 redis持久化机制

- RDB：默认方式，不需要进行配置。在一定的间隔时间中，检测key的变化情况，然后持久化数据。

  - 编辑`redis.windows.conf`文件

    ```properties
    #   after 900 sec (15 min) if at least 1 key changed
    save 900 1
    #   after 300 sec (5 min) if at least 10 keys changed
    save 300 10
    #   after 60 sec if at least 10000 keys changed
    save 60 10000
    ```

  - 重启redis服务器，并指定配置文件名称：`redis-server.exe redis.windows.conf`
  
- AOF：日志记录的方式，可以记录每一条命令的操作。可以每一次命令操作后，持久化数据。

  - 编辑`redis.windows.conf`文件

    ```properties
    # 开启AOF
    appendonly yes
    # 三种同步方式:
    # 1. 每一次操作都进行持久化
    # appendfsync always 
    # 2. 每隔一秒进行一次持久化，缺省值
    appendfsync everysec
    # 3. 不进行持久化
    # appendfsync no
    ```

  - 重启服务器，指定配置文件。

## 5. Java客户端：`Jedis`

`Jedis`是一款java操作redis数据库的工具。



### 5.1`Jedis`操作各种`Redis`中的数据结构

#### 5.1.1 字符串类型：

```java
@Test
public void test2(){
    // 1. 获取连接
    Jedis jedis = new Jedis(); // 如果使用空参构造，默认值'localhost',6379
    // 2. 操作
    // 存储
    jedis.set("username","wangwu");
    // 获取
    String username = jedis.get("username");
    System.out.println(username);

    // 可以使用setex()方法存储指定过期时间的 key value
    jedis.setex("activecode",20,"gj8a"); //将activecode：gj8a键值对存入redis，并且20秒后自动删除该键值对

    // 3. 关闭连接
    jedis.close();
}
```

#### 5.1.2 哈希类型：

```java
@Test
public void test3(){
    // 1. 获取连接
    Jedis jedis = new Jedis();
    // 2. 操作
    // 存储hash
    jedis.hset("user","name","lisi");
    jedis.hset("user","age","23");
    jedis.hset("user","gender","female");
    // 获取hash
    String name = jedis.hget("user", "name");
    System.out.println(name);
    // 获取hash的所有map中的数据
    Map<String, String> user = jedis.hgetAll("user");
    Set<String> keySet = user.keySet();
    for (String key:keySet) {
        String value = user.get(key);
        System.out.println(key+":"+value);
    }
    // 3. 关闭连接
    jedis.close();
}
```

#### 5.1.3 列表类型：

```java
@Test
public void test4() {
    // 1. 获取连接
    Jedis jedis = new Jedis();
    // 2. 操作
    jedis.del("mylist");
    // list存储
    jedis.lpush("mylist","a","b","c"); // 从左边存
    jedis.rpush("mylist","a","b","c"); // 从右边存
    // 获取
    List<String> mylist = jedis.lrange("mylist", 0, -1);
    System.out.println(mylist); // [c, b, a, a, b, c]
    // 弹出
    String element1 = jedis.lpop("mylist");
    System.out.println(element1);
    String element2 = jedis.rpop("mylist");
    System.out.println(element2);
    List<String> mylist２ = jedis.lrange("mylist", 0, -1);
    System.out.println(mylist２);
    // 3.关闭连接
    jedis.close();
}
```

#### 5.1.4 集合类型：

```java
@Test
public void test5() {
    // 1. 获取连接
    Jedis jedis = new Jedis();
    // 2. 操作
    // 存储
    jedis.sadd("myset","java","php","c");
    // 获取
    Set<String> myset = jedis.smembers("myset");
    System.out.println(myset);
    jedis.close();
}
```

#### 5.1.5 有序集合类型：

```java
@Test
public void test6() {
    // 1. 获取连接
    Jedis jedis = new Jedis();
    // 2. 操作
    // 存储
    jedis.zadd("mysortedset",8,"亚索");
    jedis.zadd("mysortedset",6,"剑圣");
    jedis.zadd("mysortedset",3,"vn");
    // 获取
    Set<String> mysortedset = jedis.zrange("mysortedset", 0, -1);
    System.out.println(mysortedset); // [vn, 剑圣, 亚索]
    jedis.close();
}
```



### 5.2 `Jedis`连接池：`JedisPool`

- 使用方法：
  - 创建`JedisPool`连接池对象
  - 调用方法`getResource()`方法获取`Jedis`连接

#### 5.2.1 `jedis`详细配置
```properties
#最大活动对象数     
redis.pool.maxTotal=1000    
#最大能够保持idel状态的对象数      
redis.pool.maxIdle=100  
#最小能够保持idel状态的对象数   
redis.pool.minIdle=50    
#当池内没有返回对象时，最大等待时间    
redis.pool.maxWaitMillis=10000    
#当调用borrow Object方法时，是否进行有效性检查    
redis.pool.testOnBorrow=true    
#当调用return Object方法时，是否进行有效性检查    
redis.pool.testOnReturn=true  
#“空闲链接”检测线程，检测的周期，毫秒数。如果为负值，表示不运行“检测线程”。默认为-1.  
redis.pool.timeBetweenEvictionRunsMillis=30000  
#向调用者输出“链接”对象时，是否检测它的空闲超时；
redis.pool.testWhileIdle=true  
# 对于“空闲链接”检测线程而言，每次检测的链接资源的个数。默认为3.  
redis.pool.numTestsPerEvictionRun=50  
#redis服务器的IP    
redis.ip=xxxxxx  
#redis服务器的Port    
redis1.port=6379   
```


#### 5.2.2 自定义`jedis`工具类


```java
package cn.zero.jedis.util;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

/**
 * JedisPool工具类
 *  加载配置文件，配置连接池参数
 *  提供获取连接的方法
 */
public class JedisPoolUtils {
    private static JedisPool jedisPool;
    static {
        // 读取配置文件
        InputStream is = JedisPoolUtils.class.getClassLoader().getResourceAsStream("jedis.properties");
        // 创建Properties对象
        Properties pro = new Properties();
        // 关联文件
        try {
            pro.load(is);
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 获取数据，设置到JedisPoolConfig中
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(Integer.parseInt(pro.getProperty("maxTotal")));
        config.setMaxIdle(Integer.parseInt(pro.getProperty("maxIdle")));

        // 初始化JedisPool
        jedisPool = new JedisPool(config, pro.getProperty("host"), Integer.parseInt(pro.getProperty("port")));
    }
    
    /**
     * 获取连接方法
     * @return
     */
    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}
```



## 6. 练习

### 6.1 需求

1. 提供index.html页面，页面中有一个省份下拉列表
2. 当 页面加载完毕后 发送ajax请求，加载所有省份。


![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/07/14/4.%E6%A1%88%E4%BE%8B-1563119468012.bmp)


- 注意：使用redis缓存一些不经常发生变化的数据。
  - 数据库的数据一旦发生改变，则需要更新缓存。
    - 数据库的表执行 **增删改**的相关操作，需要将redis缓存数据清空，再次存入。
    - 在service对应的增删改方法中，将redis数据删除。


### 6.2 代码实现

后端service实现类核心代码：

```java
	/**
     * 使用redis缓存
     * @return
     */
    @Override
    public String findAllJson() {
        // 1. 先从redis中查询数据
        // 1.1 获取redis客户端连接对象
        Jedis jedis = JedisPoolUtils.getJedis();
        // 1.2 查询数据
        String province_json = jedis.get("province");
        // 2. 判断province_json 是否为null
        if (province_json == null || province_json.length() == 0){
            // redis中无数据
            System.out.println("redis中无数据，查询数据库...");
            // 2.1 从数据库中查询
            List<Province> ps = dao.findAll();
            // 2.2 将list序列化为json
            ObjectMapper mapper = new ObjectMapper();
            try {
                province_json = mapper.writeValueAsString(ps);
            } catch (JsonProcessingException e) {
                e.printStackTrace();
            }
            // 2.3 将json数据存入redis中
            jedis.set("province",province_json);
            // 归还连接
            jedis.close();
        }else {
            System.out.println("redis中有数据，查询缓存...");
            System.out.println(province_json);
        }
        return province_json;
    }
```

后端servlet代码：

```java
package cn.zero.web.servlet;

import cn.zero.domain.Province;
import cn.zero.service.impl.ProvinceServiceImpl;
import com.fasterxml.jackson.databind.ObjectMapper;

import javax.servlet.annotation.WebServlet;
import java.io.IOException;
import java.util.List;

@WebServlet("/provinceServlet")
public class ProvinceServlet extends javax.servlet.http.HttpServlet {
    protected void doPost(javax.servlet.http.HttpServletRequest request, javax.servlet.http.HttpServletResponse response) throws javax.servlet.ServletException, IOException {
//        // 1. 调用service查询
//        ProvinceServiceImpl service = new ProvinceServiceImpl();
//        List<Province> list = service.findAll();
//        // 2. 序列化list为json
//        ObjectMapper mapper = new ObjectMapper();
//        String json = mapper.writeValueAsString(list);

        ProvinceServiceImpl service = new ProvinceServiceImpl();
        String json = service.findAllJson();
        System.out.println(json);
        // 3. 响应结果
        response.setContentType("application/json;charset=utf-8");
        response.getWriter().write(json);

    }

    protected void doGet(javax.servlet.http.HttpServletRequest request, javax.servlet.http.HttpServletResponse response) throws javax.servlet.ServletException, IOException {
        this.doPost(request, response);
    }
}
```

前端代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Province</title>
    <script src="js/jquery-3.3.1.min.js"></script>
    <script>
        $(function () {
            // 发送ajax请求，加载所有省份数据
            $.get("provinceServlet",{},function (data) {
                // data = [{"id":1,"name":"北京"},{"id":2,"name":"上海"},{"id":3,"name":"广州"},{"id":4,"name":"深圳"}]
                // 1. 获取select
                var province = $("#province");
                // 2. 遍历json数组
                $(data).each(function () {
                    // 3. 创建<option>
                    var option = "<option name='"+this.id+"'>"+this.name+"</option>"
                    // 4. 调用select的append追加option
                    province.append(option);
                });
            });
        });
    </script>
</head>
<body>
    <select name="" id="province">
        <option value="">--请选择省份--</option>
    </select>
</body>
</html>
```

