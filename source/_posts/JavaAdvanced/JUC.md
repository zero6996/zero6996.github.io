---
title: JUC
date: 2020-05-11 20:00
categories: JavaAdvanced
tags: 多线程
urlname: JUC
---



## 1. JUC

JUC是`java.util.concurrent`工具包的简称，JDK1.5开始出现的，这是一个处理线程的工具包，在此包中有很多在并发编程中很常用的工具类。

主要由三大包构成：

1. `java.util.concurrent`：并发包
2. `java.util.concurrent.atomic`：并发原子包
3. `java.util.concurrent.locks`：并发锁包

### 1.1 进程线程回顾

在学习JUC之前，复习一些基础概念

1. 什么是进程/线程？
   - 进程：windows打开任务管理器，里面运行的程序就是进程，比如idea.exe就是进程。
   - 线程：在一个进程下运行的所有程序，就是线程，比如idea中的动态代码检查，语法检查等功能就是线程。
2. 并发/并行？
   - 并发：同一时刻多个线程访问同一资源，比如春运抢票，电商的秒杀功能。
   - 并行：多个处理器同时处理多个任务，比如一个人要吃饭，手机充电这些都可以同时进行。
3. 线程有哪些状态？六种状态：NEW(新建)，RUNNABLE(就绪)，BLOCKED(阻塞)，WAITING(等待)，TIMED_WAITING(计时等待)，TERMINATED(终结)。

## 2. Lock接口

Lock接口是java并发包下的；锁实现提供了比使用同步方法和语句可以获得的更广泛的锁操作，它们允许更灵活的结构，可能具有非常不同的数学，且支持多个关联的condition对象。

### 2.1 生产者消费者案例

需求：两个线程，可以操作初始值为零的一个变量，需实现一个线程对该变量加1，一个线程对该变量减1，交替执行10轮，变量初始值为0。在高聚低合前提下，线程操作资源类。

- 基础版synchronized代码如下：

```java
public class ProdConsumeDemo {
    public static void main(String[] args) throws Exception{
        Aircondition aircondition = new Aircondition();
        new Thread(()->{
            for (int i = 0; i <= 10; i++) {
                try {
                    Thread.sleep(200);
                    aircondition.increment();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        },"生产者A").start();
        new Thread(()->{
            for (int i = 0; i <= 10; i++) {
                try {
                    Thread.sleep(300);
                    aircondition.decrement();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        },"消费者B").start();
    }
}
// 空调类
class Aircondition{
    private int number = 0;

    // 模拟生产者
    public synchronized void increment() throws Exception{
        // 1. 判断
        if (number != 0){
            this.wait();
        }
        // 2. 工作
        number++;
        System.out.println(Thread.currentThread().getName()+"\t"+number);
        // 3. 通知，唤醒其他等待线程
        this.notifyAll();
    }
    // 模拟消费者
    public synchronized void decrement() throws Exception{
        // 1. 判断
        if (number == 0){
            this.wait();
        }
        // 2. 工作
        number--;
        System.out.println(Thread.currentThread().getName()+"\t"+number);
        // 3. 通知
        this.notifyAll();
    }
}
```

现在将生产者和消费者都增加一个，共计4个线程，然后执行代码会发现数据错误，这就是多线程的虚假唤醒。原因出在if的使用，多线程判断时，不能用if；使用wait时必须注意，中断和虚假唤醒是可能的，故该方法应该**在循环中使用**；将if改为while即可。

- 改进写法，优化多线程情况下可能发生的虚假唤醒

```java
// 将代码添加和修改如下即可
// 生产者和消费者都增加一个
new Thread(()->{
    for (int i = 0; i <= 10; i++) {
        try {
            Thread.sleep(400);
            aircondition.increment();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
},"生产者C").start();

new Thread(()->{
    for (int i = 0; i <= 10; i++) {
        try {
            Thread.sleep(500);
            aircondition.decrement();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
},"消费者D").start();
// 修改if为while
while (number != 0){
    this.wait();
}
while (number == 0){
    this.wait();
}
```

### 2.2 为什么在wait使用if的情况下，会产生虚假唤醒？

