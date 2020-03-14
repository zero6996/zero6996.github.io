---
title: SpringMVC学习1
date: 2019-9-2 23:59
categories: FrameWork
tags: [SpringMVC]
description: SpringMVC基础入门，做上篇文章补充内容
---



## 1. SpringMVC基本概念

### 1.1 关于三层架构和MVC



<!---more-->



#### 1.1.1 三层架构

- 开发服务器端程序，一般基于两种形式，一种C/S架构程序，一种B/S架构程序
- 使用Java语言基本上都是开发B/S架构的程序，B/S架构又分为了三层架构
- 三层架构
  1. 表现层：WEB层，用来和客户端进行数据交互的。表现层一般会采用MVC的设计模型
  2. 业务层：处理公司具体的业务逻辑
  3. 持久层：用来操作数据库



#### 1.1.2 MVC模型

1. MVC全名是`Model View Controller `模型视图控制器，每个部分各司其职。
2. Model：数据模型，JavaBean的类，用来进行数据封装。
3. View：指JSP、HTML用来展示数据给用户
4. Controller：用来接收用户的请求，整个流程的控制器。用来进行数据校验等。



### 1.2 SpringMVC概述

#### 1.2.1 SpringMVC是什么

1. 是一种基于Java实现的MVC设计模型的请求驱动类型的轻量级Web框架。
2. Spring MVC属于SpringFrameWork的后续产品，已经融合在SpringWebFlow里面。Spring框架提供了构建 Web 应用程序的全功能MVC模块。
3. 使用Spring可插入的MVC架构，从而在使用Spring进行WEB开发时，可以选择使用Spring的SpringMVC框架或集成其他MVC开发框架，如Struts1(现在一般不用)，Struts2等。
4. 它通过一套注解，让一个简单的 Java 类成为处理请求的控制器，而无须实现任何接口。同时它还支持
    RESTful 编程风格的请求。



#### 1.2.2 SpringMVC在三层架构的位置

![三层架构](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/09/06/%E4%B8%89%E5%B1%82%E6%9E%B6%E6%9E%84-1567699806413.jpg)


#### 1.2.3 SpringMVC的优势

- 清晰的角色划分：
  - 前端控制器（DispatcherServlet）
  - 请求到处理器映射（HandlerMapping）
  - 处理器适配器（HandlerAdapter）
  - 视图解析器（ViewResolver）
  - 处理器或页面控制器（Controller）
  - 验证器（ Validator）
  - 命令对象（Command 请求参数绑定到的对象就叫命令对象）
  - 表单对象（Form Object 提供给表单展示和提交到的对象就叫表单对象）。
- 分工明确，而且扩展点相当灵活，可以很容易扩展，虽然几乎不需要。
- 由于命令对象就是一个 POJO，无需继承框架特定 API，可以使用命令对象直接作为业务对象。
- 和 Spring 其他框架无缝集成，是其它 Web 框架所不具备的。
- 可适配，通过 HandlerAdapter 可以支持任意的类作为处理器。
- 可定制性，HandlerMapping、ViewResolver 等能够非常简单的定制。
- 功能强大的数据验证、格式化、绑定机制。
- 利用 Spring 提供的 Mock 对象能够非常简单的进行 Web 层单元测试。
- 本地化、主题的解析的支持，使我们更容易进行国际化和主题的切换。
- 强大的 JSP 标签库，使 JSP 编写更容易。

- 支持RESTful风格的编程支持



#### 1.2.4 SpringMVC和Struts2的优劣分析[了解]

- 共同点：
  - 都是表现层框架，基于MVC模型编写的。
  - 底层都离不开原始ServletAPI。
  - 它们处理请求的机制都是一个核心控制器。
