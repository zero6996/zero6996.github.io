---
title: Java三大特性之多态
date: 2019-4-21 23:00
categories: Java学习笔记 # 分类
tags: [Java]
description: Java中的多态的概念和接口
---

## 1. 接口
### 1.1 概述
接口，是Java语言中一种引用类型，是方法的集合，如果说类的内部封装了成员变量、构造方法和成员方法，那么接口的内部主要就是**封装了方法**，包含抽象方法，默认方法和静态方法，私有方法。

<!--more-->

接口的定义，它与定义类方式相似，但是使用interface关键字。它也会被编译成.class文件，但一定要明确它并不是类，而是另外一种引用数据类型。
接口的使用，它不能创建对象，但是可以被实现(implements，类似于被继承)。一个实现接口的类(可以看做是接口的子类),需要实现接口中所有的抽象方法，创建该类对象，就可以调用方法了，否则它必须是一个抽象类。

### 1.2 定义格式
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/21/%E6%8E%A5%E5%8F%A3%E5%AE%9A%E4%B9%89%E6%A0%BC%E5%BC%8F-1555818337838.png)

#### 含有抽象方法
- 抽象方法：使用abstract关键字修饰，可以省略，没有方法体。该方法供子类实现使用。
代码如下：
```java
public interface InterFaceName{
    public abstract void method();
}
```

#### 含有默认方法和静态方法
- 默认方法：使用default修饰，不可省略，供子类调用或者子类重写。
- 静态方法：使用static修饰，供接口直接调用。

代码如下：
```java
public interface InterFaceName{
    public default void method(){
	// 执行语句
    }
    public static void method2(){
	// 执行语句
    }
}
```

#### 含有私有方法和私有静态方法
- 私有方法：使用private修饰，供接口中的默认方法或者静态方法调用。

代码如下：
```java
public interface InterFaceName{
    private void method(){
    	// 执行语句
    }
}
```
### 1.3 基本的实现
类与接口的关系为实现关系，即类实现接口，该类可以称为接口的实现类，也可以称为接口的子类。实现的动作类似继承，格式相似，只是关键字不同，实现使用implements关键字。

非抽象子类实现接口：
1. 必须重写接口中所有抽象方法。
2. 继承了接口的默认方法，即可以直接调用，也可以重写。
格式如下：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/21/%E6%8E%A5%E5%8F%A3%E5%B7%A5%E5%85%B7%E7%B1%BB%E5%9F%BA%E6%9C%AC%E6%A0%BC%E5%BC%8F-1555826716777.png)


代码示例：
```java
// 定义接口类
public interface LiveAble {
    // 定义抽象方法
    public abstract void eat();
    public abstract void sleep();
}
// 定义实现类
public class Animal implements LiveAble{
    @Override
    public void eat(){
        System.out.println("吃东西");
    }
    @Override
    public void sleep(){
        System.out.println("睡觉");
    }
}
// 定义测试类
public class InterfaceDemo {
    public static void main(String[] args) {
        // 创建子类对象
        Animal a = new Animal();
        a.eat();
        a.sleep();
    }
}
```

#### 默认方法的使用
可以继承，可以重写，但只能通过实现类的对象来调用。接口当中的默认方法，可以解决接口升级的问题。
1. 实现类可以继承接口类的默认方法，也可以自己重写默认方法。

代码示例如下：
```java
// 定义接口类
public interface LiveAble {
    // 定义抽象方法
    public abstract void eat();
    public abstract void sleep();
    public default void fly(){ // 定义默认方法
        System.out.println("飞行");
    }
}
// 定义实现类
public class Animal implements LiveAble{
    @Override
    public void fly(){ // 重写接口中的默认方法，也可以不重写，继承接口类中的默认方法
        System.out.println("自由自在的飞！");
    }
}
// 定义测试类
public class InterfaceDemo {
    public static void main(String[] args) {
        // 创建子类对象
        Animal a = new Animal();
        a.eat();
        a.sleep();
        a.fly(); // 如果实现类没有重写默认方法，则执行的是继承自接口类的默认方法，重写则执行实现类重写后的方法
    }
}
```
#### 静态方法的使用
静态与.class文件相关，只能使用接口名调用，不能通过实现类的类名或者实现类的对象调用