由于cpu的调度是不可控的，所以在生产者A生产完后，下一个可能不是消费者执行，而是生产者B继续执行，如果使用if就会进行判断number!=0，进入wait状态等待，此时生产者A再次获取时间片，执行代码if判断进入wait状态；这样两个生产者同时处于wait状态了，后续如果唤醒就会使number++两次，导致值为2。同理消费者也是。
所以要使用while**循环判断条件**是否满足，才可以交替执行生产和消费。

### 2.3 新版写法

使用了condition类的`condition.await();`进行资源的上锁，`condition.signalAll();`方法进行资源的唤醒。

```java
// 空调类
class Aircondition{
    private int number = 0;
    // 新版写法
    private Lock lock = new ReentrantLock();
    // 使用新版的通知唤醒
    private Condition condition = lock.newCondition();

    // 模拟生产者
    public void increment(){
        lock.lock();
        try{
            // 1. 判断
            while (number != 0){
                condition.await();
            }
            // 2. 工作
            number++;
            System.out.println(Thread.currentThread().getName()+"\t"+number);
            // 3. 通知，唤醒其他等待线程
            condition.signalAll();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }
    // 模拟生产者
    public void decrement(){
        lock.lock();
        try{
            // 1. 判断
            while (number == 0){
                condition.await();
            }
            // 2. 工作
            number--;
            System.out.println(Thread.currentThread().getName()+"\t"+number);
            // 3. 通知，唤醒其他等待线程
            condition.signalAll();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }
}
```

### 2.4 生产者和消费者知识小结

- 多线程编程模板：线程->操作->资源类，高内聚低耦合
- 多线程编程套路：判断->工作->通知
- while循环判断(防止虚假唤醒)
- 新版写法(lock+condition)





### 2.5 面试题，交替打印

需求：两个线程，一个打印1-26，一个打印A-Z，要求打印顺序为1A2B...26Z

```java
/**
ASCII码,A=65，Z=90，故A-Z=65-90
(char)65=A
*/
public class printNumberAndtheAlphabet {
    public static void main(String[] args) {
        printData data = new printData();
        new Thread(()->{
            for (int i = 1; i <= 26 ; i++) {
                data.printNumber();
            }
        },"打印数字线程").start();
        new Thread(()->{
            for (int i = 1; i <= 26 ; i++) {
                data.printAlphabet();
            }
        },"打印字母线程").start();
    }
}

class printData{
    int Number = 1;
    int ASCIINumber = 65;
    int flag = 0;
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void printNumber(){
        lock.lock();
        try{
            while (flag != 0){
                condition.await();
            }
            if (Number<=26) System.out.println(Thread.currentThread().getName()+"\t"+Number);
            Number++;
            flag++;
            condition.signalAll();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }
    public void printAlphabet(){
        lock.lock();
        try{
            while (flag==0){
                condition.await();
            }
            if (ASCIINumber<=90) System.out.println(Thread.currentThread().getName()+"\t"+(char)ASCIINumber);
            ASCIINumber++;
            flag--;
            condition.signalAll();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }
}
```

### 2.6 使用Condition精准唤醒，实现线程之间的顺序调度

要求：多线程之间按顺序调用，实现A->B->C；三个线程启动，要求AA打印5次，BB打印10次，CC打印15次，接着AA打印5次，BB打印10次，CC打印15次，共计10轮。代码示例如下：

- 资源类，完成打印操作和精准唤醒

```java
class ShareData{
    // 1=A,2=B,3=C
    private int number = 1;
    private Lock lock = new ReentrantLock();
    // 精准唤醒
    Condition c1 = lock.newCondition();
    Condition c2 = lock.newCondition();
    Condition c3 = lock.newCondition();
    public void print5(){
        lock.lock();
        try{
            // 1. 判断
            while (number != 1){
                c1.await();
            }
            // 工作
            for (int i = 0; i < 5 ; i++) {
                System.out.println(Thread.currentThread().getName() + "\t" + i);
            }
            // 通知
            // 一定要先改标志位
            number = 2;
            // 通知下一位：精准唤醒下一个线程
            c2.signal();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }
    public void print10(){
        lock.lock();
        try{
            // 1. 判断
            while (number != 2){
                c2.await();
            }
            // 工作
            for (int i = 0; i < 10 ; i++) {
                System.out.println(Thread.currentThread().getName() + "\t" + i);
            }
            // 通知
            // 一定要先改标志位
            number = 3;
            c3.signal();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }
    public void print15(){
        lock.lock();
        try{
            // 1. 判断
            while (number != 3){
                c3.await();
            }
            // 工作
            for (int i = 0; i < 15 ; i++) {
                System.out.println(Thread.currentThread().getName() + "\t" + i);
            }
            // 通知
            // 一定要先改标志位
            number = 1;
            c1.signal();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }
}
```

