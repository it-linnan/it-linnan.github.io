---
title: 'C#中程序集版本名词解释'
date: 2018-05-30 09:54:58
tags:
- C#
categories:
- 学习笔记
- 开发语言
- C#
keywords:
- c#
- 程序集
- 版本
---

# 微软内部开发有一个版本号命名规则，格式如下:

| Major      | Minor      | Build  | Revision |
| :--------: | :--------: | :----: | :------: |
| 主要版本号 | 次要版本号 | 生成号 | 修订号   |

eg：2.23.159.23
<!-- more -->

# 版本号各部分解释

## Major

名称相同但主要版本号不同的程序集不可互换。 更高版本号可能表明大幅重写无法假定向后兼容的产品。

## Minor

如果两个程序集的名称和主要版本号相同，而次要版本号不同，这指示显著增强，但照顾到了向后兼容性。 该较高的次要版本号可指示产品的修正版或完全向后兼容的新版本。

## Build

生成号的不同表示对相同源所作的重新编译。 处理器、 平台或编译器更改时，可能使用不同的生成号。

## Revision

名称、主要版本号和次要版本号都相同但修订号不同的程序集应是完全可互换的。 更高修订号可能在修复以前发布的程序集安全漏洞的版本中使用。

# 程序集相关信息（AssemblyInfo)

## AssemblyVersion（程序集版本）

在.NET Framework中编译和运行时使用的版本号，使用该版本号定位和加载指定程序集。当你在你的项目中引用了指定的程序集，其版本号将会嵌入到你的项目中。在运行时，CLR通过该版本号加载指定程序集。注意，仅当程序集使用强命名时，才会使用程序集名称、公钥、语言信息以及该版本号查找指定程序集，否则只会根据文件名进行查找。

## AssemblyFileVersion（文件版本）

在文件系统中给文件的版本号，会在Windows资源管理器中显示。但是，在.NET Framework引用类库时从来不会用到这个版本。
