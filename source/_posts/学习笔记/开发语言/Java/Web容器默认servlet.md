---
title: Web容器默认servlet
date: 2018-05-30 10:01:05
tags:
- java
- web container
categories:
- 学习笔记
- 开发语言
- Java
---

| 默认servlet-name  | 容器                         |
| :---------------: | :--------------------------: |
| default           | Tomcat Jetty JBoss GlassFish |
| _ah_default       | Google App Engine            |
| resin-file        | Resin                        |
| FileServlet       | WebLogic                     |
| SimpleFileServlet | WebSphere                    |

在spring配置文件中开启如下配置，即可开启默认servlet
{% codeblock lang:xml %}
<mvc:default-servlet-handler/>
{% endcodeblock %}