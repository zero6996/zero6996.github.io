---
title: MyBatis进阶
date: 2019-8-24 23:59
categories: Mybatis
tags: [MyBatis]
description: Mybatis进阶知识点
---



## 1. MyBatis的关联查询

### 1.1 案例：用户和订单

表结构如下：



<!--more-->



![user_orders](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/25/user_orders-1566663584689.jpg)


- user和orders
  - user与orders：一个用户可以创建多个订单，**一对多。**
  - orders与user：多个订单只由一个用户创建，**多对一**。
- orders和orderdetail
  - orders与orderdetail：一个订单可以包括多个订单明细，因为一个订单可以购买多个商品，每个商品购买信息在orderdetail记录，**一对多关系**。
  - orderdetail与orders：多个订单明细包括在一个订单中，**多对一**。
- orderdetail和items
  - orderdetail与items：多个订单明细只对应一个商品信息，**多对一**。
  - items与orderdetail：一个商品可以包括在多个订单明细中，**一对多**。



### 1.2 一对一：resultType实现

复杂查询时，单表对应的PO类已经不能满足结果集的映射。所以要根据需求**建立一个扩展类**来作为resultType的类型。

- 创建订单类

```java
public class Orders {
    private int id;
    private int user_id;
    private String number;
    private Date createtime;
    private String note;
	// 省略get/set，toString
}
```

- 创建订单类的扩展类

```java
// Orders扩展类
public class OrdersExt extends Orders {
    private String username;
    private String address;

    @Override
    public String toString() {
        return "OrdersExt{" +
                "username='" + username + '\'' +
                ", address='" + address + '\'' +
                '}' + super.toString();
    }
	// 省略get/set
}
```

- 声明订单接口

```java
public interface OrderMapper {

    public OrdersExt findOrderById(int id);
}
```

- 声明订单映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--OrderMapper接口映射文件-->
<mapper namespace="com.zero.mapper.OrderMapper">
    <select id="findOrderById" parameterType="int" resultType="ordersExt">
        select
            u.username,u.address,o.*
        from
            orders o,user u
        where
            u.`id`=o.`user_id`
            and o.id = #{id}
    </select>
</mapper>
```

- 加载映射文件

```xml
<mappers>
	<package name="com.zero.mapper"/>
</mappers>
```

- 测试

```java
public class OrderMapperTest {

    private SqlSession session = null;

    @Before
    public void before() throws IOException {
        System.out.println("before:获取session");
        //1. 读取配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapconfig.xml");
        //2. 通过SqlSessionFactoryBuilder创建SqlSessionFactory会话工厂。
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);
        session = sessionFactory.openSession();
    }

    @After
    public void after(){
        System.out.println("after:关闭会话");
        //5. 关闭会话。
        session.commit();
        session.close();
    }
	
    // 测试复杂查询一对一
    @Test
    public void test1(){
        OrderMapper mapper = session.getMapper(OrderMapper.class);
        OrdersExt order = mapper.findOrderById(3);
        System.out.println(order);
    }

}
```



### 1.3 一对一：resultMap实现

- Orders类中新增user属性，添加get/set方法，以存储订单所属用户

```java
public class Orders {
    private Integer id;
    private Integer user_id;
    private String number;
    private Date createtime;
    private String note;

    private User user; // 订单所属用户
    
    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
    // 注意toString方法要添加user信息
}
```

- mapper接口新增方法

```java
public Orders findOrderById2(int id);
```

- 映射文件中添加查询，使用resultMap

```xml
<!--如果模型里面有模型，使用resultMap-->
<resultMap id="orderRslMap" type="orders">
    <!--往orders的模型匹配数据-->
    <id property="id" column="id"/>
    <id property="number" column="number"/>
    <id property="createtime" column="createtime"/>
    <id property="note" column="note"/>

    <!--往orders中的user匹配数据
            使用association进行配置-->
    <association property="user" javaType="user">
        <id property="id" column="user_id"/>
        <id property="username" column="username"/>
        <id property="address" column="address"/>
    </association>

</resultMap>
<select id="findOrderById2" parameterType="int" resultMap="orderRslMap">
    select
    u.username,u.address,o.*
    from
    orders o,user u
    where
    u.`id`=o.`user_id`
    and o.id = #{id}
