---
title: HTTP协议和Request对象
date: 2019-6-10 23:59
categories: JavaWeb
tags: [HTTP]
description: HTTP协议和Request对象的基本使用
---



## 1. HTTP协议

### 1.1 基本概念

**HTTP协议**：超文本传输协议(Hyper Text Transfer Protocol)

**传输协议**：定义了客户端和服务器端通信时，发送数据的格式，特点如下：

1. 基于TCP/IP的高级协议
2. 默认端口号：80
3. 基于请求/响应模型的：一次请求对应一次响应
4. 无状态的：每次请求之间相互独立，不能交互数据



<!--more-->



### 1.2 请求消息数据格式

#### 1.2.1 请求行

`POST(请求方式) /login.html(请求URL)  HTTP/1.1(请求协议/版本)`

HTTP协议有7种请求方式，常用的有2种

- GET：请求参数在请求行中，在URL后；请求的URL长度是有限制的；且不太安全。
- POST：请求参数在请求体中；请求的URL长度没有限制；相对安全。

#### 1.2.2 请求头

客户端浏览器告诉服务器一些信息

常见请求头：

- User-Agent：浏览器告诉服务器，我们访问你使用的浏览器版本信息；可以在服务器端获取该头的信息，解决浏览器兼容性问题
- `Referer: http://192.168.1.6/login.html`，告诉服务器，我(当前请求)从哪里来？一般用于防盗链和统计工作。

#### 1.2.3 请求空行

空行，就是用于分隔POST请求的请求头和请求体的

#### 1.2.4 请求体(正文)

封装POST请求消息的请求参数的

#### 请求消息字符串格式：

```
POST /login.html  HTTP/1.1
Host: 192.168.1.6
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:66.0) Gecko/20100101 Firefox/66.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Referer: http://192.168.1.6/login.html
Content-Type: application/x-www-form-urlencoded
Content-Length: 10
Connection: keep-alive
Upgrade-Insecure-Requests: 1

uname=aaaa
```



## 2. Request

### 2.1 request对象和response对象的原理

- request和response对象是由服务器创建的，我们来使用它们。
- request对象是来获取请求消息的，response对象是来设置响应消息的。

插入原理图解

### 2.2 request对象继承体系结构

```
ServletRequest  -- 接口
	|	继承
HttpServletRequest -- 接口
	|	实现
org.apache.catalina.connector.RequestFacade 类(Tomcat实现的类)
```

### 2.3 request的功能

#### 2.3.1 获取请求消息数据

#### 2.3.1.1 获取请求行数据

- 请求行：`GET /reqdemo/demo1?name=zero  HTTP/1.1`
- 方法：
  - 获取请求方式(GET)：`String getMethod()`
  - **获取虚拟目录**(/reqdemo)：`String getContextPath()`
  - 获取Servlet路径(/demo1)：`String getServletPath()`
  - 获取get方式请求参数(name=zero):`String getQueryString()`
  - **获取请求URI**(`/reqdemo/demo1`)：`String getRequestURI()`
  - 获取请求URL(`http://localhost/reqdemo/demo1`)：`String getRequestURL()`
  - 获取协议及版本号(HTTP/1.1)：`String getProtocol()`
  - 获取客户端的IP地址：`String getRemoteAddr()`

> URI：统一资源标识符

#### 2.3.1.2 获取请求头数据

主要方法：

1. `String getHeader(String name)`：通过请求头的名称获取请求头的值，常用。
2. `Enumeration<String> getHeaderNames()`：获取所有的请求头名称；返回值是一个枚举类，可以当做迭代器使用。

#### 2.3.1.3 获取请求体数据

只有POST请求方式，才有请求体，在请求体中封装了POST请求的请求参数

步骤：

- 1.获取流对象
   - `BufferedReader getReader()` ：获取字符输入流，只能操作字符数据。
   - `ServletInputStream getInputStream()`：获取字节输入流，可以操作所有类型数据
- 2.从流对象中拿数据

#### 2.3.2 其他功能

#### 2.3.2.1 获取请求参数通用方式

不论是get还是post请求方式，都可以使用下列方法来获取请求参数

1. `String getParameter(String name)`：根据参数名称获取参数值
2. `String[] getParameterValues(String name)`：根据参数名称获取参数值的数组，一般用于复选框
3. `Enumeration<String> getParameterNames()`：获取所有请求从参数名称
4. `Map<String,String[]> getParameterMap()`：获取所有参数的map集合

