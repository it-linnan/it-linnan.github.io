---
title: Linux下war包操作
date: 2018-05-29 16:54:56
tags:
- linux
- war
categories:
- 学习笔记
- Linux
- 通用
keywords:
- linux
- war
---

# Unzip
{% codeblock lang:sh %}
unzip -oq common.war -d common
{% endcodeblock %}

# jar

{% codeblock lang:sh %}
jar -xvf game.war
{% endcodeblock %}