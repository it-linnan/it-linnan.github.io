---
title: 虚拟机VMware环境下CentOS初始化配置网卡
date: 2018-05-29 16:22:45
tags:
- linux
- centos
- vmware
categories:
- 学习笔记
- Linux
- CentOS
keywords:
- linux
- centos
- vmware
- 网卡
---
1. 检测网卡
{% codeblock lang:sh %}
ipconfig
{% endcodeblock %}
发现只有ens33和lo两个网卡，需要将ens33修改为eth0
<!-- more -->
1. 修改网卡名称，将其中得NAME和DEVICE项修改为eth0
{% codeblock lang:sh %}
vi /etc/sysconfig/network-scripts/ifcfg-ens33
{% endcodeblock %}

1. 修改配置文件名称
{% codeblock lang:sh %}
mv /etc/sysconfig/network-scripts/ifcfg-ens33 /etc/sysconfig/network-scripts/ifcfg-eth0
{% endcodeblock %}

1. 编辑/etc/default/grub并加入“net.ifnames=0 biosdevname=0 ”到GRUBCMDLINELINUX变量
{% codeblock lang:sh %}
vi /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto net.ifnames=0 biosdevname=0 rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
{% endcodeblock %}

1. 运行命令grub2-mkconfig -o /boot/grub2/grub.cfg 来重新生成GRUB配置并更新内核参数
{% codeblock lang:sh %}
grub2-mkconfig -o /boot/grub2/grub.cfg
{% endcodeblock %}

1. 重启OS
{% codeblock lang:sh %}
reboot
{% endcodeblock %}

1. 检测网卡以验证
{% codeblock lang:sh %}
ifconfig
{% endcodeblock %}