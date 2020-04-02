---
title: 会话技术Cookie和Session
date: 2019-6-16 15:30
categories: JavaWeb
tags: [Cookie,Session]
description: 会话技术Cookie和Session
urlname: CookieAndSession
---



## 1. 会话技术

会话是浏览器和服务器之间的多次请求和响应，在一次会话中往往会产生一些数据，可以通过会话技术(Session和Cookie)保存会话中产生的数据



<!--more-->



一次会话中包含多次请求和响应

- 一次会话：浏览器第一次给服务器资源发送请求，会话建立，直到有一方断开为止。
- 主要功能：在一次会话的范围内的多次请求之间，共享数据
- 客户端会话技术：Cookie
- 服务器端会话技术：Session

## 2. Cookie

客户端会话技术，将数据保存到客户端

### 2.1 快速入门

1. 创建Cookie对象，绑定数据：`new Cookie(String name, String value)`
2. 发送Cookie对象：`response.addCookie(Cookie cookie)`
3. 获取Cookie，拿到数据：`Cookie[] request.getCookies()`

### 2.2 实现原理

基于响应头`set-cookie`和请求头`cookie`实现 

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/06/16/Cookie%E5%8E%9F%E7%90%86-1560669241487.bmp)

### 2.3 cookie的细节

1. 一次可不可以发送多个cookie？

   可以创建多个Cookie对象，使用response调用多次addCookie方法发送cookie即可

2. cookie在浏览器中保存多长时间？

   默认情况下，当浏览器关闭后，Cookie数据被销毁。可以通过设置`setMaxAge(int seconds)`来进行持久化存储，参数填写正数：将Cookie数据写到硬盘的文件中，持久化存储，并指定cookie存活时间，时间到后，cookie文件自动失效；负数是默认值，即浏览器关闭自动销毁；零表示删除cookie信息。

3. cookie能不能存储中文？

   在tomcat8之前cookie中不能直接存储中文数据，需要将中文数据进行转码(一般采用URL编码，使用`URLEncoder`和`URLDecoder`来编解码)；在tomcat8之后，cookie支持中文数据。特殊字符还是不支持，建议使用URL编码存储，方便URL解码解析。

4. 假设在一个tomcat服务器中，部署了多个web项目，那么在这些web项目中cookie能不能共享呢？

   默认情况下cookie不能共享。可以通过`setPath(String path)`方法设置cookie的获取范围，默认情况下设置当前虚拟目录，如果要共享，将path设置为`"/"`即可。

5. 在不同的tomcat服务器之间cookie能否共享？

   可以通过`setDomain(String path)`方法设置域名，如果设置的一级域名相同，那么多个服务器之间cookie可以共享，例：`setDomain(".baidu.com")`，那么`tieba.baidu.com`和`news.baidu.com`中的cookie可以共享。

> 一级域名：一级域名中只含有一个`.`，且`.`左边要没有内容。最后一个点的右边被称为一级域名，一级域名又被称为顶级域名。一级域名左边有内容的是二级域名。例`tieba.baidu.com`中，`.baodu.com`就是一级域名。



### 2.4 Cookie的特点和作用

1. cookie存储数据在客户端浏览器
2. 浏览器对于单个cookie的大小有限制(4kb)，以及对同一个域名下的总cookie数量也有限制(20个)
3. cookie一般用于存储少量的不敏感数据
4. 在不登录的情况下，完成服务器对客户端的身份识别(记住我)



### 2.5 案例：

记住上一次访问时间

#### 2.5.1 需求

1. 访问一个Servlet，如果是第一次访问，则提示：你好，欢迎你首次访问
2. 如果不是第一次访问，则提示：欢迎回来，你上次访问时间是：xxxx

#### 2.5.2 需求分析

1. 采用cookie来完成
2. 在服务器中的Servlet判断是否有一个名为lastTime的cookie
   - 有：代表不是第一次访问，先响应数据：欢迎回来，你上次访问时间是：xxxx。在回写cookie：将lasttime更新
   - 没有：代表是第一次访问，响应数据：你好，欢迎首次访问。回写cookie：设置一个lasttime为当前时间。

#### 2.5.3 代码实现

