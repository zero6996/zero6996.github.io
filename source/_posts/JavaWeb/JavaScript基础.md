---
title: JavaScript基础
date: 2019-5-28 23:30
categories: JavaWeb
tags: [JavaScript]
description: JavaScript基础内容
---



## 1. JavaScript概述

JavaScript是一种运行在浏览器中的解释型的编程语言。每一个浏览器都有JavaScript的解析引擎，不需要编译就可以直接被浏览器解析执行了。



<!--more-->



- 功能：主要用来增强用户和html页面的交互过程，可以来控制html元素，让页面有一些动态的效果，增强用户的体验。



## 2. ECMAScript

1997年，ECMA(欧洲计算机制造商协会)，制定出**客户端脚本语言的标准**：`ECMAScript`，就是统一了所有客户端脚本语言的编码方式。

- `JavaScript = ECMAScript + JavaScript自己特有的东西(BOM+DOM)`

### 2.1 基本语法

#### 2.1.1 与html结合方式

1. 内部JS：定义`<script>`，标签体内容就是js代码
2. 外部JS：定义`<script>`，通过`src`属性引入外部的js文件

> Tips: `script`可以定义在html页面的任何地方，但定义的文字会影响执行顺序。且可以定义多个。

#### 2.1.2 注释

1. 单行注释：`// 注释内容`
2. 多行注释：`/*注释内容*/`

#### 2.1.3 数据类型

1. 原始数据类型
   1. `number`：`JavaScript`不区分整数和浮点数，统一用`number`表示。可以表示整数/小数/NaN(NaN表示Not a Number，当无法计算结果时用NaN表示)。
   2. `string`：字符串。可以使用单双引号。
   3. `boolean`：布尔值。true和false。
   4. `null`：一个对象为空的占位符。
   5. `undefined`：表示值未定义。
2. 引用数据类型：对象，`JavaScript`的对象是一组由键-值组成的无序集合。

```javascript
<script>
        var num = 123; // 定义一个整数123
        var float = 0.456; // 定义一个浮点数0.456
        var str = "abc"; // 定义一个字符串abc
        var flag = true; // 定义一个布尔值true
        var a = null; // 定义对象a为空
        var b = undefined; // 表示值b未定义
        var person = { // 定义了一个person对象
            name:'xiaozhang',
            age:20,
            city:'hangzhou'
        }
</script>
```

#### 2.1.4 变量

变量是一小块存储数据的内存空间。Java语言是强类型语言，而JavaScript是弱类型语言。

- 强类型：在开辟变量存储空间时，定义了空间将来存储的数据的数据类型。只能存储固定类型的数据
- 弱类型：在开辟变量存储空间时，不定义空间将来的存储数据类型，可以存放任意类型的数据。
- 语法：`var 变量名 = 初始化值;`

> Tips: 可以使用`typeof`运算符获取变量的类型。null运算后得到的是object类型。

```javascript
document.write(num+"-->"+typeof(num)+"<br>"); // 123-->number
document.write(str+"-->"+typeof(str)+"<br>"); // abc-->string
document.write(flag+"-->"+typeof(flag)+"<br>"); // true-->boolean
document.write(a+"-->"+typeof(a)+"<br>"); // null-->object
document.write(b+"-->"+typeof(b)+"<br>"); // undefined-->undefined
document.write("person-->"+typeof(person)+"<br>"); // person-->object
```

#### 2.1.5 运算符

1. 一元运算符：只有一个运算数的运算符。例：`++,--`
2. 算数运算符：`+ - * / % ...`
3. 赋值运算符：`=,+=,-=...`
4. 比较运算符：`>,<,>=,<=,==,===(全等于)`，全等于在比较之前先判断类型，如果类型不一样，直接返回false。
5. 逻辑运算符：`&&,||,!`。
6. 三元运算符：`表达式?值1:值2;`，判断表达式的值，如果是true则取值1，反之取值2。

