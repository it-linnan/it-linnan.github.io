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
keywords:
- c#
- ocx
---

# Winform工程中调用ocx

## 在项目中添加对ocx的引用

## 将ocx文件拖拽至工具箱

## 将ocx控件从工具箱中拖拽到窗体上

<!-- more -->

# 类库工程中调用ocx

## 编译ocx文件

### 打开Visual Studio命令提示符

### 输入命令aximp xxx.ocx，生成两个dll文件

{% asset_img 1.png %}

## 将两个dll文件放到项目路径下

## 在项目中添加引用，引用Ax开头的dll

## 调用ocx控件的方法

{% codeblock lang:csharp %}
Ocx ocx = new Ocx();
ocx.CreateControl();
ocx.method();
{% endcodeblock %}