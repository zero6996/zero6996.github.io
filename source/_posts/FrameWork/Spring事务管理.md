---
title: Spring的事务管理
date: 2019-8-18 23:52
categories: FrameWork
tags: [Spring]
description: Spring的事务管理
---



## 1. Java事务导引

### 1.1 事务简介

- 什么是事务
  - 事务是**正确执行**一系列的操作(或动作)，使得数据库从一种状态转换成另一种状态，且保证操作**全部成功**，或者**全部失败**。



<!--more-->



- 事务原则是什么
  - 事务必须服从ISO/IEC所制定的ACID原则
- ACID原则具体内容如下：
  - 原子性（Atomicity）：即不可分割性，事务要么全部被执行，要么就全部不被执行。
  - 一致性（Consistency）：事务的执行使得数据库从一种正确状态转换成另一种正确状态。
  - 隔离性（Isolation）：在事务正确提交之前，它可能的结果不应显示给任何其他事务。
  - 持久性（Durability）：事务正确提交后，其结果将永久保存在数据库中。



### 1.2 Java事务

- Java事务的产生
  - 程序操作数据库的需要。以Java编写的程序或系统，实现ACID的操作。
- Java事务实现
  - 通过JDBC相应方法间接来实现对数据库的增删改查，把事务转移到Java程序代码中进行控制。
  - 确保事务要么全部执行成功，要么撤销不执行。
- 总结：Java事务机制和原理就是操作确保数据库操作的ACID特性。



### 1.3 Java事务实现模式

- Java事务的实现
  - 通过Java代码来实现对数据库的事务性操作
- Java事务类型
  - JDBC事务：用Connection对象控制的手动模式和自动模式
  - JTA(Java Transaction API)事务：与实现无关的，与协议无关的API
  - 容器事务：应用服务器提供的，且大多是基于JTA完成(通常基于JNDI的，相当复杂的API实现)



### 1.4 三种事务的差异

- JDBC事务：控制的局限性在一个数据库连接内，但是其使用简单。
- JTA事务：功能强大，可跨越多个数据库或多DAO，使用比较复杂。
- 容器事务：主要指的是J2EE应用服务器提供的事务管理，局限于EJB应用使用。



## 2. Spring事务核心接口

### 2.1 事务接口架构

![framework](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/18/framework-1566143165106.jpg)


### 2.2 Spring事务管理器

- JDBC事务管理器(`DataSourceTransactionManager`)
  - 本事务管理器是通过调用`java.sql.Connection`来管理事务。
  - Spring配置示例
- Hibernate事务管理器(`HibernateTransactionManager`)
  - 本管理器将事务管理的职责委托给`org.hibernate.Transaction`对象来管理事务，而后者是从Hibernate Session中获取到的。
- JPA事务管理器(`JpaTransactionManager`)
  - 通过一个JPA实体管理工厂(`javax.persistence.EntityManagerFactory`接口的任意实现)将与由工厂所产生的JPA `EntityManager`合作来构建事务。
- JTA事务管理器(`JtaTransactionManager`)
  - 本管理器将事务管理的责任委托给`javax.transaction.UserTransaction`和`javax.transaction.TransactionManager`对象进行事务处理。





### 2.3 Spring事务属性定义

![Transaction_attribute](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/18/Transaction_attribute-1566143193689.jpg)


### 2.4 事务隔离级别

#### 2.4.1 数据读取类型说明

- 脏读
  - 脏读发生在一个事务A读取了被另一个事务B修改，但是还未提交的数据。假如B回滚，则事务A读取的就是无效数据。跟不可重复读类似，但是第二个事务无需执行提交。
- 不可重复读
  - 在基于锁的并行控制方法中，如果在执行select时不添加读锁，就会发生不可重复读问题，导致两次读取的数据不一致。
- 幻读
  - 幻读发生在当两个完全相同的查询执行时，第二次查询所返回的结果集跟第一次查询的不相同。