- 主类，三个线程操作资源类，执行10轮

```java
public class ConditionDemo {
    public static void main(String[] args) {
        ShareData shareData = new ShareData();
        new Thread(()->{
            for (int i = 1; i <= 10; i++) {
                shareData.print5();
            }
        },"A").start();
        new Thread(()->{
            for (int i = 1; i <= 10; i++) {
                shareData.print10();
            }
        },"B").start();
        new Thread(()->{
            for (int i = 1; i <= 10; i++) {
                shareData.print15();
            }
        },"C").start();
    }
}
```

## 3. Lambda表达式

复习lambda表达式

- 传统写法

```java
public class LambdaExpressDemo {
    public static void main(String[] args) {
		Foo foo = new Foo(){
            @Override
            public void sayHello(){
                System.out.println("hello world");
            }
        }
        foo.sayHello();
    }
}
interface Foo{
    public void sayHello();
}
```

- lambda写法

```java
Foo foo = (()->{System.out.println("hello world")};);
```

### 3.1 lambda小口诀

拷贝中括号(就是你要实现的方法的参数列表)，写死右箭头，落地大括号

```java
()->{}
```

### 3.2 函数式接口

`@FunctionalInterface`注解用于标识该接口是函数式接口，如果该接口中只有一个抽象方法(不包括默认方法和静态方法)，可以不写这个注解，底层会默认加上。

### 3.3 default方法

面试题：接口中是否允许有方法的实现？：Java8以后接口支持默认方法，可以让方法有对应的实现。

```java
@FunctionalInterface
interface Foo{
    public int add(int x,int y);
    public default int mul(int x,int y){
        return x * y;
}
Foo foo = (int x,int y)-> {
    System.out.println("come in add method");
    return x + y;
};
System.out.println(foo.mul(3,5));
```

函数式接口中允许有多个默认方法。

### 3.4 static方法

函数式接口中允许static方法的声明和实现。

```java
@FunctionalInterface
interface Foo{
    public int add(int x,int y);
    public static int diy(int x,int y){
        return x/y;
    }
}
System.out.println(Foo.diy(6,3));
```



## 4. 多线程锁

### 4.1 八锁

简单代码举例八锁，基础代码如下，后续案例均修改部分代码即可

```java
public class Locks8Demo {
    public static void main(String[] args) throws Exception {
        Phone phone = new Phone();
        Phone phone2 = new Phone();
        new Thread(()->{
            try {
                phone.sendEmail();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"A").start();
        Thread.sleep(100);
        new Thread(()->{
            try {
                //                phone.sendSMS();
                //                phone.sayHello();
                phone2.sendSMS();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"B").start();

    }
}
class Phone{
    public  static synchronized void sendEmail() throws Exception{
        TimeUnit.SECONDS.sleep(2);
        System.out.println("----->sendEmail<");
    }
    public synchronized void sendSMS() throws Exception{
        System.out.println("----->sendSMS<");
    }
    public void sayHello(){
        System.out.println("----->sayHello");
    }
}
```

1. 标准访问，先打印邮件还是短信？

```java
public synchronized void sendEmail() throws Exception{
    System.out.println("----->sendEmail<");
}
public synchronized void sendSMS() throws Exception{
    System.out.println("----->sendSMS<");
}
Phone phone = new Phone();
phone.sendEmail();
phone.sendSMS();
// 邮件>短信
```

2. 在邮件方法中暂停2秒，先打印邮件还是短信？

```java
public synchronized void sendEmail() throws Exception{
    TimeUnit.SECONDS.sleep(2);
    System.out.println("----->sendEmail<");
}
phone.sendEmail();
phone.sendSMS();
// 邮件>短信
```

1、2小结：