</select>
```

- 测试

```java
// 测试resultMap一对一
@Test
public void test2(){
    OrderMapper mapper = session.getMapper(OrderMapper.class);
    Orders order = mapper.findOrderById2(3);
    System.out.println(order);
}
```



- 小结：
  - resultType：使用resultType实现较为简单，如果POJO中没有包括查询出来的列名， 则必须增加列名对应的属性，才可完成映射。**如果没有查询结果的特殊要求建议使用resultType**。
  - resultMap：需单独定义resultMap，实现较麻烦，如果对**查询结果有特殊的要求**，使用resultMap可以完成将关联查询映射到POJO的对象属性中。
  - resultMap可以实现延迟加载，resultType无法实现延迟加载。



### 1.4 一对多

- 需求：根据订单ID查找订单信息、用户信息和订单明细信息。

```sql
-- 根据订单ID查找订单信息、用户信息和订单明细信息。

select 
  u.`username`,
  u.`address`,
  o.*,
  od.id detail_id,
  od.items_id,
  od.items_num
from 
  user u,
  orders o,
  orderdetail od
where 
  u.`id`=o.`user_id` 
  and o.`id`=od.orders_id
  and o.`id` = 3;
```

- 新建OrderDetail类

```java
// 订单详情实体类
public class OrderDetail {
    private Integer id; // 订单详情id
    private Integer items_id; // 商品id
    private Integer items_num; // 商品数量
	// 省略get/set，toString
}
```

- 修改Orders类，添加订单明细属性

```java
public class Orders {
    private Integer id;
    private Integer user_id;
    private String number;
    private Date createtime;
    private String note;

    private User user; // 订单所属用户

    // 一对多数据封装：订单明细
    private List<OrderDetail> orderDetails;
}
```

- mapper接口

```java
public interface OrderMapper {

    public OrdersExt findOrderById(int id);
    public Orders findOrderById2(int id);

    // 一对多关系映射配置
    public Orders findOrderById3(int id);

}
```

- 映射文件添加查询,resultMap中有个extends属性，可继承父类配置的数据映射关系。

```xml
<!-- =========== 一对多 ============= -->
<resultMap id="orderRslMap2" type="orders" extends="orderRslMap"> <!--如果继承了orderRslMap 下面的orders和user模型无需再次编写。-->
    <!--往orders的模型匹配数据-->
    <id property="id" column="id"/>
    <id property="number" column="number"/>
    <id property="createtime" column="createtime"/>
    <id property="note" column="note"/>

    <!--往orders中的user匹配数据
            使用association进行配置-->
    <association property="user" javaType="user">
        <id property="id" column="user_id"/>
        <id property="username" column="username"/>
        <id property="address" column="address"/>
    </association>
    <!-- 一对多匹配：往orders中的orderDetails匹配数据
            注意：集合里类型需使用ofType，而不是javaType-->
    <collection property="orderDetails" ofType="orderDetail">
        <id property="id" column="detail_id"/>
        <id property="items_id" column="items_id"/>
        <id property="items_num" column="items_num"/>
    </collection>
</resultMap>

<select id="findOrderById3" parameterType="int" resultMap="orderRslMap2">
    select
        u.`username`,
        u.`address`,
        o.*,
        od.id detail_id,
        od.items_id,
        od.items_num
    from
        user u,
        orders o,
        orderdetail od
    where
        u.`id`=o.`user_id`
        and o.`id`=od.orders_id
        and o.`id` = #{id};
</select>
```

- 测试

```java
// 测试resultMap一对一
@Test
public void test3(){
    OrderMapper mapper = session.getMapper(OrderMapper.class);
    Orders order = mapper.findOrderById3(3);
    System.out.println(order);
}
```



#### 1.4.1 小结

Mybatis使用resultMap的collection对关联查询的多条记录映射到一个list集合属性中。



### 1.5 多对多

- 需求：查询用户信息及用户购买的商品信息，要求将关系信息映射到主POJO的POJO属性中。

```sql
SELECT 
  u.`id`,
  u.`username`,
  u.`address`,
  o.`id` order_id,
  o.`number`,
  o.`createtime`,
  o.`note`,
  od.`id` detail_id,
  od.`items_id`,
  od.`items_num`,
  it.`name`,
  it.`price`,
  it.`detail`
FROM 
  user u,
  orders o,
  orderdetail od,
  items it
