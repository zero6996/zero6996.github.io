---
title: Java中的Object类和常用API
date: 2019-4-23 23:59
categories: JavaBasics # 分类
tags: [Java]
description: Java当中Object类和常用API概述
---

## 1. Object类
`java.lang.Object`类是Java语言中的根类，即所有类的父类。它中描述的所有方法子类都可以使用。在对象实例化的时候，最终找到的父类就是Object。
根据JDK源码及Object类的API文档，Object类中包含的方法有11个。本章主要学习其中的2个：
* `public String toString()`; 返回该对象的字符串表示。
* `public boolean equals(Object obj)`:指示其他某个对象是否与此对象"相等"。

<!--more-->

## 1.2 toString方法
toString方法返回该对象的字符串表示，其实返回的字符串内容就是对象的类型+@+内存地址值。`Person@3feba861`
由于该方法返回的是内存地址，所以一般要重写它以获取对象属性。
```java
public class Person {
    private String name;
    private int age;

    @Override
    public String toString() { // 重写toString方法
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

	省略get/set

}
```
在IntelliJ IDEA中，可以使用快捷键`ALT+Insert`,点击`toString()`选项，自动生成该方法。

## 1.3 equals方法
* `public boolean equals(Object obj)`:比较两个对象是否“相等”，返回布尔值。
### 默认地址比较
如果没有覆盖重写equals方法，那么object类默认进行`==`运算符的对象地址比较，只要不同就返回false。

### 对象内容比较
如果希望进行对象的内容比较，可以重写equals方法。
```java
@Override
    public boolean equals(Object o) {
        if (this == o) return true;
        // 如果o不等于空并且o不等于Person类型，返回false。 getClass() != o.getClass()  使用反射技术，判断o是否是Person类型， 等效于 obj instanceof Person
        if (o == null || getClass() != o.getClass()) return false;
        // 向下转型
        Person person = (Person) o;
        /* Object.equals(Object a, Object b); equals方法重载解析：
        Objects.equals(name, person.name); // 传入两个对象参数
        public static boolean equals(Object a, Object b) {
               // 判断a==b 或者 a不等于空，并且a与b的内存地址相同才返回true，反之false
            return (a == b) || (a != null && a.equals(b));
        }
         */
        return age == person.age && Objects.equals(name, person.name);
    }
```
使用快捷键`ALT+Insert`可以自动生成该代码。

## 1.4 Oebjects类
在**JDK7**添加了一个Objects工具类，它提供了一些方来操作对象，它由一些静态的实用方法组成，这些方法是null-save(空指针安全的)或null-tolerant(容忍空指针的),用于计算对象的hashcode、返回对象的字符串表示形式、比较两个对象。

在比较两个对象时，Object的equals方法容易抛出空指针异常，而Objects类中的equals方法就优化了这个问题。方法如下：
* `public static boolean equals(Objects a,Objects b)`:判断两个对象是否相等。
源码解析：
~~~java

    Object.equals(Object a, Object b); // 对传入的两个对象进行比较，可防止空指针异常。
    Objects.equals(name, person.name); // 传入两个对象参数
    public static boolean equals(Object a, Object b) {
           // 判断a==b 或者 a不等于空，并且a与b的内存地址相同,才能返回true，反之false。
        return (a == b) || (a != null && a.equals(b));
 }
~~~

## 2. 日期时间类
### 2.1 Date类
`java.util.Date`类表示特点的瞬间，精确到毫秒。
Date中拥有多个构造方法，部分已过时，其中有未过时的构造方法可以把毫秒值转成日期对象。
- `public Date()`:分配Date对象并初始化此对象，以表示分配它的时间(精确到毫秒)。获取当前系统的日期和时间。
- `public Date(long date)`:传递毫秒值，把毫秒值转换为Date日期，以表示自从标准基准时间(称为"历元(epoch)"，即1970年1月1日00:00:00 GMT)以来的指定毫秒数。
* `public long getTime()`：把日期对象转换成对应的时间毫秒值。
> Tips:由于我们处于东八区，所以我们的基准时间为1970年1月1日8时0分0秒。

