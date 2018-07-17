---
title: WebSphere下配置servlet可访问WEB-INF
date: 2018-05-30 09:24:20
tags:
- websphere
- was
categories:
- 学习笔记
- 部署相关
- WebSphere
keywords:
- websphere
- was
- 403
---

# {% link WebSphere的WebContainer配置项说明 https://www.ibm.com/support/knowledgecenter/was_beta_liberty/com.ibm.websphere.liberty.autogen.beta.doc/ae/rwlp_config_webContainer.html %}

WebSphere的WebContainer配置项中：

| Attributename          | Data type | Default value | Description                                                                                                                            |
| :--------------------: | :-------: | :-----------: | :------------------------------------------------------------------------------------------------------------------------------------: |
| exposeWebInfOnDispatch | boolean   | false         | If true, a servlet can access files in the WEB-INF directory. If false (default), a servlet cannot access files the WEB-INF directory. |

exposeWebInfOnDispatch属性默认值为false，servlet不能访问WEB-INF目录。

# 配置示例

按照下图配置，可访问WEB-INF目录
<!-- more -->
{% asset_img 1.png %}