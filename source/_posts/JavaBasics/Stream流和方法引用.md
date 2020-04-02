---
title: Stream流的简单使用
date: 2019-5-15 22:40
categories: JavaBasics # 分类
tags: [Stream]
description: Java中的Stream流的使用以及各种方法引用方式
urlname: stream
---



## 1. Stream流

说到Stream便容易想到I/O流，实际上，在Java 8中，得益于Lambda所带来的函数式编程，引入了一个**全新的Stream概念**，用于解决已有集合类库既有的弊端。



<!--more-->



### 1.1 引言

#### 传统集合的多步遍历代码

几乎所有的集合(如`Collection`接口或`Map`接口等)都支持直接或间接的遍历操作。而当我们需要对集合中的元素进行操作的时候，除了必需的添加、删除、获取外，最典型的就是集合遍历。

```java
public class DemoForEach{
    public static void main(String[] agrs){
        List<String> list = new ArrayList<>();
        list.add("xiaozhang");
        list.add("xiaoming");
        list.add("xiaoli");
        list.add("xiaowang");
		
        for(String name:list){
            System.out.println(name);
        }
    }
}
```

#### 循环遍历的弊端

Java 8的Lambda让我们可以更加专注于**做什么**(What)，而不是**怎么做**(How)。现在我们看一下上例代码，可以发现：
* for循环的语法就是“**怎么做**”
* for循环的循环体才是"**做什么**"

为什么使用循环？因为要进行遍历。循环是遍历的唯一方式么？遍历是指对每一个元素逐一进行处理，**而并不是从第一个到最后一个顺次处理的循环**。前者是目的，后者是方式。

试想一下，如果希望对集合中的元素进行筛选过滤：

1. 将集合A根据条件一过滤为子集B
2. 然后在根据条件二过滤为子集C

在Java 8之前的做法可能为：

```java
public class DemoNormalFilter {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("张无忌");
        list.add("周芷若");
        list.add("赵敏");
        list.add("张强");
        list.add("张三丰");
		
        List<String> zhangList = new ArrayList<>();
        for (String name:list)
            if (name.startsWith("张")) // startsWith:测试此字符串是否以指定的前缀开始。
                zhangList.add(name);

        List<String> shortList = new ArrayList<>();
        for (String name:zhangList)
            if (name.length() == 3)
                shortList.add(name);

        for (String name:shortList)
            System.out.println(name);
    }
}
```

上述代码含有三个循环，作用不同：

1. 首先筛选所以姓张的人；
2. 然后筛选名字是三个字的人；
3. 最后对筛选结果进行打印输出。

每当我们需要对集合中的元素进行操作时，总是需要进行循环遍历。我们可以使用Lambda的衍生物Stream带来更加优雅的写法。

#### Stream的更优写法

下面来看使用Java 8的Stream API后的代码：

```java
public class DemoStreamFilter {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("张无忌");
        list.add("周芷若");
        list.add("赵敏");
        list.add("张强");
        list.add("张三丰");

        list.stream()
                .filter(s -> s.startsWith("张"))
                .filter(s -> s.length() == 3)
                .forEach(System.out::println);
    }
}
```

直接阅读代码的字面意思即可完美展示无关逻辑方式的语义：**获取流、过滤姓张、过滤长度为3、逐一打印**。代码中并没有体现使用线性循环或是其他任何算法进行遍历，我们真正要做的事情内容被更好地体现在代码中。

### 1.2 流式思想概述

**注意：请暂时忘记对传统IO流的固有印象**！

整体来看，流式思想类似于工厂车间的“**生产流水线**”

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/15/Stream01-1557888489833.jpg)

当需要对多个元素进行操作(特别是多步操作)时，考虑到性能及便利性，我们应该首先拼好一个"模型"步骤方案，然后再按照方案去执行它。

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/15/StreamModel-1557888739336.jpg)

这张图展示了过滤、映射、跳过、计数等多步操作，这是一种集合元素的处理方案，而方案就是一种"函数模型"。图中的每一个方框都是一个"流"，调用指定的方法，可以从一个流模型转换为另一个流模型。而最右侧的数字3是最终结果。

这里的`filter`、`map`、`skip`都是在对函数模型进行操作，集合元素并没有真正被处理。只有当终结方法`count`执行时，整个模型才会按照指定策略执行操作。而这得益于Lambda的延迟执行特性。

> 备注：“Stream流”其实是一个集合元素的**函数模型**，它并不是集合，也不是数据结构，其本身并不存储任何元素（或其地址值）。

