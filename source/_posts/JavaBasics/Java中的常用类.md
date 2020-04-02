---
title: Java中的常用工具类
date: 2019-4-19 23:00
categories: JavaBasics # 分类
tags: [Java]
description: 简单介绍了几个Java中常用的工具类
urlname: tool-classs
---

## 1. String类概述
java.lang.String 类代表字符串。Java程序中所有的字符串文字都可以被看作是实现此类的实例。<br>
类String中包括用于检查各个字符串的方法，比如用于比较字符串、搜索字符串、提取子字符串以及创建具有翻译为大写或者小写的所有字符的字符串副本。

<!--more-->

### 1.1 字符串的特点
1. 字符串的内容永不可变。
2. 因为不可变，所以字符串是可以共享的
3. 字符串效果上相当于是char[]字符数组，但底层原理是byte[]字节数组。

### 1.2 创建字符串的常见3+1种方式
#### 三种构造方法
1. public String(): 创建一个空白字符串，不含有任何内容。
2. public String(char[] array)：根据字符数组的内容，来创建对应的字符串。
3. public String(byte[] array): 根据字节数组的内容，来创建对应的字符串。
#### 一种直接创建：
`String str = "Hello";`

```java
public class Demo02StringTwo {
    public static void main(String[] args) {
        String str1 = "abc";
        String str2 = "abc";
        char[] charArray = {'a','b','c'};
        String str3 = new String(charArray);

    }
}
```
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/19/%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%86%85%E5%AD%98%E5%9B%BE-1555640143003.png)

### 1.3 常用方法
#### 判断功能的方法
- public boolean equals(Object Obj):参数可以是任何对象，只有参数是一个字符串且内容相同时返回true，反之false。
```java
public class Demo03StringEquals {
    public static void main(String[] args) {
        String str1 = "Hello";
        String str2 = "Hello";
        char[] charArray = {'H','e','l','l','o'};
        String str3 = new String(charArray);

        System.out.println(str1.equals(str2));
        System.out.println(str2.equals(str3));
        System.out.println(str3.equals("Hello"));
        System.out.println("Hello".equals(str3));

        String str4 = null;
        System.out.println("abc".equals(str4)); // false 推荐写法
	    System.out.println(str4.equals("abc")); // 报错，NullPointerException
    }
}
注意事项：
1. 任何对象都能用Object进行接收。
2. equals方法具有对称性，也就是a.equals(b)和b.equals(a)效果是一样的。
3. 如果比较双方一个常量一个变量，推荐把常量字符串写在前面。<br>推荐："abc".equals(str),不推荐:str.equals("abc")。
```
- public boolean equalsIgnoreCase(String anotherString):将此字符串与指定对象进行比较，忽略大小写。
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/19/equalsIgnoreCase-1555643567275.png)

#### String当中与获取相关的常用方法
- public int lenght(); 获取字符串当中含有的字符个数，拿到字符串长度。
- public String concat(String str); 将指定的字符串连接到该字符串的末尾。
- public char charAt(int index); 获取指定索引位置的单个字符。
- public int indexOf(String str); 查找参数字符串在本字符串当中首次出现的索引位置，如果没有返回-1值。


举例代码如下：
```java
public class Demo04StringGet {
    public static void main(String[] args) {
        int length = "abcdefghijklmnobqrstuvwxyz".length();
        System.out.println("字符串长度为：" + length);

        //拼接字符串
        String str1 = "Hello";
        String str2 = "World";
        String str3 = str1.concat(str2);
        System.out.println(str1); // Hello
        System.out.println(str2); // World
        System.out.println(str3); // HelloWorld
        System.out.println("Hello".charAt(0)); // H
        String str4 = str3;
        System.out.println(str4.indexOf("llo")); //返回索引位置，即2
    }
}
```
- public String substring(int index):截取从参数位置一直到字符串末尾，返回新字符串
- public String substring(int begin,int end):截取从begin开始，一直到end结束，中间的字符串。[begin,end)，包含左边，不包含右边。


举例代码如下：
```java
public class Demo05SubString {
    public static void main(String[] args) {
        String str1 = "HelloWorld";
        System.out.println(str1.substring(5)); // 返回World
        System.out.println(str1.substring(4,7)); // owo , 4~6不含7
    }
}
```
#### String当中与转换相关的常用方法
- public char[] toCharArray(); 将当前字符串拆分成为字符数组作为返回值。
- public byte[] getBytes(); 获取当前字符串底层的字节数组。
- public String replace(CharSequence oldString, CharSequence newString);将所有出现的老字符串替换为新的字符串，返回替换后的结果新字符串。