同一实例对象里面如果有多个synchronized方法，某一时刻内，只要一个线程去调用其中的一个synchronized方法，其他线程都只能等待，换言之，某一时刻内只能有唯一一个线程去访问这些synchronized方法， "**锁的是当前实例对象this**，被锁定后，其他线程都不能进入到当前对象的其他synchronized方法"。

3. 新增普通sayHello方法，先打印邮件还是hello？

```java
public void sayHello(){
    System.out.println("----->sayHello");
}
phone.sendEmail();
phone.sayHello();
// hello>邮件
```

3小结：普通方法与同步锁无关，不会竞争资源

4. 两部手机，先邮件还是短信？

```java
Phone phone = new Phone();
Phone phone2 = new Phone();
phone.sendEmail();
phone2.sendSMS();
// 短信>邮件
```

4小结：不同实例对象不是同一把锁，故不会竞争资源。

5. 两个静态同步方法，同一部手机，请问先打印邮件还是短信？

```java
public static synchronized void sendEmail() throws Exception{
    TimeUnit.SECONDS.sleep(2);
    System.out.println("----->sendEmail<");
}
public static synchronized void sendSMS() throws Exception{
    System.out.println("----->sendSMS<");
}
phone.sendEmail();
phone.sendSMS();
// 邮件>短信
```

6. 两个静态同步方法，两部手机，请问先打印邮件还是短信？

```java
phone.sendEmail();
phone2.sendSMS();
//  邮件>短信
```

5和6小结：**静态同步锁=全局锁**
synchronized实现同步的基础：Java中的每一个对象都可以作为锁，具体表现为以下3种形式：
一、对于普通同步方法，锁是当前实例对象(this)
二、对于同步方法块，锁的是synchronized括号里配置的对象
三、对于静态同步方法，锁的是当前类的Class对象(全局锁)

因为5和6都是静态同步锁也就是全局锁，实际上他们属于同一个类对象，故会竞争资源。

7. 1个静态同步方法，1个普通同步方法，同一部手机，请问先打印邮件还是短信？

```java
public  static   synchronized void sendEmail() throws Exception{
    TimeUnit.SECONDS.sleep(2);
    System.out.println("----->sendEmail<");
}
public  synchronized void sendSMS() throws Exception{
    System.out.println("----->sendSMS<");
}
phone.sendEmail();
phone.sendSMS();
// 短信>邮件
```

7小结：静态同步方法和非静态同步方法不是同一把锁，故不冲突。

8. 1个静态同步方法，1个普通同步方法，两部手机，请问先打印邮件还是短信？同上。

### 4.2 八锁小结

一个对象里面如果有多个synchronized方法，某一个时刻内，只要一个线程去调用其中的一个synchronized方法了，其它的线程都只能等待，换句话说，某一个时刻内，只能有唯一一个线程去访问这些synchronized方法
锁的是当前对象this，被锁定后，其它的线程都不能进入到当前对象的其它的synchronized方法。

synchronized实现同步的基础：Java中的每一个对象都可以作为锁。
具体表现为以下3种形式。

- 对于普通同步方法，锁是当前实例对象。
- 对于静态同步方法，锁是当前类的Class对象。
- 对于同步方法块，锁是Synchonized括号里配置的对象

当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。

- 非静态同步方法：

当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。
也就是说如果一个实例对象的非静态同步方法获取锁后，该实例对象的其他非静态同步方法必须等待锁释放后才能获取锁，其他实例对象的非静态同步方法因为跟该实例对象的非静态同步方法用的是不同锁，所以无需等待该实例对象已获取锁的非静态同步方法释放锁就可以获取他们自己的锁。

> 简言之：非静态同步锁当前对象，其他非静态锁访问同实例变量的锁会产生竞争。

- 静态同步方法(全局锁)：

所有的静态同步方法用的是同一把锁---**类对象本身(Class)**；这静态同步锁和非静态同步锁两把锁是锁的两个不同的对象，所以**静态同步方法与非静态同步方法之间是不会有竞态条件的。**
但是一旦一个静态同步方法获取锁后，其他的当前类对象的所有静态同步方法都必须等待该方法释放锁后才能获取锁，不管是同一个实例对象还是不同实例对象，只要它们是同一个类的实例对象，那么它们的静态同步方法就是同一把锁。

