---
title: Spring简单入门
date: 2019-8-5 21:10
categories: FrameWork
tags: [Spring]
description: Spring基础入门
urlname: Spring-Basic
---



## 1. Spring介绍

Spring是一个开源框架，于2003年兴起的轻量级Java开发框架。Spring为简化企业级应用开发而生，使用Spring可以使简单的JavaBean实现以前只有EJB才能实现的功能。

> 简单来说：Spring是一个轻量级的控制反转(IOC)和面向切面(AOP)的容器框架。



<!--more-->



### 1.1 Spring的好处

- 方便解耦，简化开发
  - Spring就是一个大工厂，专门负责生产Bean，可以将所有对象创建和依赖关系维护，交给Spring管理。
- AOP编程的支持
  - Spring提供面向切面编程，可以方便的实现对程序进行**权限拦截**、运行监控等功能。
- 声明式事务的支持
  - 只需要通过配置就可以完成对事务的管理，而无需手动编程。
- 方便程序的测试
  - Spring对Junit4支持，可以通过注解方便的测试Spring程序。
- 方便集成各种优秀框架
  - Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架(如：Struts、Hibernate、MyBatis等)的直接支持。
- 降低JavaEE API的使用难度
  - Spring对JavaEE开发中非常难用的一些API(JDBC、JavaMail、远程调用等)，都提供了封装，使这些API应用难度大大降低。

### 1.2 Spring体系结构

Spring框架是一个分层架构，它包含一系列的功能要素并被分为大约**20个模块**。这些模块分为Core Container、Data Access/Integration、Web、AOP（Aspect Oriented Programming）、Instrumentation和测试部分。如下图所示：



![Spring](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/05/Spring-1565010640149.jpg)




## 2. Spring简单入门

基本流程：

1. 下载Spring开发包
2. 导入Spring核心jar包
3. 编写Spring核心配置文件
4. 在程序中读取Spring配置文件，通过Spring框架获取Bean，完成相应操作。



### 2.1 下载Spring开发包

Spring[官方下载地址](https://repo.spring.io/libs-release-local/org/springframework/spring/)

下载后解压，目录结构如下：



![SpringDir](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/05/SpringDir-1565010658795.jpg)


### 2.2 导入Spring核心jar包到项目中

![spring_jar](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/05/spring_jar-1565010729837.jpg)


- `spring-core-3.2.0.RELEASE.jar`
  - 包含Spring框架基本的核心工具类，Spring其他组件都要使用到这个包里的类，是其他组件的基本核心。
- `spring-beans-3.2.0.RELEASE.jar`
  - 所有应用都要用到的，它包含访问配置文件、创建和管理bean，以及进行IOC/DI操作相关的所有类
- `spring-context-3.2.0.RELEASE.jar`
  - Spring提供在基础IOC功能上的扩展功能，此外还提供许多企业级服务的支持，如邮件服务、任务调度、JNDI定位、EJB集成、远程访问、缓存以及各种视图层框架的封装等。
- `spring-expression-3.2.0.RELEASE.jar`
  - Spring表达式语言
- `commons-logging-1.2.jar`
  - 第三方的主要用于处理日志



> 注意导入时，不要导入带source的源文件。

### 2.3 编写Spring核心配置文件

#### 2.3.1 创建测试方法

- 在src下创建一个service文件夹，里面创建UserService及其对应实现类，实现add方法，直接打印一句话。

```java
public class UserServiceImpl implements UserService{
    @Override
    public void add() {
        System.out.println("创建用户...");
    }
}
```

- 创建测试方法，调用add方法。

```java

public class UserServiceTest {

    @Test
    public void test1(){
        UserService us = new UserServiceImpl();
        us.add();
    }
}
```



#### 2.3.2 使用`Spring IOC`控制反转创建实例

- 编写配置文件`beans.xml`，xsd约束文件可以在`/spring-framework-3.2.0.RC2-docs/reference/html/xsd-config.html`中查看

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 配置一个bean -->
    <bean id="userService" class="com.zero.service.UserServiceImpl"></bean>
</beans>
```



### 2.4 在程序中读取Spring配置文件，通过Spring框架获取Bean，完成相应操作。



```java
public class UserServiceTest {

    @Test
    public void test1(){
        // 不使用spring的方式，自己创建对象
        //        UserService us = new UserServiceImpl();
        //        us.add();

        // 使用spring容器方式获取UserService
        // 1. 加载beans.xml 这个spring的配置文件，内部就会创建对象
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
		// 2. 获取对象
        UserService userService1 = (UserService) context.getBean("userService");
        userService1.add();
        System.out.println(userService1); // com.zero.service.UserServiceImpl@5f058f00
        UserService userService2 = (UserService) context.getBean("userService");
        System.out.println(userService2); // com.zero.service.UserServiceImpl@5f058f00
    }
}

```

### 2.5 IOC

IOC(Inverse of Control) 反转控制的概念，就是将原本在程序中手动创建对象的控制权，交由Spring框架管理。简单的说，就是创建对象控制权被反转到了Spring框架。



### 2.6 DI解释

- Dependency Injection 依赖输入，在Spring框架负责创建Bean对象时，动态的将依赖对象注入到Bean组件。



例：在UserServiceImpl中提供一个get/set的name方法，在beans.xml中提供property去注入。

```java
public class UserServiceImpl implements UserService{

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public void add() {
        System.out.println("创建用户..."+name);
    }
}
```

配置文件中使用依赖注入：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 配置一个bean -->
    <bean id="userService" class="com.zero.service.UserServiceImpl">
        <!-- 使用DI依赖注入数据, 调用属性的set方法-->
        <property name="name" value="zhangsan"></property>
    </bean>
</beans>
```

