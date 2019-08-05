---
title: JSP和MVC
date: 2019-6-19 23:59
categories: JavaWeb
tags: [JSP]
description: JSP学习，EL表达式和JSTL标签，MVC和三层架构
---



## 1. JSP

JSP全称`Java Server Pages`，是一种动态网页开发技术。它使用JSP标签在HTML网页中插入Java代码。标签通常以`<%开头 以%>`结束。



<!--more-->



### 1.1 指令

指令主要用于配置JSP页面，导入资源文件。

- 格式：`<%@ 指令名称 属性名1=属性值1 属性名2=属性值2 ... %>`

- 主要有三种指令：page、include、taglib

#### 1.1.1 page

主要用于配置JSP页面的

1. `contentType()`：设置响应体的mime类型以及字符集，设置当前jsp页面的编码(只有高级的IDE才能生效，如果使用低级工具，则需使用`pageEncoding`属性来设置当前页面的字符集)。
2. `import`：导包，用法同Java中的导包。
3. `errorPage`：当前页面发生异常后，会自动跳转到指定的错误页面
4. `isErrorPage`：标识当前页面是否是错误页面，默认值false。设置为true代表当前页面是错误页面，就可以使用内置对象`exception`。

#### 1.1.2 include

页面包含的，用于导入其他页面的资源文件

- `<%@include file="other.jsp" %>`

#### 1.1.3 taglib

导入外部资源

- `<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>`
  - `prefix`：前缀，自定义的。

### 1.2 注释

1. html注释`<!--  --> `：只能注释html代码片段。
2. JSP注释`<%--  --%>`：可以注释所有，推荐使用。



### 1.3 内置对象

在jsp页面中不需要创建，直接使用的对象就是内置对象，一共有9个。	 

| 变量名        | 真实类型              | 作用                                         |
| ------------- | --------------------- | -------------------------------------------- |
| `pageContext` | `PageContext`         | 当前页面共享数据，还可以获取其他八个内置对象 |
| `request`     | `HttpServletRequest`  | 一次请求访问的多个资源(通过转发)             |
| `session`     | `HttpSession`         | 一次会话的多个请求间                         |
| `application` | `ServletContext`      | 所有用户间共享数据                           |
| `response`    | `HttpServletResponse` | 响应对象                                     |
| `page`        | `Object`              | 当前页面(Servlet)的对象，`this`              |
| `out`         | `JspWriter`           | 输出对象，数据输出到页面上                   |
| `config`      | `ServletConfig`       | Servlet的配置对象                            |
| `exception`   | `Throwable`           | 异常对象                                     |

#### 面试题：写出JSP中的9个内置对象

## 2. MVC开发模式

### 2.1 JSP演变历史

1. 早期只有servlet，只能使用response输出标签数据，非常麻烦
2. 后来出现了jsp，简化了Servlet的开发，但是如果过度的使用jsp，在jsp中既写大量的java代码，又写html标签，会造成代码很难维护，也很难分工协作。
3. 故此后Java的Web开发，借鉴了MVC开发模式，使得程序的设计更加合理性。

### 2.2 MVC

MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。MVC被独特的发展起来用于映射传统的输入、处理和输出功能在一个逻辑的图形化用户界面的结构中。

1. M：Model(模型)，JavaBean。主要完成具体的业务操作，如：查询数据库，封装对象等。
2. V：View(视图)，JSP。主要展示数据
3. C：Controller(控制器)，Servlet。主要功能有获取用户的输入，调用模型，将模型返回的数据交给视图进行展示。

#### 2.2.1 优缺点

优点：

1. 耦合性低，方便维护，有利于分工协作
2. 复用性高

缺点：

1. 使得项目架构变得复杂，对开发人员要求高

## 3. EL表达式

 EL（Expression Language） 是为了使JSP写起来更加简单。表达式语言的灵感来自于 ECMAScript 和 XPath 表达式语言，它提供了在 JSP 中简化表达式的方法，让Jsp的代码更加简化。

- 语法：`${表达式}`
- jsp默认支持el表达式，有两种方法可以忽略el表达式。
  1. 忽略当前jsp页面中所有的el表达式：设置jsp中的page指令`isELIgnored="true"`即可。
  2. 忽略当前el表达式：使用`\${表达式}`

### 3.1 基本使用

#### 3.1.1 运算符

1. 算数运算符：`+ - * /(div) %(mod)`
2. 比较运算符：`> < = >= <= == !=`
3. 逻辑运算符：`&&(and)  ||(or)  !(not)`
4. 空运算符：empty
   - 功能：用于判断字符串、集合、数组对象是否为null或者长度是否为0。
   - `${empty list}`：判断list是否为null或者长度为0
   - `${not empty str}`：表示判断str对象是否不为null并且长度>0

#### 3.1.2 获取值

el表达式只能从域对象中获取值，语法如下

##### 1.`${域名称.键名}`

##### 表示从指定域中获取指定键的值。

域名称对应对象：

| 域名称           | 对象                        |
| ---------------- | --------------------------- |
| pageScope        | pageContext                 |
| requestScope     | request                     |
| sessionScope     | session                     |
| applicationScope | application(ServletContext) |

例：在request域中存储了name=张三，使用`${requestScope.name}`即可获取值。

##### 2. `${键名`

表示依次从最小的域中查找是否有该键对应的值，直到找到为止。

##### 3. 获取对象、List集合、Map集合的值

