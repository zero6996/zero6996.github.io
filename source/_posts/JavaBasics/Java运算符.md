---
title: Java运算符
date: 2019-4-13 14:00
categories: JavaBasics # 分类
tags: [Java]
description: 该文章记录了Java的运算符操作
urlname: operator
---

### 算数运算符

* `+` ：加法运算，字符串连接运算
* `-` : 减法运算
* `*` : 乘法运算
* `/` : 除法运算
* `%` : 取模运算，两个数字相除取余数
* `++`,`--` : 自增自减运算


<!--more-->

Java中，整数使用以上运算符，不会得到小数。
- ++运算，变量自增长1。反之，--运算，变量自己减少1。
    - 独立运算：++i和i++没有区别
    - 混合运算：和其他变量放在一起，++i和i++就有不同结果。

```java
//变量前++：变量a自己先+1，将加1后的结果赋值给b，也就是说a先计算后赋值。
public static void main(String[] args) {
    int a = 1;
    int b = ++a;
    System.out.println(a);//计算结果是2
    System.out.println(b);//计算结果是2
}
//变量后++：变量a先把自己的值1，赋值给变量b，此时b值就是1，变量a自己再加1，a值就是2。先赋值后计算。
public static void main(String[] args) {
    int a = 1;
    int b = a++;
    System.out.println(a);//计算结果是2
    System.out.println(b);//计算结果是1
}
```

### 赋值运算符


* `=`  : 等于号
* `+=` : 加等于
* `-=` : 减等于
* `*=` : 乘等于
* `/=` : 除等于
* `%=` : 取模

赋值运算符，就是将符号右边的值，赋给左边的变量。


### 比较运算符


* `==` : 比较符号两边数据是否相等，相等结果是true。
* `<` : 比较符号左边的数据是否小于右边的数据，如果小于结果是true。
* `>` : 比较符号左边的数据是否大于右边的数据，如果大于结果是true。
* `<=` : 比较符号左边的数据是否小于或者等于右边的数据，如果小于结果是true。
* `>=` : 比较符号左边的数据是否大于或者等于右边的数据，如果小于结果是true。
* `!=` : 不等于符号 ，如果符号两边的数据不相等，结果是true。
比较运算符，是两个数据之间进行比较的运算，运算结果都是布尔值，true或者false。

### 逻辑运算符
*&&短路与*
1. 两边都真，则真
2. 一边是假，则假
短路特定：如果左边遇假则可以得到最终结果，右边不再运算

*||短路或*
1. 两边都假，则假
2. 一边是真，则真
短路特点：如果左边遇真则可以得到最终结果，右边不再运算

*！取反*
1. ！true 则是假
2. ！false 则是真
```java
//逻辑运算符，是用来连接两个布尔类型结果的运算符，运算结果都是布尔值 true 或者 false
public static void main(String[] args) {
    System.out.println(true && true);//true
    System.out.println(true && false);//false
    System.out.println(false && true);//false，右边不计算
    System.out.println(false || false);//falase
    System.out.println(false || true);//true
    System.out.println(true || false);//true，右边不计算
    System.out.println(!false);//true
}
```

TIPS：
1. 逻辑运算符只能用于boolean值。
2. 与、或需要左右各自有一个boolean值，但是取反只要有唯一一个boolean值即可。
3. 与、或两种运算符，如果有多个条件，可以连续写。多个条件：条件A && 条件B && 条件C


### 三元运算符
- 三元运算符格式：
数据类型 变量名 = 布尔类型表达式？结果1：结果2

- 三元运算符计算方式：

```java
//布尔类型表达式结果是true，三元运算符整体结果为结果1，赋值给变量。
//布尔类型表达式结果是false，三元运算符整体结果为结果2，赋值给变量。
public static void main(String[] args) {
    int i = (1==2 ? 100 : 200);
    System.out.println(i);//200
    int j = (3<=4 ? 500 : 600);
    System.out.println(j);//500
}
```
