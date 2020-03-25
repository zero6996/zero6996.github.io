---
title: SpringMVC学习2
date: 2019-9-4 23:59
categories: FrameWork
tags: [SpringMVC]
description: SpringMVC响应数据、文件上传、拦截器等内容
urlname: SpringMVC-Response-Data
---



## 1. 响应数据和结果视图

### 1.1 返回值分类



<!--more-->

#### 1.1.1 字符串

Controller方法返回字符串可以指定逻辑视图名，通过视图解析器解析为物理视图地址。

```java
// 返回值是字符串
@RequestMapping("/testString")
public String testString(Model model){
    System.out.println("testString方法执行了...");
    // 模拟从数据库中查询出User对象
    User user = new User("小明", "123", 22);
    // 使用model传递数据
    model.addAttribute("user",user);
    return "success"; // 指定逻辑视图名，经过视图解析器解析为jsp物理路径：/WEB-INF/pages/success.jsp
}
```



#### 1.1.2 void

上篇文章中学到Servlet原始API可以作为控制器中方法的参数：

```java
@RequestMapping("/testReturnVoid")
public void testReturnVoid(HttpServletRequest request,HttpServletResponse response)
throws Exception {
    xxx
}
```

所以就可以在controller方法形参上可以定义request和response，使用request或response指定响应结果

- 使用request跳转页面：

```java
/**
     * 通过request跳转页面
     * @param request
     * @param response
     * @throws Exception
     */
@RequestMapping("/testForward")
public void testForward(HttpServletRequest request,
                        HttpServletResponse response) throws Exception {
    System.out.println("testForward方法执行了...");
    request.getRequestDispatcher("/WEB-INF/pages/success.jsp").forward(request,response);
}
```

- 通过response重定向页面：

```java
/**
     * 通过response重定向页面
     * @param request
     * @param response
     * @throws Exception
     */
@RequestMapping("/testRedirect")
public void testRedirect(HttpServletRequest request,
                         HttpServletResponse response) throws Exception {
    System.out.println("testRedirect方法执行了...");
    response.sendRedirect("testString"); // 重定向到返回字符串页面
}
```

- 通过response指定响应结果：

```java
 /**
     * 通过 response 指定响应结果
     * @param request
     * @param response
     * @throws Exception
     */
@RequestMapping("/testReturnJson")
public void testReturnJson(HttpServletRequest request,
                           HttpServletResponse response) throws Exception {
    System.out.println("testReturnJson方法执行了...");
    // 解决中文乱码问题
    response.setCharacterEncoding("utf-8");
    response.setContentType("application/json;charset=utf-8");
    response.getWriter().write("响应json串");
}
```



#### 1.1.3 ModelAndView

- ModelAndView 是 SpringMVC 为我们提供的一个对象，该对象也可以用作控制器方法的返回值。

该对象中有两个主要方法

- ModelAndView addObject(String,Object)：添加模型到ModelMap对象中。
- void serViewName(String)：用于设置逻辑视图名称，视图解析器会根据名称前往指定视图。

- 示例代码

  - 控制器

  ```java
  /**
       * 返回ModelAndView
       * @return
       */
  @RequestMapping("/testReturnModelAndView")
  public ModelAndView testReturnModelAndView(){
      ModelAndView modelAndView = new ModelAndView();
      // 将user对象存入到modelAndView对象中，底层会把user对象存入request域中
      modelAndView.addObject("user",new User("小张","222",21));
      modelAndView.setViewName("success"); // 设置视图名称
      return modelAndView;
  }
  ```

  - jsp：

  ```jsp
  <h3>访问成功！</h3>
  ${user.username}
  ${user.password}
  ${user.age}
  ```

  

### 1.2 转发和重定向

#### 1.2.1 forward转发

controller方法提供了String类型的返回值之后，默认就是请求转发。我们也可以写成如下形式：

```java
@RequestMapping("/testForward")
    public String testForward()  {
        System.out.println("testForward方法执行了...");
        return "forward:/WEB-INF/pages/success.jsp"; // 使用了forward:,路径就必须写成物理视图url
    }
```

