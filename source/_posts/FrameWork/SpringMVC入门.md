---
title: SpringMVC入门
date: 2019-8-29 23:10
categories: SpringMVC
tags: [SpringMVC]
description: SpringMVC基础入门
---



## 1. SpringMVC 入门

### 1.1 简介

Spring MVC是Spring提供的一个强大而灵活的web框架，目前最好的实现MVC设计模式的框架。借助于注解，Spring MVC提供了几乎是POJO的开发模式，使得控制器的开发和测试更加简单。这些控制器一般不直接处理请求，而是将其委托给Spring上下文中的其他bean，提供Spring的依赖注入功能，这些bean被注入到控制器中。



<!--more-->



Spring MVC主要由DispatcherServlet、处理器映射、适配器、控制器、视图解析器、视图组成。



### 1.2 MVC设计模式

 MVC是一种使用 MVC（Model View Controller 模型-视图-控制器）设计创建 Web 应用程序的模式。

- Controller：负责接收并处理用户请求，响应客户端。
- Model：模型数据，业务逻辑。
- View：展示模型，与用户进行交互。



### 1.3 SpringMVC核心组件

1. DispatcherServlet：前置控制器。
2. HandlerMapping：将请求映射到Handler。
3. Handler：后端控制器，完成具体业务逻辑。
4. HandlerInterceptor：处理器拦截器。
5. HandlerExecutionChain：处理器执行链。
6. HandlerAdapter：处理器适配器。
7. ModelAndView：装载模型数据和视图信息。
8. ViewResolver：视图解析器。



### 1.4 SpringMVC实现流程

1. 客户端请求被DispatcherServlet接收。
2. DispatcherServlet将请求映射到Handler。
3. 生成Handler以及HandlerInterceptor。
4. 返回HandlerExecutionChain(Handler+HandlerInterceptor)。
5. DispatcherServlet通过HandlerAdapter执行Handler。
6. 返回一个ModelAndView。
7. DispatcherServlet通过ViewResolver进行解析。
8. 返回填充了模型数据的View，响应给客户端。



![mvc](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/08/29/mvc-1567090970613.jpg)


## 2. SpringMVC快速开发

大部分组件由框架提供，开发者只需通过配置进行关联。开发者只需手动编写Handler和View。



### 2.1 基于XML配置的使用

1. SpringMVC基础配置
2. XML配置Controller，HandlerMapping组件映射。
3. XML配置ViewResolver组件映射。



#### 2.1.1 示例

- 使用maven创建一个简单的web项目，修改pom包导入相关jar包

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <!--导入SpringMVC所需jar包-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>4.3.1.RELEASE</version>
    </dependency>
    <!--导入servlet相关jar包-->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
    </dependency>
</dependencies>
```

- 创建MyHandler控制器，完成具体业务逻辑

```java
public class MyHandler implements Controller {
    @Override
    public ModelAndView handleRequest(javax.servlet.http.HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
        // 装载模型数据和逻辑视图
        ModelAndView modelAndView = new ModelAndView();
        // 添加模型数据
        modelAndView.addObject("name","Tom");
        // 添加逻辑视图
        modelAndView.setViewName("show");

        return modelAndView;
    }
}
```

- 创建show.jsp，做前端显示

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@page isELIgnored="false" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    ${name}
</body>
</html>
```

- 资源文件夹resources下创建springmvc.xml的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
    <!--
		URL处理映射的方式有三种：
		1. BeanNameUrlHandlerMapping：通过url名字，找到对应的bean的name的控制器
		2. 在SimpleUrlHandlerMapping中配置：通过key找到bean
		3. ControllerClassNameHandlerMapping：通过类名
	-->
    <!--1. 配置HandlerMapping，将url请求映射到Handler-->
    <bean id="handlerMapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
        <!--配置简单url映射-->
        <property name="mappings">
            <props>
                <!--2. 配置test请求对应的handler-->
                <prop key="/test">testHandler</prop>
            </props>
        </property>
    </bean>

    <!--3. 配置控制器-->
    <bean id="testHandler" class="com.zero.handler.MyHandler"/>

    <!--4. 配置视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--配置前缀-->
        <property name="prefix" value="/"/>
        <!--配置后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

- 在web.xml下配置DispatcherServlet

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

  <!-- 配置：所有请求由SpringMVC管理 -->
  <servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>*.do</url-pattern>
  </servlet-mapping>