> 中文乱码问题：Tomcat8已解决get方式乱码问题，但post方式还是会乱码。需要在获取参数前，设置request的编码`request.setCharacterEncoding("utf-8");`

#### 2.3.2.2 请求转发

一种在服务器内部的资源跳转方式

使用步骤：

1. 通过request对象获取请求转发器对象：`RequestDispatcher getRequestDispatcher(String path);`
2. 使用RequestDispatcher对象来进行转发：`forward(ServletRequest request, ServletResponse response)`

特点：

1. 浏览器地址栏路径不会发生变化
2. 只能转发到当前服务器内部资源中
3. 转发是一次请求

#### 2.3.2.3 共享数据

- 域对象：一个有作用范围的对象，可以在范围内共享数据
- request域：代表一次请求的范围，一般用于请求转发的多个资源中共享数据
- 方法：
  - `void setAttribute(String name, Object obj)`：存储数据(键值对形式)。
  - `Object getAttibute(String name)`：通过键获取值
  - `void removeAttribute(String name)`：通过键移除键值对

#### 2.2.2.4 获取ServletContext

`ServletContext getServletContext()`



## 3. 综合案例：用户登录

### 3.1 需求

1. 编写login.html登录页面，有用户名和密码两个输入框
2. 使用Druid数据库连接池技术，操作mysql
3. 使用JdbcTemplate技术封装JDBC
4. 登录成功跳转SuccessServlet展示：登录成功！用户名，欢迎你
5. 登录失败跳转FailServlet展示：登录失败，用户名或密码错误。



### 3.2 开发步骤

1. 创建项目，导入html页面、配置文件、jar包。

2. 创建数据库环境，创建user表

3. 创建包domain，创建类User

   ```java
   // 用户实体类
   public class User{
       private int id;
       private String username;
       private String password;
       // 生成get/set方法
       ...
   	// 生成toString方法
       ...
   }
   ```

4. 创建util包，编写工具类JDBCUtils

   ```java
   // JDBC工具类，使用Druid连接池
   public class JDBCUtils{
       private static DataSource ds = null;
       static{ // 使用静态代码块来加载配置文件等内容
           try{
               // 1. 加载配置文件
               Properties pro = new Properties();
               // 1.1 使用ClassLoader加载配置文件，获取字节输入流
               InputStream is = JDBCUtils.class.getClassLoader().getResourceAsStream("druid.properties");
               pro.load(is);
               // 2. 使用Druid连接池技术，初始化连接池对象
               ds = DruidDataSourceFactory.createDataSource(pro);
           }catch(IOException e){
               e.printStackTrace();
           }catch(Exception e){
               e.printStackTrace();
           }
       }
       // 获取连接池对象
       public static DataSource getDataSource(){
           return ds;
       }
       // 获取连接Connection对象
       public static Connection getConnection() throws SQLException{
           return ds.getConnection();
       }
   }
   ```

5. 创建dao包，创建类UserDao，提供login方法

   ```java
   // 操作数据库中User表的类
   public class UserDao{
       // 声明JDBCTemplate对象共用
       private JdbcTemplate template = new JdbcTemplate(JDBCUtils.getDataSource());
       // 定义登录方法
       public User login(User loginUser){
           try{
               // 1. 编写sql
               String sql = "select * form user where username = ? and password = ?";
               // 2. 调用query方法,查询数据
               User user = template.queryForObject(sql,
                        new BeanPropertyRowMaper<User>(User.class),
                        loginUser.getUsername(),loginUser.getPassword());
               return user; // 返回查询到的数据对象
           }catch{DataAccessException e}{
               e.printStackTrace(); // 一般做记录日志文件处理
               return null;
           }
       }
   }
   ```