Stream(流)是一个来自数据源的元素队列

* 元素是特定类型的对象，形成一个队列。Java中的Stream并不会存储元素，而是按需计算。
* **数据源**流的来源。可以是集合、数组等。

和以前的Collection操作不同，Stream操作还有两个基础的特征：

* **Pipelinling**：中间操作都会返回流对象本身。这样多个操作可以串联成一个管道，如同流式风格(fluent style)。这样做可以对操作进行优化，比如延迟执行(laziness)和短路(short-circuiting)。
* **内部迭代**：以前对集合遍历都是通过Iterator或者增强for的方式，显式的在集合外部进行迭代，这叫做外部迭代。Stream提供了内部迭代的方式，流可以直接调用遍历方法。

当使用一个流时，通常包括三个基本步骤：获取一个数据源(source) --> 数据转换 --> 执行操作获取想要的结果。每次转换原有Stream对象不变，返回一个新的Stream对象(可以有多次转换)，这就允许对其的操作可以像链条一样排列，变成一个管道。

#### 1.3 获取流

`java.util.stream.Stream<T>` 是Java 8 新加入的最常用的流接口(该接口不是函数式接口)。

获取流的方式：

* 所有的`Collection`集合都可以通过`stream`默认方法获取流；
* `Stream`接口的静态方法`of`可以获取数组对应的流。

#### 根据Collection获取流

首先，`java.util.Collection`接口中加入了default方法 `stream` 用来获取流，所以其所有实现类均可获取流。

```java
public class DemoGetStream {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        Stream<String> stream1 = list.stream();
        Set<String> set = new HashSet<>();
        Stream<String> stream2 = set.stream();
    }
}
```

#### 根据Map获取流

`java.util.Map`接口不是`Collection`的子接口，且其K-V数据结构不符合流元素的单一特征，所以获取对应的流需要分key、value或entry等情况：

```java
public class DemoMapGetStream {
    public static void main(String[] args) {
        Map<String,String> map = new HashMap<>();
        Stream<String> keyStream = map.keySet().stream(); // 获取键流
        Stream<String> valueStream = map.values().stream(); // 获取值流
        Stream<Map.Entry<String, String>> entryStream = map.entrySet().stream(); // 获取键值对流
    }
}
```

#### 根据数组获取流

如果使用的是数组，由于数组对象不可添加默认方法，所以`Stream`接口中提供了静态方法`of`,使用实例如下：

```java
public class DemoArrGetStream {
    public static void main(String[] args) {
        String[] arr = {"张","李","王"};
        Stream<String> stream = Stream.of(arr);
    }
}
// of方法的参数是一个可变参数，所以支持数组
```

### 1.4 常用方法

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/15/StreamMethod-1557892482395.jpg)

流模型的操作很丰富，下面介绍一些常用的API。这些方法可以被分成两种：

* **延迟方法**：返回值类型仍然是`Stream`接口本身类型的方法，因此支持链式调用。(除了终结方法外，其余方法均为延迟方法)
* **终结方法**：返回值类型不再是`Stream`接口自身类型的方法，因此不再支持类似`StringBuilder`那样的链式调用。这里介绍的终结方法包括`count`和`forEach`方法。

#### 逐一处理：forEach

​	虽然方法名字叫`forEach`，但是与for循环中的"for-each"昵称不同。方法签名如下：

```java
void forEach(Consumer<? super T> action);
```

​	该方法接收一个`Consumer`接口函数，会将每一个流元素交给该函数进行处理。

> Tips: 方法名和形参列表共同组成**方法签名**。

#### 复习Consumer接口

`java,util.function.Consumer<T>`接口是一个消费型接口。

`Consumer`接口中包含抽象方法`void accept(T t)`，意为消费一个指定泛型的数据。

#### 基本使用

```java
public class DemoStreamForEach {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("张", "李", "王");
        stream.forEach(name-> System.out.println(name));
    }
}
```

#### 过滤：filter
可以通过`filter`方法将一个流转换成另一个子集流。方法签名：

```java
Stream<T> filter(Predicate<? super T> predicate);
```

该接口接收一个`Predicate`函数式接口参数作为筛选条件。

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/15/StreamFilter-1557901616757.jpg)

#### 复习Predicate接口

`java.util.stream.Predicate`函数式接口，其中唯一的抽象方法为`boolean test(T t);`