> 简言之：静态同步锁类对象本身，故称之为全局锁，其他静态同步锁访问同类对象的锁会产生竞争，非静态锁的不同对象故不会产生竞争。



## 6. 线程不安全案例

### 6.1 ArrayList线程不安全的原因详解

从以下4个维度解释ArrayList线程不安全的原因

#### 6.1.1 故障现象

报`java.util.ConcurrentModificationException`错误，ArrayList在迭代时如果同时对其进行修改就会抛出并发修改异常。

#### 6.1.2  导致原因

多线程情况下同时争抢一个资源类且没有加锁

#### 6.1.3 解决方法

1. 使用java自带的`new Vector()`，代码底层加了synchronized锁保证线程安全；
2. 使用Collections工具类的`synchronizedList(new ArrayList<>())`
3. 使用juc包下的`new CopyOnWriteArrayList<>(); `写时复制技术

#### 6.1.4 代码示例

```java
public class arrayListNotSafe {
    public static void main(String[] args) {
        mapNotSafe();
    }

    private static void mapNotSafe() {
        Map<String,String> map = new ConcurrentHashMap();//new HashMap<>();
        for (int i = 1; i <= 100; i++) {
            new Thread(()->{
                map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0,8));
                System.out.println(map);
            },String.valueOf(i)).start();
        }
    }

    private static void setNotSafe() {
        Set<String> set = new CopyOnWriteArraySet<>(new HashSet<>());//new HashSet<>();
        for (int i = 1; i <= 100; i++) {
            new Thread(()->{
                set.add(UUID.randomUUID().toString().substring(0,8));
                System.out.println(set);
            },String.valueOf(i)).start();
        }
    }

    private static void listNotSafe() {
        // 分别使用ArrayList、Vector、synchronizedList、CopyOnWriteArrayList进行测试
        List<String> list = new CopyOnWriteArrayList<>();// Collections.synchronizedList(new ArrayList<>());//new Vector<>();//new ArrayList<>();
        for (int i = 1; i <= 100; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,8));
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
}
```

### 6.2 写时复制技术

CopyOnWrite容器即写时复制的容器。往一个容器添加元素时，不直接往当前容器Object[]添加，而是先将当前容器进行Copy，复制出一个新的容器Object[] newElements，然后在新容器中添加元素，添加元素之后，将原容器的引用指向新容器setArray(newElements);

这样做的好处是可以对CopyOnWrite容器进行并发的读而无需加锁，因为当前容器不会添加任何元素，所以CopyOnWrite容器也是一种**读写分离的思想**，读和写不同的容器。

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

##  7. Callable接口

Callable类似于Runnable接口，实现Callable接口的类和实现Runnable的类都是可被其它线程执行的任务，Callable接口可以获取任务执行的返回值，通过与Future结合，可以实现异步计算；Callable经常和java线程池一起使用，同时它也是一个函数式接口。

### 7.1 获取多线程的几种方法

传统的是继承thread类和实现runnable接口，java5之后又可以实现callable接口和java的线程池获取。

### 7.2 Callable和Runnable的区别

主要有两点区别：

1. 方法不同，一个call()一个run()；
2. run方法没有返回值；call方法可以返回执行结果；
3. run方法异常只能内部消化，不能往外抛；call方法允许抛出异常。

```java
//Callable 接口
public interface Callable<V> {
    V call() throws Exception;
}
// Runnable 接口
public interface Runnable {
    public abstract void run();
}
```

### 7.3 FutureTask

由于Thread类不能直接使用Callable接口，故需要使用到FutureTask做“中间人”， FutureTask类实现RunnableFuture接口，而RunnnableFuture接口继承了Runnable和Future接口，所以说FutureTask是一个提供异步计算的结果的任务。 

可以使用FutureTask来包装Callable或者Runnable对象，然后交给Thread或线程池做执行，Callable有以下两种执行方式：

- Callable借助FutureTask执行：

```java
// 定义实现Callable接口的实现类
class MyThread implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        System.out.println("come in call Callable()");
        return 1024;
    }
}

public class CallableDemo{
    public static void main(String[] args) throws Exception{
        // 创建FutureTask对象并将callable对象放入
        FutureTask<Integer> task = new FutureTask(new MyThread());
        // 开启线程
        new Thread(task,"A").start();
        // 获取执行结果
        System.out.println(task.get());
        System.out.println(Thread.currentThread().getName()+"计算完成");
    }
}
```

