---
title: Java中的线程池与Lambda表达式
date: 2019-5-3 23:59
categories: JavaBasics # 分类
tags: [线程,Lambda]
description: Java的等待唤醒机制，线程池和Lambda表达式
---

## 1. 等待唤醒机制
**概念**：多个线程在处理同一个资源，但处理的动作(线程的任务)却不相同。

比如：线程A用来生产包子，线程B用来吃包子，包子可以理解为同一资源。线程A与线程B的处理动作，一个是生产，一个是消费。那么线程A与线程B之间就存在线程通信问题。

<!--more-->

**为什么要处理线程间通信：**

多个线程并发执行时, 在默认情况下CPU是随机切换线程的，当我们需要多个线程来共同完成一件任务，并且我们希望他们有规律的执行, 那么多线程之间需要一些协调通信，以此来帮我们达到多线程共同操作一份数据。

**如何保证线程间通信有效利用资源：**

多个线程在处理同一个资源，并且任务不同时，需要线程通信来帮助解决线程之间对同一个变量的使用或操作。 就是多个线程在操作同一份数据时， 避免对同一共享变量的争夺。也就是我们需要通过一定的手段使各个线程能有效的利用资源。而这种手段即—— **等待唤醒机制。**

### 1.1 等待唤醒机制
**什么是等待唤醒机制**

这是多个线程间的一种**协作**机制。谈到线程我们经常想到的是线程间的**竞争（race）**，比如去争夺锁，但这并不是故事的全部，线程间也会有协作机制。就好比在公司里你和你的同事们，你们可能存在在晋升时的竞争，但更多时候你们更多是一起合作以完成某些任务。

就是在一个线程进行了规定操作后，就进入等待状态**wait()**， 等待其他线程执行完他们的指定代码过后 再将其唤醒**notify()** ;在有多个线程进行等待时， 如果需要，可以使用 notifyAll()来唤醒所有的等待线程。

wait/notify 就是线程间的一种协作机制。

**等待唤醒中的方法**

等待唤醒机制就是用于解决线程间通信的问题，使用的3个方法含义如下：

1. wait：线程不再活动，不再参与调度，进入 wait set 中，因此不会浪费 CPU 资源，也不会去竞争锁了，这时的线程状态即是 WAITING。它还要等着别的线程执行一个**特别的动作**，也即是“**通知（notify）**”在这个对象上等待的线程从wait set 中释放出来，重新进入到调度队列（ready queue）中
2. notify：则选取所通知对象的 wait set 中的一个线程释放；例如，餐馆有空位置后，等候就餐最久的顾客最先入座。
3. notifyAll：则释放所通知对象的 wait set 上的全部线程。

>注意：哪怕只通知了一个等待的线程，被通知线程也不能立即恢复执行，因为它当初中断的地方是在同步块内，而此刻它已经不在持有锁，所以它需要再次尝试获取锁(可能会面临其他线程的竞争),成功后才能在当初调用wait方法之后的地方恢复执行。

>总结：如果能获取锁，线程就从waiting状态变成Runnable状态；否则，从wait set出来，又会进入entry set，线程就从waiting状态变成了Blocked状态。

**调用wait和notify方法需要注意的细节**

1. wait方法与notify方法必须要由同一个锁对象调用。因为：对应的锁对象可以通过notify唤醒使用同一个锁对象调用的wait方法后的线程。
2. wait方法与notify方法是属于Object类的方法的。因为：锁对象可以是任意对象，而任意对象的所属类都是继承了Object类的。
3. wait方法与notify方法必须要在同步代码块或者是同步函数中使用。因为：必须要通过锁对象调用这2个方法。

### 1.2 生成者与消费者问题
等待唤醒机制其实就是经典的"生产者与消费者"的问题。

拿生产包子消费包子来说等待唤醒机制如何有效利用资源：

包子铺线程生产包子，吃货线程消费包子。当包子没有时（包子状态为false），吃货线程等待，包子铺线程生产包子（即包子状态为true），并通知吃货线程（解除吃货的等待状态）,因为已经有包子了，那么包子铺线程进入等待状态。接下来，吃货线程能否进一步执行则取决于锁的获取情况。如果吃货获取到锁，那么就执行吃包子动作，包子吃完（包子状态为false），并通知包子铺线程（解除包子铺的等待状态）,吃货线程进入等待。包子铺线程能否进一步执行则取决于锁的获取情况。

