---
title: JavaScript进阶
date: 2019-6-4 16:30
categories: JavaWeb
tags: [JavaScript]
description: JavaScript进阶知识
urlname: JavaScriptHigh
---



## 1. DOM概述

DOM (Document Object Model) 译为**文档对象模型**，是 HTML 和 XML 文档的编程接口。HTML DOM 定义了访问和操作 HTML 文档的标准方法。



<!--more-->



- 功能：将标记语言文档的各个组成部分，封装为对象。可以使用这些对象，对标记语言文档进行CRUD的动态操作。
- 获取页面标签(元素Element)对象。例：`document.getElementById("id值")`，通过元素id获取元素对象。



### 1.1 DOM的3个部分

W3C的DOM标准被分为3个同的部分：

- 核心DOM(针对任何结构化文档的标准模型)
  - `Document`：文档对象
  - `Element`：元素对象
  - `Attribute`：属性对象
  - `Text`：文本对象
  - `Comment`：注释对象
  - `Node`：节点对象，上述5个对象的父对象
- XML DOM(针对XML文档的标准模型)
- HTML DOM(针对HTML文档的标准模型)



### 1.2 核心DOM模型

#### 1.2.1 Document：文档对象

##### 1.2.1.1 创建(获取)

在html dom模型中可以使用window对象来获取

1. `window.document`
2. `document`

##### 1.2.1.2 方法
获取Element对象：

- `getElementById()`：根据id属性值获取元素对象。id属性一般唯一
- `getElementByTagName()`：根据元素名称获取元素对象们。返回值是一个数组
- `getElementByClassName()`：根据Class属性值获取元素对象们。返回值是一个数组
- `getElementsByName()`：根据name属性值获取元素对象们。返回值是一个数组

创建其他DOM对象：

- `createAttribute(name)`
- `createComment()`
- `createElement()`
- `createTextNode()`



##### 1.2.1.3 属性

Element：元素对象

1. 获取/创建：通过document来获取和创建
2. 方法：
   1. `removeAttribute()`：删除属性
   2. `setAttribute()`：设置属性

Node：节点对象，其他5个的父对象

- 特点：所有dom对象都可以被认为是一个节点
- 方法：
  - `appendChild()`：向节点的子节点列表的结尾添加新的子节点
  - `removeChild()`：删除(并返回)当前节点的指定子节点。
  - `replaceChild()`：用新节点替换一个子节点。
- 属性：
  - `parentNode`：返回节点的父节点

##### 表格增删案例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>动态表格</title>
    <style>
        table{
            border: 1px solid;
            margin: auto;
            width: 500px;
        }
        td,th{
            text-align: center;
            border: 1px solid;
        }
        div{
            text-align: center;
            margin: 50px;
        }
    </style>
</head>
<body>
<div>
    <input type="text" id="id" placeholder="请输入编号">
    <input type="text" id="name"  placeholder="请输入姓名">
    <input type="text" id="gender"  placeholder="请输入性别">
    <input type="button" value="添加" id="btn_add">
</div>
<table>
    <caption>学生信息表</caption>
    <tr>
        <th>编号</th>
        <th>姓名</th>
        <th>性别</th>
        <th>操作</th>
    </tr>
    <tr>
        <td>1</td>
        <td>钢铁侠</td>
        <td>男</td>
        <td><a href="javascript:void(0);" onclick="deltr(this)">删除</a></td>
    </tr>
    <tr>
        <td>2</td>
        <td>蜘蛛侠</td>
        <td>男</td>
        <td><a href="javascript:void(0);" onclick="deltr(this)">删除</a></td>
    </tr>
    <tr>
        <td>3</td>
        <td>绿巨人</td>
        <td>男</td>
        <td><a href="javascript:void(0);" onclick="deltr(this)">删除</a></td>
    </tr>
