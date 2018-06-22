---
title: WebSphere更新jsp和web.xml
date: 2018-06-15 08:47:26
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
- jsp
- web.xml
---
在WebSphere下部署应用时，更新任何文件都应该通过控制台更新的方式去更新。而且在集群部署的情况下，通过控制台更新可以将更新文件同步到所有节点上。

在某些特殊情况下，需要手动更新。这时，就需要我们手动删除缓存，同步相关文件。如果是集群部署，还需要将更新文件手动同步到所有节点上。这里介绍一下手动更新jsp和web.xml的方法。
<!-- more -->
### 手动更新jsp

1. 将jsp文件上传到was服务器上。
1. 删除缓存目录下，jsp编译后生成的class文件
{% codeblock jsp缓存目录 lang:sh %}
{was_home}\AppServer\profiles\AppSrv01\temp
{% endcodeblock %}

### 手动更新web.xml

1. 将web.xml上传到was服务器上.
1. 将更新内容同步到web_merged.xml文件中
1. 同步缓存
{% codeblock web.xml缓存目录 lang:sh %}
config\cells\<cell_name>\applications\<ear_name>\deployments\<app_name>\<war_name>\WEB-INF
{% endcodeblock %}