```javascript
var a = 1;
document.write(++a+"<br>"); // 一元运算符
var b = 2;
document.write(a+b+"<br>"); // 二元运算符: +-*/%
document.write(b+=1+"<br>"); // 3, 赋值运算符
// document.write(a>b+"<br>"); // false,比较运算符
var str = '1';
document.write((a==str)+"<br>"); // true，string类型转number，按照字面值转换为1，1==1，故true
document.write(a===str+"<br>"); // false,全等于会先进行类型判断，类型不同直接返回false
document.write(a>false); // a>0 , true
document.write(a>0?1:0); // 三元运算符
/*
在JS中，如果运算数不是运算符所要求的类型，那么js引擎会自动的将运算数进行类型转换
* 其他类型转number：
     * string转number：按照字面值转换。如果字面值不是数字，则转为NaN（不是数字的数字）
     * boolean转number：true转为1，false转为0
逻辑运算符类型转换：
	1. number：0或NaN为假，其他都为真
	2. string：除了空字符串("")，其他都是true
	3. null&undefined:都是false
	4. 对象：所有对象都是true

*/
```

#### 2.1.6 流程控制语句

1. `if...else...`
2. `switch(变量)：case 值:具体内容;break;`，在JS中，switch语句可以接受任意的原始数据类型
3. `while`
4. `do...while`
5. `for`

```javascript

var a = 4;
var b = 7;
if (a>b){
    document.write("a>b")
} else{
    document.write("b>a")
}
var day = 3;
switch (day) {
    case 1:
        document.write("Monday");
        break;
    case 2:
        document.write("Tuesday");
        break;
    case 3:
        document.write("Wednesday");
        break;
    case 4:
        document.write("Thursday");
        break;
    case 5:
        document.write("Friday");
        break;
    case 6:
        document.write("Saturday");
        break;
    case 7:
        document.write("Sunday");
        break;

}
var c = 1;
while (c<=5){
    document.write(c+"<br>");
    c++;
}
for (i=1;i<=5;i++){
    document.write(i+"<br>")
}
```



#### 2.1.7 JS特殊语法

1. 语句以`;`结尾，如果一行只有一条语句则可省略`;`(不建议)
2. 变量的定义使用`var`关键字，也可以不使用。
   1. 使用：定义的变量是局部变量
   2. 不用：定义的变量是全局变量(不建议)



#### 2.1.8 小练习：99乘法表

```javascript
document.write("<table>")
for (i=1;i<=9;i++){
    document.write("<tr>")
    for (j=1;j<=i;j++){
        document.write("<td>")
        document.write(j+"*"+i+"="+j*i)
        document.write("</td>")
    }
    document.write("</tr>")
}
document.write("</table>")
```



## 3. 基本对象

### 3.1 Function：函数对象

```javascript
    <script>
        /*
            1.创建函数对象：
                1. function 方法名称(形参列表){
                    方法体
                }
                2. var 方法名 = function(形参列表){
                    方法体
                }
            2. 属性：
                length：返回形参的个数
            3. 特点：
                1. 方法定义时形参的类型不用写
                2. 方法是一个对象，如果定义名称相同的方法，会覆盖
                3. 在JS中，方法的调用只与方法名称有关
                4. 在方法声明中有一个隐藏的内置关键字：argument,用于接收所有形参，封装成了一个数组
            4. 调用：
                方法名称(参数列表);
         */
        function fun(a,b) {
            document.write(a+b);
        }
        // fun(3,4); // 7
        var fun2 = function (a,b) {
            document.write(a+b);
        }
        document.write(fun2.length); // 2
        fun2(3) // NaN,原因b是undefined
        /*
            定义一个求和函数
         */
        function add() {
            var sum = 0;
            for (i=0;i<arguments.length;i++){
                sum+=arguments[i];
            }
            return sum;
        }
        document.write(add(1,2,3,4,5)); // 15
    </script>
```

