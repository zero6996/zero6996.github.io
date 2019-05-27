---
title: Java流程控制语句
date: 2019-4-14 14:00
categories: Java学习笔记 # 分类
tags: [Java,流程控制语句]
description: 记录了Java的所有流程控制语句及相关使用方法
---

## 流程控制
在一个程序执行中，各条语句的执行顺序对程序的结果是有直接影响的。所以必须清楚每条语句的执行流程，而且，我们需要通过控制语句的执行顺序来实现我们要完成的功能。

<!--more-->

### 1. if else 判断语句
#### 1.1 if语句第一种格式：if
```java
if(关系表达式){
    语句体；
}
```
- 执行流程
    - 首先判断关系表达式其结果是true还是false
    - 如果是true就执行语句体
    - 如果是false就不执行语句体
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/14/if-1555208361145.png)
#### 1.2 if语句第二种格式：if...else
```java
if(关系表达式){
    语句体1；
}else{
    语句体2；
}
```
- 执行流程
    - 首先判断关系表达式看其结果是true还是false
    - 如果是true就执行语句体1
    - 如果是false就执行语句体2
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/14/if_else-1555208575720.png)
```java
public static void main(String[] args){
    // 判断给定的数据是奇数还是偶数
    // 定义变量
    int a = 1;
    if(a % 2 == 0) {
        System.out.println("a是偶数");
    } else{
        System.out.println("a是奇数");
    }
    System.out.println("结束");
}
```
#### 1.3 if语句第三种格式：if...else if...else

```java
if (判断条件1) {
    执行语句1;
} else if (判断条件2) {
    执行语句2;
}
...
}else if (判断条件n) {
    执行语句n;
} else {
    执行语句n+1;
}
```
- 执行流程
    - 首先判断关系表达式1看其结果是true还是false
    - 如果是true就执行语句体1
    - 如果是false就继续判断关系表达式2看其结果是true还是false
    - true则执行语句体2，false就继续往下判断
    - 如果没有任何关系表达式为true，则执行else语句体
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/14/ifelseifelse-1555209112731.png)

### 2. switch选择语句
#### 2.1 选择语句--switch
- switch语句格式：
```java
switch(表达式){
    case 常量值1：
        语句体1；
	break；
    case 常量值2：
	语句体2；
	break；
    ...
    default:
	语句体n+1;
	break;
}
```
- 执行流程
    - 首先计算表达式的值
    - 其次，和case依次比较，一旦有对应的值，就会执行相应的语句，在执行的过程中，遇到break就会结束。
    - 最后，如果所有的case都和表达式的值不匹配，就会执行default语句体部分，然后程序结束。
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/14/switch-1555210158065.png)

实例：
```java
public class switch_exercise {
	public static void main(String[] agrs) {
		int weekday = 6;
		switch(weekday) {
			case 1:
				System.out.println("今天星期一");
				break;
			case 2:
				System.out.println("今天星期二");
				break;
			case 3:
				System.out.println("今天星期三");
				break;
			case 4:
				System.out.println("今天星期四");
				break;
			case 5:
				System.out.println("今天星期五");
				break;
			case 6:
				System.out.println("今天星期六");
				break;
			case 7:
				System.out.println("今天星期日");
				break;
			default:
				System.out.println("输入数据有误");
				break;
		}
	}
}
```

switch语句使用的注意事项：
1. 多个case后面的数值不可重复。
2. switch后面小括号中只能是下列数据类型：
    - 基本数据类型：byte/short/char/int
    - 引用数据类型：String字符串、enum枚举
3. switch语句格式可以很灵活：前后顺序可以颠倒，而且break语句还可以省略，但需要考虑case穿透性。<br>“匹配哪一个case就从哪一个位置向下执行，直到遇到break或者整体结束为止。”


#### 2.2 case的穿透性
在switch语句中，如果case的后面不写break，将出现穿透现象，也就是不会在判断下一个case的值，直接向后运行，直到遇到break，或者整体switch结束。<br>故编写switch语句时，必须写上break。