例子：
```java
public class Demo06StringConvert {
    public static void main(String[] args) {
        char[] chars = "Hello".toCharArray();
        System.out.println(chars); // Hello
        System.out.println(chars[0]); // H
        System.out.println(chars.length); // 5
        System.out.println("=========================");
        String str = "abc";
        byte[] bytes = str.getBytes();
        System.out.println(bytes); //指向内存地址中abc对应的byte值
        System.out.println(bytes[0]); // [0] = 97,[1] = 98,[2] = 99
        String str2 = "你怎么回事小老弟，会不会玩啊，fuck";
        System.out.println(str2.replace("fuck","儒雅随和"));

    }
}
Tips：CharSequence意思是说可以接受字符串类型。是一个接口，也是一种引用类型。作为参数类型，可以把String对象传递到方法中。
```
#### String中分割字符串的方法
public String[] split(String regex); 按照参数的规则，将字符串切分为若干部分。
```java
public class Demo07StringSplit {
    public static void main(String[] args) {
        String str1 = "a,b,c";
        String[] array1 = str1.split(",");
        for(int i = 0;i<array1.length;i++){
            System.out.println(array1[i]);
        }
        String str2 = "a b c";
        String[] array2 = str2.split(" ");
        for(int i=0;i<array2.length;i++){
            System.out.println(array2[i]);
        }
        String str3 = "XXX.YYY.ZZZ";
        String[] array3 = str3.split("\\."); //注意转义
        for (int i=0;i<array3.length;i++){
            System.out.print(array3[i]);
        }
    }
}
```
#### 小练习
拼接字符串练习，输入数组{1,2,3}，返回[1#2#3]。
```
public class Demo08StringExercise {
    public static void main(String[] args) {
        int[] arr = {1,2,3,4};
        System.out.println(arrayToString(arr));

    }
    public static String arrayToString(int[] arr){
        String str = "[";
        for (int i=0;i<arr.length;i++){
            if(i != arr.length-1){
                str += (arr[i]+"#");
            }else{
                str += (arr[i]+"]");
            }
        }
        return str;
    }
}
```
统计字符个数：键盘录入一个字符，统计字符串中大小写字母及数字字符个数
```java
import java.util.Scanner;

/*
键盘输入一个字符串，统计其中各种字符出现次数，返回次数。
字符种类：大写字母，小写字母，数字，其他

 */
public class Demo09StringExercise02 {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入字符串：");
        String input = sc.next(); // 获取键盘输入的字符串
        System.out.println(stringCount(input));
    }
    public static String stringCount(String input){
        int countUpper = 0;
        int countLower = 0;
        int countNumber = 0;
        int countOther = 0;
        char[] chars = input.toCharArray();
        for(int i = 0;i < chars.length;i++){
            char ch = chars[i]; // 单个字符，底层表示为byte[]数值
            if(ch >= 'A' && ch <= 'Z'){
                countUpper++;
            }else if (ch >= 'a' && ch <= 'z'){
                countLower++;
            }else if (ch >= '0' && ch <= '9'){
                countNumber++;
            }else{
                countOther++;
            }
        }
        return "大写字母："+countUpper+"\n小写字母："+countLower+"\n数字:"+countNumber+"\n其他类型："+countOther;
    }
}
```
## 2. static关键字
### 2.1 概述
关于static关键字的使用，它可以用来修饰的成员变量和成员方法，被修饰的成员是属于类的，而不是单单是属于某个对象的。也就是说，既然属于类，就不可以靠创建对象来调用了。
### 2.2定义和使用格式
当static修饰成员变量时，该变量称为类变量。该类的每个对象都共享同一个类变量的值。任何对象都可以更改该类变量的值，但也可以在不创建该类对象的情况下对类变量进行操作。
#### 类变量
- 类变量：使用static关键字修饰的成员变量。格式：static 数据类型 变量名。

实例：
```java
//定义学生类
public class Student {
    private int sid; //学号
    private String name;
    private int age;
    static String room;
    public static int idCount = 0; //自动分配学号

    public Student() {
        this.sid = ++idCount;
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
        this.sid = ++idCount;
    }
	
	......省略get/set方法
}


//实例使用
public class Demo10Class {
    public static void main(String[] args) {
        Student stu = new Student("法师",11);
        stu.room = "101号教室";
        System.out.println("姓名："+stu.getName()+",年龄:"+stu.getAge()+"岁,教室："+stu.getRoom()+",学号："+stu.getSid());
        Student stu2 = new Student("盗贼",15);
        System.out.println("姓名："+stu2.getName()+",年龄:"+stu2.getAge()+"岁,教室："+stu2.getRoom()+",学号："+stu2.getSid());
    }
}
```
#### 静态方法
当static修饰成员方法时，该方法称为类方法。静态方法在声明中有static，建议使用类名来调用，而不需要创建类的对象。
- 类方法：使用static关键字修饰的成员方法，一般称为静态方法。<br>格式如下：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/19/%E9%9D%99%E6%80%81%E6%96%B9%E6%B3%95-1555663220471.png)


