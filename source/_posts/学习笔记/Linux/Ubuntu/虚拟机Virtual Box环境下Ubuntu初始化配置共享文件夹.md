---
title: 虚拟机Virtual Box环境下Ubuntu初始化配置共享文件夹
date: 2018-05-29 16:39:55
tags:
- linux
- ubuntu
- virtualbox
categories:
- 学习笔记
- Linux
- Ubuntu
keywords:
- linux
- ubuntu
- virtualbox
- 共享文件夹
---

# 虚拟机安装增强功能

{% asset_img 1.png %}

# 宿主机任意位置创建一个文件夹，用于宿主机与虚拟机交换文件

<!-- more -->

# 创建共享文件夹，选择在步骤2中创建的文件夹路径

{% asset_img 3-1.png %}
{% asset_img 3-2.png %}

# 虚拟机mnt文件夹下创建一个文件夹，用于虚拟机与宿主机交换文件

{% codeblock lang:sh %}
cd /mnt
mkdir share
{% endcodeblock %}

# 建立共享文件夹映射

{% codeblock lang:sh %}
cd /mnt
sudo mount -t vboxsf ShareSwap share
{% endcodeblock %}
