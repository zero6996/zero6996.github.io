---
title: JQuery基础
date: 2019-7-6 17:30
categories: JavaWeb
tags: [JQuery]
description: JQuery基础
urlname: JQueryBasic
---



## 1. JQuery基础

jQuery是一个快速、简洁的JavaScript框架，是继Prototype之后又一个优秀的JavaScript代码库（*或JavaScript框架*）。jQuery设计的宗旨是“write Less，Do More”，即倡导写更少的代码，做更多的事情。它封装JavaScript常用的功能代码，提供一种简便的JavaScript设计模式，**优化HTML文档操作**、事件处理、动画设计和Ajax交互。



<!--more-->



- JavaScript框架：本质上就是一些js文件，封装了js的原生代码而已。



## 2. 快速入门



### 2.1. 下载`JQuery`

目前JQuery有三个大版本：

1.x：兼容ie678，使用最为广泛的，官方只做BUG维护，功能不再新增。对于一般项目来说，使用1.x版本即可。

2.x：不兼容ie768，很少有人使用，官方只做BUG维护，功能不新增。如果不考虑兼容低版本浏览器可以使用2.x。

3.x：不兼容ie678，只支持最新的浏览器。除非特殊要求，一般不会使用3.x版本，很多老的JQuery插件不支持该版本。



> `jquery-xxx.js`与`jquery-xxx.min.js`的区别：
>
> 1. `jquery-xxx.js`：开发版本。供开发人员查看的，有良好的缩进和注释。体积大一些。
> 2. `jquery-xxx.min.js`：生成版本。程序中使用，没有缩进。体积小巧，程序加载更快。



### 2.2. 导入`JQuery`的js文件：导入`min.js`文件

### 2.3. 使用

```javascript
<script src="js/jquery-3.3.1.min.js"></script>
<script>
    var div1 = $("#div1");
	alert(div1.html());
</script>
```



## 3. `JQuery`对象和JS对象区别和转换

1. `JQuery`对象在操作时，更加方便。
2. `JQuery`对象和JS对象方法不通用。
3. 两者可以相互转换
   - `jq --- > js：jq对象[索引] 或者 jq对象.get(索引)`
   - `js --- > jq：$(js对象)`

```javascript
<script>
    // 1. 通过js方式来获取元素对象
    var divs = document.getElementsByTagName("div");
    alert(divs.length); // 可以将其当做数组来使用
    // 将divs中的所有div标签体内容修改为"aaa"
    for(var i = 0;i < divs.length; i++){
        // divs[i].innerHTML = "aaa";
        $(divs[i]).html("ccc"); // 将js对象转换为JQuery对象
    }
    // 2. 通过Jquery方式来获取元素对象
    var $divs = $("div");
    alert($divs);
    // 将$divs中的所有div标签体内容修改为"bbb"
    // $divs.html("bbb");
    // 将JQuery对象转换为js对象
    $divs[0].innerHTML = "ddd";
    $divs[1].innerHTML = "ddd";
</script>
```



## 4. 选择器：筛选具有相似特征的元素(标签)

### 4.1 基本操作学习

#### 1. 事件绑定

```javascript
$("#b1").click(function () {
    alert("abc");
})
```

#### 2. 入口函数

```javascript
$(function () {

});
/*
   window.onload 和 $(function)的区别
       1. window.onload只能定义一次，如定义多次，后边的会将前边的覆盖掉
       2. $(function)可以定义多次
*/
```

#### 3. 样式控制：CSS方法

```javascript
$(function () {
    $("#div1").css("backgroundColor","pink");
})
```

### 4.2 分类

#### 1. 基本选择器

| 选择器名称             | 语法                                                 |
| ---------------------- | ---------------------------------------------------- |
| 标签选择器(元素选择器) | `$("html标签名")` ：获取所有匹配标签名称的元素       |
| ID选择器               | `$("#id的属性值")`：获取与指定id属性值匹配的元素     |
| 类选择器               | `$(.class的属性值)`：获取与指定class属性值匹配的元素 |



