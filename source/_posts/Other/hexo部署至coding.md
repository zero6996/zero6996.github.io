---
title: Hexo部署至coding
date: 2020-02-29 19:23:13
categories: Other
tags: [Hexo]
description: 将hexo部署到coding上，以及域名解析
---



# 前言

因本人的hexo是部署至github的，现在换了个主题图片有点多，导致访问速度非常慢，故想到部署到国内的服务器上，所以找到了coding，准备把hexo也同步部署到coding上去。


<!--more-->


本文章记录了如何将hexo部署至coding，从而加快国内站点的访问，本文前置条件：

1. 拥有hexo博客，会hexo的基本操作；
2. 拥有coding账户，且实名(未实名不能创建静态网站)。

### 1. coding配置

首先进行coding的配置，步骤如下：

1. 打开你的coding页面，新建一个项目；

![选择项目模板.png](https://yanxuan.nosdn.127.net/d518fdebcf6f8b7bd51bc9a9c92a835f.png)

> 注意：不要选择代码托管，这样不能创建静态页面

2. 填写项目基本信息；

![填写项目基本信息.png](https://yanxuan.nosdn.127.net/7574ea9e1ac663cc9f71c8817c8c7324.png)

一般都选择默认配置即可，点击完成创建。

3. 进入项目，点击代码仓库，找到项目的https链接，后面会用到。

![coding链接.png](https://yanxuan.nosdn.127.net/d1efb6e8e01170c6d47c7bfd65b4cffd.png)



### 2. hexo配置

接下来到hexo进行相关配置。

#### 2.1 安装部署服务器插件

打开你本地的hexo文件夹，然后打开Cmd或者GitBash窗口，输入以下命令安装插件：

```bash
npm install hexo-deployer-git --save
```

![安装插件](https://yanxuan.nosdn.127.net/f023d323295d1eabc3a38146fb88abcf.png)

我安装过了所以显示的不一样。

> 本人使用的cnpm是npm的淘宝镜像源，访问速度比npm快一些，比较推荐更换，[详见链接](http://npm.taobao.org/)



#### 2.2 修改站点配置文件

该文件位于主目录下，名称是`_config.yml`，使用任意编辑器打开。

![配置文件](https://yanxuan.nosdn.127.net/ce00ec9be9c7559c78832fdf9b952fa1.png)

打开后找到`deploy`，修改配置如下：

```yml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git # 部署类型
  repo: # 上传仓库地址
    github: https://github.com/zero6996/zero6996.github.io.git # github仓库地址
    coding: https://e.coding.net/zero024/hexo.git # coding仓库地址，将你的coding链接复制到这里。
  branch: master # 推送到远程的分支
```

> 注意yml的语法，缩进都是空两格，然后`：`冒号后面要空一格才能生效

#### 2.3 部署项目

回到命令行窗口，输入`hexo d`部署项目，如果没有问题的话应该显示有如下关键信息：

```bash
Branch 'master' set up to track remote branch 'master' from 'https://github.com/zero6996/zero6996.github.io.git'.
Everything up-to-date
Branch 'master' set up to track remote branch 'master' from 'https://e.coding.net/zero024/hexo.git'.
Everything up-to-date
INFO  Deploy done: git
```

#### 2.4 配置静态网站

回到coding的hexo项目，刷新一下可以看到代码仓库中已经有代码了，下面找到左侧`构建与部署`一栏，进行静态网站的相关配置。

![构建与部署](https://yanxuan.nosdn.127.net/3fa30fd26bca77eb12b130b0ebec5c8a.png)



点击`立即发布静态网站`，进入新建静态网站页面，网站名称任意，其他选项一般默认即可。

![新建静态网站](https://yanxuan.nosdn.127.net/db064e6bcd75810068ae6efe1ca70163.png)

保存后点击`立即部署`，即可部署项目到网站，coding给了个默认地址可以访问，点击即可查看静态网站部署效果。

![部署网站](https://yanxuan.nosdn.127.net/7a304592937bb1838a379a40fc70a440.png)

### 3. 域名配置

如果你拥有域名的话，可以绑定域名，具体操作如下。

#### 3.1 域名解析

本人使用的是阿里的域名，其他服务商的操作也大致相同。

首先进入域名解析页面，点击添加记录，设置一个CNAME，主机记录www，解析线路默认，记录值为你coding的默认访问地址；

![DNS解析](https://yanxuan.nosdn.127.net/240e822daae179f599361b6a24e8f5e3.png)

在添加一个CNAME，主机记录@，解析默认，记录值同上；

![DNS解析](https://yanxuan.nosdn.127.net/d7d950a269c87f8f1da8ff6cdde9433d.png)

这样DNS解析就设置完毕了，回到coding进行域名绑定，点击你的静态网站，右上角有个设置选项，可以进行很多设置。

![域名绑定](https://yanxuan.nosdn.127.net/402a34805f92ad3cebb0cbde31017365.png)

找到自定义域名，输入你的域名地址进行绑定，添加两个。

<img src="https://yanxuan.nosdn.127.net/245ca3613c374f27bbbaabd507b6ade2.png" alt="域名绑定"  />

大功告成！你可以使用你的域名快速的访问你的博客了。

