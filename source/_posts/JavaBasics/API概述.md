---
title: API概述
date: 2019-4-18 22:00
categories: JavaBasics # 分类
tags: [Java]
description: Java当中API概述及基本使用
urlname: api
---

## 1. 概述
API(Application Programming Interface)，应用程序编程接口。Java API是一本API字典，是JDK中提供给我们使用类的说明文档。这些类将底层的代码实现封装了起来，我们不用关心类是如何实现的，只需学习这些类如何使用即可。

<!--more-->


### API使用步骤
1. 打开帮助文档。
2. 点击显示，找到索引，看到输入框。
3. 输入你要查询的类名。
4. 看包。java.lang下的类不需要导包，其他都要import
5. 看类的解释和说明。
6. 学习构造方法。
7. 使用成员方法。

## 2. Scanner类
### 2.1 什么是Scanner类？
一个可以解析基本类型和字符串的简单文本扫描器。如下代码能够使用户从System.in中获取数据：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/18/Scanner-1555573837528.png)<br>
System.in系统输入指的是通过键盘录入数据。
### 2.2 引用类型使用步骤

#### 导包
使用import关键字导包，在类的所有代码之前导包，引入要使用的类型，java.lang包下的所有类无需导入。格式：import 包名.类名;
#### 创建对象
使用该类的构造方法，创建一个该类的对象。格式：数据类型 变量名 = new 数据类型(参数列表);
#### 调用方法
调用该类的成员方法，完成指定功能。格式：变量名.方法名();
### 2.3 Scanner使用步骤
- java.util.Scanner:该类需要import导入后使用。
- public Scanner(InputStream source):构造一个新的Scanner，它生成的值是从指定的输入流扫描的。
- public int nextInt():将输入信息的下一个标记扫描为一个int值。

举例：
```java
import java.util.Scanner;

public class Demo03ScannerMax {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int Max = 0;
        for(int i = 1;i < 4;i++){
            System.out.println("请输入第" + i + "个数字：");
            int temp = sc.nextInt();
            if(temp > Max){
                Max = temp;
            }
        }
        System.out.println("最大数是：" + Max);

    }
}
```
### 2.4 匿名对象
匿名对象就是创建对象时，只有创建对象的语句，却没有把对象地址赋值给某个变量。没有变量名的对象就是匿名对象。
- 格式：new 类名(参数列表);
#### 应用场景
1. 创建匿名对象直接调用方法，没有变量名。举例：new Scanner(System.in).nextInt();
2. 一旦调用两次方法，就是创建了两个对象，语句执行完毕即对象销毁。
3. 匿名对象可以作为方法的参数和返回值。
#### 作为参数：
```java
class Test {
    public static void main(String[] args) {
        // 普通方式
        Scanner sc = new Scanner(System.in);
	input(sc); 
	//匿名对象作为方法接收的参数
	input(new Scanner(System.in));
    }
    public static void input(Scanner sc){
	System.out.println(sc);
    }
}
```
#### 作为返回值：
```java
public static Scanner getScanner(){
        return new Scanner(System.in);
    }
```

## 3. Random类
### 3.1 什么是Random类
此类的实例用于生产伪随机数。如下代码使用户能够得到一个随机数：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/18/Random-1555577504861.png)
### 3.2 Random使用三步骤
1. 导包：import java.util.Random;
2. 创建:Random r = new Random();
3. 使用:获取一个随机的int数字(范围是int所有范围，有正负两种),int num = r.nextInt();

