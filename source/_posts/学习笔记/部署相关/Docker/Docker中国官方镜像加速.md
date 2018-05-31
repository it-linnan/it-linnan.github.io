---
title: Docker中国官方镜像加速
date: 2018-05-30 08:34:36
tags:
- docker
- docker hub
categories:
- 学习笔记
- 部署相关
- Docker
---
{% link Docker中国官方镜像加速 https://www.docker-cn.com/registry-mirror external title  %}

编辑配置文件，配置镜像地址
{% codeblock   %}
vi /etc/docker/daemon.json

{
    "registry-mirrors": ["https://registry.docker-cn.com"]
}
{% endcodeblock %}
