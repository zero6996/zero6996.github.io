---
title: Filter和Listener
date: 2019-6-27 23:30
categories: JavaWeb
tags: [Filter,Listener]
description: 过滤器和监听器
---

 

## 1. Filter：过滤器

Filter也称之为过滤器，它是Servlet技术中最激动人心的技术，Web开发人员通过Filter技术，对Web服务器管理的所有Web资源：例如Jsp, Servlet, 静态图片文件或静态 html 文件等进行拦截，从而实现一些特殊的功能。例如实现URL级别的权限访问控制、过滤敏感词汇、压缩响应信息等一些高级功能。


<!--more-->

### 1.1 过滤器的作用

一般用于完成通用的操作。如登录验证、统一编码处理、敏感字符过滤等

### 1.2 快速入门

1. 定义一个类，实现接口Filter
2. 复写方法
3. 配置拦截路径。使用web.xml和注解配置。

```java
@WebFilter("/*") // 访问所有资源之前，都会执行该过滤器
public class FilterDemo1 implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("FilterDemo1执行了...");
        // 放行
        filterChain.doFilter(servletRequest,servletResponse);
    }
    @Override
    public void destroy() {
    }
}
```

### 1.3 过滤器细节

#### 1.3.1 web.xml配置方法

```xml
<filter>
    <filter-name>demo1</filter-name>
    <filter-class>cn.zero.web.filter.FilterDemo1</filter-class>
</filter>
<filter-mapping>
    <filter-name>demo1</filter-name>
    <url-pattern>/*</url-pattern> 
</filter-mapping>
```

#### 1.3.2 过滤器执行流程

1. 执行过滤器
2. 执行放行后的资源
3. 回来执行过滤器放行代码下面的代码

#### 1.3.3 过滤器生命周期方法

1. `init`：在服务器启动后，会创建Filter对象，然后调用init方法，只执行一次。一般用于加载资源。
2. `doFilter`：每一次请求被拦截资源时，会执行该方法，会执行多次
3. `destroy`：在服务器关闭后，Filter对象被销毁。如果服务器是正常关闭，则会执行destroy方法。只执行一次。一般用于释放资源。`

#### 1.3.4 过滤器配置详解

##### 拦截路径配置：

- 具体资源路径：`/index.jsp`   ，只有访问index.jsp资源时，过滤器才会被执行
- 拦截目录： `/user/*`  ，访问/user下的所有资源时，过滤器都会被执行
- 后缀名拦截：`*.jsp` ， 访问所有后缀名为jsp资源时，过滤器都会被执行
- 拦截所有资源：`/*` ， 访问所有资源时，过滤器都会被执行

##### 拦截方式配置：资源被访问的方式

- 注解配置：需要设置`dispatcharTypes`属性

  - REQUEST：默认值。浏览器直接请求资源。
  - FORWARD：转发访问资源。
  - INCLUDE：包含访问资源
  - ERROR：错误跳转资源
  - ASYNC：异步访问资源

  ```java
  //@WebFilter(value = "/index.jsp",dispatcherTypes = DispatcherType.REQUEST) // 浏览器直接请求index.jsp资源时，该过滤器会被执行
  //@WebFilter(value = "/index.jsp",dispatcherTypes = DispatcherType.FORWARD) // 转发访问index.jsp资源时，该过滤器会被执行
  @WebFilter(value = "/index.jsp",dispatcherTypes = {DispatcherType.FORWARD,DispatcherType.REQUEST}) // 直接访问或转发访问index.jsp资源时，该过滤器会被执行
  ```

  

- web.xml
  
  - 设置`<dispatcher></dispatcher>`标签即可

#### 1.3.5 过滤器链(配置多个过滤器)

如果有两个过滤器：过滤器1和过滤器2，那么他们的执行顺序是

1. 过滤器1
2. 过滤器2
3. 资源执行
4. 过滤器2
5. 过滤器1

##### 过滤器先后顺序问题

1. 注解配置：按照类名的字符串比较规则比较，值小的先执行。例：`AFilter和BFilter，AFilter就会先执行。`
2. web.xml配置：`<filter-mapping>` 谁定义在上边，谁先执行。

### 1.4 案例

#### 1. 登录验证

将以前做的用户信息管理系统加上访问权限控制。

```java
/**
 * 登录过滤器,访问除了登录资源以外的所有资源，都必须先登录
 */