- 获取对象：`${域名称.键名.属性名}`，其本质上会去调用对象的`getter`方法
- 获取List集合：`${域名称.键名[索引]}`
- 获取Map集合：`${域名称.键名.key名称}`或者`${域名称.键名["key名称"]}`

#### 3.1.3 隐式对象

EL表达式中有11个隐式对象，常用的就pageContext

`pageContext`：获取jsp其他八个内置对象

通常用它来动态获取虚拟目录：`${pageContext.request.contextPath}`

## 4. JSTL标签

- `JavaServer Pages Tag Library`：JSP标准标签库，是由Apache组织提供的开源免费的jsp标签
- 作用：用于简化和替换jsp页面上的java代码

### 4.1 使用步骤

1. 导入JSTL相关jar包
2. 引入标签库，使用taglib指令：`<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>`
3. 使用标签

### 4.2 常用的JSTL标签

#### 4.2.1 if

相当于java代码的if语句，有一个必须属性`test`。

- `test`属性：接受布尔表达式
  - 如果表达式为true，则显示if标签体内容，如果为false，则不显示标签体内容。
  - 一般情况下，test属性值会结合el表达式一起使用
- 注意：`c:if`标签没有else情况，想要else情况，则可以再定义一个`c:if`标签

#### 4.2.2 choose

相当于java代码的switch语句

#### 4.2.3 foreach

相当于java代码的for语句，代码举例如下

```jsp
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.util.List" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
<head>
    <title>foreach标签</title>
</head>
<body>
<%--
    foreach:相当于java的for语句
        1. 完成重复性操作
            属性：
                begin:开始值
                end：结束值
                var：临时遍历
                step：步长
                varStatus:循环状态对象
                    index: 容器中元素的索引，从0开始
                    count：循环次数，从1开始
        2. 遍历容器
            属性：
                items：容器对象
                var：容器中元素的临时变量
                varStatus:循环状态对象
                    index: 容器中元素的索引，从0开始
                    count：循环次数，从1开始
--%>
<c:forEach begin="0" end="10" var="i" step="2" varStatus="s">
    ${i} <h3>${s.index}</h3>
</c:forEach>

<hr>
<%
    List list = new ArrayList();
    list.add("a");
    list.add("b");
    list.add("c");
    list.add("d");
    request.setAttribute("list",list);

%>
<c:forEach items="${list}" var="str" varStatus="s">
    ${s.index} ${s.count} ${str}<br>
</c:forEach>
</body>
</html>
```

### 4.3 综合练习

在request域中有一个存有User对象的List集合。需要使用jstl+el将List集合数据展示到jsp页面的表格table中。

```jsp
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.util.List" %>
<%@ page import="cn.zero.domain.User" %>
<%@ page import="java.util.Date" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>综合练习</title>
</head>
<body>
<%--
    在request域中有一个存有User对象的List集合。
    请使用jstl+el将List集合数据展示到jsp页面的表格table中。
--%>
<%
    List list = new ArrayList();
    list.add(new User("小张",21,new Date()));
    list.add(new User("小王",22,new Date()));
    list.add(new User("小李",23,new Date()));
    request.setAttribute("list",list);
%>
<table border="1" width="500" align="center">
    <tr>
        <th>编号</th>
        <th>姓名</th>
        <th>年龄</th>
        <th>生日</th>
    </tr>
    <c:forEach items="${list}" var="user" varStatus="s">
        <c:if test="${s.count % 2 == 0}">
            <tr bgcolor="#f08080">
                <td>${s.count}</td>
                <td>${user.name}</td>
                <td>${user.age}</td>
                <td>${user.birStr}</td>
            </tr>
        </c:if>
        <c:if test="${s.count % 2 != 0}">
            <tr bgcolor="#00bfff">
                <td>${s.count}</td>
                <td>${user.name}</td>
                <td>${user.age}</td>
                <td>${user.birStr}</td>
            </tr>
        </c:if>
    </c:forEach>
</table>
</body>
</html>

```

## 5. 三层架构

三层架构(3-tier architecture) 通常意义上的三层架构就是将整个业务应用划分为：界面层（User Interface layer）、业务逻辑层（Business Logic Layer）、数据访问层（Data access layer）。

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/06/19/%E4%B8%89%E5%B1%82%E6%9E%B6%E6%9E%84-1560957316811.png)



- 界面层(表示层)：用户看的界面。用户可以通过界面上的组件和服务器进行交互。
- 业务逻辑层：处理业务逻辑的。
- 数据访问层：操作数据存储文件。



## 6. 综合案例：用户信息列表展示

### 6.1 需求

用户点击查询，显示所有用户，可以对数据进行增删改查操作

### 6.2 设计

#### 6.2.1 技术选型

`Servlet+JSP+MySQL+JDBCTemplate+Duird+BeanUtils+tomcat`

#### 6.2.2 数据库设计

```sql
create database Users; -- 创建数据库
use Users;
create table user( -- 创建表
	id int primary key auto_increment,
    name varchar(20) not null,
    gender varchar(5),
    age int,
    address varchar(32),
    qq varchar(20),
    email varchar(50)
)
```

#### 6.2.3 开发阶段

1. 环境搭建
   1. 创建数据库环境
   2. 创建项目，导入所需jar包
2. 编码

#### 6.2.4 测试

#### 6.2.5 部署运维

### 6.3 代码实现

### 6.3.1 分析图

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/06/19/1560959814288-1560959814341.png)

 

代码后续见GitHub