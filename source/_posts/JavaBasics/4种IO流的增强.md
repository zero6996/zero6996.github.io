---
title: 缓冲流、转换流、序列化流、打印流
date: 2019-5-10 23:00
categories: Java学习笔记
tags: [Java,IO流]
description: 讲述了Java中的4种IO流的增强用法
---



## 1. 缓冲流

缓冲流,也叫高效流，是对4个基本的`FileXxx` 流的增强，所以也是4个流，按照数据类型分类：

- **字节缓冲流**：`BufferedInputStream`，`BufferedOutputStream` 
- **字符缓冲流**：`BufferedReader`，`BufferedWriter`



<!--more-->

缓冲流的基本原理，是在创建流对象时，会创建一个内置的默认大小的缓冲区数组，通过缓冲区读写，减少系统IO次数，从而提高读写的效率。



### 1.1 字节缓冲流

#### 构造方法

- `public BufferedInputStream(InputStream in)` ：创建一个 新的缓冲输入流。 
- `public BufferedOutputStream(OutputStream out)`： 创建一个新的缓冲输出流。



#### 效率测试

缓冲流读写方法与基本流是一致的，通过复制大文件(560MB)，来测试一下效率。

1. 基本流代码

```java
public class BufferedDemo01 {
    public static void main(String[] args) {
        long start = System.currentTimeMillis(); // 记录开始时间
        // 创建流对象
        try(
            FileInputStream fis = new FileInputStream("D:\\chrome\\IdeaInstall.exe");
            FileOutputStream fos = new FileOutputStream("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\copy.exe");
        ){
            //读写数据
            int b;
            while ((b = fis.read())!=-1){
                fos.write(b);
            }
        }catch (IOException e) {
            e.printStackTrace();
        }
        long end = System.currentTimeMillis();// 记录结束时间
        System.out.println("普通流复制560MB所需时间："+(end-start)+"毫秒");
    }
}
// result: long long time
```

2. 缓冲流代码

```java
public class ButteredDemo02 {
    public static void main(String[] args) {
        long start = System.currentTimeMillis(); // 记录开始时间
        try(// 创建缓冲流对象
            BufferedInputStream bis = new BufferedInputStream(new FileInputStream("D:\\chrome\\IdeaInstall.exe"));
            BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\copy2.exe"))
        ){
            //读写数据
            int b;
            while ((b = bis.read())!=-1){
                bos.write(b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long end = System.currentTimeMillis(); // 记录结束时间
        System.out.println("缓冲流复制560MB所需时间："+(end-start)+"毫秒");
    }
}
// result：缓冲流复制560MB所需时间：40171毫秒
```

3. 使用数组优化

```java
public class ButteredDemo02 {
    public static void main(String[] args) {
        long start = System.currentTimeMillis(); // 记录开始时间
        try(// 创建缓冲流对象
            BufferedInputStream bis = new BufferedInputStream(new FileInputStream("D:\\chrome\\IdeaInstall.exe"));
            BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\copy2.exe"))
        ){
            //读写数据
            int len;
            byte[] bytes = new byte[8*1024]; // 使用数组对象优化
            while ((len = bis.read(bytes))!=-1){
                bos.write(bytes,0,len);  // 只写入有效字节数据
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long end = System.currentTimeMillis(); // 记录结束时间
        System.out.println("缓冲流使用数组复制560MB所需时间："+(end-start)+"毫秒");
    }
}
// result:	缓冲流使用数组复制560MB所需时间：2847毫秒
```

### 1.2 字符缓冲流

#### 构造方法

- `public BufferedReader(Reader in)` ：创建一个新的字符缓冲输入流。 
- `public BufferedWriter(Writer out)`： 创建一个新的字符缓冲输出流。

#### 特有方法

字符缓冲流的基本方法与普通字符流调用方式一致，但它具备特有的方法。

- BufferedReader：`public String readLine()`: 读一行文字。 
- BufferedWriter：`public void newLine()`: 写一行行分隔符,由系统属性定义符号。 

`readLine`方法演示：