[参考文章](https://www.cnblogs.com/balfish/p/8298296.html)



#### 2.4.2 事务隔离级别

- 隔离级别定义了一个事务可能受其他并发事务影响的程度
- 隔离级别如下：

| 隔离级别                   | 含义                                                         |
| -------------------------- | ------------------------------------------------------------ |
| ISOLATION_DEFAULT          | 使用后端数据库默认的隔离级别                                 |
| ISOLATION_READ_UNCOMMITTED | 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读 |
| ISOLATION_READ_COMMITTED   | 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生 |
| ISOLATION_REPEATABLE_READ  | 对同一字段的多次读取结果都是一致的，除非数据时被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生 |
| ISOLATION_SERIALIZABLE     | 最高的隔离级别，完全服从ACID的隔离级别，确保阻止脏读、不可重复读和幻读。也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的 |



### 2.5 事务传播行为

- 当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。即两个业务之间如何共享事务
- Spring的7种传播行为：

| 传播行为                  | 含义                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务 |
| PROPAGATION_SUPPORTS      | 表示当前方法无需事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行 |
| PROPAGATION_MANDATORY     | 表示该方法必须在事务中运行，如果当前事务不存在，则会抛异常   |
| PROPAGATION_REQUIRED_NEW  | 表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用`JTATransactionManager`的话，则需要访问`TransactionManager` |
| PROPAGATION_NOT_SUPPORTED | 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用`JTATransactionManager`的话，则需要访问`TransactionManager` |
| PROPAGATION_NEVER         | 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常 |
| PROPAGATION_NESTED        | 表示如果当前存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与`PROPAGATION_REQUIRED`一样。注意各厂商对这种传播行为的支持是有所差异的。参考资源管理器的文档来确认它们是否支持嵌套事务。 |



### 2.6 事务中注意的问题

- 事务是否只读
  - 利用数据库事务的“只读”属性，进行特点优化处理
- 注意：
  - 事务的是否“只读”属性，不同的数据库厂商支持不同
  - 通常而言：只读属性的应用要参考厂商的具体支持说明，比如：
    - Oracle的“readOnly”不起作用，不影响其增删改查
    - MySQL的“readOnly”为true，只能查，增删改则出异常
- 事务超时
  - 事务超时就是事务的一个定时器，在特定时间内事务如果没有执行完毕，那么就会自动回滚，而不是一直等待其结束。
- 设计事务时注意点
  - 为了使应用程序更好的运行，事务不能运行太长的时间。因为事务可能涉及对后端数据库的锁定，所以长时间的事务会不必要的占用数据库资源。
- 事务回滚
  - 默认情况下，事务只有遇到运行期异常才会回滚，而在遇到检查型异常时不会回滚。
- 自定义回滚策略
  - 声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚；
  - 声明事务遇到特定的异常不回滚，即使这些异常时运行期异常。

### 2.7 Spring事务状态

- 事务接口

  - 通过事务管理器获取`TransactionStatus`实例；
  - 控制事务在回滚或提交时需要应用对应的事务状态；
  - Spring事务接口：

  ```java
  // Spring事务状态接口：
  // 通过调用PlatformTransactionManager的getTransaction()
  // 获取事务状态实例
  public interface TransactionStatus{
      boolean isNewTransaction(); // 是否是新的事务
      boolean hasSavepoint(); // 是否有恢复点
      void setRollbackOnly(); // 设置为只回滚
      boolean isRollBackOnly(); // 是否为只回滚
      boolean isCompleted; // 是否已完成
  }
  ```



### 2.8 转账案例

通过代理工厂实现事务

#### 2.8.1 环境搭建

创建用户数据库表

```sql
create table account(
id int primary key auto_increment,
username varchar(50),
money int);

insert into account(username,money) values('jack',10000);
insert into account(username,money) values('rose',10000);
```

#### 2.8.2 导入jar包

- spring核心：4+1(core+context+beans+expression)+logging
- aop：4（aop联盟、spring aop、aspectj规范、spring aspect）
- 数据库：jdbc、tx
- 驱动：mysql
- 连接池：dbcp



#### 2.8.3 创建Dao层

~~~java
public class AccountDaoImpl extends JdbcDaoSupport implements AccountDao { 
    // JdbcDaoSupport，在xml中配置数据源即可使用
    // 转出
    @Override
    public void out(String outer, Integer money) {
        String sql = "update account set money = money - ? where username = ?";
        getJdbcTemplate().update(sql,money,outer);
    }
    // 进账
    @Override
    public void in(String inner, Integer money) {
        String sql = "update account set money = money + ? where username = ?";
        getJdbcTemplate().update(sql,money,inner);
    }
}
~~~



#### 2.8.4 创建Service层

```java
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;
    // 提供set方法，让spring注入属性
    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    @Override
    public void transfer(String outer, String inner, Integer money) {
        this.accountDao.out(outer,money);
        int i = 1/0; // 手动制造异常，测试事务是否其效果
        this.accountDao.in(inner,money);
    }
}
```



#### 2.8.5 配置文件

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

    <!--配置Dao-->
    <bean id="accountDao" class="com.zero.transaction.dao.Impl.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置事务管理器-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--配置service-->
    <bean id="accountService" class="com.zero.transaction.service.Impl.AccountServiceImpl">
       	<!--添加属性accountDao-->
        <property name="accountDao" ref="accountDao"/>
    </bean>

    <!--配置代理工厂-->
    <bean id="proxyService" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
        <!--接口，注意配置全类名-->
        <property name="proxyInterfaces" value="com.zero.transaction.service.AccountService"/> 
        <!--目标对象-->
        <property name="target" ref="accountService"/>
        <!--切面对象，SPring做了，就不用写了-->
        <!--事务管理器-->
        <property name="transactionManager" ref="txManager"/>
        <!--
            事务属性/详情配置
            prop.key ：确定哪些方法使用当前事务配置
            prop.text:用于配置事务详情
            格式：PROPAGATION,ISOLATION,readOnly,-Exception,+Exception
            传播行为		 隔离级别	      是否只读	异常回滚	      异常提交
        -->
        <property name="transactionAttributes">
            <props>
                <!--可以配置隔离级别-->
                <prop key="transfer">PROPAGATION_REQUIRED,ISOLATION_DEFAULT</prop>
            </props>
        </property>
    </bean>
</beans>
```

#### 2.8.6 测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:transfer.xml")
public class transferTest {

//    @Autowired
    @Resource(name = "proxyService")
    private AccountService accountService;

    // 转账测试
    @Test
    public void test1(){
        // 获取service
        accountService.transfer("jack","rose",100);
    }
}
```





## 3. 编程式事务管理

### 3.1 编程式事务实现方式



- 模板事务(`TransactionTemplate`)的方式
  - 此为Spring官方团队推荐的编程式事务管理方式
  - 主要工具为`JdbcTemplate`类。
- 平台事务管理器(`PlatformTransactionManager`)方式
  - 类似应用`JTA UserTransaction API`方式，但异常处理更简洁；
  - 辅助类为：`TransactionDefinition`和`TransactionStatus`



### 3.2 编程式事务实现案例

- 模板事务(`TransactionTemplate`)案例
  - 步骤：获取模板对象；选择事务结果类型；业务数据操作处理。

- 平台事务管理器(`PlatformTransactionManager`)案例
  - 步骤：获取事务管理器；获取事务属性对象；获取事务状态对象；创建JDBC模板对象；业务数据操作处理。
- 数据库工具类DButil

```java
public class DBUtil {
    static  final String dbDriver = "com.mysql.jdbc.Driver" ;
    static final String dbUrl = "jdbc:mysql://localhost:3306/selection_course?useUnicode=true&characterEncoding=utf-8" ;//
    static final String userName = "root" ;
    static final String password = "123456" ;

    static {
        try{//加载MySql的驱动类
            Class.forName(dbDriver) ;
        }catch(ClassNotFoundException e){
            System.out.println("找不到驱动程序类 ，加载驱动失败！");
            e.printStackTrace() ;
        }
    }
    public static Connection getConnection(){
        try {
            return DriverManager.getConnection(dbUrl , userName , password ) ;
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null ;
    }
}
```

- 数据库工具类`TemplateUtils`

```java
/**
 * Spring数据库操作工具类
 */
public class TemplateUtils {
	private final static  String dbDriver = "com.mysql.jdbc.Driver" ;
	private final static  String dbUrl = "jdbc:mysql://127.0.0.1:3306/selection_course?useUnicode=true&characterEncoding=utf-8" ;//
	private final static  String dbUser = "root";
	private final static  String dbPwd = "123456";
	
	private static BasicDataSource dataSource ;
	//静态初识：创建连接数据源
	static {
	//创建DBCP简单数据源并初始化相关数据源属性
	//private void createSimpleDataSource(){
		dataSource = new   BasicDataSource() ;
		dataSource.setDriverClassName(dbDriver);
		dataSource.setUrl(dbUrl);
		dataSource.setUsername(dbUser);
		dataSource.setPassword(dbPwd);
//		//指定数据库连接池初始连接数
//		dataSource.setInitialSize(10);
//		//设定同时向数据库申请的最大连接数
//		dataSource.setMaxTotal(50);
//		//设置连接池中保持的最少连接数量
//		dataSource.setMinIdle(5);
	//}
	}
	public static TransactionTemplate getTransactionTemplate() {  
        PlatformTransactionManager txManager = new DataSourceTransactionManager(  
                dataSource);  
        return new TransactionTemplate(txManager);  
    }  
  
    public static JdbcTemplate getJdbcTemplate() {  
        return new JdbcTemplate(dataSource);  
    }  
  
    public static NamedParameterJdbcTemplate getNamedParameterJdbcTemplate() {  
        return new NamedParameterJdbcTemplate(dataSource);  
    }  
  
    public static SimpleJdbcInsert getSimpleJdbcTemplate() {  
        return new SimpleJdbcInsert(dataSource);  
    }  
    
    /**
     * //获取事务管理器：TransactionManager
     * 根据需要，可以是如JDBC、Hibernate,这里定义JDBC事务管理其
     * @return DataSourceTransactionManager
     */
    public static DataSourceTransactionManager getDataSourceTransactionManager(){
    	 DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();
    	 // 设置数据源:此事务数据源须和正式事务管理器的数据源一致
    	 dataSourceTransactionManager.setDataSource(dataSource);
    	 return dataSourceTransactionManager;
    }
}

```



- 事务管理器示例

```java
/**
 *  事务管理器方式案例
 */
public class ProTransExample {

    // 事务模板：第二种事务编程模式
    // private TransactionTemplate transactionTemplate ;

    // 数据持久化操作
    public void addBook(final Book book) {
        // 获取事务模板对象
        TransactionTemplate tt = TemplateUtils.getTransactionTemplate();
        // 可设置事务属性，如隔离级别、超时时间等,如：
        // tt.setIsolationLevel(TransactionDefinition.ISOLATION_READ_UNCOMMITTED);
        // 内部类
        tt.execute(new TransactionCallbackWithoutResult() {
            protected void doInTransactionWithoutResult(TransactionStatus s) {
                try {
                    // 数据库操作1
                    // JdbcTemplate jdbcTemplate
                    // =TemplateUtils.getJdbcTemplate();
                    // jdbcTemplate.execute(sql);
                    // 简单模板化新增数据
                    SimpleJdbcInsert simpleInsert = TemplateUtils.getSimpleJdbcTemplate();
                    simpleInsert.withTableName("books").usingColumns("isbn", "name", "price", "pubdate");
                    Map<String, Object> parameters = new HashMap<String, Object>();
                    parameters.put("isbn", book.getIsbn());
                    parameters.put("name", book.getName());
                    parameters.put("price", book.getPrice());
                    parameters.put("pubdate", book.getPubdate());
                    simpleInsert.execute(parameters);
                    System.out.println("新增数据成功！");
                    // 或者DAO数据操作模式：
                    // BookDAO.save(book);
                } catch (Exception e) {
                    s.setRollbackOnly();
                    e.printStackTrace();
                }
            }
        });
    }

    public Book findBookByIsbn(final String isbn) {
        TransactionTemplate tt = TemplateUtils.getTransactionTemplate();
        Book book = null;
        @SuppressWarnings("unchecked")
        List<Map<String, Object>> books = (List<Map<String, Object>>) tt.execute(new TransactionCallback<Object>() {
            public Object doInTransaction(TransactionStatus arg0) {
                JdbcTemplate jdbcTemplate = TemplateUtils.getJdbcTemplate();
                return jdbcTemplate.queryForList("select isbn,name,price,pubdate from books where isbn ='" + isbn + "'");
            }
        });
        if (books.size() != 0) {// 封装获取的数据
            Map<String, Object> m = (Map) books.get(0);
            book = new Book();
            book.setIsbn(m.get("isbn").toString());
            book.setName(m.get("name").toString());
            book.setPrice((Float) m.get("price"));
            book.setPubdate((Date) m.get("pubdate"));
        }
        return book;
    }

    // 1、编程式事务管理：事务管理器PlatformTransactionManager方式实现
    public void updateBookByIsbn(Book book) {
        //第一步：获取JDBC事务管理器
        DataSourceTransactionManager dtm = TemplateUtils.getDataSourceTransactionManager();
        // 第二步：创建事务管理器属性对象
        DefaultTransactionDefinition transDef = new DefaultTransactionDefinition(); // 定义事务属性
        // 根据需要，设置事务管理器的相关属性
        // 设置传播行为属性
        transDef.setPropagationBehavior(DefaultTransactionDefinition.PROPAGATION_REQUIRED);
        // 第三步：获得事务状态对象
        TransactionStatus ts = dtm.getTransaction(transDef);
        // 第四步：基于当前事务管理器,获取数据源，创建操作数据库的JDBC模板对象
        JdbcTemplate jt = new JdbcTemplate(dtm.getDataSource());
        try {
            //第五步：业务操作
            jt.update("update books set price="+book.getPrice()+",name='"+book.getName()
                    +"'  where isbn='"+book.getIsbn()+"' ");
            // 其它数据操作如增删
            //第六步：提交事务
            dtm.commit(ts); // 如果不commit，则更新无效果
            System.out.println("修改成功");
        } catch (Exception e) {
            // 报错直接回滚事务
            dtm.rollback(ts);
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        ProTransExample pte = new ProTransExample();
        Book book = pte.findBookByIsbn("128-166-890-China"); // 查询数据
        if (book != null){ // 如果查询到就更新数据
            book.setName("华夏龙图腾");
            book.setPrice(125.5f);
            pte.updateBookByIsbn(book);
        }else {
            // 如果没有则添加数据
            Book addbook = new Book();
            addbook.setIsbn("128-166-890-China");
            addbook.setName("华夏龙图腾");
            addbook.setPrice(128.5f);
            pte.addBook(addbook);
        }



        /*
         *1-新增书籍
         * Date pd = new Date(System.currentTimeMillis()); Book book = new
         * Book(); book.setIsbn("120-166-990-China"); book.setName("人性的弱点");
         * book.setPrice(65.6f); book.setPubdate(pd);
         */
        // pte.addBook(book);
        //2-查找书籍
		/*Book bk = pte.findBookByIsbn("120-166-890-China");
		System.out.println("书籍名称：" + bk.getName());*/
        //3-更新书籍名称和价格

    }

}
```



### 3.3 小结

- 需要有效的数据源，具体数据源根据实际情况创建
- 创建编程事务管理对象
  - 事务模板(`TransactionTemplate`)
  - 事务管理器(`PlateformTransactionManager`)
- 业务逻辑处理
  - 基于`JdbcTemplate`完成业务处理。



## 4. 声明式事务管理(掌握)

### 4.1 声明式事务实现方式

- 声明式事务管理的配置类型
  
  - 5种类型：独立代理；共享代理；拦截器；tx拦截器；全注释。
  
- 声明式事务管理配置实现方式：
  
  - 5种类型的配置实现参考
  
- 转账案例**基于AOP的事务配置**：

  - 基本代码不变，修改xml配置文件如下

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
  
      <!--开启自动扫描-->
      <context:component-scan base-package="com.zero.jdbc_Template"/>
  
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
  
      <!--配置Dao-->
      <bean id="accountDao" class="com.zero.transaction.dao.Impl.AccountDaoImpl">
          <property name="dataSource" ref="dataSource"/>
      </bean>
  
      <!--配置事务管理器-->
      <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <property name="dataSource" ref="dataSource"/>
      </bean>
      <!--配置service-->
      <bean id="accountService" class="com.zero.transaction.service.Impl.AccountServiceImpl">
          <property name="accountDao" ref="accountDao"/>
      </bean>
  
      <!--使用spring的aop来配置事务-->
  
      <!--1. 配置通知事务管理器-->
      <tx:advice id="txAdvice" transaction-manager="txManager">
          <!--事务详情：传播行为，隔离级别-->
          <tx:attributes>
              <tx:method name="transfer" propagation="REQUIRED" isolation="DEFAULT"/>
          </tx:attributes>
      </tx:advice>
      <!--2. 事务通知与切入点关联-->
      <aop:config>
  <!--        <aop:pointcut id="myPoinycut" expression="execution(* com.zero.transaction.service..*.*(..))"/>-->
  <!--        <aop:advisor advice-ref="" pointcut-ref="myPoinycut"/>-->
          <!--事务与连接点关联-->
          <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.zero.transaction.service..*.*(..))"/>
      </aop:config>
  
  </beans>
  ```

  - 测试

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration("classpath:transfer_aop.xml")
  public class transferTest {
  
      @Autowired // 使用自动注入
  //    @Resource(name = "proxyService")
      private AccountService accountService;
  
      // 转账测试
      @Test
      public void test1(){
          // 获取service
          accountService.transfer("jack","rose",100);
      }
  
      // 使用aop配置事务
      @Test
      public void test2(){
          accountService.transfer("jack","rose",100);
  
      }
  }
  
  ```



