---
title: AspectJ的AOP开发
date: 2019-8-10 23:28
categories: FrameWork
tags: [Spring]
description: 主要学习Spring基于AspectJ的AOP开发
urlname: Spring-AspectJ
---



## 1. AspectJ 简介

AspectJ是一个面向切面的框架，它扩展了Java语言。AspectJ定义了AOP语法，它有一个专门的编译器用来生成遵守Java字节编码规范的Class文件。



<!--more-->



- AspectJ是一个基于Java语言的AOP框架
- Spring2.0以后新增了对AspectJ切点表达式的支持
- @AspectJ是AspectJ1.5新增功能，通过JDK5注解技术，允许直接在Bean类中定义切面
- 新版本Spring框架，建议使用AspectJ方式来开发AOP
- 使用AspectJ需要导入Spring AOP和AspectJ相关jar包
  - spring-aop-4.2.4.RELEASE.jar
  - com.springsource.org.aopalliance-1.0.0.jar
  - spring.aspects-4.2.4.RELEASE.jar
  - com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar



## 2. 注解方式实现AOP

### 2.1 注解开发：环境准备

xml配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop" xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--开启AspectJ自动代理-->
    <aop:aspectj-autoproxy/>
</beans>
```



### 2.2 `@AspectJ`提供不同的通知类型

- `@Before`：前置通知，相当于`BeforeAdvice`
- `@AfterReturning`：后置通知，相当于`AfterReturningAdvice`
- `@Around`：环绕通知，相当于`MethodInterceptor`
- `@AfterThrowing`：异常抛出通知，相当于`ThrowAdvice`
- `@After`：最终final通知，不管是否异常，该通知都会执行
- `@DeclareParents`：引介通知，相当于`IntroductionInterceptor`（了解即可）



### 2.3 在通知中通过value属性定义切点

- 通过`execution`函数，可以定义切点的方法切入
- 语法：`execution(<访问修饰符>?<返回类型><方法名>(<参数>)<异常>)`
- 示例：
  - 匹配所有类public方法：`execution(public * *(..))`
  - 匹配指定包下所有类方法：`execution(* com.zero.dao.*(..))`，不包含子包
  - `execution(* com.zero.dao..*(..))`：`..*`表示包含子包
  - 匹配指定类所有方法：`execution(* com.zero.service.UserService.*(..))`
  - 匹配实现特定接口所有类方法：`execution(* com.zero.dao.GenericDao+.*(..))`
  - 匹配所有save开头的方法：`execution(* save*(..))`



### 2.4 为目标类、定义切面类，实现AOP



#### 2.4.1 使用前置通知`@Before`



- 首先定义目标类`ProductDao`

```java
public class ProductDao {
    public void save(){ System.out.println("保存商品。。。"); }
    public void delete(){
        System.out.println("删除商品。。。");
    }
    public void update(){
        System.out.println("更新商品。。。");
    }
    public void findOne(){
        System.out.println("查询一个商品。。。");
    }
    public void findAll(){
        System.out.println("查询所有商品。。。");
    }
}
```

- 定义切面类

```java
/**
 * 切面类，注解方式
 */

@Aspect
public class MyAspectAnno {

    @Before(value = "execution( * com.zero.aspectj.demo1.ProductDao.*(..))")  // 匹配任意返回值类型的，具体目标类(com.zero.aspectj.demo1.ProductDao)下的所有任意参数的方法
    public void before(JoinPoint joinPoint){
        System.out.println("前置通知>>>>>>>>>"+joinPoint); // 可以在方法中传入JoinPoint对象，用来获得切点信息，格式为：前置通知>>>>>>>>>execution(void com.zero.aspectj.demo1.ProductDao.save())
    }
}
```

- 配置xml文件

```xml
<!--开启AspectJ自动代理-->
<aop:aspectj-autoproxy/>

<!--目标类-->
<bean id="productDao" class="com.zero.aspectj.demo1.ProductDao"/>
<!--切面类-->
<bean class="com.zero.aspectj.demo1.MyAspectAnno"/>
```

- 测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class Demo1Test {

    @Resource(name = "productDao") // 注入目标类
    private ProductDao productDao;

    @Test
    public void test1(){
        productDao.save();
        productDao.delete();
        productDao.update();
        productDao.findAll();
        productDao.findOne();

    }
}
```



> 目标类定义方法时，注意要写上访问权限修饰符`public`，不然定义`execution( * com.zero.aspectj.demo1.ProductDao.*(..))`时会匹配不到方法



#### 2.4.2 使用后置通知`@AfterReturning`

- 只需要在切面类中定义后置通知方法即可，如果目标方法有返回值，可以通过设置`returning`属性，来获取返回值。

