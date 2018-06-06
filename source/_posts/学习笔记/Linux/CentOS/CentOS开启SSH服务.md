---
title: CentOS开启SSH服务
date: 2018-05-29 16:19:47
tags:
- linux
- centos
- ssh
categories:
- 学习笔记
- Linux
- CentOS
keywords:
- linux
- centos
- ssh
---

1. 安装openssh-server
{% codeblock lang:sh %}
yum list installed |grep openssh-server
{% endcodeblock %}
如果有输出，证明已经安装了openssh-server，如果没有，需要安装
{% codeblock lang:sh %}
yum install openssh-server
{% endcodeblock %}
<!-- more -->
1. 修改sshd服务配置文件
{% codeblock lang:sh %}
vi /etc/ssh/sshd_config
{% endcodeblock %}
开启监听
{% codeblock lang:sh %}
Port 22
ListenAddress 0.0.0.0
ListenAddress ::
{% endcodeblock %}
允许远程登录
{% codeblock lang:sh %}
PermitRootLogin yes
{% endcodeblock %}
使用用户名密码作为连接验证
{% codeblock lang:sh %}
PasswordAuthentication yes
{% endcodeblock %}

1. 开启sshd服务
{% codeblock lang:sh %}
service sshd start
{% endcodeblock %}

1. 配置开机自启动
{% codeblock lang:sh %}
systemctl enable sshd
{% endcodeblock %}