### 3. for循环语句
#### 3.1 循环概述
循环语句可以在满足循环条件的情况下，反复执行某一段代码，这段被重复执行的代码被称为循环体语句。当反复执行这个循环体时，需要在合适的时候把循环判断条件修改为false，从而结束循环，否则循环将一直执行下去，形成死循环。
#### 3.2 循环语句--for
- for循环语句格式：
```java
for（初始化表达式1；布尔表达式2；步进表达式4）{
	循环体3；
}
```
- 执行流程
    - 执行顺序：1234>234>234...直到2不满足为止。
    - 1负责完成循环变量初始化
    - 2负责判断是否满足循环条件，不满足则跳出循环
    - 3具体执行的语句
    - 4循环后，循环条件所涉及变量的变化情况

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/14/for-1555210990394.png)

实例：
```java
public class for_exercise {
	public static void main(String[] args) {
		int sum = 0;
		for (int i = 1;i <= 100;i++) {
			if(i % 2 == 0) {
				sum += i;
			}
		}
		System.out.println("sum:" + sum);
	}
}
```
### 4. while循环语句
- while循环语句格式：
```java
初始化表达式1：
    while（布尔表达式2）{
	循环体3；
	步进表达式4；
}
```
- 执行流程
    - 执行顺序：1234>234>234>234...直到2不满足为止。
    - 1负责完成循环变量初始化。
    - 2负责判断是否满足循环条件，不满足则跳出循环。
    - 3具体执行的语句。
    - 4循环后，循环变量的变化情况。

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/14/while-1555221264283.png)

实例：
```java
public class while_exercise {
	public static void main(String[] agrs) {
		int sum = 0;
		int i = 1;
		while(i <= 100) {
			sum += i;
			i++;
		}
		System.out.println("1~100的值为：" + sum);
	}
}
```
### 5. do while循环语句
- do...while循环格式：
```java
初始化表达式1
    do{
	循环体3；
	步进表达式4；
}while（布尔表达式2）；
```
- 执行流程
    - 执行顺序：134>234>234...知道2不满足为止。
    - 1负责完成循环变量初始化。
    - 2负责判断是否满足循环条件，不满足则跳出循环。
    - 3具体执行的语句
    - 4循环后，循环变量的变化情况

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/14/do...while-1555221971998.png)

实例：
```java
public class do_while_exercise {
	public static void main(String[] args) {
		int i = 1;
		do {
			System.out.println("hello:" + i);
			i++;
		}while(i<=10);
	}
}
```
Tips：do...while循环特点是会无条件执行一次循环体，即使我们将循环条件写出false，也依然会循环一次。这样的循环具有一定风险，不推荐使用。
### 7.循环语句的区别
- 三种循环的小区别：
    - 控制条件语句所控制的那个变量，在for循环结束后，就不能再被访问到了，而while循环结束还可以继续使用，如果你想继续使用，就用while，否则推荐使用for。原因是for循环结束，该变量就从内存中消失，能够提高内存的使用效率
    - 在已知循环次数的时候推荐使用for，循环次数未知时推荐使用while。
    - 如果条件判断从来没有满足过，那么for循环和while循环将会执行0次，但是do...while循环会执行至少一次。
    - for循环的变量在小括号当中定义，只有循环内部可以使用。while循环和do...while循环初始化语句本来就在外面，所以出了循环可以继续使用。


### 8. 跳出语句

#### break
- 使用场景：终止switch或者循环
    - 在选择结构switch语句中
    - 在循环语句中
    - 离开使用场景的存在是没有意义的

#### continue
- 使用场景：结束本次循环，继续下一次循环

### 9.扩展知识点

#### 9.1嵌套循环
- 所谓嵌套循环，是指一个循环的循环体是另一个循环。比如for循环里面还有一个for循环，就是嵌套循环。总共的循环次数 = 外循环次数 * 内循环次数
- 嵌套循环格式：
```java
for(初始化表达式1；循环条件2；步进表达式7){
    for(初始化表达式3；循环条件4；步进表达式6){
	执行语句5；
    }
}
```
- 嵌套循环执行流程：
    - 执行顺序：123456>456>723456>456
    - 外循环一次，内循环多次
    - 比如跳绳：一共5组，每组跳10个。5组就是外循环，10个就是内循环。