```java
import java.util.Date;

public class Demo01Date {
    public static void main(String[] args) {
        System.out.println("当前系统时间"+new Date());
        System.out.println("将传入的毫秒值转换为日期时间格式"+new Date(0L)); // Tue Apr 23 20:39:00 CST 2019
        System.out.print("把日期转换为毫秒值"+new Date().getTime()); // 相当于System.currenTimeMillis()方法
    }
}
```

> Tips:在使用println方法时，会自动调用Date类中的toString方法。Date类重写了toString方法，所以结果为指定格式的字符串。

## 2.2 DateFormat类
`java.text.DateFormat`是日期/时间格式子类的抽象类，通过这个类可以帮我们完成日期和文本之间的转换，也就是可以在Date对象与String对象之间进行来回转换。
* 格式化：按照指定的格式，从Date对象转换为String对象。
* 解析：按照指定的格式，从String对象转换为Date对象。

### 构造方法
由于DateFormat为抽象类，不能直接使用，所以需要常用的子类`java.text.SimpleDateFormat`。这个类需要一默认(格式)来指定格式化或解析的标准。构造方法为：
* `public SimpleDateFormat(String pattern)`:用指定的模式和默认语言环境的日期格式符号构造SimpleDateFormat。参数pattern是一个字符串，代表日期时间的自定义格式。

### 格式规则
常用格式规则为：<br>![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/23/pattern%E6%A0%BC%E5%BC%8F-1556026542510.png)
> 更详细的格式规则，可参考SimpleDateFormat类的API文档。
创建一个SimpleDateFormat对象代码如下：
~~~java
import java.text.DateFormat;
import java.text.SimpleDateFormat;

public class Demo03DateFormat {
    public static void main(String[] args) {
        DateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    }
}
~~~

### 常用方法
- `public String format(Date date)`:将Date对象格式化为字符串。
- `public Date parse(String source)`:将字符串解析为Date对象。

#### format方法
使用DateFormat类中的方法format，将日期格式化为文本，使用步骤如下。
1. 创建SimpleDateFormat对象，构造方法中传入指定模式
2. 调用SimpleDateFormat对象中的方法format，按照构造方法中的模式，把Date日期格式转换为符合格式的字符串
```java
    // 1. 创建SimpleDateFormat对象，构造方法中传入指定模式
    DateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    // 2. 调用SimpleDateFormat对象中的方法format
    String a = format.format(new Date());
    System.out.println(a);  // 2019-04-23 22:04:11
```

#### parse方法
使用DateFormat类中的方法parse，把文本解析为日期，使用步骤如下：
1. 创建SimpleDateFormat对象，构造方法中传入指定模式
2. 调用SimpleDateFormat对象中的方法parse，把符合构造方法中模式的字符串，解析为Date日期

~~~java
DateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
Date date = sdf.parse("1996-06-03 11:45:03");
System.out.println(date);
/*
注意：
public Date parse(String source) throws ParseException
parse方法声明了一个异常叫ParseException,如果字符串和构造方法的模式不一致，那么程序就会抛出异常。
调用了一个抛出异常的方法，就必须处理这个异常，要么throws继续抛出这个异常，要么try catch自己处理。
*/
~~~

## 2.3 练习
使用日期时间相关API，计算一个人出生了多少天
```java

import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Scanner;

public class Demo02Test {
    public static void main(String[] args) throws ParseException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入你的出生日期，格式：年-月-日");
        String Birthday = sc.next();
        long CurrentTime = new Date().getTime(); // 现在时间
        DateFormat format = new SimpleDateFormat("yyyy-MM-dd"); // 设置时间格式
        long liveTime = CurrentTime - format.parse(Birthday).getTime(); // 使用parse把文本解析为日期格式在转换为毫秒值
//        System.out.println(liveTime);
        System.out.println("出生了"+liveTime/1000/60/60/24+"天"); // liveTime/1000/60/60/24 把毫秒值转换为天
    }
}
```

