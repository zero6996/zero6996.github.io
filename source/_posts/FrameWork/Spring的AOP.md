---
title: Spring的AOP
date: 2019-8-8 22:30
categories: Spring
tags: [Spring]
description: Spring的AOP
---



## 1. AOP的概念

### 1.1 什么是AOP？

在软件业，AOP为Aspect Oriented Programming的缩写，意为：[面向切面编程](https://baike.baidu.com/item/面向切面编程/6016335)，通过[预编译](https://baike.baidu.com/item/预编译/3191547)方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是[OOP](https://baike.baidu.com/item/OOP)的延续，是软件开发中的一个热点，也是[Spring](https://baike.baidu.com/item/Spring)框架中的一个重要内容，是[函数式编程](https://baike.baidu.com/item/函数式编程/4035031)的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的[耦合度](https://baike.baidu.com/item/耦合度/2603938)降低，提高程序的可重用性，同时提高了开发的效率。



<!--more-->



- AOP(Aspect Oriented Programing)，面向切面编程
- AOP采取横向抽取的机制，取代了传统纵向继承体系重复性diamante(性能监视、事务管理、安全检查、缓存)
- Spring AOP使用纯Java实现，不需要专门的编译过程和类加载器，在运行期通过**代理方式**向目标类织入增强代码。



### 1.2 AOP相关术语

- **`Joinpoint`(连接点)：所谓连接点是指那些被拦截到的点**。在spring中，这些**点指的是方法**，因为spirng只支持方法类型的连接点。
- `Pointcut`(切入点)：所谓切入点是指我们要对哪些`Joinpoint`进行拦截的定义。
- **`Advice`(通知/增强)：所谓通知是指拦截到`Joinpoint`之后所要做的事情就是通知**。通知分为前置通知，后置通知，异常通知，最终通知，环绕通知(切面要完成的功能)
- `Introduction`(引介)：引介是一种特殊的通知在不修改类代码的前提下，`Introduction`可以在运行期为类动态地添加一些方法或Filed。
- **`Target`(目标对象)：代理的目标对象。**
- `Weaving`(织入)：是指把增强应用到目标对象来创建新的代理对象的过程。spring采用动态代理织入，而AspectJ采用编译器织入和类装载期织入。
- **`Proxy`(代理)：一个类被AOP织入增强后，就产生一个结果代理类。**
- `Aspect`(切面)：是切入点和通知(引介)的结合。



![aop](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/09/aop-1565345189637.jpg)


## 2. AOP的底层实现

### 2.1 JDK动态代理

使用JDK本身的一个类`Proxy`来实现动态代理，核心代码如下：

```java
public class MyJDKProxy implements InvocationHandler {
    private final UserDao userDao;
    public MyJDKProxy(UserDao userDao){
        this.userDao = userDao;
    }
    public Object createProxy(){
        Object proxy = Proxy.newProxyInstance(userDao.getClass().getClassLoader(),userDao.getClass().getInterfaces(),this);
        return proxy;
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("所有代理对象的方法都会经过invoke调用");
        if ("save".equals(method.getName())){
            // 进行权限校验代码区
            System.out.println("权限校验...");
            return method.invoke(userDao,args);
        }
        return method.invoke(userDao,args);
    }
}
```



### 2.2 使用CGLIB生成代理

- 对于不使用接口的业务类，无法使用JDK动态代理
- CGlib采用非常底层字节码技术，可以为一个类创建子类，解决无接口代理问题。

```java
/**
 * 示意CGlib生成代理
 */
public class MyCGlibProxy implements MethodInterceptor {
    private final UserDao userDao;
    public MyCGlibProxy(UserDao userDao){
        this.userDao = userDao;
    }
    public Object createProxy(){
        // 1. 创建核心类
        Enhancer enhancer = new Enhancer();
        // 2. 设置父类
        enhancer.setSuperclass(userDao.getClass());
        // 3. 设置回调
        enhancer.setCallback(this);
        // 4. 生成代理
        Object proxy = enhancer.create();
        return proxy;
    }
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        if ("save".equals(method.getName())){
            System.out.println("权限校验");
            return methodProxy.invokeSuper(proxy,args);
        }
        return methodProxy.invokeSuper(proxy,args);
    }
}
```



### 2.3 小结

- Spring在运行期，生成动态代理对象，不需要特殊的编译器
- Spring AOP的底层就是通过JDK动态代理或CGlib动态代理技术为目标Bean执行横向织入
  - 若目标对象实现了若干接口，spring使用JDK的`java.lang.reflect.Proxy`类代理
  - 若目标对象没有实现任何接口，spring使用CGlib库生成目标对象的子类
- 程序中应优先对接口创建代理，便于程序解耦维护
- 标记为final的方法，不能被代理，因为无法进行覆盖
  - JDK动态代理，是针对接口生成子类，接口中方法不能使用final修饰
  - CGlib是针对目标类生成子类，因此类或方法不能使用final
- Spring只支持方法连接点，不提供属性连接

## 3. Spring的传统AOP

### 3.1 Spring AOP增强类型

- AOP联盟为通知Advice定义了`org.aopalliance.aop.Interface.Advice`
- Spring按照通知Advice在目标类方法的连接点位置，可以分为5类
  - 前置通知`org.springframework.aop.MethodBeforeAdvice`：在目标方法执行前实施增强
  - 后置通知`org.springframework.aop.AfterReturningAdvice`：在目标方法执行后实施增强
  - 环绕通知`org.aopalliance.intercept.MethodInterceptor`：在目标方法执行前后实施增强
  - 异常抛出通知`org.springframework.aop.ThrowsAdvice`：在方法抛出异常后实施增强
  - 引介通知`org.springframework.aop.IntroductionInterceptor`：在目标类中添加一些新的方法和属性



### 3.2 Spring AOP切面类型

- `Advisor`：代表一般切面，Advice本身就是一个切面，对目标类所有方法进行拦截
- `PointcutAdvisor`：代表具有切点的切面，可以指定拦截目标类哪些方法
- `IntroductionAdvisor`：代表引介切面，针对引介通知而使用切面(了解即可)



### 3.3 `Advisor`切面案例

`ProxyFactoryBean`常用可配置属性：

- `target`：代理的目标对象
- `proxyInterfaces`：代理要实现的接口
- `proxyTargetClass`：是否对类代理而不是接口，设置为true时，使用CGlib代理
- `interceptorNames`：需要织入目标的Advice
- `singleton`：返回代理是否为单实例，默认为单例
- `optimize`：当设置为true时，强制使用CGlib



代码示例：

- 创建学生接口及其实现类

```java
public interface StudentDao {
    void save();
    void update();
    void delete();
    void find();
}
public class StudentDaoImpl implements StudentDao {
    public void save() {
        System.out.println("学生保存");
    }
    public void update() {
        System.out.println("学生修改");
    }
    public void delete() {
        System.out.println("学生删除");
    }
    public void find() {
        System.out.println("学生查询");
    }
}
```

- 创建前置通知增强类

```java
// 使用前置通知实现增强类
public class MyBeforeAdvice implements MethodBeforeAdvice {
    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println("前置增强>>>>>>");
    }
}
```

- 配置xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

<!--    配置目标类-->
    <bean id="studentDao" class="com.zero.aop.demo2.StudentDaoImpl"/>

<!--    前置通知类型-->
    <bean id="myBeforeAdvice" class="com.zero.aop.demo2.MyBeforeAdvice"/>

<!--    Sprint AOP 产生代理对象-->
    <bean id="studentDaoProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
<!--        配置目标类-->
        <property name="target" ref="studentDao"/>
<!--        实现的接口-->
        <property name="proxyInterfaces" value="com.zero.aop.demo2.StudentDao"/>
<!--        采用拦截的名称-->
        <property name="interceptorNames" value="myBeforeAdvice"/>
    </bean>
</beans>
```

- 测试

```java
@RunWith(SpringJUnit4ClassRunner.class) 
@ContextConfiguration("classpath:applicationContext.xml")
public class demo2Test {

//    @Resource(name = "studentDao") 
    @Resource(name = "studentDaoProxy") // 使用代理对象
    private StudentDao studentDao;

    @Test
    public void test1(){
        studentDao.find();
        studentDao.delete();
        studentDao.save();
        studentDao.update();
    }
}

```



### 3.4 `PoingcutAdvisor`切点切面

- 使用普通Advice作为切面，将对目标类所有方法进行拦截，不够灵活，在实际开发中常采用带有切点的切面
- 常用`PointcutAdvisor`实现类
  - `DefaultPointcutAdvisor`：最常用的切面类型，它可以通过任意`Pointcut`和Advice组合定义切面
  - `JdkRegexpMethodPointcut`：构造正则表达式切点

- 示例，创建客户类

```java
public class CustomerDao {
    public void save() {
        System.out.println("客户保存");
    }
    public void update() {
        System.out.println("客户修改");
    }
    public void delete() {
        System.out.println("客户删除");
    }
    public void find() {
        System.out.println("客户查询");
    }
}
```

- 创建环绕通知型增强类

```java
public class MyAroundAdvice implements MethodInterceptor {
    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("环绕前增强>>>>>>");
        // 执行目标方法
        Object obj = invocation.proceed();
        System.out.println("环绕后增强>>>>>>");
        return obj;
    }
}
```

- 配置xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<!--配置目标类-->
    <bean id="customerDao" class="com.zero.aop.demo3.CustomerDao"/>

	<!--配置通知-->
    <bean id="myAroundAdvice" class="com.zero.aop.demo3.MyAroundAdvice"/>

	<!--一般的切面是使用通知作为切面的，因为要对目标类的指定方法进行增强就需要配置一个带有切入点的切面-->
    <bean id="MyAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
		<!--pattern中配置正则表达式，以达到对指定方法进行增强的效果-->
        <!--
            对单个方法进行增强使用pattern即可
            如果对多个方法，需使用patterns，多个值用逗号隔开
        -->
		<!--<property name="pattern" value=".*save.*"/>-->
        <property name="patterns" value=".*save.*,.*delete.*"/>
        <property name="advice" ref="myAroundAdvice"/>
    </bean>
	<!--配置产生代理-->
    <bean id="customerDaoProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="customerDao"/>
        <property name="proxyTargetClass" value="true"/>
        <property name="interceptorNames" value="MyAdvisor"/>
    </bean>
</beans>
```

