---
title: hexo+Git搭建个人博客 # 文章页面上的显示名称,一般是中文
date: 2018-12-04 09:48:13 # 文章生成时间,可任意修改
categories: hexo # 分类
tags: [hexo,Blog] # 文章标签,可空,多标签请用格式,注意:后面有空格
description: 使用hexo+git搭建免费的个人博客完整教程 # 附加一段文章摘要,字数最好140字以内,会出现在meta的description里面
---
## 前言
基于hexo+github pages服务搭建博客,快速,便捷,免费的搭建属于自己的个人博客

## 1.准备工作
在开始一切之前,你必须已经:
- 有一个github账号,没有的话去[注册一个](https://github.com)
- 电脑 [安装](http://www.runoob.com/nodejs/nodejs-install-setup.html) 了node.js,npm,并了解相关基础知识
- 安装了[git for windows](https://git-scm.com/downloads) (或其他git客户端工具)

<!--more-->

## 2.创建仓库
- 新建一个名为 用户名.github.io的仓库.比如说,如果你的github用户名是test
- 那么你就新建test.github.io的仓库(必须是你的用户名,其他的无效)
- 然后你的网站访问地址就是http://test.github.io 了
注意几个地方:
   - 注册的邮箱一定要验证,否则不会成功
   - 仓库名字必须是:username.github.io,其中username就是你的用户名
   - 仓库创建成功不会立即生效,需要过一段时间(等待github分配网站域名等)


## 3.绑定域名
- 域名配置最常见有2种方式,CNAME和A记录,CNAME填写域名,A记录填写IP
- 由于不带WWW方式只能采用A记录,所以先ping一下username.github.io的IP
- 然后到你的域名DNS设置页,将A记录指向你ping出来的IP,将CNAME指向:yourname.github.io
- 这样可以保证无论是否添加WWW都可以访问
- 例:
   - 记录类型:A      主机记录:@   解析线路默认  记录值:IP
   - 记录类型:CNAME  主机记录:www 解析线路默认  记录值:username.github.io
- 然后到你的github项目跟目录新建一个名为CNAME的文件(无后缀),里面填写你自定义的域名

## 4.配置SSH key
提交代码需要github权限才可以,这里使用ssh key来解决本地和服务器的连接问题
```bash
$ cd ~/.ssh # 检查本机已存在的ssh秘钥
```
如果提示:"No such file or directory"说明你是第一次使用git
```bash
ssh-keygen -t rsa -C 'your email@example.com'
```
一般情况下直接默认值就行,所以不用设置密码,一路回车就行
最终会在.ssh目录下生成两个文件,id_rsa和id_rsa.pub,这两个就是SSH Key的密钥对id_rsa是私钥,不能泄露!
```bash
cat id_rsa.pub
```
复制输出的内容,打开你的github主页,打开"Account settings">>"SSH keys"页面
1. 点击 'Add SSH Key',填上任意title,在Key文本框内粘贴id_rsa.pub文件的内容
2. 点击 'Add Key',就成功添加Key了
测试是否成功
```bash
$ ssh -T git@github.com
```
如果提示"Are you sure you want to continum connecting (yes/no)?" 输入yes,就会看到
```
Hi zero6996! You've successfully authenticated, but GitHub does not provide shell access.
```
说明SSH配置成功!
此时你还需要配置:
```bash
$ git config --global user.name 'zero6996' #你的github用户名
$ git config --global user.email 'youremail@xx.com' #你的github注册email
```
## 5.hexo简介
Hexo是一个简单,快速,强大的基于GitHub Pages的博客发布工具,支持Markdown格式,有众多优秀的插件和主题
[官网](http://hexo.io)
[Github](https://github.com/hexojs/hexo)
### 5.1.安装注意事项
1. 很多命令既可以用Windows的cmd来完成,也可以使用git bash来完成,但是部分命令会有一些问题,为避免不必要的问题,这里统一使用git bash来执行
2. hexo不同版本差别较大,网上很文章的配置信息都是基于2.x的,所以注意不要被误导
3. hexo有两种"_config.yml" 文件,一个是跟目录下的全局的"_config.yml" , 一个是各个"theme" 下的
4. 安装完node和npm以后,将npm替换为[淘宝镜像](https://npm.taobao.org/),执行以下命令即可将,后续命令全部以cnpm执行
```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
### 5.2.安装
```bash
$ cnpm install -g hexo
```
### 5.3.初始化
在本地新建一个名为hexo的文件夹(名字任意取),这里作为一个你存放代码的地方,比如我是D:\hexo
```bash
$ cd /d/hexo # 进入文件夹
$ hexo init # 初始化
```
hexo会自动下载一些文件到这个目录,包括node_modules,目录结构
```bash
$ ls
_config.yml  node_modules/  package-lock.json  scaffolds/  themes/
db.json      package.json   public/   source/
$ hexo g # 生成
$ hexo s # 启动服务
```

- 执行上述命令之后,hexo就会在public文件夹生成相关html文件,这些文件后续会提交到github上去
- "hexo s"是开启本地预览服务,打开浏览器访问[localhost:4000](http://localhost:4000)即可看到博客内容.
- 如果无显示内容或拒绝访问,就是端口占用的问题,重新开启服务,使用"hexo s -p 8888"指定端口

### 5.4.修改主题
hexo社区有丰富的主题可选,从[官网主题](https://hexo.io/themes/)自己挑选下载

下载主题(以[hexo-theme-yilia](https://github.com/litten/hexo-theme-yilia)为例):
```bash
$ cd /d/hexo/
$ git clone git@github.com:litten/hexo-theme-yilia.git themes/yilia # 从github克隆主题,到本地的themes/yilia文件夹
```
下载的主题都在hexo/themes文件夹内
```bash
$ cd themes
$ ls
landscape/  yilia/
```
修改"_config.yml"中的"theme: landscape"改为"theme: yilia",然后重新执行"hexo g"来生成页面
如果出现一些莫名其妙的问题,可以先执行"hexo clean"来清理一下public的内容,然后再来重新生成和发布

### 5.5.上传之前
上传代码到github之前,记得备份以前的所有代码,因为hexo提交代码时会把你以前的所有代码都删除
### 5.6.上传到Github
如果你一切都配置好了,发布上传只需要"hexo d"就可以了,当然关键是所有东西都配置好
首先,"ssh key"肯定要配置好
其次,配置"_config.yml"中有关deploy的部分:

```
#正确写法
deploy:
   type: git
   repository: git@github.com:zero6996/zero6996.github.io.git
   branch: master
```
如果报如下错误:
```
Deployer not found: github 或者 Deployer not found: git
```
原因是还需安装一个插件:
```
 cnpm install hexo-deployer-git --save
```
安装无误,输入github账户密码,输入"hexo d"就会将本次有改动的代码全部提交,没有改动的不会

### 5.7.常用hexo命令
```
hexo new 'postName' # 新建文章
hexo new page 'pageName' # 新建页面
hexo generate # 生成静态页面至public目录
hexo server # 开启预览访问端口(默认4000,使用-p可更换端口)
hexo deploy # 部署至Github
hexo help # 查看帮助
hexo version # 查看版本
```
以上命令均可缩写
```
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```
以及组合命令:
```
hexo s -g # 生成页面并本地预览
hexo d -g # 生成页面并上传
hexo s -p # 本地预览并改变端口
```

### 5.8._config.yml
这里面都是一些全局配置,每个参数意思都比较简单明了,需要注意的是冒号后面必须有一个空格

## 6.写博客
进入hexo根目录,执行命令:
```
hexo new 'my-first-blog'
```
hexo会帮我们在_posts下生成相关的md文件,我们只需要打开这个文件就可以开始写博客了

一般完整格式如下:
```
---
title: postName # 文章页面上的显示名称,一般是中文
date: 2018-12-5 15:46:10 # 文章生成时间,一般不用改,也可自定义
categories: 默认分类 # 分类
tags: [tag1,tag2,tag3] # 文章标签,可空,多标签请用格式
description: 附加一段文章摘要，字数最好在140字以内，会出现在meta的description里面
---

以下是正文内容
```
hexo new page 'my-second-blog' 生成的是:index.md,它不会作为文章出现在博客目录
### 6.1.写博客工具
   个人使用的是 'Visual Studio Code' IDE
### 6.2.如何让博文列表不显示全部内容
默认情况下,生成的博文目录会显示全部文章内容,如何设置文章摘要长度呢?

可以在合适的位置加上'<!--more-->'即可,显示在页面上的效果就是新增了一个'more>>'功能选项,查看更多内容
### 7.关于缺失模块
缺失模块
1. 请确保node版本大于6.2
2. 在博客根目录（注意不是yilia根目录）执行以下命令:
```bash
 cnpm i hexo-generator-json-content --save
```
3. 在根目录_config.yml里添加配置：
```bash
jsonContent:
    meta: false
    pages: false
    posts:
      title: true
      date: true
      path: true
      text: false
      raw: false
      content: false
      slug: false
      updated: false
      comments: false
      link: false
      permalink: false
      excerpt: false
      categories: false
      tags: true
```
#### 参考自
   [以上内容整理参考自文章](http://www.cnblogs.com/liuxianan/p/build-blog-website-by-hexo-github.html)