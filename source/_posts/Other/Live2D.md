---
title: Hexo博客添加live2d动态模型插件
date: 2019-11-04 14:12:13
categories: Other
tags: [Live2D]
description: Live2D插件的安装
urlname: Live2D
---

## 前言

live2d动态模型插件是一款非常有意思的插件。

使用和安装`helper-live2d`动态插件，需具备以下前提条件：


<!--more-->


1. hexo博客，没有搭建的可以看我以前的文章《[hexo+git搭建hexo个人博客](https://zero6996.github.io/2018/12/04/Other/hexo-git搭建个人博客/)》
2. Node.js环境和npm
3. 插件Github地址：[hexo-helper-live2d](https://github.com/EYHN/hexo-helper-live2d)
4. live2d模型仓库地址：[live2d-widget-models](https://github.com/xiazeyu/live2d-widget-models)以及部分模型[预览网址](https://huaji8.top/post/live2d-plugin-2.0/)



### 1. 安装模块

在hexo跟目录执行命令

```bash
cnpm install --save hexo-helper-live2d
```

### 2. 下载模型

可先查看模型预览：[hexo live2d插件 2.0](https://huaji8.top/post/live2d-plugin-2.0/)

#### 2.1 安装模型

使用`npm install {packagename}`安装单独模型，包名称列表如下：

- `live2d-widget-model-chitose`
- `live2d-widget-model-epsilon2_1`
- `live2d-widget-model-gf`
- `live2d-widget-model-haru/01`（使用`npm install --save live2d-widget-model-haru`）
- `live2d-widget-model-haru/02`（使用`npm install --save live2d-widget-model-haru`）
- `live2d-widget-model-haruto`
- `live2d-widget-model-hibiki`
- `live2d-widget-model-hijiki`
- `live2d-widget-model-izumi`
- `live2d-widget-model-koharu`
- `live2d-widget-model-miku`
- `live2d-widget-model-ni-j`
- `live2d-widget-model-nico`
- `live2d-widget-model-nietzsche`
- `live2d-widget-model-nipsilon`
- `live2d-widget-model-nito`
- `live2d-widget-model-shizuku`
- `live2d-widget-model-tororo`
- `live2d-widget-model-tsumiki`
- `live2d-widget-model-unitychan`
- `live2d-widget-model-wanko`
- `live2d-widget-model-z16`

举例本人安装的模型：

```bash
cnpm install live2d-widget-model-tororo
```

### 3. 进行详细配置

在Hexo的`_config.yml`文件下或主题的`_config.yml`文件中均可配置：

配置API查看：[live2d-widget.js API](https://l2dwidget.js.org/docs/class/src/index.js~L2Dwidget.html#instance-method-init)

#### 3.1 API配置

本人博客配置文件如下

```yml
# Live 2D settings
## 插件github地址：https://github.com/EYHN/hexo-helper-live2d
## API网址：https://l2dwidget.js.org/docs/class/src/index.js~L2Dwidget.html#instance-method-init
live2d:  
  enable: true
  scriptFrom: local # 默认
  pluginRootPath: live2dw/ # 插件在站点上的根目录(相对路径)
  pluginJsPath: lib/ # 脚本文件相对与插件根目录路径
  pluginModelPath: assets/ # 模型文件相对与插件根目录路径
  # scriptFrom: jsdelivr # jsdelivr CDN
  # scriptFrom: unpkg # unpkg CDN
  # scriptFrom: https://cdn.jsdelivr.net/npm/live2d-widget@3.x/lib/L2Dwidget.min.js # 你的自定义 url
  tagMode: false # 标签模式, 是否仅替换 live2d tag标签而非插入到所有页面中
  debug: true # 调试, 是否在控制台输出日志
  model:
    use: live2d-widget-model-tororo # npm-module package name
    # use: wanko # 博客根目录/live2d_models/ 下的目录名
    # use: ./wives/wanko # 相对于博客根目录的路径
    # use: https://cdn.jsdelivr.net/npm/live2d-widget-model-wanko@1.0.5/assets/wanko.model.json # 你的自定义 url
  display:
    position: right
    width: 145
    height: 315
  mobile:
    show: true # 是否在移动设备上显示
    scale: 0.5 # 移动设备上的缩放       
  react:
    opacityDefault: 0.7
    opacityOnHover: 0.8
```



> 本文内容参考摘录自文章[Hexo博客添加helper-live2d动态模型插件](https://joeybling.github.io/2019/05/05/Hexo%E5%8D%9A%E5%AE%A2%E6%B7%BB%E5%8A%A0helper-live2d%E5%8A%A8%E6%80%81%E6%A8%A1%E5%9E%8B%E6%8F%92%E4%BB%B6/)，欢迎大家关注该作者。