WHERE 
  u.`id` = o.`user_id`
  and o.`id` = od.`orders_id`
  and od.`items_id` = it.`id`;
```

- 映射思路
  - 将用户信息映射到user中。
  - 在user类中添加订单列表属性`List<Orders> orderslist`，将用户创建的订单映射到orderslist中。
  - 在Orders中添加订单明细列表属性`List<Orderdetail> detailList`，将订单的明细映射到detailList中。
  - 在Orderdetail中添加Items属性，将订单明细所对应的商品映射到Items中。



- mapper中添加方法

```java
public interface UserMapper {
	// 多对多查询:查询用户信息及用户购买的商品信息
    public List<User> findUserAndOrderInfo();
}
```

- 在user类中添加订单列表属性`List<Orders> orderslist`，将用户创建的订单映射到orderslist中。

```java
public class User implements Serializable {
    private int id;
    private String username;
    private String sex;
    private Date birthday;
    private String address;

    private List<Orders> orderList; // 一个用户有多张订单
}
```

- 在Orders中添加订单明细列表属性`List<Orderdetail> detailList`，将订单的明细映射到detailList中。

```java
public class Orders {
    private Integer id;
    private Integer user_id;
    private String number;
    private Date createtime;
    private String note;

    private User user; // 订单所属用户

    // 一对多数据封装：订单明细
    private List<OrderDetail> orderDetails;
}
```

- 新建items类

```java
public class Items {
    private Integer id;
    private String name;
    private Float price;
    private String detail;
}
```

- 在Orderdetail中添加Items属性，将订单明细所对应的商品映射到Items中。

```java
public class OrderDetail {
    private Integer id; // 订单详情id
    private Integer items_id; // 商品id
    private Integer items_num; // 商品数量

    private Items items; // 订单详情
}
```

- 映射文件中配置映射关系，查询方法

```xml
<!-- 多对多查询:查询用户信息及用户购买的商品信息 -->
<resultMap id="userRslMap" type="user">
    <!--1. 配置user属性-->
    <id property="id" column="id"/>
    <result property="username" column="username"/>
    <result property="address" column="address"/>

    <!--2. 配置user的orderList-->
    <collection property="orderList" ofType="orders">
        <id property="id" column="order_id"/>
        <result property="number" column="number"/>
        <result property="createtime" column="createtime"/>
        <result property="note" column="note"/>

        <!--3. 配置orders的orderDetail-->
        <collection property="orderDetails" ofType="orderDetail">
            <id property="id" column="detail_id"/>
            <result property="items_id" column="items_id"/>
            <result property="items_num" column="items_num"/>

            <!--4. 匹配订单详情的商品信息：在Orderdetail中添加Items属性，将订单明细所对应的商品映射到Items中。-->
            <association property="items" javaType="items">
                <id property="id" column="items_id"/>
                <result property="name" column="name"/>
                <result property="price" column="price"/>
                <result property="detail" column="detail"/>
            </association>
        </collection>
    </collection>
</resultMap>

<select id="findUserAndOrderInfo" resultMap="userRslMap">
    SELECT
        u.`id`,
        u.`username`,
        u.`address`,
        o.`id` order_id,
        o.`number`,
        o.`createtime`,
        o.`note`,
        od.`id` detail_id,
        od.`items_id`,
        od.`items_num`,
        it.`name`,
        it.`price`,
        it.`detail`
    FROM
        user u,
        orders o,
        orderdetail od,
        items it
    WHERE
        u.`id` = o.`user_id`
        and o.`id` = od.`orders_id`
        and od.`items_id` = it.`id`
</select>
```

- 测试

```java
/**
 * 多对多查询信息：查询用户信息及用户购买的商品信息
 */