- 区别：
  - Spring MVC 的入口是 Servlet, 而 Struts2 是 Filter。
  - Spring MVC 是基于方法设计的，而 Struts2 是基于类，Struts2 每次执行都会创建一个动作类。所以 Spring MVC 会稍微比 Struts2 快些。
  - Spring MVC 使用更加简洁,同时还支持 JSR303, 处理 ajax 的请求更方便。
  - Struts2 的 OGNL 表达式使页面的开发效率相比 Spring MVC 更高些，但执行效率并没有比 JSTL 提
    升，尤其是 Struts2 的表单标签，远没有 html 执行效率高。

> OGNL是对象 - 图形导航语言的缩写，它是一种功能强大的表达式语言，可以存取对象的任意属性，调用对象的方法，遍历整个对象的结构图，实现字段类型转化等功能。

## 2. SpringMVC入门



### 2.1 SpringMVC入门案例

#### 1. 创建Web工程，引入开发相关的jar包

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <!--版本锁定-->
    <spring.version>5.0.2.RELEASE</spring.version>
</properties>

<dependencies>
    <!--引入spring相关jar包-->
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
</dependencies>
```

#### 2. 配置核心控制器(DispatcherServlet)

- 在web.xml中配置前端控制器

```xml
<!--配置核心前端控制器-->
<servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--加载mvc配置文件-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:SpringMVC.xml</param-value>
    </init-param>
    <!--启动服务器就自动加载配置文件，生成对象-->
    <load-on-startup>1</load-on-startup>
</servlet>
<!--配置控制器映射，拦截所有请求-->
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```



#### 3. 编写SpringMVC.xml的配置文件

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

    <!--开启注解扫描-->
    <context:component-scan base-package="cn.zero.controller"/>

    <!--配置视图解析器对象-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--配置前缀后缀，访问路径等于：/WEB-INF/pages/xxx.jsp-->
        <property name="prefix" value="/WEB-INF/pages/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--开启SpringMVC框架注解的支持-->
    <mvc:annotation-driven/>

</beans>
```

#### 4. 编写index.jsp和HelloController控制器类

- index.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>主页</title>
</head>
<body>

<h3>SpringMVC快速入门</h3>
<a href="/hello">入门程序</a>
</body>
</html>

```

- HelloController

```java
// 控制器类
@Controller
public class HelloController {

    
    @RequestMapping("/hello")
    public String sayHello(){
        System.out.println("Hello SpringMVC!");
        return "success";
    }
}
```

#### 5. 在WEB-INF下创建pages文件夹，编写success.jsp的成功页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>访问成功</title>
</head>
<body>
<h3>入门程序访问成功</h3>
</body>
</html>
```

#### 6. 启动服务器

访问localhost/index.jsp，点击入门程序，查看是否正常跳转。

### 2.2 入门案例的执行过程及原理分析

#### 2.2.1 入门案例的执行流程

1. 当启动Tomcat服务器的时候，因为配置了load-on-startup标签，所以会创建DispatcherServlet对象，
就会加载springmvc.xml配置文件
2. 开启了注解扫描，那么HelloController对象就会被创建
3. 从index.jsp发送请求，请求会先到达DispatcherServlet核心控制器，根据配置@RequestMapping注解
    找到执行的具体方法
4. 根据执行方法的返回值，再根据配置的视图解析器，去指定的目录下查找指定名称的JSP文件
5. Tomcat服务器渲染页面，做出响应
6. 如下图示流程图：

![flowChart](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/09/06/flowChart-1567699840232.jpg)


#### 2.2.2 SpringMVC的请求响应流程

- 官方完整流程图：

![mvc](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/29/mvc-1567090970613.jpg)




### 2.3 组件详解

- 前端控制器(DispatcherServlet)
  - 用户请求到达前端控制器，它相当于mvc模式中的c。DispatcherServlet 是**整个流程控制的中心**，由它调用其它组件处理用户的请求，DispatcherServlet 的存在降低了组件之间的**耦合性**。
- 处理器映射器(HandlerMapping)
  - HandlerMapping 负责根据用户请求找到 Handler 即处理器，SpringMVC 提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。
