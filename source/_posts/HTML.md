---
title: HTML基础
date: 2019-5-26 23:30
categories: JavaWeb
tags: [HTML]
description: Java的Web概述和HTML基础
---

## 1. Web概述
### 1.1 软件架构

主要有如下两种架构

1. C/S：Client/Server 客户端/服务器端
2. B/S：Browser/Server 浏览器/服务器端



<!--more-->



### 1.2 资源分类

####  1.2.1 静态资源

使用静态网页开发技术发布的资源，特点如下：

- 所有用户访问，得到的结果是一样的，如：文本、图片、音视频等资源
- 静态网页开发技术：`HTML、CSS、JavaScript`
- 如果用户请求的是静态资源，那么服务器会直接将静态资源发送给浏览器。浏览器中内置了静态资源的解析引擎，可以展示静态资源

| 名称       | 作用概述                               |
| ---------- | -------------------------------------- |
| HTML       | 用于搭建基础网页，展示页面的内容       |
| CSS        | 用于美化页面，布局页面                 |
| JavaScript | 控制页面的元素，让页面有一些动态的效果 |

####  1.2.2 动态资源

使用动态网页及时发布的资源，特点：

- 所有用户访问，得到的结果可能不一样
- 动态网页开发技术：`jsp/servlet，php,asp`等
- 如果用户请求的是动态资源，那么服务器会执行动态资源，转换为静态资源，再发送给浏览器



## 2. HTML

html是最基础的网页开发语言

- `Hyper Text Markup Language`(超文本标记语言)
  - 超文本：是用超链接的方法，将各种不同空间的文字信息组织在一起的网状文本
  - 标记语言：由标签构成的语言。<标签名称> 如 html，xml

> Notice：标记语言不是编程语言



### 2.1 快速入门

#### 2.1.1 语法

```tex
1. html文档后缀名 .html 或者 .htm
2. 标签分为：围堵标签(<html></html>)和自闭标签(<link/>)
3. 标签可以嵌套：例<a><b></b></a>
4. 在开始标签中可以定义属性。属性是由键值对构成，值需用单双引号引起来。
5. html的标签不区分大小写，但建议使用小写
```
#### 特殊字符表
![特殊字符表](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/26/%E7%89%B9%E6%AE%8A%E5%AD%97%E7%AC%A6%E8%A1%A8-1558883012702.png)

### 2.2 标签