```java
public class BufferedReaderDemo01 {
    public static void main(String[] args) throws IOException {
        // 创建流对象
        BufferedReader br = new BufferedReader(new FileReader("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\read.txt"));
        // 定义字符串，保存读取的一行文字
        String line;
        // 循环读取，直到读取到null
        while ((line = br.readLine())!=null){
            System.out.print(line);
            System.out.println("，");
        }
        br.close(); // 释放资源
    }
}
```

`newLine`方法演示：

~~~java
public class BufferedWriterDemo02 {
    public static void main(String[] args) throws IOException {
        // 创建流对象
        BufferedWriter bw = new BufferedWriter(new FileWriter("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\out.txt"));
        // 写出数据
        bw.write("你好");
        bw.newLine(); // 写出换行
        bw.write("世界");
        bw.newLine();
        bw.close();
    }
}
// result:
你好
世界
~~~

### 1.3 练习：文本排序

```
3.侍中、侍郎郭攸之、费祎、董允等，此皆良实，志虑忠纯，是以先帝简拔以遗陛下。愚以为宫中之事，事无大小，悉以咨之，然后施行，必得裨补阙漏，有所广益。
8.愿陛下托臣以讨贼兴复之效，不效，则治臣之罪，以告先帝之灵。若无兴德之言，则责攸之、祎、允等之慢，以彰其咎；陛下亦宜自谋，以咨诹善道，察纳雅言，深追先帝遗诏，臣不胜受恩感激。
4.将军向宠，性行淑均，晓畅军事，试用之于昔日，先帝称之曰能，是以众议举宠为督。愚以为营中之事，悉以咨之，必能使行阵和睦，优劣得所。
2.宫中府中，俱为一体，陟罚臧否，不宜异同。若有作奸犯科及为忠善者，宜付有司论其刑赏，以昭陛下平明之理，不宜偏私，使内外异法也。
1.先帝创业未半而中道崩殂，今天下三分，益州疲弊，此诚危急存亡之秋也。然侍卫之臣不懈于内，忠志之士忘身于外者，盖追先帝之殊遇，欲报之于陛下也。诚宜开张圣听，以光先帝遗德，恢弘志士之气，不宜妄自菲薄，引喻失义，以塞忠谏之路也。
9.今当远离，临表涕零，不知所言。
6.臣本布衣，躬耕于南阳，苟全性命于乱世，不求闻达于诸侯。先帝不以臣卑鄙，猥自枉屈，三顾臣于草庐之中，咨臣以当世之事，由是感激，遂许先帝以驱驰。后值倾覆，受任于败军之际，奉命于危难之间，尔来二十有一年矣。
7.先帝知臣谨慎，故临崩寄臣以大事也。受命以来，夙夜忧叹，恐付托不效，以伤先帝之明，故五月渡泸，深入不毛。今南方已定，兵甲已足，当奖率三军，北定中原，庶竭驽钝，攘除奸凶，兴复汉室，还于旧都。此臣所以报先帝而忠陛下之职分也。至于斟酌损益，进尽忠言，则攸之、祎、允之任也。
5.亲贤臣，远小人，此先汉所以兴隆也；亲小人，远贤臣，此后汉所以倾颓也。先帝在时，每与臣论此事，未尝不叹息痛恨于桓、灵也。侍中、尚书、长史、参军，此悉贞良死节之臣，愿陛下亲之信之，则汉室之隆，可计日而待也。
```

#### 案例分析

1. 逐行读取文本信息。
2. 解析文本信息到集合中。
3. 遍历集合，按顺序，写出文本信息。

#### 代码实现

```java
public class BufferedExercise {
    public static void main(String[] args) {
        // 创建map集合，保存文本数据，键为序号，值为文字
        HashMap<String,String> lineMap = new HashMap<>();
        // 创建流对象
        try(
                BufferedReader br = new BufferedReader(new FileReader("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\出师表.txt"));
                BufferedWriter bw = new BufferedWriter(new FileWriter("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\sort.txt"))
        ){
            // 循环读取数据，解析存入map中
            String line;
            while((line = br.readLine())!=null){
                // 解析文本
                String[] split = line.split("\\."); // 以"."作为分隔符，将一行分为序号和文字两部分
                lineMap.put(split[0],split[1]);// 序号为键，文字为值，存入集合中
            }
            // 遍历map集合
            for (int i = 1; i <= lineMap.size(); i++) {
                String key = String.valueOf(i); // 获取键名为i的键
                // 获取map中的文本
                String value = lineMap.get(key); // 获取键名为key的对应值
                // 拼接字符串，并写出
                bw.write(key + "." + value);
                bw.newLine(); // 写出换行
            }
        }catch (IOException e){
            e.printStackTrace();
        }
    }
}
```

