---
title: Java的final关键字及内部类
date: 2019-4-22 22:00
categories: JavaBasics # 分类
tags: [Java]
description: Java中的final关键字，权限、内部类和引用类型
urlname: final-innerClass
---


## 1. final关键字
继承中子类可以在父类的继承上改写父类内容，比如方法重写。但如果我们随意的继承API中提供的类，改写其内容，显示不合适。为了避免这种随意改写的起来，Java提供了final关键字，用于修改不可改变内容。

<!--more-->

- final：不可改变。可用于修饰类、方法和变量。
    - 类：被修改的类，不能被继承。
    - 方法：被修饰的方法，不能被重写。
    - 变量：被修饰的变量，不能被重新赋值。

### 1.1 使用方式
- 修饰类：final class 类名{}
- 修饰方法：修饰符 final 返回值类型 方法名(参数列表){方法体}
- 修饰变量有以下三种情况：
    - 局部变量--基本类型：基本类型的局部变量，被final修饰后，只能赋值一次，不能再更改。
    - 局部变量--引用类型：引用类型的局部变量，被final修饰后，只能指向一个对象，地址不能再更改。但是不影响对象内部成员变量值的修改。
    - 成员变量：成员变量涉及到初始化的问题，初始化方式有两种，显式初始化和构造方法初始化。

代码示例如下：
```java
public class FinalDemo01 {
    public static void main(String[] args) {
        // 1. 以下示意局部变量基本类型使用final关键字修饰。
        final int a; // 使用final修饰，声明变量。
        a = 10; // 第一次赋值，编译通过
//        a = 20; // 第二次赋值，报错。
        final int b = 10; // 申明变量同时直接赋值，使用final修饰
//        b = 20; // 报错,不可重新赋值

        for(int i = 0; i < 10;i++){
            final int c = i;
            System.out.print(c);
        }

        // 2. 以下示例局部变量引用类型使用final关键字修饰。
        final TestFinal tf = new TestFinal("张三",11); // 创建一个测试对象。
//        tf = new TestFinal(); // 新创建一个对象，把tf指向该对象。报错，使用final修饰后无法改变地址值。

    }
}
// 成员变量initialize
public class TestFinal {
    // 创建类时直接初始化，并用final修饰。
/*    final String USERNAME = "张山";
    private int age;
*/
    final String USERNAME;
    private int age;

    // 使用构造方法initialize
    public TestFinal(String USERNAME, int age) {
        this.USERNAME = USERNAME;
        this.age = age;
    }
}
```

## 2. 权限修饰符
在Java中提供了四种访问权限，使用不同的访问权限修饰符修饰时，被修饰的内容会有不同的访问权限。
- public：公共的
- protected：受保护的
- default：默认的
- private：私有的

### 2.1 不同权限的访问能力
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/22/Java%E5%9B%9B%E7%A7%8D%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90-1555902457643.png)<br>
public具有最大权限。private则是最小权限。
故编写代码时，如果没有特殊的考虑，应该这样使用权限：
- 成员变量使用private，隐藏细节。
- 构造方法使用public，方便创建对象。
- 成员方法使用public，方便调用方法。
- 不加权限修饰符的，其访问能力与default修饰符相同。

## 3. 内部类
如果一个类A定义在另一个类B里面，里面的那个类A就称为内部类，B则称为外部类。
### 成员内部类：定义在类中**方法外**的类。
在描述事物时，若一个事物内部还包含其他事物，就可以使用内部类这种结构。比如，汽车类Car中包含发动机类Engine，这个Engine类就可以使用内部类来描述，定义在成员位置。
```java
class Car{ //外部类 
	...
    class Engine{ //成员内部类
	...
    }
}
```
- 内部类可以直接访问外部类的成员，包括私有成员。
- 访问重名的成员变量，使用格式：外部类名称.this.外部类成员变量名
如何使用成员内部类？两种方式：
1. 间接方式：在外部类的方法中，使用内部类；然后通过调用外部类方法来间接调用内部类。
2. 直接方式：[外部类名称.内部类名称 对象名 = new 外部类名称().new 内部类名称();]

