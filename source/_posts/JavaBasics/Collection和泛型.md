---
title: Collection集合和泛型
date: 2019-4-24 22:00
categories: JavaBasics # 分类
tags: [Java,集合]
description: Java中Collection集合和泛型的概念
urlname: collection
---


## 1. Collection集合
* **集合**:集合是java中提供的一种容器，可以用来存储多个数据。
集合和数组的区别
* 数组的长度是固定的。集合的长度是可变的。
* 数组中存储的是同一类型的元素，可以存储基本数据类型值。集合存储的都是对象，而且对象的类型可以不一致。当对象多的时候，使用集合进行存储。

<!--more-->

## 1.1 集合框架
JavaSE提供了满足各种需求的API，在使用这些API前，先了解其继承与接口操作架构，才能了解何时采用哪个类，以及类之间如何彼此合作，从而达到灵活应用。

集合按照其存储结构可以分成两大类，分别是单列集合`java.util.Collection`和双列集合`java.util.Map`，今天我们主要学习`Collction`集合，后续学习`Map`集合。

* **Collection**：单列集合类的根接口，用于存储一系列符合某种规则的元素，它有两个重要的子接口，分别是`java.util.List`和`java.util.Set`。其中，`List`的特点是元素有序、元素可重复。`Set`的特点是元素无序，而且不可重复。`List`接口的主要实现类有`java.util.ArrayList`和`java.util.LinkedList`，`Set`接口的主要实现类有`java.util.HashSet`和`java.util.TreeSet`。
如图描述集合类的继承体系：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/24/01_%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%E4%BB%8B%E7%BB%8D-1556083479913.bmp)

## 1.2 Collection常用功能
Coollection是所有单列集合的父接口，因此在Collection中定义了单列集合(List和Set)通用的一些方法，这些方法可用于操作所有的单列集合。方法如下：

* `public boolean add(E e)`：  把给定的对象添加到当前集合中 。
* `public void clear()` :清空集合中所有的元素。
* `public boolean remove(E e)`: 把给定的对象在当前集合中删除。
* `public boolean contains(E e)`: 判断当前集合中是否包含给定的对象。
* `public boolean isEmpty()`: 判断当前集合是否为空。
* `public int size()`: 返回集合中元素的个数。
* `public Object[] toArray()`: 把集合中的元素，存储到数组中。
代码示例：
~~~java
import java.util.ArrayList;
import java.util.Collection;

public class DemoCollection {
    public static void main(String[] args) {
        Collection<String> coll = new ArrayList<String>(); // 创建集合对象，多态写法
        // 使用添加功能，add(String s)；
        coll.add("小米");
        coll.add("小洪");
        coll.add("小丽");
        System.out.println(coll); // [小米, 小洪, 小丽]
        // 使用判断功能 contains(String s);
        System.out.println(coll.contains("小张")); // false
        System.out.println(coll.contains("小米")); // true
        System.out.println(coll.size()); // 返回集合中元素个数

        System.out.println(coll.remove("小洪"));
        System.out.println(coll);
        Object[] obj = coll.toArray(); // 转换成一个Object数组
        for (int i = 0; i < obj.length; i++) {
            System.out.println(obj[i]);
        }
        coll.clear(); // 清空集合
        System.out.println(coll.isEmpty()); // 判断集合是否为空
    }
}
~~~

## 2. Iterator迭代器
### 2.1 Iterator接口
`java.util.Iterator`接口主要用于迭代访问`Collection`中的元素，因此`Iterator`对象也被称为迭代器。
* `public Iterator iterator()`:获取集合对应的迭代器，用来遍历集合中的元素。
* **迭代**：即Collection集合元素的通用获取方式。在取元素之前先要判断集合中有没有元素，如果有就把这个元素取出来，然后继续判断。直到把集合中所有元素全部取出。这种取出方式专业术语称为迭代。

Iterator接口常用的方法如下：
* `public E next()`: 返回迭代的下一个元素。
* `public boolean hasNext()`: 如果仍有元素可以迭代，则返回true。
> E ：代表泛型的意思，指的是迭代出元素的数据类型
```java
public class DemoIterator {
    public static void main(String[] args) {
        Collection<String> coll = new ArrayList<>();
        // 添加元素
        coll.add("盖伦");
        coll.add("艾希");
        coll.add("剑圣");
        Iterator<String> it = coll.iterator(); // 每个集合对象都有自己的迭代器。
        while (it.hasNext()){//判断是否仍然有迭代元素
            System.out.println(it.next()); // 获取迭代出的元素
        }
    }
}
```