```java
@WebServlet("/CookieExercise")
public class CookieExercise extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 设置响应的消息体的数据格式以及编码
        response.setContentType("text/html;charset=utf-8");
        // 1. 获取所有cookie
        Cookie[] cookies = request.getCookies();
        boolean flag = false; // 默认cookie没有lastTime
        // 如果有cookie，则进行遍历
        if (cookies != null && cookies.length > 0){
            // 2. 遍历cookie数据
            for (Cookie cookie : cookies) {
                // 3. 获取cookie的名称
                String name = cookie.getName();
                // 4. 判断名称是否是：lastTime
                if ("lastTime".equals(name)){
                    // 有，则代表不是第一次访问
                    flag = true; // 代表有lastTime
                    // 设置cookie的value
                    // 获取当前时间的字符串，重新设置cookie的值，然后发送
                    Date date = new Date();
                    SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
                    String str_date = sdf.format(date);
                    System.out.println("编码前："+str_date);
                    // url编码
                    str_date = URLEncoder.encode(str_date,"utf-8");
                    System.out.println("编码后："+str_date);
                    // 设置cookie的存活时间一个月
                    cookie.setMaxAge(60 * 60 * 24 * 30);
                    response.addCookie(cookie); // 将cookie值添加
                    // 5. 响应数据
                    // 获取cookie的value值
                    String value = cookie.getValue();
                    System.out.println("解码前："+value);
                    // url解码
                    value = URLDecoder.decode(value,"utf-8");
                    System.out.println("解码后："+value);
                    // 将数据响应
                    response.getWriter().write("<h3>欢迎回来,你的上次访问时是"+value+"</h3>");
                    break;
                }
            }
        }
        // 如果没有cookie信息或者没有lastTime
        if (cookies == null || cookies.length == 0 || flag == false){
            // 没有，代表是第一次访问
            // 获取当前时间字符串，设置cookie值发送
            Date date = new Date();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
            String str_date = sdf.format(date);
            System.out.println("编码前："+str_date);
            // url编码
            str_date = URLEncoder.encode(str_date,"utf-8");
            System.out.println("编码后："+str_date);
            // 新建cookie对象，设置lastTime
            Cookie cookie = new Cookie("lastTime", str_date);
            // 设置cookie存活时间
            cookie.setMaxAge(60 * 60 * 24 * 30);
            // 发送cookie
            response.addCookie(cookie);
            // 响应数据
            response.getWriter().write("<h3>你好，欢迎首次访问</h3>");
        }
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```

## 3. JSP简单入门

Java Server Pages（Java服务器端页面），简称JSP。可以理解为一个特殊的页面，其中既可以定义html标签，又可以定义java代码。主要用于简化书写。

### 3.1 原理

- JSP本质上就是一个Servlet，通过`HttpJspBase`来生成Servlet。

`public abstract class HttpJspBase extends HttpServlet implements HttpJspPage`

### 3.2 JSP的脚本

就是JSP定义Java代码的方式

1. `<% 代码 %>`：定义的java代码，在service方法中。service方法中可以定义什么，该脚本中就可以定义什么。
2. `<!% 代码 %>`：定义的java代码，JSP转换后在java类的成员位置(成员变量、成员方法等)
3. `<%= 代码 %>`：定义的java代码，会输出到页面上。 输出语句中可以定义什么，该脚本中就可以定义什么。

### 3.3 JSP的内置对象

内置对象就是在JSP页面中不需要获取和创建，可以直接使用的对象。JSP一共有9个内置对象。今天简单学习3个。

- request
- response
- out：字符输出流对象。可以将数据输出到页面上。和`response.getWriter()`类似
- `response.getWriter()`和`out.write()`的区别
  1. 在tomcat服务器真正给客户端做出响应之前，会先找response缓冲区数据，再找out缓冲区数据
  2. `response.getWriter()`数据输出永远在`out.write()`的之前

## 4. Session

 服务器端会话技术，在一次会话的多次请求间共享数据，将数据保存在服务器端的对象(`HttpSession`)中。

### 4.1 `HttpSession`对象

1. 获取`HttpSession`对象：`HttpSession session = request.getSession();`
2. 使用`HttpSession`对象，有三个常用方法
   - `Object getAttribute(String name)`
   - `void setAttribute(String name, Object value)`
   - `void removeAttribute(String name)`

### 4.2 原理

- 服务器如何确保在一次会话范围内，多次获取的Session对象是同一个？

Session的实现是依赖于Cookie的。当客户端第一次向服务器发起请求时，没有cookie。会在内存中创建一个新的Session对象，并通过响应头`set-cookie`将`JSESSIONID=xxxxx`传递给客户端，客户端保存在cookie信息中。第二次向服务器发起请求时，请求头中会携带这个`JSESSIONID=xxxxx`，服务器自动获取后会查找内存中是否有这个ID的session对象，能找到就说明是同一个session对象。
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/06/16/Session%E5%8E%9F%E7%90%86-1560647657175.bmp)

### 4.3 细节问题

#### 1. 当客户端关闭后，服务器不关闭，两次获取的session是否为同一个？

- 默认情况下，不是

- 如果需要两次相同，可以创建Cookie，键为JSESSIONID，并设置cookie存活时间。在服务器不关闭的情况下，在存活时间内，只要是同一个cookie访问，两次获取的session对象就是同一个。

  ```java
  // 1. 获取session
  HttpSession session = request.getSession();
  // 2. 期望客户端关闭后，session也相同
  Cookie c = new Cookie("JSESSIONID", session.getId()); // 手动设置sessionid为当前的session对象id
  c.setMaxAge(60 * 60); // 设置cookie最大存活时间
  response.addCookie(c); // 发送cookie
  ```

#### 2. 客户端不关闭，服务器关闭后，两次获取的session是同一个吗？