该方法将会产生一个boolean值结果，代表指定条件是否满足。结果为true则保留元素，反之不保留。

#### 基本使用

~~~java
public class Demo2Filter {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("张三丰", "张翠山", "赵敏", "周芷若", "张无忌");
        // 对Stream流中的元素进行过滤，只要姓张的
        Stream<String> stream2 = stream.filter(name -> name.startsWith("张"));
        // 遍历stream2流
        stream2.forEach(name-> System.out.println(name));
    }
}
~~~

#### 映射：map

如果需要将流中的元素映射到另一个流中，可以使用`map`方法。方法签名：

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

该接口需要一个`Function`函数式接口参数，可以将当前流中的T类型数据转换为另一种R类型的流。

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/15/StreamMap-1557903369553.jpg)

#### 复习Function接口

此前我们已经学习过`java.util.stream.Function`函数式接口，其中唯一的抽象方法为：

```java
R apply(T t);
```

这可以将一种T类型转换成为R类型，而这种转换的动作，就称为”映射“。

#### 基本使用

Stream流中的`map`方法基本使用的代码如下：

```java
public class StreamMap {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("10", "15", "17");
        Stream<Integer> result = stream.map(str -> Integer.parseInt(str));
        result.forEach(str-> System.out.println(str));
    }
}
```

上述代码中，`map`方法的参数通过方法引用，将字符串类型转换为了int类型(并自动装箱为`Integer`类对象)。

#### 统计个数：count

正如集合`Collection`当中的`size`方法一样，流提供`count`方法来统计元素个数

该方法返回一个long值代表元素个数。基本使用如下：

```java
public class DemoStreamCount {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("1", "2", "3", "4");
        System.out.println(stream.count()); // 4
    }
}
```

#### 取用前几个：limit

`limit`方法可以对流进行截取，只取用前n个。方法签名：

```java
Stream<T> limit(long maxSize);
```

参数是一个long型，如果集合当前长度大于参数则进行截取；否则不进行操作。基本使用：

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/15/StreamLimit-1557906224811.jpg)
```java
public class DemoStreamLimit {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("a", "b", "c", "d", "e");
        Stream<String> limit = stream.limit(3); // 截取流的前两个元素
        limit.forEach(s-> System.out.println(s)); // a,b,c
    }
}
```

#### 跳过前几个：skip

如果希望跳过前几个元素，可以使用`skip`方法获取一个截取之后的新流：

```java
Stream<T> skip(long n);
```

如果流的当前长度大于n，则跳过前n个；否则将会得到一个长度为0的空流。

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/15/StreamSkip-1557906391858.jpg)

```java
public class DemoStreamSkip {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("1", "2", "3", "4", "5", "6");
        Stream<String> skip = stream.skip(3);
        skip.forEach(s-> System.out.println(s)); // 4,5,6
    }
}
```

#### 组合：concat

如果有两个流，希望合并成为一个流，那么可以使用`Stream`接口的静态方法`concat`。方法签名：

```java
static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b);
```

> 注：这是一个静态方法，与`java.lang.String`中的`concat`方法是不同的。

代码示例

```java
public class DemoStreamConcat {
    public static void main(String[] args) {
        Stream<String> h = Stream.of("hello");
        Stream<String> w = Stream.of("world!");
        Stream<String> result = Stream.concat(h, w);
        result.forEach(s-> System.out.println(s));
    }
}
```

### 1.5 练习：集合元素的处理

现在有两个`ArrayList`集合存储队伍当中多个成员姓名，要去使用循环依次进行以下若干步骤：

1. 第一个队伍只要名字为3个字的成员姓名；存储到一个新集合中。

2. 第一个队伍筛选之后只要前3个人；存储到一个新集合中。
3. 第二个队伍只要姓张的成员姓名；存储到一个新集合中。
4. 第二个队伍筛选之后不要前2个人；存储到一个新集合中。
5. 将两个队伍合并为一个队伍；存储到一个新集合中。
6. 根据姓名创建 Person 对象；存储到一个新集合中。
7. 打印整个队伍的Person对象信息。

代码实现(传统方式)

