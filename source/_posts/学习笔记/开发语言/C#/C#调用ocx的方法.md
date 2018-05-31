---
title: 'C#调用ocx的方法'
date: 2018-05-30 09:49:04
tags:
- C#
- ocx
categories:
- 学习笔记
- 开发语言
- C#
---
一、Winform工程中调用ocx

1. 在项目中添加对ocx的引用
1. 将ocx文件拖拽至工具箱
1. 将ocx控件从工具箱中拖拽到窗体上

二、类库工程中调用ocx

1. 编译ocx文件

    1. 打开Visual Studio命令提示符
    1. 输入命令aximp xxx.ocx，生成两个dll文件

1. 将两个dll文件放到项目路径下
1. 在项目中添加引用，引用Ax开头的dll
1. 调用ocx控件的方法

{% codeblock lang:c#  %}
Ocx ocx = new Ocx();
ocx.CreateControl();
ocx.method();
{% endcodeblock %}

{% asset_img 1.png %}