@Test
public void test2() {
    UserMapper mapper = session.getMapper(UserMapper.class);
    List<User> infos = mapper.findUserAndOrderInfo();
    for (User user:infos){
        System.out.println("用户信息："+user);
        for (Orders order:user.getOrderList()){
            System.out.println("订单信息："+order);
            for (OrderDetail od:order.getOrderDetails()){
                System.out.println("订单详情：" + od);
            }
            System.out.println("<--------------------------------->");
        }
    }
}
```

- 输出结果

```
用户信息：User{id=1, username='赵六', sex='null', birthday=null, address='杭州市'}
订单信息：Orders{id=3, user_id=null, number='1000010', createtime=Wed Feb 04 13:22:35 CST 2015, note='null', user=null, orderDetails=[OrderDetail{id=1, items_id=1, items_num=1, items=Items{id=1, name='台式机', price=3000.0, detail='该电脑质量非常好！！！！'}}, OrderDetail{id=2, items_id=2, items_num=3, items=Items{id=2, name='笔记本', price=6000.0, detail='笔记本性能好，质量好！！！！！'}}]}
订单详情：OrderDetail{id=1, items_id=1, items_num=1, items=Items{id=1, name='台式机', price=3000.0, detail='该电脑质量非常好！！！！'}}
订单详情：OrderDetail{id=2, items_id=2, items_num=3, items=Items{id=2, name='笔记本', price=6000.0, detail='笔记本性能好，质量好！！！！！'}}
<--------------------------------->
订单信息：Orders{id=4, user_id=null, number='1000011', createtime=Tue Feb 03 13:22:41 CST 2015, note='null', user=null, orderDetails=[OrderDetail{id=3, items_id=3, items_num=4, items=Items{id=3, name='背包', price=200.0, detail='名牌背包，容量大质量好！！！！'}}, OrderDetail{id=4, items_id=2, items_num=3, items=Items{id=2, name='笔记本', price=6000.0, detail='笔记本性能好，质量好！！！！！'}}]}
订单详情：OrderDetail{id=3, items_id=3, items_num=4, items=Items{id=3, name='背包', price=200.0, detail='名牌背包，容量大质量好！！！！'}}
订单详情：OrderDetail{id=4, items_id=2, items_num=3, items=Items{id=2, name='笔记本', price=6000.0, detail='笔记本性能好，质量好！！！！！'}}
<--------------------------------->
```



#### 1.5.1 小结

- resultType：将查询结果安装sql列名和POJO属性名一致性映射到POJO中。
- resultMap：使用association和collection完成一对一和一对多高级映射（对结果有特殊的映射要求）
  - association：将关联查询信息映射到一个POJO对象中。
  - collection：将关联查询信息映射到一个list集合中。

> - 模型里面有模型，用association
> - 模型里面有集合，用collection



## 2. 延时加载

延迟加载又叫懒加载，也叫按需加载。即先加载主信息，在需要的时候，再去加载从信息。

在Mybatis中，resultMap标签的association标签和collection标签具有延迟加载的功能。

- 分别在两个接口中添加方法

```java
public interface UserMapper {
    public User findUserById(int id);
}
public interface OrderMapper {
	// 懒加载订单的用户数据
	public List<Orders> findOrderAndUserByLazyloading();
}
```

- 映射文件

```xml
<!--UserMapper映射文件-->
<mapper namespace="com.zero.mapper.UserMapper">
	<select id="findUserById" parameterType="int" resultType="user">
		SELECT * FROM USER WHERE id = #{id}
	</select>
</mapper>
<!--OrderMapper接口映射文件-->
<mapper namespace="com.zero.mapper.OrderMapper">
	<!-- =========== 懒加载 ============= -->
    <resultMap id="orderLazyloadingRslMap" type="orders">
        <id property="id" column="id"/>
        <result property="note" column="note"/>
        <result property="number" column="number"/>
        <result property="createtime" column="createtime"/>
        <!-- 配置懒加载查询语句，column是子查询所需参数-->
        <association property="user" select="com.zero.mapper.UserMapper.findUserById" column="user_id"/>
    </resultMap>
    <select id="findOrderAndUserByLazyloading" resultMap="orderLazyloadingRslMap">
        select * from orders
    </select>