```java
public class ExerciseOldFor {
    public static void main(String[] args) {
        ArrayList<String> one = new ArrayList<>();
        // ....
        //队伍2
        ArrayList<String> two = new ArrayList<>();
        // ....
        // 1. 第一个队伍只要名字为3个字的成员姓名；存储到一个新集合中。
        List<String> oneA = new ArrayList<>();
        for (String name:one){
            if (name.length()==3){
                oneA.add(name);
            }
        }
        // 2. 第一个队伍筛选之后只要前3个人；存储到一个新集合中。
        List<String> oneB = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            oneB.add(oneA.get(i));
        }
        // 3. 第二个队伍只要姓张的成员姓名；存储到一个新集合中。
        List<String> twoA = new ArrayList<>();
        for (String name:two){
            if (name.startsWith("张")){
                twoA.add(name);
            }
        }
        // 4. 第二个队伍筛选之后不要前2个人；存储到一个新集合中。
        List<String> twoB = new ArrayList<>();
        for (int i = 2; i < twoA.size(); i++) {
            twoB.add(twoA.get(i));
        }
        // 5. 将两个队伍合并为一个队伍；存储到一个新集合中。
        List<String> totalNames = new ArrayList<>();
        totalNames.addAll(oneB);
        totalNames.addAll(twoB);
        // 6. 根据姓名创建 Person 对象；存储到一个新集合中。
        List<Person> personList = new ArrayList<>();
        for (String name:totalNames){
            personList.add(new Person(name));
        }
        // 7. 打印整个队伍的Person对象信息。
        for (Person p:personList){
            System.out.println(p);
        }
    }
}
```

Stream流方式实现

```java
public class ExerciseStream {
    public static void main(String[] args) {
        // 队伍1
        ArrayList<String> one = new ArrayList<>();
        one.add("迪丽热巴");
        one.add("宋远桥");
        one.add("苏星河");
        one.add("石破天");
        one.add("石中玉");
        one.add("老子");
        one.add("庄子");
        one.add("洪七公");
        //队伍2
        ArrayList<String> two = new ArrayList<>();
        two.add("古力娜扎");
        two.add("张无忌");
        two.add("赵丽颖");
        two.add("张三丰");
        two.add("尼古拉斯赵四");
        two.add("张天爱");
        two.add("张二狗");
        // 1. 第一个队伍只要名字为3个字的成员姓名；存储到一个新集合中。
        Stream<String> threeName = one.stream().filter(n -> n.length() == 3);
        // 2. 第一个队伍筛选之后只要前3个人；存储到一个新集合中。
        Stream<String> firstThreePeople = threeName.limit(3);
        // 3. 第二个队伍只要姓张的成员姓名；存储到一个新集合中。
        Stream<String> firstNameZhang = two.stream().filter(s -> s.startsWith("张"));
        // 4. 第二个队伍筛选之后不要前2个人；存储到一个新集合中。
        Stream<String> skipTwoPeople = firstNameZhang.skip(2);
        // 5. 将两个队伍合并为一个队伍；存储到一个新集合中。
        Stream<String> concat = Stream.concat(firstThreePeople, skipTwoPeople);
        // 6. 根据姓名创建 Person 对象；存储到一个新集合中。
        List<Person> people = new ArrayList<>();
        concat.forEach(s -> people.add(new Person(s)));
        // 7. 打印整个队伍的Person对象信息。
        people.forEach(p-> System.out.println(p));
        // 以上3步合并写法
        Stream.concat(firstThreePeople,skipTwoPeople).map(Person::new).forEach(System.out::println);
    }
}
```

## 2. 方法引用

在使用Lambda表达式时，我们实际上传递进去的代码就是一种解决方案：拿什么参数做什么操作。那么考虑一种情况：如果我们在Lambda中所指定的操作方案，已经有地方存在相同方案，那是否还有必要写重复逻辑？

### 2.1 冗余的Lambda场景

下面是一个简单的函数式接口以便应用Lambda表达式：

```java
public interface Printable{
    void print(String str);
}
```

在`Printable`接口当中唯一的抽象方法`print`接收一个字符串参数，目的就是为了打印显示它。那么下面通过Lambda来实现一下：

```java
public class DemoPrintImpl {
    private static void printString(String s,Printable data){
        data.print(s);
    }
    public static void main(String[] args) {
        printString("hello",s-> System.out.println(s));
    }
}
```

其中`printString`方法只管调用`Printable`接口的`print`方法，并不管该方法的具体实现逻辑会将字符串如何操作。而`main`方法通过Lambda表达式指定了函数式接口`Printable`的具体操作方案为：**拿到一个String数据后，在控制台中输出它。**

### 2.2 问题分析

这段代码的问题在于，对字符串进行控制台打印输出的操作方案，已经有了现成的实现，那就是`System.out`对象中的`println(String)`方法。既然Lambda希望做到的事情就是调用`println(String)`方法，那何必自己手动调用呢。