</table>
<script>
    // 1. 获取按钮
    var btn_add = document.getElementById("btn_add");
    btn_add.onclick = function () {
        // 2. 获取文本框内容
        var id = document.getElementById("id").value;
        var name = document.getElementById("name").value;
        var gender = document.getElementById("gender").value;
        // 3. 创建td，赋值td的标签体
        // id的td
        var td_id = document.createElement("td"); // 创建一个td元素
        var text_id = document.createTextNode(id); // 将获取的id标签内容字符串转换为文本节点
        td_id.appendChild(text_id); // 将节点添加进td中
        // name的td
        var td_name = document.createElement("td");
        td_name.appendChild(document.createTextNode(name));
        // gender的td
        var td_gender = document.createElement("td");
        td_gender.appendChild(document.createTextNode(gender));
        // 删除操作
        var del_td = document.createElement("td"); // 创建td
        var del_a = document.createElement("a"); // 创建a标签
        del_a.setAttribute("href","javascript:void(0);"); // 设置a标签属性
        del_a.setAttribute("onclick","deltr(this)"); // 绑定onclick事件
        var text_a = document.createTextNode("删除"); // 创建一个文本节点，并设置内容
        del_a.appendChild(text_a); // 将文本节点添加进a标签中
        del_td.appendChild(del_a); // 将a标签添加进td中
        // 创建tr，将td添加进去
        var tr = document.createElement("tr");
        tr.appendChild(td_id);
        tr.appendChild(td_name);
        tr.appendChild(td_gender);
        tr.appendChild(del_td);
        // 获取table，将tr添加到table中
        var table = document.getElementsByTagName("table")[0]; // 返回一个数组列表，故取第一个
        table.appendChild(tr);
    }
    // 可使用innerHTML优化上述内容
    btn_add.onclick = function ·() {
        // 2. 获取文本框内容
        var id = document.getElementById("id").value;
        var name = document.getElementById("name").value;
        var gender = document.getElementById("gender").value;
        var table = document.getElementsByTagName("table")[0];
        table.innerHTML += "<tr>\n" +
            "        <td>"+id+"</td>\n" +
            "        <td>"+name+"</td>\n" +
            "        <td>"+gender+"</td>\n" +
            "        <td><a href=\"javascript:void(0);\" onclick=\"deltr(this)\">删除</a></td>\n" +
            "    </tr>"
    }
    function deltr(obj) {
        var table = obj.parentNode.parentNode.parentNode; // 获取当前a标签的父父父级元素table
        var tr = obj.parentNode.parentNode; // 获取父父级tr
        table.removeChild(tr);
    }