```java
// 示例update有返回值，返回更新商品人员
public String update(){
    System.out.println("更新商品。。。");
    return "某某某更新了商品";
}

// 使用后置通知，定义增强方法
@AfterReturning(value = "execution(* com.zero.aspectj.demo1.ProductDao.update(..))",returning = "result")
public void afterReturning(Object result){ // 通过returning属性，可以定义方法返回值，作为参数
    System.out.println("后置通知>>>>>>>>"+result);
}
```



#### 2.4.3 使用环绕通知`@Around`

- `Around`方法的返回值就是目标代理方法执行的返回值
- 通过参数`ProceedingJoinPoing`，可以拦截目标方法执行

- 示例定义环绕通知方法

```java
// 环绕通知
@Around(value = "execution(* com.zero.aspectj.demo1.ProductDao.delete(..))")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    System.out.println("环绕前通知>>>>>>>>>"); // 环绕前代码区
    Object obj = joinPoint.proceed(); // 执行目标方法
    System.out.println("环绕后通知>>>>>>>>>");// 环绕后代码区
    return obj;
}
```



> 如果不调用`ProceedingJoinPoing`的`proceed`方法，那么目标方法就不会被执行，即被拦截了。





#### 2.4.4 异常抛出通知`@AfterThrowing`

- 通过设置`throwing`属性，可以设置发生异常时的处理

```java
// 目标类
public void findOne(){
    System.out.println("查询一个商品。。。");
    int i = 1/0; // 手动制造一个异常，测试异常通知
}


// 切面类中定义异常通知操作
// 异常通知
@AfterThrowing(value = "execution(* com.zero.aspectj.demo1.ProductDao.findOne(..))",throwing = "error") // 设置throwing，获取异常信息
public void afterThrowing(Throwable error){
    System.out.println("某某某方法有异常抛出>>>>>>>>>"+error.getMessage()); 
}
```



#### 2.4.5 最终通知`@After`

- 无论是否出现异常，最终通知总是会被执行的

```java
// 目标类
public void findAll(){
    System.out.println("查询所有商品。。。");
    int i = 1/0; // 手动制造异常，测试最终通知
}

// 最终通知,无论增强方法是否有异常都会执行
@After(value = "execution(* com.zero.aspectj.demo1.ProductDao.findAll(..))")
public void after(){
    System.out.println("最终通知>>>>>>>>>>>");
}
```



#### 2.4.6 通过`@Pointcut`为切点命名

- 在每个通知内定义切点，会造成工作量大，不易维护，对于重复的切点，可以使用`@Pointcut`进行定义
- 切点方法：`private void 无参方法，方法名为切点名`
- 当通知多个切点时，可以使用||进行连接。
- 切面类代码示例

```java
@Aspect
public class MyAspectAnno {

    // 前置通知
    @Before(value = "savePointcut()")  // 匹配任意返回值类型的，具体目标类(com.zero.aspectj.demo1.ProductDao)下的所有任意参数的方法
    public void before(JoinPoint joinPoint){
        System.out.println("前置通知>>>>>>>>>"+joinPoint); // 可以在方法中传入JoinPoint对象，用来获得切点信息，格式为：前置通知>>>>>>>>>execution(void com.zero.aspectj.demo1.ProductDao.save())
    }

    // 后置通知
    @AfterReturning(value = "updatePointcut()||findOnePointcut()",returning = "result") // 多个切点通知，用||连接
    public void afterReturning(Object result){ // 通过returning属性，可以定义方法返回值，作为参数
        System.out.println("后置通知>>>>>>>>"+result);
    }

    // 环绕通知
    @Around(value = "deletePointcut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕前通知>>>>>>>>>");
        Object obj = joinPoint.proceed(); // 执行目标方法
        System.out.println("环绕后通知>>>>>>>>>");
        return obj;
    }

    // 异常通知
    @AfterThrowing(value = "findOnePointcut()",throwing = "error") // 设置throwing，获取异常信息
    public void afterThrowing(Throwable error){
        System.out.println("某某某方法有异常抛出>>>>>>>>>"+error.getMessage());
    }

    // 最终通知
    @After(value = "findAllPointcut()")
    public void after(){
        System.out.println("最终通知>>>>>>>>>>>");
    }

    // 使用Pointcut为所有切点命名
    @Pointcut(value = "execution(* com.zero.aspectj.demo1.ProductDao.save(..))")
    private void savePointcut(){}
    @Pointcut(value = "execution(* com.zero.aspectj.demo1.ProductDao.delete(..))")
    private void deletePointcut(){}
    @Pointcut(value = "execution(* com.zero.aspectj.demo1.ProductDao.update(..))")
    private void updatePointcut(){}
    @Pointcut(value = "execution(* com.zero.aspectj.demo1.ProductDao.findOne(..))")
    private void findOnePointcut(){}
    @Pointcut(value = "execution(* com.zero.aspectj.demo1.ProductDao.findAll(..))")
    private void findAllPointcut(){}

}
```