- 注意：如果用了`forward:`，则路径必须写成实际视图 URL，不能写逻辑视图。
- 它相当于：`request.getRequestDispatcher("/WEB-INF/pages/success.jsp").forward(request,response);`
- 使用请求转发，即可以转发到jsp，也可以转发到其他控制器方法。例：`return "forward:testString"; `

#### 1.2.2 Redirect重定向

```java
@RequestMapping("/testRedirect")
public String testRedirect() {
    System.out.println("testRedirect方法执行了...");
    return "redirect:testString"; // 重定向到返回字符串页面
}
```

- 相当于：`response.sendRedirect("testString");`
- 注意：如果是重定向到 jsp 页面，则 jsp 页面不能写在 WEB-INF 目录中，否则无法找到。

### 1.3 ResponseBody响应Json数据

- 作用：该注解用于将Controller的方法返回的对象，通过`HttpMessageConverter`接口转换为指定格式的数据如：json,xml 等，通过Response响应给客户端。



#### 1.3.1 关于静态资源拦截问题

`DispatcherServlet`会拦截到所有的资源，导致一个问题就是静态资源（img、css、js）也会被拦截到，从而不能被使用。解决问题就是需要配置静态资源不进行拦截

- 方法1：在SpringMVC.xml配置文件内使用`mvc:resources`标签配置资源文件不拦截
  - location元素表示webapp目录下的包下的所有文件
  - mapping元素表示以`/static`开头的所有请求路径

```xml
<!-- 设置静态资源不过滤 -->
<mvc:resources location="/css/" mapping="/css/**"/> <!-- 样式 -->
<mvc:resources location="/images/" mapping="/images/**"/> <!-- 图片 -->
<mvc:resources location="/js/" mapping="/js/**"/> <!-- javascript -->
```

- 方法2：在web.xml配置文件中配置如下

```xml
<!--设置访问静态资源-->
<servlet-mapping>
    <servlet-name>default</servlet-name>
    <url-pattern>*.css</url-pattern>
</servlet-mapping>
<servlet-mapping>
    <servlet-name>default</servlet-name>
    <url-pattern>*.js</url-pattern>
</servlet-mapping>
```

#### 1.3.2 示例

简单的获取json数据

- 控制器代码

```java
/**
     * 测试响应json数据
     * @param
     * @return
     */
@RequestMapping("/testResponseJson")
public void testResponseJson(@RequestBody String body){
    System.out.println("异步请求:"+body);
}
```

- jsp

```jsp
<script>
    // 页面加载,绑定单机事件
    $(function () {
        $("#btn").click(function () {
            // 发送ajax请求
            $.ajax({
                type:"post",
                url:"user/testResponseJson",
                contentType:"application/json;charset=utf-8",
                data:'{"username":"小明","password":"123","age":33}',
                dataType:"json",
                success:function (data) {
                    alert(data);
                }
            })
        })
    })
</script>
<h3>6. ResponseBody响应Json数据</h3>
<button id="btn">发送Ajax请求</button>
```

- 测试结果：`异步请求:{"username":"小明","password":"123","age":33}`

如果需要将获取的json格式数据转换为JavaBean对象，则需要导入额外的jar包

- 在pom.xml中导入坐标

```xml
<!--导入json和JavaBean对象相互转换所需jar包：jackson-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.9.0</version>
</dependency>
```

- 控制器代码修改

```java
@RequestMapping("/testResponseJson")
public @ResponseBody User testResponseJson(@RequestBody User user){
    System.out.println("异步请求:"+user);
    user.setUsername("花花");
    user.setAge(18);
    return user;
}
```

- 前端修改js

```javascript
<script>
    // 页面加载,绑定单机事件
    $(function () {
    $("#btn").click(function () {
        // 发送ajax请求
        $.ajax({
            type:"post",
            url:"user/testResponseJson",
            contentType:"application/json;charset=utf-8",
            data:'{"username":"小明","password":"123","age":33}',
            dataType:"json",
            success:function (data) {
                // data是服务器端响应的json数据
                alert(data.username);
                alert(data.password);
                alert(data.age);
            }
        })
    })
})
</script>
```