```javascript
<script type="text/javascript">
    $(function () {
        // <input type="button" value="改变 id 为 one 的元素的背景色为 红色"  id="b1"/>
        $("#b1").click(function () {
            $("#one").css("backgroundColor","pink");
        });
        // <input type="button" value=" 改变元素名为 <div> 的所有元素的背景色为 红色"  id="b2"/>
        $("#b2").click(function () {
            $("div").css("backgroundColor","pink");
        });
        // <input type="button" value=" 改变 class 为 mini 的所有元素的背景色为 红色"  id="b3"/>
        $("#b3").click(function () {
            $(".mini").css("backgroundColor","pink");
        });
        // <input type="button" value=" 改变所有的<span>元素和 id 为 two 的元素的背景色为红色"  id="b4"/>
        $("#b4").click(function () {
            $("span").css("backgroundColor","pink");
            $("#two").css("backgroundColor","pink");
        });
});
</script>
```

#### 2. 层级选择器

| 选择器名称 | 语法                                     |
| ---------- | ---------------------------------------- |
| 后代选择器 | `$("A B ")`：选择A元素内部的所有B元素    |
| 子选择器   | `$("A > B")`：选择A元素内部的所有B子元素 |

```javascript
<script type="text/javascript">
    $(function () {
        // <input type="button" value=" 改变 <body> 内所有 <div> 的背景色为红色"  id="b1"/>
        $("#b1").click(function () {
            $("body div").css("backgroundColor","pink");
        });
        // <input type="button" value=" 改变 <body> 内子 <div> 的背景色为 红色"  id="b2"/>
        $("#b2").click(function () {
            $("body > div").css("backgroundColor","pink");
        });
	})
</script>
```

#### 3. 属性选择器

| 选择器名称     | 语法                                                  |
| -------------- | ----------------------------------------------------- |
| 属性名称选择器 | `$("A[属性名]")`：选择包含指定属性名称的元素          |
| 属性选择器     | `$("A[属性名 = '值']")`：选择指定属性等于指定值的元素 |
| 复合属性选择器 | `$("A[属性名='值'][]...")`：包含多个属性条件的选择器  |



```javascript
<script type="text/javascript">
    $(function () {
        // <input type="button" value=" 含有属性title 的div元素背景色为红色"  id="b1"/>
        $("#b1").click(function () {
            $("div[title]").css("backgroundColor","pink");
        });
        // <input type="button" value=" 属性title值等于test的div元素背景色为红色"  id="b2"/>
        $("#b2").click(function () {
            $("div[title='test']").css("backgroundColor","pink");
        });
        // <input type="button" value=" 属性title值不等于test的div元素(没有属性title的也将被选中)背景色为红色"  id="b3"/>
        $("#b3").click(function () {
            $("div[title != 'test']").css("backgroundColor","pink");
        });
        // <input type="button" value=" 属性title值 以te开始 的div元素背景色为红色"  id="b4"/>
        $("#b4").click(function () {
            $("div[title ^= 'te']").css("backgroundColor","pink");
        });
        // <input type="button" value=" 属性title值 以est结束 的div元素背景色为红色"  id="b5"/>
        $("#b5").click(function () {
            $("div[title $= 'est']").css("backgroundColor","pink");
        });
        // <input type="button" value="属性title值 含有es的div元素背景色为红色"  id="b6"/>
        $("#b6").click(function () {
            $("div[title*='es']").css("backgroundColor","pink");
        });
        // <input type="button" value="选取有属性id的div元素，然后在结果中选取属性title值含有“es”的 div 元素背景色为红色"  id="b7"/>
        $("#b7").click(function () {
            $("div[id][title*='es']").css("backgroundColor","pink");
        });
	})
</script>
```

#### 4. 过滤选择器

| 选择器名称     | 语法                                       |
| -------------- | ------------------------------------------ |
| 首元素选择器   | `:first`：获得选择的元素中的第一个元素     |
| 尾元素选择器   | `:last`：获得选择的元素中的最后一个元素    |
| 非元素选择器   | `:not(selector)`：不包含指定内容的元素     |
| 偶数选择器     | `:even`：偶数，从0开始计数                 |
| 计数选择器     | `:odd`：奇数，从0开始计数                  |
| 等于索引选择器 | `:eq(index)`：指定索引元素                 |
| 大于索引选择器 | `:gt(index)`：大于指定索引元素             |
| 小于索引选择器 | `:lt(index)`：小于指定索引元素             |
| 标题选择器     | `:header`：获取标题`(h1~h6)`元素，固定写法 |



