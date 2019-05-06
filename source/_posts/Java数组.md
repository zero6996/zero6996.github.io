---
title: Java数组
date: 2019-4-16 16:00
categories: Java学习笔记 # 分类
tags: [Java]
description: 数组的基本概念，特点以及相关应用
---

## 数组概念
- 数组的基本概念：数组就是存储数据长度固定的容器，包装多个数据的数据类型要一致。
- 容器：是将多个数据存储在一起，每个数据称为该容器的元素。
- 数组的特点：
    - 数组是一种引用数据类型。
    - 数组当中的多个数据，类型必须统一。
    - 数组的长度在程序运行期间不可改变。

<!--more-->

## 1. 数组的定义和访问
### 1.1 数组的定义
- 数组的初始化：在内存当中创建一个数组，并且向其中赋予一些默认值。
- 两种常见的初始化方式：
    - 动态初始化（指定长度）
    - 静态初始化（指定内容）
- 动态初始化格式：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/16/%E5%8A%A8%E6%80%81%E5%88%9D%E5%A7%8B%E5%8C%96-1555383964746.png)
- 数组定义格式详解：
    - 数组存储的数据类型：创建的数组容器可以存储什么数据类型。
    - []:表示数组。
    - 数组的名字：为定义的数组起个变量名，满足标识符规范，可以使用名字操作数组。
    - new：关键字，创建数组使用的关键字。
    - 数组存储的数据类型：创建的数组容器可以存储什么数据类型。
    - [长度]：数组的长度，表示数组容器中可以存储多少个元素。
    - 注意：数组有定长特性，长度一旦指定，不可更改。类似你买一个水杯，买了一个1升的水杯，总容量就是1升，不多不少。
- 举例：定义可以存储3个整数的数组容器，代码如下：
```java
int[] arr = new int[3];
```
- 静态初始化标准格式：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/16/%E9%9D%99%E6%80%81%E5%88%9D%E5%A7%8B%E5%8C%96-1555384404508.png)
- 举例：定义存储1,2,3,4,5整数的数组容器。
```java
int[] arr = new int[]{1,2,3,4,5};
```
- 静态初始化省略格式：

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/16/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15553845603050-1555384614682.png)
- 举例：定义存储1,2，3整数的数组容器：
```java
int[] arr = {1,2,3};
```
注意：
1. 静态初始化没有直接指定长度，但仍然会自动推算得到长度。
2. 静态初始化标准格式可以拆分为两个步骤。
3. 动态初始化也可以拆分成为两个步骤。
4. 静态初始化一旦使用省略格式，就不能拆分成为两个步骤了。
5. 不确定数组当中具体内容，使用动态初始化；确认了具体内容，用静态初始化。
```java
//静态初始化的标准格式拆分：
int[] arrayA;
arrayA = new int[]{1,2,3};
//动态初始化的拆分步骤：
int[] arrayB;
arrayB = new intp[3];
```

### 1.2 数组的访问
- 索引：每一个存储到数组的元素，都会自动的拥有一个编号，从0开始，这个自动编号称为数组索引(index),可以通过数组的索引访问到数组中的元素。
- 格式：数组名[索引]
- 数组长度属性：每个数组都具有长度，而且是固定的，Java中赋予了数组的一个属性，可以获取到数组的长度，语句为：数据名.length。属性length的执行结果是数组的长度，int类型结果。由此可以推断出，数组的最大索引值为数组名.length-1。
```java
public class Demo02ArrayUse {
    public static void main(String[] agrs){
        int[] arr = new int[]{1,2,3,4,5};
        System.out.println(arr.length);
        System.out.println(arr[arr.length-1]);
    }
}
```
- 索引访问数组中的元素：
    - 数组名[索引]=数组，为数组中的元素赋值
    - 变量=数组名[索引]，获取数组中的元素

## 2. 数组原理内存图
### 2.1 内存概述
内存是计算机中的重要原件，临时存储区域，作用是运行程序。我们编写的程序是存放在硬盘中的，在硬盘中的程序是不会运行的，必须放进内存中才能运行，运行完毕后会清空内存。<br>Java虚拟机要运行程序，必须要对内存进行空间的分配和管理。

### 2.1 Java虚拟机的内存划分
为了提高运算效率，就对空间进行了不同区域的划分，因为每一片区域都有特定的处理数据方式和内存管理方式。
- JVM的内存划分：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/16/JVM%E5%86%85%E5%AD%98%E5%88%92%E5%88%86-1555393728931.png)

