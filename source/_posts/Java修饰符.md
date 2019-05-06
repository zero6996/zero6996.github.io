---
title: Java修饰符
date: 2019-4-12 16:00
categories: Java学习笔记 # 分类
tags: [Java]
description: 该文章记录了Java语言的修饰符,以及相关使用方式
---

## 前言
Java语言提供了很多修饰符,主要分为以下两类:
- 访问修饰符
- 非访问修饰符
修饰符用来定义类、方法或者变量，通常放在语句的最前端。

<!--more-->

### 访问控制修饰符
Java中，可以使用==访问控制符==来保护对类、变量、方法和构造方法的访问。Java支持4种不同的访问权限。
- default（即缺省，什么也不写）：在同一包内可见，不使用任何修饰符。使用对象：类、接口、变量、方法。
- private：在同一类内可见。使用对象：变量、方法。注意：不能修饰类（外部类）
- public：对所有类可见。使用对象：类、接口、变量、方法
- protected：对同一包内的类和所有子类可见。使用对象：变量，方法。注意：不能修饰类（外部类）。

#### 默认访问修饰符-不使用任何关键字
使用默认访问修饰符声明的变量和方法，对同一包内的类是可见的。接口里的变量都隐式声明为public static final，而接口里的方法默认情况下访问权限为publi。
如下例，变量和方法的声明可以不使用任何修饰符
```java
String version = "3.0.3";
boolean processOrder(){
    return true;
}
```

#### 私有访问修饰符-private
私有访问修饰符是最严格的访问级别，所以被声明为private的方法、变量和构造方法只能被所属类访问，并且类和接口不能声明为private。<br>声明为私有访问类型的变量只能通过类中公共的getter方法被外部类访问。<br>Private访问控制修饰符的使用主要用来隐藏类的实现细节和保护类的数据。<br>下面的类使用了私有访问控制符：
```java
public class Logger{
    private String format;
    public String getFormat(){
	return this.format;
    }
    public void setFormat(String format){
	this.format = format;
    }
}
```
实例中，Logger类中的format变量为私有变量，所以其他类不能直接得到和设置该变量的值。为了使其他类能够操作该变量，定义了两个public方法：getFormat()（能够返回format的值）和setFormat(String)（设置format的值）

#### 公有访问修饰符-public

被声明为public的类、方法、构造方法和接口能够被任何其他类访问。<br>如果几个相互访问的public类分布在不同的包中，则需要导入相应public类所在包。由于类的继承性，类的所有公有方法和变量都能被其子类继承。<br>以下函数使用了公有访问控制：
```java
public static void main(String[] args){
	// ...
}
```
Java程序的main()方法必须设置公有的，否则，Java解释器将不能运行该类。

