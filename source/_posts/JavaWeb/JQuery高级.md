---
title: JQuery高级
date: 2019-7-9 23:50
categories: JavaWeb
tags: [JQuery]
description: JQuery进阶内容
urlname: JQueryHigh
---



## JQuery高级
JQuery进阶内容，包括图片显示和隐藏方式、遍历方式、事件绑定方法以及自定义插件。

<!--more-->

## 1. 动画

主要有三种方式显示和隐藏元素

### 1.1 默认显示和隐藏方式

1. `show([speed,[easing],[fn]])`
2. `hide([speed,[easing],[fn]])`
3. `toggle([speed,[easing],[fn]])`



**通用参数概述**：

1. `speed`：动画的速度。三个预定义的值`("slow","normal","fast")`或表示动画时长的毫秒数值(如：1000)
2. `easing`：用来指定切换效果，默认是`"swing"`(动画执行时效果是：先慢，中间快，最后再慢)，可用参数`"linear"`(动画执行时速度是匀速的)
3. `fn`：在动画完成时执行的函数，每个元素执行一次。

### 1.2 滑动显示和隐藏方式

1. `slideDown([speed,[easing],[fn]])`
2. `slideUp([speed,[easing],[fn]])`
3. `slideToggle([speed,[easing],[fn]])`

### 1.3 淡入淡出显示和隐藏方式

1. `fadeIn([speed,[easing],[fn]])`
2. `fadeOut([speed,[easing],[fn]])`
3. `fadeToggle([speed,[easing],[fn]])`



### 示例

```javascript
<script>
    // 隐藏div
    function hideFn() {
    // 默认方式
    // $("#showDiv").hide(5000,"swing");
    // 滑动方式
    // $("#showDiv").slideUp("slow");
    // 淡入淡出方式
    $("#showDiv").fadeOut("slow");
}
// 显示div
function showFn() {
    // 默认方式
    // $("#showDiv").show("slow","swing");
    // 滑动方式
    // $("#showDiv").slideDown("slow");
    // 淡入淡出方式
    $("#showDiv").fadeIn("slow");
}
// 切换显示和隐藏div
function toggleFn() {
    // 默认方式
    // $("#showDiv").toggle("slow");
    // 滑动方式
    // $("#showDiv").slideToggle("slow");
    // 淡入淡出方式
    $("#showDiv").fadeToggle("slow");
}
</script>
```



## 2. 遍历

### 2.1 JS的遍历方式

- `for(初始值;循环结束条件;步长)`

### 2.2 JQuery的遍历方式

#### 1. `JQ对象.each(callback)`

- 语法：`jquery对象.each(function(index,element){});`
  - `index`：就是元素在集合中的索引
  - `element`：就是集合中的每一个元素对象
  - `this`：集合中的每一个元素对象（当前元素对象）
- 回调函数返回值
  - `true`：如果当前`function`返回`false`，则结束循环`(相当于break)`。
  - `false`：如果当前`function`返回`true`，则结束本次循环，继续下次循环`(相当于continue)`。

#### 2. `$.each(object,[callback])`

#### 3. `for..of`

3.0版本之后提供的方式，使用方法：`for(元素对象 of 容器对象)`

### 示例

```javascript
<script type="text/javascript">
    $(function () {
    // 1. js方法遍历
    // 获取所有ul下的li
    var citys = $("#city li");
    // 遍历li
    for (var i = 0; i < citys.length; i++){
        if ("上海" == citys[i].innerHTML ){
            break;
            continue;
        }
        alert(i+":"+citys[i].innerHTML);
    }
    
    // 2. jq方法遍历，JQ对象.each(callback)
    citys.each(function (index,element) {
        // 获取li对象,使用this
        // alert(this.innerHTML);
        // 获取li对象，在回调函数中定义参数，index,element
        // 判断如果是上海，则结束循环
        if ("上海" == $(element).html()){
            // 如果当前function返回为false，则结束循环(break);
            // 如果返回为true，则结束本次循环，继续下次循环(continue);
            return true;
        }
        alert(index+":"+$(element).html());
    })

    // 3. $.each(object,[callback]);
    $.each(citys,function () {
        alert($(this).html());
    })
    
    // 4. for..of(jquery3.0之后提供的方式)
    for (li of citys){
        alert(li.innerHTML);
    }
})
</script>
```



