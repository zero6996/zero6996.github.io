---
title: Java三大特性之继承
date: 2019-4-20 16:00
categories: JavaBasics # 分类
tags: [三大特性]
description: Java中的继承和抽象类相关内容
---

## 概述
多个类中存在相同属性和行为时，将这些内容抽取到单独一个类中，那么多个类无需再定义这些属性和行为，只要继承那一个类即可。其中，多个类可以称为子类，单独那个类称为父类、超类或者基类。

<!--more-->

### 1. 继承
- 定义：继承就是子类继承父类的属性和行为，使得子类对象具有与父类相同的属性、相同的行为。子类可以直接访问父类中的非私有的属性和行为。
- 优点：提高了代码的复用性，类与类之间产生了关系，是多态的前提。

### 1.1 继承的格式
通过extends关键字，可以声明一个子类继承另外一个父类，实例如下：
```java
//定义父类
public class Employee {
    String name;
    public void work(){
        System.out.println("努力工作");
    }
}
// 定义子类继承父类
class Teacher extends Employee{
    public void printName(){
        System.out.println("name:" + name);
    }
}
// 测试
public class Demo01Extends {
    public static void main(String[] args) {
        Teacher t = new Teacher();
        t.name = "小明";
        t.printName();
        t.work();
    }
}
```
### 1.2 继承后的成员变量
- 如果子类父类中成员变量未重名，访问没有影响。
- 如果成员变量有重名的，访问会有影响，需要使用super关键字修饰父类成员变量。实例如下：
```java
public class Demo02Super {
    public static void main(String[] args) {
        Zi z = new Zi();
        z.show();
    }
}

class Fu{
    int num = 5;
}
class Zi extends Fu{
    int num = 6;
    public void show(){
        // 访问父类中的num
        System.out.println("父类num：" + super.num); //使用super关键字即可访问父类成员变量
        //访问子类中的num
        System.out.println("子类num：" + this.num);
    }
}
```
注意：
1. 父类中的成员变量是非私有时，子类可以直接访问。若父类成员变量私有了，子类是不能直接访问的，可以通过父类设置的getxxx/setxxx方法间接访问。

### 1.3 继承后的成员方法。
1. 如果子类父类中成员方法不重名，这时调用是没有影响的。对象在调用方法时，会现在子类寻找对应方法，若子类存在就执行子类中的方法，若子类不存在该方法就会执行父类中对应的方法。
2. 如果子类父类中成员方法重名，这时访问会造成一种特殊情况，叫做方法重写。
3. 重写(Override)：子类中出现与父类一模一样的方法时(返回值类型，方法名和参数列表都相同)，会出现覆盖效果，也称为覆盖或者覆写。声明不变，重新实现。

#### 重写的应用
子类可以根据需要，定义属于自己的行为。即沿袭了父类的功能名称，有根据子类需要重新实现父类方法，从而进行扩展。
```java
// 父类
public class Phone {
    public void call(){
        System.out.println("打电话");
    }
    public void sendMessage(){
        System.out.println("发短信");
    }
    public void showNum(){
        System.out.println("来电显示号码");
    }
}
// 子类继承父类，并重写父类方法，扩展自己功能
public class NewPhone extends Phone{
    public void showNum(){
        super.showNum();
        System.out.println("来电显示姓名");
        System.out.println("来电显示头像");
    }
}
// 实例类
public class Demo04Extends {
    public static void main(String[] args) {
        NewPhone ph = new NewPhone();
        ph.call();
        ph.sendMessage();
        ph.showNum();
    }
}
```
方法覆盖重写的注意事项：
1. 必须保证父子类之间方法的名称相同，参数列表也相同。
@Override:写在方法前面,检测是否是有效的正确覆盖重写，可选
2. 子类方法的返回值必须小于等于父类方法的返回值范围。
扩展：java.lang.Object类是所有类的公共最高父类(祖宗类),String是Object的子类。
3. 子类方法的权限必须大于等于父类方法的权限修饰符。
扩展：public > protected > (default) > private 注：(default)不是关键字default，而是什么都不写，留空。

