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
---
1. 安装openssh-server

{% codeblock %}
yum list installed |grep openssh-server
{% endcodeblock %}
如果有输出，证明已经安装了openssh-server，如果没有，需要安装
{% codeblock %}
yum install openssh-server
{% endcodeblock %}

1. 修改sshd服务配置文件

{% codeblock %}
vi /etc/ssh/sshd_config
{% endcodeblock %}

开启监听
{% codeblock %}
Port 22
ListenAddress 0.0.0.0
ListenAddress ::
{% endcodeblock %}
允许远程登录
{% codeblock %}
PermitRootLogin yes
{% endcodeblock %}
使用用户名密码作为连接验证
{% codeblock %}
PasswordAuthentication yes
{% endcodeblock %}

1. 开启sshd服务

{% codeblock %}
service sshd start
{% endcodeblock %}

1. 配置开机自启动

{% codeblock %}
systemctl enable sshd
{% endcodeblock %}