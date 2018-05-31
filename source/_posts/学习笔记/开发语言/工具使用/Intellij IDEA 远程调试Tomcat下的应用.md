---
title: Intellij IDEA 远程调试Tomcat下的应用
date: 2018-05-30 10:09:04
tags:
- intellij idea
- tomcat
categories:
- 学习笔记
- 开发语言
- 工具使用
---

1. 在Tomcat的bin目录下找到catalina.bat，搜索JPDA_ADDRESS，即为远程调试监听端口，默认为8000

{% codeblock %}
set JPDA_ADDRESS = 8000
{% endcodeblock %}

1. cd到bin目录下，运行catalina.bat，并开启远程调试功能

{% codeblock %}
catalina.bat jpda start
{% endcodeblock %}

1. 在intellij中，新建远程服务器

{% asset_img 3.png %}

1. 配置远程服务的ip和端口

{% asset_img 4.png %}

1. 配置远程调试监听端口

{% asset_img 5.png %}

1. 在intellij中启动即可