举例如下：
```java
public class Demo05Random {
    public static void main(String[] args) {
        int a = 0;
        while(a < 100){
            System.out.println("随机数是：" + new Random().nextInt());
            a++;
        }

    }
}
```
获取一个随机的int数字(参数代表了范围，左闭右开区间):int num = r.nextInt(3)实际含义是：[0,3),也就是0~2。
#### 猜数字小游戏
```java
import java.util.Random;
import java.util.Scanner;

public class Demo06guessNumber {
    public static void main(String[] args) {
        int number = new Random().nextInt(100) + 1;
        int count = 0;
        while (true){
            System.out.println("请输入你要猜的数字(1-100)：");
            int guessNumber = new Scanner(System.in).nextInt();
            if(guessNumber > number){
                System.out.println("You enter number is big , again! ");
            }else if(guessNumber < number){
                System.out.println("You enter number is small , again!");
            }else{
                System.out.println("Bin1go！你使用了" + count + "次。");
                break;
            }
            count++;
        }
    }
}
```
## 3. ArrayList类
### 3.1 什么是ArrayList类？
java.util.ArrayList是大小可变的数组的实现，存储在内的数据称为元素。此类提供一些方法来操作内部存储的元素。Array中可不断添加元素，其大小也自动增长。
### 3.2 ArrayList使用步骤
- java.util.ArrayList<E>:该类需要import导入后使用。<br>
<E>,表示一种指定的数据类型，叫做泛型。E，取自Element(元素)的首字母。在出现E的地方，我们使用一种引用数据类型将其替换即可，表示我们将存储哪种引用类型的元素。
- 基本格式：ArrayList<String> list = new ArrayList<String>();JDK7以后，右边泛型的尖括号可以留空，但<>仍要写。ArrayList<String> list = new ArrayList<>();
### 3.3 常用方法和遍历
对于元素的操作，基本体现在增、删、查。常用方法如下：
- public boolean add(E e);向集合当中添加元素，参数的类型和泛型一致。返回值代表添加是否成功。
- public E get(int index);从集合当中获取元素，参数是索引编号，返回值就是对应位置的元素。
- public E remove(int index);从集合中删除元素，参数是索引，返回值就是被删除的元素。
- public int size();获取集合的尺寸长度，返回值是集合中包含的元素个数。

### 3.4 存储基本数据类型
ArrayList对象不能存储基本类型，只能存储引用类型的数据。类似<int>不能写，但是存储基本数据类型对应的包装类型是可以的。所以，想要存储基本类型数据，<>中的数据类型，必须转换后才能编写。转换写法如下：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/18/%E6%95%B0%E7%BB%84%E9%9B%86%E5%90%88%E5%A6%82%E4%BD%95%E5%AD%98%E5%82%A8%E5%8C%85%E8%A3%85%E7%B1%BB-1555590027103.png)<br>
可以发现，只有Integer和Character需要特殊记忆，其他基本类型只是首字母大写即可。那么存储基本类型数据，代码如下：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/18/Integer%E4%BD%BF%E7%94%A8%E8%B7%9D%E7%A6%BB-1555590228509.png)<br>
#### 自定义格式打印集合内容
```java
import java.util.ArrayList;

public class DemoPrintArrayList {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("盗贼");
        list.add("法师");
        list.add("术士");
        list.add("战士");
        print(list);
    }
    public static void print(ArrayList<String> list){
        System.out.print("{");
        for (int i = 0;i < list.size();i++){
            if (i != list.size()-1){
                System.out.print(list.get(i)+ "@");
            }else {
                System.out.print( list.get(i)+ "}");
            }
        }
    }
}
```
#### 定义获取偶数元素集合方法
```java
import java.util.ArrayList;
import java.util.Random;

/*
定义获取所有偶数元素集合的方法(ArrayList类型作为返回值)
 */

public class DemoGetEvenNumber {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>();
        for(int i = 0;i < 20;i++){
            list.add(new Random().nextInt(1000)+1);
        }
        System.out.println(list);
        System.out.print(getEvenNumber(list));
    }

    public static ArrayList<Integer> getEvenNumber(ArrayList<Integer> list){
        ArrayList<Integer> Even = new ArrayList<>();
        for(int i = 0;i < list.size();i++){
            if(list.get(i) %2 == 0){
                Even.add(list.get(i));
            }
        }
        return Even;
    }

}
```