---
title: CentOS防火墙使用
date: 2018-05-29 16:22:45
tags: 
- linux
- centos
- firewall
categories:
- 学习笔记
- Linux
- CentOS
keywords:
- linux
- centos
- firewall
---

# 启动防火墙

查看防火墙是否启动
{% codeblock lang:sh %}
systemctl status firewalld
{% endcodeblock %}
如果没有启动，启动
{% codeblock lang:sh %}
systemctl start firewalld
{% endcodeblock %}

# 开放端口命令

<!-- more -->
{% codeblock lang:sh %}
firewall-cmd --zone=public --add-port=1. 1. /tcp --permanent
systemctl restart firewalld
{% endcodeblock %}

# 关闭端口命令

{% codeblock lang:sh %}
firewall-cmd --zone=public --remove-port=1. 1. /tcp -permanent
systemctl restart firewalld
{% endcodeblock %}

# 查看开放端口命令

{% codeblock lang:sh %}
firewall-cmd --zone=public --list-ports
{% endcodeblock %}