## 3. XML方式实现AOP

### 3.1 使用XML配置切面

- 编写客户类及其实现类

```java
public class CustomerDaoImpl implements CustomerDao{
    public void save() {
        System.out.println("保存客户...");
    }
    public void update() {
        System.out.println("修改客户...");
    }
    public void delete() {
        System.out.println("删除客户...");
    }
    public void findOne() {
        System.out.println("查询一个客户...");
    }
    public void findAll() {
        System.out.println("查询全部客户...");
    }
}
```

- 编写切面类

```java
/**
 * 切面类，XML方式
 */
public class MyAspectXml {

    // 前置通知
    public void before(){
        System.out.println("XML方式的前置通知>>>>>>>>");
    }
    public void afterReturning(Object obj){
        System.out.println("后置增强>>>>>>>>"+obj);
    }
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable{
        System.out.println("环绕前增强>>>>>>>");
        Object obj = joinPoint.proceed();
        System.out.println("环绕后增强>>>>>>>");
        return obj;
    }
    public void afterThrowing(Throwable e){
        System.out.println("异常抛出通知>>>>>>"+e.getMessage());
    }
    public void after(){
        System.out.println("最终通知>>>>>>>>");
    }
}
```

- 完成切面类的配置

```xml
<!--使用XML配置方式完成AOP开发-->

<!--配置目标类-->
<bean id="customerDao" class="com.zero.aspectj.demo2.CustomerDaoImpl"/>

<!--配置切面类-->
<bean id="myAspectXml" class="com.zero.aspectj.demo2.MyAspectXml"/>
```

- 配置AOP完成增强

```xml
<!--进行AOP相关配置-->
<aop:config>
    <!--配置切入点：哪些类的哪些方法需要增强-->
    <aop:pointcut id="savePointcut" expression="execution(* com.zero.aspectj.demo2.CustomerDao.save(..))"/>
    <aop:pointcut id="updatePointcut" expression="execution(* com.zero.aspectj.demo2.CustomerDao.update(..))"/>
    <aop:pointcut id="deletePointcut" expression="execution(* com.zero.aspectj.demo2.CustomerDao.delete(..))"/>
    <aop:pointcut id="findOnePointcut" expression="execution(* com.zero.aspectj.demo2.CustomerDao.findOne(..))"/>
    <aop:pointcut id="findAllPointcut" expression="execution(* com.zero.aspectj.demo2.CustomerDao.findAll(..))"/>

    <!--配置AOP的切面-->
    <aop:aspect ref="myAspectXml">
        <!--配置前置通知-->
        <aop:before method="before" pointcut-ref="savePointcut"/>
        <!--配置后置通知-->
        <aop:after-returning method="afterReturning" pointcut-ref="updatePointcut" returning="obj"/>
        <!--配置环绕通知-->
        <aop:around method="around" pointcut-ref="deletePointcut"/>
        <!--配置异常抛出通知-->
        <aop:after-throwing method="afterThrowing" pointcut-ref="findOnePointcut" throwing="e"/>
        <!--配置最终通知-->
        <aop:after method="after" pointcut-ref="findAllPointcut" />
    </aop:aspect>
</aop:config>
```

- 测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext2.xml")
public class Demo2Test {

    @Resource(name = "customerDao")
    private CustomerDao customerDao;


    @Test
    public void test1(){
        customerDao.save();
        customerDao.update();
        customerDao.delete();
        customerDao.findOne();
        customerDao.findAll();
    }
}
```





## 总结

- 使用AspectJ完成AOP开发所需环境的配置，需要哪些jar包?

```xml
<!--导入Spring基本开发包-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.2.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.2.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>4.2.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>4.2.4.RELEASE</version>
</dependency>
<!--导入AOP相关包-->
<dependency>
    <groupId>aopalliance</groupId>
    <artifactId>aopalliance</artifactId>
    <version>1.0</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>4.2.4.RELEASE</version>
</dependency>
<!--导入AspectJ相关包-->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>1.8.9</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.2.4.RELEASE</version>
</dependency>
<!--导入测试相关包-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>4.2.4.RELEASE</version>
</dependency>
<!--导入DI属性注入相关包,java注解类-->
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.2</version>
</dependency>
```

- 注解方式实现AOP
- AspectJ的各种通知类型
- 切面类的定义，切入点的配置
- XML方式实现AOP，AOP的XML配置