### 1.4 继承关系中构造方法的访问特点
1. 子类构造方法当中有一个默认隐含的"super()" 调用，所以一定是先调用父类构造，后执行的子类构造。
2. 子类构造可以通过super关键字来调用父类重载构造
3. super的父类构造调用，必须是子类构造方法的第一个语句。子类构造不能调用多次super构造。
小结：子类必须调用父类构造方法，不写则赠送super(); 写了则用写的super调用，super有且只能有一个，还必须是第一个。

### 1.5 super和this关键字用法小结
super关键字三种用法：
1. 在子类的成员方法中，访问父类的成员变量。super.父类成员变量;
2. 在子类的成员方法中，访问父类的成员方法。super.父类成员方法();
3. 在子类的构造方法中，访问父类的构造方法。super();
this关键字三种用法：
1. 在本类的成员方法中，访问本类的成员变量。this.本类成员变量;
2. 在本类的成员方法中，访问本类的另一个成员方法。this.本类成员方法();
3. 在本类的构造方法中，访问本类的另一个构造方法。this(...);
注意：
A. this(...) 调用也必须是构造方法的第一个语句，且唯一一个。
B. super和this两种构造调用，不能同时使用。
```java
public class Zi extends Fu{
    int num = 20;

    public Zi(){
        this(8); // 本类的无参构造，调用本类的有参构造
    }
    public Zi(int n){

    }

    public void showNum(){
        int num = 10;
        System.out.println(num); // 局部变量
        System.out.println(this.num); // 本类成员变量
        System.out.println(super.num); //  父类成员变量
    }
    public void methodA(){
        System.out.println("AAA");
    }
    public void methodB(){
        this.methodA(); // 调用本类成员方法
        System.out.println("BBB");
    }
}
```
super和this内存图
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/20/super%26this%E5%86%85%E5%AD%98%E5%9B%BE-1555741452451.png)
### 1.6 Java继承的三个特点
1. Java语言只支持单继承，一个类的直接父类只能有唯一个
2. Java支持多层继承(继承体系)
3. 子类和父类是一种相对的概念

## 2. 抽象类
父类中的方法，被它的子类覆盖重写，子类各自的实现都不尽相同。那么父类的方法声明和方法主体，只有声明还有意义，而方法主体则没有存在的意义了。我们把没有方法主体的方法称为**抽象方法**。Java语法规定，包含抽象方法的类就是**抽象类**。
- 抽象方法：没有方法体的方法。
- 抽象类：包含抽象方法的类。

### 2.1 abstract使用格式
### 2.2 抽象方法
 - 使用abstract关键字修饰方法，该方法就成了抽象方法，抽象方法只包含一个方法名，而没有方法体。
 - 定义格式：修饰符 abstract 返回值类型 方法名(参数列表);
 - 代码示例：public abstract void run();
### 2.3 抽象类
- 如果一个类包含抽象方法，那么该类必须是抽象类。
```java
定义格式：
abstract class 类名称{
	...
}
代码示例：
public abstract class Animal{ // 抽象类
    public abstract void run(); // 抽象方法
}
```
### 2.4 抽象的使用
1. 不能使用new来直接创建抽象类对象。
2. 必须用一个子类来继承抽象父类。
3. 子类必须覆盖重写抽象父类当中所有的抽象方法。覆盖重写(实现方法)：子类去掉抽象方法的abstract关键字，然后补上方法体大括号。
4. 创建子类对象进行使用。
```java
// 抽象类
public abstract class Animal {
    public abstract void eat(); // 抽象方法
}
// 子类继承抽象类
public class Cat extends Animal{
    @Override
    public void eat(){ // 重写抽象方法，并补上大括号和具体实现
        System.out.println("猫吃鱼");
    }
}
// 实例
public class DemoMain {
    public static void main(String[] args) {
        Cat cat = new Cat(); // 创建子类对象
        cat.eat(); // 调用方法
    }
}
```
### 2.5 注意事项
关于抽象类的使用，注意事项总结：
1. 抽象类不能创建对象。只能创建其非抽象子类的对象。原因：抽象类的抽象方法没有具体方法体，没有意义
2. 抽象类中，可以有构造方法，是供子类创建对象时，初始化父类成员使用的。因为子类的构造方法中，有默认的super(); 会调用父类构造方法。
3. 抽象类中，不一定包含抽象方法，但有抽象方法的类必定是抽象类。未包含抽象方法的抽象类，目的就是不想让调用者创建该类对象，通常用于某些特殊的类结构设计。
4. 抽象类的子类，必须重写父类中所有的抽象方法，否则会报错。除非该子类也是抽象类。