实例：
```java
//MyClass
public class MyClass {
    int num; // 成员变量
    static int numStatic; //静态变量
    public void method(){
        System.out.println("这是一个成员方法！");
        System.out.println(num); //成员方法可以访问成员变量
        System.out.println(numStatic); // 成员方法可以访问静态变量
    }
    public static void methodStatic(){
        System.out.print("这是一个静态方法！");
        System.out.println(numStatic); //静态方法可以访问静态变量
        //System.out.println(num); //静态方法无法访问成员变量
    }
}
//实例类
public class Demo01StaticMethod {
    public static void main(String[] args) {
        MyClass obj = new MyClass(); // 首先创建对象
        obj.method(); // 才能使用没有static关键字的内容

        //对于静态方法来说，可以通过对象名进行调用，也可以直接通过类名称来调用。
        obj.methodStatic(); // 不推荐写法，编译器编译时会自动优化为下面方法。
        MyClass.methodStatic(); // 推荐写法，类名称.静态方法。

        // 对于本类当中的静态方法，可以省略名称
        myMethod();
        Demo01StaticMethod.myMethod(); // 完全等效于上面
    }
    public static void myMethod(){
        System.out.println("自己的方法!");
    }
}
```
- 静态方法调用总结：
    - 静态方法可以直接访问类变量和静态方法。
    - 静态方法不能直接访问普通成员变量或成员方法。原因：因为在内存中是[先]有的静态内容，[后]有的非静态内容,"前人不知后事，后人尽知前史"。
    - 静态方法中，不能使用this关键字。
    - 无论是成员变量，还是成员方法。如果有了static，都推荐使用类名称进行调用。静态变量：类名称.静态变量；静态方法：类名称.静态方法();

#### 2.3 静态原理图解
static修饰的内容：
- 是随着类的加载而加载的，且只加载一次。
- 存储于一块固定的内存区域(静态区),所以，可以直接被类名调用。
- 它优先于对象存在，所以，可以被所有对象共享。
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/19/static%E5%86%85%E5%AD%98%E5%9B%BE-1555683796541.png)

#### 静态代码块
- 静态代码块：定义在成员位置，使用static修饰的代码块{}。
    - 位置：类中方法外。
    - 执行：随着类的加载而执行且只执行一次，优先于main方法和构造方法的执行。
- 格式：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/19/%E9%9D%99%E6%80%81%E4%BB%A3%E7%A0%81%E5%9D%97-1555684497323.png)
- 作用：用来一次性地对静态成员变量进行赋值。

## 3. Arrays类
java.util.Arrays此类包含用来操作数组的各种方法，比如排序和搜索等。其所有方法均为静态方法，调用起来非常简单。
### 3.1 操作数组的方法
- public static String toString(int[] a): 返回指定数组内容的字符串表示形式。
- public static void sort(int[] a): 对指定的int型数组按数字升序进行排序。