### 2.2 迭代器的实现原理
上面的案例已经完成了Iterator遍历集合的整个过程。当遍历集合时，首先通过调用it集合的iterator()方法获取迭代器对象，然后使用hasNext()方法判断集合中是否存在下一个元素，如果存在，则调用next()方法将元素取出来，反之则说明达到了集合末尾，停止遍历元素。

Iterator迭代器对象在遍历集合时，内部采用指针的方式来跟踪集合中的元素。如下图例演示Iterator对象迭代元素的过程：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/24/%E8%BF%AD%E4%BB%A3%E5%99%A8%E5%8E%9F%E7%90%86%E5%9B%BE%E8%A7%A3-1556091723158.png)

在调用Iterator的next方法之前，迭代器的索引位于第一个元素之前，不指向任何元素，当第一次调用迭代器的next方法后，迭代器的索引会向后移动一位，指向第一个元素并将该元素返回，当再次调用next方法时，迭代器的索引会指向第二个元素并将该元素返回，依此类推，直到hasNext方法返回false，表示到达了集合的末尾，终止对元素的遍历。

### 2.3 增强for
增强for循环(也称for each循环)是JDK 1.5以后出来的一个高级for循环，专门用来遍历数组和集合的。内部原理是一个Iterator迭代器，所以在遍历的过程中，不能对集合中的元素进行增删操作。格式如下：
~~~java
for(元素数据类型 变量：Collection集合or数组){
    // 操作代码
}
~~~
它能用于遍历Collection和数组。通常只进行遍历元素，不能在遍历过程中对集合进行增删操作。

#### 遍历示例
```java
import java.util.ArrayList;
import java.util.Collection;

public class DemoSuperFor {
    public static void main(String[] args) {
        int[] arr = {3,4,5,6,7};
        for(int a:arr){// a代表数组中的每个元素
            System.out.println(a);
        }
	// 进行元素遍历
        Collection<String> coll = new ArrayList<>();
        coll.add("蛮王");
        coll.add("剑圣");
        coll.add("赵信");
        // 使用增强for遍历
        for(String s:coll){ // 变量s代表被遍历到的集合园
            System.out.println(s);
        }
    }
}
```

> Tips:增强for循环必须有被遍历的目标。且目标只能是Collection或是数组。新式for仅仅用作遍历操作出现。

## 3. 泛型
集合中是可以存放任意对象的，只要把对象存储集合后，那么这时它们都会被提升成Object类型。当我们在取出每一个对象并进行相应操作时，必须采用类型转换。

观察一下代码：
~~~java
public class DemoGeneric {
    public static void main(String[] args) {
        Collection coll = new ArrayList();
        coll.add("abc");
        coll.add("itcast");
        coll.add(5); // 由于集合没有做泛型限定，任何类型都可以存放
        Iterator it = coll.iterator();
        while (it.hasNext()){
            System.out.println(((String)it.next()).length());
        }
    }
}
~~~
当程序运行到最后时会抛出**java.lang.ClassCastException**错误。因为集合中有一个int类型存在，所以发生了类型转换异常。Collection虽然可以存储各种对象，但实际上通常只存储同一类型对象。

* **泛型**：可以在类或方法中预支地使用未知的类型。

> Tips:一般在创建对象时，将未知的类型确定具体的类型。当没有指定泛型时，默认类型为Objec类型。

### 3.1 使用泛型的好处
* 将运行时期的ClassCastException，转移到了编译时期变成了编译失败。
* 避免了类型强转的麻烦。

代码示例：
```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.Iterator;

public class DemoGeneric02 {
    public static void main(String[] args) {
        Collection<String> list = new ArrayList<String>();
        list.add("abc");
        list.add("itcast");
        //list.add(5); // 编译不通过，类型不同
        Iterator<String> it = list.iterator();
        while(it.hasNext()){
            String str = it.next(); // 当使用了Iterator<String>控制元素类型后，就不用强转了。
            System.out.println(str.length());
        }
    }
}
```

> Tips:泛型是数据类型的一部分，我们将类名与泛型合并一起看作数据类型。

### 3.2 泛型的定义与使用
在集合中会大量使用到泛型，这里来完整地学习泛型知识。<br>
泛型，用来灵活地将数据类型应用到不同的类、方法、接口当中。将数据类型作为参数进行传递。

#### 定义和使用含有泛型的类
定义格式：
~~~
修饰符 class 类名<代表泛型的变量>{ }
~~~
例如，API中的ArrayList集合：
```java
class ArrayList<E>{
    public boolean add(E e){ }
    public E get(int index){ }
}
```
**在创建对象的时候确定泛型**
例如：`ArrayList<String> list = new ArrayList<String>();`