### 2.3 用方法引用改进代码

```java
public class Demo02PrintRef {
    private static void printString(Printable data){
        data.print("hello");
    }
    public static void main(String[] args) {
        //        printString(s-> System.out.println(s));
        /*
        分析：
            Lambda表达式的目的，打印参数传递的字符串
            把参数s，传递给了System.out对象，调用out对象中的方法println对字符串进行了输出
            注意：
                1. System.out对象是已经存在的
                2. println方法也是已经存在的
            所以我们可以使用方法引用来优化Lambda表达式
            可以使用System.out方法直接引用(调用)println方法
         */
        printString(System.out::println);
    }
}
```

其中双冒号`::`写法，这被称为“**方法引用**”，而双冒号是一种新的语法。

### 2.4 方法引用符

双冒号`::`为引用运算符，而它所在的表达式被称为**方法引用**。如果Lambda要表达的函数方案已经存在于某个方法的实现中，那么则可以通过双冒号来引用该方法作为Lambda的替代者。

#### 语义分析

如上例中，`System.out`对象中有一个重载的`println(String)`方法恰好就是我们所需要的。那么对于`printString`方法的函数式接口参数，对比下面两种写法，完全等效：

* Lambda表达式写法：`s -> System.out.println(s);`
* 方法引用写法：`System.out::println`

第一种语义是指：拿到参数之后经Lambda之手，继而传递给`System.out.println`方法去处理。

第二种等效写法的语义是指：直接让`System.out`中的`println`方法来取代Lambda。两种写法的执行效果完全一样，而第二种方法引用的写法复用了已有方案，更加简洁。

> 注：Lambda 中 传递的参数 一定是方法引用中 的那个方法可以接收的类型,否则会抛出异常

#### 推导与省略

如果使用Lambda，那么根据“**可推导即可省略**”的原则，无需指定参数类型，也无需指定重载形式----它们都将被自动推导，而如果使用方法引用，也是同样可以根据上下文进行推导。

函数式接口是Lambda的基础，而方法引用是Lambda的孪生兄弟。


### 2.5 通过对象名引用成员方法

这是最常见的一种用法，与上例相同。如果一个类中已经存在了一个成员方法：

```java
public class MethodRefObject {
    public void printUpperCase(String str){
        System.out.println(str.toUpperCase());
    }
}
```

函数式接口定义不变，那么当需要使用这个`printUpperCase`成员方法来替代`Printable`接口的Lambda时，已经具有了`MethodRefObject`类的对象实例，则可以通过对象名引用成员方法，如下：

```java
public class Demo04MethodRef {
    private static void printString(Printable p){
        p.print("hello");
    }
    public static void main(String[] args) {
        MethodRefObject obj = new MethodRefObject();
        printString(obj::printUpperCase); // HELLO
    }
}
```

### 2.6 通过类名称引用静态方法

由于在`java.lang.Math`类中已经存在了静态方法`abs`，所以当我们需要通过Lambda来调用该方法时，有两种写法。首先是函数式接口：

```java
@FunctionalInterface
public interface Calcable {
	int calc(int num);
}
```

两种写法调用：

```java
public class Demo05CalcLambda {
    private static void method(int num,Calcable ca){
        System.out.println(ca.calc(num));
    }
    public static void main(String[] args) {
        // Lambda表达式写法
        method(-10,n -> Math.abs(n)); // 10
        // 通过类名称引用静态方法
        method(-10,Math::abs); // 10
    }
}
```

上述例子中，两种写法是完全等效的：

- Lambda表达式：`n->Math.abs(n)`
- 方法引用：`Math::abs`

### 2.7 通过super引用成员方法

如果存在继承关系，当Lambda中需要出现super调用时，也可以使用方法引用进行替代。代码示例如下：

```java
// 首先是函数式接口：
@FunctionalInterface
public interface Greetable {
    void greet();
}
// 定义父类
public class Human {
    public void sayHello(){
        System.out.println("Hello!");
    }
}
// 定义子类继承父类
public class Man extends Human {
    // 子类重写父类sayHello方法
    @Override
    public void sayHello(){
        System.out.println("大家好，我是Man！");
    }
    // 定义方法method，参数传递Greetable接口
    public void method(Greetable g){
        g.greet();
    }
    public void show(){
        // 调用method方法，使用Lambda表达式
        method(()->new Human().sayHello()); // 创建父类human对象，调用父类的sayHello方法
        // 因为有子父类关系，所以存在一个关键字super，代表父类；所以我们可以直接使用super，调用父类的成员方法
        // 使用super关键字引用父类的成员方法
        method(super::sayHello);
    }
    public static void main(String[] args) {
        new Man().show(); // 调用show方法
    }
}

```