举例：
```java
import java.util.Arrays;

public class Demo03Arrays {
    public static void main(String[] args) {
        int[] arr = {1,3,54,86,37,28};
        System.out.println(arr); // 打印数组内存地址
        String str = Arrays.toString(arr); //将指定数组内容变成字符串返回。
        System.out.println(str);
        int[] arr2 = {2,5,3,8,4,9,1};
        Arrays.sort(arr2); // 升序排序
        System.out.println(Arrays.toString(arr2)); // [1, 2, 3, 4, 5, 8, 9]
    }
}
```
#### 小练习
对一个随机字符串进行升序排序，然后倒序输出
```java
import java.util.Arrays;

public class Demo04ArraysExercise {
    public static void main(String[] args) {
        String str = "ehiaroaklcmbzhywqiem";
        char[] chars = str.toCharArray();
        Arrays.sort(chars); // 对字符串进行升序排序
        System.out.println(chars);
        for (int i = chars.length - 1; i >= 0; i--) { //快捷键:[chars.forr] , 对chars对象进行倒序遍历; [fori],正序遍历
            System.out.print(chars[i]);
        }
    }
}
```
## 4. Math类
java.lang.Math类包含用于执行基本数学运算的方法，如初等指数、对数、平方根和三角函数。类似这样的工具类，里面提供了大量的静态方法，完成与数学运算的操作。
### 4.1 基本运算的方法
- public static double abs(double num): 返回绝对值。
- public static double ceil(double num): 向上取整。
- public static double floor(double num): 向下取整。
- public static double round(double num): 四舍五入。


实例：
```java
public class Demo05Math {
    public static void main(String[] args) {
        System.out.println("以下示例abs用法");
        System.out.println(Math.abs(3.14)); // 3.14
        System.out.println(Math.abs(-2.5)); // 2.5
        System.out.println(Math.abs(0)); // 0
        System.out.println("以下示例ceil用法");
        System.out.println(Math.ceil(3.9)); // 4.0
        System.out.println(Math.ceil(3.1)); // 4.0
        System.out.println(Math.ceil(3.0)); // 3.0
        System.out.println("以下示例floor用法");
        System.out.println(Math.floor(30.9)); // 30.0
        System.out.println(Math.floor(30.1)); // 30.0
        System.out.println(Math.floor(31.0)); // 31.0
        System.out.println("以下示例round用法");
        System.out.println(Math.round(20.4)); //20
        System.out.println(Math.round(20.5)); //21
        System.out.println("圆周率：" + Math.PI);
    }
}
```
#### 小练习
请使用 Math 相关的API，计算在 -10.8 到 5.9 之间，绝对值大于 6 或者小于 2.1 的整数有多少个？
```java
public class Demo06mathExercise {
    public static void main(String[] args) {
        double min = -10.8;
        double max = 5.9;
        int num = 0;
        for (int i = (int)min; i < max; i++) {
            int abs = Math.abs(i); // 绝对值化
            if(abs > 6 || abs <2.1){
                num++;
            }
        }
        System.out.println(num);
    }
}
```

## 总结
### 字符串类及常用方法API
1. public String(): 创建一个空白字符串，不含有任何内容。
2. public String(char[] array)：根据字符数组的内容，来创建对应的字符串。
3. public String(byte[] array): 根据字节数组的内容，来创建对应的字符串
4. public boolean equals(Object Obj):参数可以是任何对象，只有参数是一个字符串且内容相同时返回true，反之false。
5. public boolean equalsIgnoreCase(String anotherString):将此字符串与指定对象进行比较，忽略大小写。
6. public int lenght(); 获取字符串当中含有的字符个数，拿到字符串长度。
7. public String concat(String str); 将指定的字符串连接到该字符串的末尾。
8. public char charAt(int index); 获取指定索引位置的单个字符。
9. public int indexOf(String str); 查找参数字符串在本字符串当中首次出现的索引位置，如果没有返回-1值。
10. public String substring(int index):截取从参数位置一直到字符串末尾，返回新字符串。
11. public String substring(int begin,int end):截取从begin开始，一直到end结束，中间的字符串。[begin,end)，包含左边，不包含右边。
12. public char[] toCharArray(); 将当前字符串拆分成为字符数组作为返回值。
13. public byte[] getBytes(); 获取当前字符串底层的字节数组。
14. public String replace(CharSequence oldString, CharSequence newString);将所有出现的老字符串替换为新的字符串，返回替换后的结果新字符串。
15. public String[] split(String regex); 按照参数的规则，将字符串切分为若干部分。

### static关键字及常用方法
1. 类变量：使用static关键字修饰的成员变量。格式：static 数据类型 变量名；
2. 类方法：使用static关键字修饰的成员方法，一般称为静态方法。

### Arrays类操作数组的常用方法
1. public static String toString(int[] a): 返回指定数组内容的字符串表示形式。
2. public static void sort(int[] a): 对指定的int型数组按数字升序进行排序。

### Math类常用方法
1. public static double abs(double num): 返回绝对值。
2. public static double ceil(double num): 向上取整。
3. public static double floor(double num): 向下取整。
4. public static double round(double num): 四舍五入。
