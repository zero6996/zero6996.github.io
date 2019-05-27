---
title: Junit单元测试，反射和注解
date: 2019-5-16 23:59
categories: Java学习笔记 # 分类
tags: [Java,Junit,反射,注解]
description: Junit单元测试的基本使用，反射概述和注解
---

## 1. Junit单元测试
#### 测试的分类：

1. 黑盒测试：不需要写代码，给输入值，看输出值是否达到预期值。
2. 白盒测试：要写代码。关注程序具体的执行流程。

<!--more-->

#### Junit使用步骤

1. 创建测试包(创建一个与被测试包同级的文件夹)
2. 包内定义测试类(命名规范：被测试类名称+Test)
3. 在测试类里面定义测试方法，如下图所示：

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/16/Junit01-1557992450228.jpg)

**参考资料**：[Junit5基本使用](https://www.ibm.com/developerworks/cn/java/j-introducing-junit5-part1-jupiter-api/index.html)

#### 部分断言方法

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/16/AssertMethod-1557993543134.jpg)

#### 代码示例：

```java
// 定义计算类
public class Calculator {
    public int add(int a, int b){
//        int i = 3/0;
        return a - b; // 故意写错，以测试accertEquals方法
    }
    public int sub(int a, int b){
        return a - b;
    }
}
// 定义测试类
import Junit.Calculator;
import org.junit.jupiter.api.Test; // Junit5改进后，所有方法都在jupiter.api下
import org.junit.jupiter.api.Assertions; 

public class CalculatorTest {
    @BeforeAll // 一般用于资源的申请
    public void init(){
        System.out.println("所有测试方法执行前自动执行");
    }
    @AfterAll // 用于资源的释放
    public void close(){
        System.out.println("所有测试方法结束后自动执行");
    }
    @Test
    public void testAdd(){
        Calculator c = new Calculator();
        int add = c.add(1, 2);
        // 断言，我断言这个结果是3
        Assertions.assertEquals(3,add); // 不通过，报错，并显示红色
    }
    @Test
    public void testSub(){
        Calculator c = new Calculator();
        int sub = c.sub(1, 2);
        Assertions.assertEquals(-1,sub); // 通过，显示绿色
    }
}
```



## 2. 反射：框架设计的灵魂

框架被称为半成品软件。我们可以在框架的基础上进行软件开发，简化编码。

**反射**：将类的各个组成部分封装为其他对象，这就是反射机制

反射的好处：

1. 可以在程序运行过程中，操作这些对象。
2. 可以解耦，提高程序的可扩展性。

### 2.1 获取Class对象的方式

1. **Class.forName("全类名")**：将字节码文件加载进内存，返回Class对象。多用于配置文件，将类名定义在配置文件中。来读取文件，加载类。
2. **类名.class**：通过类名的属性class获取。多用于参数的传递。
3. **对象.getClass()**：`getClass()`方法在`Object`类中定义，所以可以直接使用。多用于对象获取字节码的方式。

#### 代码示例

~~~java
public class ReflectDemoGetClass {
    public static void main(String[] args) throws Exception {
        // 1.Class.forName("全类名")：将字节码文件加载进内存，返回Class对象。
        Class cls1 = Class.forName("Demo5_16.reflect.Person");
        System.out.println(cls1);
        // 2.类名.class：通过类名的属性class获取
        Class cls2 = Person.class;
        System.out.println(cls2);
        // 3. 对象.getClass()：`getClass()`方法在`Object`类中定义，所以可以直接使用
        Class cls3 = new Person().getClass();
        System.out.println(cls3);

        // 使用==比较对象内存地址
        System.out.println(cls1 == cls2); // true
        System.out.println(cls2 == cls3); // true
    }
}
~~~

>Tips ：同一字节码文件(*.class)在一次程序运行过程中，只会被加载一次，不论通过哪一种方式获取的Class对象都是同一个。

![Java代码在计算机中的三个阶段](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/16/Java%E4%BB%A3%E7%A0%81%E7%9A%84%E4%B8%89%E4%B8%AA%E9%98%B6%E6%AE%B5-1558017486873.bmp)

### 2.2 Class对象的功能：