示例如下：
```java
// 接口类
public interface LiveAble {
    public static void run(){
        System.out.println("这是静态方法，只能通过接口类调用");
    }
}

// 实现类无法继承或重写接口类静态方法
// 测试类
public class InterfaceDemo {
    public static void main(String[] args) {
        // 创建子类对象
        Animal a = new Animal();
	// a.run(); 报错，无法通过实现类调用静态方法
        LiveAble.run(); // 接口类的静态方法只能通过接口类来调用
    }
}
```

#### 私有方法的使用
- 私有方法：只有默认方法可以调用
- 私有静态方法：默认方法和静态方法可以调用。
如果一个接口中有多个默认方法且方法中有代码重复内容，那么可以抽取出来，封装到私有方法中，供默认方法去调用。私有方法是对默认方法和静态方法的辅助。
```java
public interface LiveAble {
    default void func(){
	func1();
	func2();
    }
    private void func1(){
	System.out.println("跑起来~~~");
    }
    private void func2(){
	System.out.println("跑起来~~~");
    }
}
```
#### 接口的常量定义和使用
接口当中也可可以定义"成员变量"，但是必须使用public static final三个关键字进行修饰。可以认为就是接口的[常量]。
格式：public static final 数据类型 常量名称 = 数据值; 一旦使用final关键字进行修饰，说明不可改变。
注意事项：
1. 接口中的常量，可以省略public static final。
2. 接口中的常量，必须进行赋值，不能不赋值。
3. 接口中常量的名称规则：使用完全大写的字母，单词间用下划线进行分隔。

### 接口内容小结
在Java 9+版本中，接口的内容可以有：
- 成员变量其实是常量。
    - 格式：[public] [sttaic] [final] 数据类型 常量名称 = 数据值; 
    - 注意：常量必须进行赋值，且一旦赋值不能改变。常量名称完全大写，用下划线分隔。
- 接口中最重要的就是抽象方法。
    - 格式：[public] [abstract] 返回值类型 方法名称(参数列表);
    - 注意：实现类必须覆盖重写接口的所有抽象方法，除非实现类是抽象类。
- 从Java 8开始，接口里允许定义默认方法。
    - 格式：[public] default 返回值类型 方法名称(参数列表){方法体}
    - 注意：默认方法也可以被覆盖重写
- 从Java 8开始，接口里允许定义静态方法。
    - 格式：[public] static 返回值类型 方法名称(参数列表){方法体}
    - 注意：应该通过接口名称进行调用，无法通过实现类对象调用接口静态方法。
- 从Java 9开始，接口里允许定义私有方法。
    - 普通私有方法：private 返回值类型 方法名称(参数列表){方法体}
    - 静态私有方法：private static 返回值类型 方法名称(参数列表){方法体}
    - 注意：private方法只要接口自己才能调用，无法被实现类或者别人使用。


### 接口的多实现
在继承体系中，一个类只能继承一个父类。而对于接口而言，一个类是可以实现多个接口的，这叫做接口的多实现。并且，一个类能继承一个父类，同时实现多个接口。
格式如下：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/21/%E6%8E%A5%E5%8F%A3%E7%9A%84%E5%A4%9A%E5%AE%9E%E7%8E%B0%E6%A0%BC%E5%BC%8F-1555836946283.png)<br>
使用接口的时候，需注意：
1. 接口是没有静态代码块或者构造方法的。
2. 一个类的直接父类是唯一的，但是一个类可以同时实现多个接口的。
格式：
public class MyIntfaceImpl implements MyInterfaceA,MyInterfaceB{
    // 覆盖重写所有抽象方法
}
3. 如果实现类实现的多个接口中，存在重复的抽象方法，只需覆盖重写一次即可。
4. 如果实现类没有覆盖重写所以接口的所有抽象方法，那么实现类必须是抽象类。
5. 如果实现类所实现的多个接口当中，存在重复的默认方法，那么实现类一定要对冲突的默认方法进行覆盖重写。
6. 一个类如果直接父类当中的方法，和接口当中的默认方法产生了冲突，子类优先用父类当中的方法。

### 接口的多继承
一个接口能继承另一个或者多个接口。接口的继承使用extends关键字，子接口继承父接口的方法。如果父接口中的默认方法有重名，那么子接口需要重写一次。
```java
// 父接口A
public interface MyInterfaceA {
    public abstract void methodA();
    public abstract void methodCommon();
    public default void methodDefault(){
        System.out.println("接口A默认方法");
    }
}
// 父接口B
public interface MyInterfaceB {
    public abstract void methodB();
    public abstract void methodCommon();
    public default void methodDefault(){
        System.out.println("接口B默认方法");
    }
}
// 子接口
public interface MyInterface extends MyInterfaceA,MyInterfaceB{
    public abstract void method();
    @Override
    default void methodDefault() { // 重写父接口的默认方法
    }
}
```
1. 类与类之间是单继承的。直接父类只有一个
2. 类与接口之间是多实现的。一个类可以实现多个接口。
3. 接口与接口之间是多继承的。

