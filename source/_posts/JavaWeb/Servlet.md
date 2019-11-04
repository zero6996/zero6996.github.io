---
title: Servlet接口
date: 2019-6-9 22:30
categories: JavaWeb
description: JavaServlet的基本使用
---



## 1. Servlet

Java Servlet是运行在服务器端的小程序，它是作为来自web浏览器或其他HTTP客户端的请求和HTTP服务器上的数据库或应用程序之间的中间层



<!--more-->



### 1.1 创建

Servlet有三种创建方式：

1. 实现Servlet接口
2. 继承GenericServlet类
3. 继承HttpServlet方法

#### 1.1.1 实现Servlet接口

```java
/*
Servlet的生命周期：从Servlet被创建到Servlet被销毁的过程。
一次创建，到处服务
一个Servlet只会有一个对象，服务所有的请求

1. 实例化 (使用构造方法创建对象)
2. 初始化 执行init方法
3. 服务 执行service方法
4. 销毁 执行destroy方法
*/
@WebServlet("/demo1") // 添加注解，设置虚拟访问路径
public class ServletDemo1 implements Servlet{ // 实现Servlet接口，重写其全部方法
    // 生命周期方法：当Servlet第一次被创建对象时执行该方法，该方法在整个生命周期中只执行一次
    @Override
    public void init(ServletConfig servletConfig) throws ServletException{
        System.out.println("====Servlet初始化了====");
    }
    // 生命周期方法：对客户端响应的方法，该方法会被多次执行，每次请求该servlet都会执行该方法
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException{
        System.out.println("====Servlet服务中====");
    }
    // 生命周期方法： 当Servlet被销毁时执行该方法
    @Override
    public void destroy(){
        System.out.println("====Servlet销毁了====");
    }
    // 当停止Tomcat时也就会销毁的Servlet
    @Override
    public ServletConfig getServletConfig(){
        return null;
    }
    @Override
    public String getServletInfo(){
        return null;
    }
}
```

#### 1.1.2 继承GenericServlet类

`GenericServlet`类默认将Servlet接口中其他方法做了默认空实现，只将`service()`方法作为抽象方法，在定义Servlet类时，可以继承`GenericeSerlvet`类，实现`service()`方法即可。

```java
public class ServletDemo2 extends GenericServlet{
    @Override
    public void service(ServletRequest arg0, ServletResponse arg1) throws ServletException,IOException{
        System.out.println("创建方法2")
    }
}
```

#### 1.1.3 继承HttpServlet方法

HttpServlet是对HTTP协议的一种封装，简化操作

1. 定义类继承HttpServlet
2. 复写doGet/doPost方法

```java
public class ServletDemo3 extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
        System.out.println("get方法执行");
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
        System.out.println("post方法执行");
        doGet(req,resp);
}
```

#### 1.1.4 配置Servlet

在web.xml中配置

```xml
<!--配置servlet-->
<servlet>
	<servlet-name>demo1</servlet-name>
    <servlet-class>servlet.ServletDemo1</servlet-class> <!--全类名-->
</servlet>
<servlet-mapping>
	<servlet-name>demo1</servlet-name>
    <url-pattern>/demo1</url-pattern> <!--servlet访问路径-->
</servlet-mapping>
```

`urlpartten`：Servlet访问路径

1. 一个Servlet可以定义多个访问路径：`@WebServlet({"/d4","/dd4","/ddd4"})`
2. 路径定义规则：
   - `/xxx`：路径匹配
   - `/xxx/xxx`：多层路径，类似目录结构
   - `*.do`：扩展名匹配

**Servlet执行原理**：

1. 当服务器接受到客户端浏览器的请求后，会解析请求URL路径，获取访问的Servlet的资源路径
2. 查询web.xml文件，是否有对应的`<url-pattern>`标签体内容
3. 如果有，则在找到对应的`<servlet-class>`全类名
4. tomcat会将字节码文件加载进内存，并创建其对象
5. 调用其方法

> Notice：`Servlet3.0`以后可以使用注解配置，在类上使用`@WebServlet("资源路径")`，即可完成配置。

### 1.2 Servlet中的生命周期方法

#### 1.2.1. 被创建：执行init方法，只执行一次

默认情况下，第一次被访问时，Servlet被创建。可以在xml中配置执行Servlet的创建时机。

- 在`<servlet>`标签下配置
  - 初次访问时就创建：`<load-on-startup>`的值为负数
  - 在服务器启动时就创建：`<load-on-startup>`的值为0或正整数

Server的init方法只被执行一次，故一个Servlet在内存中只存在一个对象，Servlet是单例的

- 多个用户同时访问时，可能存在线程安全问题
- 解决方式：尽量不要在Servlet中定义成员变量，即使定义了成员变量，也不要对其修改值。

#### 1.2.2 提供服务：执行service方法，执行多次

- 每次访问Servlet时，service方法都会被调用一次

#### 1.2.3 被销毁：执行destroy方法，只执行一次

- Servlet被销毁时执行。服务器关闭时，Servlet被销毁。
- 只有服务器正常关闭时，才会执行destroy方法。
- destroy方法在Servlet被销毁之前执行，一般用于释放资源。