```javascript
<script type="text/javascript">
    $(function () {
        // <input type="button" value=" 改变第一个 div 元素的背景色为 红色"  id="b1"/>
        $("#b1").click(function () {
            $("div:first").css("backgroundColor","pink");
        });
        // <input type="button" value=" 改变最后一个 div 元素的背景色为 红色"  id="b2"/>
        $("#b2").click(function () {
            $("div:last").css("backgroundColor","pink");
        });
        // <input type="button" value=" 改变class不为 one 的所有 div 元素的背景色为 红色"  id="b3"/>
        $("#b3").click(function () {
            $("div:not(.one)").css("backgroundColor","pink");
        });
        // <input type="button" value=" 改变索引值为偶数的 div 元素的背景色为 红色"  id="b4"/>
        $("#b4").click(function () {
            $("div:even").css("backgroundColor","pink");
        });
        // <input type="button" value=" 改变索引值为奇数的 div 元素的背景色为 红色"  id="b5"/>
        $("#b5").click(function () {
            $("div:odd").css("backgroundColor","pink");
        });
        // <input type="button" value=" 改变索引值为大于 3 的 div 元素的背景色为 红色"  id="b6"/>
        $("#b6").click(function () {
            $("div:gt(3)").css("backgroundColor","pink");
        });
        // <input type="button" value=" 改变索引值为等于 3 的 div 元素的背景色为 红色"  id="b7"/>
        $("#b7").click(function () {
            $("div:eq(3)").css("backgroundColor","pink");
        });
        // <input type="button" value=" 改变索引值为小于 3 的 div 元素的背景色为 红色"  id="b8"/>
        $("#b8").click(function () {
            $("div:lt(3)").css("backgroundColor","pink");
        });
        // <input type="button" value=" 改变所有的标题元素的背景色为 红色"  id="b9"/>
        $("#b9").click(function () {
            $(":header").css("backgroundColor","pink");
        });
	})
</script>
```

#### 5. 表单过滤选择器

| 选择器名称       | 语法                                  |
| ---------------- | ------------------------------------- |
| 可用元素选择器   | `:enabled`：获取可用元素              |
| 不可用元素选择器 | `:disabled`：获取不可用元素           |
| 选中选择器       | `:checked`：获取单选/复选框选中的元素 |
| 选中选择器       | `:selected`：获取下拉框选中的元素     |



```javascript
<script type="text/javascript">
    $(function () {
        // <input type="button" value=" 利用 jQuery 对象的 val() 方法改变表单内可用 <input> 元素的值"  id="b1"/>
        $("#b1").click(function () {
            $("input[type='text']:enabled").val("可用元素");
        });
        // <input type="button" value=" 利用 jQuery 对象的 val() 方法改变表单内不可用 <input> 元素的值"  id="b2"/>
        $("#b2").click(function () {
            $("input[type='text']:disabled").val("不可用元素");
        });
        // <input type="button" value=" 利用 jQuery 对象的 length 属性获取复选框选中的个数"  id="b3"/>
        $("#b3").click(function () {
            var result = $("input[type='checkbox']:checked").length;
            alert(result)
        });
        // <input type="button" value=" 利用 jQuery 对象的 length 属性获取下拉框选中的个数"  id="b4"/>
        $("#b4").click(function () {
            alert($("#job > option:selected").length);
        });
	})
</script>
```



## 5. DOM操作

### 5.1 内容操作

1. `html()`：获取/设置元素的标签体内容
2. `text()`：获取/设置元素的标签体纯文本内容
3. `val()`：获取/设置元素的value属性值