- 处理器(Handler)
  - 它就是我们开发中要编写的具体**业务控制器**。由 DispatcherServlet 把用户请求转发到 Handler。由Handler 对具体的用户请求进行处理。
- 处理器适配器(HandlerAdapter)
  - 通过 HandlerAdapter 对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。
- 视图解析器(ViewResolver)
  - View Resolver 负责将处理结果生成 View 视图，View Resolver首先根据逻辑视图名解析成物理视图名
    即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。
- 视图(View)
  - SpringMVC 框架提供了很多的 View 视图类型的支持，包括：jstlView、freemarkerView、pdfView
    等。我们最常用的视图就是 jsp。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。

#### 2.3.1 `<mvc:annotation-driven/>`

在SpringMVC 的各个组件中，处理器映射器、处理器适配器、视图解析器称为SpringMVC的三大组件。

使用自动加载 RequestMappingHandlerMapping(处理映射器)和RequestMappingHandlerAdapter(处 理 适 配 器)， 可用在SpringMVC.xml配置文件中使用替代注解处理器和适配器的配置。



### 2.4 @RequestMapping注解详解

1. RequestMapping注解的作用是**建立请求URL和处理方法之间的对应关系**
2. RequestMapping注解可以作用在方法和类上
   1. 作用在类上：第一级的访问目录
   2. 作用在方法上：第二级的访问目录
   3. 细节：前端路径可以不写 `/` 表示应用的根目录开始
   4. 细节：`${pageContext.request.contextPath}`也可以省略不写，但是路径上不能写 /
3. RequestMapping的属性
   1. path：指定请求路径的url
   2. value：value属性等同于path属性，指定请求路径url
   3. mthod：指定该方法的请求方式
   4. params：指定限制请求参数的条件，允许简单的表达式，例`params={“money!100”}`表示请求参数中money不能是100。
   5. headers：发送的请求中必须包含的请求头

> 以上属性出现两个以上时，他们的关系是&&逻辑与关系

## 3. 请求参数的绑定[重点]

### 3.1 绑定说明

#### 3.1.1 绑定的机制

- 表单提交的数据都是key=value格式的，例：username=xiaoming&password=123456
- SpringMVC的参数绑定过程是把表单提交的请求参数，作为控制器中方法的参数进行绑定的
- 要求：提交表单的name和参数的名称必须是相同的



#### 3.1.2 支持的数据类型

- 基本类型参数：包括基本类型和String类型
- POJO类型参数：包括实体类，以及关联的实体类
- 数组和集合类型参数：包括List结构和Map结构的集合(包括数组)



#### 3.1.3 使用要求

- 如果是基本类型或String类型
  - 要求参数名称必须和控制器中方法的形参名称保持一致。(严格区分大小写)
- 如果是 POJO 类型，或者它的关联对象
  - 要求表单中参数名称和 POJO 类的属性名称保持一致。并且控制器方法的参数类型是 POJO 类型。
- 如果是集合类型,有两种方式：
  - 要求集合类型的请求参数必须在 POJO 中。在表单中请求参数名称要和 POJO 中集合属性名称相同。给 List 集合中的元素赋值，使用下标。给 Map 集合中的元素赋值，使用键值对。
  - 接收的请求参数是 json 格式数据。需要借助一个注解实现。



#### 3.1.4 绑定参数示例

##### 1. 基本类型和String类型作为参数

- 控制器代码

```java
@RequestMapping("/testParam")
public String testParam(String username){
    System.out.println("用户名："+username);
    return "success";
}
```

- jsp代码

```jsp
<a href="param/testParam?username=xiaoming">请求参数绑定</a>
```



##### 2. POJO类型作为参数

- 需创建实体类Account和User，User类作为账户类的参数。

```java
public class Account implements Serializable {
    private String username;
    private String password;
    private Double money;
    private User user;
}
public class User {
    private String name;
    private Integer age;
}
```

- 控制器代码