## 2. SpringMVC实现文件上传

### 2.1 文件上传回顾

#### 2.1.1 必要前提

- form表单的enctype(表单请求正文的类型)取值必须是：`multipart/form-data`，默认值是`application/x-www-form-urlencoded`。
- method属性取值必须是Post
- 提供一个文件选择域`<input type="file"/>`

#### 2.1.2 原理分析

- 当form表单的enctype取值不是默认值后，`request.getParameter()`将会失效。
- `enctype=”application/x-www-form-urlencoded”`时，form 表单的正文内容是键值对形式。
- 当form表单的enctype取值为`multipart/form-data`时，请求正文内容就变成：每一部分都是MIME类型描述的正文。

```
-----------------------------7de1a433602ac 					-->分界符
Content-Disposition: form-data; name="userName" 			-->协议头
文件上传测试											  		-->协议的正文
-----------------------------7de1a433602ac
Content-Disposition: form-data; name="file";
filename="C:\Users\zhy\Desktop\fileupload_demofile\b.txt"
Content-Type: text/plain 								    -->协议的类型（MIME 类型）
文件实际内容xxxxxx
-----------------------------7de1a433602ac--
```

#### 2.1.3 借助第三方组件实现文件上传

使用 Commons-fileupload 组件实现文件上传，需要导入该组件相应的支撑 jar 包：`Commons-fileupload 和commons-io`。commons-io 不属于文件上传组件的开发 jar 文件，但Commons-fileupload 组件从 1.1 版本开始，它工作时需要 commons-io 包的支持。

- 在pom.xml中导入所需jar包

```xml
<!--导入文件上传相关jar包-->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.4</version>
</dependency>
```



### 2.2 SpringMVC传统方式的文件上传

传统方式的文件上传，指的是我们上传的文件和访问的应用存在于同一台服务器上。且上传完毕后，浏览器可能跳转。

#### 2.2.1 编写控制器

```java
/**
* SpringMVC框架方式文件上传
*/
@RequestMapping("/MVCfileUpLoad")
public String MVCfileUpLoad(String picname, MultipartFile uploadFile,
                            HttpServletRequest request) throws Exception{
    System.out.println("SpringMVC框架方式文件上传");
    // 1.定义文件名称
    String fileName = "";
    // 1.1. 获取原始文件名
    String originalFilename = uploadFile.getOriginalFilename();
    // 1.2. 截取文件扩展名
    String extentName = originalFilename.substring(originalFilename.lastIndexOf(".") + 1,
                                                   originalFilename.length());
    // 1.3. 将文件加上随机数，防止文件重复
    String uuid = UUID.randomUUID().toString().replace("-", "").toUpperCase();
    // 1.4. 判断是否输入了文件名
    if (!StringUtils.isEmpty(picname)){
        fileName = uuid+"_"+picname+"."+extentName;
    }else {
        fileName = uuid+"_"+originalFilename;
    }
    System.out.println("文件名称："+fileName);
    // 2. 获取文件路径
    String basePath = request.getSession().getServletContext().getRealPath("/uploads");
    System.out.println("文件路径："+basePath);
    // 3. 解决用一文件夹中文件过多问题
    String datePath = new SimpleDateFormat("yyyy-MM-dd").format(new Date());
    // 4. 判断路径是否存在
    File file = new File(basePath+"/"+datePath);
    if (!file.exists()){ // 如果文件夹不存在
        file.mkdirs(); // 创建
    }
    // 5. 使用MultipartFile接口中方法，将上传的文件写到指定位置
    uploadFile.transferTo(new File(file,fileName));
    return "success";
}
```

#### 2.2.2 编写jsp页面

```jsp
<h3>文件上传</h3>
<form action="/file/MVCfileUpLoad" method="post" enctype="multipart/form-data">
    文件名称：<input type="text" name="picname">
    选择上传文件:<input type="file" name="uploadFile"/> <br>
    <input type="submit" value="上传">
</form>
```