## 2. 转换流

### 2.1 字符编码和字符集

#### 字符编码

计算机中储存的信息都是用二进制数表示的，而我们在屏幕上看到的数字、英文、标点符号、汉字等字符是二进制数转换之后的结果。按照某种规则，将字符存储到计算机中，称为**编码** 。反之，将存储在计算机中的二进制数按照某种规则解析显示出来，称为**解码** 。比如说，按照A规则存储，同样按照A规则解析，那么就能显示正确的文本符号。反之，按照A规则存储，再按照B规则解析，就会导致乱码现象。

**编码**:字符(能看懂的)--字节(看不懂的)

**解码**:字节(看不懂的)-->字符(能看懂的)

* **字符编码`Character Encoding`** : 就是一套自然语言的字符与二进制数之间的对应规则。

#### 字符集

- **字符集 `Charset`**：也叫编码表。是一个系统支持的所有字符的集合，包括各国家文字、标点符号、图形符号、数字等。

![](https://i.loli.net/2019/05/10/5cd540cb5cccd.jpg)

当指定了**编码**，它所对应的**字符集**自然就指定了，所以**编码**才是我们最终要关心的。

- **ASCII字符集** ：
	- ASCII（American Standard Code for Information Interchange，美国信息交换标准代码）是基于拉丁字母的一套电脑编码系统，用于显示现代英语，主要包括控制字符（回车键、退格、换行键等）和可显示字符（英文大小写字符、阿拉伯数字和西文符号）。
	- 基本的ASCII字符集，使用7位（bits）表示一个字符，共128字符。ASCII的扩展字符集使用8位（bits）表示一个字符，共256字符，方便支持欧洲常用字符。

* **ISO-8859-1字符集**：
  * 拉丁码表，别名Latin-1，用于显示欧洲使用的语言，包括荷兰、丹麦、德语、意大利语、西班牙语等。
  * ISO-8859-1使用单字节编码，兼容ASCII编码。
* **GBxxx字符集**：
  * GB就是国标的意思，是为了显示中文而设计的一套字符集。
  * **GB2312**：简体中文码表。一个小于127的字符的意义与原来相同。但两个大于127的字符连在一起时，就表示一个汉字，这样大约可以组合了包含7000多个简体汉字，此外数学符号、罗马希腊的字母、日文的假名们都编进去了，连在ASCII里本来就有的数字、标点、字母都统统重新编了两个字节长的编码，这就是常说的"全角"字符，而原来在127号以下的那些就叫"半角"字符了。
  * **GBK**：最常用的中文码表。是在GB2312标准基础上的扩展规范，使用了双字节编码方案，共收录了21003个汉字，完全兼容GB2312标准，同时支持繁体汉字以及日韩汉字等。
  * **GB18030**：最新的中文码表。收录汉字70244个，采用多字节编码，每个字可以由1个、2个或4个字节组成。支持中国国内少数民族的文字，同时支持繁体汉字以及日韩汉字等。
* **Unicode字符集** ：
  * Unicode编码系统为表达任意语言的任意字符而设计，是业界的一种标准，也称为统一码、标准万国码。
  * 它最多使用4个字节的数字来表达每个字母、符号，或者文字。有三种编码方案，UTF-8、UTF-16和UTF-32。最为常用的UTF-8编码。
  * UTF-8编码，可以用来表示Unicode标准中任何字符，它是电子邮件、网页及其他存储或传送文字的应用中，优先采用的编码。互联网工程工作小组（IETF）要求所有互联网协议都必须支持UTF-8编码。所以，我们开发Web应用，也要使用UTF-8编码。它使用一至四个字节为每个字符编码，编码规则：
    1. 128个US-ASCII字符，只需一个字节编码。
    2. 拉丁文等字符，需要二个字节编码。 
    3. 大部分常用字（含中文），使用三个字节编码。
    4. 其他极少使用的Unicode辅助字符，使用四字节编码。

### 2.2 编码引出的问题

在IDEA中，使用`FileReader` 读取项目中的文本文件。由于IDEA的设置，都是默认的`UTF-8`编码，所以没有任何问题。但是，当读取Windows系统中创建的文本文件时，由于Windows系统的默认是GBK编码，就会出现乱码。

### 2.3 InputStreamReader类

转换流`java.io.InputStreamReader`，是Reader的子类，是从字节流到字符流的桥梁。它读取字节，并使用指定的字符集将其解码为字符。它的字符集可以由名称指定，也可以接受平台的默认字符集。

#### 构造方法

- `InputStreamReader(InputStream in)`: 创建一个使用默认字符集的字符流。 
- `InputStreamReader(InputStream in, String charsetName)`: 创建一个指定字符集的字符流。`

#### 指定编码读取Demo

~~~java
public class ReaderDemo {
    public static void main(String[] args) throws IOException {
        // 定义文件路径
        String FilePath = "D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\file_gbk.txt";
        // 创建流对象，默认UTF8编码
        InputStreamReader isr = new InputStreamReader(new FileInputStream(FilePath));
        // 创建流对象，指定GBK编码
        InputStreamReader isr2 = new InputStreamReader(new FileInputStream(FilePath),"GBK");
        // 定义变量，保存字符
        int read;
        // 使用默认编码字符流读取，乱码
        while ((read = isr.read())!=-1){
            System.out.print((char)read); // ��Һ�
        }
        isr.close();
        // 使用指定GBK编码字符流读取
        while ((read = isr2.read())!=-1){
            System.out.println((char)read); // 大家好
        }
        isr2.close();
    }
}
~~~

### 2.4 OutPutStreamWriter类
转换流`java.io.OutputStreamWriter` ，是Writer的子类，是从字符流到字节流的桥梁。使用指定的字符集将字符编码为字节。它的字符集可以由名称指定，也可以接受平台的默认字符集。 

#### 构造方法

- `OutputStreamWriter(OutputStream in)`: 创建一个使用默认字符集的字符流。 
- `OutputStreamWriter(OutputStream in, String charsetName)`: 创建一个指定字符集的字符流。

#### 指定编码写出Demo

```java
public class OutPutDemo {
    public static void main(String[] args) throws IOException {
        String FilePath = "D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\";
        // 创建流对象，默认UTF8编码
        OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(FilePath + "out2.txt"));
        // 写出数据
        osw.write("你好"); // 保存为6个字节
        osw.close();
        // 创建流对象，指定GBK编码
        OutputStreamWriter osw2 = new OutputStreamWriter(new FileOutputStream(FilePath + "out3.txt"),"GBK");
        // 写出数据
        osw2.write("你好"); // 保存为4个字节
        osw2.close();
    }
}
```

#### 转换流图解

**转换流是字节与字符间的桥梁！**

![](https://i.loli.net/2019/05/10/5cd56e55421e2.jpg)



### 2.5 练习：转换文件编码

将GBK编码的文本文件，转换为UTF-8编码的文本文件。

#### 案例分析

1. 指定GBK编码的转换流，读取文本文件。
2. 使用UTF-8编码的转换流，写出文本文件。

#### 代码实现

```java
public class CodeSwitchDemo {
    public static void main(String[] args) {
        try( // 1. 指定GBK编码的转换流，读取文本文件。
             InputStreamReader isr = new InputStreamReader(new FileInputStream("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\file_gbk.txt"),"GBK");
             OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\codeswitch.txt")) // 默认就是utf8
        ){
            // 读取文件
            char[] chars = new char[1024]; // 定义字符数组
            int len;
            while ((len = isr.read(chars))!=-1){ // 使用字符数组优化读取
                osw.write(chars,0,len); // 写出
            }
        }catch (IOException e){
            e.printStackTrace();
        }
    }
}
```

## 3. 序列化

Java 提供了一种对象**序列化**的机制。用一个字节序列可以表示一个对象，该字节序列包含该`对象的数据`、`对象的类型`和`对象中存储的属性`等信息。字节序列写出到文件之后，相当于文件中**持久保存**了一个对象的信息。 

反之，该字节序列还可以从文件中读取回来，重构对象，对它进行**反序列化**。`对象的数据`、`对象的类型`和`对象中存储的数据`信息，都可以用来在内存中创建对象。看图理解序列化： 

![](https://i.loli.net/2019/05/10/5cd575db8ecac.jpg)



### 3.1 ObjectOutputStream类

`java.io.ObjectOutputStream ` 类，将Java对象的原始数据类型写出到文件,实现对象的持久存储。

#### 构造方法

* `public ObjectOutputStream(OutputStream out)`：创建一个指定`OutputStream`的`ObjectOutputStream`。

#### 对象序列化操作(写对象操作)

1.  一个对象要想序列化，必须满足两个条件

- 该类必须实现`java.io.Serializable ` 接口，`Serializable` 是一个标记接口，不实现此接口的类将不会使任何状态序列化或反序列化，会抛出`NotSerializableException` 。
- 该类的所有属性必须是可序列化的。如果有一个属性不需要可序列化的，则该属性必须注明是瞬态的，使用`transient` 关键字修饰。

~~~java
public class Employee implements java.io.Serializable {
    public String name;
    public String address;
    public transient int age; // transient瞬态修饰成员,不会被序列化
    public void addressCheck() {
      	System.out.println("Address  check : " + name + " -- " + address);
    }
}
~~~

2. 写出对象方法

- `public final void writeObject (Object obj)` : 将指定的对象写出。

#### 代码示例

~~~java
// 测试类
/*
java.io.ObjectOutputStream extends OutputStream
    ObjectOutputStream: 对象序列化流
    作用：把对象以流的方式写入到文件中保存

    构造方法：
        ObjectOutputStream(OutputStream out): 创建写入指定OutputStream的ObjectOutputStream
        参数：OutputStream out：字节输出流
    特有成员方法：
        void writeObject(Object obj):将指定对象写入ObjectOutputStream
*/
public class Demo01ObjectOutputStream {
    public static void main(String[] args) throws IOException {
        // 1. 创建ObjectOutputStream，构造方法中传递字节输出流
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\person.txt"));
        // 2. 使用ObjectOutputStream对象中的方法writeObject()，把对象写入到文件中
        oos.writeObject(new Person("小张",11));
        // 3. 释放资源
        oos.close();
    }
}
// Person类
/*
    序列化和反序列化时，会抛出NotSerializableException(没有序列化异常)
    类通过实现 Java.io.Serializable 接口以启用其序列化功能。未实现此接口的类将无法使其进行任何状态序列化或反序列化。
    Serializable接口也叫标记型接口
        要进行序列化和反序列化的类必须实现Serializable接口，就会给类添加一个标记
        当我们进行序列化和反序列化时，就会检测该类是否有这个标记
            有：就可以进行序列化和反序列化操作
            无：就会抛出NotSerializableException异常
 */
public class Person implements Serializable {
    private String name;
    private int age;

    public Person() {
    }
    // 省略get/set等方法
}
~~~

### 3.2 ObjectInputStream类

`ObjectInputStream`反序列化流，将之前使用`ObjectOutputStream`序列化的原始数据恢复为对象。 

#### 构造方法

- `public ObjectInputStream(InputStream in) `： 创建一个指定`InputStream`的`ObjectInputStream`。

#### 反序列化操作1(读对象操作)

如果能找到一个对象的class文件，我们可以进行反序列化操作，调用`ObjectInputStream`读取对象的方法：

- `public final Object readObject ()` : 读取一个对象。

#### 代码示例

```java
/*
java.io.ObjectInputStream extends InputStream
    ObjectInputStream:对象的反序列化流
    作用：把文件中保存的对象，以流的方式读取出来使用

    构造方法：
        ObjectInputStream(InputStream in)：创建从指定InputStream中读取的ObjectInputStream
        参数：InputStream in：字节输入流
    特有成员方法：
        Object readObject(): 从ObjectInputStream 读取对象
*/
public class Demo01InputStream {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        // 1. 创建ObjectInputStream对象，构造方法中传递字节输入流
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\person.txt"));
        // 2. 使用ObjectInputStream对象的方法readObject() 读取保存对象的文件
        Object obj = ois.readObject();
       	/*
       		readObject()方法声明抛出了ClassNotFoundException(class文件找不到异常)
            当不存在对象的class文件时抛出此异常
            反序列化前提：
                1. 类必须实现Serializable
                2. 必须存在类对应的class文件
       	*/
        // 3. 释放资源
        ois.close();
        // 4. 使用读取出来的对象
        System.out.println(obj); // Person{name='小张', age=11}
    }
}
```

> Tips:   **对于JVM可以反序列化对象，它必须是能够找到class文件的类。如果找不到该类的class文件，则抛出一个 `ClassNotFoundException` 异常。**  



#### 反序列化操作2

**另外，当JVM反序列化对象时，能找到class文件，但是class文件在序列化对象之后发生了修改，那么反序列化操作也会失败，抛出一个`InvalidClassException`异常。**发生这个异常的原因如下：

- 该类的序列版本号与从流中读取的类描述符的版本号不匹配 
- 该类包含未知数据类型 
- 该类没有可访问的无参数构造方法 

`Serializable` 接口给需要序列化的类，提供了一个序列版本号。

`serialVersionUID` 该版本号的目的在于验证序列化的对象和对应类是否版本匹配。

```java
public class Person implements Serializable {
    private static final long serialVersionUID = 1L; // 加入固定序列版本号
    private String name;
    public int age;	
    // 省略构造方法，get/set等
}
```

### 3.3 练习：序列化集合

1. 将存有多个自定义对象的集合序列化操作，保存到`list.txt`文件中。
2. 反序列化`list.txt` ，并遍历集合，打印对象信息。

#### 案例分析

1. 把若干学生对象 ，保存到集合中。
2. 把集合序列化。
3. 反序列化读取时，只需要读取一次，转换为集合类型。
4. 遍历集合，可以打印所有的学生信息

#### 代码实现

~~~java
// 序列化操作
public class StudentSerializableDemo {
    public static void main(String[] args) throws IOException {
    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\students.txt"));
    ArrayList<Person> arrayList = new ArrayList<>();
    arrayList.add(new Person("xiaoming",12));
    arrayList.add(new Person("xiaohua",17));
    arrayList.add(new Person("xiaoli",15));
    oos.writeObject(arrayList);
    oos.close();
}
}
// 反序列化操作
public class StudentDemo2 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\students.txt"));
        // 读取，强转为ArrayList类型
        ArrayList<Person> stuobj = (ArrayList<Person>) ois.readObject();
        // 遍历打印
        for (Person p:stuobj){
            System.out.println(p);
        }
        ois.close();
    }
}
~~~

## 4. 打印流

我们在控制台打印输出，是调用`print`方法和`println`方法完成的，这两个方法都来自于`java.io.PrintStream`类，该类能够方便地打印各种数据类型的值，是一种便捷的输出方式。

### 4.1 PringStream类

#### 构造方法

- `public PrintStream(String fileName)`： 使用指定的文件名创建一个新的打印流。

#### 改变打印流向

`System.out`就是`PrintStream`类型的，只不过它的流向是系统规定的，打印在控制台上。不过，既然是流对象，我们就可以改变它的流向。

```java
public class PrintDemo {
    public static void main(String[] args) throws FileNotFoundException {
        System.out.println(97);
        // 创建打印流，指定文件名称
        PrintStream ps = new PrintStream("D:\\IDEA_WorkSpace\\Demo\\Demo5_10\\print.txt");
        // 设置系统的打印流向，输出到print.txt中
        System.setOut(ps);
        // 调用系统打印里，就会在print.txt中输出97
        System.out.println(97);
    }
}
```













### 今日目标
- [x] 能够使用字节缓冲流读取数据到程序。FileInputStream
- [x] 能够使用字节缓冲流写出数据到文件。FileOutPutStream
- [x] 能够明确字符缓冲流的作用和基本用法。FileReader，FileWriter
- [x] 能够使用缓冲流的特殊功能。数组优化
- [x] 能够阐述编码表的意义
- [x] 能够使用转换流读取指定编码的文本文件。 InputStreamReader
- [x] 能够使用转换流写入指定编码的文本文件。 OutputStreamWriter
- [x] 能够说出打印流的特点。 可以改变打印流向，输出到文件中
- [x] 能够使用序列化流写出对象到文件。ObjectOutputStream
- [x] 能够使用反序列化流读取文件到程序中。ObjectInputStream