- 借助线程池来运行

```java
// 创建一个线程池
ExecutorService pool = Executors.newCachedThreadPool();
// 提交一个任务
Future<Integer> future = pool.submit(new MyThread());
// 获取执行结果
System.out.println(future.get());
// 关闭线程池
pool.shutdown();
```

## 8. JUC常用辅助类

### 8.1 `CountDownLatch`减法计数器

案例：下课教室走人场景，有6位同学要走但走的顺序不一定，班长必须等待所有同学走完才允许关门走人。

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        // 使用CountDownLatch计数器来控制线程顺序，定义一个值为6的计数器。
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t离开教室");
                countDownLatch.countDown();
            }, String.valueOf(i)).start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName() + "\t班长关门走人");
    }

}
```

CountDownLatch主要有两个方法，当一个或多个线程调用await方法时，这些线程会阻塞。其他线程调用countDown方法会将计数器减1(调用countDown方法的线程不会阻塞)，当计数器的值为0时，被await方法阻塞的线程会被唤醒，继续执行。

### 8.2 `CyclicBarrier`循环屏障

CyclicBarrier字面意思是可循环的(Cyclic)使用的屏障(Barrier)；主要可以做到让一组线程到达一个屏障(也可叫同步点)时被阻塞，直到最后一个线程到达屏障时，屏障才会开门让所有被屏障拦截的线程继续工作。

- 线程进入屏障使用CyclicBarrier的`await()`方法，代码案例如下：

```java
// 案例：只有集齐7颗龙珠，才能召唤神龙
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier barrier = new CyclicBarrier(7,()-> System.out.println(">>>7龙珠召唤神龙<<<"));
        for (int i = 1; i <= 7; i++) {
            final int finalI = i;
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"\t收集到第："+ finalI +"颗龙珠");
                try {
                    barrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

### 8.3 `Semaphore`信号灯

在信号量上定义两种操作：

- **acquire(获取)**：当一个线程调用acquire操作时，它要么成功通过，获取信号量(信号量减1)，要么一直等待下去，直到有线程释放信号量或超时。
- **release(释放)**：实际上会将信号量的值加1，然后唤醒等待的线程。

> **信号量主要用于多个共享资源的互斥使用和对并发线程数的控制。**

案例代码如下：

```java
// 模拟抢车位案例
public class SemaphoreDemo {
    public static void main(String[] args) {
        // 模拟资源类，有3个空车位
        Semaphore semaphore = new Semaphore(3);
        for (int i = 1; i <= 6; i++) {
            new Thread(()->{
                try {
                    // 占用
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"\t抢到了车位");
                    TimeUnit.SECONDS.sleep(3);
                    // 释放
                    System.out.println(Thread.currentThread().getName()+"\t离开了车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

## 9. 可重入读写锁`ReentrantReadWriteLock`

当多个线程同时读取一个资源类没有任何问题，所以为了满足并发量，读取共享资源应该可以同时进行；但是，如果有一个线程想去写共享资源时，为了保证数据的一致性，就不应该有其他线程可以同时对该资源进行读写操作了。

- 代码案例，使用ReentrantReadWriteLock读写锁对不同的读写资源操作上不同的锁。

```java
// 自定义模拟一个缓存类
class MyCache{
    private volatile Map<String,Object> map = new HashMap<>();
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    
    public void put(String key,Object value){
        readWriteLock.writeLock().lock();
        try{
            System.out.println(Thread.currentThread().getName()+"\t>>>开始写入"+key);
            // 暂停一定毫秒时间
            try { TimeUnit.MILLISECONDS.sleep(300); } catch (InterruptedException e) { e.printStackTrace(); }
            map.put(key,value);
            System.out.println(Thread.currentThread().getName()+"\t>>>写入成功"+key);
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            readWriteLock.writeLock().unlock();
        }
    }
    public void get(String key){
        readWriteLock.readLock().lock();
        try{
            System.out.println(Thread.currentThread().getName()+"\t>>>开始获取数据"+key);
            // 暂停一定时间
            try { TimeUnit.MILLISECONDS.sleep(300); } catch (InterruptedException e) { e.printStackTrace(); }
            Object o = map.get(key);
            System.out.println(Thread.currentThread().getName()+"\t>>>成功获取数据"+o);
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            readWriteLock.readLock().unlock();
        }
    }
}
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache cache = new MyCache();
        // 模拟并发情况下5个线程写入
        for (int i = 1; i <= 5; i++) {
            final int finalI = i;
            new Thread(()->{
                cache.put(finalI +"", finalI +"");
            },String.valueOf(i)).start();
        }
        // 模拟并发情况下5个线程读取
        for (int i = 1; i <= 5; i++) {
            final int finalI = i;
            new Thread(()->{
                cache.get(finalI +"");
            },String.valueOf(i)).start();
        }
    }
}
```

> 读取资源应该共享读，写操作时应该排他写；以此保证数据的一致性。

## 10. 阻塞队列

阻塞队列`BlockingQueue`是一个队列，数据结构如下：

![阻塞队列](http://yanxuan.nosdn.127.net/9d12e170ba9bc52fa7cfbb7d6bed2d10.png)

阻塞队列主要具有这两点特性：

- 当队列为空时，从队列中**获取**元素的操作将会被阻塞
- 当队列为满时，从队列中**添加**元素的操作将会被阻塞

试图从空的队列中获取元素的线程将会被阻塞，直到其他线程往空的队列插入新的元素；试图向已满的队列中添加新元素的线程将会被阻塞，直到其他线程从队列中移除一个或多个元素或者完全清空，使队列变得空闲起来并后续新增。

> 简言之：<font color="red">空时不能消费，满时不能增加</font>

阻塞队列的用处：在多线程领域，所谓阻塞，就是在某些情况下会**挂起**线程(即阻塞)，一旦条件满足，被挂起的线程又会被自动**唤醒**。

concurrent包发布以前，在多线程环境下，我们都必须自己去控制很多细节，尤其还要兼顾效率和线程安全，这对编码带来不小的复杂度；阻塞队列`BlockingQueue`就可以帮我们很好的解决这些问题，让我们无需关心什么时候需要阻塞线程，什么时候需要唤醒线程。

### 10.1 架构介绍

阻塞队列所有已知实现类架构图如下：

![阻塞队列实现类](http://yanxuan.nosdn.127.net/fe2c909a5fdc12c30fe20081a75075af.png)

各种实现类分析：

1. `ArrayBlockingQueue`：**由数据结构组成的有界阻塞队列**
2. `LinkedBlockingQueue`：**由链表结构组成的有界阻塞队列(但大小默认值为`integer.MAX_VALUE`)**
3. `PriorityBlockingQueue`：支持优先级排序的无界阻塞队列
4. `DelayQueue`：使用优先级队列实现的延迟无界阻塞队列
5. `SynchronousQueue`：**不存储元素的阻塞队列，也即单元素队列**
6. `LinkedTransferQueue`：由链表组成的无界阻塞队列
7. `LinkedBlockingDeque`：由链表组成的双向阻塞队列

### 10.2 核心方法

| 方法类型 | 抛出异常  |  特殊值  |  阻塞  |        超时        |
| :------: | :-------: | :------: | :----: | :----------------: |
|   插入   |  add(e)   | offer(e) | put(e) | offer(e,time,unit) |
|   移除   | remove()  |  poll()  | take() |  poll(time,unit)   |
|   检查   | element() |  peek()  | 不可用 |       不可用       |

- 抛出异常
  - 当阻塞队列满时，再往队列中add插入元素会抛出`illegalStateException:Queue full`异常；
  - 当阻塞队列空时，再往队列里remove移除元素会抛出`NoSuchElementException`异常。
- 特殊值
  - 插入方法，成功true失败false
  - 移除方法，成功返回出队元素，无出队元素返回null
- 一直阻塞
  - 阻塞队列满时，生产者线程继续put元素，队列会一直阻塞生产者线程直到put成功或响应中断退出；
  - 阻塞队列空时，消费者线程继续take元素，队列会一直阻塞消费者线程直到队列可用。
- 超时退出
  - 当阻塞队列满时，队列会阻塞生产者线程一定时间，超过限时后生产者线程会退出。


