---
title: SSM整合Demo
date: 2019-9-5 23:00
categories: SpringMVC
tags: [SpringMVC]
description: SSM整合小项目，主要学习如何配置整合SSM
---



## 1. SSM整合项目

### 1.1 搭建环境

<!--more-->



- SSM整合可以使用多种方式，这里选择XML+注解的方式
- 整合思路
  - 先搭建整合环境
  - 把Spring的配置搭建完成
  - 使用Spring整合SpringMVC框架
  - 最后使用Spring整合MyBatis框架



### 1.2 基本环境

- 创建数据库表和表结构

```sql
create database ssm01;
use ssm01;
create table account(
	id int primary key auto_increment,
	name varchar(100),
	money double
);
```



- 创建一个Maven工程，在pom.xml文件中引入坐标依赖

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <!--设置版本锁定-->
    <spring.version>5.0.2.RELEASE</spring.version>
    <slf4j.version>1.6.6</slf4j.version>
    <log4j.version>1.2.12</log4j.version>
    <mysql.version>5.1.6</mysql.version>
    <mybatis.version>3.4.5</mybatis.version>
</properties>

<dependencies>
    <!-- spring -->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.6.8</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.0</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>jstl</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>
    <!-- log start -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>${log4j.version}</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${slf4j.version}</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>${slf4j.version}</version>
    </dependency>
    <!-- log end -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>${mybatis.version}</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>1.3.0</version>
    </dependency>
    <dependency>
        <groupId>c3p0</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.1.2</version>
        <type>jar</type>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

- 创建基础环境，具体如下图所示：


![ssm01](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/09/06/ssm01-1567700025391.jpg)


- AccountDao代码

```java
/**
 * 账户dao接口
 */
public interface AccountDao {

    /**
     * 查询所有用户
     * @return
     */
    public List<Account> findAll();

    /**
     * 保存账户
     * @param account
     */
    public void SaveAccount(Account account);
}
```

- Account实体类代码

```java
/**
 * 账户类
 */
public class Account implements Serializable {
    private Integer id;
    private String name;
    private Double money;
}
```

- 业务层代码

```java
public class AccountServiceImpl implements AccountService {

    @Override
    public List<Account> findAll() {
        System.out.println("业务层：查询所有账户....");
        return null;
    }

    @Override
    public void SaveAccount(Account account) {
        System.out.println("业务层：保存账户....");
    }
}
```



## 2. Spring框架代码编写



### 2.1 搭建和测试Spring开发环境

- 在resources项目中创建applicationContext.xml配置文件，编写具体的配置信息。

```xml
<?xml version="1.0" encoding="UTF-8"?>
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

    <!-- 开启注解扫描，要扫描的是service和dao层的注解
        要忽略web层注解，因为web层让SpringMVC框架去管理 -->
    <context:component-scan base-package="com.zero">
        <!-- 配置要忽略的注解 -->
        <context:exclude-filter type="annotation"
                             expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
</beans>
```

- 编写一个测试方法进行测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class TestSpring {

    @Resource(name = "accountService")
    private AccountService accountService;

    @Test
    public void test1(){
        accountService.findAll();
        accountService.SaveAccount(new Account());
    }
}
```



## 3. Spring整合SpringMVC框架

### 3.1 搭建和测试SpringMVC的开发环境

- 在web.xml中配置DispatcherServlet前端控制器，中文乱码过滤器

```xml
<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <!--配置DispatcherServlet前端控制器-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--配置初始化参数，创建完dispatcherServlet对象后加载springMVC.xml配置文件-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springMVC.xml</param-value>
    </init-param>
    <!--服务器启动时，就让dispatcherServlet对象创建-->
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

  <!-- 配置解决中文乱码的过滤器 -->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

</web-app>
```

- 创建springMVC.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!--扫描controller的注解，别的不用扫描-->
    <context:component-scan base-package="com.zero.controller">
        <!-- 配置包含的注解 -->
        <context:include-filter type="annotation"
                                expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!--配置视图解析器-->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--配置前后缀-->
        <property name="prefix" value="/WEB-INF/pages/" />
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--设置静态资源不过滤-->
    <mvc:resources mapping="/css/**" location="/css/"/>
    <mvc:resources mapping="/images/**" location="/images/"/>
    <mvc:resources mapping="/js/**" location="/js/"/>

    <!--开启对SpringMVC的注解支持-->
    <mvc:annotation-driven/>

</beans>
```

- 控制器代码

```java
@Controller
@RequestMapping("account")
public class AccountController {

    @RequestMapping("/findAll")
    public String findAll(){
        System.out.println("表现层：查询所有用户....");
        return "list";
    }
}
```

- jsp：`<a href="/account/findAll">查询所有用户</a>`





### 3.2 Spring整合SpringMVC的框架

- 目的：能在controller中成功调用service对象中的方法，查询数据。
- 在项目启动时，就去加载applicationContext.xml的配置文件，在web.xml中配置`ContextLoaderListener`监听器（该监听器默认只能加载WEB-INF目录下的applicationContext.xml的配置文件）。

```xml
<!--配置Spring提供的监听器，用于启动服务器时加载容器-->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<!--手动指定springMVC配置文件位置-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
```

- 修改控制器代码，注入service属性，调用方法

```java
 @Controller
@RequestMapping("account")
public class AccountController {

    // 注入service对象
    @Autowired
    private AccountService accountService;

    @RequestMapping("/findAll")
    public String findAll(){
        System.out.println("表现层：查询所有用户....");
        // 调用service的方法
        accountService.findAll();
        return "list";
    }
}
```

