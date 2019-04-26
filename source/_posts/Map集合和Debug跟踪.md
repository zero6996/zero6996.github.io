---
title: Map集合和Debug跟踪
date: 2019-4-26 23:59
categories: Java学习笔记 # 分类
tags: [Java]
description: Java中Map集合的特点及使用，Debug跟踪的使用
---

## 1. Map集合
生活中我们常会看到一种集合：IP地址与主机名，身份证号与个人等，这种一一对应的关系，叫做映射。Java中的`java.util.Map`接口是专门用来存放这种映射关系的集合类。

<!--more-->

### 1.1 概述
通过查看Map接口描述，发现Map接口下的集合与Collection接口下的集合，它们存储数据的形式不同，如下图。
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/26/Collection%E4%B8%8EMap-1556257061416.bmp)

* `Collection`中的集合，元素是孤立存在的，向集合中存储元素采用一个个元素的方式存储。
* `Map`中的集合，元素是成对存在的。每个元素由键值对组成，通过键可以找对应的值。
* `Collection`中的集合称为单列集合，`Map`中的集合称为双列集合。
* 需要注意的是，`Map`中的集合不能包含重复的键，值可以重复；每个键只能对应一个值。

### 1.2 Map常用子类
Map有多个子类，这里主要讲解常用的HashMap集合、LinkedHashMap集合。

* **HashMap<K,V>**：存储数据采用的哈希表结构，元素的存取顺序不一致。由于要保证键的唯一、不重复，需要重写键的hashCode()和equals()方法。
* **LinkedHashMap**：HashMap下有个子类LinkedHashMap，存储数据采用的哈希表结构+链表结构。通过链表结构可以保证元素的存取顺序一致；通过哈希表结构可以保证键的唯一性，需要重写键的hashCode和equals方法。

> Tips:Map接口中的集合都有两个泛型变量<K,V>,在使用时，需为两个泛型变量赋予数据类型。两个泛型变量<K,V>的数据类型可以相同，也可不同。

### 1.3 Map接口中的常用方法
Map接口中定义了多种方法，常用的如下：
* `public V put(K key, V value)`:  把指定的键与指定的值添加到Map集合中。
* `public V remove(Object key)`: 把指定的键 所对应的键值对元素 在Map集合中删除，返回被删除元素的值。
* `public V get(Object key)` 根据指定的键，在Map集合中获取对应的值。
* `boolean containsKey(Object key)  ` 判断集合中是否包含指定的键。
* `public Set<K> keySet()`: 获取Map集合中所有的键，存储到Set集合中。
* `public Set<Map.Entry<K,V>> entrySet()`: 获取到Map集合中所有的键值对对象的集合(Set集合)。

Map接口的方法示例：
~~~java
public class MapDemo {
    public static void main(String[] args) {
        // 创建Map对象
        HashMap<String,Integer> map = new HashMap<>();
        // put(键,值)添加元素
        System.out.println(map.put("小明",15)); // 如果添加的元素对键名没有在集合中，会返回一个null，并添加元素
        System.out.println(map.put("小明",11)); // 如果添加的元素对键名在集合中存在，则返回该键对应的值，然后替换为新值
        map.put("小张",18);
        map.put("李小",16);
        System.out.println(map); // {小明=15, 小张=18, 李小=16}
        // remove(键名)删除指定元素
        System.out.println(map.remove("李小"));
        System.out.println(map); // {小明=15, 小张=18}
        // 通过键名查询对应的值
        System.out.println(map.get("小明")); // 15
        System.out.println(map.get("小张")); // 18
    }
}
~~~

> Tips:
>
> 使用put方法时，若指定的键(key)在集合中没有，则表示没有这个键值对，返回null，并把指定键值添加到集合中。
>
>若指定的键(key)在集合中存在，则返回值为集合中键对应的值，并把指定键所对应的值，替换为新值。

### 1.4 Map集合遍历键找值的方式
遍历方式：即通过元素中的键名，获取键所对应的值。
```java
public class DemoGetValue {
    public static void main(String[] args) {
        HashMap<String,Integer> map = new HashMap<>();
        map.put("小明",16);
        map.put("小李",19);
        map.put("小张",17);
        map.put("小德",12);
        // 获取所以键，存储到set集合中
        Set<String> keys = map.keySet();
        for(String key:keys){
            System.out.println(key+"的年龄是："+map.get(key)+"岁");
        }
    }
}
```