### 3.2 Array：数组对象

#### 3.2.1 创建

1. `var arr = new Array(元素列表);`
2. `var arr = new Array(默认长度);`
3. `var arr = [元素列表];`

#### 3.2.2 方法

1. `join(参数)`：将数组中的元素按照指定的分隔符拼接为字符串
2. `push()`：向数组的末尾添加一个或更多元素，并返回新的长度。

#### 3.2.3 属性

`length`：数组长度

#### 3.2.4 特点

1. 在JS中，数组可以存储不同的类型元素
2. JS中，数组长度是可变的。

#### 3.2.5 示例

```javascript
var arr1 = new Array(1,2,3);
var arr2 = new Array(5);
var arr3 = [1,"abc",true]; // 数组可以存储不同的类型元素
document.write(arr1.join("-->")); // 1-->2-->3
arr1.push(4,5);
document.write(arr1); // 1,2,3,4,5
document.write(arr2.length); // 5
```

### 3.3 Date：日期对象

1. 创建：`var date = new Date();`
2. 方法：
   - `toLocaleString()`：返回当前date对象对应的时间本地字符串格式。
   - `getTime()`：获取毫秒值。返回当前日期对象描述的时间到1970年1月1日零点的毫秒值差。

### 3.4 Math：数学对象

1. 创建：Math对象不用创建，可以直接使用。例：`Math.方法名();`
2. 方法：
   - `random()`：返回0~1之间的随机数。含0不含1
   - `ceil(x)`：对数进行上舍入(向上取整)
   - `floor(x)`：对数进行下舍入(向下取整)
   - `round(x)`：把数四舍五入为最接近的整数
3. 属性：`PI`

```javascript
document.write(Math.PI+"<br>"); // 3.141592653589793
document.write(Math.random()+"<br>"); // 0.16764969291724752
document.write(Math.ceil(3.14)+"<br>"); // 4
document.write(Math.floor(3.14)+"<br>"); // 3
document.write(Math.round(3.14)+"<br>"); // 3
// 取一个0~100之间的随机整数
var number = Math.floor(Math.random()*100+1);
document.write(number);
```

### 3.5 RegExp：正则表达式对象

1. 创建：
   - `var reg = new RegExp("正则表达式");`
   - `var reg = /正则表达式/;`
2. 方法：test(参数)：验证指定的字符串是否符合正则定义的规范

```javascript
var reg1 = new RegExp("\\d");
var reg2 = /[a-z]/;
document.write(reg1.test("a")); // false
document.write(reg1.test(2)); // true
document.write(reg2.test("b")); // true
```

### 3.6 Global

#### 3.6.1 特点

全局对象，这个Global中封装的方法不需要对象就可以直接调用。例：`方法名();`

#### 3.6.2 方法

- `encodeURI()`：url编码
- `encodeURI()`：url编码
- `encodeURIComponent()`：url编码，能编码的字符更多
-  `decodeURIComponent()`：url解码
- `parseInt()`：将字符串转为数字。逐一判断每一个字符是否是数字，直到不是数字为止，将前边数字部分转为number类型。
- `isNaN()`：判断一个值是否是NaN。NaN参与的==比较全部为false。
- `eval()`：将可执行的`JavaScript `字符串转换为JS脚本来执行。

```javascript
var str = "零度科技"; // UTF8编码转换中文，一个中文代表3个字节,即'零= %E9%9B%B6'
var encode = encodeURI(str);
document.write(encode); // %E9%9B%B6%E5%BA%A6%E7%A7%91%E6%8A%80
document.write(decodeURI(encode)); // 零度科技
var number = "123木头人321";
document.write(parseInt(number)); // 123
var a = NaN;
document.write(isNaN(number)); // false
document.write(isNaN(a)); // true
var b = "alert(123)";
eval(b); // 将会执行alert(123)这段js代码，在浏览器弹出123
```

