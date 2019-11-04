---
title: XML简单入门
date: 2019-6-7 20:30
categories: JavaWeb
description: XML的简单入门
---



## 1. XML基础

### 1.1 概念

XML指可扩展标记语言(Extensible Markup Language)，是一种**标记语言**，类似HTML。xml被设计用于**传输和存储数据**，而不是显示数据，标签都是自定义的。



<!--more-->



### 1.2 功能

- 存储数据：项目配置文件
- 传输数据：在网络中传输

### 1.3 xml与html的区别

1. xml标签都是自定义的，html标签是预定义
2. xml的语法严格，html语法松散
3. xml是存储数据的，html是展示数据

> W3C：万维网联盟，是Web技术领域最具权威和影响力的国际中立性技术标准机构

## 2. 语法

### 2.1 基本语法

1. xml文档的后缀名 `.xml`
2. xml第一行必须定义为文档声明：`<?xml version='1.0' ?>`
3. xml文档中有且仅有一个根标签
4. 属性值必须使用引号引起来
5. 标签必须正确关闭
6. xml标签名称区分大小写

#### 2.1.1 快速入门

```xml
<?xml version='1.0' ?>

<users>
    <user id="1">
        <name>张三</name>
        <age>20</age>
        <gender>男</gender>
    </user>
    <user id="2">
        <name>李四</name>
        <age>21</age>
        <gender>男</gender>
    </user>
</users>
```

### 2.2 组成部分

#### 2.2.1 文档声明

1. 格式：`<?xml 属性列表 ?>`
2. 属性列表：
   - `version`：版本号。必须的属性
   - `encoding`：编码方式。告知解析引擎当前文档使用的字符集，默认值`ISO-8859-1`
   - `standalone`：是否独立。`yes`表示不依赖其他文件，`no`表示依赖其他文件

#### 2.2.2 指令(了解即可)

可以结合css样式，解析页面，需加入声明：`<?xml-stylesheet type="text/css" href="style.css" ?>`

#### 2.2.3 标签

标签名称都是自定义的，不过有以下几点规则：

1. 名称可以包含字母、数字及其他的字符
2. 名称不能以数字或者标点符号开始
3. 名称不能以字母xml(或者XML、XmL等等)开始
4. 名称不能包含空格

#### 2.2.4 属性

id属性值唯一

#### 2.2.5 文本

- CDATA区：在该区域中的数据会被原样展示，格式：`<![CDATA[ 数据]]>`

### 2.3 约束

规定xml文档的书写规则，作为框架的使用者，我们只要能够在xml中引入约束文档，并能简单的读懂约束文档即可。

#### 2.3.1 分类

1. DTO：一种简单的约束技术
2. Schema：一种复杂的约束技术

#### 2.3.2 DTD

引入dtd文档到xml文档中

- 内部dtd：将约束规则直接定义在xml文档中
- 外部dtd：将约束规则定义在外部的dtd文件中
  - 引入**本地dtd**文档：`<!DOCTYPE 根标签名 SYSTEM "dtd文件位置"`
  - 引入**网络dtd**文档：`<!DOCTYPE 根标签名 PUBLIC "dtd文件名" "dtd文件URL"`



#### 2.3.3 Schema

引入

1. 填写xml文档的根元素
2. 引入xsi前缀：`xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"`
3. 引入xsd文件命名空间：`xsi:schemaLocation="http://www.zero.cn/xml  student.xsd"`
4. 为每一个xsd约束声明一个前缀，作为标识：`xmlns:z1="http://www.zero.cn/xml" `

## 3. 解析

操作xml文档，将文档中的数据读取到内存中

#### 3.1 主要操作

1. 解析(读取)：将文档中的数据读取到内存中
2. 写入：将内存中的数据保存到xml文档中。持久化存储



#### 3.2 解析xml的方式

1. DOM：将标记语言文档一次性加载进内存，在内存中形成一颗dom树
   - 优点：操作方便，可以对文档进行CRUD的所有操作
   - 缺点：占内存
