---
title: WebSphere自带jar包与应用jar包冲突的解决办法
date: 2018-06-01 10:45:36
tags:
- websphere
- was
categories:
- 学习笔记
- 部署相关
- WebSphere
keywords:
- websphere
- was
- jar包冲突
---

# 问题现象

在WebSphere8.5下安装我们的应用，一直提示NullPointerException，然而在Tomcat、Weblogic下都没有该问题。

分析错误日志，发现报错信息与log4j有关，而我们也确实使用了log4j，版本是2.10.0。

# 尝试解决

尝试着将log4j的jar包和slf4j的jar包从war包中删除，再安装，就启动成功了。此时再手动将之前移除的jar包放到应用的lib路径下，日志也可以正常打印了。

# 原因

原因可能是我们所使用的log4j、slf4j的jar包，websphere8.5本身提供，而且自带的jar包与我们使用的版本不一致。在默认情况下，容器会优先加载自带的jar包，从而导致应用启动失败，提示NullPointerException。

# 解决方案

<!-- more -->

## 新建共享库文件夹

在应用服务器上，新建共享库文件夹，将你希望优先加载的jar包放在该路径下。**如果是集群部署，需要在所有集群服务器上，都建立这样的文件夹，路径需要保持一致。**

## 在控制台中，新建共享库

点击左侧菜单【环境->共享库】
{% asset_img 2-1.png %}
点击【新建】按钮。类路径填写在上一步中新建的共享库文件夹路径
{% asset_img 2-2.png %}

## 在已安装的应用中，引入共享库

进入应用的配置界面，点击【共享库引用】
{% asset_img 3-1.png %}
选择应用程序或模块，点击【引用共享库】按钮
{% asset_img 3-2.png %}
选择需要引用的共享库
{% asset_img 3-3.png %}

## 安装应用过程中，引入共享库

选择详细安装
{% asset_img 4-1.png %}
在【步骤 4: 映射共享库】中，引入共享库
{% asset_img 4-2.png %}
{% asset_img 4-3.png %}