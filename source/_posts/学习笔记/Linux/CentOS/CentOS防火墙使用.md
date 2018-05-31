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
---
1. 启动防火墙

查看防火墙是否启动
{% codeblock %}
systemctl status firewalld
{% endcodeblock %}

如果没有启动，启动
{% codeblock %}
systemctl start firewalld
{% endcodeblock %}

1. 开放端口命令

{% codeblock %}
firewall-cmd --zone=public --add-port=1. 1. /tcp --permanent
systemctl restart firewalld
{% endcodeblock %}

1. 关闭端口命令

{% codeblock %}
firewall-cmd --zone=public --remove-port=1. 1. /tcp -permanent
systemctl restart firewalld
{% endcodeblock %}

1. 查看开放端口命令

{% codeblock %}
firewall-cmd --zone=public --list-ports
{% endcodeblock %}