6. 编写Servelt.LoginServlet类

   ```java
   // 主要Servlet类,负责处理登录验证操作，登录成功失败后的转发操作
   @WebServlet("/loginServlet") // 设置资源路径
   public class LoginServlet extends HttpServlet{
       @Override
       protected void doGet(HttpServletRequest req,HttpServletResponse resp) throws Servlet Exception,IOException{
           // 1. 设置编码
           req.setCharacterEncoding("utf-8");
           // 2. 获取请求参数
           String username = req.getParameter("username");
           String password = req.getParameter("password");
           // 3. 将数据封装成user对象
           User loginUser = new User();
           loginUser.setUsername(username);
           loginUser.setPassword(password);
           // 4. 调用UserDao的login方法
          	UserDao dao = new UserDao();
           User user = dao.login(loginUser); // 进行用户数据验证，返回一个User对象的完整数据
           // 5. 判断是否正常登录
           if(user == null){
               // 登录失败，转发向失败页面
               req.getRequestDispatcher("/failServlet").forward(req,resp);
           }else{
               // 登录成功，跳转成功页面
               // 将user对象放入request域中
               req.setAttribute("user",user);
               // 转发
               req.getRequestDispatcher("/successServlet").forward(req,resp);
           }
       }
       @Override
       protected void doPost(HttpServletRequest req,HttpServletResponse resp) throws Servlet Exception,IOException{
           this.doGet(req,resp);
       }
   }
   ```

7. 编写FailServlet和SuccessServlet类

   ```java
   // 失败处理页面,直接返回一句话
   @WebServlet("/failServlet")
   public class FailServlet extends HttpServlet{
       protected void doPost(HttpServletRequest req,HttpServletResponse resp) throws Servlet Exception,IOException{
           // 设置编码
           resp.setContentType("text/html;charset=utf-8");
           // 输出
           resp.getWriter().write("登录失败，用户名或密码错误");
       }
       protected void doGet(HttpServletRequest req,HttpServletResponse resp) throws Servlet Exception,IOException{
           this.doPost(req,resp);
       }
   }
   // 成功页面
   @WebServlet("/successServlet")
   public class SuccessServlet extends HttpServlet{
       protected void doPost(HttpServletRequest req,HttpServletResponse resp) throws Servlet Exception,IOException{
           // 获取request域中共享的user对象
           User user = (User)request.getAttibute("user");
           if(user != null){
               // 设置编码
               resp.setContentType("text/html;charset=utf-8");
               // 输出
               resp.getWriter().write("登录成功！"+user.getUsername()+"欢迎你");
       	}
       }
       protected void doGet(HttpServletRequest req,HttpServletResponse resp) throws Servlet Exception,IOException{
           this.doPost(req,resp);
       }
   }
   ```

8. 将login.html中form表单的action路径设置为虚拟目录+Servlet的资源路径

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
   	<meta charset="UTF-8">
       <title>登录页面</title>
   </head>
   <body>
   	<form action="/LoginDemo/loginServlet" method="post">
           用户名：<input type="text" name="username"><br>
           密码：<input type="password" name="password"><br>
           <input type="submit" value="登录">
       </form>    
   </body>
   </html>
   ```

#### 3.2.1 BeanUtils

apache提供的一个工具类，使用BeanUtils工具类，可以简化数据的封装，主要用于封装JavaBean

- JavaBean：标准的Java类，有以下要求
  - 类必须被public修饰
  - 必须提供空参的构造器
  - 成员变量必须使用private修饰
  - 提供公共setter和getter方法

- 功能：封装数据
- 概念：
  - 成员变量
  - 属性：
- 方法
  - `setProperty()`：设置值
  - `getProperty()`：获取值
  - `populate(Object obj, Map map)`：将map集合的键值对信息，封装到对应的JavaBean对象中

#### 3.2.2 简化LoginServlet

将上述例子中的LoginServlet类中**获取请求参数**和**封装User对象**步骤进行简化

```java
protected void doGet(HttpServletRequest req,HttpServletResponse resp) throws Servlet Exception,IOException{
        // 1. 设置编码
        req.setCharacterEncoding("utf-8");
        // 2. 获取所有请求参数
   		Map<String, String[]> map = req.getParameterMap();
        // 3. 创建User对象
        User loginUser = new User();
    	// 3.1 使用populate方法将数据封装进loginUser对象
    	try{
            BeanUtils.populate(loginUser,map);
        }catch(Exception e){
            e.printStackTrace();
        }
        // 4. 调用UserDao的login方法
       	UserDao dao = new UserDao();
        User user = dao.login(loginUser); // 进行用户数据验证，返回一个User对象的完整数据
        // 5. 判断是否正常登录
        if(user == null){
            // 登录失败，转发向失败页面
            req.getRequestDispatcher("/failServlet").forward(req,resp);
        }else{
            // 登录成功，跳转成功页面
            // 将user对象放入request域中
            req.setAttribute("user",user);
            // 转发
            req.getRequestDispatcher("/successServlet").forward(req,resp);
        }
    }
```