- 不是同一个，但是我们要确保数据不丢失。
- session的钝化：在服务器正常关闭之前，将session对象序列化到硬盘上。
- session的活化：在服务器启动后，将session文件转化为内存中的session对象即可。

#### 3. session什么时候被销毁

1. 服务器被关闭
2. session对象调用`invalidate()`方法
3. session默认失效时间是 30分钟，可以在web.xml中配置`session-config`

### 4.4 session的特点

1. session用于存储一次会话的多次请求的数据，存储在服务器端
2. session可以存储任意类型，任意大小的数据

#### session与cookie的区别

1. session存储数据在服务器端，cookie在客户端
2. session没有数据大小限制，cookie有限制(4kb)
3. session数据安全，cookie相对而言不安全



## 5. 案例



### 5.1 需求

1. 访问带有验证码的登陆页面login.jsp
2. 用户输入用户名，密码以及验证码
   - 如果用户名和密码输入错误，跳转登陆页面，提示：用户名或密码错误
   - 如果验证码输入有误，跳转登陆页面，提示：验证码错误
   - 如果全部输入正确，则跳转主页success.jsp，显示：用户名，欢迎你



### 5.2 分析


![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/06/16/%E7%99%BB%E5%BD%95%E6%A1%88%E4%BE%8B-1560651918633.bmp)

### 5.3 代码实现

1. 登陆的Servlet

```java
@WebServlet("/loginServlet")
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 1. 设置request编码
        request.setCharacterEncoding("utf-8");
        // 2. 获取参数
        // todo：改进，获取参数Map,封装进User对象
        Map<String, String[]> map = request.getParameterMap();
        User loginUser = new User();
        try {
            BeanUtils.populate(loginUser,map);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
        // 获取用户输入的验证码
        String checkCode = request.getParameter("checkCode");
        // 3. 获取生成的验证码
        HttpSession session = request.getSession();
        String checkCode_session = (String) session.getAttribute("checkCode_session");
        // 3.1 删除session中存储的验证码
        session.removeAttribute("checkCode_session");
        // 4. 判断验证码是否正确
        if (checkCode_session != null && checkCode_session.equalsIgnoreCase(checkCode)){// 忽略大小写比较
            // 验证码正确，调用UserDao的login方法进行登陆操作
            UserDao dao = new UserDao();
            User user = dao.login(loginUser);
            // 判断用户名是否成功登陆
            if (user != null ){ // todo: 需要调用UserDao查询数据库，进行数据对比
                // 登陆成功
                // 存储用户信息
                session.setAttribute("User",user.getUsername());
                // 重定向到success.jsp
                response.sendRedirect(request.getContextPath() + "/success.jsp");
            }else {
                // 登陆失败
                // 存储信息到request
                request.setAttribute("login_error","用户名或密码错误！");
                // 转发到登陆页面
                request.getRequestDispatcher("/login.jsp").forward(request,response);
            }

        }else {
            // 验证码不一致
            // 存储信息到request
            request.setAttribute("cc_error","验证码错误！");
            // 转发到登陆页面
            request.getRequestDispatcher("/login.jsp").forward(request,response);
        }
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```

2. 登陆的前端页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Login</title>
    <script>
        window.onload = function () {
            document.getElementById("img").onclick = function () {
                this.src = "/Demo6_14/checkCodeServlet?time=" + new Date().getTime();
            }
        }
    </script>
    <style>
        div{
            color: red;
        }
    </style>
</head>
<body>
<form action="/Demo6_14/loginServlet" method="post">
    <table>
        <tr>
            <td>用户名</td>
            <td><input type="text" name="username"></td>
        </tr>
        <tr>
            <td>密码</td>
            <td><input type="password" name="password"></td>
        </tr>
        <tr>
            <td>验证码</td>
            <td><input type="text" name="checkCode"></td>
        </tr>
        <tr>
            <td colspan="2"><img src="/Demo6_14/checkCodeServlet" id="img"></td>
        </tr>
        <tr>
            <td colspan="2"><input type="submit" value="登陆"></td>
        </tr>
    </table>
</form>
    <div><%= request.getAttribute("cc_error") == null ? "" : request.getAttribute("cc_error") %></div>
    <div><%= request.getAttribute("login_error") == null ? "" : request.getAttribute("login_error") %></div>
</body>
</html>
```

3. Dao类

```java
// 操作数据库中User表的类，主要查询数据
public class UserDao {
    // 声明JDBCTemplate对象共用
    private JdbcTemplate template = new JdbcTemplate(JDBCUtils.getDataSource());

    // 登陆方法
    public User login(User loginUser){
        try{
            String sql = "select * from user where username = ? and password = ?";
            User user = template.queryForObject(sql,
                    new BeanPropertyRowMapper<User>(User.class),
                    loginUser.getUsername(), loginUser.getPassword());
            return user;
        }catch (DataAccessException e){
            // 登陆失败，打印异常，返回null
            e.printStackTrace();
            return null;
        }
    }
}
```

4. 验证码生成类在上一篇文章