## 3. 事件绑定

### 3.1 `JQuery标准的绑定方式`

- `JQuery对象.事件方法(回调函数)`

> 如果调用事件方法，不传递回调函数，则会触发浏览器默认行为。

- `表单对象.submit();`  ： 让表单提交



```javascript
<script type="text/javascript">
    $(function () {
    // 1. 获取name对象，表单click事件
    $("#name").click(function () {
        alert("我被点击了")
    });
    // 给name绑定聚焦离焦事件
    $("#name").mouseover(function () {
        alert("鼠标来了...")
    })
    $("#name").mouseout(function () {
        alert("鼠标走了...")
    })
    // 简化操作，链式编程
    $("#name").mouseover(function () {
        alert("鼠标来了...");
    }).mouseout(function () {
        alert("鼠标走了")
    });
    $("#name").focus(); // 浏览器默认行为：让文本输入框获得焦点
})
</script>
```

### 3.2 `on绑定事件/off解除绑定`

- `JQuery对象.on("事件名称",回调函数)`
- `JQuery对象.off("事件名称")`
  - 如果`off`方法不传递任何参数，则会将组件上的所有事件全部解绑。



```javascript
<script type="text/javascript">
     $(function () {
     // 1. 使用on绑定点击事件
     $("#btn").on("click",function () {
         alert("我被点击了...")
     })
     $("#btn2").click(function () {
         alert("解除了单击事件...")
         // 解绑btn单击事件
         // $("#btn").off("click");
         $("#btn").off();// 将组件上的所有事件全部解绑
     })
 })
</script>
```



### 3.3 事件切换：`toggle`

- `JQuery对象.toggle(fn1,fu2...)`：当单击JQ对象对应的组件后，会执行`fn1`，第二次点击会执行`fn2...`

> Notice：1.9版本之后`.toggle()`方法删除，使用`JQuery Migrate`(迁移)插件可恢复此功能。

```java
<script src="../js/jquery-migrate-1.0.0.js" type="text/javascript" charset="utf-8"></script>
<script type="text/javascript">
    $(function () {
    // 获取按钮，调用toggle方法
    $("#btn").toggle(function () {
        $("#myDiv").css("backgroundColor","green");
    },function () {
        $("#myDiv").css("backgroundColor","pink");
    })
})
</script>
```



## 4. 综合案例

#### 4.1 广告图片自动显示与隐藏

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>广告的自动显示与隐藏</title>
    <style>
        #content{width:100%;height:500px;background:#999}
    </style>
    <!--引入jquery-->
    <script type="text/javascript" src="../js/jquery-3.3.1.min.js"></script>
    <script>
        /*
            需求：
                1. 当页面加载完，3秒后，自动显示广告。
                2. 广告显示5秒后，自动消失。
         */
        // 1. 使用定时器来完成。setTimeout(执行一次定时器)
        // 2. 通过控制display属性来实现显示和隐藏
        $(function () {
            // 定义定时器，调用adShow方法 3秒后执行一次
            setTimeout(adShow,3000);
            // 定义定时器，调用adHide方法，8秒后执行一次
            setTimeout(adHide,8000);
        })
        // 显示广告
        function adShow() {
            // 获取div，调用显示方法
            $("#ad").show("slow");
        }
        // 隐藏广告
        function adHide() {
            $("#ad").hide("slow");
        }
    </script>
</head>
<body>
<!-- 整体的DIV -->
<div>
    <!-- 广告DIV -->
    <div id="ad" style="display: none;">
        <img style="width:100%" src="../img/adv.jpg" />
    </div>

    <!-- 下方正文部分 -->
    <div id="content">
        正文部分
    </div>