- 测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext2.xml")
public class demo3Test {

//    @Resource(name="customerDao") // 不使用代理
    @Resource(name = "customerDaoProxy") // 使用代理对象
    private CustomerDao customerDao;

    @Test
    public void test1(){
        customerDao.find();
        customerDao.delete();
        customerDao.save();
        customerDao.update();
    }
}
```



## 4. Spring传统AOP的自动代理请

- 前面的案例中，每个代理都是通过`ProxyFactoryBean`织入切面代理，在实际开发中，非常多的Bean每个都配置`ProxyFactoryBean`的话，开发维护量巨大
- 解决方案：自动创建代理
  - `BeanNameAutoProxyCreator`：根据Bean名称创建代理
  - `DefaultAdvisorAutoProxyCreator`：根据Advisor本身包含信息创建代理
  - `AnnotationAwareAspectJAutoProxyCreator`：基于Bean中的AspectJ注解进行自动代理

### 4.1 基于Bean名称的自动代理

- 代码示例对所有以DAO结尾Bean所有方法进行代理

  - 将前面案例的`CustomerDao,StudentDao,StudentDaoImpl,MyBeforeAdvice,MyAroundAdvice`复制到一个文件夹，然后配置xml。

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  <!--    配置目标类-->
      <bean id="studentDao" class="com.zero.aop.demo4.StudentDaoImpl"/>
      <bean id="customerDao" class="com.zero.aop.demo4.CustomerDao"/>
      <!--配置通知：前置通知-->
      <bean id="myBeforeAdvice" class="com.zero.aop.demo4.MyBeforeAdvice"/>
      <!--配置通知：环绕通知-->
      <bean id="myAroundAdvice" class="com.zero.aop.demo4.MyAroundAdvice"/>
  
      <bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
          <property name="beanNames" value="*Dao"/>
          <property name="interceptorNames" value="myBeforeAdvice"/>
      </bean>
  </beans>
  ```

  - 测试

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration("classpath:applicationContext3.xml")
  public class demo4Test {
  
      @Resource(name = "studentDao")
      private StudentDao studentDao;
      @Resource(name = "customerDao")
      private CustomerDao customerDao;
  
      @Test
      public void test1(){
          studentDao.find();
          studentDao.delete();
          studentDao.save();
          studentDao.update();
  
          customerDao.find();
          customerDao.delete();
          customerDao.save();
          customerDao.update();
  
  
      }
  }
  ```

  

### 4.2 基于切面信息的自动代理

- 配置环绕代理，将上述案例xml修改如下即可对指定方法进行增强

```xml
<!--根据切面信息创建代理-->
<!--配置目标类-->
<bean id="studentDao" class="com.zero.aop.demo4.StudentDaoImpl"/>
<bean id="customerDao" class="com.zero.aop.demo4.CustomerDao"/>
<!--配置通知：前置通知-->
<bean id="myBeforeAdvice" class="com.zero.aop.demo4.MyBeforeAdvice"/>
<!--配置通知：环绕通知-->
<bean id="myAroundAdvice" class="com.zero.aop.demo4.MyAroundAdvice"/>

<!--配置切面-->
<bean id="myAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <!--对指定包下指定方法进行增强,注意转义-->
    <property name="pattern" value="com\.zero\.aop\.demo4\.CustomerDao\.save"/>
    <property name="advice" ref="myAroundAdvice"/>
</bean>
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"></bean>
```



## 总结

- 什么是AOP
- 了解AOP相关术语：连接点，切入点，织入，目标对象，代理对象等。
- 传统AOP实现原理：JDK动态代理，CGlib代理
- Spring的传统AOP：增强类型，切面类型。没有切入点的切面和有切入点的切面。
- 自动代理：基于Bean名称的，基于切面信息的

