---
title: 数据结构概述和List/Set集合
date: 2019-4-25 23:59
categories: Java学习笔记 # 分类
tags: [Java,基本数据结构,Collections工具类]
description: 数据结构简述以及Java中的List/Set集合、Collections工具类
---


## 1. 数据结构
### 常见的数据结构
数据存储的常用结构有：栈、队列、数组、链表和红黑树。下面分别了解一下：

#### 栈

* **栈**：**stack**，又称堆栈，它是运算受限的线性表，限定仅在表尾进行插入和删除操作的线性表；又被称为后进先出(Last In First Out)的线性表，简称LIFO结构。

<!--more-->


简单的说：采用该结构的集合，对元素的存取有以下特点：

* 先进后出(即，先存进去的元素，要在后它存进去的元素依次取出后，才能取出该元素)。
* 栈的入口、出口都是栈的顶端位置。
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E5%A0%86%E6%A0%88-1556160250265.png)
* **压栈**：就是存元素。即、把元素存储到栈的顶端位置，栈中已有元素依次向栈底方向移动一个位置。
* **弹栈**：就是取元素。即、把栈的顶端位置元素取出，栈中已有元素依次向栈顶方向移动一个位置。

#### 队列
* **队列**：**queue**，简称队，它同堆栈一样，也是一种运算受限的线性表，只允许在一端进行插入操作，而在另一端进行删除操作的线性表，又被称为先进先出(First In First Out)的线性表，简称FIFO结构。
采用该结构的集合，对元素的存取有如下特点：
* 先进先出(即，先存进去的元素，会比后它进来的元素先出去)。
* 队列的入口、出口各占一侧。如下图示：<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E9%98%9F%E5%88%97%E5%9B%BE-1556161254994.bmp)

#### 数组
* **数组**：**Array**是有序的元素序列，数组是在内存中开辟一段连续的空间，并在此空间存放元素。
数据结构特点：<br>
* 查找元素快：通过索引，可以快速访问指定位置的元素
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E6%95%B0%E7%BB%84%E6%9F%A5%E8%AF%A2%E5%BF%AB-1556161542860.png)

* 增删元素慢
  * **指定索引位置增加元素**： 需要创建一个新数组，将指定新元素存储在指定索引位置，在把原数组元素根据索引，复制到新数组对应索引的位置。如下图：<br>![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E6%95%B0%E7%BB%84%E6%B7%BB%E5%8A%A0-1556161717343.png)
  * **指定索引位置删除元素**：需要创建一个新数组，把原数组元素根据索引，复制到新数组对应索引位置，原数组中指定索引位置元素不复制到新数组中。<br>![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E6%95%B0%E7%BB%84%E5%88%A0%E9%99%A4-1556161853599.png)

#### 链表
* **链表**：**linked list**由一系列结点node(链表中每一个元素称为结点)组成，结点可以在运行时i动态生成。每个结点包括两个部分：一个是存储数据元素的数据域，另一个是存储下一个结点地址的指针域(每一个节点包含指向下一个节点的指针)。链表结构有单向链表与双向链表，这里介绍的是单向链表。<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E5%8D%95%E9%93%BE%E8%A1%A8%E7%BB%93%E6%9E%84%E7%89%B9%E7%82%B9-1556162344765.png)<br>
采用该数据结构的集合，对元素的存取有如下的特点：
* 多个结点之间，通过地址进行连接。例如多个人手拉手，每个人使用自己的右手拉住下一个人的左手，依次类推，这样多个人就连在一起了。<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E5%8D%95%E9%93%BE%E8%A1%A8%E7%BB%93%E6%9E%84-1556162662545.png)<br>
* 查找元素慢：想查找某个元素，需要通过连接的节点，依次向后查找指定元素。
* 增删元素快：只需修改连接下个元素的地址即可。<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E5%A2%9E%E5%8A%A0%E7%BB%93%E7%82%B9-1556163144732.png)
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E5%88%A0%E9%99%A4%E7%BB%93%E7%82%B9-1556163150767.bmp)