@WebFilter("/*")
public class LoginFilter implements Filter {
    public void destroy() {
    }
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        // 0. 强制转换
        HttpServletRequest request = (HttpServletRequest) req;
        // 1. 获取资源请求路径
        String uri = request.getRequestURI();
        // 2. 判断是否是登录相关的资源,要注意排除掉css/js/图片/验证码等资源
        if (uri.contains("/login.jsp") || uri.contains("loginServlet") || uri.contains("checkCodeServlet") || uri.contains("/css/") || uri.contains("/js/") || uri.contains("/fonts/")){
            // 是， 说明用户就是想登录，放行
            chain.doFilter(req,resp);
        }else {
            // 不是登录相关资源，需验证用户是否登录才能放行
            // 3. 从session中获取loginUser
            Object loginUser = request.getSession().getAttribute("loginUser");
            if(loginUser != null){
                // 登录过了，放行
                chain.doFilter(req,resp);
            }else {
                // 没有登录，跳转到登录页面
                request.setAttribute("login_msg","你尚未登录，请登录");
                request.getRequestDispatcher("/login.jsp").forward(request,resp);
            }
        }
        // 2. 判断当前用户是否登录，通过session值是否有user来判断
    }
    public void init(FilterConfig config) throws ServletException {
    }
}
```



#### 2. 敏感词汇过滤

对用户管理系统项目的录入数据进行敏感词汇过滤，需要对request对象进行增强

##### 增强对象的功能：

通过设计模式来解决。

> **设计模式**：一些通用的解决固定问题的方式。

这里主要使用代理模式来增强对象功能。

##### 代理模式

代理(Proxy)是一种设计模式，提供了对目标对象另外的访问方式；即通过**代理对象**访问**目标对象**。这样做的好处是：可以在目标对象实现的基础上，**增强**额外的**功能**操作，即扩展目标对象的功能。

详见：[博客文章](https://www.cnblogs.com/cenyu/p/6289209.html)

> 代理模式的关键点是：代理对象与目标对象。代理对象是对目标对象的扩展，并会调用目标对象，类似Python的装饰器。

目标对象：真实的对象，被代理的对象。

代理对象：代理真实对象做一些操作。

代理模式：代理对象代理真实对象，达到增强真实对象功能的目的。

##### 实现方式

分静态代理和动态代理，这里主要讲解动态代理的使用。

##### 动态代理

动态代理有以下特点：

1. 代理对象,不需要实现接口
2. 代理对象的生成,是利用JDK的API,动态的在内存中构建代理对象(需要我们指定创建代理对象/目标对象实现的接口的类型)
3. 动态代理也叫做:JDK代理,接口代理

###### JDK生成代理对象的API

代理类所在包：`java.lang.reflect.Proxy`

实现代理使用`newProxyInstance`方法，该方法需接收三个参数，分别是：

- `ClassLoader loader`：指定当前目标对象使用类加载器，获取类加载器的方法是固定的。
- `Class<?>[] interfaces`：目标对象实现的接口类型，使用泛型方式确认类型
- `InvocationHandler()`：事件处理，执行目标对象的方法时，会触发事件处理器的方法，将当前执行目标对象的方法作为参数传入。

增强方式：

1. 增强参数列表
2. 增强返回值类型
3. 增强方法体执行逻辑

##### 代码示例：敏感词汇过滤器

```java
public class SensitiveWordsFilter implements Filter {
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        // 1. 创建代理对象，增强getParameter方法
        ServletRequest proxy_req = (ServletRequest) Proxy.newProxyInstance(req.getClass().getClassLoader(), req.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                // 增强getParameter方法

                // 判断是否是getParameter方法
                if (method.getName().equals("getParameter")){
                    // 增强返回值
                    // 1. 获取返回值
                    String value = (String) method.invoke(req, args);
                    // 2. 判断不为空
                    if (value != null){
                        // 3. 遍历敏感词汇数组,判断返回值是否有敏感词
                        for (String str: list){
                            if (value.contains(str)){
                                // 有，则替换为***
                                value = value.replaceAll(str, "***");
                            }
                        }
                    }
                    return value;
                }
            	return method.invoke(req,args);
            });
            // 放行
            chain.doFilter(proxy_req,resp);
        }
        private List<String> list = new ArrayList<>(); // 敏感词汇集合
    public void init(FilterConfig config) throws ServletException {
        try{
            // 1. 获取文件真实路径
            ServletContext servletContext = config.getServletContext();
            String realPath = servletContext.getRealPath("/WEB-INF/classes/SensitiveWords.txt");
            // 2. 读取文件
            BufferedReader br = new BufferedReader(new FileReader(realPath));
            // 3. 将文件的每一行数据添加到list中
            String line = null;
            while((line = br.readLine()) != null){
                list.add(line);
            }
            br.close();
            System.out.println(list);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    public void destroy() {
    }
}  
```



## 2. Listener：监听器

web的三大组件之一。用于监听web常见对象如：`HttpServletRequest,HttpSession,ServletContext。`

### 2.1 监听器的作用

监听web对象创建与销毁。

监听web对象的属性变化。

监听session绑定JavaBean操作。

### 2.2 事件监听机制基本概念

- 事件：一件事件
- 事件源：产生这件事情的源头。
- 监听器：对某件事情进行处理监听的一个对象
- 注册监听：将事件、事件源、监听器绑定在一起。当事件源上发生某个事件后，执行监听器代码。



### 2.3 `ServletContextListener`

用于监听`SeervletContext`对象的创建和销毁。有如下两个方法：

- `void contextDestroyed(ServletContextEvent sce)`：`ServletContext`对象被销毁之前会调用该方法。
- `void contextInitialized(ServletContextEvent sce)`：`ServletContext`对象创建后会调用该方法。

### 2.4 步骤

1. 定义一个类，实现`ServletContextListener`接口

2. 复写方法

3. 配置

   - web.xml配置

     ```xml
     <listener>
         <listener-class>cn.zero.listener.ContextLoaderListener</listener-class>
     </listener>
     ```

   - 注解配置：`@WebListener`



#### 代码示例

~~~java
@WebListener
public class ContextLoaderListener implements ServletContextListener {

    /**
     * 监听ServletContext对象创建的。ServletContext对象服务器启动后自动创建。
     *
     * 在服务器启动后自动调用
     * @param servletContextEvent
     */
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        // 加载资源文件
        // 1. 获取ServletContext对象
        ServletContext servletContext = servletContextEvent.getServletContext();
        // 2. 加载资源文件
        String contextConfigLocation = servletContext.getInitParameter("contextConfigLocation");
        // 3. 获取真实路径
        String realPath = servletContext.getRealPath(contextConfigLocation);
        // 4. 加载进内存
        try {
            FileInputStream fis = new FileInputStream(realPath);
            System.out.println(fis);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        System.out.println("ServletContext对象被创建了....");
    }

    /**
     * 在服务器关闭后，ServletContext对象被销毁。
     *
     * 当服务器正常关闭后该方法被调用。
     * @param servletContextEvent
     */
    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        System.out.println("ServletContext对象被销毁了....");

    }
}
~~~