</div>
</body>
</html>
```

#### 4.2 抽奖

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>jquery案例之抽奖</title>
    <script type="text/javascript" src="../js/jquery-3.3.1.min.js"></script>
    <script>
        /*
            分析：
                1. 给开始按钮绑定单击事件
                    1.1 定义循环定时器
                    1.2 切换小相框的src属性
                        - 定义数组，存放图片资源路径
                        - 生成随机数，数组索引
                2. 给结束按钮绑定单击事件
                    1.1 停止定时器
                    1.2 给大相框设置src属性
                        - 等于小相框当前src属性
         */
        // 定义图片路径数组
        var imgs = ["../img/man00.jpg",
                    "../img/man01.jpg",
                    "../img/man02.jpg",
                    "../img/man03.jpg",
                    "../img/man04.jpg",
                    "../img/man05.jpg",
                    "../img/man06.jpg"]
        var startId;
        var index; // 随机角标
        $(function () {
            // 处理按钮是否可以使用效果
            $("#startID").prop("disabled",false);
            $("#stopID").prop("disabled",true);

            // 1. 给开始按钮绑定单击事件
            $("#startID").click(function () {
                // 1.1 定义循环定时器， 20毫秒执行一次
                startId = setInterval(function () {
                    // 处理按钮是否可以使用效果
                    $("#startID").prop("disabled",true);
                    $("#stopID").prop("disabled",false);

                    // 1.2 生成随机角标 0-6
                    index = Math.floor(Math.random() * 7);  // 0.000~0.999 --> *7 => 0.0~6.9999
                    // 1.3 小相框src属性
                    $("#img1ID").prop("src",imgs[index]);
                },20);
            })
            // 2. 给结束按钮绑定单击事件
            $("#stopID").click(function () {
                // 处理按钮是否可以使用效果
                $("#startID").prop("disabled",false);
                $("#stopID").prop("disabled",true);
                // 2.1 停止定时器
                clearInterval(startId);
                // 2.2 给大相框设置src属性
                $("#img2ID").prop("src",imgs[index]).hide();
                // 1秒之后显示
                $("#img2ID").show(1000);
            })
        })
    </script>
</head>
<body>

<!-- 小像框 -->
<div style="border-style:dotted;width:160px;height:100px">
    <img id="img1ID" src="../img/man00.jpg" style="width:160px;height:100px"/>
</div>

<!-- 大像框 -->
<div
        style="border-style:double;width:800px;height:500px;position:absolute;left:500px;top:10px">
    <img id="img2ID" src="../img/man00.jpg" width="800px" height="500px"/>
</div>
<!-- 开始按钮 -->
<input
        id="startID"
        type="button"
        value="点击开始"
        style="width:150px;height:150px;font-size:22px"
>
<!-- 停止按钮 -->
<input
        id="stopID"
        type="button"
        value="点击停止"
        style="width:150px;height:150px;font-size:22px"
>
</body>
</html>
```

## 5. 插件

主要用于增强JQuery的功能，实现方式：

- `$.fn.extend(object)`：增强通过JQuery获取的对象的功能，`$("#id")`

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>01-jQuery对象进行方法扩展</title>
    <script src="../js/jquery-3.3.1.min.js" type="text/javascript" charset="utf-8"></script>
    <script type="text/javascript">
        // 1. 定义JQuery的对象插件
        $.fn.extend({
            // 定义了一个check()方法。所有的jq对象都可以调用该方法
            check:function () {
                // 让复选框选中
                // this:调用该方法的jq对象
                this.prop("checked",true);
            },
            uncheck:function () {
              // 不选中
                this.prop("checked",false);
            }
        });

        $(function () {
            $("#btn-check").click(function () {
                $("input[type='checkbox']").check();
            });
            $("#btn-uncheck").click(function () {
                $("input[type='checkbox']").uncheck();
            });
        })
    </script>
</head>
<body>
<input id="btn-check" type="button" value="点击选中复选框" >
<input id="btn-uncheck" type="button" value="点击取消复选框选中" >
<br/>
<input type="checkbox" value="football">足球
<input type="checkbox" value="basketball">篮球
<input type="checkbox" value="volleyball">排球

</body>
</html>
```

- `$.extend(object)`：增强JQuery对象自身的功能，`$/jQuery`

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>01-jQuery对象进行方法扩展</title>
    <script src="../js/jquery-3.3.1.min.js" type="text/javascript" charset="utf-8"></script>
    <script type="text/javascript">
        //对全局方法扩展2个方法，扩展min方法：求2个值的最小值；扩展max方法：求2个值最大值
        $.extend({
            max:function (a,b) {
                // 返回大值
                return a >= b ? a:b;
            },
            min:function (a,b) {
                // 返回小值
                return a <= b ? a:b;
            }
        });
        // 调用全局方法
        alert($.max(4,3));
        alert($.min(4,3));

    </script>
</head>
<body>
</body>
</html>
```