```java
// 成员内部类示例
public class Body { // 内部类

    public class Heart{ // 成员内部类

        // 内部类方法
        public void beat(){
            System.out.println("心脏跳动：砰砰砰！");
            System.out.println("我叫：" + name); // 内部类可以直接访问外部类成员变量
        }
    }

    private String name; // 外部类成员变量

    // 外部类方法
    public void methodBody(){
        System.out.println("外部类的方法");
        new Heart().beat(); // 匿名对象
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
// 成员内部类示例测试类
public class InnerClassDemo02 {
    public static void main(String[] args) {
        Body body = new Body();
        body.methodBody(); // 间接通过外部类方法访问内部类
        System.out.println("下面使用直接访问方式访问内部类");
        Body.Heart heart = new Body().new Heart();
        heart.beat();
    }
}

// 访问重名的成员变量示例
public class Other {

    int num = 10; // 外部类的成员变量

    public class Inner{
        int num = 20; // 内部类的成员变量
        public void methodInner(){
            int num = 30; // 内部类方法的局部变量
            System.out.println(num); // 局部变量，就近原则
            System.out.println(this.num); // 内部类成员变量
            System.out.println(Other.this.num); // 外部类成员变量
        }
    }
}
// 访问重名的成员变量示例测试类
public class InnerClassDemo03 {
    public static void main(String[] args) {
        Other.Inner obj = new Other().new Inner();
        obj.methodInner();
    }
}
```

### 局部内部类
如果一个类是定义在一个**方法内部**的，那么这就是一个局部内部类
“局部”：只有当前所属的方法才能使用它，出了这个方法外面就不能用了。
```java
// 局部内部类
public class Outer {

    public void methodOuter(){
        class Inner{ // 局部内部类
            int num = 10;
            public void methodInner(){
                System.out.println(num);
            }
        }
        Inner inner = new Inner(); // 创建局部内部类对象
        inner.methodInner(); // 调用内部类方法,将打印10
    }
}
// 测试类
public class DemoMain {
    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.methodOuter();
    }
}
```
小结类的权限修饰符：
public > protected > (default) > private
定义一个类时，权限修饰符规则：
1. 外部类：public / (default)
2. 成员内部类：public / protected / (default) / private
3. 局部内部类：什么都不能写

注意事项：
局部内部类，如果希望访问所在方法的局部变量，那么这个局部变量必须是[有效的final的]
备注：从Java 8+开始，只要局部变量属实不变，那么final关键字可省略
原因：
1. new出来的对象在堆内存中。
2. 局部变量是跟着方法走的，在栈内存当中。
3. 方法运行结束之后，立即出栈，局部变量就会立即消失。
4. 但new出来的对象会在堆中持续存在，直到垃圾回收消失。

### 匿名内部类(重点)
- 匿名内部类：是内部类的简化写法。它本质是一个**带具体实现的父类或者父接口的匿名**的子类对象。
开发中，最常用的内部类就是匿名内部类。以接口为例，当使用一个接口时，得做如下几步操作：
1. 定义子类
2. 重写接口中的方法
3. 创建子类对象
4. 调用重写后的方法
我们最终的目的只是为了调用方法，那么该如何简化。匿名内部类就可以处理这种情况。

```java
// 接口类
public interface MyInterface {
    void method();
    void method2();
}
/*
匿名内部类的定义格式：
接口名称 对象名 = new 接口名称(){
    // 覆盖重写所有抽象方法
}

对格式 “new 接口名称() {...}” 进行解析：
1. new代表创建对象的动作
2. 接口名称就是匿名内部类需要实现哪个接口
3. {...} 这是匿名内部类的内容

另外还有注意几点问题：
1. 匿名内部类，在创建对象时，只能使用唯一一次。
如果希望多次创建对象，而且类的内容一样的话，那么就必须使用单独定义的实现类了。
2. 匿名对象，在[调用方法]时，只能调用唯一一次，如果希望同一对象调用多次方法，那么必须给对象起名。
3. 匿名内部类是省略了[实现类/子类名称]，但匿名对象是省略了[对象名称]。匿名内部类和匿名对象不是一回事！
 */
// 主方法测试类
public class DemoMain {
    public static void main(String[] args) {
        // 使用匿名内部类
        MyInterface obj = new MyInterface() {
            @Override
            public void method() {
                System.out.println("使用匿名内部类实现了方法1！");
            }

            @Override
            public void method2() {
                System.out.println("匿名内部类实现方法2");
            }
        };
        obj.method();
        obj.method2();

        // 使用了匿名内部类，而且省略了对象名称，也是匿名对象
        new MyInterface(){

            @Override
            public void method() {
                System.out.println("使用匿名对象方式，该内部类方法只能调用一次");
            }

            @Override
            public void method2() {

            }
        }.method(); // 因为匿名对象无法调用第二次方法，所以需要再创建一个匿名内部类的匿名对象才能二次调用。
    }
}
```