Java内存需划分为5个部分：
1. ==栈(Stack)==：存放的都是方法中的局部变量。方法运行时使用的内存。
    - 局部变量：方法的参数，或者是方法{}内部的变量
    - 作用域：一旦超出作用域，立刻从栈内存当中消失。
2. ==堆(Heap)==：凡是new出来的东西，都在堆当中。存储对象或数组。	
    - 堆内存里面的东西都有一个地址值：16进制
    - 堆内存里面的数据，都有默认值。规则：

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/16/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E9%BB%98%E8%AE%A4%E5%80%BC-1555394445700.png)

3. ==方法区(Method Area)==：存储.class相关信息，包含方法的信息。
4. 本地方法栈(Native Method Stack)：与操作系统相关。
5. 寄存器(pc Register)：与CPU相关。

### 2.2 数据在内存中的存储
```java
public class Demo02ArrayOne {
    public static void main(String[] args){
        int[] arr = new int[3];
        System.out.println(arr); // [I@5f184fc6
    }
}
```
以上方法执行，输出结果是[I@5f184fc6。这个是数组在内存中的地址。new出来的内容，都是在堆内存中存储的，而方法中的变量arr保存的是数组的内存地址。<br>
一个数组内存图：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/16/%E6%95%B0%E7%BB%84%E5%86%85%E5%AD%98%E5%9B%BE%E8%AF%A6%E8%A7%A3-1555396483239.png)
两个数组内存图：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/16/%E4%B8%A4%E4%B8%AA%E6%95%B0%E7%BB%84%E5%86%85%E5%AD%98%E5%9B%BE-1555397156716.png)

## 3. 数组的常见操作
### 3.1 数组越界异常
```java
public static void main(String[] args){
    int[] arr = {1,2,3};
    System.out.print(arr[3]);
}
```
创建数组，赋值3个元素，数组索引就是0,1,2,没有索引3。如果我们访问了数组中不存在的索引，程序运行后就会抛出`ArrayIndexOutOfBoundsException`数组越界异常。
### 3.2 数组空指针异常
```java
public class Demo02ArrayNull {
    public static void main(String[] args){
        int [] array = null;
        // array = new int[3];
        System.out.println(array[0]);
    }
}
```
所有的引用类型变量，都可以赋值为一个null值。但代表其中什么都没有。
- 数组必须进行new初始化才能使用其中的元素，如果赋值一个null，没有进行new创建数组，程序运行后将出现`NullPointerException`空指针异常。

### 3.3 数组遍历
- 数组遍历：将数组中的每个元素分别获取出来，就是遍历。遍历也是数组操作中的基石。
```java
public class Demo02Array_bianli {
    public static void main(String[] args){
        int[] arr = {1,2,3,4,5};
        for(int i = 0;i < arr.length;i++){
            System.out.println(arr[i]);
        }
    }
}
```
### 3.4 数组获取最大值元素
- 最大值获取：从数组的所有元素中找出最大值。
- 实现思路：
    - 定义变量，保存数组0索引上的元素
    - 遍历数组，获取出数组中的每个元素
    - 将遍历到的元素和保存数组0索引上值的变量进行比较
    - 如果数组元素值大于保存变量值，则重新赋值给变量，替换掉
    - 数组循环遍历结束，变量中保存的就是数组中的最大值
```java
public class Demo02ArrayMAX {
    public static void main(String[] args){
        int[] arr = {5,15,2000,10000,100,4000};
        int max = 0;
        for(int i = 0;i < arr.length;i++){
            if(arr[i] > max){
                max = arr[i];
            }
        }
        System.out.println("arr数组最大值为：" + max);
    }
}
```
### 3.5 数组反转
- 数组的反转：数组中的元素颠倒顺序，例如原始数组为1,2,3,4,5，反转后的数组为5,4,3,2,1
- 实现思路：数组最远端的元素互换位置
- 定义两个变量，保存数组的最小索引和最大索引
- 两个索引上的元素交换位置
- 最小索引++，最大索引--，再次交换位置
- 最小索引超过了最大索引，数组反转操作结束


## 4. 数组作为方法参数和返回值
### 4.1 数组作为方法参数
以前的方法中我们学习了方法的参数和返回值，但是使用的都是基本数据类型。
- 数组作为方法参数传递，传递的参数是数组内存的地址。
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/16/reverse-1555405679573.png)