</script>
</body>
</html>
```

### 1.3 HTML DOM

- 标签体的设置和获取：`innerHTML`

- 使用html元素对象的属性，详见[W3C参考书](http://www.w3school.com.cn/jsref/index.asp)

- 控制元素样式：
  - 使用元素的`style`属性来设置
  ```javascript
  div1.style.border = "1px solid red";
  div1.style.width = "200px";
  div1.style.fontSize = "20px"; // font-size = fontSize
  ```
  - 提前定义好类选择器的样式，通过元素的`className`属性来设置class属性值。
  
  ```html
  <style>
      .d1{
          border: 1px solid red;
          width: 100px;
          height: 100px;
      }
  </style>
  <script>
  	var div2 = document.getElementById("div2");
      div2.onclick = function () {
          div2.className = "d1";
      }
  </script>
  ```
  
  

## 2. BOM

BOM(Browser Object Model)浏览器对象模型，将浏览器的各个组成部分封装成对象

### 2.1 主要组成

- Window：窗口对象
- Navigator：浏览器对象
- Screen：显示器屏幕对象
- History：历史记录对象
- Location：地址栏对象

主要学习窗口对象，历史记录对象和地址栏对象，属性方法可参考：[W3School](http://www.w3school.com.cn/jsref/index.asp)

### 2.2 Window对象

#### 2.2.1 创建

window对象无需创建，可直接使用。

#### 2.2.2 方法

##### 2.2.2.1 与弹出框有关的方法

- `alert()`：显示带有一段信息和一个确认按钮的警告框。
- `confirm()`：显示带有一段消息以及确认按钮和取消按钮的对话框。返回一个布尔值。
- `prompt()`：显示可提示用户输入的对话框。返回值是用户输入的值。

##### 2.2.2.2 与打开关闭有关的方法

- `close()`：关闭浏览器窗口，谁调用关闭谁。
- `open()`：打开一个新的浏览器窗口。返回值是新的窗口对象。

##### 2.2.2.3 与定时器有关的方法

- `setTimeout()`：在指定的毫秒数后调用函数或计算表达式。
  - 参数1：js代码或者方法对象
  - 参数2：毫秒值
  - 返回值：唯一标识，用于取消定时器
- `clearTimeout()`：取消由`setTimeout()`方法设置的timeout。
- `setInterval()`：按照指定的周期来循环调用函数或计算表达式。
- `clearInterval()`：取消由setInterval()设置的timeout。

##### 轮播图

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>轮播图</title>
    <script>
        var number = 1;
        function fun() {
            number++;
            var banner = document.getElementById("banner");
            banner.src = "img/banner_"+number+".jpg";
            if (number==3){
                number = 0;
            }
        }
        setInterval(fun,3000);
    </script>
</head>
<body>
<img src="img/banner_1.jpg" id="banner" width="100%">
</body>
</html>
```

#### 2.2.3 属性

获取其他BOM对象：

`window.history`，`window.localtion`，`window.Navigator`，`window.Screen`

获取DOM对象：`window.document`

#### 2.2.4 特点

- Window对象不需要创建可以直接使用。例：`window.方法名();`
- window引用**可以省略**。直接`方法名();`即可

### 2.3 Location对象

1. 创建(获取)：`window.location`
2. 方法：`reload()`：重新加载当前页面(刷新)。
3. 属性：`href`：设置或返回完整的URL

#### 地址栏对象案例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>自动跳转</title>
    <style>
        p{
            text-align: center;
        }
        span{
            color: red;
        }
    </style>
    <script>
        var t = 5;
        function fun() {
            t--;
            if (t<=0){
                clearInterval();
                location.href = "https://www.baidu.com";
            }
            var times = document.getElementById("time");
            times.innerHTML = t;
        }
        setInterval(fun,1000);
    </script>
