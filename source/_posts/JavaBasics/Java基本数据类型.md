---
title: Java基本数据类型
date: 2019-4-11 16:00
categories: JavaBasics # 分类
tags: [Java]
description: 该文章介绍了Java基本数据类型,引用数据类型,以及类型转换。
---

## 前言
变量就是申请内存来存储值.也就是说,当创建变量时,需要在内存中申请空间.<br>内存管理系统根据变量的类型为变量分配存储空间,分配的空间只能用来存储该类型数据.
![title](https://i.loli.net/2019/04/11/5cae9c0f20f05.jpg)<br>
因此,通过定义不同类型的变量,可以在内存中存储整数,小数或者字符<br>Java的两大数据类型:
- 内置数据类型
- 引用数据类型

<!--more-->

### 内置数据类型

Java语言提供了四类八种基本类型。整数型,浮点型, 字符类型,布尔型。

##### byte
- byte数据类型是8位,有符号的,以二进制补码表示的整数;
- 最小值是-128(-2^7), 最大值是127(2^7-1), 默认值为0;
- byte类型用在大型数组中节约空间,主要代替整数,因为byte变量占用的空间只有int类型的四分之一;
- 例子:byte a = 100, byte b = -50;

##### short
- short数据类型是16位,有符号的以二进制补码表示的整数;
- 最小值是-32768(-2^15),最大值是32767(2^15-1), 默认值为0;
- short数据类型也可以像byte那样节省空间. 一个short变量是int型变量的二分之一;
- 例子:short s = 1000, short r = -20000
##### int
- int数据类型是32位,有符号的以二进制补码表示的整数;
- 最小值是-2 147 483 648(-2^31),最大值是2147483647(2^31-1),默认值为0;
- 一般地整型变量默认为int类型;
- 例子:int a = 10000, int b = - 200000
##### long
- long数据类型是64位,有符号的二进制补码表示的整数;
- 最小值是-9 223 372 036 854 775 808(-2^63);
- 最大值是9 223 372 036 854 775 807(2^63-1);
- 这种类型主要使用在需要比较大整数的系统上;
- 默认值是0L;
- 例子:long a = 1000000L, Long b = -2000000L
##### float
- float数据类型是单精度,32位,符合IEEE754标准的浮点数;
- float在存储大型浮点数值时可以节省内存空间;
- 默认值是0.0f;
- 浮点数不能用来表示精确的值,如货币;
- 例子:float f1 = 234.5f
##### double
- double数据类型是双精度,64位,符合IEEE754标准的浮点数;
- 浮点数的默认类型为double类型;
- double类型同样不能表示精确的值;
- 默认值为0.0d;
- 例子: double d1 = 123.4
##### boolean
- boolean数据类型表示一位的信息;
- 只有两个取值:true和false;
- 这种类型只作为一种标志来记录true/false情况;
- 默认值是false;
- 例子:boolean one = true
##### char
- char类型是一个单一的16位Unicode字符;
- 最小值是\u0000(即为0),最大值是\uffff(即为65535);
- char数据类型可以存储任何字符;
- 例子:char letter = 'A'
![title](https://i.loli.net/2019/04/13/5cb143c1690a8.png)

#### 变量定义的格式包括三个要素：数据类型、变量名、数据值
```java
//数据类型 变量名 = 数据值;
//例子：
byte b = 100；
short s = 1000;
int i = 5;
long l = 10000000000L;
float f = 5.5F;
double d = 8.5;
boolean bool = true;
char c = 'A';
```
#### 实例
```java
public class PrimitiveType {
	public static void main(String[] args) {
		//byte
		System.out.println("基本类型:byte 二进制位数:" + Byte.SIZE);
		System.out.println("包装类:java.lang.Byte");
		System.out.println("最小值:Byte.MIN_VALUE=" + Byte.MIN_VALUE);
		System.out.println("最大值:Byte.MAX_VALUE=" + Byte.MAX_VALUE);
		System.out.println();
		
		//short
		System.out.println("基本类型: short 二进制位数:" + Short.SIZE);
		System.out.println("包装类:java.lang.Short");
		System.out.println("最小值:Short.MIN_VALUE=" + Short.MIN_VALUE);
		System.out.println("最大值:Short.MAX_VALUE=" + Short.MAX_VALUE);
		System.out.println();
		
		//int
		System.out.println("基本类型: int 二级制位数:" + Integer.SIZE);
		System.out.println("包装类:java.lang.Integer");
		System.out.println("最小值:Integer.MIN_VALUE=" + Integer.MIN_VALUE);
		System.out.println("最大值:Integer.MAX_VALUE=" + Integer.MAX_VALUE);
		System.out.println();
		
		//long
		System.out.println("基本类型: long 二进制位数:" + Long.SIZE);
		System.out.println("包装类:java.lang.Long");
		System.out.println("最小值:Long.MIN_VALUE" + Long.MIN_VALUE);
		System.out.println("最大值:Long.MAX_VALUE" + Long.MAX_VALUE);
		System.out.println();
		
		//float
		System.out.println("基本类型: float 二进制位数:" + Float.SIZE);
		System.out.println("包装类:java.lang.Float");
		System.out.println("最小值:Float.MIN_VALUE" + Float.MIN_VALUE);
		System.out.println("最大值:Float.MAX_VALUE" + Float.MAX_VALUE);
		System.out.println();
		
		//double
		System.out.println("基本类型: double 二进制位数:" + Double.SIZE);
		System.out.println("包装类:java.lang.Double");
		System.out.println("最小值:Double.MIN_VALUE" + Double.MIN_VALUE);
		System.out.println("最大值:Double.MAX_VALUE" + Double.MAX_VALUE);
		System.out.println();
		
		//char
		System.out.println("基本类型: char 二进制位数:" + Character.SIZE);
		System.out.println("包装类: java.lang.Character");
		// 以数值形式而不是字符形式将Character.MIN_VALUE输出至控制台
		System.out.println("最小值:Character.MIN_VALUE=" + (int) Character.MIN_VALUE);
		System.out.println("最大值:Character.MAX_VALUE=" + (int) Character.MAX_VALUE);
	}
}
//学习类之后将该程序改进一下:输入任意类型就打印该类型的信息
```
### 引用类型
- 在Java中,引用类型的变量非常类似于C/C++的指针.引用类型指向一个对象,指向对象的变量是引用变量.这些变量在声明时被指定为一个特定的类型. 比如Employee,Puppy等. 变量一旦声明,类型就不能被改变了.
- 对象,数组都是引用数据类型
- 所以引用类型的默认值都是null
- 一个引用变量可以用来引用任何与之兼容的类型
- 例子: Site site = new Site('Java')
### Java常量
常量是指在Java程序中固定不变的数据。以下是常量的分类：
1. 字符串常量：凡是用双引号引起来的部分，叫做字符串常量。例如："abc", "Hello","123"
2. 字符常量：凡是用单引号引起来的单个字符，就是字符常量。例如：'A','b','9','中'
3. 整数常量：直接写上的数字，没有小数点。例如：100,123,200,0，-1
4. 浮点数常量：直接写上数字，有小数点的。例如：2.5，-3.14,0.0
5. 布尔常量：只有量中取值。true，false。
6. 空常量：null。代表没有任何数据。
在Java中使用final关键字来修饰常量,声明方式和变量类似:
```java
final double PI = 3.1415927;
```
虽然常量名也可以用小写,但为了方便识别,通常使用大写字母表示常量.<br>字面量可以赋给任何内置类型的变量.例如:
```java
byte a = 68;
char a = 'A';
```
byte,int,long和short都可以用十进制,16进制以及8进制的方式来表示.<br>当使用常量时,前缀0表示8进制,而前缀0x代办16进制,例如:
```java
int decimal = 100;
int octal = 0144;
int hexa = 0x64;
```
和其他语言一样,Java的字符串常量也是包含在两个引号之间的字符序列,例子:
```java
"Hello World"
"two\nlines"
"\"This is in quotes\""
```
字符串常量和字符常量都可以包含任何Unicode字符,例如:
```java
char a = "\u0001";
String a = "\u0001";
```
Java支持的转义字符序列

### 自动类型转换（显式）
整型,实型(常量),字符型数据可以混合运算.运算中,不同类型的数据先转化为同一类型,然后进行运算.<br>转换从低级到高级
```java
低------------------------------------->高
byte,short,char--> int--> long--> float--> double
```
数据类型转换必须满足如下规则:
1. 不能对boolean类型进行类型转换
2. 不能把对象类型转换成不相关类的对象
3. 在把容量大的类型转换为容量小的类型时必须使用强制类型转换
4. 转换过程中可能导致溢出或损失精度,例如:
```java
int i = 128;
byte b = (byte)i;
```
因为byte类型是8位,最大值为127,所以当int强制转换为byte类型时,值128会导致溢出
<br>5. 浮点数到整数的转换是通过舍弃小数得到,而不是四舍五入,例如:
```java
(int)23.7 == 23;
(int)-45.89f == -45;
```
#### 自动类型转换(隐式)
必须满足转换前的数据类型的位数要低于转换后的数据类型,例如 short数据类型的位数为16位,就可以自动转换位数为32的int类型,同样float数据类型的位数为32位,可以自动转换为64位的double类型.<br>
- 特点：代码不需要进行特殊处理，自动完成
- 规则：数据范围从小到大。

```java
public class AutoSwitch{
    public static viod main(String[] args){
	char c1 = 'a'; // 定义一个char类型
	int i1 = c1; //char自动类型转换为int
	System.out.println("char自动类型转换为int后的值=" + i1);
	char c2 = 'A'; //定义一个char类型
	int i2 = c2 + 1; //char类型金额int类型计算
	System.out.println("char类型和int计算后的值为 = " + i2);
    }
}
//运行结果为:
//char自动类型转换为int后的值=97
//char类型和int计算后的值为 = 66
//解析:c1的值为字符a,查ASCII码表的对应的int类型值为97,A对应值为65,故i2=65+1=66
```
#### 强制类型转换(显式)
将取值范围大的类型强制转换成取值范围小的类型
- 转换格式：数据类型 变量名 = （数据类型）被强转的数据值；

```java
//double类型强制转换为int类型，直接去掉小数点
int i = (int)1.5;
public class QiangZhiZhuanHuan{
    public static viod main(String[] args){
	int i1 = 123; //
	byte b = (byte)i1; //int类型强制类型转换为byte
	System.out.println("int强制类型转换为byte后的值为:" + b);
    }
}
//运行结果:
int强制类型转换为byte后的值等于123
```
Tips：
1. 浮点转成整数，直接取消小数点，可能会造成数据损失精度。
2. int强制转成short砍掉2个字节，可能会造成数据溢出，导致数据丢失。  
3. byte/short/char这三种类型都可以发生数学运算，例如加法“+”。
4. byte/short/char这三种类型在运算时，都会首先自动被提升成为int类型，然后再计算。
5. boolean类型不能发生任何数据类型转换
#### 强制类型转换(隐式)
1. 整数的默认类型是int
2. 浮点型不存在这种情况,因为在定义float类型时必须在数字后面跟上F或者f
对于byte/short/char三种类型来说，如果右侧赋值的数值没有超过范围，那么javac编译器将会自动隐式地为我们补上一个(byte)(short)(char)。
- 如果没有超过左侧的范围，编译器补上强转。
- 如果右侧超过了左侧的范围，编译报错。
```java
public class Notice{
    public static void main(String[] args){
	//右侧确实是一个int数字，但没有超过左侧的范围，故自动隐式强转不会报错。
	//int --> byte 高位到低位，不是自动类型转换
	byte num1 = 30; // 右侧没有超过左侧范围
	System.out.println(num1);
	
	//byte num2 = 128; // 右侧超过了左侧范围

	// int --> char 只要右侧没超出范围，编译器会自动补上一个隐含的(char)，进行自动隐式强转。
	char strings = /*(char)*/ 65；
	System.out.println(strings)
	
    }
}
```

#### 编译器的常量优化
在给变量进行赋值时，如果右侧表达式当中全部是常量，没有任何变量，那么编译器javac将会直接将若干个常量表达式计算得到结果。<br>short result = 5+8；//等号右边全部是常量，没有任何变量参与运算，编译之后，得到的.class字节码文件当中相当于==直接就是==：short result = 13;<br>右侧的常量结果数值，没有超过左侧范围，所以正确。<br>这称为“编译器的常量优化”

但是注意：一旦表达式当中有变量参与，那么就不能进行这种优化了。
### ASCII编码表
ASCII码表：American Standard Code for Information Interchange，美国信息交换标准码。
Unicode码表：万国码。也是数学和符号的对照关系，开头0-127部分和ASCII完全一样，但从128开始包含更多的字符。
![title](https://i.loli.net/2019/04/13/5cb190886bf51.jpg)
只需特殊记住三个字符，分别是
- '0'=48
- 'A'=65
- 'a'=97