- 使用注解来实现事务

  - 基本代码不变，修改xml如下

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
  
      <!--开启自动扫描-->
      <context:component-scan base-package="com.zero.jdbc_Template"/>
  
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
  
      <!--配置Dao-->
      <bean id="accountDao" class="com.zero.transaction.dao.Impl.AccountDaoImpl">
          <property name="dataSource" ref="dataSource"/>
      </bean>
  
      <!--配置service-->
      <bean id="accountService" class="com.zero.transaction.service.Impl.AccountServiceImpl">
          <property name="accountDao" ref="accountDao"/>
      </bean>
  
      <!--使用spring的注解来实现事务-->
  
      <!--1. 配置事务管理器-->
      <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <property name="dataSource" ref="dataSource"/>
      </bean>
      <!--2. 开启事务注解驱动-->
      <tx:annotation-driven transaction-manager="txManager"/>
  
  </beans>
  ```

  - 在需要开启事务的实现类上添加注解信息

  ```java
  @Transactional(propagation = Propagation.REQUIRED,isolation = Isolation.DEFAULT)
  public class AccountServiceImpl implements AccountService {
  
      private AccountDao accountDao;
      // 提供set方法，让spring注入属性
      public void setAccountDao(AccountDao accountDao) {
          this.accountDao = accountDao;
      }
  
      @Override
      public void transfer(String outer, String inner, Integer money) {
          this.accountDao.out(outer,money);
  //        int i = 1/0;
          this.accountDao.in(inner,money);
      }
  }
  ```

  - 测试

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration("classpath:transfer_anno.xml") // 修改为注解配置的xml即可
  public class transferTest {
  
      @Autowired
  //    @Resource(name = "proxyService")
      private AccountService accountService;
  
      // 使用aop配置事务
      @Test
      public void test2(){
          accountService.transfer("jack","rose",100);
  
      }
  }
  ```

  

## 5. 事务管理最佳实践

### 5.1 编程事务管理和声明事务管理的区别

- 编程式事务允许用户在代码中精确定义事务的边界；
- 声明式事务有助于用户将操作与事务规则进行解耦
  - 基于AOP交由Spring容器实现
  - 实现关注点聚焦在业务逻辑上
- 简言
  - 编程式事务侵入到了业务代码里面，但是提供了更加详细的事务管理；而声明式事务由于基于AOP，所以即能起到事务管理的作用，又不影响业务代码的具体实现。



### 5.2 两种事务的选择

- 小型应用、事务操作少
  - 建议编程式事务管理实现：`TransactionTemplate`
  - 简单、显式操作、直观明显、可以设置事务名称
- 大型应用，事务操作量多
  - 业务复杂度高、关联性紧密，建议声明式事务管理实现
  - 关注点聚焦到业务层面，实现业务和事务的解耦。

## 6. Spring事务管理总结

- 事务与Spring事务管理
- Spring事务核心接口类
- 编程式事务实现
- 声明式事务实现
- 事务的综合案例