#### 红黑树
* **二叉树**:**binary tree**,是每个节点不超过2的有序**树(tree)**。即每个节点最多只有两个分支(不存在分支度大于2的节点)的数结构。通常分支被称作"左子树"和"右子树"。二叉树的分支具有左右次序，不能颠倒。<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E4%BA%8C%E5%8F%89%E6%A0%91-1556164470030.bmp)
二叉树有一种比较有意思的叫做红黑树，红黑树本身就是一颗二叉查找树，将节点插入后，该树仍然是一颗二叉查找树。也意味着，树的键值仍然有序的。
红黑树的约束：
1. 节点可以是红色的或者黑色的
2. 根节点是黑色的
3. 叶子节点(特指空节点)是黑色的
4. 每个红色节点的子节点都是黑色的
5. 任何一个节点到其每一个叶子节点的所有路径上黑色节点数相同
红黑树特点：速度特别快,趋近平衡树,查找叶子元素最少和最多次数不多于二倍
[红黑树详解](https://github.com/CarpenterLee/JCFInternals/blob/master/markdown/5-TreeSet%20and%20TreeMap.md)

## 2. List集合
### 2.1 List接口
`java.util.List`接口继承自`Collection`接口，是单列集合的一个重要分支，习惯将实现了`List`接口的对象称为List集合。在List集合中允许出现重复的元素，所有的元素是以一种线性方式进行存储的，在程序中可以通过索引来访问集合中的指定元素。另外，List集合还有一个特定就是元素有序，即元素的存入顺序和取出顺序一致。

List接口特点：
1. 它是一个元素存取有序的集合。例如，存元素的顺序是11、22、33.那么集合中，元素的存储就是按照11、22、33的顺序完成的。
2. 它是一个带有索引的集合，通过索引就可以精确的操作集合中的元素(与数组的索引是一个道理)
3. 集合中可以有重复的元素，通过元素的equals方法，来比较是否为重复的元素。

### 2.2 List接口中常用的方法
List作为Collection集合的子接口，不但继承了Collection接口中的全部方法，而且还增加了一些根据元素索引来操作集合的特有方法。
* `public void add(int index, E element)`:将指定的元素，添加到该集合中的指定位置上。
* `public E get(int index)`: 返回集合中指定位置的元素
* `public E remove(int index)`:移除列表中指定位置的元素，返回的是被移除的元素
* `public E set(int index,E element)`:用指定元素替换集合中指定位置的元素，返回值是更新前的元素。

List集合特有的方法都是跟索引相关的，代码如下：
```java
import java.util.ArrayList;
import java.util.List;

public class Demo01List {
    public static void main(String[] args) {
        // 创建List集合对象
        List<String> list =  new ArrayList<String>();

        //往 尾部添加 元素
        list.add("小米");
        list.add("小天");
        list.add("小风");
        System.out.println(list);  // [小米, 小天, 小风]

        // add(int index,String s) 往指定位置添加元素
        list.add(1,"小洪");
        System.out.println(list); // [小米, 小洪, 小天, 小风]
        // remove(int index) 删除指定位置元素，返回被删除元素
        System.out.println("删除索引位置为2的元素"+list.remove(2)); // 删除索引位置为2的元素小天
        // set(int index, String s) 将指定位置的元素进行替换(修改)
        System.out.println(list.set(1,"小华")); // 返回的是被替换的元素， 小洪
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i)); // get(int index) 获取指定索引位置的元素
        }
    }
}
```

## 3. List的子类
`java.util.ArrayList`集合数据存储的结构是数组结构。元素增删慢，查找快，因日常开发中使用最多的功能为查询数据、遍历数据，所以ArrayList是常用的集合。

### 3.1 LinkedList集合
`java.util.LinkedList`集合数据存储的结构是链表结构。方便元素添加、删除的集合。
> LinkedList是一个双向链表，如下图就是一个双向链表。
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E5%8F%8C%E5%90%91%E9%93%BE%E8%A1%A8-1556175171141.png)

实际开发中对一个集合元素的添加与删除经常涉及到首尾操作，而LinkedList提供了大量首尾操作的方法。这些方法了解即可：
* `public void addFirst(E e)`:将指定元素插入此列表的开头。
* `public void addLast(E e)`:将指定元素添加到此列表的结尾。
* `public E getFirst()`:返回此列表的第一个元素。
* `public E getLast()`:返回此列表的最后一个元素。
* `public E removeFirst()`:移除并返回此列表的第一个元素。
* `public E removeLast()`:移除并返回此列表的最后一个元素。
* `public E pop()`:从此列表所表示的堆栈处弹出一个元素。
* `public void push(E e)`:将元素推入此列表所表示的堆栈。
* `public boolean isEmpty()`：如果列表不包含元素，则返回true。

