---
title: WebSphere下忘记控制台密码
date: 2018-05-30 09:46:20
tags:
- websphere
- was
categories:
---
取消控制台认证
在was node的安装目录下，查找安全文件security.xml
{% codeblock %}
$WAS_Profile_HOME\config\cells
{% endcodeblock %}

在xml中，查找第一个enabled属性，将其修改为false，再重新启动即可
