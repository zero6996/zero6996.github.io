---
title: Maven
date: 2019-7-16 23:00
categories: JavaWeb
tags: [Maven]
description: Maven基本概念和使用
---


## 1. `Maven`基础概念和配置

Maven是一个项目管理工具，它包含了一个项目对象模型(`POM:Project Object Model`)，一组标准集合，一个项目生命周期(`Project Lifecycle`)，一个依赖管理系统(`Dependency Management System`)，和用来运行定义在生命周期阶段(`phase`)中插件(`plugin`)目标(`goal`)的逻辑。



<!--more-->





### 1.1 `Maven`能解决的问题

1. 依赖管理
2. 编译代码
3. 单元测试
4. 打包项目



### 1.2 安装与配置



#### 1.2.1 官网下载`Maven`

官网下载[链接](http://maven.apache.org/download.cgi)

点击`apache-maven-3.6.1-bin.zip`下载

![downloadMaven](D:\资料\Java\downloadMaven.png)


#### 1.2.2 解压

![jieya](D:\资料\Java\img\jieya.png)



#### 1.2.3 配置环境变量

> 注意：安装maven之前，确保电脑中已安装JDK，如安装Maven3则必须是JDK1.7以上。

新建一个系统变量，变量名：`MAVEN_HOME`，变量值为你解压的文件路径，例我的路径是：`D:\Maven\apache-maven-3.6.1`



![add_variable](D:\资料\Java\img\add_variable.png)





添加到系统路径中：



![addToPath](D:\资料\Java\img\addToPath.png)



完成后打开`cmd`输入`mvn -v`，显示版本信息则配置正确。



![mvn](D:\资料\Java\img\mvn.png)



### 1.3 本地仓库配置

仓库分三类：本地仓库，远程仓库[私服]，中央仓库。

- 本地仓库：用于存储从远程仓库或中央仓库下载的插件和jar包，项目使用的一些插件或jar包。优先从本地仓库查找，默认本地仓库位置在`${user.home}/.m2/repository`，`${user.home}`表示windows用户目录。

在`maven`的`conf`文件夹下的`settings.xml`文件中配置本地仓库。

![1563242920527](C:\Users\14908\AppData\Roaming\Typora\typora-user-images\1563242920527.png)

- 远程仓库：又叫私服仓库，如果私服仓库存在，且在本地仓库所需的插件或jar包没有的情况下，会从当前仓库下载。
- 中央仓库：在maven软件中内置一个远程仓库地址`http://repo1.maven.org/maven2`。它是中央仓库(`Central Repository`)，服务于整个互联网，由Maven官方团队维护，里面存储了非常全面的jar包，包含了世界上大部分主流的开源项目构件。



## 2. `Maven`工程的认识

### 2.1 `Maven`工程的目录结构

- 工程目录结构：

![mavenProject](D:\资料\Java\img\mavenProject.png)

作为一个`Maven`工程，它的`src`目录和`pom.xml`是必备的。

- `target`：项目输出位置，编译后的`class`文件会输出到此目录。
- `pom.xml`：`maven`项目核心配置文件。

- `src`目录结构：

![src](D:\资料\Java\img\src.png)

- 标准`src`目录结构：

  - `src/main/java`：存放项目的`.java`文件。
  - `src/main/resources`：存放项目资源文件，如`spring,hibernate`配置文件。
  - `src/test/java`：存放所有单元测试`.java`文件，如`Junit`测试类。
  - `src/test/resources`：测试资源文件。



- 运行项目：进入maven工程目录(当前目录有`pom.xml`)，打开`cmd`，运行`mvn tomcat:run`命令即可。



## 3. `Maven`常用命令

我们可以在`cmd`中通过一系列的`maven`命令来对我们的`maven`工程进行编译、测试、运行、打包、安装、部署。

### 3.1 `compile`

`compile`是`maven`工程的编译命令，作用是将`src/main/java`下的文件编译为`class`文件输出到`target`目录下。



### 3.2 `test`

`test`是`maven`工程的测试命令，执行`mvn test`命令，会执行`src/test/java`下的单元测试类。



### 3.3 `clean`

`clean`是`maven`工程的清理命令，执行`mvn clean`命令会删除`target`目录及内容。



### 3.4 `package`

`package`是`maven`工程的打包命令，对于`java`工程执行该命令会打包成`jar`包，`web`工程则会打包成`war`包。



### 3.5 `install`

`install`是`maven`工程的安装命令，执行该命令会将`maven`打包成`jar`包或`war`包发布到本地仓库。



### 3.6 `Maven`指令的生命周期

`maven`对项目构建过程分为三套相互独立的生命周期，请注意是***三套***且***相互独立***的。

这三套生命周期分别是：

1. `Clean Lifecycle`：在进行真正的构建之前进行一些清理工作。
2. `Default Lifecycle`：构建的核心部分，进行编译、测试、打包、部署等等。
3. `Site Lifecycle`：生成项目报告、站点，发布站点。



### 3.7 `maven`的概念模型

`Maven`包含了一个项目对象模型(`POM:Project Object Model`)，一组标准集合，一个项目生命周期(`Project Lifecycle`)，一个依赖管理系统(`Dependency Management System`)，和用来运行定义在生命周期阶段(`phase`)中插件(`plugin`)目标(`goal`)的逻辑。

![maven_model](D:\资料\Java\img\maven_model.png)



- 项目对象模型：一个`maven`工程都有一个`pom.xml`文件，通过该文件定义项目的坐标、项目依赖、项目信息、插件目标等。

- 依赖管理系统：通过maven的依赖管理对项目所依赖的jar包进行统一管理。

  - 如：项目依赖`junit4.9`，通过在`pom.xml`中定义`junit4.9`的依赖即使用`junit4.9`，如下所示`junit4.9`的依赖定义：

  ```xml
  <dependencies>
      <!-- 此项目运行使用junit，所以此项目依赖junit -->
      <dependency>
          <!-- junit的项目名称 -->
          <groupId>junit</groupId>
          <!-- junit的模块名称 -->
          <artifactId>junit</artifactId>
          <!-- junit版本 -->
          <version>4.9</version>
          <!-- 依赖范围：单元测试时使用junit -->
          <scope>test</scope>
      </dependency>
  </dependencies>
  ```

- 一个项目生命周期：使用maven完成项目的构建，项目构建包括：清理、编译、测试、部署等过程。maven将这些过程规范为一个生命周期，如下所示：

![lifecycle](D:\资料\Java\img\lifecycle.png)

- 一组标准集合：maven将整个项目管理过程定义一组标准，比如：通过maven构建工程有标准的目录结构，标准的生命周期阶段、依赖管理和标准的坐标定义等。
- 插件(`plugin`)目标(`goal`)：maven管理项目生命周期过程都是基于插件完成的。



## 4. IDEA开发`maven`项目



### 4.1 IDEA的`maven`配置

打开`-->File-->Settings`，搜索maven，配置如下内容：



![IDEASettingMaven](D:\资料\Java\img\IDEASettingMaven.png)



### 4.2 在IDEA中创建一个`maven`的`web`工程

新建工程，选择IDEA提供好的maven的web工程模板



![create_project](D:\资料\Java\img\create_project.png)

点击`Next`填写项目信息



![create_project02](D:\资料\Java\img\create_project02.png)



点击`Next`，配置相关属性，此处不做改动。



![create_project03](D:\资料\Java\img\create_project03.png)



再点击`Next`选择项目所在目录，最后点击`Finish`，等待项目构建。显示如下信息则构建成功。



![create_project04](D:\资料\Java\img\create_project04.png)



最终目录结构并不完整，需手动补齐。手动添加`src/main/java`目录，将`java`目录设置为`Sources Root`。

#### 4.2.1 创建一个Servlet

```java
package com.zero.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/testServlet")
public class TestServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.getRequestDispatcher("/hello.jsp").forward(request,response);
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```

#### 4.2.2  配置`pom.xml`

此时编译器会提示servlet相关包不存在，需要在`pom.xml`中添加坐标。添加jar包坐标时，还可指定该jar包的作用范围。

```xml
<dependencies>
<!--    放置项目运行所依赖的jar包-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.0</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

依赖范围表：

| 依赖范围   | 编译时有效 | 测试时有效 | 运行时有效 | 例                          |
| ---------- | ---------- | ---------- | ---------- | --------------------------- |
| `compile`  | Y          | Y          | Y          | `spring-core`               |
| `test`     | -          | Y          | -          | `Junit`                     |
| `provided` | Y          | Y          | -          | `servlet-api`               |
| `runtime`  | -          | Y          | Y          | JDBC驱动                    |
| `system`   | Y          | Y          | -          | 本地的，Maven仓库之外的类库 |

`scope`范围依赖小结：

- 默认引入的jar包使用`compile`[默认范围，可不写,编译、测试、运行都有效]
- `servlet-api、jsp-api`使用`provided`[编译、测试有效，运行时无效，防止和tomcat下jar包冲突]
- JDBC驱动包使用`runtime`[测试、运行有效]
- `Junit`使用`test`[仅测试有效]

依赖范围由强到弱的顺序是：`compile>provided>runtime>test`



#### 4.2.3 运行

点击`M`图标，输入`tomcat7:run`运行maven项目。

![pom](D:\资料\Java\img\pom.png)



#### 4.2.4 配置断点调试

在项目配置中添加maven项目，设置操作命令。

![debug](D:\资料\Java\img\debug.png)

点击小虫子，即可启动`debug`模式。

![debug2](D:\资料\Java\img\debug2.png)





## 5. 总结



### 5.1 `maven`仓库

1. `maven`仓库的类型有哪些？
2. `maven`工程查找仓库的流程是什么？
3. 本地仓库如何配置？



### 5.2 常用的`maven`命令

- `compile`：编译
- `clean`：清理
- `test`：测试
- `package`：打包
- `install`：安装





### 5.3 坐标定义

在`pom.xml`中定义坐标，内容包括：`groupId、artifactld、version`，示例如下：

```xml
<!--项目名称，定义为组织名+项目名，类似包名-->
<groupId>com.zero.maven</groupId>
<!--模块名称-->
<artifactId>maven-first</artifactId>
<!--当前项目版本号，snapshot为快照版本即非正式版本，release为正式发布版本-->
<version>1.0-SNAPSHOT</version>
<!--打包类型-->
<packaging>war</packaging>
	<!--
		jar:执行package会打包成jar包
 		war：执行package会打包成war包
		pom：用于maven工程的继承，通常父工程设置为pom
	-->
```



### 5.4 `pom`基本配置

`pom.xml`是`Maven`项目的核心配置文件，位于每个工程的根目录，基本配置如下：

~~~xml
<project>文件的根节点</project>
<modelversion>pom.xml使用的对象模型版本</modelversion>
<groupId>项目名称，一般写项目的域名</groupId>
<artifactld>模块名称或子项目名</artifactld>
<version>产品的版本号</version>
<packaging>打包类型，有jar、war、pom等</packaging>
<name>项目的显示名，常用于Maven生成的文档</name>
<description>项目描述，常用于Maven生成的文档</description>
<dependencies>项目依赖构件配置，配置项目依赖构件的坐标</dependencies>
<build>项目构建配置，配置编译、运行插件等</build>
~~~