```java
/**
     * 接收参数，封装为POJO类型
     * @param account
     * @return
     */
@RequestMapping(value = "/testPojo",method = RequestMethod.POST)
public String testPojo(Account account){
    System.out.println(account);
    return "success";
}
```

- Jsp

```jsp
<form action="/param/testPojo" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    余额：<input type="text" name="money"><br>
    真实姓名：<input type="text" name="user.name"><br>
    年龄：<input type="text" name="user.age"><br>
    <input type="submit" value="提交">
</form>
```

- 访问结果：`Account{username='admin', password='123', money=333.0, user=User{name='xiaoming', age=22}}`



##### 3. POJO类中包含集合类型参数

- 修改Account类

```java
public class Account implements Serializable {
    private String username;
    private String password;
    private Double money;

    // 演示绑定POJO对象
//    private User user;

    // POJO类中包含集合类型参数
    private List<User> list;
    private Map<String,User> map;
}
```

- 控制器添加方法

```java
/**
     * POJO类中包含集合类型参数
     */
@RequestMapping(value = "/testSetType",method = RequestMethod.POST)
public String testSetType(Account account){
    System.out.println(account);
    return "success";
}
```

- Jsp

```jsp
<h3>POJO类中包含集合类型参数</h3>
<form action="/param/testSetType" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    余额：<input type="text" name="money"><br>
    <h4>封装到list集合中</h4>
    真实姓名：<input type="text" name="list[0].name"><br>
    年龄：<input type="text" name="list[0].age"><br>
    <h4>封装到map集合中</h4>
    真实姓名：<input type="text" name="map['user'].name"><br>
    年龄：<input type="text" name="map['user'].age"><br>
    <input type="submit" value="提交">
</form>
```

- 访问结果：`Account{username='admin', password='111', money=333.0, list=[User{name='小明', age=22}], map={user=User{name='小李', age=21}}}`



##### 4. POST请求参数中文乱码问题

在web.xml中，添加过滤器

```xml
<!--添加过滤器，解决POST中文乱码问题-->
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
```



### 3.2 特殊情况

如果User类中有一个Date参数，在表单提交时日期格式不正确，封装POJO对象就会失败，这时就需要自己定义一个类型转换器。 

#### 3.2.1 自定义类型转换器

表单提交的任何数据类型全部都是字符串类型，但是后台定义Integer类型，数据也可以封装上，说明Spring框架内部会默认进行数据类型转换。如果想自定义数据类型转换，可以实现Converter的接口

##### 1. 第一步：定义一个类，实现Converter接口，该接口有两个泛型。

```java
/**
 * 将字符串类型转换为日期格式
 */
public class StringToDateConverter implements Converter<String, Date> {

    /**
     * @param source 传入的字符串参数
     * @return
     */
    @Override
    public Date convert(String source) {
        if (source == null){
            throw new RuntimeException("please entry param！");
        }
        DateFormat df = new SimpleDateFormat("yyyy-MM-dd");
        // 把字符串转换为日期
        try {
            return df.parse(source);
        } catch (Exception e) {
            throw new RuntimeException("Data switch Error！");
        }
    }
}
```

##### 2. 在SpringMVC.xml中配置类型转换器

```xml
<!--配置自定义类型转换器-->
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
    <!--注册转换器类-->
    <property name="converters">
        <set>
            <bean class="cn.zero.utils.StringToDateConverter"/>
        </set>
    </property>
</bean>

<!--开启SpringMVC框架注解的支持
	配置一下让转换器生效-->
<mvc:annotation-driven conversion-service="conversionService"/>
```

- 重启服务器在测试一下，就支持`yyyy-MM-dd`的日期格式了。



#### 3.2.2 使用ServletAPI对象作为方法参数

- 示例获取Servlet对象

  - 在控制器中新增方法

  ```java
  /**
       * 测试获取ServetlAPI
       * @param request
       * @param response
       * @param session
       * @return
       */
  @RequestMapping("/testServletAPI")
  public String testServletAPI(HttpServletRequest request,
                               HttpServletResponse response,
                               HttpSession session){
      System.out.println(request);
      System.out.println(response);
      System.out.println(session);
      return "success";
  }
  ```

  - jsp代码：`<a href="/param/testServletAPI">测试访问ServletAPI</a>`

