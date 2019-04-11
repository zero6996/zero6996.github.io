---
title: Java基础
date: 2019-4-9 16:00
categories: Java # 分类
tags: [Java,Java环境搭建,基础语法]
description: 该文章记录了Java简介,开发环境搭建,基础知识等
---

## Java简介

[参考Runoob](https://www.runoob.com/java/java-intro.html)

## 搭建开发环境

[下载JDK](https://www.oracle.com/technetwork/java/javase/downloads/index.html)

- 配置环境变量

[下载Eclipse](https://www.eclipse.org/downloads/)


<!--more-->
### 基础语法
编写Java程序时,需注意以下几点:
- 大小写敏感:Java是大小写敏感的,这就意味着标识符Hello与hello是不同的
- 类名:对于所有的类来说,类名的首字母应该大写.如果类名由若干单词组成,那么需采用驼峰命名法
- 方法名:所有的方法都应该以小写字母开头.如果方法名含有若干单词,则后面的每个单词首字母大写
- 源文件名:源文件名必须和类名相同.当保存文件时,你应该使用类名作为文件名保存,文件名的后缀为.java(如文件名和类名不相同则会导致编译错误)
- 主方法入口:所有的Java程序由public static void main(String []args)方法开始执行
### Java标识符
Java所有的组成部分都需要名字. 类名,变量名以及方法名都被称为标识符.
关于Java标识符,有一下几点需要注意:
- 所有的标识符都应该以字母(A-Za-z),美元符($),或者下划线(_)开始
- 首字符之后可以是字母(A-Za-z),美元符,下划线或数字的任何字符组合
- 关键字不能用作标识符
- 标识符大小写敏感
- 合法标识符举例:age, $salary, _value, __1_value
- 非法标识符举例:123abc, -salary
### Java修饰符
像其他语言一样,Java可以使用修饰符来修饰类中方法和属性.主要有两类修饰符:
- 访问控制修饰符: default, public, protected, private
- 非访问控制修饰符: final, abstract, static, synchronized
### Java变量
Java中主要有如下几种类型的变量
- 局部变量
- 类变量(静态变量)
- 成员变量(非静态变量)
### Java数组
数组是存储在堆上的对象,可以保存多个同类型变量
### Java枚举
枚举限制变量只能是预先设定好的值. 使用枚举可以减少代码中的bug.
例如:为果汁店设计一个程序,它将限制果汁为小杯,中杯,大杯. 这就意味着它不允许顾客点除了这三种尺寸外的果汁
```
class FreshJuice {
    enum FreshJuinceSize{ SMALL, MEDIUM, LARGE }
    FreshJuiceSize size;
}

public class FreshJuiceTest {
    public static void main(String []args){
        FreshJuice juice = new FreshJuice();
	juice.size = FreshJuice.FreshJuiceSize.MEDIUM;
    }
}
//注意：枚举可以单独声明或者声明在类里面。方法、变量、构造函数也可以在枚举中定义
```
### Java关键字
以下列出Java关键字,这些保留字不能用于常量,变量,和任何标识符的名称.
[详见runoob Java关键字](https://www.runoob.com/java/java-basic-syntax.html)
### Java注释
类似于C/C++, Java也支持单行以及多行注释.注释字符将被Java编译器忽略.
```
public class HelloWorld {
   /* 这是第一个Java程序
    *它将打印Hello World
    * 这是一个多行注释的示例
    */
    public static void main(String []args){
       // 这是单行注释的示例
       /* 这个也是单行注释的示例 */
       System.out.println("Hello World"); 
    }
}
```
### Java空行
空白行或者有注释的行,Java编译器都会忽略掉
### 继承
在Java中,一个类可以由其他类派生.如果你要创建一个类,而且已经存在一个类具有你所需的属性或者方法,那么你可以将新创建的类继承该类.
利用继承的方法,可以重用已存在类的方法和属性,而不用重写这些代码. 被继承的类称为超类(super class),派生类称为子类(sub class)
### 接口
在Java中,接口可以理解为对象间相互通信的协议.接口在继承中扮演者重要的角色
接口只定义派生要用到的方法,但是方法的具体实现完全取决于派生类
### Java源程序与编译型运行区别
如图所示:
![Java源程序与编译型运行区别](https://i.loli.net/2019/04/10/5cad995bb0b6b.png)
## Java对象和类
Java作为一种面向对象语言,支持以下基本概念:
- 多态
- 继承
- 封装
- 抽象
- 类
- 对象
- 实例
- 方法
- 重载
以下重点研究对象和类的概念
- 对象:对象是类的一个实例,例如,一条狗是一个对象,它的状态有:颜色,名字,品种;行为有:摇尾巴,叫,吃等
- 类:类是一个模板,它描述一类对象的行为和状态.
下图中男孩女孩为类,具体每个人为该类的对象:
![object-class.jpg](https://i.loli.net/2019/04/10/5cad9e36a063b.jpg)
### Java中的对象
现在让我们深入了解什么是对象。看看周围真实的世界，会发现身边有很多对象，车，狗，人等等。所有这些对象都有自己的状态和行为。<br>
拿一条狗来举例，它的状态有：名字、品种、颜色，行为有：叫、摇尾巴和跑。<br>
对比现实对象和软件对象，它们之间十分相似。<br>
软件对象也有状态和行为。软件对象的状态就是属性，行为通过方法体现。<br>
在软件开发中，方法操作对象内部状态的改变，对象的相互调用也是通过方法来完成。
### Java中的类
类可以看成是创建Java对象的模板,
下面通过一个简单的类来理解下Java中类的定义:
```
public class Dog{
    String breed;
    int age;
    String color;
    void barking(){
	}
    void hungry(){
	}
    void sleeping(){
        }
}
```
一个类可以包含以下类型变量:
- 局部变量:在方法,构造方法或者语句块中定义的变量被称为局部变量.变量声明和初始化都是在方法中,方法结束后,变量就会自动销毁.
- 成员变量:成员变量是定义在类中,方法体之外的变量.这种变量在创建对象时实例化.成员变量可以被类中方法,构造方法和特定类的语句块访问
- 类变量:类变量也声明在类中,方法体之外,但必须声明为static类型
一个类可以拥有多个方法,在上面的例子中:barking(),hungry(),sleeping()都是Dog类的方法
### 构造方法
每个类都有构造方法. 如果没有显式地为类定义构造方法,Java编译器将会为该类提供一个默认构造方法<br>在创建一个对象时,至少要调用一个构造方法,构造方法的名称必须与类同名,一个类可以有多个构造方法.<br>下面是一个构造方法实例:
```
public class Puppy{
    public Puppy(){
	}
    public Puppy(String name){
	//这个构造器仅有一个参数:name
	}
}
```
### 创建对象
对象是根据类创建的.在Java中,使用关键字==new==来创建一个新的对象.创建对象需要以下三步:
- 声明:声明一个对象,包括对象名称和对象类型
- 实例化:使用关键字new来创建一个对象
- 初始化:使用new创建对象时,会调用构造方法初始化对象
下面是一个创建对象的例子:
```
public class Puppy{
    public Puppy(String name){
	//这个构造器仅有一个参数:name
	Syetem.out.println("小狗的名字是:"+name);
    }
    public static void main(String []args){
	//下面语句将创建一个Puppy对象
	Puppy myPuppy = new Puppy("tommy");
    }
}
```
### 访问实例变量和方法
通过已创建的对象来访问成员变量和成员方法,如下所示:
```
/* 实例化对象 */
Object referenceVariable = new Constructor();
/* 访问类中的变量 */
referenceVariable.variableName;
/* 访问类中的方法 */
referenceVariable.methodName();
```
### 实例
下面的例子展示如何访问实例变量和调用成员方法:
```
public class Puppy {
	int puppyAge;
	public Puppy(String name) {
		/*这个构造器仅有一个参数:name*/
		System.out.println("小狗的名字是:" + name);
	}
	
	public void setAge(int age) {
		puppyAge = age;
	}
	public int getAge() {
		System.out.println("小狗的年龄为:" + puppyAge);
		return puppyAge;
	}
	
	public static void main(String []args) {
		//使用new创建对象
		Puppy myPuppy = new Puppy("tommy");
		//通过方法来设定age
		myPuppy.setAge( 2 );
		//调用另一个方法获取age
		myPuppy.getAge();
		//你也可以通过下面方法访问成员变量
		System.out.println("变量值:" + myPuppy.puppyAge);
	}
}

```
### 源文件声明规则
当在一个源文件中定义多个类,并且还有import语句和package语句时,要特别注意以下规则
- 一个源文件中只能有一个public类
- 一个源文件可以有多个非public类
- 源文件的名称应该和public类的类名保持一致.例如:源文件中public类的类名是Test,那么源文件应该命名为Test.java
- 如果一个类定义在某个包中,那么package语句应该在源文件的首行
- 如果源文件包含import语句,那么应该放在package语句和类定义之间.如果没有package语句,那么import语句应该在源文件中最前面
- import语句和package语句对源文件中定义的所有类都有效.在同一源文件中,不能给不同的类不同的包声明.
类有若干种访问级别,并且类也分不同的类型:抽象类和final类等.<br>除了上述提到的几种类型,Java还有一些特殊类,如:内部类,匿名类.
### Java包
包主要用来对类和接口进行分类,当开发Java程序时，可能编写成百上千的类，因此很有必要对类和接口进行分类。
### import语句
在Java中,如果给出一个完整的限定名,包括包名,类名,那么Java编译器就可以很容易地定位到源代码或者类.import语句就是用来提供一个合理的路径,使得编译器可以找到某个类.<br>例如,下面的命令行将会命令编译器载入java_installation/java/io路径下的所有类.
```
import java.io.*;
```