1. 获取成员变量
```java
// 四种方式获取成员变量
Filed[] getFields() // 获取所有public修饰的成员变量
Filed getFiled(String name) // 获取public修饰的指定名称的成员变量
Field[] getDeclaredFields() // 获取所有的成员变量,不考虑修饰符
Filed getDeclaredField(String name) // 获取指定名称的成员变量,如果不是public修饰的成员变量，可以使用暴力反射获取
// 示例代码如下:
// Person类
public class Person {
    private String name;
    private int age;

    public String a;
    protected String b;
    public String c;
    private String d;
	// 忽略get\set等方法    
}
public class ReflectDemo01 {
    public static void main(String[] args) throws Exception{
        // 获取Person的Class对象
        Class<Person> personClass = Person.class;
        // 获取成员变量
        // 1. Field[] getFields() 获取所有public修饰的成员变量
        Field[] fields = personClass.getFields();
        for (Field f:fields)
            System.out.println(f); // public java.lang.String reflect.Person.a
        // 2.Field getFiled(String name) // 获取public修饰的指定名称的成员变量
        Field a = personClass.getField("a");
        // 获取成员变量a的值
        Person p = new Person();
        Object value = a.get(p);
        System.out.println(value); // null
        // 设置a的值
        a.set(p,"小明");
        System.out.println(p);
        // 3. Field[] getDeclaredFields() : 获取所有的成员变量,不考虑修饰符
        Field[] declaredFields = personClass.getDeclaredFields();
        for (Field f:declaredFields)
            System.out.println(f);
        // 4. Filed getDeclaredField(String name) : 获取指定名称的成员变量
        Field d = personClass.getDeclaredField("d");
        // 忽略访问权限修饰符的安全检查
        d.setAccessible(true); // 暴力反射
        System.out.println(d.get(p)); // null
    }
}
```
2. 获取构造方法
```java
// 四种方式获取构造方法
Constructor<?>[] getConstructors() // 获取所有的public修饰的构造方法
Constructor<T> getConstructor(类<?>... parameterTypes) // 获取指定的public修饰的构造方法
Constructor<T> getDeclaredConstructor(类<?>... parameterTypes) // 获取所有的构造方法，忽略修饰符
Constructor<?>[] getDeclaredConstructor() // 获取指定名称的构造方法，非public修饰的可使用暴力反射获取
// 代码举例
public class ReflectDemo02 {
    public static void main(String[] args) throws Exception{
        // 获取Person的Class对象
        Class<Person> personClass = Person.class;
        // 1. Constructor<T> getConstructor(类<?>... parameterTypes)
        // 获取构造器
        Constructor<Person> constructor = personClass.getConstructor(String.class, int.class);
//        System.out.println(constructor); // public reflect.Person(java.lang.String,int)
        // 使用构造器创建对象实例
        Person person = constructor.newInstance("小明",22);
        System.out.println(person);
        // 如果使用空参数构造方法创建对象，操作可以简化为如下：
        Person o = personClass.newInstance(); // 不推荐使用
        System.out.println(o);
    }
}
```
3. 获取成员方法
```java
// 四种方式获取成员方法
Method[] getMethods() // 获取所有public修饰的方法
Method getMethod(String name,类<?>... parameterTypes) // 获取指定名称的public修饰的方法
Method[] getDeclaredMethods() // 获取所有的成员方法，不考虑修饰符
Method getDeclaredMethod(String name,类<?>... parameterTypes) // // 获取指定名称的成员方法,如果不是public修饰的成员变量，可以使用暴力反射获取
// 代码示例
public class ReflectDemo03 {
    public static void main(String[] args) throws Exception{
        // 获取Person的Class对象
        Class<Person> personClass = Person.class;
        // 1. Method getMethod(String name,类<?>... parameterTypes): 获取指定名称的public修饰的方法
        Method eat = personClass.getMethod("eat");
        // 执行无参方法
        eat.invoke(new Person());
        // 执行有参方法，需传递参数列表
        personClass.getMethod("eat",String.class).invoke(new Person(),"food");
        // 2. Method[] getDeclaredMethods(): 获取所有public修饰的方法
        Method[] methods = personClass.getMethods();
        for (Method method:methods)
            System.out.println(method);
        // 3. Method[] getDeclaredMethods(): 获取所有的成员方法，不考虑修饰符
        Method[] declaredMethods = personClass.getDeclaredMethods();
        for (Method method:declaredMethods) {
            System.out.println(method);
            System.out.println(method.getName()); // 获取方法名称
        }
        // 获取类名称
        System.out.println(personClass.getName()); // reflect.Person
    }
}
```
4. 获取类名 ：`String getName()`

#### Field：成员变量
1. 设置值：`void set(Object obj,Object value)`
2. 获取值：`get(Object obj)`
3. 忽略访问权限修饰符的安全检查(又称**暴力反射**)：`setAccessible(true)`

#### Constructor：构造方法

创建对象：`T newInstance(Object... initargs)`

> 如果使用空参数构造方法创建对象，操作可以简化为：直接使用Class对象的newInstance方法

#### Method：方法对象

执行方法：`Object invoke(Object obj, Object... args)`

获取方法名称：`String getName()`

 

### 2.3 练习案例

创建一个"框架"(类)，不改变该类的任何代码的前提下，可以帮我们创建任意类的对象，并执行其中的任意方法。

#### 实现步骤：