## 4. 常用注解

### 4.1 @RequestParam

#### 1. 使用说明

- 作用：把请求中指定名称的参数给控制器中的形参赋值
- 属性：
  - value：请求参数中的名称
  - required：请求参数中是否必须提供此参数。默认为true，表示必须提供，不提供将报错。

#### 2. 示例

- 控制器代码

```java
@Controller
@RequestMapping("anno")
public class AnnotationController {

    @RequestMapping("/useRequestParam")
    public String useRequestParam(@RequestParam("name") String username,
                                  @RequestParam(value = "age",required = false) Integer age){
        System.out.println(username+":"+age);
        return "success";
    }
}
```

- jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>常用注解测试</title>
</head>
<body>
<h2>1. 测试requestParam注解的使用</h2>
<a href="/anno/useRequestParam?name=xiaoming">@RequestParam注解</a>
</body>
</html>
```

- 结果：`xiaoming:null`，因配置了required=false，故请求参数中无age不会报错。

### 4.2 @RequestBody

#### 1. 使用说明

- 作用：用于获取请求体内容。直接使用得到是key=value&key=value结构的数据。get请求方式不适用。
- 属性：
  - required：是否必须有请求体。默认值true，表示必须有请求体，故get请求方式会报错。如取值为false，get请求得到的是null。

#### 2. 示例

- 控制器代码

```java
/**
     * RequestBody:用于获取请求体内容。
     * @param body
     * @return
     */
@RequestMapping("/useRequestBody")
public String useRequestBody(@RequestBody(required = false) String body){
    System.out.println(body);
    return "success";
}
```

- jsp

```jsp
<h3>2. 测试@RequestBody注解使用</h3>
<h5>POST请求测试</h5>
<form action="/anno/useRequestBody" method="post">
    用户名：<input type="text" name="name"><br>
    年龄：<input type="text" name="age"><br>
    <input type="submit" value="提交">
</form>
<h5>GET请求测试</h5>
<a href="/anno/useRequestBody?body=test">requestBody注解get请求</a>
```

- 测试结果：
  - POST请求：`name=xiaoming&age=1`
  - GET请求：`null`



### 4.3 @ResponseBody

#### 1. 使用说明

该注解的作用是将控制器方法返回的对象通过适当的转换器转换为指定格式之后，写入到response对象的body区，通常用来**返回JSON数据**或者是XML数据。

- 注意：使用此注解后不会在走视图解析器，而是直接将数据写入到输出流中，效果等同于通过response对象输出指定格式数据(`response.getWriter.write(JSONObject.fromObject(user).toString());`)。

#### 2. 示例

- 控制器代码

```java
@RequestMapping("/testResponseBody")
@ResponseBody
public Student testResponseBody(){
    return new Student("小明","男");
}
```

- 访问页面结果：`{"name":"小明","sex":"男"}`
- 因返回值满足key-value(对象或map)格式，所以会自动将响应头的Content-Type设置为了`application/html;charset=utf-8`，然后把转换后的内容以输出流的形式响应给客户端。

> 注意：使用该注解返回json数据还需额外jar包：jackson，依赖坐标如下
>
> ```xml
> <!--添加对json数据的支持 -->
> <dependency>
>  <groupId>com.fasterxml.jackson.core</groupId>
>  <artifactId>jackson-databind</artifactId>
>  <version>2.8.3</version>
> </dependency>
> <dependency>
>  <groupId>com.fasterxml.jackson.core</groupId>
>  <artifactId>jackson-core</artifactId>
>  <version>2.9.1</version>
> </dependency>
> ```

- 如果返回是字符串类型，控制器代码如下

```java
@RequestMapping(value = "/testResponseBody",produces = "text/html;charset=utf-8") 
@ResponseBody
public String testResponseBody(){
    return "你好!";
}
```

- 访问结果：`你好!`
- 如果返回值不能解析为json格式，注解就会将其直接以输出流形式输出到页面上。

> 注：使用produces设置响应头Content-Type为`text/html;charset=utf-8`解决中文乱码问题。



### 4.4 @PathVaribale

#### 1. 使用说明

路径变量，用于绑定 url 中的占位符。例如：请求url中/delete/{id}，这个{id}就是url占位符。url 支持占位符是 Spring3.0 之后加入的，是 SpringMVC支持**REST风格**URL的一个重要标志。

- 属性
  - value：用于指定url中占位符名称。
  - required：是否必须提供占位符。

#### 2. 示例

- 控制器代码

```java
/**
     * PathVaribale：用于绑定 url 中的占位符
     */