LinkedList是List的子类，List中的方法LinkedList都是可以使用的。方法演示：
~~~java
import java.util.LinkedList;

public class Demo02LinkedList {
    public static void main(String[] args) {
        LinkedList<Integer> list = new LinkedList<>();
        list.addFirst(0);
        list.addLast(1);
        list.addLast(2);
        list.addLast(3);
        System.out.println(list); // [0,1,2,3]
        System.out.println("返回此列表第一个元素:"+list.getFirst()); // 0
        System.out.println("返回此列表最后一个元素:"+list.getLast()); // 3
        System.out.println("删除第一个元素并返回它:"+list.removeFirst()); // 0
        System.out.println("删除最后一个元素并返回它:"+list.removeLast()); // 3
        System.out.println(list.pop()); // 弹出，就是移除元素,栈在列表左侧
        list.push(3);
        System.out.println(list);
        System.out.println(list.isEmpty()); // 判断列表是否为空，空则返回true
    }
}
~~~

## 4. Set接口
`java.util.Set`接口和`java.util.List`接口一样，同样继承自`Collection`接口，它与`Collection`接口中的方法基本一致，并没有对该接口进行功能上的扩充，只是比`Collection`接口更加严格了。与`List`接口不同的是，`Set`接口中元素无序，并且有规则保证存入元素不出现重复。

`Set`集合有多个子类，这里主要介绍其中的`java.util.HashSet`、`java.util.LinkedHashSet`这两个集合。
> Tips：Set集合取出元素的方式可以采用：迭代器、增强for。

### 4.1 HashSet集合介绍
`java.util.HashSet`是`Set`接口的一个实现类，它存储的元素是不可重复的，并且元素都是无序的。它的底层实现其实是一个`java.util.HashMap`支持。

`HashSet`是根据对象的哈希值来确定元素在集合中的存储位置，因此具有良好的存取和查找性能。保证元素唯一性的方式依赖与：`hashCode`与`equals`方法。以下代码示意使用Set集合存储：
~~~java
import java.util.HashSet;

public class Demo03HashSet {
    public static void main(String[] args) {
        // 创建 Set集合
        HashSet<String> set = new HashSet<>();
        // 添加元素
        set.add(new String("cba"));
        set.add("abc");
        set.add("bac");
        set.add("cba");
        // 遍历
        for (String s:set){
            System.out.println(s); // cba,abc,bac 自动去重了
        }
    }
}
~~~

#### hashCode方法
hashCode方法返回对象的哈希值
```java
/*
    哈希值：是一个十进制的整数，由系统随机给出(就是对象的地址值，是一个逻辑地址，不是数据实际存储的物理地址)
    在Object类有一个方法，可以获取对象的哈希值
    int hashCode() 返回该对象的哈希码值
    hashCode()方法的源码：
        public native int hashCode();
        native:代办该方法调用的是本地操作系统的方法。
 */
public class DemoHashCode {
    public static void main(String[] args) {
        // Person类继承了Object类，所以可以使用Object类的hashCode方法
        Person p1 = new Person();
        System.out.println(p1.hashCode()); // 1072408673
        System.out.println(new Person().hashCode()); // 1531448569

        System.out.println("abc".hashCode()); // 96354
        // 以下两个为特殊情况，内容不同hash值相同
        System.out.println("重地".hashCode()); // 1179395
        System.out.println("通话".hashCode()); // 1179395
    }
}
```

Set集合去重原理：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/Set%E5%8E%BB%E9%87%8D%E5%8E%9F%E7%90%86-1556188205465.png)

### 4.2 HashSet集合存储数据的结构(哈希表)
#### 什么是哈希表？
在JDK 1.8之前，哈希表底层采用数组+链表实现，即使用链表处理冲突，同一hash值的链表都存储在一个链表里。但是当位于一个桶中的元素较多，即hash值相等的元素较多时，通过Key值依次查找的效率较低。而JDK 1.8中，哈希表存储采用数组+链表+红黑树实现，而链表长度超过阈值(8)时，将链表转换为红黑树，这样大大减少了查找时间。

简单的来说，哈希表是由数组+链表+红黑树(JDK1.8增加了红黑树部分)实现的，如下图示。
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E5%93%88%E5%B8%8C%E8%A1%A8-1556182995014.png)
哈希表存储流程图：
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/25/%E5%93%88%E5%B8%8C%E6%B5%81%E7%A8%8B%E5%9B%BE-1556183133856.png)