</mapper>
```

- 测试

```java
// 懒加载
@Test
public void test４(){
    OrderMapper mapper = session.getMapper(OrderMapper.class);
    List<Orders> orders = mapper.findOrderAndUserByLazyloading();
    for (Orders order:orders){
        System.out.println("订单信息："+order);
        System.out.println("订单用户："+order.getUser());
    }

}
```

- 全局配置文件中配置懒加载

```xml
<configuration>
    <!--懒加载配置-->
    <settings>
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>
</configuration>
```



## 3. 查询缓存

Mybatis提供查询缓存，用于减轻数据压力，提高数据库抗压力。

Mybatis的缓存，包括**一级缓存和二级缓存**，一级缓存是默认使用的。二级缓存需手动开启。



![cache](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/25/cache-1566663654286.jpg)

在操作数据库时需构造SqlSession对象，在对象中有一个数据结构(HashMap)用于缓存数据。不同的SqlSession之间的缓存数据区域是互不影响的。

- Mybatis一级缓存的作用域是同一个SqlSession，在同一个sqlSession中两次执行相同的sql语句，第一次执行完毕会将数据库中查询的数据写到缓存（内存），第二次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。当一个sqlSession结束后该sqlSession中的一级缓存也就不存在了。Mybatis默认开启一级缓存。
- Mybatis二级缓存是**多个SqlSession共享**的，其作用域是mapper的同一个namespace，不同的sqlSession两次执行相同namespace下的sql语句且向sql中传递参数也相同即最终执行相同的sql语句，第一次执行完毕会将数据库中查询的数据写到缓存（内存），第二次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。Mybatis默认没有开启二级缓存，需要在setting全局参数中配置开启二级缓存。



### 3. 1 一级缓存原理

![OneLevelCache](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/25/OneLevelCache-1566663673662.jpg)

- 第一次发起查询，会先去找缓存中是否有id为1的用户信息，如果没有，从数据库中查询用户信息。获取信息后再将用户信息存储到一级缓存中。

- 第二次发起查询用户id为1的用户信息时，先去缓存中找，如缓存中有，直接从缓存中获取用户信息。
- 如果SqlSession执行了commit操作(执行插入、更新、删除)，会清空SqlSession中的一级缓存，以确保缓存中存储的是最新数据，避免脏读。
- Mybatis内部存储缓存使用一个HashMap。key为HashCode+sqlId+Sql语句，value是查询出来映射生成的java对象。



- 测试

```java
@Test
public void test1() throws IOException {
    //1. 读取配置文件
    InputStream is = Resources.getResourceAsStream("SqlMapconfig.xml");
    //2. 通过SqlSessionFactoryBuilder创建SqlSessionFactory会话工厂。
    SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);
    // 3. 会话
    SqlSession session = sessionFactory.openSession();
    // 4. 通过会话获取dao接口
    UserMapper mapper = session.getMapper(UserMapper.class);
    /**
         * 默认情况下，一级缓存也就是session级别缓存是开启的
         * 保存、删除、更新操作，一级缓存数据会自动清空
         */
    User user1 = mapper.findUserById(1);
    System.out.println("user1"+user1);
    // 保存用户,测试缓存是否清空
    mapper.save(new User("小花","1",null,"杭州"));
    session.commit();
    User user2 = mapper.findUserById(1);
    System.out.println("user2"+user1);

}
```


### 3.2 二级缓存原理

![TwoLevelCache](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/25/TwoLevelCache-1566663699382.jpg)

- SqlSession1去查询用户id为1的用户信息，查询到用户信息后会将查询数据存储到二级缓存中。
- SqlSession2去查询用户id为1的用户信息时，会先去缓存中找是否有数据，有则直接从缓存中取。
- 二级缓存区域是根据mapper的namespace划分的，同一个namespace的mapper查询数据放在同一个区域，如果使用mapper代理方法每个mapper的namespace都不同，此时可以理解为二级缓存区域是根据mapper划分。
- 每次查询会先从缓存区域找，如果找不到从数据库查询，查询到数据将数据写入缓存。
- sqlSession执行insert、update、delete等操作commit提交后会清空缓存区域。
- 开启二级缓存需在配置文件中手动配置。



#### 3.2.1 代码示例

- 开启二级缓存总开关

```xml
<settings>
    <!--懒加载配置-->
    <setting name="lazyLoadingEnabled" value="true"/>
    <!--开启二级缓存-->
    <setting name="cacheEnabled" value="true"/>
</settings>
```

- 在UserMapper中配置二级缓存

```xml
<mapper namespace="com.zero.mapper.UserMapper">
    <!-- 配置缓存：该类下所有查询方法都使用二级缓存
        type不写，默认使用的是mybatis自带的缓存技术：perpetualCache -->
    <cache></cache>
