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
keywords:
- linux
- ubuntu
- ssh
---

# 安装ssh服务

{% codeblock lang:sh %}
apt-get install openssh-server
{% endcodeblock %}

# 开启ssh服务

{% codeblock lang:sh %}
service ssh start
{% endcodeblock %}

# ssh配置文件

{% codeblock lang:sh %}
vi /etc/ssh/sshd_config
{% endcodeblock %}