## 4. 引用类型用法总结
实际开发中，引用类型的使用非常重要且普遍。基本类型可以作为成员变量、作为方法的参数、作为方法的返回值，那么引用类型也是可以的。

### 4.1 class作为成员变量
具体代码示例如下：
```java
// 定义一个英雄类
public class Hero {
    private String name; // 名称
    private int blood;  // 生命值
    private Weapon weapon; // 添加武器属性

    public Hero() {
    }

    public Hero(String name, int blood, Weapon weapon) {
        this.name = name;
        this.blood = blood;
        this.weapon = weapon;
    }

    public void attack(){
        System.out.println(name+"使用了"+weapon.getName()+"攻击敌方");
    }

	... 省略get/set方法
}
// 定义一个武器类
public class Weapon {
    private String name; // 武器名称

    public Weapon(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
// 测试类
public class DemoMain {
    public static void main(String[] args) {
        // 创建英雄对象
        Hero hero = new Hero();
        // 创建武器对象
        Weapon wp = new Weapon("霜之哀伤");
        hero.setName("伊利丹"); // 设置英雄名字
        hero.setWeapon(wp); // 将武器交给英雄
        hero.attack(); // 攻击
    }
}
```

### 4.2 interface作为方法参数和返回值类型
以下示意接口作为方法参数：
```java
// 接口类
public interface Skill {
    void use(); // 释放技能
}
// 实现类 
public class SkillImpl implements Skill {
    @Override
    public void use() {
        System.out.println("biubiubiu");
    }
}
// 英雄类
public class Hero {
    private String name; // 英雄名称
    private Skill skill; // 英雄技能

    public Hero() {
    }

    public Hero(String name, Skill skill) {
        this.name = name;
        this.skill = skill;
    }

    public void attack(){
        System.out.println("我叫"+name+"，开始释放技能：");
        skill.use(); //调用接口中的抽象方法
        System.out.println("释放技能完成。");
    }
	....省略get/set方法
}
// 测试类
public class DemoGame {
    public static void main(String[] args) {
        Hero hero = new Hero();
        hero.setName("艾希"); // 设置英雄名称

        // 设置英雄技能
        // 使用实现类作为参数传递
//        hero.setSkill(new SkillImpl());
//        hero.attack();
        // 使用匿名内部类作为参数
//        Skill skill = new Skill() {
//            @Override
//            public void use() {
//                System.out.println("pia~~pia~~pia~~");
//            }
//        };
        // 使用匿名内部类和匿名对象
        hero.setSkill(new Skill() {
            @Override
            public void use() {
                System.out.println("da~da~da~");
            }
        });
        hero.attack();

    }
}
```

以下示例接口作为方法的返回值类型
```java
public class DemoInterface {
    public static void main(String[] args) {
        // 左边是接口名称，右边是实现类名称，这就是多态写法
        List<String> list = new ArrayList<>(); // 

        List<String> result = addNames(list);
        for (int i = 0; i < result.size(); i++) {
            System.out.println(result.get(i));
        }
    }

    public static List<String> addNumbers(List<String> list){
        list.add("1");
        list.add("2");
        list.add("3");

        return list;
    }
}
// 用一个接口或是自定义类作为方法的参数或者返回值都是可以的
```

### 5. 综合案例--发红包[界面版]
完整代码见：[SendRedPacket](https://github.com/zero6996/JavaDemo/tree/master/Demo4_22/DemoRedPacket)

手气红包算法思路解析：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/22/%E6%89%8B%E6%B0%94%E7%BA%A2%E5%8C%85%E7%AE%97%E6%B3%95%E6%80%9D%E8%B7%AF%E8%A7%A3%E6%9E%90-1555941568465.png)

### 案例总结
通过发红包案例，你都学到了什么？请思考如下问题：
1. 基础语法，你是否清晰？
2. 一些基本的类的方法，你是否能够调用
3. 案例中哪里体现了继承，继承的作用是什么？
4. 接口作为参数，如何使用？
5. 接口作为成员变量，如何使用?
6. 如何简化接口的使用方式？

## 今日总结
1. 使用final关键字修饰类、方法和变量。
2. Java中的权限修饰符。
3. 成员内部类、局部内部类、匿名内部类、匿名对象的基本使用格式。
4. 引用类型作为参数，返回值类型的使用方式。
5. 综合案例发红包代码理解。