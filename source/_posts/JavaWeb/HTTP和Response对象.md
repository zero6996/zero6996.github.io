---
title: HTTP协议和Response
date: 2019-6-13 23:59
categories: JavaWeb
description: HTTP协议和Response对象的基本使用
urlname: http-response
---



## 1. HTTP协议

HTTP协议分为**请求消息**和**响应消息**两部分。请求消息是客户端发送给服务器端的数据，数据格式由请求行、请求头、请求空行、请求体4部分组成。下面讲解响应消息格式。



<!--more-->



## 1.1 响应消息

响应消息是服务器端发送给客户端的数据，具体数据格式如下4部分。

#### 1.1.1 响应行

1. 格式：HTTP/1.1(协议/版本)  200(响应状态码)  OK(状态码描述)
2. 响应状态码：服务器告诉客户端浏览器本次请求和响应的一个状态，状态码都是3位数字，具体分类如下。
   - 1xx：服务器接收客户端消息，但没有接收完毕，等待一段时间后，服务器会发送1xx状态码询问客户端是否还要继续发送数据。比较特殊的情况，了解即可。
   - 2xx：成功。例：200表示成功
   - 3xx：重定向。例：302(重定向)，304(访问缓存)
   - 4xx：客户端错误。例：404(请求路径没有对应的资源或没权限访问)，405(请求方式没有对应的doXxx方法)
   - 5xx：服务器端错误。例：500(服务器内部出现异常)

#### 1.1.2 响应头

1. 格式：头名称：值
2. 常见响应头：
   - Content-Type：服务器告诉客户端本次响应体数据格式以及编码格式
   - Content-disposition：服务器告诉客户端以什么格式打开响应体数据。默认值`in-line`：在当前页面内打开。`attachment;filename=xxx`：以附件形式打开响应体(用于文件下载)。

#### 1.1.3 响应空行

#### 1.1.4 响应体

就是传输的数据。

#### 响应字符串格式

```
HTTP/1.1 200 OK		<-----响应行
Content-Type: text/html;charset=UTF-8		<-----响应头
Content-Length: 90
Date: Thu, 13 Jun 2019 10:01:19 GMT
		<-----响应空行
<html>		<-----响应体
  <head>
    <title>$Title$</title>
  </head>
  <body>
  hello,response
  </body>
</html>
```



## 2. Response对象

主要功能是设置响应消息

### 2.1 设置响应行

- 设置状态码：`setStatus(int sc)`

### 2.2 设置响应头

- 设置响应头：`setHeader(String name,String value)`

### 2.3 设置响应体数据

使用步骤有以下两步

#### 2.3.1 获取输出流

- 字符输出流：`PrintWriter getWriter()`
- 字节输出流：`ServletOutputStream getOutputSteam()`

#### 2.3.2 使用输出流

将数据输出到客户端浏览器

### 2.4 小案例

#### 2.4.1 完成重定向

- 重定向：资源的跳转方式

- 代码实现

  ```java
  // 1. 设置状态码为302
  response.setStatus(302);
  // 2. 设置响应头location
  response.setHeader("location","/Demo6_13/responseDemo2");
  // 简单的重定向方法
  response.sendRedirect("/Demo6_13/responseDemo2");
  ```

- 重定向(`redirect`)的特点

  - 地址栏发生变化
  - 重定向可以访问其他站点(服务器)的资源
  - 重定向是两次请求。所以不能使用request对象来共享数据

- 转发(`forward`)的特点

  - 转发地址栏路径不变
  - 转发只能访问当前服务器下的资源
  - 转发是一次请求。故可以使用request对象来共享数据

- 路径的写法分类

  - 相对路径：通过相对路径不可以确定唯一资源

    - 例：`./index.html`。不以`/`开头，而是以`.`开头的路径
    - 规则：找到当前资源和目标资源之间的相对位置关系。例：`./`表示当前目录，`../`表示上一级目录。

  - 绝对路径：通过绝对路径可以确定唯一资源

    - 例：`http://localhost/Demo6_13/responseDemo2`或`/Demo6_13/responseDemo2`。这种以`/`开头的路径
    - 规则：判断定义的路径是给谁用的？判断请求将来从哪儿发出。给客户端浏览器使用需要加虚拟目录(项目的访问路径)，给服务器使用不需要加虚拟目录。

    > 虚拟目录建议使用`request.getContextPath()`来动态获取