2. SAX：逐行读取，基于事件驱动的。
   - 优点：不占内存
   - 缺点：只能读取，不能增删改



#### 3.3 xml常见的解析器

1. JAXP：sun公司提供的解析器，支持dom和sax两种思想
2. DOM4J：一款非常优秀的解析器
3. Jsoup：一款Java的HTML解析器，可直接解析某个URL地址、HTML文本内容。它提供了一套非常省力的API，可通过DOM，CSS以及类似于JQuery的操作方法来取出和操作数据。
4. PULL：Android操作系统内置的解析器，sax方式。



#### 3.4 Jsoup解析器

使用步骤：

1. 导入jar包
2. 获取Document对象
3. 获取对应的标签Element对象
4. 获取数据

#### 3.4.1 代码示例

```java
public class DemoJsoup01 {
    public static void main(String[] args) throws IOException {
        // 1.获取Document对象，根据xml文档获取
        // 1.1 获取users.xml的path
        String path = DemoJsoup01.class.getClassLoader().getResource("Demo6_7/users.xml").getPath();
        // 1.2 解析xml文档，加载文档进内存，获取dom树---> Document
        Document document = Jsoup.parse(new File(path), "utf-8");
        // 2. 获取所有name元素对象
        Elements elements = document.getElementsByTag("name");
        System.out.println(elements.size()); // 2
        // 2.1 获取第一个name的element对象
        Element element = elements.get(0);
        String name = element.text();
        System.out.println(name);
    }
}
```

#### 3.4.2 对象的使用

* Jsoup：工具类，可以解析html或xml文档，返回document
   - 主要方法：`parse`，解析html和xml文档，返回document
     - `parse(File in, String charsetName)`：解析xml或html文件的，最常用方法。
     - `parse(String html)`：解析xml或html字符串
     - `parse(URL url,int timeoutMillis)`：通过网络路径获取指定的html或xml的文档对象。
   
* Document：文档对象。代表内存中的dom树
   - 获取Element对象常用方法
     
     - `getElementById(String id)`：根据id属性值获取唯一的element对象
     - `getElementsByTag(String tagName)`：根据标签名称获取元素对象集合
  - `getElementsByAttribute(String key)`：根据属性名称获取元素对象集合
     - `getElementsByAttributeValue(String key, String value)`：根据对应的属性名和属性值获取元素对象集合
     
     **代码示例**：
     
     ```java
     public class DemoJsoup02 {
         public static void main(String[] args) throws IOException {
             // 1. 获取users.xml的path
             String path = DemoJsoup02.class.getClassLoader().getResource("Demo6_7/users.xml").getPath();
             // 1.1 解析xml文档，加载文档进内存，获取dom树
             Document document = Jsoup.parse(new File(path), "utf-8");
             // 2. 获取所有的user元素对象
             Elements users = document.getElementsByTag("user");
             System.out.println(users);
             System.out.println("----------------------------");
             // 3.获取属性名为id的元素对象们
             Elements ids = document.getElementsByAttribute("id");
             System.out.println(ids);
             System.out.println("----------------------------");
             // 4. 获取uid属性值为1的元素对象们
             Elements uid = document.getElementsByAttributeValue("uid", "1");
             System.out.println(uid);
             System.out.println("----------------------------");
             // 5. 获取id属性值为“z1”的元素对象
             Element z1 = document.getElementById("z1");
             System.out.println(z1);
         }
     }
     ```
   
* Elements：元素Element对象的集合。可以当做`ArrayList<Element>`来使用