以下举例自定义泛型类
~~~java
public class DemoTestE {
    public static void main(String[] args) {
        // 创建一个泛型为String的类
        DemoE<String> mye = new DemoE<String>();
        // 调用set方法
        mye.setMvp("我是一个字符串");
        // 调用get方法
        String str = mye.getMvp();
        System.out.println(str);

        // 创建一个泛型为Integer的类
        DemoE<Integer> my2 = new DemoE<>();
        my2.setMvp(123);
        System.out.println(my2.getMvp());
    }
}
~~~

#### 含有泛型的方法
格式： `修饰符<代表泛型的变量> 返回值类型 方法名(参数){ }`
代码举例：
```java
// 自定义一个含有泛型的方法
public class MyGenericMethod {
    public <MVP> void show(MVP mvp){
        System.out.println(mvp.getClass()); // 获取当前类对象
    }
    public <MVP> MVP show2(MVP mvp){
        return mvp;
    }
}
// 使用该类方法，任意输入数据类型
public class GenericMethodDemo {
    public static void main(String[] args) {
        // 创建对象
        MyGenericMethod mgm = new MyGenericMethod();
        // 调用方法
        mgm.show("aaa");
        mgm.show(123);
        mgm.show('A');
        mgm.show(1.30);
    }
}
```

#### 含有泛型的接口
格式：`修饰符 interface名称<代表泛型的变量>{ }`
使用方法基本和类的一致，代码如下：
~~~java
// 含有泛型的接口类
public interface MyGenericInterface<E> {
    public abstract void add(E e);
    public abstract E getE();
}
// 使用格式1： 定义类时确定泛型的类型
public class MyGenericInterfaceImpl implements MyGenericInterface<String> {
    @Override
    public void add(String e){
        System.out.println("接口使用泛型");
    }
    @Override
    public String getE(){
        return null;
    }
}
// 此时，泛型E的是就是String类型
// 使用格式2：始终不确定泛型的类型，直到创建对象时，确定泛型的类型
public class MyImpl2<E> implements MyGenericInterface<E>{

    @Override
    public void add(E e) {
        System.out.println("在创建对象时会确认泛型的类型");
    }

    @Override
    public E getE() {
        return null;
    }
}
// 测试
public class GenericTest {
    public static void main(String[] args) {
        MyImpl2<String> my = new MyImpl2<String>(); // 在创建对象时指定泛型类型
        my.add("aa");
        MyImpl2<Integer> my2 = new MyImpl2<Integer>(); 
        my2.add(1);
    }
}
~~~

### 3.4 泛型通配符
当使用泛型类或者接口时，传递的数据中，泛型类型不确定，可以通过通配符<?>表示。一旦使用泛型的通配符后，只能使用Object类中的共性方法，集合中元素自身方法无法使用。

#### 通配符的基本使用
泛型的通配符：不知道使用什么类型来接收时，可以使用？(？表示未知通配符)。

```java
import java.util.ArrayList;
import java.util.Collection;

public class TestWildCard {
    public static void main(String[] args) {
        Collection<Integer> list = new ArrayList<Integer>();
        list.add(123);
        getElement(list); // 可以接受任意类型数据
        Collection<String> str = new ArrayList<String>();
        str.add("str");
        getElement(str);
    }
    public static void getElement(Collection<?> coll){ // 不知道会传入什么类型数据，可以用？
        System.out.println(coll);
    }
}
```

> Tips:泛型不存在继承关系，例如Collection<Object> list = new ArrayList<String>();这种是错误的。

#### 通配符高级使用--受限泛型
前面设置泛型的时候，实际上是可以任意设置的，只要是类就可以设置。但是在Java的泛型中可以指定一个泛型**上限**和**下限**。

**泛型的上限**：

* **格式**：`类型名称<？extends类>对象名称`
* **意义**：`只能接收该类型及其子类`

**泛型的下限**：

- **格式**:`类型名称<?super类>对象名称`
- **意义**：`只能接收该类型及其父类型`

比如：现已知Object，String类，Number类，Integer类，其中Number是Integer的父类。

~~~java
/*
泛型的上限限定：? extends E 代表使用的泛型只能是E类型的子类/本身
泛型的下线限定：? super E   代表使用的泛型只能是E类型的父类/本身
 */
import java.util.ArrayList;
import java.util.Collection;