#### 2.4.2 服务器输出字符数据到浏览器

先获取字符输出流，再输出数据。

```java
// 3.1 获取流对象之前，将流的默认编码ISO-8859-1设置为utf-8
response.setCharacterEncoding("utf-8");
// 3.2 告诉浏览器服务器发送的消息体数据的编码，建议浏览器使用该编码来解码
response.setHeader("Content-type","text/html;charset=utf-8");
// 3.3 可以使用简化的形式，设置编码
response.setContentType("text/html;charset=utf-8");
// Notice: 3.1步骤可以省略，推荐使用3.3方法
// 1. 获取字符输出流
PrintWriter pw = response.getWriter();
// 2. 输出数据
pw.write("<h1>hello response</h1>");
// 3. 解决中文乱码问题
pw.write("你好，response");
/* 
	乱码原因是：PrintWriter pw = response.getWriter(); 获取的流默认编码是ISO-8859-1
	故需在获取流之前，设置该流的默认编码，或者直接告诉浏览器响应体使用的编码，推荐其解码方式。
*/
```

#### 2.4.3 服务器输出字节数据到浏览器

获取字节输出流输出数据，一般用于获取图片字节数据然后输出到浏览器显示图片

```java
// 1. 获取字节输出流
ServletOutputStream sos = response.getOutputStream();
// 2. 输出数据
sos.write("你好".getBytes());
sos.write("hello".getBytes());
```

#### 2.4.4 验证码

本质就是一张图片，主要作用是防止恶意表单注册

```java
@WebServlet("/checkCodeServlet")
public class CheckCodeServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 1. 创建一个对象，在内存中画图片(验证码图片对象)
        int width = 100;
        int height = 50;
        BufferedImage image = new BufferedImage(width,height,BufferedImage.TYPE_INT_RGB);

        // 2. 美化图片
        // 2.1 填充背景色
        Graphics g = image.getGraphics(); // 获取画笔对象
        g.setColor(Color.PINK); // 设置画笔颜色
        g.fillRect(0,0,width,height); // 填充颜色
        // 2.2 画边框
        g.setColor(Color.blue);
        g.drawRect(0,0,width-1, height-1);
        // 2.3 写验证码
        String str = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        // 2.3.1 生产随机角标
        Random ran = new Random();
        for (int i = 1; i <= 4; i++) {
            int index = ran.nextInt(str.length());
            // 2.3.2 获取字符
            char ch = str.charAt(index);
            g.drawString(ch+"" ,width/5*i,height/2);
        }
        // 2.4 画干扰线
        g.setColor(Color.GREEN);
        for (int i = 0; i < 10; i++) {
            // 2.4.1 随机生产坐标点
            int x1 = ran.nextInt(width);
            int x2 = ran.nextInt(width);
            int y1 = ran.nextInt(height);
            int y2 = ran.nextInt(height);
            // 根据坐标点画线
            g.drawLine(x1,y1,x2,y2);
        }
        // 3. 将图片输出到页面展示
        ImageIO.write(image,"jpg",response.getOutputStream());
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}
```

#### 在网页中使用验证码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>验证码</title>
    <script>
        // 1.给超链接和图片绑定单击事件
        // 2. 重新设置图片的src属性值
        window.onload = function () {
            // 1. 获取图片对象
            var img = document.getElementById("checkCode");
            // 2. 创建单击事件
            changeimg = function () {
                // 获取时间戳
                var date = new Date().getTime();
                img.src = "/Demo6_13/checkCodeServlet?"+date;
            }
            // 3. 绑定单击事件
            img.onclick = changeimg;
            document.getElementById("change").onclick = changeimg;
        }
    </script>
