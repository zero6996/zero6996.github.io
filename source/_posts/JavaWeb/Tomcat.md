---
title: Tomcat服务器
date: 2019-6-8 22:50
categories: JavaWeb
description: Tomcat服务器的简单使用和项目部署
urlname: tomcat
---



## 1. Web服务器软件

- 安装了服务器软件的计算机就是**服务器**

- 服务器软件：接收用户的请求，处理请求，返回响应
- Web服务器软件：接收用户的请求，处理请求，做出响应。在web服务器软件中，可以部署web项目，让用户通过浏览器来访问这些项目



<!--more-->



### 1.1 常见的Java相关的Web服务器软件

1. WebLogic：Oracle公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
2. WebSphere：IBM公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
3. JBOSS：JBOOS公司的，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
4. Tomcat：Apache基金组织，中小型的JavaEE服务器，仅仅支持少量的JavaEE规范Servlet/jsp。开源免费的



> Tips：**JavaEE**是Java语言在企业开发中使用的技术规范的总和，一共规定了13项大的规范。



### 1.2 Tomcat

#### 1.2.1 下载

1. 前往Tomcat官网[下载](https://tomcat.apache.org/download-80.cgi)安装版：`Windows Service Installer`，也可以下载解压版。

#### 1.2.2 安装

1. 安装版点击安装即可，软件会自动注册服务
2. 解压版解压即可使用

##### Tomcat目录结构：

![path](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/06/08/tomcat%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84-1560005163914.png)

> Notice：注意，安装或解压的路径不要有中文

#### 1.2.3 卸载

1. 安装版进入文件夹点击`Uninstall.exe`即可卸载软件
2. 解压版直接删除文件夹即可

#### 1.2.4 启动

安装版会自动注册并启动服务，无需手动启动

解压版启动方式：

1. 进入解压后的文件夹，进入bin目录，双击`startup.bat`运行该文件
2. 访问：浏览器输入：`http://localhost:8080` 即可本地访问

#### 1.2.5 关闭

解压版关闭

1. `bin/shutdows.bat`，双击即可关闭服务
2. 直接关闭窗口或者`ctrl+c`

根据进程PID关闭

1. `cmd`输入`netstat -ano`，打印当前运行进程PID等信息，找到本地地址端口号8080的，查看其PID。
2. 打开任务管理器，进程显示PID，然后根据PID号直接结束进程。

>  Tips: windows下删除服务命令：`sc delete 服务名称`

[安装参考文章](https://blog.csdn.net/u011982967/article/details/80999552)

#### 1.2.6 配置

##### 项目部署的方式

1. 直接将项目文件夹放在webapps目录下即可，也可将项目打包成一个war包，再将war包放入wabapps目录下，war包会自动解压。
2. 配置`conf/server.xml`文件完成部署，在`<Host>`标签体中配置：`<Context docBase="D:\Project" path="/index" />`；docBase：项目存放的本地路径，path：虚拟目录。

3. 在`conf\Catalina\localhost\`下创建任意名称(文件名称即虚拟路径)的xml文件。在文件中编写`<Context docBase="D:\Project />"`

##### 静态项目和动态项目

```
Java动态项目目录结构：
-- 项目名称
    -- WEB-INF
    	-- web.xml：该项目的核心配置文件
    	-- classes目录：放置字节码文件
    	-- lib目录：放置项目依赖的jar包
```