## 2.4 Calendar类
`java.util.Calendar`是日历类，在Date后出现，替换掉了许多Date方法。该类将所有可能用到的时间信息封装为静态成员变量，方便获取。日历类就是方便获取各个时间属性的。

### 获取方式
Calenday为抽象类，由于语言敏感性，Calendar类在创建对象时并非直接创建，而是通过静态方法创建，返回子类对象，如下：
Calendar静态方法
* `public static Calendar getInstance()`:使用默认时区和语言环境获取一个日历。
示例代码：
~~~java
import java.util.Calendar;

public class Demo04Calendar {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance(); // 多态

    }
}
~~~

### 常用方法
根据Calendar类的API文档，常用方法有：
- `public int get(int field)`:返回给定日历字段的值。
- `public void set(int field, int value)`:将给定的日历字段设置为给定值。
- `public abstract void add(int field,int amount)`:根据日历规则，为给定的日历字段添加或减去指定的时间量。
- `public Date getTime()`:返回一个表示此Calendar时间值的Date对象,返回从历元到现在的毫秒偏移量。

Calendar类中提供很多成员常量，代表给定的日历字段：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/23/Calendar%E7%B1%BB%E4%B8%AD%E7%9A%84%E6%88%90%E5%91%98%E5%B8%B8%E9%87%8F-1556029579519.png)

#### get/set方法
get方法用来获取指定字段的值，set方法用来设置指定字段的值，演示代码如下：
```java
import java.util.Calendar;

public class Demo04Calendar {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance(); // 多态
        int year = cal.get(Calendar.YEAR); // 获取当前年份
        int month = cal.get(Calendar.MONTH)+1; //因为国外月份是0~11，我们是1~12.
        int day = cal.get(Calendar.DAY_OF_MONTH); // 当前天数
//        System.out.println(day);
//        System.out.println(year+"年"+month+"月"+day+"号");
        cal.set(Calendar.YEAR,2020); //自定义日期
        cal.set(Calendar.MONTH,06);
        cal.set(Calendar.DAY_OF_MONTH,03);
        int year1 = cal.get(Calendar.YEAR);
        int month1 = cal.get(Calendar.MONTH);
        int day1 = cal.get(Calendar.DAY_OF_MONTH);
        System.out.println(year1+"年"+month1+"月"+day1+"号");
    }
}
```

#### add方法
add方法可以对指定日历字段的值进行加减操作，如果第二个参数为整数则加上偏移量，负数则减去偏移量，演示代码如下：
~~~java
import java.util.Calendar;

public class Demo05CalendarAddMethod {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        int year = cal.get(Calendar.YEAR);
        int month = cal.get(Calendar.MONTH)+1;
        int day = cal.get(Calendar.DAY_OF_MONTH);
        System.out.println(year+"年"+month+"月"+day+"号");
        // 使用add方法
        cal.add(Calendar.YEAR,1); // 当前年份+1
        cal.add(Calendar.MONTH,2);
        cal.add(Calendar.DAY_OF_MONTH,-20); // 天数-20天
        int year1 = cal.get(Calendar.YEAR);
        int month1 = cal.get(Calendar.MONTH)+1;
        int day1 = cal.get(Calendar.DAY_OF_MONTH);
        System.out.println(year1+"年"+month1+"月"+day1+"号");

    }
}
~~~

#### getTime方法
Calendar中的getTime方法并不是获取毫秒时刻，而是拿到对应的Date对象。
```java
import java.util.Calendar;
import java.util.Date;

public class Demo06CalendarGetTime {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        Date date = cal.getTime(); // 获取的是Date对象
        System.out.println(date); // Tue Apr 23 22:52:25 CST 2019
    }
}
```