#### 受保护的访问修饰符-protected
protected需要从以下两个点来分析说明：
- 子类与基类在同一包中：被声明protected的变量、方法和构造器能够被同一个包中的任何其他类访问；
- 子类与基类不在同一包中：那么在子类中，子类实例可以访问其从基类继承而来的protected方法，而不能访问基类实例的protected方法。
protected可以修饰数据成员，构造方法，方法成员，不能修饰类(内部类除外)。<br>
子类能访问protected修饰符声明的方法和变量，这样就能保护不相关的类使用这些方法和变量。<br>下面的父类使用了protected访问修饰符，子类重写了父类的openSpeaker()方法。
```java
class AudioPlayer{
    protected boolean openSpeaker(Speaker sp){
	//实现细节
    }
}

class StreamingAudioPlayer extends AudioPlayer{
    protected boolean openSpeaker(Speaker sp){
    	//实现细节
    }
}
```
如果把 openSpeaker() 方法声明为 private，那么除了 AudioPlayer 之外的类将不能访问该方法。<br>
如果把 openSpeaker() 声明为 public，那么所有的类都能够访问该方法。<br>
如果我们只想让该方法对其所在类的子类可见，则将该方法声明为 protected。<br>
[Java protected关键字详解](https://www.runoob.com/w3cnote/java-protected-keyword-detailed-explanation.html)

#### 访问控制和继承
请注意以下方法继承的规则:
- 父类中声明为public的方法在子类中也必须为public。
- 父类中声明为protected的方法在子类中要么声明为protected，要么声明为public，不能声明为private。
- 父类中声明为private的方法，不能被继承。

### 非访问修饰符
为了实现一些其他功能，Java也提供了许多非访问修饰符。
- static修饰符，用来修饰类方法和类变量。
- final修饰符，用来修饰类、方法和变量，final修饰的类不能被继承，修饰的方法不能被继承类重新定义，修饰的变量为常量，是不可修改的。
- abstract修饰符，用来创建抽象类和抽象方法。
- synchronized和volatile修饰符，主要用于线程的编程。

#### static修饰符
- 静态变量：static关键字用来声明独立于对象的静态变量，无论一个类实例化多少对象，它的静态变量只有一份拷贝。静态变量也被称为类变量。局部变量不能被声明为static变量。
- 静态方法：static关键字用来声明独立于对象的静态方法。静态方法不能使用类的非静态变量。静态方法从参数列表得到数据，然后计算这些数据。

对类变量和方法的访问可以直接使用classname.variablename和classname.methodname方式访问。<br>如下示例，static修饰符用来创建类方法和类变量。
```java
public class InstanceCounter {
   private static int numInstances = 0;
   protected static int getCount() {
      return numInstances;
   }
 
   private static void addInstance() {
      numInstances++;
   }
 
   InstanceCounter() {
      InstanceCounter.addInstance();
   }
 
   public static void main(String[] arguments) {
      System.out.println("Starting with " +
      InstanceCounter.getCount() + " instances");
      for (int i = 0; i < 500; ++i){
         new InstanceCounter();
          }
      System.out.println("Created " +
      InstanceCounter.getCount() + " instances");
   }
}
//以上示例运行结果如下：
Starting with 0 instances
Created 500 instances
```
#### final修饰符
final变量：final表示“最后的，最终的”意思，变量一旦赋值后，不能被重新赋值。被final修饰的实例变量必须显式指定初始值。final修饰符通常和static修饰符一起使用来创建类常量。
```java
//实例
public class Test{
  final int value = 10;
  // 下面是声明常量的实例
  public static final int BOXWIDTH = 6;
  static final String TITLE = "Manager";
 
  public void changeValue(){
     value = 12; //将输出一个错误
  }
}
```
##### final方法
- 类中的final方法可以被子类继承，但不能被子类修改。
- 声明final方法的主要目的是防止该方法的内容被修改。
如下所示，使用final修饰符声明方法。
```java
public class Test{
    public final void changeName(){
	//方法体
    }
}
```
##### final类
final类不能被继承，没有类能够继承final类的任何特性。
```java
//实例
public final class Test{
	//类体
}
```
#### abstract修饰符
##### 抽象类：
- 抽象类不能用来实例化对象，声明抽象类的唯一目的是为了对该类进行扩充。
- 一个类不能同时被abstract和final修饰。如果一个类包含抽象方法，那么该类一定要声明为抽象类，否则将会编译错误。
- 抽象类可以包含抽象方法和非抽象方法。
```java
//实例
abstract class Caravan{
   private double price;
   private String model;
   private String year;
   public abstract void goFast(); //抽象方法
   public abstract void changeColor();

}
```

##### 抽象方法
- 抽象方法是一种没有任何实现的方法，该方法的具体实现由子类提供。
- 抽象方法不能被声明成final和static。
- 任何继承抽象类的子类必须实现父类的所有抽象方法，除非该子类也是抽象类。
- 如果一个类包含若干个抽象方法，那么该类必须声明为抽象类，抽象类可以不包含抽象方法。
抽象方法的声明以分号结尾，例如:public abstract sample();
```java
//实例
public abstract class SuperClass{
    abstract void m(); //抽象方法
}
 
class SubClass extends SuperClass{
     //实现抽象方法
      void m(){
          .........
      }
}
```

#### synchronized(同步)修饰符
synchronized关键字声明的方法同一时间只能被一个线程访问。synchronized修饰符可以应用于四个访问修饰符。
```java
//实例
public synchronized void showDetails(){
    .......
}
```
#### transient(瞬态)修饰符
序列化的对象包含被transient修饰的实例变量时，java虚拟机（JVM）跳过该特定的变量。<br>该修饰符包含在定义变量的语句中，用来预处理类和变量的数据类型。
```java
//实例
public transient int limit = 55;   // 不会持久化
public int b; // 持久化
```
#### volatile（挥发性）修饰符
volatile修饰的成员变量在每次被线程访问时，都强制从共享内存中重新读取该成员变量的值。而且当成员变量发生变化时，会强制线程将变量值回写到共享内存。这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。<br>一个volatile对象引用可能是null。
```java
//实例
public class MyRunnable implements Runnable
{
    private volatile boolean active;
    public void run()
    {
        active = true;
        while (active) // 第一行
        {
            // 代码
        }
    }
    public void stop()
    {
        active = false; // 第二行
    }
}
```
通常情况下，在一个线程调用 run() 方法（在 Runnable 开启的线程），在另一个线程调用 stop() 方法。 如果 第一行 中缓冲区的 active 值被使用，那么在 第二行 的 active 值为 false 时循环不会停止。<br>
但是以上代码中我们使用了 volatile 修饰 active，所以该循环会停止。

### 本章总结
Java的类（外部类）有2种访问权限：public，default。<br>而方法和变量有4种：public、default、protected，private。<br>其中默认访问权限和protected很相似，有着细微的差别。
- public意味着任何地方的其他类都能访问。
- default则是同一个包的类可以访问。
- protected表示同一个包的类可以访问，其他的包的该类子类也可以访问。
- private表示只有自己类能够访问。

修饰符：abstract、static、final
- abstract：表示是抽象类。使用对象：类、接口、方法。
- static：可以当做普通类使用，而不用先实例化一个外部类。（用他修饰后，就成了静态内部类了）。使用对象：类、变量、方法、初始化函数（注意：修饰类时只能修饰内部类）
- final：表示类不可以被继承。使用对象：类、变量、方法。

##### 关于static
- static全局变量与普通的全局变量：static全局变量只初使化一次，防止在其他文件单元中被引用;
- static局部变量和普通局部变量：static局部变量只被初始化一次，下一次依据上一次结果值；
- static函数与普通函数：static函数在内存中只有一份，普通函数在每个被调用中维持一份拷贝。
