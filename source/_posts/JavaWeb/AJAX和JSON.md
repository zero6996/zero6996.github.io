---
title: AJAX和Json
date: 2019-7-10 23:59
categories: JavaWeb
tags: [Ajax,Json]
description: AJAX和Json基础知识
urlname: ajax-json
---



## AJAX

Ajax 即“**A**synchronous **J**avascript **A**nd **X**ML”（异步 JavaScript 和 XML），是指一种创建交互式网页应用的网页开发技术。



<!--more-->



 **同步和异步**：在客户端和服务器端相互通信的基础上。

- 同步：客户端必须等待服务器端的响应。在等待的期间客户端不能做其他操作。
- 异步：客户端不需要等待服务器端的响应。在服务器处理请求的过程中，客户端可以进行其他操作。



Ajax 是一种在无需重新加载整个网页的情况下，能够更新部分网页的技术。

通过在后台与服务器进行少量数据交换，Ajax 可以使网页实现异步更新。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。而传统的网页（不使用 Ajax）如果需要更新内容，必须重载整个网页页面。

Ajax可以**提升用户的体验**。



### 实现方式

#### 1. 原生的JS实现方式

```javascript
<script>
    // 定义方法
    function fun() {
        // 发送异步请求
        // 1. 创建核心对象(固定写法)
        var xmlhttp;
        if (window.XMLHttpRequest)
        {// code for IE7+, Firefox, Chrome, Opera, Safari
            xmlhttp=new XMLHttpRequest();
        }
        else
        {// code for IE6, IE5
            xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
        }

        // 2. 建立连接
        /*
                参数：
                1. 请求方式：GET,POST
                   - get方式：请求参数在URL后边拼接。send方法为空参
                   - post方式：请求参数在send方法中定义
                2. 请求的URL：
                3. 同步还是异步请求：true异步，false同步
        */
        xmlhttp.open("GET","/Demo7_10/ajaxServlet?username=xiaoming",true);

        // 3. 发送请求
        xmlhttp.send();
        // 4. 接受和处理来自服务器的响应结果
        // 获取方式：xmlhttp.responseText; 什么时候获取? 当服务器响应成功后再获取
        // 当xmlhttp对象的就绪状态改变时，触发事件：onreadystatechange。
        xmlhttp.onreadystatechange=function()
        {
            // 判断readyState就绪状态是否为4，并且status状态码为200
            if (xmlhttp.readyState==4 && xmlhttp.status==200)
            {
                // 获取服务器的响应结果
                var responseText = xmlhttp.responseText;
                alert(responseText);
            }
        }
	}
</script>
```

#### 2. JQuery实现方式

1. `$.ajax()`，语法：`$.ajax({键值对})`；

   ```javascript
   <script>
       // 定义方法
       function fun() {
           // 使用 $.ajax()发送异步请求
           $.ajax({
               url:"ajaxServlet",// 请求路径
               type:"POST", // 请求方式
               // data:"username=jack$age=23", // 请求参数
               data:{"username":"jack","age":23},
               success:function (data) {
                   alert(data);
               }, // 响应成功后执行的函数
               error:function () {
                   alert("出错啦...")
               }, // 表示如果请求响应出现错误，会执行的回调函数
               dataType:"text" // 设置接受到的响应数据的格式
           });
   	}
   </script>
   ```

2. `$.get()`，发送get请求，语法：`$.get(url,[data],[callback],[type])`，参数如下。

   - `url`：请求路径
   - `data`：请求参数
   - `callback`：回调函数
   - `type`：响应结果的类型

   ```javascript
   <script>
       // 定义方法
       function fun() {
           // 使用 $.get()发送异步请求
           $.get("ajaxServlet",{username:"rose"},function (data) {
               alert(data);
           },"text");
   	}
   </script>
   ```

3. `$.post()`，发送post请求，语法：`$.post(url,[data],[callback],[type])`，参数同上。



## Json

### 1. 基本概念

- JSON 指的是 JavaScript 对象表示法（*J*ava*S*cript *O*bject *N*otation），是一种轻量级的数据交换格式。

- **JSON 是存储和交换文本信息的语法。**
- 进行数据的传输。
- **JSON 比 XML 更小、更快，更易解析。**



### 2.  JSON 语法规则

#### 2.1 基本规则

JSON 语法是 JavaScript 对象表示法语法的子集。

- 数据在名称/值对中：json数据是由键值对构成的
  - 键用引号(单双都可)引起来，也可以不使用引号。
  - 值的取值类型：
    - 数字（整数或浮点数）
    - 字符串（在双引号中）
    - 逻辑值（true或false）
    - 数组（在方括号中），例：`{"persons":[{},{}]}`
    - 对象（在花括号中），例：`{"address":{"province":"杭州"...}}`
    - null
- 数据由逗号分隔：多个键值对由逗号分隔
- 花括号保存对象：使用`{}`定义json格式
- 方括号保存数组：`[]`

#### 2.2 获取数据

1. json对象.键名
2. json对象["键名"]
3. 数组对象[索引]
4. 遍历

```javascript
 <script>
    // 1. 定义基本格式
    var person = {name:"xiaozhang",age:23,'gender':true};

    var ps = [
        {name:"xiaozhang",age:23,'gender':true},
        {name:"xiaoli",age:25,'gender':true},
        {name:"xiaohua",age:19,'gender':false}];

    // 获取person对象中的所有键和值
    // for in 循环
    for(var key in person){
        alert(key+"："+person[key]);
    }
    // 获取ps数组中所有对象的键值
    for (var i = 0; i < ps.length; i++){
        var p = ps[i];
        for (var key in p){
            alert(key+"："+p[key]);
        }
    }
</script>
```



