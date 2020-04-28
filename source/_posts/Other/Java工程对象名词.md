---
title: Java工程对象名词
date: 2020-04-20 16:40
categories: Other
tags: [Java Noun]
urlname: Java-Noun
---



## Java工程中各种O对象的概念

### 1. PO

Persistent Object，持久对象。

与数据库里表字段一一对应，映射了数据库中对应的表。PO由一些属性，以及set/get方法组成；一般情况下，一个表对应一个PO。

### 2. VO

Vlue Object，又名表现层对象(View Object)。

通常用于业务之间的数据传递，和PO一样也是仅仅包含数据而已。但应是抽象出的业务对象，可以和表对应，也可以不用，具体根据业务的需求。对于页面上要展示的对象，可以封装成一个VO对象，将所需数据封装进去。

### 3. BO

Bussiness Object，业务对象。

封装业务逻辑的Java对象，通过调用DAO方法，结合PO、VO进行业务操作。一个BO对象可以包括多个PO对象。如常见的工作简历栗子，简历可以理解为一个BO，简历中又包括工作经历，项目经历等，这些可以理解为一个个PO，由这些PO组成BO。

### 4. DAO

Data Access Object，数据访问对象。

此对象专门用于访问数据库，通常和PO结合使用。DAO中包含了各种操作数据库的方法；通过这些方法，结合PO对数据库进行相关的CURD操作，夹在业务逻辑与数据库资源中间。

### 5. DTO

Data Trasfer Object，数据传输对象。

主要用于远程调用等需要大量传输对象的地方。比如有一张表有100个字段，那么对应的PO就会有100个属性；但是页面上只需显示10个字段，客户端用Web服务来获取数据，没必要将整个PO对象传递到客户端。

这时就可以用只有这10个属性的DTO来传递结果到客户端，这样也不会暴露服务端表结构，同时达到客户端传输数据的要求。如果用这个对象来对应页面显示，那此时它的身份就转为了VO。

### 6. POJO

Plain Ordinary Java Object，简单无规则的Java对象，就是传统意义上普通的Java对象。





[本文摘录自](https://www.cnblogs.com/shilei-ysl/p/11032304.html)