</head>
<body>
<p><span id="time">5</span>秒之后自动跳转页面</p>
</body>
</html>
```

### 2.4 History对象

1. 创建(获取)：`window.histtory`
2. 方法
   - `back()`：加载历史列表中的前一个URL。
   - `forward()`：加载历史列表中的下一个URL。
   - `go(参数)`：加载历史列表中的某个具体页面。参数是正负数，代表前进/后退几个历史记录。
3. 属性：`length`，返回当前窗口历史列表中的URL数量。



## 3. 事件监听机制

### 3.1 概念

某些组件被执行了某些操作后，触发某些代码的执行。

- 事件：某些操作。如单击、双击，键盘按下，鼠标移动等
- 事件源：组件。如按钮、文本输入框...
- 监听器：js代码
- 注册监听：将事件，事件源，监听器结合在一起。当事件源上发生了某个事件，则触发执行某个监听器代码。



### 3.2 常见的事件

#### 3.2.1 点击事件

1. `onclick`：单击事件
2. `ondblclick`：双击事件

#### 3.2.2 焦点事件

1. `onblur`：失去焦点，一般用于表单校验
2. `onfocus`：元素获得焦点

#### 3.2.3 加载事件

`onload`：一张页面或一副图像完成加载。

#### 3.2.4 鼠标事件

1. `onmousedown`：鼠标按钮被按下。可以定义形参接收`event`对象，使用`event.button`，返回点击的鼠标值：左键0，中键1，右键2
2. `onmouseup`：鼠标按钮被松开。
3. `onmousemove`：鼠标被移动。
4. `onmouseover`：鼠标移到某元素之上。
5. `onmouseout`：鼠标从某元素移开。

#### 3.2.5 键盘事件

1. `onkeydown`：某个键盘按键被按下。
2. `onkeyup`：某个键盘按键被松开。
3. `onkeypress`：某个键盘按键被按下并松开。

#### 3.2.6 选择和改变

1. `onchange`：域的内容被改变
2. `onselect`：文本被选中

#### 3.2.7 表单事件

1. `onsubmit`：确认按钮被点击，一般用于表单校验
2. `onreset`：重置按钮被点击



### 3.3 表格全选案例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>表格全选</title>
    <style>
        table{
            border: 1px solid;
            width: 500px;
            margin-left: 30%;
        }

        td,th{
            text-align: center;
            border: 1px solid;
        }
        div{
            margin-top: 10px;
            margin-left: 30%;
        }
        .out{
            background-color: white;
        }
        .over{
            background-color: pink;
        }
    </style>
    <script>
        // 在页面加载完成后绑定事件
        window.onload = function () {
            // 给全选按钮绑定单击事件
            document.getElementById("selectAll").onclick = function () {
                // 1.获取所有的checkbox
                var cbs = document.getElementsByName("cb");
                // 2. 遍历
                for(var i = 0; i < cbs.length; i++){
                    // 3. 设置每一个cb的状态为选中
                    cbs[i].checked = true;
                }
            }
            // 给全不选绑定事件
            document.getElementById("unSelectAll").onclick = function () {
                // 1.获取所有的checkbox
                var cbs = document.getElementsByName("cb");
                // 2. 遍历
                for(var i = 0; i < cbs.length; i++){
                    // 3. 设置每一个cb的状态为未选中
                    cbs[i].checked = false;
                }
            }
            //给反选绑定事件
            document.getElementById("selectRev").onclick = function () {
                // 1.获取所有的checkbox
                var cbs = document.getElementsByName("cb");
                // 2. 遍历
                for(var i = 0; i < cbs.length; i++){
                    // 3. 获取当前cb的状态，直接取反赋值
                    cbs[i].checked = !cbs[i].checked;
                }
            }
            // 第一个cb
            document.getElementById("firstCb").onclick = function () {
                // 1.获取所有的checkbox
                var cbs = document.getElementsByName("cb");
                // 2. 遍历
                for(var i = 0; i < cbs.length; i++){
                    // 3. 获取每一个cb的状态和第一个cb的状态一致
                    cbs[i].checked = this.checked;
                }
            }
            // 给所有tr绑定鼠标移到元素上和移出元素事件
            var trs = document.getElementsByTagName("tr");
            // 遍历
            for (var i = 0; i < trs.length; i++){
                // 移到元素上事件
                trs[i].onmouseover = function () {
                    this.className = "over";
                }
                // 移出元素事件
                trs[i].onmouseout = function () {
                    this.className = "out";
                }
            }
        }
    </script>
</head>
<body>

<table>
    <caption>学生信息表</caption>
    <tr>
        <th><input type="checkbox" name="cb" id="firstCb"></th>
        <th>编号</th>
        <th>姓名</th>
        <th>性别</th>
        <th>操作</th>
    </tr>

    <tr>
        <td><input type="checkbox"  name="cb"></td>
        <td>1</td>
        <td>钢铁侠</td>
        <td>男</td>
        <td><a href="javascript:void(0);">删除</a></td>
    </tr>

    <tr>
        <td><input type="checkbox"  name="cb"></td>
        <td>2</td>
        <td>绿巨人</td>
        <td>男</td>
        <td><a href="javascript:void(0);">删除</a></td>
    </tr>

    <tr>
        <td><input type="checkbox"  name="cb"></td>
        <td>3</td>
        <td>雷神</td>
        <td>?</td>
        <td><a href="javascript:void(0);">删除</a></td>
    </tr>
</table>
<div>
    <input type="button" id="selectAll" value="全选">
    <input type="button" id="unSelectAll" value="全不选">
    <input type="button" id="selectRev" value="反选">
</div>
</body>
</html>
```

