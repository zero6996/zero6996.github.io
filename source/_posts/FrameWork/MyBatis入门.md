---
title: MyBatis基础入门
date: 2019-8-21 22:50
categories: FrameWork
tags: [MyBatis]
description: Mybatis基础入门
urlname: MyBatis-Basic
---



## 1. MyBatis简介

MyBatis本是apache的一个开源项目iBatis，2010年这个项目由 apache software foundation迁移到了google code，并改名为MyBatis，实质上MyBatis是对iBatis进行进行了一些改进。



<!--more-->



- Mybatis是一个优秀的持久层框架，它对jdbc的操作数据库过程进行封装，使开发者只需要关注SQL本身，而不需要花费精力去处理例如注册驱动、创建connection、创建statement、手动设置参数、结果集检索等jdbc繁杂的过程代码。
- 对jdbc封装的框架有：Hibernate、DBUtils、jdbcTemplate、Mybatis
- Mybatis原理：
  - 通过**XML或注解**的方式将要执行的各种statement(preparedStatement)配置起来，并通过java对象和statement中的sql进行映射生成最终执行的sql语句，最后由mybatis框架执行sql并将结果映射成java对象返回。
- MyBatis框架核心
  1. mybatis配置文件：包括全局配置文件和映射文件。其中全局配置文件配置了数据源、事务等信息；映射文件配置了SQL执行相关的信息。
  2. mybatis通过读取配置文件信息(全局配置文件和映射文件)，构造出SQLSessionFactory，即会话工厂。
  3. 通过SQLSessionFactory，可以创建SQLSession(会话)。Mybatis是通过SQLSession来操作数据库的。
  4. SQLSession本身不能直接操作数据库，它是通过底层的Executor执行器接口来操作数据库的。Executor接口有两个实现类，一个普通执行器，一个是缓存执行器(默认)。
  5. Executor执行器要处理的SQL信息是封装到一个底层对象MappedStatement中。该对象包括：SQL语句、输入参数映射信息、输出信息集映射信息。其中输入参数和输出结果的映射类型包括HashMap集合对象、POJO对象类型。



> POJO：Plain Ordinary Java Object，简单的Java对象，实际就是普通JavaBeans，是为了避免和EJB混淆所创造的简称



## 2. MyBatis入门

### 2.1 环境准备

