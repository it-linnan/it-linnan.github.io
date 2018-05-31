---
title: Ubuntu开启SSH服务
date: 2018-05-29 16:35:01
tags:
- linux
- ubuntu
- ssh
categories:
- 学习笔记
- Linux
- Ubuntu
---
1. 安装ssh服务

{% codeblock %}
apt-get install openssh-server
{% endcodeblock %}

1. 开启ssh服务

{% codeblock %}
service ssh start
{% endcodeblock %}

1. ssh配置文件

{% codeblock %}
vi /etc/ssh/sshd_config
{% endcodeblock %}