```javascript
<script>
    $(function () {
        // 获取myinput 的value值
        alert($("#myinput").val());
        // 设置值
        alert($("#myinput").val("李四"));
        // 获取mydiv的标签体内容
        alert($("#mydiv").html());
        // 设置值
        alert($("#mydiv").html("<h4>hello</h4>"));
        // 获取mydiv纯文本内容
        alert($("#mydiv").text());
        // 设置值
        alert($("#mydiv").text("aaa"));
	})
</script>
```



### 5.2 属性操作

#### 1. 通用属性操作

1. `attr()`：获取/设置元素的属性
2. `removeAttr()`：删除属性
3. `prop()`：获取/设置元素的属性
4. `removeProp()`：删除属性

```javascript
<script type="text/javascript">
    $(function () {
        //获取北京节点的name属性值
        alert($("#bj").attr("name"));
        //设置北京节点的name属性的值为dabeijing
        $("#bj").attr("name","dabeijing");
        //新增北京节点的discription属性 属性值是didu
        $("#bj").attr("discription","didu");
        //删除北京节点的name属性并检验name属性是否存在
        $("#bj").removeAttr("name");
        //获得hobby的的选中状态
        alert($("#hobby").prop("checked"));
	})
</script>
```

> `attr`和`prop`区别？
>
> 1. 如果操作的是元素的固有属性，则建议使用`prop`
> 2. 如果操作的是元素自定义的属性，则建议使用`attr`

#### 2. 对class属性操作

1. `addClass()`：添加class属性值
2. `removeClass()`：删除class属性值
3. `toggleClass()`：切换class属性值
4. `css()`：操作元素css样式



```javascript
<script type="text/javascript">
    $(function () {
        //<input type="button" value="采用属性增加样式(改变id=one的样式)"  id="b1"/>
        $("#b1").click(function () {
            $("#one").prop("class","second");
        });
        //<input type="button" value=" addClass"  id="b2"/>
        $("#b2").click(function () {
            $("#one").addClass("second");
        });
        //<input type="button" value="removeClass"  id="b3"/>
        $("#b3").click(function () {
            $("#one").removeClass("second");
        });
        //<input type="button" value=" 切换样式"  id="b4"/>
        $("#b4").click(function () {
            $("#one").toggleClass("second");
        });
        //<input type="button" value=" 通过css()获得id为one背景颜色"  id="b5"/>
        $("#b5").click(function () {
            alert($("#one").css("backgroundColor"));
        });
        //<input type="button" value=" 通过css()设置id为one背景颜色为绿色"  id="b6"/>
        $("#b6").click(function () {
            $("#one").css("backgroundColor","blue");
        });
	})
</script>
```



### 5.3 CRUD操作

| 方法名称         | 作用                     | 例子                                                         |
| ---------------- | ------------------------ | ------------------------------------------------------------ |
| `append()`       | 父元素将子元素追加到末尾 | `对象1.append(对象2)`：将对象2添加到对象1元素内部末尾处。    |
| `prepend()`      | 父元素将子元素追加到开头 | `对象1.prepend(对象2)`：将对象2添加到对象1元素内部开头处。   |
| `appendTo()`     |                          | `对象1.appendTo(对象2)`：将对象1添加到对象2内部末尾处。      |
| `prependTo()`    |                          | `对象1.prependTo(对象2)`：将对象1添加到对象2内部开头处。     |
| `after()`        | 添加元素到元素后边       | `对象1.after(对象2)`：将对象2添加到对象1后边。对象1和2是兄弟关系。 |
| `before()`       | 添加元素到元素前边       | `对象1.before(对象2)`：将对象2添加到对象1前边。对象1和2是兄弟关系。 |
| `insertAfter()`  |                          | `对象1.insertAfter(对象2)`：将对象1添加到对象2后边。对象1和2是兄弟关系。 |
| `insertBefore()` |                          | `对象1.insertBefore(对象2)`：将对象1添加到对象2前边。对象1和对象2是兄弟关系。 |
| `remov()`        | 移除元素                 | `对象.remove()`：将对象删除                                  |
| `empty()`        | 清空元素的所有后代元素   | `对象.empty()`：将对象的后代元素全部清空，但保留当前对象及其属性节点。 |



