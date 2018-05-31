---
title: Linux下端口进程互查
date: 2018-05-29 16:54:46
tags:
- linux
categories:
- 学习笔记
- Linux
- 通用
---
1. 先查看进程pid

{% codeblock %}
ps -ef |grep 进程名
{% endcodeblock %}

1. 通过pid查看占用端口

{% codeblock %}
netstat -nap |grep 进程id
{% endcodeblock %}

1. 通过端口查看进程

{% codeblock %}
netstat -nap |grep 端口号
{% endcodeblock %}