### 2.8 通过this引用成员方法

this代表当前对象，如果需要引用的方法就是当前类中的成员方法，那么可以使用`this::成员方法`的格式来使用方法引用。

```java
// 定义简单的函数式接口
@FunctionalInterface
public interface Richable {
    void buy();
}
// 定义使用类
public class Child {
    private void buyGame(){
        System.out.println("买了游戏");
    }
    private void computer(Richable r){
        r.buy();
    }
    public void soHappy(){
        // 使用lambda表达式
        computer(()->System.out.println("买了游戏"))；
        // 使用this关键字，调用本类中已经存在的方法
        computer(()->this.buyGame())
        // 使用方法引用
        computer(this::buyGame);
        // 上述三种方法完全等效
    }
     public static void main(String[] args) {
        new Child().soHappy(); // 买了游戏
    }
}
```

### 2.9 类的构造器引用

由于构造器的名称和类名完全一样，并不固定。所以构造器引用使用`类名称::new`的格式表示。

代码示例:

```java
// Person类
public class Person {
    private String name;

    public Person() {
    }

    public Person(String name) {
        this.name = name;
    }
	// ....
}
// 定义一个车间Person对象的函数式接口
@FunctionalInterface
public interface PersonBuilder {
    // 定义一个方法，根据传递的姓名，创建Person对象返回
    Person builderPerson(String name);
}
// 类的构造器引用
public class DemoBuilderPerson {
    // 定义一个方法，参数传递姓名和PersonBuilder接口，方法中通过姓名创建Person对象
    public static void printName(String name,PersonBuilder pb){
        Person person = pb.builderPerson(name);
        System.out.println(person.getName());
    }
    public static void main(String[] args) {
        // 调用printName方法，方法的参数PersonBuilder接口是函数式接口，故可以使用Lambda表达式
        printName("小张",name -> new Person(name));
        /*
            使用方法引用优化lambda表达式
            构造方法new Person(String name) 已知
            创建对象已知 new
            就可以使用Person引用new创建对象
         */
        printName("小明",Person::new); // 使用Person类的带参构造方法，通过传递的姓名创建对象
    }
}
```

### 2.10 数组的构造器引用

数组也是`Object`的子类对象，所以同样具有构造器，只是语法稍有不同。如果对应到Lambda的使用场景中时，示例代码如下：

```java
// 定义一个创建是数组的函数式接口
@FunctionalInterface
public interface ArrayBuilder {
    // 定义一个创建int类型数组的方法，参数传递数组的长度，返回创建好的int类型数组
    int[] builderArray(int length);
}
// 数组的构造器引用
public class DemoArrayBuilder {
    /*
        定义一个方法
        方法的参数传递创建数组的长度和ArrayBuilder接口
        方法内部根据传递的长度使用ArrayBuilder中的方法创建数组并返回
     */
    public static int[] createArray(int length,ArrayBuilder ab){
        return ab.builderArray(length);
    }

    public static void main(String[] args) {
        // 调用createArray方法，传递数组的长度和Lambda表达式
        int[] arr = createArray(10,(len)->new int[len]);
        System.out.println(arr.length); // 10
        /*
            使用方法引用优化lambda表达式
            已知创建的就是int[]数组
            数组的长度也是已知的
            就可以使用方法引用
            int[]引用new，根据参数传递的长度来创建数组
         */
        int[] arr2 = createArray(10,int[]::new);
        System.out.println(arr2.length); // 10
    }
}
```





## 今日目标

- [x] 能够理解流与集合相比的优点。让我们专注于做什么，而不是怎么做
- [x] 能够理解流的延迟执行特点。Stream可以按需计算，只有当终结方法执行时，整个模型才会按照指定策略执行操作。
- [x] 能够通过集合、映射或数组获取流。实现类.stream();Stream.of(数组)。
- [x] 能够掌握常用的流操作。forEach,filter,map,count,limit,skip,concat
- [x] 能够使用输出语句的方法引用。System.out::println？
- [ ] 能够通过4种方式使用方法引用
- [x] 能够使用类和数组的构造器引用。类名称::new;数组[]::new