</web-app>
```

- 启动服务器，访问/test.do路径资源，查看是否显示姓名。



### 2.2 基于注解方式的使用[重要]

1. SpringMVC基础配置。
2. Controller，HandlerMapping通过**注解进行映射。**
3. XML中配置ViewResolver组件映射。



#### 2.2.1 示例

- 其余配置不变，新建一个控制器类，使用注解声明

```java
// 基于注解实现映射
@Controller
public class AnnotationHandler {

    /**
     * 业务方法1：ModelAndView完成数据的传递，视图的解析
     */
    @RequestMapping("/modelAndViewTest")
    public ModelAndView modelAndViewTest(){
        // 创建ModelAndView对象
        ModelAndView modelAndView = new ModelAndView();
        // 填充模型数据
        modelAndView.addObject("name","Tom");
        // 设置逻辑视图
        modelAndView.setViewName("show");
        return modelAndView;
    }

    /**
     * 业务方法2：Model传值，String进行视图解析
     * @return
     */
    @RequestMapping("/modelTest")
    public String ModelTest(Model model){
        // 填充模型数据
        model.addAttribute("name","Jerry");
        // 设置逻辑视图
        return "show";
    }

    /**
     * 业务方法3：Map传值，String进行视图解析
     */
    @RequestMapping("/mapTest")
    public String MapTest(Map<String,String> map){
        // 填充模型数据
        map.put("name","Cat");
        // 设置逻辑视图
        return "show";
    }
}
```

- mvc配置文件中，开启自动扫描，并删除前面配置的两个bean对象即可

```xml
<!--1. 配置自动扫描包-->
<context:component-scan base-package="com.zero.backoffice.web.controller"/>
```

- 重启服务器，访问`/modelAndViewTest`等资源路径，测试是否成功。



### 2.3 URL处理器映射[了解]

- BeanNameUrlHandlerMapping

  - 功能：寻找controller，根据url请求去匹配bean的name属性，找到对应的控制器。

  ```xml
  <!-- 1. 配置处理器映射，mvc默认的处理器映射器
       BeanNameUrlHandlerMapping:根据bean的name属性的url去找Controller -->
      <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
  
  <bean name="/user.do" class="com.zero.backoffice.web.controller.UserController"/>
  ```

- SimpleUrlHandlerMapping

  - 功能：寻找controller，根据浏览器url匹配简单url的key，key就是通过controller的id找到对应的Controller

  ```xml
  <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter">
      <property name="mappings">
      	<props>
          	<prop key="/user1.do">userController</prop>
          </props>
      </property>
  </bean>
  <bean id="userController" name="/user.do" class="com.zero.web.controller.UserController"/>
  ```

- ControllerClassNameHandlerMapping

  - 功能：寻找controller，根据类名.do来访问，类名首字母小写。如下访问路径就是`userController.do`

  ```xml
  <bean class="org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping"/>
  <bean class="com.zero.backoffice.web.controller.UserController"/>
  ```

  

### 2.4 处理器适配器

- SimpleControllerHandlerAdapter

  - 功能：执行controller，调用controller里面的方法，返回ModelAndView

  ```xml
  <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping"/>
  ```

- HttpRequestHandlerAdapter

  - 执行控制器：负责调用实现HttpRequestHandler接口的控制器




### 2.5 乱码问题

- POST请求乱码，需在web.xml中配置编码过滤器

```xml
<!-- 配置编码过滤器  -->
<filter>
    <filter-name>EncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>EncodingFilter</filter-name>
</filter-mapping>
```

- GET请求乱码：Tomcat8默认进行了url编码，故get请求不会乱码，tomcat7会乱码。



## 3. SpringMVC接收请求参数

### 3.1 封装参数分析

- 常用的数据绑定类型
  - 基本数据类型
  - 包装类
  - 数组
  - 对象(POJO)
  - 集合(List、Set、Map)
  - JSON

### 3.2 接收基本数据类型

- 编写一个注册类register.jsp

```jsp
<body>
    <form action="${pageContext.request.contextPath}/user/register.do" method="post">
        用户名:<input type="text" name="username"><br>
        密码:<input type="text" name="password"><br>
        性别:<input type="text" name="gender"><br>
        年龄:<input type="text" name="age"><br>
        生日:<input type="text" name="birthday"><br>
        爱好:<input type="checkbox" name="hobbyIds" value="1">打球
             <input type="checkbox" name="hobbyIds" value="2">看书
             <input type="checkbox" name="hobbyIds" value="3">玩游戏<br>
        <input type="submit">
    </form>