@RequestMapping("/usePathVariable/{id}")
public String usePathVariable(@PathVariable("id") Integer id){
    System.out.println(id);
    return "success";
}
```

- jsp

```jsp
<h3>3. 测试@PathVaribale注解</h3>
<a href="/anno/usePathVariable/100">PathVariable注解的使用</a>
```

- 测试结构：`100`



#### 3. 关于REST风格URL

- 什么是REST：
  - **表现层状态转换**（REST，英文：**Representational State Transfer**），一种万维网软件架构风格，目的是便于不同软件/程序在网络中互相传递信息。它是一种针对网络应用的设计和开发方式，可以降低开发的复杂性，提高系统的可伸缩性。

[详细内容见文章](https://www.jianshu.com/p/c5c83872dad2)

- RESTful的优点：结构清晰、符合标准、易于理解、扩展方便。RESTful可以通过一套统一的接口为 Web、iOS和Android提供服务，另外对于很多平台来说（比如像Facebook，Twiter、微博、微信等开放平台），它们不需要有显式的前端，只需要一套提供服务的接口，于是RESTful便是它们最好的选择。
- RESTful的特性
  - **资源(Resources)**：网络上的一个实体，或说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的存在。通过一个URI来指向它。
  - **表现层(Representation)**：把资源具体呈现出来的形式，叫做它的表现层。比如，文本可以用 txt 格式表现，也可以用 HTML 格式、XML 格式、JSON 格式表现，甚至可以采用二进制格式。
  - **状态转化(Status Transfer)**：每发出一个请求，就代表了客户端和服务器的一次交互过程。HTTP协议，是一个无状态协议，即所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生“状态转化”。而这种转化是建立在表现层上的。所以就是“表现层状态转化”。具体说，就是HTTP协议里面，四个表示操作方式的动词以及对应的基本操作：GET(获取资源)、POST(新建资源)、PUT(更新资源)、DELETE(删除资源)。



#### 4. 基于HiddentHttpMethodFilter的REST风格示例[了解]

- 作用：

  - 由于浏览器form表单只支持GET和POST请求，而DELETE、PUT等method并不支持，Spring3.0添加了一个过滤器，可以将浏览器请求改为指定的请求方式，发送给我们的控制器方法，使得支持DELETE和PUT请求。

- 使用步骤：

  - 第一步：在web.xml中配置该过滤器
  - 第二步：请求方式必须使用POST请求
  - 第三部：按照要求提供_method请求参数，该参数的取值就是我们需要的请求方式。

- 示例代码

  - jsp

  ```jsp
  <!-- 更新 -->
  <form action="springmvc/testRestPUT/1" method="post">
      用户名称：<input type="text" name="username"><br/>
      <input type="hidden" name="_method" value="PUT">
      <input type="submit" value="更新">
  </form>
  ```

  - 控制器

  ```java
  /**
  * put 请求：更新
  * @param username
  * @return
  */
  @RequestMapping(value="/testRestPUT/{id}",method=RequestMethod.PUT)
  public String testRestfulURLPUT(@PathVariable("id")Integer id,User user){
      System.out.println("rest put "+id+","+user);
      return "success";
  }
  
  ```



### 4.5 @RequestHeader

#### 1. 使用说明

- 作用：用于获取请求消息头
- 属性
  - value：提供消息头名称
  - required：是否必须有此消息头

> 了解即可，一般不用

#### 2. 示例

- 控制器代码

```java
/**
     * RequestHeader：用于获取请求消息头。
     */
