---
title: JDBC_Template
date: 2019-8-11 23:28
categories: FrameWork
tags: [Spring]
description: 使用Spring组件JDBC Template简化持久化操作
---



## 1. JDBC Template

Spring对数据库的操作在jdbc上面做了深层次的封装，简化了持久层操作。使用spring的注入功能，可以把DataSource注册到JdbcTemplate中。



<!--more-->

 

- 为了简化持久化操作，Spring在JDBC API之上提供了JDBC Template组件





### 1.1 准备阶段

- 先创建数据库

```sql
drop database if exists selection_course;

create database selection_course;
use selection_course;

create table course
(
   id                   int not null auto_increment,
   name                 char(20),
   score                int,
   primary key (id)
);

create table selection
(
   student              int not null,
   course               int not null,
   selection_time       datetime,
   score                int,
   primary key (student, course)
);

create table student
(
   id                   int not null auto_increment,
   name                 varchar(20),
   sex                  char(2),
   born                 date,
   primary key (id)
);

alter table selection add constraint FK_Reference_1 foreign key (course)
      references course (id) on delete restrict on update restrict;

alter table selection add constraint FK_Reference_2 foreign key (student)
      references student (id) on delete restrict on update restrict;



-- 修改表字符集
alter table course default character set utf8 collate utf8_general_ci;

insert into course(id,name,score) values(1001,'英语',5);
insert into course(id,name,score) values(1002,'操作系统',5);
insert into course(id,name,score) values(1003,'数据结构',3);

commit;
```

> 如果出现1366问题，是表字符集编码问题，需修改表默认字符集为utf8。

- 创建Maven项目，导入相关jar包
  - MySQL驱动
  - Spring组件（core、beans、context、aop）
  - JDBC Template（jdbc、tx）

```xml
<!--导入数据库驱动包-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.32</version>
</dependency>
<!--导入Template相关包-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>${spring.version}</version>
</dependency>
```

- 创建`spring.xml`文件，配置如下内容

  - 数据源
  - JDBC Template

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xmlns:tx="http://www.springframework.org/schema/tx"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans.xsd
      http://www.springframework.org/schema/context
      http://www.springframework.org/schema/context/spring-context.xsd
      http://www.springframework.org/schema/aop
      http://www.springframework.org/schema/aop/spring-aop.xsd
      http://www.springframework.org/schema/tx
      http://www.springframework.org/schema/tx/spring-tx.xsd">
      <!--配置数据源-->
      <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
          <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
          <property name="url" value="jdbc:mysql://localhost:3306/selection_cource?useUnicode=true&amp;characterEncoding=utf-8"/>
          <property name="username" value="root"/>
          <property name="password" value="123456"/>
      </bean>
      <!--配置Template-->
      <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
          <property name="dataSource" ref="dataSource"/>
      </bean>
  </beans>
  ```



### 1.2 JDBC Template基本使用

#### 1.2.1 execute方法

```java
public void testExecute(){
    jdbcTemplate.execute("create table user1(id int,name varchar(20))");
}
```

#### 1.2.2 update与batchUpdate方法

- update方法：对数据进行增删改操作

```java
int update(String sql, Object[] agrs)
int update(String slq, Object... args)

    
// 方法示例
// int update(String sql, Object[] agrs)
@Test
public void testUpdate(){
    String sql = "insert into student(name,sex) values(?,?)";
    jdbcTemplate.update(sql,new Object[]{"小明","男"});
}

// int update(String slq, Object... args)
@Test
public void testUpdate2(){
    String sql = "update student set sex=? where name=?";
    jdbcTemplate.update(sql,"女","小明");
}
```

- batchUpdate方法：批量增删改操作

```java 
int[] batchUpdate(String[] sql)
int[] barchUpdate(String sql, List<Object[]> args)

// 方法示例
// int[] batchUpdate(String[] sql)
@Test
public void testBatchUpdate(){
    String[] sqls = {
        "insert into student(name,sex) values('小李','女')",
        "insert into student(name,sex) values('小花','女')",
        "update student set sex='男' where name = '小李'"
    };
    jdbcTemplate.batchUpdate(sqls);
}