</body>
```

- 控制器中新增注册方法

```java
@Controller
@RequestMapping("user")
public class UserController{
    @RequestMapping("/toRegister")
    public String toRegister(){
        return "user/register";
    }
    /**
     * 第一种接收表单参数的方式：
     *      直接在形参列表写要接受的参数
     *      默认日期格式： MM/DD/YYYY
     * @return
     */
    @RequestMapping("/register")
    public String register(String username, String password,
                           int age, String gender, Date birthday,
                           String[] hobbyIds){
        System.out.println(username);
        return "user/info";
    }
}
```

### 3.3 接收POJO类型

- 模型类User新增对应属性

```java
public class User {
    private String username;
    private String password;
    private String gender;
    private int age;
    private String birthday;
    private String[] hobbyIds;
}
```

- 控制器中新增方法

```java
/**
     * 第二种接收表单参数的方式：
     *     使用POJO对象接收
     * @return
     */
@RequestMapping("/register2")
public String register2(User user){
    System.out.println(user);
    return "user/info";
}
```

- register.jsp中修改`<form action="${pageContext.request.contextPath}/user/register.do" method="post">`

- info.jsp做数据展示

```jsp
<body>
<h4>用户信息</h4>
用户名:${user.username}<br>
密码:${user.password}<br>
性别:${user.gender}<br>
年龄:${user.age}<br>
生日:${user.birthday}<br>
爱好:${user.hobbyIds}<br>
</body>
```

### 3.4 接收包装类型参数

- 创建UserExt类，把User写成一个类的属性，模型里面有模型

```java
public class UserExt {
    private User user;

    @Override
    public String toString() {
        return "UserExt{" +
                "user=" + user +
                '}';
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}

```

- 控制器添加注册方法

```java
/**
     * 第三种接收表单参数的方式：接收包装类型参数
     *     使用包装类，相当于模型里面有模型
     * @return
     */
@RequestMapping("/register3")
public String register3(UserExt user){
    System.out.println(user);
    return "user/info";
}
```

- 修改form表单

```jsp
<body>
    <form action="${pageContext.request.contextPath}/user/register3.do" method="post">
        用户名:<input type="text" name="user.username"><br>
        密码:<input type="text" name="user.password"><br>
        性别:<input type="text" name="user.gender"><br>
        年龄:<input type="text" name="user.age"><br>
        生日:<input type="text" name="user.birthday"><br>
        爱好:<input type="checkbox" name="user.hobbyIds" value="1">打球
             <input type="checkbox" name="user.hobbyIds" value="2">看书
             <input type="checkbox" name="user.hobbyIds" value="3">玩游戏<br>
        <input type="submit">
    </form>
</body>
```

### 3.5 接收List集合类型参数

- 修改UserExt类，添加list集合属性

```java
public class UserExt {
    private User user;

    private List<User> users = new ArrayList<>();

    @Override
    public String toString() {
        return "UserExt{" +
                "user=" + user +
                ", users=" + users +
                '}';
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public List<User> getUsers() {
        return users;
    }

    public void setUsers(List<User> users) {
        this.users = users;
    }
}
```

- register.jsp中新增一个测试form表单

```jsp
<h4>接收集合类型的参数</h4>
<form action="${pageContext.request.contextPath}/user/register4.do" method="post">
    用户名1:<input type="text" name="users[0].username"><br>
    密码1:<input type="text" name="users[0].password"><br>
    <hr>
    用户名2:<input type="text" name="users[1].username"><br>
    密码2:<input type="text" name="users[1].password"><br>
    <input type="submit" value="保存">
</form>
```

- 添加注册方法

```java
/**
     * 第四种接收表单参数的方式：接收List集合类型参数
     * @return
     */
@RequestMapping("/register4")
public String register4(UserExt user){
    System.out.println(user.getUsers());
    return "user/info";
}
```



### 3.6 接收Map集合类型参数

- UserExt添加一个map集合属性

```java
public class UserExt {
    private User user;

    private List<User> users = new ArrayList<>();