</mapper>
```

- User类序列化

```java
public class User implements Serializable {}
```

- 测试

```java
// 二级缓存
@Test
public void test2() throws IOException {
    //1. 读取配置文件
    InputStream is = Resources.getResourceAsStream("SqlMapconfig.xml");
    //2. 通过SqlSessionFactoryBuilder创建SqlSessionFactory会话工厂。
    SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);
    // 3. 会话
    SqlSession session1 = sessionFactory.openSession();
    SqlSession session2 = sessionFactory.openSession();
    SqlSession session3 = sessionFactory.openSession();
    // 4. 通过会话获取dao接口
    UserMapper mapper1 = session1.getMapper(UserMapper.class);
    UserMapper mapper2 = session2.getMapper(UserMapper.class);
    UserMapper mapper3 = session3.getMapper(UserMapper.class);

    User user1 = mapper1.findUserById(1);
    System.out.println("user1"+user1);
    // session关闭后才会写入二级缓存
    session1.close();

    // 执行修改更新删除等操作，提交后，会清空二级缓存
    mapper3.save(user1);
    session3.commit();

    User user2 = mapper2.findUserById(1);
    System.out.println("user2"+user2);

    session3.close();

}

```

- 关于禁用指定方法二级缓存

```xml
<select id="findUserById" parameterType="int" resultType="user" useCache="false">
     SELECT * FROM USER WHERE id = #{id}
</select>
```

- 关于刷新缓存

```xml
<!--
        默认情况下，插入、更新、删除会清空二级缓存
        默认flushCache="true"
        如果设置为false，插入、更新、删除就不会清空二级缓存
    -->