```javascript
<script type="text/javascript">
    $(function () {
        // <input type="button" value="将反恐放置到city的后面"  id="b1"/>
        $("#b1").click(function () {
            // append
            // $("#city").append($("#fk"));
            // appendTo
            $("#fk").appendTo($("#city"));
        });
        // <input type="button" value="将反恐放置到city的最前面"  id="b2"/>
        $("#b2").click(function () {
            // prepend
            $("#fk").prependTo($("#city"));
        });
        // <input type="button" value="将反恐插入到天津后面"  id="b3"/>
        $("#b3").click(function () {
            // after
            // $("#tj").after($("#fk"));
            // insertAfter
            $("#fk").insertAfter($("#tj"));
        });
        // <input type="button" value="将反恐插入到天津前面"  id="b4"/>
        $("#b4").click(function () {
            $("#fk").insertBefore($("#tj"));
        });
        // <input type="button" value="删除<li id='bj' name='beijing'>北京</li>"  id="b1"/>
        $("#b1").click(function () {
            $("#bj").remove();
        });
        // <input type="button" value="删除city所有的li节点   清空元素中的所有后代节点(不包含属性节点)"  id="b2"/>
        $("#b2").click(function () {
            $("#city").empty();
        });
	})
</script>
```



## 小案例

### 1. 隔行换色

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<script  src="../../js/jquery-3.3.1.min.js"></script>
		
		<script>
			//需求：将数据行的奇数行背景色设置为 pink，偶数行背景色设置为 yellow
			$(function () {
				// 1. 获取数据行的奇数行的tr，设置背景色为pink
				$("tr:gt(1):odd").css("backgroundColor","pink")
				// 2. 获取数据行的偶数行的tr，设置背景色为yellow
				$("tr:gt(1):even").css("backgroundColor","yellow")
			});
		</script>
	</head>
	<body>
		<table id="tab1" border="1" width="800" align="center" >
			<tr>
				<td colspan="5"><input type="button" value="删除"></td>
			</tr>
			<tr style="background-color: #999999;">
				<th><input type="checkbox"></th>
				<th>分类ID</th>
				<th>分类名称</th>
				<th>分类描述</th>
				<th>操作</th>
			</tr>
			<tr>
				<td><input type="checkbox"></td>
				<td>0</td>
				<td>手机数码</td>
				<td>手机数码类商品</td>
				<td><a href="">修改</a>|<a href="">删除</a></td>
			</tr>
			<tr>
				<td><input type="checkbox"></td>
				<td>1</td>
				<td>电脑办公</td>
				<td>电脑办公类商品</td>
				<td><a href="">修改</a>|<a href="">删除</a></td>
			</tr>
			<tr>
				<td><input type="checkbox"></td>
				<td>2</td>
				<td>鞋靴箱包</td>
				<td>鞋靴箱包类商品</td>
				<td><a href="">修改</a>|<a href="">删除</a></td>
			</tr>
			<tr>
				<td><input type="checkbox"></td>
				<td>3</td>
				<td>家居饰品</td>
				<td>家居饰品类商品</td>
				<td><a href="">修改</a>|<a href="">删除</a></td>
			</tr>
		</table>
	</body>
</html>
```

### 2. 全选/全不选

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<script  src="../../js/jquery-3.3.1.min.js"></script>
		<script>
			// 需要保证下边的选中状态和第一个复选框的状态一致即可。
			function selectAll(obj) {
				// 获取下边的所有复选框
				$(".itemSelect").prop("checked",obj.checked);
			}
		</script>
	</head>
	<body>
		<table id="tab1" border="1" width="800" align="center" >
			<tr>
				<td colspan="5"><input type="button" value="删除"></td>
			</tr>
			<tr>
				<th><input type="checkbox" onclick="selectAll(this)" ></th>
				<th>分类ID</th>
				<th>分类名称</th>
				<th>分类描述</th>
				<th>操作</th>
			</tr>
			<tr>
				<td><input type="checkbox" class="itemSelect"></td>
				<td>1</td>
				<td>手机数码</td>
				<td>手机数码类商品</td>
				<td><a href="">修改</a>|<a href="">删除</a></td>
			</tr>
			<tr>
				<td><input type="checkbox" class="itemSelect"></td>
				<td>2</td>
				<td>电脑办公</td>
				<td>电脑办公类商品</td>
				<td><a href="">修改</a>|<a href="">删除</a></td>
			</tr>
			<tr>
				<td><input type="checkbox" class="itemSelect"></td>
				<td>3</td>
				<td>鞋靴箱包</td>
				<td>鞋靴箱包类商品</td>
				<td><a href="">修改</a>|<a href="">删除</a></td>
			</tr>
			<tr>
				<td><input type="checkbox" class="itemSelect"></td>
				<td>4</td>
				<td>家居饰品</td>
				<td>家居饰品类商品</td>
				<td><a href="">修改</a>|<a href="">删除</a></td>
			</tr>
		</table>
	</body>
</html>
```