### 3.4 表单检验案例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>注册页面</title>
<style>
    *{
        margin: 0px;
        padding: 0px;
        box-sizing: border-box;
    }
    body{
        background: url("img/register_bg.png") no-repeat center;
        padding-top: 25px;
    }

    .rg_layout{
        width: 900px;
        height: 500px;
        border: 8px solid #EEEEEE;
        background-color: white;
        /*让div水平居中*/
        margin: auto;
    }

    .rg_left{
        /*border: 1px solid red;*/
        float: left;
        margin: 15px;
    }
    .rg_left > p:first-child{
        color:#FFD026;
        font-size: 20px;
    }

    .rg_left > p:last-child{
        color:#A6A6A6;
        font-size: 20px;

    }

    .rg_center{
        float: left;
       /* border: 1px solid red;*/

    }

    .rg_right{
        /*border: 1px solid red;*/
        float: right;
        margin: 15px;
    }

    .rg_right > p:first-child{
        font-size: 15px;

    }
    .rg_right p a {
        color:pink;
    }

    .td_left{
        width: 100px;
        text-align: right;
        height: 45px;
    }
    .td_right{
        padding-left: 50px ;
    }

    #username,#password,#email,#name,#tel,#birthday,#checkcode{
        width: 251px;
        height: 32px;
        border: 1px solid #A6A6A6 ;
        /*设置边框圆角*/
        border-radius: 5px;
        padding-left: 10px;
    }
    #checkcode{
        width: 110px;
    }

    #img_check{
        height: 32px;
        vertical-align: middle;
    }

    #btn_sub{
        width: 150px;
        height: 40px;
        background-color: #FFD026;
        border: 1px solid #FFD026 ;
    }
    .error{
        color: red;
    }
    #td_sub{
        padding-left: 150px;
    }
</style>
<script>
    // 1. 给表单绑定submit事件。监听器中判断每个方法校验的结果。如果都为true中返回true,有一项false则返回false
    // 2. 定义一些方法分别校验各个表单项
    // 3. 给各个表单项绑定失去焦点onblur事件
    window.onload = function () {
        // 1. 给表单绑定submit事件。
        document.getElementById("form").onsubmit = function () {
            // 调用用户名校验方法 checkName();
            // 调用密码校验方法 checkPwd();
            return checkName() && checkPwd() && checkEmail() && checkTname() && checkPhone();
        }

        // 给各个表单项分别绑定离焦事件
        document.getElementById("username").onblur = checkName;
        document.getElementById("password").onblur = checkPwd;
        document.getElementById("email").onblur = checkEmail;
        document.getElementById("name").onblur = checkTname;
        document.getElementById("tel").onblur = checkPhone;
        // 校验用户名方法
        function checkName() {
            // 1. 获取用户名值
            var username = document.getElementById("username").value;
            // 2. 定义正则表达式
            var reg_username = /^\w{6,12}$/;
            // 3. 判断值是否符合正则规则
            var flag = reg_username.test(username);
            // 4. 提示信息
            var s_username = document.getElementById("s_username");
            if (flag){
                // 提示绿色对勾
                s_username.innerHTML = "<img width='30' height='20' src='img/gou.png'/>"
            } else{
                // 提示红色用户名错误
                s_username.innerHTML = "用户名格式错误！"
            }
            return flag;
        }
        // 校验密码
        function checkPwd() {
            // 1. 获取密码值
            var password = document.getElementById("password").value;
            // 2. 定义正则表达式
            var reg_password = /^(?![^a-zA-Z]+$)(?!\D+$)/;
            // 3. 判断值是否符合正则规则
            var flag = reg_password.test(password);
            // 4. 提示信息
            var s_pwd = document.getElementById("s_pwd");
            if (flag){
                // 提示绿色对勾
                s_pwd.innerHTML = "<img width='30' height='20' src='img/gou.png'/>"
            } else{
                // 提示红色密码错误
                s_pwd.innerHTML = "密码格式错误！"
            }
            return flag;
        }
        // 校验邮箱
        function checkEmail() {
            var reg_email = /^([a-zA-Z]|[0-9])(\w|\-)+@[a-zA-Z0-9]+\.([a-zA-Z]{2,4})$/;
            var flag = reg_email.test(document.getElementById("email").value);
            var s_email = document.getElementById("s_email");
            if (flag){
                s_email.innerHTML = "<img width='30' height='20' src='img/gou.png'/>";
            }else{
                s_email.innerHTML = "邮箱格式错误";
            }
        }
        // 校验姓名
        function checkTname() {
            var flag = /^[\u4E00-\u9FA5\uf900-\ufa2d]{2,4}$/.test(document.getElementById("name").value);
            var s_name = document.getElementById("s_name");
            if (flag){
                s_name.innerHTML = "<img width='30' height='20' src='img/gou.png'/>";
            }else{
                s_name.innerHTML = "姓名格式错误";
            }
        }
        // 校验手机号
        function checkPhone() {
            var flag = /^1[3|4|5|8][0-9]\d{4,8}$/.test(document.getElementById("tel").value);
            var s_phone = document.getElementById("s_phone");
            if (flag){
                s_phone.innerHTML = "<img width='30' height='20' src='img/gou.png'/>";
            } else{
                s_phone.innerHTML = "手机号码格式错误"
            }
        }
    }