### 1.5 Entry键值对对象
Map中存放的是两种对象，一种称为key(键)，一种称为value(值)，在Map中是一一对应关系，这一对对象又称做Map中的一个`Entry(项)`。`Entry`将键值对的对应关系封装成了对象。即键值对对象，这样我们遍历Map集合时，就可以从每一个键值对(Entry)对象中获取对应的键与对应的值。

Entry同样提供了获取对应键和对应值的方法：
* `public K getKey()`: 获取Entry对象中的键。
* `public V getValue()`:获取Entry对象中的值。

在Map集合中也提供了获取所以Entry对象的方法：
* `public Set<Map.Entry<K,V>> entrySet()`:获取Map集合中所有的键值对对象的集合(Set集合)。

### 1.6 Map集合遍历键值对方式
键值对方式：即通过集合中每个键值对(Entry)对象，获取键值对(Entry)对象中的键与值。
操作步骤：
1. 获取Map集合中，所以键值对(Entry)对象，以Set集合形式返回。方法提示：`entrySet()`。
2.  遍历包含键值对(Entry)对象的Set集合，得到每一个键值对(Entry)对象。
3.  通过键值对(Entry)对象，获取Entry对象中的键与值。  方法提示:`getkey() getValue()`   
~~~java
public class EntryDemo {
    public static void main(String[] args) {
        HashMap<String,Integer> map = new HashMap<>();
        map.put("小米",32);
        map.put("小花",25);
        map.put("小马",52);
        Set<Map.Entry<String,Integer>> entrys = map.entrySet();
        for (Map.Entry<String,Integer> entry:entrys){
            String key = entry.getKey();
            int value = entry.getValue();
            System.out.println(key+"的年龄是:"+value+"岁");
        }
    }
}
~~~

> Tips: Map集合不能直接使用迭代器或foreach进行遍历。但是转Set之后就可以使用了。

### 1.7 HashMap存储自定义类型键值
练习：每位学生（姓名，年龄）都有自己的家庭住址。那么，既然有对应关系，则将学生对象和家庭住址存储到map集合中。学生作为键, 家庭住址作为值。
```java
// 学生类
public class Student {
    private String name;
    private int age;
	// ... 省略get/set方法，初始化构造方法等
}
// 测试类
public class HashMapTest {
    public static void main(String[] args) {
        // 创建HashMap集合对象
        Map<Student, String> map = new HashMap<>();
        map.put(new Student("张三",25),"上海");
        map.put(new Student("李四",28),"北京");
        map.put(new Student("王五",20),"广州");
        map.put(new Student("赵六",19),"深圳");
        map.put(new Student("孙钱",24),"杭州");
        // 取出元素
        Set<Student> keys = map.keySet(); // 获取所以键名放入set中
        for (Student key:keys){
            System.out.println(key.toString() + "....." + map.get(key)); // 注意Student类要重写toString方法
        }
    }
}
```

### 1.8 LinkedHashMap
HashMap可以保证成对元素唯一性，且查询速度很快，可存入的顺序是无序的，如果要保证有序，可以使用`HashMap`下的一个子类`LinkedHashMap`，它是链表和哈希表组成的一个数据存储结构。
~~~java
public class LinkedHashMapDemo {
    public static void main(String[] args) {
        LinkedHashMap<String,Integer> linkMap = new LinkedHashMap<>();
        linkMap.put("小米",11);
        linkMap.put("小花",15);
        linkMap.put("小中",19);
        Set<Map.Entry<String,Integer>> entrys = linkMap.entrySet();
        for(Map.Entry<String,Integer> entry:entrys){
            System.out.println(entry.getKey() + "的年龄是：" + entry.getValue() + "岁");
        }
    }
}
// 如此就保证了存入元素的有序性
~~~

### 1.9 Map集合练习
计算一个字符串中每个字符出现次数。
```java
public class MapTest {
    public static void main(String[] args) {
        System.out.println("请输入一个字符串:");
        String line = new Scanner(System.in).nextLine();
        findChar(line);
    }
    public static void findChar(String line){
        // 创建一个集合，存储字符及出现次数
        HashMap<Character,Integer> map = new HashMap<>();
        for(int i = 0;i < line.length();i++){
            char c = line.charAt(i); // 返回指定索引处字符
            if(!map.containsKey(c)){ // 判断字符 是否在键集中,不在则执行下面语句
                map.put(c,1); // 第一次出现，添加该字符，并计数1
            }else{ // 则说明字符键名在键集中了，就是出现过
                Integer count = map.get(c); // 获取前一次值
                map.put(c,++count); // 再次存入，并更新值
            }
        }
        System.out.println(map);
    }
}
```