1. **文件标签**：构成html最基本的标签
| 标签类型 | 标签名称          | 标签作用                                   |
| -------- | ----------------- | ------------------------------------------ |
| 根标签   | `html`            | html文档的根标签                           |
| 头标签   | `head`            | 用于指导html文档的一些属性，引入外部的资源 |
| 标题标签 | `title`           | 该页面标题                                 |
| 体标签   | `body`            | 写主体内容                                 |
| 注解标签 | `<!DOCTYPE html>` | html5中定义该文档是html文档                |
2. **文本标签**：和文本有关的标签
```html
1. 注释：<!-- 注释内容 -->
2. <h1> to <h6> ：标题标签
3. <p>：段落标签
4. <br>：换行标签
5. <hr>：水平线
	属性：
		- color：颜色
		- width：宽度
		- size：高度
		- align：对其方式
			1. center：居中
			2. left：左对齐
			3. right：右对齐
6. <b>：字体加粗
7. <i>：字体斜体
8. <font>：字体标签
9. <center>：文本居中
	属性：
		- color：颜色
		- size：大小
		- face：字体
10. 属性定义：
	- color：
		1. 可以使用颜色的英文单词：red、green、blue
		2. rgb(值1，值2，值3)：值的范围：0~255,。例：rgb(0,0,255)
		3. #值1值2值3：值的范围00~FF之间。例：#FF00FF
	- width：
		1. 数值：width='20'，数值的单位，默认是px(像素)
		2. 数值%：占比相对于父元素的比例
```
3. 图片标签：`img展示图片，属性：src(指定图片的位置)`
4. 列表标签：有序列表`(ol，li)`和无序列表`(ul，li)`
5. 链接标签
```html
a标签：定义一个超链接
	属性：
		1. href：指定访问资源的URL
		2. target：指定打开资源的方式(_self:默认值，在当前页面打开；_blank:在新页面打开)
```
6. div和span
| 类型     | 标签名称 | 作用                |
| -------- | -------- | ------------------- |
| 块级标签 | div      | 每一个div占满一整行 |
| 行内标签 | span     | 文本信息在一行展示  |
7. 语义化标签：html5中为了提高程序的可读性，提供了一些标签。分别是页眉`<header>`和页脚`<footer>`
8. 表格标签
| 标签名称    | 标签作用         | 标签属性                                                     |
| ----------- | ---------------- | ------------------------------------------------------------ |
| table       | 定义表格         | `width：宽度、border：边框、cellpadding：定义内容和单元格的距离、cellspacing：定义单元格之间的距离。如为0,则单元格的线会合为一条、bgcolor：背景色、align：对齐方式` |
| tr          | 定义行           | `bgcolor：背景色、align：对齐方式`                           |
| td          | 定义单元格       | `colspan：合并列、rowspan：合并行`                           |
| `<caption>` | 表格标题         | \                                                            |
| `<thead>`   | 表示表格的头部分 | \                                                            |
| `<tbody>`   | 表示表格的体部分 | \                                                            |
| `<tfoot>`   | 表示表格的脚部分 | \                                                            |
**演示**：
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>表格标签</title>
</head>
<body>
    <table border="1" width="50%" cellpadding="0" cellspacing="0" bgcolor="#faebd7" align="center">
        <tr>
           <!-- <td>编号</td>
            <td>姓名</td>
            <td>成绩</td>-->
            <th>编号</th>
            <th>姓名</th>
            <th>成绩</th>
        </tr>
        <tr>
            <td>1</td>
            <td>小龙女</td>
            <td>100</td>
        </tr>
        <tr>
            <td>2</td>
            <td>杨过</td>
            <td>50</td>
        </tr>
    </table>
    <hr>
    <table border="1" width="50%" cellpadding="0" cellspacing="0" bgcolor="#faebd7" align="center">
        <thead>
            <caption>学生信息表</caption>
            <tr>
                <!-- <td>编号</td>
                 <td>姓名</td>
                 <td>成绩</td>-->
                <th>编号</th>
                <th>姓名</th>
                <th>成绩</th>
            </tr>
        </thead>
        <tbody>
            <tr bgcolor="#7fffd4" align="center">
                <td>1</td>
                <td>小龙女</td>
                <td>100</td>
            </tr>
            <tr>
                <td>2</td>
                <td>杨过</td>
                <td>50</td>
            </tr>
        </tbody>
        <tfoot>
            <tr>
                <td>3</td>
                <td>尹志平</td>
                <td>10</td>
            </tr>
        </tfoot>
    </table>
    <hr>
    <table border="1" width="50%" cellpadding="0" cellspacing="0" bgcolor="#faebd7" align="center">
        <tr>
            <th rowspan="2">编号</th>
            <th>姓名</th>
            <th>成绩</th>
        </tr>
        <tr>
            <td>小龙女</td>
            <td>100</td>
        </tr>
        <tr>
            <td>2</td>
            <td colspan="2">杨过</td>
        </tr>
    </table>
    <hr>
