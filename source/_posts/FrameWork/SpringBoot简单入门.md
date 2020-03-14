---
title: SpringBoot简单入门
date: 2020-3-8 09:00
categories: FrameWork
tags: [SpringBoot]
description: SpringBoot的简单入门和一些遇到的问题及解决方法
---



## SpringBoot简单入门

Spring是一个JavaWeb开发框架，极大的简化开发，降低了对配置文件的要求；能迅速的开发web应用，几行代码就可以开发一个http接口。



<!--more-->



### 1. 创建简单项目

1. 点击菜单->New->Project->Spring Initalizr 然后点击Next

![New Project](http://yanxuan.nosdn.127.net/b9e28697b634eab06db8a659ccede011.png)

2. 输入如图所示两个地方参数，其他参数一般默认，然后Next

![New Project](http://yanxuan.nosdn.127.net/a2e77ddd87976719e7a10347a0927741.png)

3. 接着选择Web,然后勾选Spring Web即可，Next

![选择Web](http://yanxuan.nosdn.127.net/075b6bf7031e1a28d641e9ee64fc64f7.png)

4. 最后确认一下项目名称和位置即可，点击Finish完成项目创建。
5. 项目创建好后，自带一个Application，在里面填写如下内容

![建立简单映射](http://yanxuan.nosdn.127.net/cb1facc5ce154216aeae0a72342b3210.png)

6. 启动Application项目，测试访问。



### 2. 关于打包问题

通常部署SpringBoot项目可以采用两种方式：全部打包成一个jar包，或者打包成一个war包。

#### 2.1 打包成jar包

使用IDEA的话非常简单，找到右侧侧边栏的maven图标，点击项目生命周期，然后双击`install`即可，没问题应该显示`BUILD SUCCESS`。

![Maven](http://yanxuan.nosdn.127.net/1c7d5f6b064fdb92b777e4b6d4485677.png)

> 使用maven的`mvn install`一样可以，该命令需在项目根目录打开命令行窗口下运行。

该命令会在你项目的target目录下生产一个jar文件，使用命令`java -jar yourjarName.jar`运行该jar文件即可启动项目。

![运行jar](http://yanxuan.nosdn.127.net/d51e68036cb149702808e7bf79d388e3.png)

这个jar包里面包含项目的全部文件，以及Tomcat服务器，所以无论放到哪里只要有java环境就可以运行启动项目。



#### 2.2 打包成war包

1. 修改`pom.xml`，添加打包类型和依赖

```xml
<!-- 打包成war包需加这一行-->
<packaging>war</packaging>

<!-- 打包从war包时需添加此依赖,防止打包冲突-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope> <!-- 表示只在编译和测试过程有效，生成war包不会加入-->
</dependency>
```

2. 修改启动类文件，继承` SpringBootServletInitializer `

```java
@SpringBootApplication
public class HelloDemoApplication extends SpringBootServletInitializer {

	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
		return builder.sources(HelloDemoApplication.class);
	}

	public static void main(String[] args) {
		SpringApplication.run(HelloDemoApplication.class, args);
	}

}
```

> SpringBoot项目想运行在第三方Tomcat上必须实现`SpringBootServletInitializer`接口的`configure`方法才能让外部容器运行。

3. 使用maven的`clean`命令，然后使用`package`命令，就会在target目录下生成`.war`文件

![生成war](http://yanxuan.nosdn.127.net/17f5a60920545e0780cf37f492cf7689.png)

4. 将该文件复制到外部Tomcat的webapps目录下，重命名为`ROOT.war`，方便访问。

> 不修改为ROOT，需要加项目名称才可以访问。

![部署war](http://yanxuan.nosdn.127.net/4acfb821a780f0388eb6355810d275dd.png)

5. bin目录下点击`startup.bat`文件，启动项目，测试访问。

![运行war](http://yanxuan.nosdn.127.net/84c10736da31d123eb3761476362d5af.png)



### 3. 关于Tomcat中文乱码问题

在tomcat的conf文件夹下，打开`logging.properties`文件，将`java.util.logging.ConsoleHandler.encoding = UTF-8`修改`java.util.logging.ConsoleHandler.encoding = GBK`

![解决乱码](http://yanxuan.nosdn.127.net/cc6dbe40be3cc9beb0dedbd015abfd7c.png)



### 4. SpringBoot热部署

1. 在settings中开启` Build project automatically `，自动构建项目功能。

![UTOOLS1583487581224.png](http://yanxuan.nosdn.127.net/8a4d3961d543c268a9f3831b8f8bd043.png)

2. 开启动态自动编译，同时按住`Ctrl+Shift+Alt+/`，然后点击`Registry`，勾选如下参数，并配置时间。

![UTOOLS1583487512368.png](http://yanxuan.nosdn.127.net/8a6d4c9a57fd673d6f8a11d3b0a6d9fe.png)

3. 开启项目的热部署策略，在项目配置中选择如下选项即可。

![UTOOLS1583487836015.png](http://yanxuan.nosdn.127.net/7e0d63ba102733533ac83ee8e86f9efb.png)

4. 项目添加热部署插件，在xml中配置如下。

```xml
<!-- 热部署依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional> <!-- true才生效 -->
    <scope>runtime</scope> <!--仅在运行期有效，不参与打包-->
</dependency>
```

5. 保险起见，浏览器f12 ✅Disable cache。

### 5. 关于错误处理

定义一个全局的异常处理器，然后跳转到错误页面

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(value = Exception.class)
    public ModelAndView defaultErrorHandler(HttpServletRequest req,Exception e) throws Exception{
        ModelAndView view = new ModelAndView();
        view.addObject("exception",e);
        view.addObject("url",req.getRequestURL());
        view.setViewName("errorPage");
        return view;
    }
}
```

