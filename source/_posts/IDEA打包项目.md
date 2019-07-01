---
title: IDEA打包项目
date: 2019-7-1 22:30
categories: JavaWeb
tags: [项目部署]
description: 将IDEA中的项目打包成war包，方便部署。
---



### 如何将IDEA中的项目打包成war包？

<!--more-->

#### 1. 打开项目结构，选择`Artifacts`，添加一个空的`web`应用

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/07/01/1561992152052-1561995934782.png)

#### 2. 改名

按需改名


![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/07/01/1561992273664-1561995961288.png)


#### 3. 创建`META-INF`

点击创建，选择项目跟目录，即可创建成功。

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/07/01/1561992318178-1561995972034.png)


#### 4. 导入项目文件

选中项目，点击`Put into Output Root`，即可将项目相关的`lib`包和`class`文件全部导入。

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/07/01/1561992419651-1561995980538.png)




导入项目文件后，会自动生成为如下目录结构：

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/07/01/1561992441169-1561995987803.png)


#### 5. 构建项目

上述步骤完成后点击`OK`，返回主界面将项目构建一下。

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/07/01/1561992635331-1561995998209.png)

![title](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/07/01/1561992656642-1561996005603.png)



#### 6. 查看war包

在你项目的`artifacts`文件夹下，有你刚刚构建的项目文件夹，里面便是构建好的`war`包。