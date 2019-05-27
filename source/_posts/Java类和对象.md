---
title: Java类和对象
date: 2019-4-17 17:45
categories: Java学习笔记 # 分类
tags: [Java,类和对象]
description: 主要内容是Java的类和对象，封装和构造方法
---


## 1. 面向对象思想
### 1.1 面向对象思想概述
Java语言是一种面向对象的程序设计语言，而面向对象思想是一种程序设计思想，我们在面向对象思想的指引下，使用Java语言去设计，开发计算机程序。这里的对象泛指现实中一切事物，每种事物都具备自己的属性和行为。面向对象思想就是在计算机程序设计过程中，参照现实中事物，将事物的属性特征、行为特征抽象出来，描述成计算机事件的设计思想。它区别于面向过程思想，强调的是通过调用对象的行为来实现功能，而不是自己一步一步的去操作实现。

<!--more-->

#### 举例
- 洗衣服：
    - 面向过程：把衣服脱下来-->找一个盆-->放点洗衣粉-->加点水-->浸泡10分钟-->揉一揉-->清洗衣服-->拧干-->晾起来
    - 面向对象：把衣服脱下来-->打开全自动洗衣机-->放衣服-->开启洗衣机-->晾起来
- 区别：
    - 面向过程：强调步骤
    - 面向对象：强调对象，这里对象指洗衣机
#### 特点
面向对象思想是一种更符合我们思考习惯的思想，它可以将复杂的事情简单化，并将我们从执行者变成了指挥者。面向对象的语言中，包含了三大基本特征，即封装、继承和多态。

### 1.2 类和对象
#### 什么是类？
- 类：是一组相关属性和行为的集合。可以看成是一类事物的模板，使用事物的属性特征和行为特征来描述该类事物。
- 属性：就是该类事物的状态信息
- 行为：就是该类事物能够做什么事
- 举例：小猫。属性：名字，颜色，年龄，体重。行为：走，跳，叫，吃。
#### 什么是对象？
- 对象：是一类事物的具体体现。对象是类的一个实例，必然具备该类事物的属性和行为。
- 现实中，一类事物(猫类)的一个实例:一只小猫。
- 举例：一只小猫。属性：tom，5kg，2years，yellow。行为：反复横跳，喵喵叫，吃饭。
#### 类和对象的关系
- 类是对一类事物的描述，是抽象的。
- 对象是一类事物的实例，是具体的，实际存在的。
- 类是对象的模板，对象是类的实体。

### 1.3 类的定义
#### 事物与类的对比
现实世界的一类事物：
- 属性：事物的状态信息。
- 行为：事物能够做什么。
Java中用class描述事物也是如此：
- 成员变量：对应事物的属性。
- 成员方法：对应事物的行为。
#### 类的定义格式
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/17/%E7%B1%BB%E5%AE%9A%E4%B9%89%E6%A0%BC%E5%BC%8F-1555469808569.png)
- 定义类：就是定义类的成员，包括成员变量和成员方法。
- 成员变量：和以前定义变量几乎是一样的。只不过位置发生了改变。在类中，方法外。
- 成员方法：和以前定义方法几乎一样的。只不过把static去掉。<br>
举例：
```java
public class Student {
    //成员变量;
    String name; // 姓名
    int age; //年龄

    //成员方法
    public void eat(){
        System.out.println("吃饭饭");
    }
    public void sleep(){
        System.out.println("睡觉觉");
    }
    public void study() {
        System.out.println("学习");
    }
}
```

### 1.4 对象的使用
#### 对象的使用格式
- 使用步骤1，导包：也就是指出需要使用的类，在什么位置，同一包下可省略导包步骤。<br>import 包名称.类名称;<br>import com.zero.demo.demo03_Class.Student;
- 步骤2，创建对象：类名 对象名 = new 类名();
- 步骤3，使用对象访问类中成员:
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/17/%E5%AF%B9%E8%B1%A1%E8%AE%BF%E9%97%AE%E7%B1%BB%E4%B8%AD%E6%88%90%E5%91%98-1555470287058.png)

举例：
```java
public class Demo03Student {
    public static void main(String[] args){
        Student stu = new Student();
        System.out.println(stu.name = "校长");
        System.out.println(stu.age = 55);
        stu.eat();
        stu.sleep();
        stu.study();
    }
}
```
注意：如果成员变量没有进行赋值，那么将会有一个默认值，规则和数组一样。
#### 成员变量的默认值
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/17/%E6%88%90%E5%91%98%E5%8F%98%E9%87%8F%E9%BB%98%E8%AE%A4%E5%80%BC-1555471157001.png)

### 1.5 对象内存图
一个对象，调用一个方法内存图：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/17/%E5%AF%B9%E8%B1%A1%E5%86%85%E5%AD%98%E5%9B%BE-1555472618000.png)
两个对象，调用一个方法内存图：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/17/two%E5%AF%B9%E8%B1%A1%E5%86%85%E5%AD%98%E5%9B%BE-1555482190913.png)
两个引用指向同一个对象的内存图：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/17/%E4%B8%A4%E4%B8%AA%E5%BC%95%E7%94%A8%E6%8C%87%E5%90%91%E5%90%8C%E4%B8%80%E5%AF%B9%E8%B1%A1-1555482832109.png)
使用对象类型作为方法的参数：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/17/%E5%AF%B9%E8%B1%A1%E7%B1%BB%E5%9E%8B%E4%BD%9C%E4%B8%BA%E6%96%B9%E6%B3%95%E5%8F%82%E6%95%B0-1555484961711.png)