    private Map<String,Object> infos = new HashMap<String,Object>(); // 使用map来接收参数
}
```

- 添加注册方法

```java
/**
     * 第五种接收表单参数的方式：接收Map集合类型参数
     * @return
     */
@RequestMapping("/register5")
public String register5(UserExt user){
    System.out.println(user.getInfos());
    return "user/info";
}
```

- 修改表单

```jsp
<h4>
    表单数据用map接收
</h4>
<form action="${pageContext.request.contextPath}/user/register5.do" method="post">
        用户名:<input type="text" name="infos['username']"><br>
        密码:<input type="text" name="infos['password']"><br>
        性别:<input type="text" name="infos['gender']"><br>
        年龄:<input type="text" name="infos['age']"><br>
        <input type="submit">
</form>
```



## 4. 页面回显

- 控制器配置一个列表方法

```java
@RequestMapping("/list")
    public String userlist(Model model){
        // 1. 模拟一个数据库中的数据
        List<User> users = new ArrayList<>();
        users.add(new User("小明","male",22,"2018"));
        users.add(new User("小李","male",12,"2019"));
        users.add(new User("小哈","male",32,"2014"));
        // 将数据存储到model中
        model.addAttribute("userList",users);
        return "user/userList";
    }
```

- 前端jsp展示数据

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
<head>
    <title>UserList</title>
</head>
<body>
用户列表:
<table border="1">
    <tr>
        <td>姓名：</td>
        <td>年龄</td>
        <td>性别</td>
        <td>生日</td>
    </tr>
    <c:forEach items="${userList}" var="user">
        <tr>
            <td>${user.username}</td>
            <td>${user.age}</td>
            <td>${user.gender}</td>
            <td>${user.birthday}</td>
        </tr>
    </c:forEach>
</table>
</body>
</html>
```

- 修改用户方法示例

```java
// 编辑页面
@RequestMapping("/edit")
public String edit(Integer id, Model model){
    // 模拟通过id在数据库中查询数据,返回一个user对象，再将user对象存入model
    System.out.println("修改的用户id:"+id);
    User user = new User(id, "小明", "male", 22, "2018");
    model.addAttribute("user",user);
    return "user/userEdit";
}

// 修改用户信息
@RequestMapping("/update")
public String update(Integer id,Model model){
    System.out.println("修改的用户id:"+id);
    return "user/userList";
}
```

- 编辑jsp界面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Edit</title>
</head>
<body>
<form action="${pageContext.request.contextPath}/user/update.do?id=${user.id}" method="post">
    用户名：<input type="text" name="username" value="${user.username}"><br>
    年龄：<input type="text" name="age" value="${user.age}"><br>
    性别：<input type="text" name="gender" value="${user.gender}"><br>
    生日：<input type="text" name="birthday" value="${user.birthday}"><br>

    <button type="submit" value="提交修改"/>
</form>
</body>
</html>
```



## 5. URL模板映射

模板映射可以RESTful软件架构风格。

- 修改url格式，体验RESTful

```jsp
<a href="${pageContext.request.contextPath}/user/edit1/${user.id}.do">RESTful修改</a>
<!--配置接收url模板映射：
	{}：匹配接收页面url路径参数-->
```

- 控制器中添加方法

```java
// 编辑页面,使用RESTful风格。{id}里面参数通过@PathVariable注入到后面参数Integer id中去
@RequestMapping("/edit1/{id}")
public String edit1(@PathVariable Integer id, Model model){
    // 模拟通过id在数据库中查询数据,返回一个user对象，再将user对象存入model
    System.out.println("修改的用户id:"+id);
    User user = new User(id, "小明", "male", 22, "2018");
    model.addAttribute("user",user);
    return "user/userEdit";
}
```

- 需在web.xml中配置rest路径

```xml
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>*.do</url-pattern>
</servlet-mapping>

<!--RESTful风格-->
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/rest/*</url-pattern>
</servlet-mapping>
```

- 测试页面，`http://localhost/rest/user/edit1/1`



## 6. 转发和重定向

- 转发到同一个控制器的方法：`forward`

```java
// 示例转发到list.do。同一个控制器中转发
@RequestMapping("/test1")
public String test1(){
    return "forward:list.do";
}
```

- 转发到不同的控制器

```java
@Controller
@RequestMapping("stu")
public class StudentController {
    // 示例转发到list.do。不同控制器中转发,加上域名/user即可
    @RequestMapping("/test1")
    public String test1(){
        return "forward:/user/list.do";
    }
}
```

- 重定向，只需改成`redirect`即可

```java
@Controller
@RequestMapping("stu")
public class StudentController {
    // 示例重定向到list.do。不同控制器中转发
    @RequestMapping("/test1")
    public String test1(){
        return "redirect:/user/list.do";
    }

}
```



## 7. RequestParam

- 该注解是对参数的一些说明描述
  - value：参数名称
  - defaultValue：默认值
  - required：参数是否必须有值。如果为true，参数又是空且未定义默认值，则会报错。



- 示例

```java
@Controller
@RequestMapping("stu")
public class StudentController {
    // 示例注解请求参数配置
    @RequestMapping("/test1")
    public String test1(@RequestParam(value = "uid",required = true,defaultValue = "1")Integer uid){
        System.out.println(uid);
        return "redirect:/user/list.do";
    }
}
```