## 2. 知识点补充
### 2.1 JDK9对集合添加的优化
Java 9，添加了几种集合工厂方法，更方便创建少量元素的集合、map实例。新的List、Set、Map的静态工厂方法可以更方便地创建集合的不可变实例。示例如下：
```java
public class JDK9NewMethod {
    public static void main(String[] args) {
        Set<String> str = Set.of("a","b","c");
//        str1.add("d"); // 这里使用add方法，编译不会报错，执行会报错，因为是不可变集合。
        System.out.println(str);
        List<String> list = List.of("a","b","c");
        System.out.println(list);
        Map<String,Integer> map = Map.of("a",1,"b",2);
        System.out.println(map);
    }
}
```
注意事项：

> 1. of()方法只是Map,List,Set这三个接口的静态方法，其父类接口和子类实现并没有这类方法，如HashSet，ArrayList等。
> 2.返回的集合是不可变的。

### 2.2 Debug跟踪
#### 使用IDEA的断点调试功能，查看程序的运行过程
下图为调试窗口基本功能。<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/26/debug%E8%B0%83%E8%AF%95%E7%AA%97%E5%8F%A3%E5%8A%9F%E8%83%BD%E7%AE%80%E8%BF%B0-1556287135342.png)

## 3. 斗地主Map实现
### 需求分析：
1. 准备牌：
完成数字与纸牌的映射关系：
使用双列Map(HashMap)集合，完成一个数字与字符串纸牌的对应关系(相当于一个字典)。
2. 洗牌：
通过数字完成洗牌发牌
3. 发牌：
将每个人以及底牌设计为ArrayList<String>,将最后3张牌直接存放于底牌，剩余牌通过对3取模依次发牌。
存放的过程中要求数字大小与斗地主规则的大小对应。
将代表不同纸牌的数字分配给不同的玩家与底牌。
4. 看牌：
通过Map集合找到对应字符展示。
通过查询纸牌与数字的对应关系，由数字转成纸牌字符串再进行展示。<br>
![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/04/26/04_%E6%96%97%E5%9C%B0%E4%B8%BB%E6%A1%88%E4%BE%8B%E9%9C%80%E6%B1%82%E5%88%86%E6%9E%90-1556289337309.bmp)
~~~java
public class Poker {
    public static void main(String[] args) {
        // 准备牌
        List<String> colors = List.of("♥","♦","♠","♣");
        List<String> numbers = List.of("2","3","4","5","6","7","8","9","10","J","Q","K","A");
        HashMap<Integer,String> pokerMap = new HashMap<>();
        int cardID = 0;
        for (String color:colors){
            for (String num:numbers){
                pokerMap.put(cardID,color+num);
                cardID++;
            }
        }
        pokerMap.put(52,"小☺");
        pokerMap.put(53,"大☠");
//        System.out.println(pokerMap);
        ArrayList<Integer> cardKeys = new ArrayList<>();
        cardKeys.addAll(pokerMap.keySet());// 获取map的键名集合，并转换为List集合，方便打乱。
        // 打乱顺序，洗牌
        Collections.shuffle(cardKeys); // 牌编号集合
        // 创建3个玩家集合和底牌集合。
        ArrayList<String> bottomCard = new ArrayList<>(); // 底牌
        ArrayList<String> player1 = new ArrayList<>(); // 玩家1
        ArrayList<String> player2 = new ArrayList<>(); // 玩家2
        ArrayList<String> player3 = new ArrayList<>(); // 玩家3
        // 发牌的编号
        for (int i = 0;i < cardKeys.size();i++){
            if (i >= 51){
                bottomCard.add(pokerMap.get(cardKeys.get(i))); // 根据牌编号，获取对应在牌盒中的牌，放入底牌
            }else if (i % 3 == 0){
                player1.add(pokerMap.get(cardKeys.get(i))); // 根据牌编号，获取对应在牌盒中的牌，放入玩家1手中
            }else if (i % 3 == 1){
                player2.add(pokerMap.get(cardKeys.get(i))); // 根据牌编号，获取对应在牌盒中的牌，放入玩家2手中
            }else{
                player3.add(pokerMap.get(cardKeys.get(i))); // 根据牌编号，获取对应在牌盒中的牌，放入玩家3手中
            }
        }
        // 根据花色排序，后续可改进为根据数字排序
        Collections.sort(bottomCard);
        Collections.sort(player1);
        Collections.sort(player2);
        Collections.sort(player3);
        // 看牌
        System.out.println(player1);
        System.out.println(player2);
        System.out.println(player3);
        System.out.println(bottomCard);
    }
}
~~~


### 今日总结
1. Map集合中的子类和常用方法
2. Debug跟踪的使用
3. 使用Map实现斗地主案例