## 3. System类
`java.lang.System`类中提供了大量的静态方法，可以获取与系统相关的信息或者系统级操作，常用方法有：
- `public static long currentTimeMillis()`:返回以毫秒为单位的当前时间。
- `public static void arraycopy(Object src,int srcPos,Object dest, int destPos, int length)`:将数组中指定的数据拷贝到另一个数组中。

### 3.1 currentTimeMillis方法
currentTimeMillis方法获取的是当前系统时间与1970年01月01日00:00(计算机元年)之间的毫秒差值。
```java
public class Demo07CurrentTime {
    public static void main(String[] args) {
        System.out.println(System.currentTimeMillis());
    }
}
```

### 小练习1
计算for循环打印数字1~9999所需的时间(毫秒)。
~~~java
public class Demo08SystemTest {
    public static void main(String[] args) {
        long CurrentTime = System.currentTimeMillis();
        for(int i = 1;i < 10000;i++){
            System.out.println(i);
        }
        System.out.println("打印1~9999花费"+(System.currentTimeMillis() - CurrentTime)+"毫秒");
    }
}
~~~

### 3.2 arraycopy方法
该方法将数组指定数据拷贝到另一个数组中，数组的拷贝动作是系统级的，性能很高。System.arraycopy方法具有5个参数，含义分别为：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/23/arratcopy%E5%8F%82%E6%95%B0%E5%90%AB%E4%B9%89-1556031999370.png)

### 小练习2
将src数组中前3个元素，复制到dest数组的前3个位置上。
复制元素前：src[1,2,3,4,5]，dest[6,7,8,9,10]
复制元素后：src[1,2,3,4,5], dest[1,2,3,9,10]
```java
arraycopy(源数组，源数组索引起始位置，目标数组，目标数组索引起始位置，复制元素个数)
 */
public class Demo09SystemArrayCopy {
    public static void main(String[] args) {
        int[] src = {1,2,3,4,5};
        int[] dest = {6,7,8,9,10};
        System.arraycopy(src,0,dest,0,3);
        for (int i = 0; i < dest.length; i++) {
            System.out.print(dest[i]+","); // 123910
        }
    }

}
```

## 4. StringBuilder类
### 4.1 字符串拼接问题
由于String类的对象内容不可变，所以每当进行字符串拼接时，都会在内存中创建一新的对象。
字符串是常量，它们的值在创建后不能被更改。如果字符串进行拼接操作，每次拼接时都会创建一个新的String对象，耗时又浪费空间。可以使用`java.lang.StringBuilder`类。

### 4.2 StringBuilder类概述
StringBuilder又称为可变字符序列,它是一个类似于 String 的字符串缓冲区，通过某些方法调用可以改变该序列的长度和内容。即它是一个容器，容器中可以装很多字符串。并且能够对其中的字符串进行各种操作。
它的内部拥有一个数组用来存放字符串内容，进行字符串拼接时，直接在数组中加入新内容。StringBuilder会自动维护数组的扩容。原理如下图所示：(默认16字符空间，超过自动扩充)
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/23/01_StringBuilder%E7%9A%84%E5%8E%9F%E7%90%86-1556033213070.bmp)

### 4.3 构造方法
常用的构造方法有2个：
- `public StringBuilder()`:构造一个空的StringBuilder容器。
- `public StringBuilder(String str)`:构造一个容器同时将一个字符串添加进去。
```java
public class Demo10StringBuilder {
    public static void main(String[] args) {
        StringBuilder strb = new StringBuilder(); // 创建一个空容器对象
        StringBuilder strb2 = new StringBuilder("Hello"); // 带参构造方法
    }
}
```

### 4.4 常用方法
StringBuilder中常用的方法有2个：
- `public StringBuilder append(...)`：添加任意类型数据的字符串形式，并返回当前对象自身。
- `public String toString()`：将当前StringBuilder对象转换为String对象。