// int[] barchUpdate(String sql, List<Object[]> args),适用于同步sql语句执行,使用率较高。
@Test
public void testBatchUpdate2(){
    String sql = "insert into selection(student,course) values(?,?)";
    List<Object[]> list = new ArrayList<Object[]>();
    list.add(new Object[]{2,1002});
    list.add(new Object[]{2,1003});
    jdbcTemplate.batchUpdate(sql,list);
}
```



#### 1.2.3 query与queryXXX方法

- 查询简单数据项

  - 获取一个

  ```java
  T queryForObject(String sql, Class<T> type)
  T queryForObject(String sql, Object[] args, Class<T> type)
  T queryForObject(String sql, Class<T> type,Object... arg)
      
  // 方法示例
  // T queryForObject(String sql, Class<T> type)
  @Test
  public void testQueryForObject(){
      String sql = "select count(*) from student";
      int count = jdbcTemplate.queryForObject(sql, Integer.class); // 第一个参数sql语句，第二个参数返回值类型
      System.out.println(count);
  }
  ```

  - 获取多个

  ```java
  T queryForList(String sql, Class<T> type)
  T queryForList(String sql, Object[] args, Class<T> type)
  T queryForList(String sql, Class<T> type,Object... arg)
      
  // 方法示例
  // T queryForList(String sql, Class<T> type)
  @Test
  public void testQueryForList(){
      String sql = "select name from student where sex=?";
      List<String> names = jdbcTemplate.queryForList(sql, String.class, "女");
      System.out.println(names);
  }
  ```

- 查询复杂对象（封装为Map）

  - 获取一个

  ```java
  Map queryForMap(String sql)
  Map queryForMap(String sql,Object[] args)
  Map queryForMap(String sql,Object... arg)
      
  // 方法示例
  // Map queryForMap(String sql)
  @Test
  public void testQueryForMap(){
      String sql = "select * from student where id = ?";
      Map<String, Object> map = jdbcTemplate.queryForMap(sql, 1);
      System.out.println(map); // {id=1, name=小明, sex=女, born=null}
  }
  ```

  - 获取多个

  ```java
  List<Map<String,Object>> queryForList(String sql)
  List<Map<String,Object>> queryForList(String sql,Object[] args)
  List<Map<String,Object>> queryForList(String sql,Object... arg)
      
  // 方法示例
  // List<Map<String,Object>> queryForList(String sql)
  @Test
  public void testQueryList(){
      String sql = "select * from student";
      List<Map<String, Object>> stus = jdbcTemplate.queryForList(sql);
      System.out.println(stus); // [{id=1, name=小明, sex=女, born=null}, {id=2, name=小李, sex=男, born=null}, {id=3, name=小花, sex=女, born=null}]
  }
  ```

- 查询复杂对象（封装为实体对象）

  - RowMapper接口

  - 获取一个

  ```java
  T queryForObject(String sql,RowMapper<T> mapper)
  T queryForObject(String sql,Object[] args,RowMapper<T> mapper)
  T queryForObject(String sql,RowMapper<T> mapper,Object... arg)
      
  // 方法示例
  @Test
  public void testQueryEntity1(){
      String sql = "select * from student where id = ?";
      Student student = jdbcTemplate.queryForObject(sql, new RowMapper<Student>() {
          // 设置映射关系
          public Student mapRow(ResultSet rs, int rowNum) throws SQLException {
              Student student = new Student();
              student.setId(rs.getInt("id"));
              student.setName(rs.getString("name"));
              student.setSex(rs.getString("sex"));
              student.setBorn(rs.getDate("born"));
              return student;
          }
      },1);
      System.out.println(student); // Student{id=1, name='小明', sex='女', born=null}
      
      //        Student student = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Student>(Student.class), 1);
      //        System.out.println(student);
  }
  ```

  - 获取多个

  ```java
  List<T> query(String sql,RowMapper<T> mapper)
  List<T> query(String sql,Object[] args,RowMapper<T> mapper)
  List<T> query(String sql,RowMapper<T> mapper,Object... arg)
      
  // 方法示例
  // List<T> query(String sql,RowMapper<T> mapper) 获取多条记录，封装成实体类
  @Test
  public void testQueryEntity2(){
      String sql = "select * from student";
      List<Student> stus = jdbcTemplate.query(sql, new RowMapper<Student>() {
          // 设置映射关系
          public Student mapRow(ResultSet rs, int rowNum) throws SQLException {
              Student student = new Student();
              student.setId(rs.getInt("id"));
              student.setName(rs.getString("name"));
              student.setSex(rs.getString("sex"));
              student.setBorn(rs.getDate("born"));
              return student;
          }
      });
      System.out.println(stus);
  }
  ```



### 1.3 优缺点分析

- 优点：简单灵活
- 缺点：
  - SQL与Java代码掺杂
  - 功能不丰富


## 2. 连接池技术

- JDBC(Java DataBase Connecttivity)，java数据库连接。是一种用于执行SQL语句的Java API
- ODBC(Open DataBase Connectivity)，开发数据库连接。是微软公司提供的一组对数据库访问的标准API(应用程序编程接口)
- DBCP(DataBase Connection Pool)数据库连接池，是Java数据库连接池的一种，由Apache开发。
- C3P0是一个开源的JDBC连接池，它实现了数据源和JNDI绑定，支持JDBC3规范和JDBC2的标准扩展。目前使用它的开源项目有Hibernate、Spring等。
- 提问：c3p0和dbcp的区别？
  - dbcp没有自动回收空闲连接的功能；c3p0有自动回收空闲连接的功能。
  - 对数据连接的处理方式不同：C3P0提供最大空闲时间，DBCP提供最大连接数。C3P0当连接超过最大空闲连接时间时，当前连接就会被断掉。DBCP当前连接超过最大连接数时，所有连接都会被断开。



### 2.1 配置DBCP

导入`commons-dbcp2.jar`、`commons-pool.jar`到工程。在spring配置文件中配置如下。

```xml
<!--配置DBCP数据源-->
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/selection_course?useUnicode=true&amp;characterEncoding=utf-8"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</bean>
<!--配置Template-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>