### 1.6 成员变量和局部变量的区别
变量根据定义位置的不同，变量有不同的名字
- 在类中的位置不同*重点*
    - 成员变量：类中，方法外
    - 局部变量：方法中或者方法声明上(形参)
- 作用范围不一样*重点*
    - 成员变量：类中
    - 局部变量：方法中
- 初始化值的不同*重点*
    - 成员变量：有默认值
    - 局部变量：无默认值。必须先定义，赋值，才能使用。
- 在内存中的位置不同(了解)
    - 成员变量：堆内存
    - 局部变量：栈内存
- 生命周期不同(了解)
    - 成员变量：随着对象创建而诞生，随着对象被垃圾回收而消失
    - 局部变量：随着方法进栈而诞生，随着方法出栈而消失
## 2. 三大特征之一：封装
### 2.1 封装概述
面向对象编程语言是对象客观世界的模拟，客观世界里成员变量都是隐藏在对象内部的，外界无法直接操作和修改。<br>封装可以被认为是一个保护屏障，防止该类的代码和数据被其他类随意访问。要访问该类的数据，必须通过指定的方式。适当的封装可以让代码更容易理解与维护，也加强了代码的安全性。
#### 原则
将属性隐藏起来，若需要访问某个属性，提供公共方法对其访问。
### 2.2 封装的步骤
1. 使用private关键字来修饰成员变量。
2. 对需要访问的成员变量，提供对应的一对getxxx方法、setxxx方法。

### 2.3 封装的操作--private关键字
##### private的含义
1. private是一个权限修饰符，代表最小权限。
2. 可以修饰成员变量和成员方法。
3. 被private修饰后的成员变量和成员方法，只能在本类中才能访问。
#### private的使用格式
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/17/private%E4%BD%BF%E7%94%A8%E6%A0%BC%E5%BC%8F-1555489770741.png)

举例：
```java
public class Student {
private String name; // 使用private修饰成员变量
private int age;
}
// 定义getxxx方法和setxxx方法，提供对外接口
public class Student {
    private String name;
    private int age;
    public void setName(String n) {
	name = n;
}
    public String getName() {
	return name;
}
    public void setAge(int a) {
	age = a;
}
    public int getAge() {
	return age;
}
}
```
### 2.4 封装优化--this关键字
如果方法形参和成员变量名一致，会导致成员变量赋值失败。 这是由于形参变量名与成员变量名重名，导致成员变量名被隐藏，方法中的变量名，无法访问到成员变量，从而赋值失败。我们需使用this关键字，来解决这个重名问题。
#### this的含义
this代表所在类的当前对象的引用(地址值)，即对象自己的引用。
- 使用格式：this.成员变量名;

举例:

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/17/this%E6%96%B9%E6%B3%95%E4%BC%98%E5%8C%96-1555491440061.png)
### 2.5 封装优化--构造方法
当一个对象被创建时，构造方法用来初始化该对象，给对象的成员变量赋初始值。
#### 构造方法的定义格式
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/17/%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95%E5%AE%9A%E4%B9%89%E6%A0%BC%E5%BC%8F-1555491533588.png)<br>
构造方法的写法上，方法名与它所在的类名相同。它没有返回值，所以不需要返回值类型，也不需要void。举例如下：
```java
public class Student {
    private String name;
    private int age;
    // 无参数构造方法
    public Student() {}
    // 有参数构造方法
    public Student(String name,int age) {
	this.name = name;
	this.age = age;
}
}
```
注意事项：
1. 构造方法的名称必须和所在的类名称完全一模一样。
2. 构造方法不要写返回值类型，连void都不写。
3. 构造方法不能return一个具体的返回值。
4. 如果没有编写任何构造方法，那么编译器会默认生成一个，没有参数，方法体也为空。public Student(){}；
5. 一般编写了至少一个构造方法，那么编译器将不再生成。

### 2.6 标准代码--JavaBean
JavaBean是Java语言编写类的一种标志规范。符合JavaBean的类，要求类必须是具体的和公共的，并且具有无参数的构造方法，提供用来操作成员变量的set和get方法。所以一个标准类通常要拥有下面四个组成部分：
1. 所有的成员变量都要使用private关键字修饰。
2. 为每个成员变量编写一对getxxx/setxxx方法。
3. 编写一个无参数的构造方法。
4. 编写一个全参数的构造方法。
```java
public class ClassName{
    //成员变量
    //构造方法
    //无参构造方法【必须】
    //有参构造方法【建议】
    //成员方法
    //getXxx()
    //setXxx()
}
```
编写符合JavaBean规范的类，以学生类为例，标准代码如下：
```java
public class JavaBeanStudent {
    private String name;
    private  int age;

    public JavaBeanStudent() {
    }

    public JavaBeanStudent(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
以上代码可以通过Code->Generate功能自动生成：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/17/JavaBean%20%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90-1555493892266.png)