JDK1.8引入红黑树大程度优化了HashMap性能，而保证HashSet集合元素的唯一是根据对象的hashCode和equals方法决定的。如果我们往集合中存放自定义的对象，那么为了保证其唯一，就必须重写hashCode和equals方法建立当前对象的比较方式。

### 4.3 HashSet存储自定义类型元素
给HashSet存放自定义类型的元素时，需要重写对象中的hashCode和equals方法，建立自己的比较方式，才能保证HashCode集合中对象的唯一性。以下代码示例：
~~~java
import java.util.HashSet;
/*
    HashSet存储自定义类型元素
    set集合保存元素唯一：
        存储的元素(String,Integer,....student,Person),必须重写hashCode方法和equals方法
    要求：
        同名同年龄的人，视为同一人，只能存储一次
 */
public class Demo04MyHashSet {
    public static void main(String[] args) {
        // 创建hashSet集合，存储Person
        HashSet<Person> set = new HashSet<>();
        Person p1 = new Person("小米",11);
        Person p2 = new Person("小米",11);
        Person p3 = new Person("小米",12);
        System.out.println(p1.hashCode()); // 1072408673
        System.out.println(p2.hashCode()); // 1531448569
        // 因为hashcode值不一样，所以认为是不同元素，没有起到去重效果。
        System.out.println(p1==p2); // false 比较对象地址值不同，也认定是两个不同元素
        System.out.println(p1.equals(p2));
        set.add(p1);
        set.add(p2);
        set.add(p3);
        System.out.println(set);
    }
}
~~~

### 4.4 LinkedHashSet
HashSet可以保证元素唯一，可是元素存放进去是没有顺序的，如果要保证允许，可以使用HashSet的一个子类`java.util.LinkedHashSet`，它是链表和哈希表组合的一个数据存储结构。
示例如下：
```java
import java.util.Iterator;
import java.util.LinkedHashSet;
import java.util.Set;

public class DemoLinkedHashSet {
    public static void main(String[] args) {
        Set<String> set = new LinkedHashSet<>();
        set.add("aaa");
        set.add("ccc");
        set.add("bbb");
        set.add("ddd");
        for(String s:set){
            System.out.println(s);
        }
    }
}
```


### 4.5 可变参数
如果我们定义一个方法需要接受多个参数，并且多个参数类型一致，我们可以对其简化成如下格式：

```
修饰符 返回值类型 方法名(参数类型... 形参名){ }
```

**...**用在参数上，我们称之为可变参数，这个写法完全等价与

```
修饰符 返回值类型 方法名(参数类型[] 形参名){ }
```

后面的这种定义，在调用时必须传递数组，而前者可以直接传递数据。

同样是代表数组，但是在调用这个带有可变参数的方法时，不用创建数组(这就是简单方便之处),直接将数组中的元素作为实际参数进行传递。其实是编译成的class文件，先将这些元素封装到一个数组中，再进行传递。这些动作在编译.class文件时，就自动完成了。
代码示例如下：
~~~java
public class ChangeArgs {
    public static void main(String[] args) {
        int[] arr = {1,3,65,491,5};
        int sum = getSum(arr);
        System.out.println(sum);

        int sum2 = getSum(6,7,2,63,737); // 使用了可变参数写法，可以传递多个数据
        System.out.println(sum2);
    }
    // 原始写法
//    public static int getSum(int[] arr){
//        int sum = 0;
//        for(int a : arr){
//            sum += a;
//        }
//
//        return sum;
//    }
    // 使用可变参数写法
    public static int getSum(int... arr){
        int sum = 0;
        for (int a:arr){
            sum += a;
        }
        return sum;
    }
}
~~~

## 5. Collections
### 5.1 常用功能
* `java.utils.Collections`是集合工具类，用来对集合进行操作。部分方法如下：

- `public static <T> boolean addAll(Collection<T> c, T... elements)  `:往集合中添加一些元素。
- `public static void shuffle(List<?> list) 打乱顺序`:打乱集合顺序。
- `public static <T> void sort(List<T> list)`:将集合中元素按照默认规则排序。
- `public static <T> void sort(List<T> list，Comparator<? super T> )`:将集合中元素按照指定规则排序。

