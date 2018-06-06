---
title: Ubuntu离线安装软件包
date: 2018-05-29 16:39:42
tags:
- linux
- ubuntu
- deb
categories:
- 学习笔记
- Linux
- Ubuntu
keywords:
- linux
- ubuntu
- deb
- 离线安装
---
Ubuntu软件包格式为deb

离线安装命令
{% codeblock lang:sh %}
sudo dpkg -i xxx.deb
{% endcodeblock %}

PS:使用apt-get命令安装软件时，所有下载的deb包都缓存到了/var/cache/apt/archives目录下