<!--依赖注入，注意一定要提供set方法-->
<bean id="studentDao" class="com.zero.jdbc_Template.dao.Impl.StudentDaoImpl">
    <property name="jdbcTemplate" ref="jdbcTemplate"/>
</bean>
```

### 2.2 配置C3P0

导入`c3p0-0.9.2.1.jar`到工程。在spring配置文件中配置如下。

~~~xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/selection_course?useUnicode=true&amp;characterEncoding=utf-8"/>
    <property name="user" value="root"/>
    <property name="password" value="123456"/>
</bean>
~~~



### 2.3 关于`JdbcDaoSupport`

JdbcDaoSupport是spring框架为我们提供的一个类，该类中定义了一个JdbcTemplate对象，我们可以直接获取使用，但是要想创建该对象，需要为其提供一个数据源。

[参考文章](https://blog.csdn.net/weixin_42112635/article/details/88020509)



### 2.4 关于引用外部属性文件

- 将数据库连接的信息配置到属性文件中：

```properties
username=root
password=123456
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://127.0.0.1:3306/selection_course?useUnicode=true&amp;characterEncoding=utf-8
```

- 在spring配置文件中引入外部的属性文件

```xml
<!-- 引入外部属性文件： -->
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="location" value="classpath:jdbc.properties"/>
</bean>

<!--方法2-->
<context:property-placeholder location="classpath:jdbc.properties"/>
```







## 总结

- 持久化操作特点
- ORM：对象关系映射
- JDBC Template是Spring框架对JDBC操作的封装，简单、灵活但不够强大
- 实际应用中还需和其他ORM框架混合使用。