- 下载Mybatis
  - [下载地址](https://github.com/mybatis/mybatis-3/releases)
- 创建项目导包

![mybatis](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/21/mybatis-1566398238994.jpg)


- 添加日志配置

  - 项目src下创建log4j.properties文件，内容如下：

  ```properties
  # Global logging configuration
  log4j.rootLogger=DEBUG, stdout
  # Console output...
  log4j.appender.stdout=org.apache.log4j.ConsoleAppender
  log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
  log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
  ```

  

### 2.2 开发步骤

1. 根据需求创建PO类
2. 创建全局配置文件：SqlMapConfig.xml
3. 编写映射文件
4. 在SqlMapConfig.xml中加载映射文件
5. 编写测试代码，连接并操作数据库
   - 读取配置文件
   - 通过SqlSessionFactoryBuilder创建SqlSessionFactory会话工厂。
   - 通过SqlSessionFactory创建SqlSession。
   - 调用SqlSession的操作数据库方法。
   - 关闭会话。



#### 2.2.1 创建PO类

```java
public class User implements Serializable {
    private int id;
    private String username;
    private String sex;
    private Date birthday;
    private String address;
	// 省略get/set
    // 省略toString
}
```



#### 2.2.2 创建全局配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 配置mybatis的环境信息 -->
    <environments default="development">
        <environment id="development">
            <!-- 配置JDBC事务控制，由mybatis进行管理 -->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 配置数据源，采用dbcp连接池 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///mybatis01?useUnicode=true&amp;characterEncoding=utf8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
</configuration>

```



#### 2.2.3 编写映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
	namespace：命名空间，它的作用就是对SQL进行分类化管理，可以理解为SQL隔离
	注意：使用mapper代理开发时，namespace有特殊且重要的作用
 -->
<mapper namespace="test">
    <!--
        [id]：statement的id，要求在命名空间内唯一
        [parameterType]：参数输入类型
        [resultType]：查询出的单条结果集对应的java类型，返回值类型
        [#{}]： 表示一个占位符?
        [#{id}]：表示该占位符待接收参数的名称为id。
            注意：如果参数为简单类型时，#{}里面的参数名称可以是任意定义
     -->
    <select id="findUserById" parameterType="int" resultType="com.zero.model.User">
		SELECT * FROM USER WHERE id = #{id1}
	</select>

</mapper>
```



#### 2.2.4 加载映射文件

```xml
<!--加载映射文件-->
<mappers>
    <mapper resource="com/zero/sqlmap/User.xml"/>
</mappers>
```



#### 2.2.5 编写测试

```java
public class UserTest {

    @Test
    public void test1() throws IOException {
        //1. 读取配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapconfig.xml");
        //2. 通过SqlSessionFactoryBuilder创建SqlSessionFactory会话工厂。
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);
        //3. 通过SqlSessionFactory创建SqlSession。
        SqlSession session = sessionFactory.openSession();
        //4. 调用SqlSession的操作数据库方法。
        User user = session.selectOne("findUserById",10);
        System.out.println(user);
        //5. 关闭会话。
        session.commit();
    }
}
```



> junit4.12可能会报`java.lang.NoClassDefFoundError: org/hamcrest/SelfDescribing`，版本切换为5.4后不会报错。



#### 2.2.6 查询案例

##### 1. 模糊查询

- 模糊查询，修改映射文件，添加如下内容：

```xml
<!--
        模糊查询用户信息：
            [${}]：表示拼接SQL字符串
             [${value}]：表示要拼接的是简单类型参数。
             注意：
            1、如果参数为简单类型时，${}里面的参数名称必须为value
            2、${}会引起SQL注入，一般情况下不推荐使用。但是有些场景必须使用${}，比如order by ${colname}
        -->
<select id="findUserByName" parameterType="String" resultType="com.zero.model.User">
    SELECT * FROM USER WHERE username like '%${value}%'
</select>



```

- 测试文件添加查询语句

```java
// 通过姓名模糊查询用户集合
List<User> users = session.selectList("findUserByName", "张");
System.out.println(users);
```



##### 2. 添加用户

- 修改映射文件，添加内容

```xml
<!--插入用户信息-->
<insert id="insertUser" parameterType="com.zero.model.User">
    <!--如果主键的值是通过MySQL自增机制生成的，那么我们此处无需再‘显式’的给ID赋值-->
    INSERT into user (username,sex,birthday,address) values (#{username},#{sex},#{birthday},#{address})
</insert>
```

- 测试代码，注意：需在PO类User中添加一个没有id属性的全参构造方法

```java
// 插入用户
session.insert("insertUser",new User("zero","男",new Date(),"杭州"));

```



##### 3. 删除用户

- 修改映射文件

```xml
<!--通过id删除用户-->
<delete id="deleteUserById" parameterType="int">
    delete from user where id = #{id}
</delete>
<!--通过姓名删除用户-->
<delete id="deleteUserByName" parameterType="String">
    delete from user where username = #{name}
</delete>
```

- 测试代码

```java
// 通过id删除用户
session.delete("deleteUserById",28);
// 通过姓名删除用户，test是映射文件的命名空间
session.delete("test.deleteUserByName","张小明");
```



##### 4. 更新用户

- 映射文件

```xml
<!--通过id更新用户-->
<update id="updateUserById" parameterType="com.zero.model.User">
    update user set username=#{username},sex=#{sex} where id=#{id}
</update>
```

- 测试代码

```java
// 通过id更新用户
User upUser = new User();
upUser.setId(1); // 输入要修改的用户id
upUser.setUsername("赵六儿");// 需修改的用户姓名
upUser.setSex("女");
session.update("updateUserById",upUser);
```



#### 2.2.7 返回MySQL自增主键

- MySQL自增主键，是指在insert之前MySQL会自动生成一个自增的主键
- 可以通过MySQL的函数获取刚插入的自增主键：`select LAST_INSERT_ID()`，该函数在insert语句之后调用。

- 修改映射文件添加用户内容

```xml
<!--插入用户信息-->
<insert id="insertUser" parameterType="com.zero.model.User">
    <!--
   [selectKey标签]：通过select查询来生成主键
   [keyProperty]：指定存放生成主键的属性
   [resultType]：生成主键所对应的Java类型
   [order]：指定该查询主键SQL语句的执行顺序，相对于insert语句
   [last_insert_id]：MySQL的函数，要配合insert语句一起使用 -->
    <selectKey keyProperty="id" resultType="int" order="AFTER">
        SELECT LAST_INSERT_ID()
    </selectKey>
    <!--如果主键的值是通过MySQL自增机制生成的，那么我们此处无需再‘显式’的给ID赋值-->
    INSERT into user (username,sex,birthday,address) values (#{username},#{sex},#{birthday},#{address})
</insert>
```

- 使用Junit4的注解功能，抽取重复代码

```java
private SqlSession session = null;
@Before
public void before() throws IOException {
    System.out.println("before:获取session");
    //1. 读取配置文件
    InputStream is = Resources.getResourceAsStream("SqlMapconfig.xml");
    //2. 通过SqlSessionFactoryBuilder创建SqlSessionFactory会话工厂。
    SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);
    //3. 通过SqlSessionFactory创建SqlSession。
    session = sessionFactory.openSession();
}

@After
public void after(){
    System.out.println("after:关闭会话");
    //5. 关闭会话。
    session.close();
}
```

- 测试代码

```java
 // 插入用户后，将主键返回，设置到模型id中
@Test
public void test2(){
    User user = new User("zero001", "男", new Date(), "杭州");
    int Row = session.insert("insertUser", user);
    session.commit();
    System.out.println("受影响行数："+Row);
    System.out.println("用户ID:"+user.getId());
}
```



#### 2.2.8 返回MySQL自增UUID

- 映射文件

```xml
<insert id="insertUser" parameterType="com.gyf.domain.User">
    <selectKey keyProperty="id" resultType="String" order="BEFORE">
        SELECT UUID()
    </selectKey>
    INSERT INTO USER (username,sex,birthday,address) 
    VALUES(#{username},#{sex},#{birthday},#{address})
</insert>
```



### 2.3 小结

- `parameterType`和`resultType`
  - `parameterType`：指定输入参数的java类型，可以填写别名或Java类的全限定名(就是全类名)。
  - `resultType`：指定输出结果的java类型，可以填写别名或Java类的全限定名。
- `#{}`和`${}`
  - `#{}`：相当于预处理中的占位符
      - `#{}`里面的参数表示接受java输入参数的名称
      - `#{}`可以接受HashMap、POJO类型的参数。当接受简单类型参数时，里面可以是vlaue，也可以是其他。
      - `#{}`可以防止SQL注入
  - `${}`：相当于拼接SQL串，对传入的值不做任何解释的原样输出。
      - 会引起SQL注入，所有需谨慎使用
      - 可以接受HashMap、POJO类型的参数。当接受简单类型的参数时，`${}`里面只能是value。
- `selectOne`和`selectList`
  - `selectOne`：只能查询0或1条记录，大于1条记录就会报错。
  - `selectList`：可以查询0条或N条记录。



### 2.4 MyBatis的Dao编写(了解即可)

- Dao层

```java
public class UserDaoImpl  implements UserDao {
    private SqlSessionFactory ssf;

    public UserDaoImpl() {
    }

    public UserDaoImpl(SqlSessionFactory ssf) {
        this.ssf = ssf;
    }

    @Override
    public void save(User user) {
        // 获取session
        SqlSession session = ssf.openSession();

        // 插入用户
        session.insert("insertUset",user);

        session.commit();
        session.close();
    }

    @Override
    public User findUserById(int id) {
        // 获取session
        SqlSession session = ssf.openSession();

        // 查询用户
        User user = session.selectOne("findUserById", id);

        session.commit();
        session.close();

        return user;
    }
}
```



- 测试

```java
public class UserDaoTest {
//    private SqlSession session = null;
    private SqlSessionFactory sessionFactory;

    @Before
    public void before() throws IOException {
        System.out.println("before:获取session");
        //1. 读取配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapconfig.xml");
        //2. 通过SqlSessionFactoryBuilder创建SqlSessionFactory会话工厂。
        sessionFactory = new SqlSessionFactoryBuilder().build(is);
    }


    @Test
    public void test1(){
        // 调用dao
        // 1. 创建dao
        UserDaoImpl userDao = new UserDaoImpl(sessionFactory);
        // 2. 查询
        User user = userDao.findUserById(1);
        System.out.println(user);
    }
}
```



### 2.5 MyBatis的Dao编写(mapper代理方式实现)

Mapper代理的开发方式，程序员只需编写mapper接口(相当于dao接口)即可。MyBatis会自动的为mapper接口生成动态代理实现类。



- 开发规范
  - mapper接口的全限定名要和mapper映射文件的namespace的值相同
  - mapper接口的方法名称要和映射文件中的statement的id相同
  - mapper的接口方法参数只能有一个，且类型要和mapper映射文件中statement的parameterType的值保持一致。
  - mapper接口的返回值类型要和mapper映射文件中statement的resultType值或resultMap中的type值保持一致。



- 编写示例文件

```java
// mapper类
public interface UserMapper {
    public void save(User user);
    public User findUserById(int id);
}
```

映射文件:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
	注意：使用mapper代理开发时，namespace值必须是接口全限定名
 -->
<mapper namespace="com.zero.mapper.UserMapper">

    <insert id="save" parameterType="com.zero.model.User">
        INSERT into user (username,sex,birthday,address)
        values (#{username},#{sex},#{birthday},#{address})
    </insert>


    <select id="findUserById" parameterType="int" resultType="com.zero.model.User">
		SELECT * FROM USER WHERE id = #{id}
	</select>

</mapper>
```

开发规范：

![mapper](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/21/mapper-1566398292193.jpg)


- 添加映射配置文件

```xml
<!--加载映射文件-->
<mappers>
    <mapper resource="com/zero/mapper/UserMapper.xml"/>
</mappers>
```



- 测试

```java
public class UserMapperTest {

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
        session.close();
    }

    @Test
    public void test1(){
        // 通过session获取接口代理类[默认代理：JDK实现的代理]
        UserMapper mapper = session.getMapper(UserMapper.class);
        // 查询
        User user = mapper.findUserById(1);
        System.out.println(user);
        mapper.save(new User("xxx", "x", new Date(), "xx"));
        session.commit();
    }
}
```



## 3. 全局配置文件其他配置

### 3.1 properties数据库文件配置

- 在src下配置`db.properties`文件

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql:///mybatis01?useUnicode=true&amp;characterEncoding=utf8
username=root
password=123456
```

- 在全局配置文件中加载并使用

```xml
<configuration>
    <!--1. 加载配置文件-->
    <properties resource="db.properties"/>

    <!-- 配置mybatis的环境信息 -->
    <environments default="development">
        <environment id="development">
            <!-- 配置JDBC事务控制，由mybatis进行管理 -->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 配置数据源，采用dbcp连接池 -->
            <dataSource type="POOLED">
                <!--2. 使用${}，可以引用一句加载的java配置文件中的信息-->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```



### 3.2 `typeAliases`

别名的使用是为了在映射文件中，更方便的去指定参数和结果集的类型，不再用写很长一段全限定名。

- mybatis支持的别名
  - java的全部基本数据类型及其对应包装类，引用数据类型等等。
- 自定义别名

```xml
<!--
在全局配置文件中
进行别名配置-->
<typeAliases>
    <!--单个定义别名-->
    <typeAlias type="com.zero.model.User" alias="user"/>
    <!-- 批量配置别名-->
    <!-- [name]：指定批量定义别名的类包，别名为类名，第一个字母小写(User=user)-->
    <package name="com.zero.model"/>
</typeAliases>
```



### 3.3 加载映射文件的几种方式

- 使用相对于类路径的资源。

```xml
<mappers>
    <mapper resource="com/zero/mapper/UserMapper.xml"/>
</mappers>
```

- 使用mapper接口的全限定名。注意：此方法要求mapper接口和mapper映射文件名称相同，且在同一目录下。

```xml
<mappers>
    <mapper class="com.zero.mapper.UserMapper"/>
</mappers>
```

​		如果没有映射文件，那么在接口中要声明注解。不推荐使用

```java
public interface UserMapper {

    @Insert("INSERT into user (username,sex,birthday,address) values (#{username},#{sex},#{birthday},#{address})")
    public void save(User user);
    @Select("SELECT * FROM USER WHERE id = #{id}")
    public User findUserById(int id);
}
```

- 注册指定包下的所有映射文件。推荐

```xml
<mappers>
<!--3. 注册指定包下的所有映射文件-->
    <package name="com.zero.mapper"/>
</mappers>
```



## 4. MyBatis的映射文件



### 4.1 输入映射`ParameterType`

指定输入参数的java类型，可以使用别名或类的全限定名。它可以接收简单类型、POJO对象、HashMap



#### 4.1.1 传递简单类型

```xml
<select id="findUserById" parameterType="int" resultType="com.zero.model.User">
    SELECT * FROM USER WHERE id = #{id}
</select>
```



#### 4.1.2 传递POJO对象

```xml
<!--插入用户信息-->
<insert id="insertUser" parameterType="user">
    <selectKey keyProperty="id" resultType="int" order="AFTER">
        SELECT LAST_INSERT_ID()
    </selectKey>
    INSERT into user (username,sex,birthday,address) values (#{username},#{sex},#{birthday},#{address})
</insert>
```



#### 4.1.3 传递POJO包装对象

开发中通过pojo传递查询条件 ，查询条件是综合的查询条件。不仅包括用户查询条件还包括其它的查询条件（比如将用户购买商品信息也作为查询条件），这时可以使用包装对象传递输入参数。

- 需求：综合查询用户信息，需传入复杂查询条件，如（用户信息、订单信息、商品信息）。

- 定义POJO包装类

```java
public class UserQueryVO {

    private User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}
```

- 修改映射接口

```java
public interface UserMapper {
    public void save(User user);

    public User findUserById(int id);
    public List<User> findUserByUserQueryVo(UserQueryVO vo);
}
```

- 修改映射文件，添加查询方法

```xml
<!--查找用户id为n的用户-->
<select id="findUserByUserQueryVo" parameterType="userQueryVO" resultType="user">
    select u.* from user u where u.id = #{user.id}
</select>
```

- 测试

```java
@Test
public void test1(){
    UserMapper mapper = session.getMapper(UserMapper.class);
    // 构造查询对象
    UserQueryVO query = new UserQueryVO();
    // 设置set
    User user = new User();
    user.setId(1);
    query.setUser(user);
    // 查询用户列表
    List<User> queryVo = mapper.findUserByUserQueryVo(query);
    System.out.println(queryVo);
}
```



#### 4.1.4 传递Map对象

- 修改映射接口文件,添加方法

```java
public interface UserMapper {

    public void save(User user);

    public User findUserById(int id);

    public List<User> findUserByUserQueryVo(UserQueryVO vo);

    public List<User> findUserByMap(Map<String,Object> map);
}
```

- 修改映射文件，添加查询语句

```xml
<select id="findUserByMap" parameterType="hashMap" resultType="user">
    select u.* from user u
    where username like '%${username}%' and sex=#{sex}
</select>
```

- 测试

```java
 @Test
public void  test2(){
    UserMapper mapper = session.getMapper(UserMapper.class);
    // 构建map
    Map<String, Object> map = new HashMap<>();
    map.put("username","张");
    map.put("sex","1");
    // 查询数据
    List<User> users = mapper.findUserByMap(map);
    System.out.println(users);
}
```



> 如果查询不到数据，原因是全局配置文件加载外部数据库属性文件会影响到映射文件中的数据读取，需在全局配置文件中将加载外部数据库属性文件关闭。



### 4.2 输出映射`resultType/resultMap`

#### 4.2.1 `resultType`

- 使用`resultType`进行结果映射时，查询的列名和映射的POJO属性名完全一致，该列才能映射成功。
- 如果查询的列名和映射的POJO属性名全部不一致，则不会创建POJO对象。
- 如果查询的列名和映射的POJO属性名有一个一致，就会创建POJO对象。



##### 4.2.1.1 输出简单类型

当输出结果只有一列时，可以使用`resultType`指定简单类型作为输出结果类型。

- 示例输出一个`count(*)`

- 映射接口文件中添加方法

```java
public int findUserCount(UserQueryVO vo);
```

- 映射文件中添加查询方法

```xml
<select id="findUserCount" parameterType="userQueryVO" resultType="int">
    select count(*) from user where sex=#{user.sex}
</select>
```

- 测试

```java
@Test
public void  test3(){
    UserMapper mapper = session.getMapper(UserMapper.class);
    UserQueryVO query = new UserQueryVO();
    User user = new User();
    user.setSex("1");
    query.setUser(user);
    int userCount = mapper.findUserCount(query);
    System.out.println(userCount);
}
```



##### 4.2.1.2 输出单个POJO对象

- 接口文件添加方法

```java
public User findUserById(int id);
```

- 映射文件添加查询方法

```xml
<select id="findUserById" parameterType="int" resultType="user">
    SELECT * FROM USER WHERE id = #{id}
</select>
```



##### 4.2.1.3 输出多个POJO对象

- 接口

```java
public List<User> findUserByUserQueryVo(UserQueryVO vo);
```

- 映射文件

```xml
<select id="findUserByUserQueryVo" parameterType="userQueryVO" resultType="user">
    select u.* from user u where u.id = #{user.id}
</select>
```



##### 4.2.1 小结

输出单个POJO对象和POJO列表时，mapper映射文件中的resultType的类型是一样的，mapper接口的方法返回值不同。

同样的mapper映射文件，返回单个对象和对象列表时，mapper接口在生成动态代理时，会根据返回值的类型，决定调用selectOne方法还是selectList方法。

> - POJO对象属性跟数据库列名一致，使用resultType。
> - 不一致，使用resultMap定义映射关系。



#### 4.2.2 `resultMap`

如果查询出来的列名和属性名不一致，通过定义一个resultMap将**列名**和POJO**属性名**之间作一个**映射关系**。

- mapper接口

```java
public User findUserByIdResultMap(int id);
```

- 映射文件

```xml
<!--设置返回类型为resultMap-->
<resultMap id="userResultMap" type="user">
    <!--自定义resultMap列名与属性之间的映射关系-->
    <id property="id" column="id_" />
    <result property="username" column="username_" />
    <result property="sex" column="sex_" />
    <result property="birthday" column="birthday_" />
    <result property="address" column="address_" />
</resultMap>
<select id="findUserByIdResultMap" parameterType="int" resultMap="userResultMap">
    select id id_, username username_, sex sex_,birthday birthday_, address address_
    from user where id=#{id};
</select>
```



## 5. 动态SQL

### 5.1 if和where

- if标签：作为判断入参来使用的，如果条件符合，则把if标签体内的SQL拼接上。

- where标签：会去掉条件中的第一个and符号。

- 动态SQL示例：

  - 修改mapper接口

  ```java
  public List<User> findUserList(UserQueryVO vo);
  ```

  - 修改映射文件

  ```xml
  <!--    动态拼接SQL-->
  <select id="findUserList" parameterType="userQueryVO" resultType="user">
      select * from user
      -- 条件判断
      <where>
          <if test="user != null">
              <if test="user.sex != null and user.sex != ''">
                  sex=#{user.sex}
              </if>
              <if test="user.username != null and user.username != ''">
                  and username like '%${user.username}%'
              </if>
          </if>
      </where>
  </select>
  ```

  - 测试

  ```java
  // 动态拼接SQL
  @Test
  public void  test5(){
      UserMapper mapper = session.getMapper(UserMapper.class);
      UserQueryVO query = new UserQueryVO();
      User user = new User();
      user.setSex("1");
      user.setUsername("张");
      query.setUser(user);
      List<User> list = mapper.findUserList(query);
      System.out.println(list);
  }
  ```



### 5.2 SQL片段

MyBatis提供了SQL片段功能，可提高SQL的可重用性

```xml
<!--声明一个SQL片段-->
<sql id="select_user_where">
    <if test="user != null">
        <if test="user.sex != null and user.sex != ''">
            sex=#{user.sex}
        </if>
        <if test="user.username != null and user.username != ''">
            and username like '%${user.username}%'
        </if>
        <if test="user.address != null and user.address != ''">
            and address like '%${user.address}%'
        </if>
    </if>
</sql>

<!--动态拼接SQL-->
<select id="findUserList" parameterType="userQueryVO" resultType="user">
    select * from user
    -- 条件判断
    <where>
        -- 引用SQL片段
        <include refid="select_user_where"/>
    </where>
</select>
```



### 5.3 forEach遍历

- 查询对象类中新增ids属性，存放id列表

```java
public class UserQueryVO {

    private User user;

    private List<Integer> ids;

    public List<Integer> getIds() {
        return ids;
    }

    public void setIds(List<Integer> ids) {
        this.ids = ids;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}
```

- mapper接口

```java
/**
     * 查询多个id的用户数据
     */
public List<User> findUserByIds(UserQueryVO vo);
```

- 映射文件

```xml
<!--foreach示例-->
<select id="findUserByIds" parameterType="userQueryVO" resultType="user">
    select * from user
    -- select * from user where id in (10,24,37);
    <where>
        <if test="ids != null and ids.size > 0">
            <!--
                    foreach标签：表示一个foreach循环
                    collection：集合参数的名称，如果是直接传入集合参数，则该处参数名称只能填写[list]
                    item：每次遍历出来的对象
                    open：开始遍历时拼接的串
                    close：结束遍历时拼接的串
                    separator：遍历出的每个对象之间需要拼接的字符
                -->
            <foreach collection="ids" item="id" open="id in(" close=")" separator=",">
                ${id}
            </foreach>
        </if>
    </where>
</select>
```

- 测试

```java
@Test
public void  test6() {
    UserMapper mapper = session.getMapper(UserMapper.class);
    UserQueryVO query = new UserQueryVO();
    List<Integer> ids = new ArrayList<>();
    //select * from user where id in (10,24,37);
    ids.add(10);
    ids.add(24);
    ids.add(37);
    query.setIds(ids);
    List<User> users = mapper.findUserByIds(query);
    System.out.println(users);
}
```



- 关于参数直接传入id集合

  - mapper接口

  ```java
  public List<User> findUserByLists(List<Integer> list);
  ```

  - 映射文件

  ```xml
  <!--如果参数是数组的话，parameterType可以写全限定名，也可以写别名-->
  <select id="findUserByLists" parameterType="java.util.List" resultType="user">
      select * from user
      <where>
          <if test="list != null and list.size > 0">
              -- 如果是直接传入集合参数，则该处参数名称只能填写[list]
              <foreach collection="list" item="id" open="id in(" close=")" separator=",">
                  ${id}
              </foreach>
          </if>
      </where>
  </select>
  ```

  - 测试同上，查询方法改一下即可。



## 6. MyBatis与Hibernate的区别

### 6.1 MyBatis技术特点

- 好处：
  - 通过直接编写SQL语句，可以直接对SQL进行性能优化
  - 学习成本低，有SQL基础，就可以学习MyBatis，容易上手
  - 由于直接编写SQL语句，所以灵活多变，代码维护性更好
- 缺点
  - 不能支持数据库无关性，即数据库发生变更，要写多套代码进行支持，移植性不好。
  - 需要编写结果映射。

### 6.2 Hibernate技术特点

- 好处
  - 标准的ORM框架，程序员无需编写SQL语句。
  - 具有良好的数据库无关性，即数据库发生变化，代码无需再次编写。
    - 例：mysql数据迁移到oracle，只需更改方言配置即可
- 缺点
  - 学习门槛高，需要对数据关系模型有良好的基础，而且在设置OR映射时，需考虑好性能和对象模型的权衡
  - 我们不能自主的去进行SQL性能优化。

### 6.3 MyBatis应用场景

需求多变的互联网项目，例如电商项目

### 6.4 Hibernate应用场景

需求明确，业务固定的项目，例如OA项目，ERP项目等。