<insert id="save" parameterType="user" flushCache="false">
    INSERT into user (username,sex,birthday,address)
    values (#{username},#{sex},#{birthday},#{address})
</insert>
```



### 3.3 整合Ehcache

EhCache 是一个纯Java的进程内缓存框架，具有快速、精干等特点，是一个分布式的缓存框架。

Mybatis本身是一个持久层框架，它不是专门的缓存框架，所以它对缓存的实现不够好，不能支持分布式。

- 分布式：系统为了提高性能，通常会对系统采用分布式部署(集群部署方式)

- 整合思路

  - Cache是一个借口，它的默认实现是mybatis的PerpetualCache。如果想整合mybatis的二级缓存，那么实现Cache接口即可。

- 实现步骤：

  - 先添加jar包

  ![ehcache](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/25/ehcache-1566663742344.jpg)

  - 设置映射文件中的cache标签的type值为ehcache的实现类

  ```xml
  <mapper namespace="com.zero.mapper.UserMapper">
      <!-- 配置缓存：该类下所有查询方法都使用二级缓存
          type不写，默认使用的是mybatis自带的缓存技术：perpetualCache -->
      <cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
  </mapper>
  ```

  - 在src下添加ehcache的配置文件

  ```xml
  <ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
      
      <diskStore path="java.io.tmpdir"/>
      <!--
      	maxElementsInMemory :设置基于内存的缓存中可存放的对象最大数目
      	eternal:设置对象是否为永久的,true表示永不过期,此时将忽略
      	timeToIdleSeconds 和 timeToLiveSeconds属性; 默认值是false
      	timeToIdleSeconds:设置对象空闲最长时间,以秒为单位, 超过这个时间,对象过期。
          当对象过期时,EHCache会把它从缓存中清除。如果此值为0,表示对象可以无限期地处于空闲状态。
      	timeToLiveSeconds:设置对象生存最长时间,超过这个时间,对象过期。
          如果此值为0,表示对象可以无限期地存在于缓存中. 该属性值必须大于或等于 timeToIdleSeconds 属性值
      	overflowToDisk:设置基于内在的缓存中的对象数目达到上限后,是否把溢出的对象写到基于硬盘的缓存中
      	diskPersistent 当jvm结束时是否持久化对象 true false 默认是false
      	diskExpiryThreadIntervalSeconds 指定专门用于清除过期对象的监听线程的轮询时间
          memoryStoreEvictionPolicy - 当内存缓存达到最大，有新的element加入的时候，
          移除缓存中element的策略。默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出）
      -->
      <defaultCache
              maxElementsInMemory="10000"
              eternal="false"
              timeToIdleSeconds="120"
              timeToLiveSeconds="120"
              maxElementsOnDisk="10000000"
              diskExpiryThreadIntervalSeconds="120"
              memoryStoreEvictionPolicy="LRU">
          <persistence strategy="localTempSwap"/>
      </defaultCache>
  </ehcache>
  ```

  - 测试：用上一个二级缓存案例即可

- 二级缓存应用场景
  
  - 对于访问响应速度要求高，但是实时性不高的查询，可以采用二级缓存技术。

## 4. MyBatis整合Spring

### 4.1 创建工程导包

- 导入Mybatis包及依赖包

![mybatis](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/25/mybatis-1566663766728.jpg)

![mybatis_relyon](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/25/mybatis_relyon-1566663784695.jpg)


- 导入mysql数据库驱动：`mysql-connector-java-5.1.7-bin.jar`

- DBCP连接池相关包：`commons-dbcp-1.4.jar`和`commons-pool-1.6.jar`
- spring+mvc相关包：

![spring_mvc](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/25/spring_mvc-1566663800152.jpg)

- Mybatis-spring整合包：`mybatis-spring-1.2.2.jar`



### 4.2 配置mybatis的核心配置文件

- 创建核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--加载配置文件-->
<!--    <properties resource="db.properties"/>-->
    <!--别名配置-->
    <typeAliases>
        <!-- 批量配置别名-->
        <!-- [name]：指定批量定义别名的类包，别名为类名，第一个字母小写-->
        <package name="com.zero.sm.model"/>
    </typeAliases>
    
    <!--加载映射文件-->
    <mappers>
        <package name="com/zero/sm/sqlmap/User.xml"/>
        <!-- 加载mapper包下所有映射文件-->
        <package name="com.zero.sm.mapper"/>
    </mappers>
</configuration>
```

- 创建User模型

```java
public class User implements Serializable {
    private int id;
    private String username;
    private String sex;
    private Date birthday;
    private String address;
	// 省略get、set、toString
}
```

- User映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="user">
    <select id="findUserById" parameterType="int" resultType="user" >
		SELECT * FROM USER WHERE id = #{id}
	</select>
</mapper>
```



### 4.3 配置spring数据源

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-3.2.xsd
		http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-3.2.xsd ">


    <!-- 1.配置数据库，dbcp数据库连接池 -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql:///mybatis01?useUnicode=true&amp;characterEncoding=utf8"/>
        <property name="username" value="root"/>
		<property name="password" value="123456"/>
		<!-- 最大连接  -->
        <property name="maxActive" value="10"/>
        <!--最大空闲数  -->
        <property name="maxIdle" value="5"/>
    </bean>
</beans>
```



### 4.4 spring配置SqlSessionFactory

```xml
 <!--2. 配置会话工厂-->
<bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!--引用数据源-->
    <property name="dataSource" ref="dataSource"/>
    <!--设置全局配置文件路径-->
    <property name="configLocation" value="classpath:SqlMapconfig.xml"/>
</bean>
```



### 4.5 编写UserDaoImpl，省略接口

```java
public class UserDaoImpl extends SqlSessionDaoSupport implements UserDao {
    @Override
    public User findUserById(int id) {
        return getSqlSession().selectOne("user.findUserById",id); // 调用父类方法获取会话对象，然后执行方法
    }
}
```



### 4.6 spring中配置daobean

```java
<!--3. 配置dao-->
<bean id="userDao" class="com.zero.sm.dao.Impl.UserDaoImpl">
    <property name="sqlSessionFactory" ref="sessionFactory"/>
</bean>
```



### 4.7 测试

```java
public class Demo01 {
    @Test
    public void test1(){
        // 1. 加载spring的配置文件
        ApplicationContext context = new ClassPathXmlApplicationContext("springConfig.xml");
        // 2. 获取dao的bean
        UserDao userDao = (UserDao) context.getBean("userDao");
        // 3. 调用dao方法
        User user = userDao.findUserById(1);
        System.out.println(user);
    }
}
```



### 4.8 换成Mapper接口整合dao

- 创建Mapper映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.zero.sm.mapper.UserMapper">
    <select id="findUserById" parameterType="int" resultType="user" >
		SELECT * FROM USER WHERE id = #{id}
	</select>
</mapper>
```



- 加载映射文件

```xml
<mappers>
    <mapper resource="com/zero/sm/sqlmap/User.xml"/>
    <!-- 加载mapper包下所有映射文件-->
    <package name="com.zero.sm.mapper"/>
</mappers>
```

- spring配置MapperFactoryBean

```xml
<!-- 第二种方法-->
<!--4. 使用工厂Bean生成userMapper对象-->
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
    <property name="sqlSessionFactory" ref="sessionFactory"/>
    <property name="mapperInterface" value="com.zero.sm.mapper.UserMapper"/>
</bean>
```

- 测试

```java
 @Test
public void test2() {
    // 1. 加载spring的配置文件
    ApplicationContext context = new ClassPathXmlApplicationContext("springConfig.xml");
    // 2. 获取dao的bean
    UserMapper userMapper = (UserMapper) context.getBean("userMapper");
    User us = userMapper.findUserById(1);
    System.out.println(us);
}
```

### 4.9 使用`MapperScannerConfigurer`批量扫描创建代理对象


mapper代理开发方式批量mapper配置  ,内部会扫描指定包下的mapper，创建代理对象，bean的名字默认为mapper接口类名的首字母小写。


```xml
 <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
     <property name="basePackage" value="com.zero.sm.mapper"/>
     <property name="sqlSessionFactoryBeanName" value="sessionFactory"/>
</bean>
```

测试方法同上案例。

> Notice：JDK1.8以上使用该方式，无法创建bean，改成JDK1.7即可；或spring版本3.2.9以上。



## 5. 逆向工程

### 5.1 简介

简单的说，就是通过数据库中的表单，自动生成java代码。

Mybatis官方提供了逆向工程，可以针对单表自动生成Mybatis代码(`mapper.java\mapper.xml\po类`)

- [下载地址](https://github.com/mybatis/generator/releases)



### 5.2 使用方法

1. 创建简单的java项目；
2. 导入jar包，创建generator配置文件；
3. 使用java类来执行逆向工程；
4. 把生成的代码拷贝到项目中；
5. 在正式项目中使用逆向工程生成的代码。



### 5.3 案例示范

#### 5.3.1  第一步：创建generator配置文件

在src下，创建generator配置文件，文件内容可以从官方docs目录下的index.html中找到相关代码

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

    <context id="mysqlTables" targetRuntime="MyBatis3">
        <!--数据库配置-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql:///mybatis01"
                        userId="root"
                        password="123456">
        </jdbcConnection>

        <!--java类型解析-->
        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!--模型生成包名-->
        <javaModelGenerator targetPackage="com.zero.sm.model" targetProject=".\src">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!--mybatis的映射 .xml-->
        <sqlMapGenerator targetPackage="com.zero.sm.mapper"  targetProject=".\src">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!--mybatis的mapper接口生成包路径-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.zero.sm.mapper"  targetProject=".\src">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!-- 配置生成表的模型
            domainObjectName: 默认会根据表名生成，首字母自动大写
            -->
        <table tableName="items"/>
        <table tableName="orderdetail" domainObjectName="OrderDetail"/>
        <table tableName="orders"/>
        <table tableName="user"/>

    </context>
</generatorConfiguration>
```



#### 5.3.2  第二步：使用java类来执行逆向工程

需导入mysql驱动包和mybatis的逆向工程包

```java
public class Main {
    public static void main(String[] args)  throws Exception{
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        File configFile = new File("src/generator.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config,
                callback, warnings);
        myBatisGenerator.generate(null);
    }
}
```



#### 5.3.3  第三步：把生成的代码拷贝到项目中

![AutoGenerator](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/25/AutoGenerator-1566663834855.jpg)




#### 5.3.4  第四步：测试

```java
// 测试基本查询
@Test
public void test2() {
    // 1. 加载spring的配置文件
    ApplicationContext context = new ClassPathXmlApplicationContext("springConfig.xml");
    // 2. 获取dao的bean
    UserMapper userMapper = (UserMapper) context.getBean("userMapper");

    // 3. 通过主键查询对象
    User user = userMapper.selectByPrimaryKey(1);
    System.out.println(user);

}

// 测试Example
@Test
public void test3() {
    // 自动生成的Example是用于查询的
    // 1. 加载spring的配置文件
    ApplicationContext context = new ClassPathXmlApplicationContext("springConfig.xml");
    // 2. 获取dao的bean
    UserMapper userMapper = (UserMapper) context.getBean("userMapper");

    UserExample example = new UserExample();
    // 封装查询条件
    UserExample.Criteria criteria = example.createCriteria();
    criteria.andSexEqualTo("1"); // 查询男性用户
    // 通过封装后的查询条件查询
    List<User> users = userMapper.selectByExample(example);
    System.out.println(users);
}
```