#### 2.2.3 配置文件解析器

在SpringMVC.xml中添加如下配置

```xml
<!--配置文件上传解析器,id:multipartResolver是固定值-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!--设置上传文件的最大尺寸为 5MB=5*1024*1024=5242880字节 -->
    <property name="maxUploadSize">
        <value>5242880</value>
    </property>
</bean>
```



### 2.3 跨服务器方式的文件上传

#### 2.3.1 分服务器的目的

实际开发中，会有很多处理不同功能的服务器，例

- 应用服务器：负责部署我们的应用
- 数据库服务器：运行数据库
- 缓存和消息服务器：负责处理大并发访问的缓存和消息
- 文件服务器：负责存储用户上传文件的服务器

#### 2.3.2 准备文件服务器

创建一个新的Tomcat服务器，在WEB-INF下创建一个uploads文件夹，用于存放文件

#### 2.3.3 具体代码

- 控制器

```java
/**
     * 跨服务器文件上传
     * @return
     */
@RequestMapping("/MVCfileUpLoad2")
public String MVCfileUpLoad2(String picname, MultipartFile uploadFile) throws Exception{
    System.out.println("跨服务器文件上传");
    // 1.定义文件名称
    String fileName = "";
    // 1.1. 获取原始文件名
    String originalFilename = uploadFile.getOriginalFilename();
    System.out.println(originalFilename);
    // 1.2. 截取文件扩展名
    String extentName = originalFilename.substring(originalFilename.lastIndexOf(".") + 1,
                                                   originalFilename.length());
    // 1.3. 将文件加上随机数，防止文件重复
    String uuid = UUID.randomUUID().toString().replace("-", "").toUpperCase();
    if (!StringUtils.isEmpty(picname)){
        fileName = uuid+"_"+picname+"."+extentName;
    }else {
        fileName = uuid+"_"+originalFilename;
    }
    // 定义上传文件服务器路径
    String path = "http://localhost:8080/uploads/";

    // 完成文件上传，跨服务器版
    // 创建客户端的对象
    Client client = Client.create();
    // 和文件服务器进行连接
    WebResource resource = client.resource(path+ fileName);
    // 上传文件     todo:中文乱码问题?
    resource.put(uploadFile.getBytes());
    return "success";
}
```

- jsp

```jsp
<h3>跨服务器文件上传</h3>
<form action="/file/MVCfileUpLoad2" method="post" enctype="multipart/form-data">
    文件名称：<input type="text" name="picname">
    选择上传文件:<input type="file" name="uploadFile"/> <br>
    <input type="submit" value="上传">
</form>
```

-  403Forbidden问题

上传文件涉及到读写权限，这个报错的意思就是服务器（Tomcat）没有写入的权限，需要在服务器的web.xml文件中找到servlet标签，在servlet里添加如下字段，开启文件读写

```xml
<init-param>
    <param-name>readonly</param-name>
    <param-value>false</param-value>
</init-param>
```

- 409 Conflict 问题

文件夹未创建，在服务器`target\fileUploadServer`下创建uploads文件夹即可

- 关于中文文件名上传报错问题