@RequestMapping("/useRequestHeader")
public String useRequestHeader(@RequestHeader(value = "Accept-Language",required = false)String requestHeader){
    System.out.println(requestHeader);
    return "success";
}
```

- jsp

```jsp
<h3>4. 测试@RequestHeader注解</h3>
<a href="/anno/useRequestHeader">获取请求消息头</a>
```

- 结果：`zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7`

### 4.6 @CookieValue

#### 1. 使用说明

- 作用：用于把指定cookie名称的值传入控制器方法参数
- 属性：
  - value：指定cookie的名称
  - required：是否必须有此cookie

#### 2. 示例

- 控制器代码

```java
/**
     * @CookieValue：用于把指定cookie名称的值传入控制器方法参数。
     * @param cookieValue
     * @return
     */
@RequestMapping("/useCookieValue")
public String useCookieValue(@CookieValue(value = "JSESSIONID",required = false)String cookieValue){
    System.out.println(cookieValue);
    return "success";
}
```

- jsp

```jsp
<h3>5. 测试@CookieValue</h3>
<a href="/anno/useCookieValue">绑定cookie的值</a>
```

- 结果：`C8001453C280DF881950883018287F9C`

### 4.7 @ModelAttribute

#### 1. 使用说明

- 作用：该注解是SpringMVC4.3版本以后新加入的。它可以用于修饰方法和参数。
  - **出现在方法上**，表示当前方法会在控制器的方法执行之前，先执行。它可以修饰有具体返回值的方法，也可以修饰没有返回值的方法。
  - **出现在参数上**，获取指定的数据给参数赋值。
- 属性：
  - value：用于获取数据的key。key可以是POJO的属性名称，也可以是map结构的key。
- 应用场景
  - 当表单提交数据不是完整的实体类数据时，保证没有提交数据的字段使用数据库对象原来的数据。

#### 2. 示例

- 基于POJO属性的基本使用

  - 控制器代码

  ```java
  /**
       * 出现在方法上：
       *    被ModelAttribute 修饰的方法
       *    当前方法会在控制器的方法执行之前，先执行。
       * @param user
       */
  @ModelAttribute
  public void showModel(User user){
      System.out.println("执行了showModel方法"+user.getName());
  }
  
  /**
       * 接收请求的方法
       * @param user
       * @return
       */
  @RequestMapping("/testModelAttribute")
  public String testModelAttribute(User user) {
      System.out.println("执行了控制器的方法"+user.getName());
      return "success";
  }
  ```

  - jsp

  ```jsp
  <h3>6. 测试@ModelAttribute</h3>
  <a href="/anno/testModelAttribute?name=test">测试ModelAttribute注解的使用</a>
  ```

  - 测试结果：
  ```
  执行了showModel方法test
  执行了控制器的方法test
  ```

- 基于Map的应用场景示例：修饰方法带返回值

  - 控制器代码

  ```java
  /**
       * 出现在方法上：
       *    被ModelAttribute 修饰的方法
       *    当前方法会在控制器的方法执行之前，先执行。
       *    在控制器执行前，查询数据库中用户信息
       * @param name
       */
  @ModelAttribute
  public User showModel(String name){
      // 调用数据库查询方法，返回用户对象
      User user = findUserByName(name);
      System.out.println("执行了findUser方法"+user);
      return user;
  }
  
  /**
       * 模拟修改用户方法
       * @param user
       * @return
       */
  @RequestMapping("/updateUser")
  public String updateUser(User user) {
      System.out.println("控制器中处理请求的方法：修改用户"+user);
      return "success";
  }
  
  /**
       * 模拟数据库查询操作
       * @param name
       * @return
       */
  private User findUserByName(String name) {
      User user = new User();
      user.setName(name);
      user.setAge(19);
      user.setDate(new Date());
      return user;
  }
  ```

  - jsp

  ```jsp
  <h4>6.1 2 基于Map应用场景示例：修饰方法带返回值</h4>
  <form action="/anno/updateUser" method="post">
      用户名称：<input type="text" name="name"><br>
      年龄：<input type="text" name="age"><br>
      <input type="submit" value="保存">
  </form>
  ```

  - 测试结果：

  ```
  执行了findUser方法User{name='小王', age=19, date=Sun Sep 01 17:02:46 CST 2019}
  控制器中处理请求的方法：修改用户User{name='小王', age=11, date=Sun Sep 01 17:02:46 CST 2019}
  ```

- 基于Map的应用场景示例：修饰方法不带返回值

  - 前端代码不变，修改控制器方法

  ```java
  @ModelAttribute
  public void showModel(String name, Map<String,User> map){
      // 调用数据库查询方法，返回用户对象
      User user = findUserByName(name);
      System.out.println("showModel"+System.identityHashCode(user));
      System.out.println("执行了findUser方法"+user);
      map.put("user",user);
  }
  
  @RequestMapping("/updateUser")
      public String updateUser(@ModelAttribute("user") User user) {
          System.out.println("update"+System.identityHashCode(user));
          System.out.println("控制器中处理请求的方法：修改用户"+user);
          return "success";
      }
  ```

  

### 4.8 @SessionAttribute

#### 1. 使用说明

- 作用：用于多次执行控制器方法间的参数共享
- 属性
  - value：用于指定存入的属性名称
  - type：用于指定存入的数据类型

#### 2. 示例

##### 2.1 存储值

- 控制器代码

```java
@Controller
@RequestMapping("anno")
@SessionAttributes(value = {"msg"}) // @SessionAttributes该注解只能作用在类上，这里使用效果是将msg再存一份到session域中
public class AnnotationController {
    