代码实例如下：
~~~java
// 资源类
public class Baozi {
    // 设置包子的属性
    String pier; // 皮
    String xianer; // 陷
    boolean flag = false; // 包子的状态：有就true，没有就false
}
// 生产者类
public class BaoZiPu extends Thread{ // 生成者(包子铺)类：是一个线程类，可以继承Thread
    // 1.需要在成员位置创建一个包子变量
    private Baozi bz;
    // 2.使用带参构造方法，为包子变量赋值
    public BaoZiPu(String name, Baozi bz){
        super(name);
        this.bz = bz;
    }

    // 设置线程任务(run)：生成包子
    /*
    包子铺线程和包子线程关系--->通信(互斥)
        必须使用同步技术保证两个线程只能有一个在执行
        锁对象必须保持唯一，可以使用包子对象作为锁对象
        包子铺类和吃货的类就需要把包子对象作为参数传递进来
            1.需要在成员位置创建一个包子变量
            2.使用带参构造方法，为包子变量赋值
     */
    @Override
    public void run(){
        int count = 0;
        // 造包子
        while (true){
             // 必须使用同步技术保证两个线程只能有一个在执行
            synchronized (bz) {
                if (bz.flag == true) { // 对包子状态进行判断
                    try {
                        bz.wait(); // true：有包子,包子铺调用wait方法进入等待状态
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                // 没有包子，包子铺生产包子
                System.out.println("包子铺开始做包子");
                if (count % 2 == 0) {
                    bz.pier = "面皮";
                    bz.xianer = "五花肉";
                } else {
                    bz.pier = "薄皮";
                    bz.xianer = "韭菜鸡蛋";
                }
                count++;
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 包子铺生成好了包子
                bz.flag = true; // 修改包子的状态为true
                System.out.println("包子造好了：" + bz.pier + bz.xianer+",吃货来吃吧!");
                bz.notify(); // 唤醒吃货线程，起来吃包子了
            }
        }
    }
}
// 消费者类
public class BaoZiPu extends Thread{ // 生成者(包子铺)类：是一个线程类，可以继承Thread
    // 1.需要在成员位置创建一个包子变量
    private Baozi bz;
    // 2.使用带参构造方法，为包子变量赋值
    public BaoZiPu(String name, Baozi bz){
        super(name);
        this.bz = bz;
    }

    // 设置线程任务(run)：生成包子
    /*
    包子铺线程和包子线程关系--->通信(互斥)
        必须使用同步技术保证两个线程只能有一个在执行
        锁对象必须保持唯一，可以使用包子对象作为锁对象
        包子铺类和吃货的类就需要把包子对象作为参数传递进来
            1.需要在成员位置创建一个包子变量
            2.使用带参构造方法，为包子变量赋值
     */
    @Override
    public void run(){
        int count = 0;
        // 造包子
        while (true){
             // 必须使用同步技术保证两个线程只能有一个在执行
            synchronized (bz) {
                if (bz.flag == true) { // 对包子状态进行判断
                    try {
                        bz.wait(); // true：有包子,包子铺调用wait方法进入等待状态
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                // 没有包子，包子铺生产包子
                System.out.println("包子铺开始做包子");
                if (count % 2 == 0) {
                    bz.pier = "面皮";
                    bz.xianer = "五花肉";
                } else {
                    bz.pier = "薄皮";
                    bz.xianer = "韭菜鸡蛋";
                }
                count++;
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 包子铺生成好了包子
                bz.flag = true; // 修改包子的状态为true
                System.out.println("包子造好了：" + bz.pier + bz.xianer+",吃货来吃吧!");
                bz.notify(); // 唤醒吃货线程，起来吃包子了
            }
        }
    }
}
// 测试类
public class TestDemo {
    public static void main(String[] args) {
        Baozi bz = new Baozi(); // 创建包子对象
        new BaoZiPu("包子铺",bz).start(); // 创建包子铺线程，开启，生产包子
        new Chihuo("吃货",bz).start(); // 创建吃货线程，开启，吃包子
    }
}
~~~

## 2. 线程池
### 2.1 线程池思想概述
如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。

那么有没有一种办法使得线程可以复用，就是执行完一个任务，并不被销毁，而是可以继续执行其他的任务？

在Java中可以通过线程池来达到这样的效果。下面就详细讲解一下Java的线程池。

### 2.2 线程池概念
* **线程池**：其实就是一个容纳多个线程的容器，其中的线程可以反复使用，省去了频繁创建线程对象的操作，无需反复创建线程而消耗过多资源。
下图描述线程池的工作原理：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/29/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%8E%9F%E7%90%86-1556526310665.bmp)

合理利用线程池能够带来三个好处：
1. 降低资源消耗。减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。
2. 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
3. 提高线程的可管理性。可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

### 2.3 线程池的使用
Java里面线程池的顶级接口是`java.util.concurrent.Executor`，但是严格意义上讲`Executor`并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是`java.util.concurrent.ExecutorService`。

要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的,因此`java.util.concurrent.Executors`线程工厂类里面提供了一些静态工厂，生成一些常用的线程池。官方建议使用Executors工程类来创建线程池对象。

Executors类中有个创建线程池的方法如下：
* `public static ExecutorService newFixedThreadPool(int n Threads)`：返回线程池对象。(创建的是有界线程池,也就是池中的线程个数可以指定最大数量)

使用线程池对象的方法如下：
* `public Future<?> submit(Runnable task)`:获取线程池中的某一个线程对象，并执行

> Future接口：用来记录线程任务执行完毕后产生的结果。线程池创建与使用。

使用线程池中线程对象的步骤：
1. 创建线程池对象。
2. 创建Runnable接口子类对象。(task)
3. 提交Runnable接口子类对象。(take task)
4. 关闭线程池(一般不做)。

Runnable实现类实例如下：
```java
// 实现类
public class MyRunnable implements Runnable{
    @Override
    public void run() {
        System.out.println("我要一个教练");
        try{
            Thread.sleep(2000);
        }catch(InterruptedException e){
            e.printStackTrace();
        }
        System.out.println("教练来了："+Thread.currentThread().getName());
        System.out.println("教我游泳,教完后，教练回到了游泳池");
    }
}
// 线程池测试类
public class ThreadPoolDemo {
    public static void main(String[] args) {
        // 创建线程池对象
        ExecutorService service = Executors.newFixedThreadPool(2); // 创建了包含2个线程对象的线程池
        // 创建Runnable实例对象
        MyRunnable r = new MyRunnable();
//        new Thread(r).start(); // 使用传统方式，自己创建线程对象
        service.submit(r); // 从线程池中获取线程对象，然后会自动调用MyRunnable中的run()
        service.submit(r); // 再次获取一个线程对象
        service.submit(r); // again
        /*
            注意：submit方法调用结束后，程序并没终止，是因为线程池控制了线程的关闭
            将使用完的线程又归还到了线程池中
         */
        service.shutdown();
    }
}
```

## 3. Lambda表达式
### 3.1 函数式编程思想概述
在数学中，**函数**就是有输入量、输出量的一套计算方案，也就是“拿什么东西做什么事情”。相对而言，面向对象过分强调“必须通过对象的形式来做事情”，而函数式思想则尽量忽略面向对象的复杂语法——**强调做什么，而不是以什么形式做**

面向对象的思想:做一件事情,找一个能解决这个事情的对象,调用对象的方法,完成事情。

函数式编程思想:只要能获取到结果,谁去做的,怎么做的都不重要,重视的是结果,不重视过程

### 3.2 冗余的Runnable代码
#### 传统写法
当需要启动一个线程去完成任务是，通常会通过`java.lang.Runnable`接口来定义任务内容，并使`java.lang.Thread`类来启动该线程。如下所示：
~~~java
public class Demo01Runnable {
	public static void main(String[] args) {
    	// 匿名内部类
		Runnable task = new Runnable() {
			@Override
			public void run() { // 覆盖重写抽象方法
				System.out.println("多线程任务具体执行内容！");
			}
		};
		new Thread(task).start(); // 启动线程
	}
}
~~~
本着“一切皆对象”的思想，这种做法是无可厚非的：首先创建一个`Runnable`接口的匿名内部类对象来指定任务内容，再将其交给一个线程来启动。
#### 代码分析
对于`Runnable`的匿名内部类用法，可以分析出几点内容：
- `Thread`类需要`Runnable`接口作为参数，其中的抽象`run`方法是用来指定线程任务内容的核心；
- 为了指定`run`的方法体，**不得不**需要`Runnable`接口的实现类；
- 为了省去定义一个`RunnableImpl`实现类的麻烦，**不得不**使用匿名内部类；
- 必须覆盖重写抽象`run`方法，所以方法名称、方法参数、方法返回值**不得不**再写一遍，且不能写错；
- 而实际上，**似乎只有方法体才是关键所在**。

### 3.3 编程思想转换
#### 专注于具体做什么，而不是这么做
很多时候我们为了做具体的事情而不得不创建了一个匿名内部类对象来完成，这是非必要的。我们真正希望做的事情是：将run方法体内的代码传递给Thread类知晓。

**传递一段代码** --这才是我们真正的目的。而创建对象只是受限于面向对象语法而不得不采取的一种手段方式。如果我们将关注点从“怎么做”回归到“做什么”的本质上，就会发现只要能够更好地达到目的，过程与形式其实并不重要。

2014年3月Oracle所发布的Java 8（JDK 1.8）中，加入了**Lambda表达式**的重量级新特性，为我们打开了新世界的大门。

### 3.4 体验Lambda的更优写法
借助Java 8的全新语法，上述`Runnable`接口的匿名内部类写法可以通过更简单的Lambda表达式达到等效：
```java
public class DemoLambdaRunnable {
    public static void main(String[] args) {
        new Thread(() -> System.out.println("多线程任务执行！")).start(); // 启动线程
    }
}
```
这段代码同上一段执行效果完全一样。从代码的语义中可以看出：我们启动了一个线程，而线程任务的内容以一种更加简洁的形式被指定。

### 3.5 回顾匿名内部类
#### 使用实现类
要启动一个线程，需要创建一`Thread`类的对象调用`start`方法。而为了指定线程执行的内容，需要调用`Thread`类的构造方法：
* `public Thread(Runnable target)`
为了获取`Runnable`接口的实现对象，还要为该接口定义一个实现类`RunnableImpl`：
~~~java
public class RunnableImpl implements Runnable{
    @Override
    public void run(){
	System.out.println("多线程任务执行！")
    }
}
~~~
然后创建该实现类的对象作为`Thread`类的构造参数：
```java
public class ThreadInitParam{
    public static void main(String[] args){
	Runnable task = new RunnableImpl();
	new Thread(task).start();
    }
}
```

#### 使用匿名内部类
这个`RunnableImpl`类只是为了实现`Runnable`接口而存在的，而且仅被使用了唯一一次，所以使用匿名内部类的语法即可省去该类的单独定义，即匿名内部类：
```java
public class ThreadNameless {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("多线程任务执行！");
            }
        }).start();
    }
}
```
#### 匿名内部类的好处与弊端
一方面，匿名内部类可以帮我们**省去实现类的定义**；另一方面，匿名内部类的语法——**确实太复杂了！**

#### 语义分析
仔细分析该代码中的语义，`Runnable`接口只有一个`run`方法的定义：
* `public abstract void run();`
即制定了一种做事情的方案（其实就是一个函数）：
- **无参数**：不需要任何条件即可执行该方案。
- **无返回值**：该方案不产生任何结果。
- **代码块**（方法体）：该方案的具体执行步骤。

同样的语义体现在`Lambda`语法中，要更加简单：
~~~java
() -> System.out.println("多线程任务执行！")
~~~
- 前面的一对小括号即`run`方法的参数（无），代表不需要任何条件；
- 中间的一个箭头代表将前面的参数传递给后面的代码；
- 后面的输出语句即业务逻辑代码。

### 3.6 Lambda标准格式
Lambda省去面向对象的条条框框，格式由**3个部分**组成：
- 一些参数
- 一个箭头
- 一段代码
Lambda表达式的**标准格式**为：(参数类型 参数名称) -> { 代码语句 }
格式说明：
- 小括号内的语法与传统方法参数列表一致：无参数则留空；多个参数则用逗号分隔。
- `->`是新引入的语法格式，代表指向动作。
- 大括号内的语法与传统方法体要求基本一致。


### 3.7 Lambda的参数和返回值
// Todo 使用数组存储多个Person对象。

// Todo 对数组中的Person对象使用Arrays的sort方法通过年龄进行升序排序。

下面举例演示`java.util.Comparator<T>`接口的使用场景代码，其中的抽象方法定义为：
* `public abstract int compare(T o1, T o2);`

当需要对一个对象数组进行排序时，`Arrays.sort`方法需要一个`Comparator`接口实例来指定排序的规则。假设有一个`Person`类，含有`String name`和`int age`两个成员变量：
~~~java
public class Person{
    private String name;
    private int age;

    // 省略构造器、toString方法与Getter Setter 
}
~~~

#### 传统写法
使用传统代码对`Person[]`数组进行排序
~~~java
public class DemoComparator {
    public static void main(String[] args) {
        // 创建一个Person数组
        Person[] array = {
                new Person("xiaom",13),
                new Person("xiaoz",17),
                new Person("xiaol",15)};
        // Comparator接口的实例（使用了匿名内部类）重写了compare方法，制定了"按照年龄从小到大升序(前-后)"的排序规则。
        Comparator<Person> comp = new Comparator<Person>() {
            @Override
            public int compare(Person o1, Person o2) {
                return o1.getAge() - o2.getAge();
            }
        };
        Arrays.sort(array,comp); // 第二个参数为排序规则，即Comparator接口实例
        for(Person person:array){
            System.out.println(person);
        }
    }
}
~~~

#### 代码分析
- 为了排序，`Arrays.sort`方法需要排序规则，即`Comparator`接口的实例，抽象方法`compare`是关键；
- 为了指定`compare`的方法体，**不得不**需要`Comparator`接口的实现类；
- 为了省去定义一个`ComparatorImpl`实现类的麻烦，**不得不**使用匿名内部类；
- 必须覆盖重写抽象`compare`方法，所以方法名称、方法参数、方法返回值**不得不**再写一遍，且不能写错；
- 实际上，**只有参数和方法体才是关键**。

#### Lambda写法
```java
public class DemoComparatorLambda {
    public static void main(String[] args) {
        // 创建一个Person数组
        Person[] array = {
                new Person("xiaom",13),
                new Person("xiaoz",17),
                new Person("xiaol",15)};
        Arrays.sort(array,(Person a,Person b) -> {return a.getAge() - b.getAge();});
        for(Person person:array){
            System.out.println(person);
        }
    }
}
```

### 3.8 练习
给定一个计算器`Calculator`接口，内含抽象方法`calc`可以将两个int数字相加得到和值：
```java
// 定义接口类
public interface Calculator {
    public abstract int calc(int a,int b);
}
// 定义测试类
public class DemoInvokeCalc {
    public static void main(String[] args) {
        // 使用匿名内部类实现
        invokeCalc(10, 20, new Calculator() {
            @Override
            public int calc(int a, int b) {
                return a + b;
            }
        });
        // TODO 请在此使用Lambda[标准格式]调用invokeCalc方法来计算120+130的结果
        invokeCalc(120,130,(int a,int b)-> {return a+b;}); // 使用Lambda表达式实现
	invokeCalc(20,30,(a, b) -> a + b); // Lambda表达式省略格式写法
    }
    private static void invokeCalc(int a,int b,Calculator calculator){// 三个参数，2个int类型整数，1个接口
        int result = calculator.calc(a, b);
        System.out.println("结果是："+result);
    }
}
```

### 3.9 Lambda省略格式
#### 可推导即可省略
Lambda强调的是“做什么”而不是“怎么做”，所以凡是可以根据上下文推导得知的信息，都可以省略。例如上例还可以使用Lambda的省略写法：
~~~java
public static void main(String[] args){
	invokeCalc(120,130,(a,b)->a+b);
}
~~~
#### 省略规则
在Lambda标准格式的基础上，使用省略写法的规则为：
1. 小括号内参数的类型可以省略；
2. 如果小括号内**有且仅有一个参**，则小括号可以省略；
3. 如果大括号内**有且仅有一个语句**，则无论是否有返回值，都可以省略大括号、return关键字及语句分号。

### 3.10 Lambda的使用前提
Lambda的语法非常简洁，完全没有面向对象复杂的束缚。但是使用时有几个问题需要特别注意：
1. 使用Lambda必须具有接口，且要求**接口中有且仅有一个抽象方法**。
   无论是JDK内置的`Runnable`、`Comparator`接口还是自定义的接口，只有当接口中的抽象方法存在且唯一时，才可以使用Lambda。
2. 使用Lambda必须具有**上下文推断**。
   也就是方法的参数或局部变量类型必须为Lambda对应的接口类型，才能使用Lambda作为该接口的实例。

> 注：有且仅有一个抽象方法的接口，称为“**函数式接口**”。