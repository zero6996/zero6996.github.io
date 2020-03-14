---
title: Git # 文章页面上的显示名称,一般是中文
date: 2019-10-28 22:48:13 # 文章生成时间,可任意修改
categories: Other # 分类
tags: [Git]
description: git简单使用速查
---


## 1. Git简介

Git是目前世界上最先进的**分布式版本控制系统**。



<!--more-->



Linus大神为了解决Linux代码管理问题，在2005年花费两周时间用C编写了Git，此后迅速成为最流行的分布式版本控制系统。2008年，GitHub网站上线，它为开源项目免费提供Git存储，无数开源项目迁移至GitHub。

> 本文仅做Git最基本操作的学习，方便速查。详细教程请看廖雪峰[Git教程](https://www.liaoxuefeng.com/wiki/896043488029600)



### 1.1 安装Git

[进入官网](https://git-scm.com/)下载Git安装包，直接默认安装即可。

打开CMD，输入git，测试是否安装成功。



## 2. 远程仓库

使用远程仓库，需要先配置SSH。

### 2.1 配置SSH

SSH用于身份验证，我们首先需要生成一个SSH密钥，然后添加到GitHub中。

1. 打开Git Bash
2. 输入以下命令生成一个ssh密钥：`ssh-keygen -t rsa -C 'your email@example.com'`，一般情况下无需设置，一路回车就行。
3. 输入命令`cd ~/.ssh/`，进入ssh文件夹。里面有两个文件，`id_rsa`和`id_rsa.pub`，这两个就是SSH Key的密钥对。其中`id_rsa`是私钥，不能泄露！`id_rsa.pub`是公钥，可以公开使用。
4. 将SSH公钥复制到剪贴板：`clip < ~/.ssh/id_rsa.pub`
5. 登录GitHub，右上角点击个人头像，然后点击`settings`进入设置界面。
6. 点击`SSH and GPG keys`，然后点击`New SSH key`，填上任意title，在Key文本框内粘贴`id_rsa.pub`文件的内容，点击`Add Key`，就成功添加SSH key了！

### 2.2 克隆远程仓库

首先需要在GitHub创建一个仓库，直接点击`New repository`，然后根据提示即可创建仓库。接下来我们将远程仓库克隆到本地，进行操作。

1. 打开GitHub仓库界面，点击右边的`clone or download`，选择SSH方式，复制仓库地址。
2. 打开Git Bash
3. 输入`git git@github.com:zero6996/Learn_Git.git`，即可将远程仓库克隆到本地。



## 3. Git基本操作

- 初始化git，将当前文件夹交由git管理：`git init`

- 查看当前分支状态：`git status`
- 提交到暂存区：`git add <file>`
- 提交到仓库：`git commit file -m "message"`

### 3.1 配置用户信息

- 配置全局用户信息：`git config --system user.email youremail@email.com`
- 配置当前用户信息：`git config --global user.name yourname`

> 用户配置文件在`.git/config`

### 3.2 查看日志

- 查看当前工作日志：`git log`
- 查看历史工作日志：`git reflog`
- 查看更改日志：`git diff`

### 3.3 回退版本

- 退回到上N个版本：`git reset --hard HEAD^`，一个`^`代表退回一个版本，以此类推。

> HEAD指向的版本就是当前版本

- 退回到指定版本：`git reset --hard commit_id`，`commit_id`是指定版本号，举例`git reset --hard cbc5fdb`。

### 3.4 文件操作

- 移动文件：`git mv movefile_name targetdir/`
- 删除文件：`git rm filename`，如果该文件已提交到版本库中，则直接删除版本库中文件。

## 4. 分支管理

### 4.1 分支基本操作

- 查看分支：`git branch`
- 创建分支：`git branch <name>`
- 切换分支：`git checkout <name>`或者`git switch <name>`
- 创建+切换分支：`git checkout -b <name>`或`git switch -c <name>`
- 合并某分支到当前分支：`git merge <name>`
- 删除分支：`git branch -d <name>`

## 5. 标签管理

标签可以用于给版本库中的版本打标记，方便版本管理。

- 命令 `git tag <tagname> `用于新建一个标签，默认为HEAD，也可以指定一个commit_id。
- 命令 `git tag -a <tagname> -m 'message...`可以指定标签信息。
- 命令` git tag` 可以查看所有标签。

- 命令 `git push origin <tagname>` 可以推送一个本地标签到远程
- 命令 `git push origin --tags `可以推送本地全部未推送的标签到远程
- 命令 `git tag -d <tagname> `可以删除一个本地标签
- 命令 `git push origin :refs/tags/<tagname> `可以删除一个远程标签