#### 2.3 JSON数据和Java对象的相互转换

- JSON解析器，常见的解析器有：`Jsonlib,Gson,fastjson,jackson`



##### 2.3.1 JSON转为Java对象

- 导入`jackson`的相关jar包
- 创建`jackson`核心对象：`ObjectMapper`
- 调用`ObjectMapper`的相关方法进行转换：`readValue(json字符串数据，Class)`

```java
// 演示JSON字符串转换为Java对象
@Test
public void test5() throws Exception {
    // 1. 初始化JSON字符串
    String json = "{\"gender\":\"男\",\"name\":\"张三\",\"age\":23}";
    // 2. 创建ObjectMapper对象
    ObjectMapper mapper = new ObjectMapper();
    // 3. 转为Java对象 Person
    Person person = mapper.readValue(json, Person.class);
    System.out.println(person); // Person{name='张三', age=23, gender='男', birthday=null}
}
```



##### 2.3.2 Java对象转为JSON

1. 导入`jackson`的相关jar包
2. 创建`jackson`核心对象：`ObjectMapper`
3. 调用`ObjectMapper`的相关方法进行转换

**转换方法**：

- `writeValue(参数，obj)`
- 常用参数：
    - `File`：将obj对象转换为json字符串，并保存到指定的文件中。
    - `Writer`：将obj对象转换为json字符串，并将json数据填充到字符输出流中。
    - `OutputStream`：将obj对象转换为json字符串，并将json数据填充到字节输出流中。
- `writeValueAsString(obj)`：将对象转换为json字符串

```java
public void test1() throws Exception {
        // 1. 创建Person对象
    Person person = new Person();
    person.setName("张三");
    person.setAge(23);
    person.setGender("男");

    // 2. 创建Jackson核心对象 ObjectMapper
    ObjectMapper mapper = new ObjectMapper();
    // 3. 调用方法转换
    String json = mapper.writeValueAsString(person);
    System.out.println(json); // {"name":"张三","age":23,"gender":"男"}
    // writeValue,将数据写到d://json.txt中
    mapper.writeValue(new File("d://json.txt"),person);
    // writeValue,将数据关联到Writer中
    mapper.writeValue(new FileWriter("d://json2.txt"),person);
}
```



**注解**：

- `@JsonIgnore`：排除属性

- `@JsonFormat`：属性值格式化
  - `@JsonFormat(pattern = "yyyy-MM-dd")`

**复杂Java对象转换**：

- `List`：数组
- `Map`：对象格式一致





### 综合练习

用户注册页面，输入用户名，异步请求数据库验证用户名是否存在。

- 服务器响应的数据，在客户端使用时，要想当做json数据格式使用，方法有如下：
  - `$.get(type);`将最后一个参数type指定为“json”；
  - 在服务器端设置MIME类型：`response.setContentType("application/json;charset=utf-8");`



### 代码实现

- 前端部分：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>用户注册</title>
    <script src="js/jquery-3.3.1.min.js"></script>
    <script>
        // 在页面加载完成后
        $(function () {
            // 给username绑定blur(离焦)事件
            $("#username").blur(function () {
                // 获取username文本框输入的值
                var username = $(this).val();
                // 发送ajax请求
                // 期望服务器响应会的数据格式：{"userExsit":true,"msg":"用户名已存在！"}
                //                        {"userExsit":false,"msg":"用户名可用！"}
                $.get("regUserServlet",{username:username},function (data) {
                    var span = $("#s_username");
                    if (data["userExsit"] == true){
                        // 用户名已存在
                        span.css("color","red");
                        span.html(data.msg);
                    }else{
                        // 用户名不存在
                        span.css("color","green");
                        span.html(data.msg);
                    }
                })
            });
        })
    </script>
</head>
<body>
    <form action="">
        <input type="text" id="username" name="username" placeholder="请输入用户名">
        <span id="s_username"></span>
        <br>
        <input type="password" name="password" placeholder="请输入密码"><br>
        <input type="submit" value="注册">
    </form>
</body>
</html>
```

- 后端部分：

```java
package cn.zero.web.servlet;

import com.fasterxml.jackson.databind.ObjectMapper;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

@WebServlet("/regUserServlet")
public class RegUserServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 1. 获取用户名
        String username = request.getParameter("username");
        String json = "{\"userExsit\":true,\"msg\":\"用户名已存在！\"}";
        // 2. 调用service层判断用户名是否存在
        // 期望服务器响应会的数据格式：{"userExsit":true,"msg":"用户名已存在！"}
        //                        {"userExsit":false,"msg":"用户名可用！"}
        // 设置响应的数据格式为json
        response.setContentType("application/json;charset=utf-8"); // 解决乱码问题
        Map<String,Object> map = new HashMap<>();
        if ("张三".equals(username)) {
            map.put("userExsit",true);
            map.put("msg","用户名已存在，请更换！");
        }else{
            map.put("userExsit",false);
            map.put("msg","用户名可用");
        }
        // 将map转为json，并传递给客户端
        ObjectMapper mapper = new ObjectMapper();
        mapper.writeValue(response.getWriter(),map);
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```