</script>
</head>
<body>

<div class="rg_layout">
    <div class="rg_left">
        <p>新用户注册</p>
        <p>USER REGISTER</p>
    </div>
    <div class="rg_center">
        <div class="rg_form">
            <!--定义表单 form-->
            <form action="#" method="post" id="form">
                <table>
                    <tr>
                        <td class="td_left"><label for="username">用户名</label></td>
                        <td class="td_right">
                            <input type="text" name="username" id="username" placeholder="请输入用户名">
                            <span id="s_username" class="error"></span>
                        </td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="password">密码</label></td>
                        <td class="td_right">
                            <input type="password" name="password" id="password" placeholder="请输入密码">
                            <span id="s_pwd" class="error"></span>
                        </td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="email">Email</label></td>
                        <td class="td_right">
                            <input type="email" name="email" id="email" placeholder="请输入邮箱">
                            <span id="s_email" class="error"></span>
                        </td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="name">姓名</label></td>
                        <td class="td_right">
                            <input type="text" name="name" id="name" placeholder="请输入姓名">
                            <span id="s_name" class="error"></span>
                        </td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="tel">手机号</label></td>
                        <td class="td_right">
                            <input type="text" name="tel" id="tel" placeholder="请输入手机号">
                            <span id="s_phone" class="error"></span>
                        </td>
                    </tr>

                    <tr>
                        <td class="td_left"><label>性别</label></td>
                        <td class="td_right">
                            <input type="radio" name="gender" value="male" checked> 男
                            <input type="radio" name="gender" value="female"> 女
                        </td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="birthday">出生日期</label></td>
                        <td class="td_right"><input type="date" name="birthday" id="birthday" placeholder="请输入出生日期"></td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="checkcode" >验证码</label></td>
                        <td class="td_right"><input type="text" name="checkcode" id="checkcode" placeholder="请输入验证码">
                            <img id="img_check" src="img/verify_code.jpg">
                        </td>
                    </tr>
                    <tr>
                        <td colspan="2" id="td_sub"><input type="submit" id="btn_sub" value="注册"></td>
                    </tr>
                </table>
            </form>
        </div>
    </div>
    <div class="rg_right">
        <p>已有账号?<a href="#">立即登录</a></p>
    </div>
</div>
</body>
</html>
```