* Element：元素对象
   1. 获取子元素对象
      - `getElementById(String id)`：根据id属性值获取唯一的element对象
      - `getElementsByTag(String tagName)`：根据标签名称获取元素对象集合
      - `getElementsByAttribute(String key)`：根据属性名称获取元素对象集合
      - `getElementsByAttributeValue(String key, String value)`：根据对应的属性名和属性值获取元素对象集合
   2. 获取属性值
      - `String attr(String key)`：根据属性名称获取属性值
   3. 获取文本内容
      - `String text()`：获取文本内容
      - `String html()`：获取标签体的所有内容(包括子标签的字符串内容)

   **代码示例**:

   ```java
   public class DemoJsoup03 {
       public static void main(String[] args) throws IOException {
           // 1. 获取student.xml的path
           String path = DemoJsoup03.class.getClassLoader().getResource("Demo6_7/users.xml").getPath();
           // 1.1 解析xml文档，加载文档进内存，获取dom树---> Document
           Document document = Jsoup.parse(new File(path), "utf-8");
           // 2. 获取第一个user元素对象
           Element user = document.getElementsByTag("user").get(0);
           // 2.1 根据元素对象获取下面的子元素name
           Elements name = user.getElementsByTag("name");
           // 2.2 根据属性名称获取属性值
           String id = name.attr("id");
           System.out.println(id); // z1
           // 2.3 获取name元素的文本内容
           System.out.println(name.text()); // 张三
           // 2.4 获取第一个user元素对象下面的所有标签体内容
           System.out.println(user.html());
       }
   }
   ```

* Node：节点对象

   - 是Document和Element的父类

#### 3.4.3 快捷查询方式

* selector：选择器

   - `Elements select(String cssQuery)`：语法类似CSS的元素选择器，具体使用参考[Selector类](https://jsoup.org/apidocs/org/jsoup/select/Selector.html)中定义的语法

   ```java
   public class DemoJsoup04 {
       public static void main(String[] args) throws IOException {
           // 1. 获取student.xml的path
           String path = DemoJsoup04.class.getClassLoader().getResource("Demo6_7/users.xml").getPath();
           // 2. 解析xml文档，加载文档进内存，获取dom树
           Document document = Jsoup.parse(new File(path), "utf-8");
           // 3. 查询name标签
           Elements names = document.select("name");
           System.out.println(names);
           // 4. 查询id值为“z1”的元素
           Elements ids = document.select("#z1");
           System.out.println(ids);
           // 5. 获取user标签且uid属性值为1的age子标签
           Elements age = document.select("user[uid='1'] > age");
           System.out.println(age);
       }
   }
   ```

* XPath：

XPath即为XML路径语言，它是一种用来确定XML(标准通用标记语言的子集)文档中某部分位置的语言

- 使用Jsoup的Xpath需额外导入jar包
- Xpath使用语法查询w3cshool的[参考手册](http://www.w3school.com.cn/xpath/xpath_syntax.asp)

**代码示例**：

```java
public class DemoJsoup05 {
    public static void main(String[] args) throws IOException, XpathSyntaxErrorException {
        // 1. 获取student.xml的path
        String path = DemoJsoup05.class.getClassLoader().getResource("Demo6_7/users.xml").getPath();
        // 2. 解析xml文档，加载文档进内存，获取dom树
        Document document = Jsoup.parse(new File(path), "utf-8");
        // 3. 根据document，创建JXDocument对象
        JXDocument jxDocument = new JXDocument(document);
        // 4. 结合xpath语法查询
        // 4.1 查询所有的user标签
        List<JXNode> jxNodes = jxDocument.selN("//user");
        for (JXNode jxNode : jxNodes) {
            System.out.println(jxNode);
        }
        // 4.2 所有user标签下的name标签
        List<JXNode> jxNames = jxDocument.selN("//user/name");
        for (JXNode jxName : jxNames) {
            System.out.println(jxName);
        }
        // 4.3 查询user标签下带有id属性的name标签
        List<JXNode> jxId = jxDocument.selN("//user/name[@id]");
        for (JXNode jxNode : jxId) {
            System.out.println(jxNode);
        }
        // 4.4 查询user标签下id属性值为“z1”的name标签
        List<JXNode> jxIdisz1 = jxDocument.selN("//user/name[@id='z1']");
        for (JXNode jxNode : jxIdisz1) {
            System.out.println(jxNode);
        }
    }
}
```