代码示例：
```java
public class DemoCollections {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>();
        // 原写法
//        list.add(1);
//        list.add(2);
//        list.add(43);
//        list.add(675);
        // 使用工具类添加元素
        Collections.addAll(list,64,267,1242,5);
        System.out.println(list); // [64, 267, 1242, 5]
        // 排序
        Collections.sort(list);
        System.out.println(list); // [5, 64, 267, 1242]
    }
}
```

### 5.2 Comparator比较器
`public static <T> void sort(List<T> list)`:将集合中元素按照默认规则排序。

代码示例：
~~~java
public class Demo02 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<String>();
        list.add("cba");
        list.add("aba");
        list.add("sba");
        list.add("nba");
        //排序方法
        Collections.sort(list);
        System.out.println(list); // [aba, cba, nba, sba]
    }
}
~~~

排序方式，简单的就是比较两个对象之间的大小，在Java中提供了两种比较实现的方式，一种是比较死板的采用`java.lang.Comparable`接口去实现，一种是灵活的当我需要做排序的时候在去选择的`java.util.Comparator`接口完成。

我们采用的`public static <T> void sort(List<T> list)`这个方法完成的排序，实际上要求了被排序的类型需要实现Comparable接口完成比较的功能，在String类型上如下：

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence { }
```
String类实现了这个接口，并完成了比较规则的定义，但是这样就把这种规则写死了，如果要实现一些别的操作，可以使用`public static <T> void sort(List<T> list，Comparator<? super T> )`方法灵活的完成。这个里面就涉及到了Comparator这个接口，位于位于java.util包下，排序是comparator能实现的功能之一,该接口代表一个比较器，比较器具有可比性！顾名思义就是做排序的，通俗地讲需要比较两个对象谁排在前谁排在后，那么比较的方法就是：

* ` public int compare(String o1, String o2)`：比较其两个参数的顺序。

> 两个对象比较的结果有三种：大于，等于，小于。

> 如果要按照升序排序，则o1 小于o2，返回（负数），相等返回0，01大于02返回（正数）

> 如果要按照降序排序，则o1 小于o2，返回（正数），相等返回0，01大于02返回（负数）

```java
public class Demo03 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<String>();
        list.add("cba");
        list.add("aba");
        list.add("sba");
        list.add("nba");
        //排序方法  按照第一个单词的降序
        Collections.sort(list, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o2.charAt(0) - o1.charAt(0);
            }
        });
        System.out.println(list); // [sba, nba, cba, aba]
    }
}
```

### 5.3 简述Comparable和Comparator两个接口的区别。

**Comparable**：强行对实现它的每个类的对象进行整体排序。这种排序被称为类的自然排序，类的compareTo方法被称为它的自然比较方法。只能在类中实现compareTo()一次，不能经常修改类的代码实现自己想要的排序。实现此接口的对象列表（和数组）可以通过Collections.sort（和Arrays.sort）进行自动排序，对象可以用作有序映射中的键或有序集合中的元素，无需指定比较器。

**Comparator**强行对某个对象进行整体排序。可以将Comparator 传递给sort方法（如Collections.sort或 Arrays.sort），从而允许在排序顺序上实现精确控制。还可以使用Comparator来控制某些数据结构（如有序set或有序映射）的顺序，或者为那些没有自然顺序的对象collection提供排序。

### 5.4 Exercise
创建一个学生类，存储到ArrayList集合中完成指定排序操作。
```java
// Student类
public class Student implements Comparable<Student>{
    private String name;
    private int age;

    public Student() {
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }


    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public int compareTo(Student o) {
        return this.age-o.age; // 升序
    }
}

// 测试类
public class Demo04 {
    public static void main(String[] args) {
        ArrayList<Student> list = new ArrayList<>();
        list.add(new Student("小明",11));
        list.add(new Student("小李",14));
        list.add(new Student("小张",17));
        list.add(new Student("小化",16));
        // 升序操作
        Collections.sort(list);// 报错，要求 该list中元素类型  必须实现比较器Comparable接口
        for (Student sd:list){
            System.out.println(sd);
        }
    }
}
```

必须实现Comparable接口，重写compareTo方法，才能进行升序操作。


### 今日总结
1. 基本数据结构简述
2. List集合的相关操作方法
3. LinkedList集合的方法
4. Set，HashSet，LinkedHashSet等集合的使用方法，自定义元素等
5. 哈希表定义
6. 可变参数的使用
7. Collections工具类的使用方法
8. Comparator比较器
9. Comparable和Comparator两个接口的区别