## 3. 继承的综合案例
群主发普通红包。某群有多名成员，群主发红包。规则：
1. 群主有一笔金额，从群主余额扣除，平均分为n份，让成员领取。
2. 成员领取红包后，保存到成员余额中。
根据描述，完成案例中所有类的定义以及指定类之间的继承关系，并完成发红包的操作。

### 分析案例
根据描述分析，得出如下继承体系：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/20/%E5%8F%91%E7%BA%A2%E5%8C%85%E6%A1%88%E4%BE%8B%E7%BB%A7%E6%89%BF%E4%BD%93%E7%B3%BB-1555750639750.png)<br>
代码实现：
```java
// 定义父类Users
public class Users {
    private String name;
    private double money;

    public Users() {
    }

    public Users(String name, double money) {
        this.name = name;
        this.money = money;
    }
    public void show(){
        System.out.println("我叫"+name+"，我有"+money+"块钱");
    }
    ... ...省略get/set方法
}
// 定义群主类
// 群主类
public class Manager extends Users{
    public Manager(){ // 无参构造方法

    }

    public Manager(String name, double money) { // 全参构造方法
        super(name, money); // 调用了父类的构造方法，并初始化值
    }

    // 定义群主发红包方法，参数为发红包金额和份数
    public ArrayList<Double> send(double totalMoney, int count){
        // 首先需要一个集合，用来存储若干个红包的金额
        ArrayList<Double> redList = new ArrayList<>();
        //首先看一下群主有多少钱
        double leftMoney = super.getMoney(); // 群主当前余额
        if(totalMoney > leftMoney){
            System.out.println("余额不足");
            return redList; // 返回空集合
        }

        //扣钱，其实就是重置余额
        super.setMoney(leftMoney - totalMoney);

        //发红包需平均拆分为count份
        double avg = (int)totalMoney / count; // int化后丢失精度,如何把零头单独拿出来？
        double mod = totalMoney - avg * count;//余数，也就是甩下的零头
        // 除不来的零头，包在最后一个红包里
        // 下面把红包一个一个放到集合中
        for (int i = 0; i < count - 1; i++) {
            redList.add(avg);
        }
        double last = avg + mod; //最后一个红包
        redList.add(last);

        return redList; // 返回的是一个红包数组集合
    }
}

//定义群员类
// 成员类
public class Member extends Users{
    public Member(){

    }

    public Member(String name, double money) {
        super(name, money);
}

    public void receive(ArrayList<Double> list){
        // 从多个红包当中随机抽取一个，给我自己。
        // 随机获取一个集和当中的索引编号
        int index = new Random().nextInt(list.size());
        // 根据索引，从集合中删除，并且得到被删除的红包值，给自己钱
        double delta = list.remove(index);
        // 成员查看一下自己的余额
        double money = super.getMoney();
        // 将得到的红包金额加入自己的余额
        super.setMoney(delta + money);
    }

}

// 定义主方法实例
public class MainRedPacket {
    public static void main(String[] args) {
        Manager manager = new Manager("群主",100.765);
        Member one = new Member("成员A",0);
        Member two = new Member("成员B",0);
        Member three = new Member("成员C",0);

        manager.show();
        one.show();
        two.show();
        three.show();
        System.out.println("===================");
        ArrayList<Double> redList = manager.send(55.687,3);
        one.receive(redList);
        two.receive(redList);
        three.receive(redList);

        manager.show();
        one.show();
        two.show();
        three.show();
    }
}
```

### 总结
1. Java继承的概念，特点
2. extends关键字的使用
3. 继承后成员变量成员方法构造方法的访问特点
4. 方法的覆盖重写
5. super和this关键字的使用
6. 抽象类，抽象方法的概念
7. abstract关键字的使用
8. 综合案例练习