1. 将需要创建的对象的全类名和需要执行的方法定义在配置文件中
2. 在程序中加载读取配置文件
3. 使用反射技术来加载类文件进内存
4. 创建对象
5. 执行方法

#### 代码实例

```java
public class MyReflectFrame {
    public static void main(String[] args) throws Exception {
        // 可以创建任意类的对象，可以执行任意方法
        // 前提：不能改变该类的任何代码的前提下，可以帮我们创建任意类的对象，并且执行其中任意方法
//        new Student().sleep();
        // 1. 加载配置文件
        // 1.1 创建Properties对象
        Properties pro = new Properties();
        // 1.2 加载配置文件，转换一个集合
        // 1.2.1 获取class目录下的配置文件
        ClassLoader classLoader = MyReflectFrame.class.getClassLoader();// 获取这个字节码文件的类加载器
        InputStream is = classLoader.getResourceAsStream("D:\\IDEA_WorkSpace\\JavaWeb\\src\\Demo5_16\\pro.properties"); // 返回用于读取指定资源的输入流
        pro.load(is); // NullPointerException: inStream parameter is null!!! (后续回来解决)

        // 2. 获取配置文件中定义的数据
        String className = pro.getProperty("className"); // 获取配置文件中的类名
        String methodName = pro.getProperty("methodName"); // 获取配置文件中的方法名

        // 3.加载该类进内存
        Class cls = Class.forName(className);
        // 4. 创建对象
        Object obj = cls.newInstance();
        // 5. 获取方法对象
        Method method = cls.getMethod(methodName);
        // 6. 执行方法
        method.invoke(obj);
    }
}
```



## 3. 注解

**注解**：说明程序的，给计算机看的。

注释：用文字描述程序的，给程序员看的。

#### 定义

注解(Annotation)，也叫元数据。一种代码级别的说明。它是JDK1.5及以后版本引入的一个特性，与类、接口、枚举是在同一个层次。它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明，注释。

#### 概念描述

* JDK1.5 之后的新特性
* 代码级别的说明，用以说明程序的
* 使用注解：`@注解名称`

**作用分类**：

1. 编写文档：通过代码里标识的注解生成文档[生成文档doc文档]
2. 代码分析：通过代码里标识的注解对代码进行分析[使用反射]
3. 编译检查：通过代码里标识的注解让编译器能够实现基本的编译检查[Override]

### 3.1 JDK中预定义的一些注解

* `@Override`：检测被该注解标注的方法是否是继承自父类(接口)的
* `@Deprecated`：该注解标注的内容，表示已过时
* `@SuppressWarnings`：压制警告。一般传递参数all，`@SuppressWarnings("all")`

#### 代码示例

~~~java
@SuppressWarnings("all") // 压制全部警告
public class AnnoDemo2 {
    @Override
    public String toString(){
        return super.toString();
    }
    @Deprecated
    public void show1(){
        //有缺陷
    }
    public void show2(){
        // 优化版show2
    }
    public void demo(){
        show1();
    }
}
~~~

### 3.2 自定义注解

**基本定义格式**

```java
public @interface 注解名称{
    属性列表;
}
```

其本质**就是一个接口**，该接口默认继承Annotation接口`public interface MyAnno extends java.lang.annotation.Annotation{...}`

#### 属性

接口中的抽象方法被称为属性，定义属性有以下要求：

1. 属性的返回值类型有这些取值：基本数据类型、String、枚举、注解、以上类型的数组。
```java
// 代码示例
public @interface MyAnno {
    int age(); // 八种基本数据类型
    String name() default "zhangsan"; // String类型
    Person per(); // 枚举类型
    MyAnno2 anno2(); // 注解类型
    String[] strarr(); // 以上类型数组均可
}
```
2. 定义了属性，在使用时需给属性赋值
```java
// 赋值示例
@MyAnno(age = 12,per = Person.P1,anno2 = @MyAnno2,strarr = {"a","b"})
public class Worker {
}
```

> 小总结：
>
> 1. 如果定义属性时，使用default关键字给属性默认初始化值，则使用注解时，可以不进行属性的赋值。
> 2. 如果只有一个属性需要赋值，且属性名称是value，则名称value可省略，直接写值。
> 3. 数组赋值时，值使用{}包裹。如果数组中只有一个值，则{}可省略。

#### 元注解(用于描述注解的注解)

* `@Target`：描述注解能够作用的位置，默认value值有`ElementType`枚举类型，常用取值有如下三个：
  * TYPE：可以作用于类上
  * METHOD：可以作用于方法上
  * FIELD：可以作用于成员变量上
* `@Retention`：描述注解被保留的阶段
  * `@Retention(RetentionPolicy.RUNTIME)`：当前被描述的注解，会保留到class字节码文件中，并被JVM读取到。
* `@Documented`：描述注解是否被抽取到api文档中
* `@Inherited`：描述注解是否被子类继承

### 3.3 在程序中使用(解析)注解

