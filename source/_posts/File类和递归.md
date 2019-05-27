---
title: Java中File类和递归
date: 2019-5-5 23:59
categories: Java学习笔记 # 分类
tags: [Java,File类,递归]
description: Java中File类的概述和基本使用，递归概念和使用
---

## 1. File类
`java.io.File` 类是文件和目录路径名的抽象表示，主要用于文件和目录的创建、查找和删除等操作。

<!--more-->

### 1.1 构造方法
- `public File(String pathname) ` ：通过将给定的**路径名字符串**转换为抽象路径名来创建新的 File实例。  
- `public File(String parent, String child) ` ：从**父路径名字符串和子路径名字符串**创建新的 File实例。
- `public File(File parent, String child)` ：从**父抽象路径名和子路径名字符串**创建新的 File实例。

代码举例如下：
~~~java
public class DemoFileClass {
    public static void main(String[] args) {
        // 通过文件绝对路径
        String pathname = "D:\\IDEA_WorkSpace\\Demo\\Demo5_5\\testfile.txt";
        File file1 = new File(pathname);
//        System.out.println(file1);
        // 通过父路径和子路径字符串
        String parent = "D:\\IDEA_WorkSpace\\Demo\\Demo5_5";
        String child = "testfile.txt";
        File file2 = new File(parent,child);
//        System.out.println(file2);
        // 通过父级File对象和子路径字符串
        File parentDir = new File("D:\\IDEA_WorkSpace\\Demo\\Demo5_5");
        File file3 = new File(parentDir,child);
        System.out.println(file3);
    }
}
~~~

> 一个File对象代表硬盘中实际存在的一个文件或者目录；无论该路径下是否存在文件或者目录，都不影响File对象的创建。

### 1.2 常用方法
#### 获取功能的方法
- `public String getAbsolutePath() ` ：返回此File的绝对路径名字符串。
- ` public String getPath() ` ：将此File转换为路径名字符串。 
- `public String getName()`  ：返回由此File表示的文件或目录的名称。  
- `public long length()`  ：返回由此File表示的文件的长度。 

实例代码如下：
```java
public class GetFileDemo {
    public static void main(String[] args) {
        File file = new File("D:\\IDEA_WorkSpace\\Demo\\Demo5_5\\testfile.txt");
        System.out.println("文件绝对路径:" + file.getAbsolutePath());
        System.out.println("文件构造路径:" + file.getPath());
        System.out.println("文件名称:" + file.getName());
        System.out.println("文件长度:" + file.length()+"字节");
        File fileDir = new File("D:\\IDEA_WorkSpace\\Demo\\Demo5_5");
        System.out.println("目录绝对路径：" + fileDir.getAbsolutePath());
        System.out.println("目录构造路径：" + fileDir.getPath());
        System.out.println("目录名称：" + fileDir.getName());
        System.out.println("目录长度：" + fileDir.length());
    }
}
// 上述输出结果：
文件绝对路径:D:\IDEA_WorkSpace\Demo\Demo5_5\testfile.txt
文件构造路径:D:\IDEA_WorkSpace\Demo\Demo5_5\testfile.txt
文件名称:testfile.txt
文件长度:32字节
目录绝对路径：D:\IDEA_WorkSpace\Demo\Demo5_5
目录构造路径：D:\IDEA_WorkSpace\Demo\Demo5_5
目录名称：Demo5_5
目录长度：4096
```
> API中说明：length(), 表示文件的长度。但是File对象表示目录，则返回值未指定。

#### 绝对路径和相对路径
- **绝对路径**：从盘符开始的路径，这是一个完整的路径。
- **相对路径**：相对于项目目录的路径，这是一个便捷的路径，开发中经常使用。
~~~java
public class FilePath {
    public static void main(String[] args) {
        System.out.println(new File("D:\\IDEA_WorkSpace\\Demo\\" +
                "Demo5_5\\testfile.txt").getAbsolutePath());
        System.out.println(new File("testfile.txt").getAbsolutePath());
    }
}
// output result
D:\IDEA_WorkSpace\Demo\Demo5_5\testfile.txt
D:\IDEA_WorkSpace\testfile.txt
~~~

#### 判定功能的方法
- `public boolean exists()` ：此File表示的文件或目录是否实际存在。
- `public boolean isDirectory()` ：此File表示的是否为目录。
- `public boolean isFile()` ：此File表示的是否为文件。