### 3. qq表情选择

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>QQ表情选择</title>
	 <script  src="../../js/jquery-3.3.1.min.js"></script>
    <style type="text/css">
    *{margin: 0;padding: 0;list-style: none;}

    .emoji{margin:50px;}
    ul{overflow: hidden;}
    li{float: left;width: 48px;height: 48px;cursor: pointer;}
    .emoji img{ cursor: pointer; }
    </style>
	<script>
        //需求：点击qq表情，将其追加到发言框中
        $(function () {
            // 1. 给img图片添加onclick事件
            $("ul img").click(function () {
                // 2. 追加到p标签中即可,$(this)：将js对象转换为JQuery对象
                $(".word").append($(this).clone());
            });

        });
    </script>
	
</head>
<body>
    <div class="emoji">
        <ul>
            <li><img src="img/01.gif" height="22" width="22" alt="" /></li>
            <li><img src="img/02.gif" height="22" width="22" alt="" /></li>
            <li><img src="img/03.gif" height="22" width="22" alt="" /></li>
            <li><img src="img/04.gif" height="22" width="22" alt="" /></li>
            <li><img src="img/05.gif" height="22" width="22" alt="" /></li>
            <li><img src="img/06.gif" height="22" width="22" alt="" /></li>
            <li><img src="img/07.gif" height="22" width="22" alt="" /></li>
            <li><img src="img/08.gif" height="22" width="22" alt="" /></li>
            <li><img src="img/09.gif" height="22" width="22" alt="" /></li>
            <li><img src="img/10.gif" height="22" width="22" alt="" /></li>
            <li><img src="img/11.gif" height="22" width="22" alt="" /></li>
            <li><img src="img/12.gif" height="22" width="22" alt="" /></li>
        </ul>
        <p class="word">
            <strong>请发言：</strong>
            <img src="img/12.gif" height="22" width="22" alt="" />
        </p>
    </div>
</body>
</html>
```



### 4. 多选下拉列表左右移动

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<script  src="../../js/jquery-3.3.1.min.js"></script>
		<style>
			#leftName , #btn,#rightName{
				float: left;
				width: 100px;
				height: 300px;
			}
			#toRight,#toLeft{
				margin-top:100px ;
				margin-left:30px;
				width: 50px;
			}

			.border{
				height: 500px;
				padding: 100px;
			}
		</style>
		<script>
			//需求：实现下拉列表选择条目左右选择功能
			$(function () {
				$("#toRight").click(function () {
					// 获取右边的下拉列表对象，append(左边下拉列表选中的option)
					$("#rightName").append($("#leftName > option:selected"));
				});
				$("#toLeft").click(function () {
					$("#leftName").append($("#rightName > option:selected"));
				});
			});
		</script>
	</head>
	<body>
		<div class="border">
			<select id="leftName" multiple="multiple">
				<option>张三</option>
				<option>李四</option>
				<option>王五</option>
				<option>赵六</option>
			</select>
			<div id="btn">
				<input type="button" id="toRight" value="-->"><br>
				<input type="button" id="toLeft" value="<--">
			</div>
			<select id="rightName" multiple="multiple">
				<option>钱七</option>
			</select>
		</div>
	</body>
</html>
```