就是获取注解中定义的属性值

使用步骤：

1. 获取注解定义的位置的对象(Class，Method，Field)
2. 获取指定的注解：`getAnnotation(Class)`

```java
// 其实就是在内存中生成了一个该注解接口的子类实现对象
public class ProImpl implements Pro{
    public String className(){
        return "Demo5_16.annotation.Demo1";
    }
    public String methodName(){
        return "show"
    }
}
```

3. 调用注解中的抽象方法获取配置的属性值

#### 代码示例

```java
// 自定义的注解Pro
@Target(ElementType.TYPE) // 可作用于类上
@Retention(RetentionPolicy.RUNTIME) // 保留在runtime阶段
public @interface Pro {
    String className();
    String methodName();
}
// 方法类
public class Demo01 {
    public void show(){
        System.out.println("demo1...show....");
    }
}
// 测试类
@Pro(className = "Demo5_16.annotation.Demo01",methodName = "show")
public class MyReflectFrame {
    public static void main(String[] args) throws Exception {
        // 可以创建任意类的对象，可以执行任意方法
        // 前提：不能改变该类的任何代码的前提下，可以帮我们创建任意类的对象，并且执行其中任意方法

        // 解析注解
        // 1.1 获取该类的字节码文件对象
        Class<MyReflectFrame> myreflect = MyReflectFrame.class;
        // 2. 获取上边的注解对象(其实就是在内存中生成了一个该注解接口的子类实现对象)
        Pro an = myreflect.getAnnotation(Pro.class);
        // 3. 调用注解对象中定义的抽象方法，获取返回值
        String className = an.className();
        String methodName = an.methodName();
        System.out.println(className);
        System.out.println(methodName);
        // 4. 加载该类进内存
        Class cls = Class.forName(className); // ClassNotFoundException: Demo5_16.annotation.Demo1 ?!!!
        // 5. 创建对象
        Object obj = cls.newInstance();
        // 6. 获取方法对象
        Method method = cls.getMethod(methodName);
        // 7. 执行方法
        method.invoke(obj);
    }
}
```

### 3.4 综合案例

```java
// 定义简单的计算类，使用@Check注解
public class Calculator {
    @Check
    public void add(){
        String str = null;
        System.out.println(str.toString()); // 会报空指针异常
        System.out.println("1 + 0 =" + (1 + 0));
    }
    @Check
    public void sub(){
        System.out.println("1 - 0 =" + (1 - 0));
    }
    @Check
    public void mul(){
        System.out.println("1 * 0 =" + (1 * 0));
    }
    @Check
    public void div(){
        System.out.println("1 / 0 =" + (1 / 0));
    }
}
// 定义Check注解类
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Check {
}
// 定义测试框架类TestCheck
// 当主方法执行后，会自动地运行被检测的所有方法(加了Check注解的方法)，判断方法是否有异常，记录到文件中
public class TestCheck {
    public static void main(String[] args) throws IOException {
        Calculator calc = new Calculator();
        // 2. 获取字节码文件对象
        Class cls = calc.getClass();
        // 3. 获取所有方法
        Method[] methods = cls.getMethods();
        int count = 0; // 出现异常次数
        BufferedWriter bw = new BufferedWriter(new FileWriter("D:\\IDEA_WorkSpace\\JavaWeb\\src\\Demo5_16\\Check\\bug" + System.currentTimeMillis() + "txt",true));
        for (Method method : methods) {
            // 4. 判断方法上是否有Check注解
            if (method.isAnnotationPresent(Check.class)){ // 判断该方法的注解对象是否为输入的注解对象，返回布尔值，true则是。
                // 5. 有，则执行方法
                try {
                    method.invoke(calc); // 调用方法
                } catch (Exception e) {
                    // 6. 捕获异常,将异常记录到文件中
                    count ++;
                    bw.write(method.getName() + "方法出异常了"); // 获取方法名称
                    bw.newLine();
                    bw.write("异常的名称："+e.getCause());
                    bw.newLine();
                    bw.write("异常的原因："+e.getCause().getMessage());
                    bw.newLine();
                    bw.write("---------------------------");
                    bw.newLine();
                }
            }
        }
        bw.write("本次测试一共出现" + count + "次异常");
        bw.flush();
        bw.close();
    }
}
/* 输出文件：
div方法出异常了
异常的名称：java.lang.ArithmeticException: / by zero
异常的原因：/ by zero
---------------------------
add方法出异常了
异常的名称：java.lang.NullPointerException
异常的原因：null
---------------------------
本次测试一共出现2次异常
*/
```

#### 小结

1. 大多数时候，我们会使用注解，而不是自定义注解
2. 注解给谁用？一般给编译器和解析程序(TestCheck就是解析程序)用
3. 注解不是程序的一部分，可以理解为注解就是一个标签