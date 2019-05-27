---
title: CSS基础
date: 2019-5-27 22:10
categories: JavaWeb
tags: [CSS]
description: CSS基础内容
---



## 1. CSS基础

概念：Cascading Style Sheets(层叠样式表)。功能强大，可以将内容展示和样式控制分离。

> 层叠：多个样式可以作用在用一个html的元素上，同时生效



<!--more-->



### 1.1 CSS的使用：CSS与HTML结合方式

1. 内联样式：在标签内使用`style`属性指定css代码，例`<div style="color:red;">this is css style</div>`
2. 内部样式：在head标签内，定义style标签，style标签体内容就是css代码
```html
<style>
    div{
        color:blue;
    }
</style>
<div>
    this is inner style
</div>
```
3. 外部样式：定义css资源文件，在head标签内，定义link标签，引入外部的资源文件。

```html
1. 新建一个a.css的文件，内容如下
div{
	color:red;
}
2. head标签内使用link标签引入
<head>
    <link rel="stylesheet" href="css/a.css">
</head>
<div>
 	this is outer style
</div>
```



## 2. CSS语法

```html
格式如下：
	选择器{
		属性名1：属性值1;
		属性名2：属性值2;
		....
	}
```

- 选择器：筛选具有相似特征的元素

> Notice：每一对属性需要使用;分隔开，最后一对属性可以不加;



## 3. 选择器

分类有：基础选择器和扩展选择器

### 3.1 基础选择器

#### 3.1.1 ID选择器

选择具体的id属性值的元素，建议在一个html页面中id值唯一

- 语法：`#id属性值{css}`

#### 3.1.2 元素选择器

选择具有相同标签名称的元素

- 语法：`标签名称{css}`

- 注意：id选择器优先级高于元素选择器

#### 3.1.3 类选择器

选择具有相同的class属性值的元素

- 语法：`.class属性值{css}`
- 注意：类选择器优先级高于元素选择器

### 3.2 扩展选择器

#### 3.2.1 选择所有元素

- 语法： `*{}`

#### 3.2.2 并集选择器

- 语法：`选择器1,选择器2{}`

#### 3.2.3 子选择器

筛选选择器1元素下的选择器2元素

- 语法：`选择器1 选择器2{}`

#### 3.2.4 父选择器

筛选选择器2的父元素选择器1

- 语法：`选择器1>选择器2{}`

#### 3.2.5 属性选择器

选择元素名称，属性名=属性值的元素

- 语法：`元素名称[属性名=“属性值”]{}`

#### 3.2.6 伪类选择器

选择一些元素具体的状态

- 语法：`元素:状态{}`
- 例：

```html
<a></a>标签
状态有：
	1. link：初始化的状态
	2. visited：被访问过的状态
	3. active：正在访问状态
	4. hover：鼠标悬浮状态
```

## 4. 属性

### 4.1 字体、文本

1. `font-size`：字体大小
2. `color`：文本颜色
3. `text-align`：对齐方式
4. `line-heigh`t：行高

### 4.2背景

- `background`

### 4.3 边框

- `border`：设置边框，符合属性

### 4.4 尺寸

- `width`：宽度
- `height`：高度

### 4.5 盒子模型

主要用于控制布局

- `margin`：外边距
- `padding`：内边距
  - 默认情况下内边距会影响整个盒子的大小
  - `box-sizing: border-box;`，设置盒子的属性，让`width`和`height`就是最终盒子的大小
- `float`：浮动
  - `float: left;` 左浮动
  - `float: right;` 右浮动

## 5. 综合案例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>注册页面</title>
    <style>
        *{
            margin: 0px; /*外边距*/
            padding: 0px;/*内边距*/
            box-sizing: border-box;
        }
        body{
            background: url("img/register_bg.png") no-repeat center;/*背景图居中*/
        }
        .rg_foreign{
            width: 900px;
            height: 500px;
            border: 8px solid #EEEEEE;/*边框颜色*/
            background-color: white;/*背景色白色*/
            /*div水平居中*/
            margin: auto;
            margin-top: 15px; /*外上边距*/
        }
        .rg_left{
            float: left;
            margin: 15px;
        }
        .rg_left > p:first-child{
            color: #FFD026;
            font-size: 22px;
        }
        .rg_left > p:last-child{
            color: #A6A6A6;
            font-size: 22px;
        }
        .rg_center{
            float: left;
            width: 450px;

        }
        .rg_right{
            float: right;
            margin: 15px;
        }
        .rg_right p{
            font-size: 15px;
        }
        .rg_right p a{
            color: pink;
        }
        .td_left{
            width: 100px;
            text-align: right;
            height: 45px;
        }
        .td_right{
            padding-left: 35px;

        }
        #uname,#password,#email,#name,#tel,#birthday,#checked{
            width: 250px;
            height: 32px;
            border: 1px solid #A6A6A6;
            /*设置边框圆角*/
            border-radius: 5px;
            padding-left: 10px;
        }
        #checked{
            width: 110px;
        }
        #img_check{
            height: 32px;
            vertical-align: middle; /*设置垂直居中*/
        }
        #btn_submit{
           width: 150px;
            height: 40px;
            background-color: #FFD026;
            border: 1px solid #FFD026;
        }
    </style>
</head>
<body>
<div class="rg_foreign">
    <div class="rg_left">
        <p>新用户注册</p>
        <p>USER REGISTER</p>
    </div>
    <div class="rg_center">
        <div class="rg_form">
            <form action="./login.html" method="post">
                <table>
                    <tr>
                        <td class="td_left"><label for="uname">用户名</label></td>
                        <td class="td_right"><input type="text" name="uname" id="uname" placeholder="请输入用户名"></td>
                    </tr>
                    <tr>
                        <td class="td_left"><label for="password">密码</label></td>
                        <td class="td_right"><input type="password" name="password" id="password" placeholder="请输入密码"></td>
                    </tr>
                    <tr>
                        <td class="td_left"><label for="email">Email</label></td>
                        <td class="td_right"><input type="email" name="email" id="email" placeholder="请输入邮箱"></td>
                    </tr>
                    <tr>
                        <td class="td_left"><label for="name">姓名</label></td>
                        <td class="td_right"><input type="text" name="name" id="name" placeholder="请输入真实姓名"></td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="tel">手机号</label></td>
                        <td class="td_right"><input type="text" name="tel" id="tel" placeholder="请输入手机号"></td>
                    </tr>
                    <tr>
                        <td class="td_left">性别</td>
                        <td class="td_right">
                            <input type="radio" name="gender" value="male" checked>男
                            <input type="radio" name="gender" value="female">女
                        </td>
                    </tr>
                    <tr>
                        <td class="td_left"><label for="birthday">出生日期</label></td>
                        <td class="td_right"><input type="date" name="birthday" id="birthday"></td>
                    </tr>
                    <tr>
                        <td class="td_left"><label for="checked">验证码</label></td>
                        <td class="td_right">
                            <input type="text" name="checked" id="checked" placeholder="请输入验证码">
                            <img src="img/verify_code.jpg" alt="" id="img_check">
                        </td>
                    </tr>
                    <tr>
                        <td colspan="2" align="center"><input type="submit" id="btn_submit" value="注册"></td>
                    </tr>
                </table>
            </form>
        </div>
    </div>
    <div class="rg_right">
        <p>已有账号? <a href="#">立即登录</a></p>
    </div>
</div>
</body>
</html>
```