[见文章](https://yq.aliyun.com/articles/641394)

Tomcat版本8.5，尚未解决问题

## 3. SpringMVC中的异常处理

### 3.1 异常处理的思路

系统中异常包括两类：预期异常和运行时异常`RuntimeException`，前者通过捕获异常从而获取异常信息，后者主要通过规范代码开发、测试通过手段减少运行时异常的发生。

系统的dao、service、controller出现都通过throws Exception向上抛出，最后由springMVC前端控制器交由异常处理器进行异常处理，如下图示：

![Exception](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/09/06/Exception-1567699963265.jpg)


### 3.2 实现步骤

- 控制器代码，模拟一个异常

```java
@Controller
@RequestMapping("user")
public class UserController {
    @RequestMapping("/testException")
    public String testException() throws SysException{
        System.out.println("testException.....");
        try {
            // 模拟异常
            int i = 1/0;
        } catch (Exception e) {
            e.printStackTrace();
            // 抛出自定义异常信息
            throw new SysException("查询错误....");
        }
        return "success";
    }
}
```

- jsp访问页面和错误页面

访问页面：

```jsp
<h3>异常处理</h3>
<a href="/user/testException">测试异常处理</a>
```

错误页面：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>出错啦！</title>
</head>
<body>
<h4>${errorMsg}</h4>

</body>
</html>
```

- 编写自定义异常类

```java
/**
 * 自定义异常类
 */
public class SysException extends Exception {
    // 存储提示信息的
    private String message;

    public SysException(String message) {
        this.message = message;
    }

    @Override
    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

- 编写自定义异常解析器

```java
/**
 * 异常处理类
 */
public class SysExceptionResolver implements HandlerExceptionResolver {
    /**
     * 处理异常业务逻辑
     * @param request
     * @param response
     * @param handler
     * @param ex 当前抛出的异常对象
     * @return
     */
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        // 获取异常对象
        SysException e = null;
        // 如果抛出的是系统自定义异常则直接转换
        if (ex instanceof SysException){ // instanceof：判断ex是否是SysException的对象、直接或间接子类、其接口实现类
            e = (SysException) ex;
        }else {
            // 如果抛出的不是系统自定义异常则重新构造一个系统错误异常
            e = new SysException("系统维护中....");
        }
        // 创建ModelAndView对象，跳转页面
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("errorMsg",e.getMessage()); // 获取异常消息，存入对象
        modelAndView.setViewName("error"); // 跳转错误页面
        return modelAndView;
    }
}
```

- 在SpringMVC.xml中，配置异常处理器

```xml
<!--配置自定义异常处理器对象-->
<bean id="SysExceptionResolver" class="cn.zero.exception.SysExceptionResolver"/>
```



## 4. SpringMVC中的拦截器

### 4.1 拦截器的作用

- SpringMVC的处理器拦截器类似于Servlet开发中的过滤器Filter，用于对处理器进行**预处理和后处理。**

- 可以定义拦截器链，连接器链就是将拦截器按着一定的顺序结成一条链，在访问被拦截的方法时，拦截器链
  中的拦截器会按着定义的顺序执行。
-  拦截器和过滤器的功能比较类似，区别点如下：
  -  过滤器是Servlet规范的一部分，任何框架都可以使用过滤器技术。
  - 拦截器是SpringMVC框架独有的。
  - 过滤器配置了`/*`，可以拦截任何资源
  - 拦截器只会对**控制器中的方法进行拦截**，不会拦截类似js、css等资源。
- 拦截器也是AOP思想的一种实现方式。
- 想要自定义拦截器，需要实现`HandlerInterceptor`接口。

### 4.2 自定义拦截器的步骤

- 编写拦截器类，实现`HandlerInterceptor`接口，重写方法

```java
/**
 * 定义拦截器
 */
public class CustomInterceptor implements HandlerInterceptor {

    /**
     * 预处理，controller方法执行前处理
     * return true放行，执行下一个拦截器，如果没有下一拦截器，执行控制器方法
     * return false不放行
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("CustomInterceptor执行了....预处理");
//        request.getRequestDispatcher("/WEB-INF/pages/error.jsp").forward(request,response);
        return true;
    }

    /**
     * 后处理，controller方法执行后处理
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle....后处理");
//        request.getRequestDispatcher("/WEB-INF/pages/error.jsp").forward(request,response);
    }

    /**
     * 结尾处理，最后执行的，success.jsp页面执行完毕后，该方法执行。
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion....尾处理");
    }
}
```

- 配置拦截器

```xml
<!--配置拦截器-->
<mvc:interceptors>
    <mvc:interceptor>
        <!--要拦截的具体方法-->
        <mvc:mapping path="/user/*"/>
        <!--不要拦截的方法-->
        <!--<mvc:exclude-mapping path=""/>-->
        <!-- 配置拦截器对象-->
        <bean class="com.zero.interceptor.CustomInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

- 控制器

```java
@Controller
@RequestMapping("user")
public class UserController {

    @RequestMapping("/testInterceptor")
    public String testInterceptor(){
        System.out.println("testInterceptor....控制器执行");
        return "success";
    }
}
```

- success.jsp

```jsp
<h3>访问成功！</h3>
<% System.out.println("success.jsp执行了.....");%>
```

- 测试结果

```
CustomInterceptor执行了....预处理
testInterceptor....控制器执行
postHandle....后处理
success.jsp执行了.....
afterCompletion....尾处理
```



### 4.3 拦截器简单案例(验证用户是否登录)

#### 4.3.1 实现思路

- 编写登录页面，需要一个控制器访问页面。
- 登录页面有提交表单的动作，需在控制器中处理。
  - 判断用户名密码是否正确
  - 正确，向session中写入用户信息
  - 返回登录成功
- 拦截用户请求，判断用户是否登录
  - 如已经登录，放行
  - 如未登录，跳转到登录页面
- 登录后使用重定向退出，涉及到`RedirectView`类
  - 作用：跟`return "redirect:xxx"`类似，也是重定向操作。
  - 重定向相对地址：`return new RedirectView("index.jsp")`，在此请求路径下找相应路。
  - 重定向绝对路径：`return new RedirectView("/index.jsp"`，相当于项目路径+此路径。

#### 4.3.2 核心代码

- 控制器代码

```java
@Controller
@RequestMapping("user")
public class UserController {
    
    /**
     * 跳转登录页面
     * @param model
     * @return
     * @throws Exception
     */
    @RequestMapping("/login")
    public String login(Model model) throws Exception{
        return "login";
    }

    /**
     * 登录提交
     * @param session
     * @param userId 用户账户
     * @param pwd 密码
     * @return
     * @throws Exception
     */
    @RequestMapping("/loginSubmit")
    public String loginSubmit(HttpSession session,String userId,String pwd) throws Exception{
        // 在session中记录用户身份信息
        session.setAttribute("activeUser",userId);
        System.out.println("用户已登录");
        return "success";
    }

    /**
     * 退出
     * @param session
     * @param request
     * @param response
     * @return
     * @throws Exception
     */
    @RequestMapping("/logOut")
    public RedirectView logOut(HttpSession session,
                         HttpServletRequest request,
                         HttpServletResponse response) throws Exception{
        // 设置session过期
        session.invalidate();
       	// return new RedirectView("index.jsp"); 相当于：localhost/user/index.jsp
        return new RedirectView("/index.jsp"); // 相当于：localhost/index.jsp
    }
}
```

- 定义登录拦截器

```java
/**
 * 登录拦截器
 */
public class LoginInterceptor implements HandlerInterceptor {
    /**
     * 预处理验证用户是否登录
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("登录拦截器执行");
        // 如果是登录页面则放行
        if (request.getRequestURI().indexOf("login")>=0){
            return true;
        }
        HttpSession session = request.getSession();
        // 如果用户已登录也放行
        if (session.getAttribute("activeUser")!=null){
            return true;
        }

        // 用户没有登录则跳转到登录页面
        request.getRequestDispatcher("/WEB-INF/pages/login.jsp").forward(request,response);
        System.out.println("用户尚未登录");
        return false;
    }
}
```

- 配置拦截器

```xml
<!--配置登录拦截器-->
<mvc:interceptors>
    <mvc:interceptor>
        <!--要拦截的具体方法-->
        <mvc:mapping path="/user/**"/>
        <!--不要拦截的方法-->
        <!--<mvc:exclude-mapping path=""/>-->
        <!-- 配置拦截器对象-->
        <bean class="com.zero.interceptor.LoginInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

- jsp登录页面，其他页面就不贴代码了

```jsp
<h3>用户登录</h3>
<form action="/user/loginSubmit" method="post">
    用户名称：<input type="text" name="userId"><br>
    密码：<input type="password" name="pwd"><br>
    <input type="submit" value="登录">
</form>
```