#### append方法
append方法具有多种重载形式，可以接收任意类型的参数。任何数据作为参数都会将对应的字符串内容添加到StringBuilder中。示例如下：
```java
public class Demo10StringBuilder {
    public static void main(String[] args) {
        StringBuilder builder = new StringBuilder(); // 创建一个空容器对象
        StringBuilder result = builder.append("hello"); //向容器中添加一个字符串类型
        System.out.println(builder); // hello
        System.out.println(result); // hello
        if(builder == result){
            System.out.println("它们是同一个容器对象");
        }
        // 可以添加任意类型,且可以往容器中连续放入
        builder.append(100).append(true).append(10.0).append('A');
        System.out.println(builder); // hello100true10.0A
	/*
	如果在开发中，遇到调用一个对象后，返回一个对象的情况，可以使用返回的对象继续调用方法。
	如上情况，可以把它们连在一起，这叫做链式编程。
	*/
    }
}
```

#### toString方法
通过toString方法，StringBuilder对象将会转换为不可变的String对象。如下：
~~~java
public class Demo11StringToString {
    public static void main(String[] args) {
        StringBuilder builder = new StringBuilder("hello").append("world");
        String str = builder.toString(); // 将builder容器转换为字符串类型。
        System.out.println(str);
        

    }
}
~~~

## 5. 包装类
Java提供了两个类型系统，基本类型与引用类型。使用基本类型在于效率，然而很多情况，会创建对象使用，因为对象可以做更多的功能，如果想要我们的基本类型像对象一样操作，就可以使用基本类型对应的包装类，如下：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/23/%E5%9F%BA%E6%9C%AC%E7%B1%BB%E5%9E%8B%E5%AF%B9%E5%BA%94%E5%8C%85%E8%A3%85%E7%B1%BB-1556034717119.png)

### 5.1 装箱与拆箱
基本类型与对应的包装类对象之间，来回转换的过程称为"装箱" 与 "拆箱"：
* **装箱**:从基本类型转换为对应的包装类对象。
* **拆箱**:从包装类对象转换为对应的基本类型。

用Integer与int为例：
```java
public class Demo12PackClass {
    public static void main(String[] args) {
        // 基本数值 --> 包装对象
        Integer i = new Integer(4); // 使用构造方法
        Integer iii = Integer.valueOf(4); // /使用包装类中的valueOf方法

        // 包装对象 --> 基本数值
        int num = i.intValue();
    }
}
```

### 5.2 自动装箱与自动拆箱
由于我们经常要做基本类型与包装类之间的转换，从Java 5开始，基本类型与包装类的装箱、拆箱动作可以自动完成。示例如下：
~~~java
Integer i = 4;//自动装箱。相当于Integer i = Integer.valueOf(4);
i = i + 5;//等号右边：将i对象转成基本数值(自动拆箱) i.intValue() + 5;
//加法运算完成后，再次装箱，把基本数值转成对象。
~~~

### 5.3 基本类型与字符串之间的转换
#### 基本类型转换为String
基本类型转换String总共有三种方法，这里只讲解最简单一种方式：基本类型直接与""连接即可，如34+""。

#### String转换成对应的基本类型

除了Character类之外，其他所有包装类都具有parseXxx静态方法可以将字符串参数转换为对应的基本类型：

- `public static byte parseByte(String s)`：将字符串参数转换为对应的byte基本类型。
- `public static short parseShort(String s)`：将字符串参数转换为对应的short基本类型。
- `public static int parseInt(String s)`：将字符串参数转换为对应的int基本类型。
- `public static long parseLong(String s)`：将字符串参数转换为对应的long基本类型。
- `public static float parseFloat(String s)`：将字符串参数转换为对应的float基本类型。
- `public static double parseDouble(String s)`：将字符串参数转换为对应的double基本类型。
- `public static boolean parseBoolean(String s)`：将字符串参数转换为对应的boolean基本类型。

> 如果字符串参数的内容无法正确转换为对应的基本类型，则会抛出`java.lang.NumberFormatException`异常。