	/**
     * SessionAttribute注解
     * 将值存储到request域中
     * @return
     */
    @RequestMapping("/useSessionAttribute")
    public String useSessionAttribute(Model model) {
        // 底层会存储到request域对象中
        model.addAttribute("msg","小明");
        return "success";
    }
}
```

- Anno.jsp

```jsp
<h3>7. 测试@SessionAttribute</h3>
<a href="/anno/useSessionAttribute">使用SessionAttribute存储值</a><br>
```

- success.jsp

```jsp
<body>
<h3>测试程序访问成功</h3>
存储到request域中：${msg}
<br>
存储到session域中：${sessionScope}
</body>
```

- 测试结果：

```
测试程序访问成功
存储到request域中：小明 
存储到session域中：{msg=小明}
```

##### 2.2 获取值

- 控制器

```java
/**
     * 获取值
     * @param map
     * @return
     */
@RequestMapping("/getSessionAttribute")
public String getSessionAttribute(ModelMap map) { // model无法获取，只能通过它的实现类ModelMap来获取
    System.out.println("getSessionAttribute....");
    // 获取存储在request域中的数据
    String msg = (String) map.get("msg");
    System.out.println(msg);
    return "success";
}
```

- jsp

```jsp
<a href="/anno/getSessionAttribute">获取值</a><br>
```

- 测试结果，后台显示：

```
getSessionAttribute....
小明
```

##### 2.3 清除值

- 控制器

```java
	/**
     * 删除值
     * @param status
     * @return
     */
@RequestMapping("/delSessionAttribute")
public String delSessionAttribute(SessionStatus status) {
    System.out.println("delSessionAttribute....");
    status.setComplete(); // 删除session域中存储的值
    return "success";
}
```

- jsp

```jsp
<a href="/anno/delSessionAttribute">删除值</a>
```

- 测试结果，后台显示：`delSessionAttribute....`