- 测试结果

```
表现层：查询所有用户....
业务层：查询所有账户....
```



## 4. Spring整合MyBatis框架

### 4.1 搭建和测试MyBatis的环境

- 在resources资源文件夹下创建SqlMapConfig.xml的配置文件，编写核心配置文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--MyBatis核心配置文件-->
<configuration>
    <!--配置数据库环境-->
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///ssm01"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 引入映射配置文件，这里使用的是注解方式 -->
    <mappers>
        <!--<mapper class="com.zero.dao.AccountDao"/>-->
        <!-- 该包下所有的dao接口都可以使用 -->
        <package name="com.zero.dao"/>
    </mappers>
</configuration>
```

- 使用注解方式进行SQL语句编写

```java
/**
 * 账户dao接口
 */
@Resource
public interface AccountDao {

    /**
     * 查询所有用户
     * @return
     */
    @Select("select * from account")
    public List<Account> findAll();

    /**
     * 保存账户
     * @param account
     */
    @Insert("insert into account (name,money) values(#{name},#{money})")
    public void SaveAccount(Account account);
}
```

- 编写测试方法

```java
public class TestMyBatis {

    /**
     * 测试查询
     * @throws Exception
     */
    @Test
    public void test1() throws Exception {
        // 加载配置文件
        InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
        // 创建工厂
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
        // 创建sqlSession对象
        SqlSession session = factory.openSession();
        // 获取代理对象
        AccountDao dao = session.getMapper(AccountDao.class);
        List<Account> list = dao.findAll();
        for (Account account:list){
            System.out.println(account);
        }
        // 释放资源
        session.close();
        inputStream.close();
    }

    /**
     * 测试保存
     * @throws Exception
     */
    @Test
    public void test2() throws Exception {
        Account account = new Account();
        account.setName("小黑");
        account.setMoney(400d);
        // 加载配置文件
        InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
        // 创建工厂
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
        // 创建sqlSession对象
        SqlSession session = factory.openSession();
        // 获取代理对象
        AccountDao dao = session.getMapper(AccountDao.class);
        dao.SaveAccount(account);
        // 提交事务
        session.commit();
        // 释放资源
        session.close();
        inputStream.close();
    }
}
```



### 4.2 Spring整合MyBatis框架

- 目的：把SqlMapConfig.xml配置文件配置到Spring的配置文件中。

#### 4.2.1 让Spring接管MyBatis的Session工厂

```xml
<!--spring整合MyBatis框架-->
<!--配置c3p0连接池-->
<bean id="DataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql:///ssm01"/>
    <property name="user" value="root"/>
    <property name="password" value="123456"/>
</bean>
<!--配置SqlSessionFactory工厂-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="DataSource"/>
</bean>
```

#### 4.2.2 配置自动扫描所有Mapper接口和文件

```xml
<!--配置AccountDao接口所在包-->
<!-- 引入映射配置文件-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.zero.dao"/>
</bean>
```



#### 4.2.4 测试

- 在AccountDao接口上添加注解

```java
/**
 * 账户dao接口
 */
@Repository
public interface AccountDao {
}
```

- 在service层调用Dao

```java
@Service("accountService") // 将service交由IOC容器管理
public class AccountServiceImpl implements AccountService {

    // 注入Dao接口
    @Autowired
    private AccountDao accountDao;

    @Override
    public List<Account> findAll() {
        System.out.println("业务层：查询所有账户....");
        // 调用dao查询
        return accountDao.findAll();
    }

    @Override
    public void SaveAccount(Account account) {
        System.out.println("业务层：保存账户....");
        accountDao.SaveAccount(account);
    }
}
```

- 控制器代码

```java
@Controller
@RequestMapping("account")
public class AccountController {

    // 注入service对象
    @Autowired
    private AccountService accountService;

    @RequestMapping("/findAll")
    public String findAll(Model model){
        System.out.println("表现层：查询所有用户....");
        // 调用service的方法
        List<Account> list = accountService.findAll();
        model.addAttribute("list",list); // 存入request域
        return "list";
    }
}
```

- `list.jsp`页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>用户列表</title>
</head>
<body>
<h3>用户列表</h3>
<c:forEach items="${list}" var="account">
    用户姓名：${account.name}<br>
    余额：${account.money}<br>
</c:forEach>
</body>
</html>
```



#### 4.2.5 配置Spring的声明式事务管理

- 在`applicationContext.xml`中配置事务

```xml
<!--3. 配置Spring框架声明式事务管理-->
<!--3.1 配置事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="DataSource"/>
</bean>
<!--3.2 配置事务通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="find*" read-only="true"/>
        <tx:method name="*" isolation="DEFAULT"/>
    </tx:attributes>
</tx:advice>
<!--3.3 配置AOP增强-->
<aop:config>
    <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.zero.service.Impl.*ServiceImpl.*(..))"/>
</aop:config>
```

- 控制器新增保存用户方法

```java
/**
     * 保存用户
     * @param account
     * @return
     */
@RequestMapping("/save")
public String save(Account account){
    System.out.println("表现层：保存用户....");
    accountService.SaveAccount(account);
    return "redirect:findAll";
}
```

- `index.jsp`中添加代码，提交用户表单

```jsp
<h3>保存用户</h3>
<form action="/account/save" method="post">
    用户名：<input type="text" name="name"><br>
    金额：<input type="text" name="money"><br>
    <input type="submit" value="保存">
</form>
```