</body>
</html>
```

#### 2.2.1 综合案例演示
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>旅游网</title>
</head>
<body>
    <!--采用table来完成布局-->
    <!--最外层的table，用于整个页面的布局-->
    <table width="100%" align="center">
        <!--第1行-->
        <tr>
            <td>
                <img src="image/top_banner.jpg" width="100%" alt="">
            </td>
        </tr>
        <!--第2行-->
        <tr>
            <td>
                <table width="100%" align="center">
                    <tr>
                        <td>
                            <img src="image/logo.jpg" alt="">
                        </td>
                        <td>
                            <img src="image/search.png" alt="">
                        </td>
                        <td>
                            <img src="image/hotel_tel.png" alt="">
                        </td>
                    </tr>
                </table>
            </td>
        </tr>

        <!--第3行-->
        <tr>
            <td>
                <table width="100%" align="center">
                    <tr bgcolor="#ffd700" align="center" heigth="45">
                        <td>
                            <a href="">首页</a>
                        </td>
                        <td>
                            门票
                        </td>
                        <td>
                            酒店
                        </td>
                        <td>
                            香港车票
                        </td>
                        <td>
                            出境游
                        </td>
                        <td>
                            国内游
                        </td>
                        <td>
                            港澳游
                        </td>
                        <td>
                            抱团定制
                        </td>
                        <td>
                            全球自由行
                        </td>
                        <td>
                            收藏排行榜
                        </td>
                    </tr>
                </table>
            </td>
        </tr>

        <!--第4行-->
        <tr>
            <td>
                <img src="image/banner_3.jpg" alt="" width="100%">
            </td>
        </tr>
        <!--第5行:旅游精选-->
        <tr>
            <td>
                <img src="image/icon_5.jpg" alt="">
                旅游精选
                <hr color="#ffd700">
            </td>
        </tr>
        <!--第6行-->
        <tr>
            <td>
                <table align="center" width="95%">
                    <tr>
                        <td>
                            <img src="image/jiangxuan_1.jpg" alt="">
                            <p>上海飞三亚5天4晚自由行(春节销售+亲子+蜜月行)</p>
                            <font color="red">&yen; 899</font>
                        </td>
                        <td>
                            <img src="image/jiangxuan_1.jpg" alt="">
                            <p>上海飞三亚5天4晚自由行(春节销售+亲子+蜜月行)</p>
                            <font color="red">&yen; 899</font>
                        </td>
                        <td>
                            <img src="image/jiangxuan_1.jpg" alt="">
                            <p>上海飞三亚5天4晚自由行(春节销售+亲子+蜜月行)</p>
                            <font color="red">&yen; 899</font>
                        </td>
                        <td>
                            <img src="image/jiangxuan_1.jpg" alt="">
                            <p>上海飞三亚5天4晚自由行(春节销售+亲子+蜜月行)</p>
                            <font color="red">&yen; 899</font>
                        </td>

                    </tr>
                </table>
            </td>
        </tr>
        <!--第7行：国内游-->
        <tr>
            <td>
                <img src="image/icon_6.jpg" alt="">
                国内游
                <hr color="#ffd700">
            </td>
        </tr>
        <!--第8行-->
        <tr>
            <td>
                <table align="center" width="95%">
                    <tr>
                        <td rowspan="2">
                            <img src="image/guonei_1.jpg" alt="">
                        </td>
                        <td>
                            <img src="image/jiangxuan_2.jpg" alt="">
                            <p>上海飞三亚5天4晚自由行(春节销售+亲子+蜜月行)</p>
                            <font color="red">&yen; 699</font>
                        </td>
                        <td>
                            <img src="image/jiangxuan_2.jpg" alt="">
                            <p>上海飞三亚5天4晚自由行(春节销售+亲子+蜜月行)</p>
                            <font color="red">&yen; 699</font>
                        </td>
                        <td>
                            <img src="image/jiangxuan_2.jpg" alt="">
                            <p>上海飞三亚5天4晚自由行(春节销售+亲子+蜜月行)</p>
                            <font color="red">&yen; 699</font>
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <img src="image/jiangxuan_2.jpg" alt="">
                            <p>上海飞三亚5天4晚自由行(春节销售+亲子+蜜月行)</p>
                            <font color="red">&yen; 699</font>
                        </td>
                        <td>
                            <img src="image/jiangxuan_2.jpg" alt="">
                            <p>上海飞三亚5天4晚自由行(春节销售+亲子+蜜月行)</p>
                            <font color="red">&yen; 699</font>
                        </td>
                        <td>
                            <img src="image/jiangxuan_2.jpg" alt="">
                            <p>上海飞三亚5天4晚自由行(春节销售+亲子+蜜月行)</p>
                            <font color="red">&yen; 699</font>
                        </td>
                    </tr>
                </table>
            </td>
        </tr>
        <!--第9行:境外游-->
        <!--第10行-->
        <tr>
            <td>
                <img src="image/footer_service.png" alt="" width="100%">
            </td>
        </tr>
        <!--第11行-->
        <tr>
            <td align="center" bgcolor="#ffd700" height="40">
                <font color="gray" size="2">
                    浙江零度科技有限公司
                    版权所有 Copyright 2022-2030&copy; All Right Reserved
                </font>
            </td>
        </tr>
    </table>
</body>
</html>
```