示例代码如下：
```java
public class FileIs {
    public static void main(String[] args) {
        File file = new File("D:\\IDEA_WorkSpace\\Demo\\Demo5_5\\testfile.txt");
        File dir = new File("D:\\IDEA_WorkSpace\\Demo\\Demo5_5");
        // 判断是否存在文件或者目录,返回布尔值
        System.out.println(file.exists()); // true
        System.out.println(dir.exists()); // true
	System.out.println(new File("A:\\testDir").exists()); // false	
        // 判断是文件还是目录
//        System.out.println("is file?: " + file.isFile()); // true
//        System.out.println("is file?: " + dir.isFile()); // false
//        System.out.println("is directory?: " + file.isDirectory()); // false
//        System.out.println("is directory?: " + dir.isDirectory()); // true
        Judge(file); 
        Judge(dir);
    }
    public static void Judge(File file){
        if (file.isFile())
            System.out.println(file + "  是文件");
        if (file.isDirectory())
            System.out.println(file + "  是目录");
    }
}
```

#### 创建删除功能的方法
- `public boolean createNewFile()` ：当且仅当具有该名称的文件尚不存在时，创建一个新的空文件。 
- `public boolean delete()` ：删除由此File表示的文件或目录。  
- `public boolean mkdir()` ：创建由此File表示的目录。
- `public boolean mkdirs()` ：创建由此File表示的目录，包括任何必需但不存在的父目录。

示例如下：
~~~java
public class FileCreDel{
    final static String rootpath = "D:\\IDEA_WorkSpace\\Demo\\Demo5_5\\";
    public static void main (String[] args) throws IOException {
        System.out.println("创建文件:1\n创建文件夹：2\n删除文件或文件夹：3\n退出：4");
        Scanner sc = new Scanner(System.in);
        while (true) {
            int pattern = sc.nextInt();
            switch (pattern) {
                case 1:
                    System.out.println("输入需要创建的文件名:");
                    CreFile(sc.next());
                    break;
                case 2:
                    System.out.println("输入需要创建的文件夹名:");
                    MkDir(sc.next());
                    break;
                case 3:
                    System.out.println("输入需要删除的文件或文件夹名称:");
                    Del(sc.next());
                    break;
                case 4:
                    System.exit(0);
            }
        }
    }
    public static void CreFile(String name) throws IOException {
        File file = new File(rootpath + name);
        if (file.exists() == false) {
            file.createNewFile();
            System.out.println(name + "文件创建成功");
        } else{
            System.out.println(name + "文件已经存在，创建失败！");
        }
    }
    public static void MkDir(String name) throws IOException{
        File dir = new File(rootpath + name);
        if (dir.exists() == false){
            dir.mkdirs();
            System.out.println(name + "文件夹创建成功");
        }else{
            System.out.println(name + "文件夹已存在，创建失败！");
        }
    }
    public static void Del(String name){
        if (new File(rootpath + name).delete() == true){
            System.out.println("删除成功！");
        }else {
            System.out.println("删除失败，文件或文件夹不存在！");
        }
    }
}
~~~

> API中说明：delete方法，如果此File表示目录，则目录必须为空才能删除。

### 1.3 目录的遍历
- `public String[] list()` ：返回一个String数组，表示该File目录中的所有子文件或目录。
- `public File[] listFiles()` ：返回一个File数组，表示该File目录中的所有的子文件或目录。
~~~java
public class FileFor {
    public static void main(String[] args) {
        File dir = new File("D:\\IDEA_WorkSpace\\Demo\\Demo5_5\\");
        // 获取当前目录下的文件以及文件夹的名称。
        String[] names = dir.list();
        for(String name:names){
            System.out.println(name);
        }
        // 获取当前目录下的文件以及文件夹对象，只要拿到了文件对象，那么就可以获取更多信息。
        File[] files = dir.listFiles();
        for (File file:files){
            System.out.println(file);
        }
    }
}
~~~

> Tips:调用listFiles方法的File对象，表示的必须是实际存在的目录，否则返回null，无法进行遍历。

## 2. 递归
### 2.1 概述
* **递归**：指在当前方法内调用自己的这种现象。
* **递归的分类:**
	- 递归分为两种，直接递归和间接递归。
	- 直接递归称为方法自身调用自己。
	- 间接递归可以A方法调用B方法，B方法调用C方法，C方法调用A方法。
* **注意事项**：
	- 递归一定要有条件限定，保证递归能够停止下来，否则会发生栈内存溢出。
	- 在递归中虽然有限定条件，但是递归次数不能太多。否则也会发生栈内存溢出。
	- 构造方法,禁止递归。

### 2.2 递归累加求和
#### 计算1~n的和
**分析**：num的累和 = num + (num-1)的累和，所以可以把累和的操作定义成一个方法，递归调用。

实例如下：
```java
public class DemoRecursion02 {
    public static void main(String[] args) {
        int num = 5;
        System.out.println(getSum(num));
    }
    public static int getSum(int num){
        if (num == 1)
            return 1;
        return num+getSum(--num); // 为什么num--会报错？ num--：num先参与运算，在减一，这样传递过去的一直是5，故--num才行，先自减1在参与运算。
    }
}
```