</head>
<body>
<img src="/Demo6_13/checkCodeServlet" id="checkCode">
<a href="#" id="change" >看不清？换一张</a>
</body>
</html>
```

## 3. ServletContext对象

这个对象代表整个Web应用，可以和程序的容器(服务器)来进行通信

### 3.1 获取

1. 通过request对象获取：`request.getServletContext();`
2. 通过HttpServlet获取：`this.getServletContext();`

### 3.2 功能

#### 3.2.1 获取MIME类型

- MIME类型：在互联网通信过程中定义的一种文件数据类型，存储在服务器的`web.xml`文件中
- 格式：大类型/小类型，例：`text/html`，`image/jpeg`
- 获取方法：`String getMimeType(String file)`

#### 3.2.2 域对象：共享数据

常用方法如下：

1. `setAttribute(String name,Object value)`
2. `getAttribute(String name)`
3. `removeAttribute(String name)`

- ServletContext对象范围：所有用户所有请求的数据

#### 3.2.3 获取文件的真实(服务器)路径

- `String getRealPath(String path)`

```java
// 获取文件的服务器路径
String c = context.getRealPath("/c.txt"); // web目录下的资源访问
String b = context.getRealPath("/WEB-INF/b.txt"); // WEB-INF目录下的资源访问
String a = context.getRealPath("/WEB-INF/classes/a.txt"); // src目录下的资源访问
```

## 4. 综合案例：文件下载

### 4. 1 案例需求

1. 页面显示超链接
2. 点击超链接后弹出下载提示框
3. 完成图片文件下载

### 4.2 分析

1. 浏览器默认超链接指向的资源如果能够被解析，则在浏览器中展示，如果不能解析，则弹出下载提示框。不满足需求！
2. 任何资源都必须弹出下载提示框
3. 使用响应头设置资源的打开方式：`content-disposition:attachment;filename=xxx`

### 4.3 步骤

1. 定义页面，编辑超链接href属性，指向Servlet，传递资源名称filename
2. 定义Servlet
   1. 获取文件名称
   2. 使用字节输入流加载文件进内存
   3. 指定response的响应头：`content-disposition:attachment;filename=xxx`
   4. 将数据写出到response输出流，返回给客户端

### 4.2 代码实现

`downloadServlet`代码，主要实现文件的输入输出

```java
@WebServlet("/downloadServlet")
public class DownloadServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 1. 获取请求参数，文件名称
        String filename = request.getParameter("filename");
        // 2. 使用字节输入流加载文件进内存
        // 2.1 找到文件服务器路径
        ServletContext context = this.getServletContext();
        String realPath = context.getRealPath("/img/" + filename);
        // 2.2 用字节流关联文件对象
        FileInputStream fis = new FileInputStream(realPath);
        // 3. 设置response响应头
        // 3.1 设置响应头类型
        String mimeType = context.getMimeType(filename); // 获取文件的mime类型
        response.setHeader("content-type",mimeType);
        // 3.2 解决中文文件名问题
        // 3.2.1 获取user-agent请求头
        String agent = request.getHeader("user-agent");
        // 3.2.2 使用工具类方法编码文件名
        filename = DownLoadUtils.getFileName(agent, filename);
        response.setHeader("content-disposition","attachment;filename="+filename);
        // 4. 将输入流的数据写出到输出流中，将文件输出到浏览器中
        ServletOutputStream sos = response.getOutputStream();
        byte[] buff = new byte[1024 * 8];
        int len = 0;
        while ((len = fis.read(buff)) != -1){
            sos.write(buff,0,len);
        }
        fis.close();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}
```

前端html页面，提供了文件下载的超链接

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>下载文件</title>
</head>
<body>
<h2>文件下载</h2>
<br>
<a href="/Demo6_13/downloadServlet?filename=九尾.jpg">九尾</a>
<a href="/Demo6_13/downloadServlet?filename=2.jpg">图片2</a>
</body>
</html>
```

DownLoadUtils工具类，主要功能获取客户端的浏览器版本信息，然后根据不同的版本信息，设置filename的编码方式

```java
public class DownLoadUtils {

    public static String getFileName(String agent, String filename) throws UnsupportedEncodingException {
        if (agent.contains("MSIE")) {
            // IE浏览器
            filename = URLEncoder.encode(filename, "utf-8");
            filename = filename.replace("+", " ");
        } else if (agent.contains("Firefox")) {
            // 火狐浏览器
            BASE64Encoder base64Encoder = new BASE64Encoder();
            filename = "=?utf-8?B?" + base64Encoder.encode(filename.getBytes("utf-8")) + "?=";
        } else {
            // 其它浏览器
            filename = URLEncoder.encode(filename, "utf-8");
        }
        return filename;
    }

    public static void getFileName(String agent) {
    }
}

```