public class Demo01 {
    public static void main(String[] args) {
        Collection<Integer> ints = new ArrayList<>();
        Collection<String> str = new ArrayList<>();
        Collection<Number> num = new ArrayList<>();
        Collection<Object> obj = new ArrayList<>();

        getElement1(ints);
        getElement1(str); // 报错 不是Number子类或本身
        getElement1(num);
        getElement1(obj); // 报错 是Number父类，不符合

        getElement2(ints); // 报错
        getElement2(str); // 报错
        getElement2(num);
        getElement2(obj);
        /*
            类与类之间的继承关系
            Integer extends Number extends Object
            String extends Object
         */
    }
    // 泛型的上限：此时的泛型?，必须是Number类型或者Number类型的子类
    public static void getElement1(Collection<? extends Number> coll){}
    // 泛型的下限：此时的泛型?，必须是Number类型或者Number类型的父类
    public static void getElement2(Collection<? super Number> coll){}
}
~~~

## 4 综合案例
按照斗地主的规则，完成洗牌发牌的动作。
具体规则：使用54张牌打乱顺序，三个玩家参与游戏，三人交替摸牌，每人17张牌，最后三张留底牌。

### 4.1 案例分析
* 准备牌：

  牌可以设计为一个ArrayList<String>,每个字符串为一张牌。
  每张牌由花色数字两部分组成，我们可以使用花色集合与数字集合嵌套迭代完成每张牌的组装。
  牌由Collection类的shuffle方法进行随机排序。

* 发牌

  将每个人以底牌设计为ArrayList<String>，将最后3张牌直接存于底牌，剩余牌通过对3取模依次发牌。

* 看牌

  直接打印每个集合。

### 代码实现
~~~java
import java.util.ArrayList;
        import java.util.Collections;

/*
1. 准备牌：54张牌，存到一个集合中
   特殊牌：大小王
   其余52张牌：
     定义一个数组/集合，存储4种花色
     定义一个数组/集合，存储13个序号:2AKQJ....3
   循环嵌套遍历两个数组/集合，组装52张牌

2. 洗牌
    使用集合工具类Collection的方法
    static void shuffle(List<?> list) 使用指定的随机源对指定列表进行置换
    会随机打乱集合中的元素顺序。

3. 发牌
    要求：1人17张牌，剩余3张作为底牌，1人以张轮流发牌，集合索引(0-53)%3
    定义4个集合，存储3位玩家的牌和底牌
    索引%2，有两值(0,1) 0%2=0,1%2=1,2%2=0,3%2=1
    索引%3，有三值(0,1,2) 0%3=0,1%3=1,2%3=2,3%3=0

4. 看牌
    直接打印集合即可。

 */
public class Poker {
    public static void main(String[] args) {
        ArrayList<String> flowerColor = new ArrayList<>(); // 花色
        ArrayList<String> cardNumber = new ArrayList<>(); // 牌数字
        ArrayList<String> pokerBox = new ArrayList<>(); // 牌盒
        // 添加花色
        flowerColor.add("♥");
        flowerColor.add("♦");
        flowerColor.add("♠");
        flowerColor.add("♣");
        // 添加牌数字
        for (int i = 2; i <= 10; i++) {
            cardNumber.add(i + ""); // 将int数字字符串化
        }
        cardNumber.add("J");
        cardNumber.add("Q");
        cardNumber.add("K");
        cardNumber.add("A");
        // 组装牌
        for (String color:flowerColor){
            for(String numbers:cardNumber){
                pokerBox.add(color+numbers);
//                System.out.println(color+numbers);
            }
        }
        // 大小王
        pokerBox.add("小☺");
        pokerBox.add("大☠");
//        System.out.println(pokerBox);
        // Collection类的shuffle方法进行随机排序。
        Collections.shuffle(pokerBox);

        // 创建玩家集合和底牌集合
        ArrayList<String> player1 = new ArrayList<String>();
        ArrayList<String> player2 = new ArrayList<String>();
        ArrayList<String> player3 = new ArrayList<String>();
        ArrayList<String> dipai = new ArrayList<String>();

        // 遍历牌盒
        for (int i = 0; i < pokerBox.size(); i++) {
            String card = pokerBox.get(i); // 获取一张牌
            // 留三张底牌，放入底牌盒中
            if(i >= 51){
                dipai.add(card);
            }else{
                if(i%3==0){
                    player1.add(card);
                }else if(i%3==1){
                    player2.add(card);
                }else {
                    player3.add(card);
                }
            }

        }
        System.out.println(player1);
        System.out.println(player2);
        System.out.println(player3);

    }
}
~~~


