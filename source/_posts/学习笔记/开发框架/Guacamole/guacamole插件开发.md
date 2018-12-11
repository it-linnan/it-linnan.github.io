---
title: guacamole插件开发
date: 2018-11-16 16:33:49
tags:
- guacamole
- guacamole extensions
categories:
- 学习笔记
- 开发框架
- Guacamole
keywords:
- guacamole
- guacamole extensions
- guacamole插件
---

Guacamole手册：{% link Guacamole手册 http://guacamole.apache.org/doc/gug/ %}

{% blockquote Guacamole http://guacamole.apache.org/ Apache Guacamole %}

Guacamole是什么？

1. Apache Guacamole是一个无客户端的远程桌面网关。
1. 它支持VNC，RDP和SSH等标准协议。
1. 我们称之为无客户端，因为不需要插件或客户端软件。
1. 感谢HTML5，一旦Guacamole安装在服务器上，您访问桌面所需的只是一个Web浏览器。

为什么要用Guacamole？

1. 随时随地访问PC
1. 保持桌面在云端
1. 免费、开源
1. 建立在文档齐全的API之上
1. 商业支持

{% endblockquote %}

<!-- more -->

# Guacamole架构

Guacamole架构下，发起一次远程桌面请求的流程是：

1. 用户使用Web浏览器，访问Guacamole客户端应用，使用Guacamole协议请求一个远程桌面
1. Guacamole客户端应用将Guacamole协议转发给guacd服务端
1. guacd服务端解释Guacamole协议，转换成远程桌面支持的协议后，请求到真实的远程桌面

{% asset_img guacamole介绍.png guacamole介绍 %}

## Guacamole协议

Guacamole协议，是一种用于远程显示渲染和事件传输的协议。Guacamole协议不同于远程桌面协议（VNC或RDP等），它建立在远程桌面协议之上，旨在提供跨平台的远程桌面功能，不依赖于特定的桌面环境。由一个中间件，将Guacamole协议翻译成具体的远程桌面协议。当新增一个远程桌面协议时，只需要在中间件上加入新的翻译转换策略，而不需要修改客户端应用。

## guacd

guacd是Guacamole的核心，它动态加载对远程桌面协议（称为“客户端插件”）的支持，并根据从Web应用程序收到的指令将它们连接到远程桌面吗，也就是上文提到的翻译中间件。

guacd是一个守护进程，它与Guacamole一起安装并在后台运行，侦听来自Web应用程序的TCP连接。 guacd也不了解任何特定的远程桌面协议，而是实现了足够的Guacamole协议来确定需要加载哪些协议支持以及必须将哪些参数传递给它。 加载客户端插件后，它将独立于guacd运行，并完全控制自身与Web应用程序之间的通信，直到客户端插件终止。

guacd和所有客户端插件依赖于一个公共库libguac，它使通过Guacamole协议的通信更容易，更抽象。

## Gucamole客户端应用

用户实际使用的Guacamole部分是Web应用程序。Web应用程序不实现任何远程桌面协议。 它依赖于guacd，只需要实现一个漂亮的Web界面和身份验证层。

# 安装guacd

官方提供两种安装方式，一种是下载源码，手动编译安装；一种是docker镜像。

官方文档上由详细说明：

{% link 编译安装 http://guacamole.apache.org/doc/gug/installing-guacamole.html %}

{% link docker镜像 http://guacamole.apache.org/doc/gug/guacamole-docker.html %}

# guacamole插件开发

Guacamole的插件可以：

1. 提供备用身份验证方法和连接/用户数据源。
1. 提供将在Guacamole执行身份验证和隧道连接等任务时通知的事件侦听器。
1. 通过额外的CSS文件和静态资源个性化Guacamole。
1. 通过提供将自动加载的JavaScript来扩展Guacamole的JavaScript代码。
1. 添加其他显示语言，或更改现有语言的翻译字符串。

## guacamole插件加载方式

Guacamole客户端应用是与插件是分离部署的，客户端应用在启动阶段，动态加载插件。因此需要理解插件的加载方式，才能开发出符合自身需求的插件。

因为应用与插件是分离部署的，所以Guacamole约定了一个配置路径，用于存放资源文件，其中就包括插件。客户端应用在启动时，到配置路径下查找是否有插件，如果有，则加载到应用中。

## guacamole配置路径

guacamole配置路径称为GUACAMOLE_HOME，默认情况下位于/etc/guacamole。所有配置文件，插件等都存放在此目录中。

GUACAMOLE_HOME的结构是严格定义的，由以下可选文件组成：

1. guacamole.properties  
Guacamole的主配置文件。此文件中的属性指示Guacamole将如何连接到guacd，并可能配置已安装的身份验证扩展的行为。

1. logback.xml  
Guacamole使用的日志框架是Logback。默认情况下，Guacamole只将日志输出到控制台。可以通过提供该文件自定义日志输出行为。

2. extensions/  
插件存放目录，Guacamole会在启动时自动加载此目录中的所有.jar文件。

1. lib/  
插件依赖jar包存放目录。如果插件依赖第三方jar，例如数据库驱动程序，就可以放在这个目录下。

## 插件格式

插件是一个.jar文件，jar中包含类、资源和guacamole扩展清单。其中，扩展清单是插件的描述文件，需要放在jar文件的根目录下。Guacamole客户端应用启动时，读取扩展清单，来进行动态加载插件。

### 扩展清单

扩展清单是一个json文件，文件名约定为guac-mainfest.json。如果你和我一样，基于maven开发，那么根据maven对资源文件的约定，扩展清单需要放在resouces文件夹下。扩展清单是一个配置文件，配置项如下：

|属性|是否必填|描述|
|:--:|:--:|:--:|
|guacamoleVersion|是|插件依赖的guacamole版本号，如果插件不依赖于特定版本，则可以使用*，将绕过版本兼容性检查。|
|name|是|易于阅读、理解的名字|
|namespace|是|插件的唯一标识。如果插件中包含静态资源，那么这些资源将以该命名空间作为上下文根。|
|authProviders|否|一组限定类名，这些类是AuthenticationProvider的子类，插件中提供的用于身份验证的类。|
|listeners|否|一组限定类名，这些类是Listener的子类，插件中提供的用于监听事件的类。|
|js|否|一组JavaScript文件路径，路径必须是相对路径。|
|css|否|一组CSS文件路径，路径必须是相对路径。|
|html|否|一组HTML文件路径，路径必须是相对路径。|
|translations|是|一组翻译文件路径，路径必须是相对路径。此处声明的翻译文件将自动添加到可用语言中,如果翻译文件提供Guacamole中已存在的语言，则其字符串将覆盖现有翻译的字符串|
|resources|否|一个对象，其中每个属性名称是Web资源文件的名称，每个值都是该资源的mimetype，所有路径都必须是相对路径。此处声明的Web资源将在app/ext/NAMESPACE/PATH中提供给应用程序，其中NAMESPACE是namespace属性的值，PATH是声明的Web资源文件名。|

{% codeblock 最简单的扩展清单 lang:json %}
{
    "guacamoleVersion" : "0.9.14",
    "name" : "My Extension",
    "namespace" : "my-extension"
}
{% endcodeblock %}

{% codeblock 扩展清单 lang:json %}
{
    "guacamoleVersion" : "0.9.14",
    "name"      : "My Extension",
    "namespace" : "my-extension",
    "css" : [ "theme.css" ],
    "html" : [ "loginDisclaimer.html" ],
    "resources" : {
        "images/logo.png"   : "image/png",
        "images/cancel.png" : "image/png",
        "images/delete.png" : "image/png"
    }
}
{% endcodeblock %}