### 2.3 递归求阶乘
* **阶乘**：所有小于及等于该数的正整数的积。
```java
n的阶乘：n! = n * (n-1) *...* 3 * 2 * 1 
```
**分析**：这与累和类似,只不过换成了乘法运算，学员可以自己练习，需要注意阶乘值符合int类型的范围。
```
推理得出：n! = n * (n-1)!
```
代码实现如下：
~~~java
public class DemoRecursion03 {
    public static void main(String[] args) {
        int n = 5;
        System.out.println(n + "的阶乘为：" + getValue(n));
    }
    public static int getValue(int n){
        if (n == 1)
            return 1;
        return n * getValue(--n);
    }
}
~~~

### 2.4 递归打印多级目录
**分析**：多级目录的打印，就是当目录的嵌套。遍历之前，无从知道到底有多少级目录，所以我们还是要使用递归实现。

示例代码：
```java
public class RecursionPrintDir {
    public static void main(String[] args) {
        File dir = new File("D:\\IDEA_WorkSpace\\Demo\\");
        printDir(dir);
    }
    public static void printDir(File dir){
        File[] files = dir.listFiles(); // 获取文件或目录file对象
        for (File file:files){
            if (file.isFile()){
                System.out.println("文件名：" + file.getAbsolutePath()); //如果是文件则打印绝对路径
            }else{
                System.out.println("目录：" + file.getAbsolutePath());
                printDir(file); // 继续遍历，调用printdir，形成递归
            }
        }
    }
}
```

## 3. 综合案例
### 3.1 文件搜索
输入文件名称，输出文件路径
**分析**：

1. 目录搜索，无法判断多少级目录，所以使用递归，遍历所有目录。
2. 遍历目录时，获取的子文件，通过文件名称，判断是否符合条件。

代码实现：
```java
public class SearchFile {
    static String rootpath = "D:\\IDEA_WorkSpace\\Demo\\Demo5_5\\";
//    final static String filename = "FileFor.java";
    public static void main(String[] args) {
        System.out.println("输入你要搜索的文件名：");
        searchfile(new File(rootpath),new Scanner(System.in).next());
    }
    public static void searchfile(File dir,String filename){
        File[] files = dir.listFiles();
        for (File file:files){
            if(file.isFile()){
                if (file.getName().equals(filename) == true){
                    System.out.println(file.getAbsolutePath());
                }
            }else{
                searchfile(file,filename);
            }
        }
    }
}
```

> 注意：==和equals的区别！！！

### 3.2 文件过滤器的优化
`java.io.FileFilter`是一个接口，是File的过滤器。 该接口的对象可以传递给File类的`listFiles(FileFilter)` 作为参数， 接口中只有一个方法。

`boolean accept(File pathname)  ` ：测试pathname是否应该包含在当前File目录中，符合则返回true。

**分析**：

1. 接口作为参数，需要传递子类对象，重写其中方法。我们选择匿名内部类方式，比较简单。
2. `accept`方法，参数为File，表示当前File下所有的子文件和子目录。保留住则返回true，过滤掉则返回false。保留规则：
   1. 要么是.java文件。
   2. 要么是目录，用于继续遍历。
3. 通过过滤器的作用，`listFiles(FileFilter)`返回的数组元素中，子文件对象都是符合条件的，可以直接打印。

代码实现：
```java
public class DemoFileFilter {
    static String rootpath = "D:\\IDEA_WorkSpace\\Demo\\Demo5_5\\";
    public static void main(String[] args) {
        printDir(new File(rootpath));
    }
    public static void printDir(File dir){
        // 匿名内部类方式,创建过滤器子类对象
        File[] files = dir.listFiles(new FileFilter() {
            @Override
            public boolean accept(File pathname) {
                return pathname.getName().endsWith(".java")||pathname.isDirectory();// 过滤掉除了.java文件和目录以外的所有东西
            }
        });
        // 循环打印过滤后的数据
        for (File file:files){
            if (file.isFile()){
                System.out.println("文件名："+file.getAbsolutePath());
            }else{
                printDir(file);
            }
        }

    }
}
```

### 3.3 Lambda优化
代码示例如下：
```java
public static void printDir2(File dir){
        File[] files = dir.listFiles(f->{return f.getName().endsWith(".java") || f.isDirectory();});
        for (File file:files){
            if (file.isFile()){
                System.out.println("文件名："+file.getAbsolutePath());
            }else{
                printDir2(file);
            }
        }

    }
```

## 今日总结
1. Java中的File类
2. 文件和目录的创建和删除操作
3. File类的一些方法
4. 递归的概念以及使用
**遇到的问题**：
1. ==与equals的区别，使用过程中的注意事项
2. Lambda表达式还不熟练
3. 时间管理