注意事项：
1. 多个父接口当中的抽象方法如果重复，没关系
2. 多个父接口中的默认方法如果重复，那么子接口必须进行默认方法的覆盖重写，[而且带着default关键字

## 2.多态
- 多态：是指同一行为，具有多个不同表现形式。

### 2.1多态的体现
多态的体现格式：父类名称 对象名 = new 子类名称();
代码中体现多态性，其实就是一句话：父类引用指向子类对象
#### 成员变量在多态中的规则
```java
// 父类
public class Fu {
    int num = 10;
    public void showNum(){
        System.out.println(num);
    }
}
// 子类
public class Zi extends Fu {
    int num = 20;

    @Override
    public void showNum(){
        System.out.println(num);
    }
}
// 测试类
public class DemoMulti01 {
    public static void main(String[] args) {
        Fu obj = new Zi();
        System.out.println(obj.num); // 直接访问成员变量时，左边是谁就先访问谁的

        // 子类没有覆盖重写，就是父：10
        // 子类如果覆盖重写，就是子：20
        obj.showNum();
    }
}
```
注意事项：
1. 直接通过对象名称访问成员变量：看等号左边是谁优先访问谁，没有则向上查找。
2. 间接通过成员方法来访问成员变量：看该方法属于谁优先访问谁，没有则向上找。

#### 多态中成员方法的使用特点
当使用多态方式调用方法时，首先检查父类中是否有该方法，如果没有，则编译错误；如果有，执行的是子类重写后方法。
```java
// 父类
public class Fu {
    int num = 10;
    public void showNum(){
        System.out.println(num);
    }
    public void method(){
        System.out.println("父类方法");
    }

    public void methodFu(){
        System.out.println("父类特有方法");
    }
}
// 子类
public class Zi extends Fu {
    int num = 20;

    @Override
    public void showNum(){
        System.out.println(num);
    }
    @Override
    public void method(){
        System.out.println("子类方法");
    }

    public void methodZi(){
        System.out.println("子类特有方法");
    }
}
// 测试类
/*
在多态的代码中，成员方法的访问规则是：
    看new的是谁，就优先用谁，没有则向上找。

成员方法口诀：编译看左边，运行看右边。

对比一下：
成员变量：编译看左边，运行还看左边。
成员方法：编译看左边，运行看右边。
 */
public class Demo02MultiMethod {
    public static void main(String[] args) {
            Fu obj = new Zi(); // 多态
            // 运行看右边：左边是Fu，Fu有method方法，所以编译通过，但是运行时看右边是Zi，故运行子类方法。
            obj.method(); // 父子都有，优先用子
            // 运行看右：左边是Fu，Fu有methodFu方法，编译通过，但运行是看右Zi，子类没有该方法，就向上找到父类，然后执行。
            obj.methodFu(); // 子类没有，向上找到父类有

            // 编译看左边：左边是Fu，Fu中没有methodZi方法，所以编译报错。
            // obj.methodZi(); 错误写法！

    }
}
```
### 2.2 多态的好处
可以使程序编写的更简单，并且有良好的扩展性。

### 2.3 引用类型转换
多态的转型分为向上转型和向下转型两种：
#### 向上转型
- 概述：多态本身就子类类型向父类类型向上转换的过程，这个过程是默认的。
- 使用格式：父类名称 变量名 = new 子类类型(); 如 Animal a = new Cat();
- 含义：右侧创建一个子类对象，把它当做父类来看待使用。如上创建了一只猫，当做动物看待，没问题。
- 注意事项：向上转型一定是安全的，从小范围转向了大范围，从小范围的猫，向上转换成为更大范围的动物。
- 类似于：double num = 100; // 正确， int--> double， 自动类型转换。

#### 向下转型
- 概述：父类类型向子类类型向下转换的过程，这个过程是强制的。一个已经向上转型的子类对象，将父类引用转为子类引用，可以使用强制类型转换的格式，便是向下转型。
- 格式：子类名称 对象名 = (子类名称)父类对象；如 Cat cat = (Cat)a;
- 含义：将父类对象，[还原]成为本来的子类对象。


注意事项：
1. 必须保证对象本来创建的时候，就是猫，才能向下转型成为猫。
2. 如果对象创建时本来就不是猫，现在非要向下转型为猫，会报错ClassCastException
3. 类似于： int num = (int)10.0; // 可以   int num = (int)10.5 //不可以，精度损失。

#### instanceof关键字
为了避免ClassCastException的发生，Java提供了 instanceof 关键字，给引用变量做类型的校验。
格式：变量名 instanceof 数据类型; 如果变量属于该数据类型，返回true，反之返回false。
```java
// 父类
public abstract class Animal {
    public abstract void eat();
}
// 子类1
public class Cat extends Animal{

    @Override
    public void eat() {
        System.out.println("猫吃鱼");
    }

    public void jump(){
        System.out.println("猫跳跳");
    }
}
// 子类2
public class Dog extends Animal{
    @Override
    public void eat() {
        System.out.println("狗吃shit");
    }
    public void watchHouse(){
        System.out.println("看家");
    }
}
// 测试类
public class MainMethod {
    public static void main(String[] args) {
        Animal animal = new Dog();
        animal.eat();

        getAnimal(new Cat());
    }
    public static void getAnimal(Animal animal){
        if(animal instanceof Dog){
            Dog dog = (Dog)animal;
            dog.watchHouse();
        }
        if(animal instanceof Cat){
            Cat cat = (Cat)animal;
            cat.jump();
        }
    }
}
```

## 3. 接口多态的综合案例
定义USB接口，具备最基本的开启功能和关闭功能。鼠标和键盘要想能在电脑上使用，那么鼠标和键盘也必须遵守USB规范，实现USB接口，否则鼠标和键盘的生产出来也无法使用。
### 3.1 案例分析
进行描述笔记本类，实现笔记本使用USB鼠标、USB键盘
- USB接口，包含开启功能、关闭功能
- 笔记本类，包含运行功能、关机功能、使用USB设备功能
- 鼠标类，要实现USB接口，并具备点击的方法
- 键盘类，要实现USB接口，具备敲击的方法

具体代码实现如下：
```java
// 接口类
public interface USB {
    public abstract void open(); // 打开设备
    public abstract void close(); // 关闭设备
}
// 笔记本类
public class Laptop {
    public void powerOn(){
        System.out.println("笔记本电脑开机");
    }
    public void powerOff(){
        System.out.println("笔记本电脑关机");
    }

    // 使用USB设备的方法，使用接口作为方法参数
    public void usbDevice(USB usb){
        usb.open();
        if(usb instanceof Mouse){
            Mouse mouse = (Mouse)usb; // 向下转型
            mouse.click();
        }else if (usb instanceof Keyboard){
            Keyboard keyboard = (Keyboard)usb;
            keyboard.click();
        }
        usb.close();
    }
}
// 鼠标类
// 鼠标就是一个USB设备
public class Mouse implements USB{
    @Override
    public void open() {
        System.out.println("打开鼠标");
    }

    @Override
    public void close() {
        System.out.println("关闭鼠标");
    }

    public void click(){
        System.out.println("鼠标点击");
    }
}
// 键盘类
// 键盘也是一个USB设备
public class Keyboard implements USB{
    @Override
    public void open() {
        System.out.println("打开键盘");
    }

    @Override
    public void close() {
        System.out.println("关闭键盘");
    }

    public void click(){
        System.out.println("键盘输入");
    }
}
// 测试类
public class DemoMain {
    public static void main(String[] args) {
        // 首先创建一个电脑
        Laptop laptop = new Laptop();
        laptop.powerOn();

        // 准备一个鼠标,供电脑使用
        USB usbMouse = new Mouse(); // 首先向上转型
        laptop.usbDevice(usbMouse);

        // 创建一个USB键盘
        Keyboard keyboard = new Keyboard(); //没有使用多态写法
        // 方法参数是USB类型，传递进去的是实现类对象
        laptop.usbDevice(keyboard); // 正确写法！ 自动发生了向上转型！

        laptop.powerOff();
    }
}
```
该案例主要练习对接口的基本使用，对象的上下转型以及接口作为对象参数的使用。

### 今日总结
1. 接口的概述以及定义格式，以及实现接口的格式。interface关键字，implements关键字
2. 接口中的抽象方法，默认方法和静态方法，私有方法和私有静态方法的各自使用特点。
3. 接口中成员方法，成员变量的特点。
4. 接口的多继承。
5. 多态的概念以及前提。
6. 多态中的向上向下转型格式以及实际使用方法。
7. instanceof关键字。