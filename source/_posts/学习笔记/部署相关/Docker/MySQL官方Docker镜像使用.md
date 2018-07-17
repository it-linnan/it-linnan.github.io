---
title: MySQL官方Docker镜像使用
date: 2018-05-30 08:36:18
tags:
- docker
- mysql
- linux
- ubuntu
categories:
- 学习笔记
- 部署相关
- Docker
keywords:
- docker
- mysql
- linux
- ubuntu
---

# 启动mysql服务实例

## 拉取镜像

{% codeblock lang:sh %}
docker pull mysql
docker pull mysql:5.7
{% endcodeblock %}

## 启动实例

{% codeblock lang:sh %}
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -d mysql:5.7
{% endcodeblock %}

# 测试mysql服务实例

## 安装mysql客户端

<!-- more -->
{% codeblock lang:sh %}
apt-get install mysql-client-core-5.7
{% endcodeblock %}

## 登录mysql

{% codeblock lang:sh %}
mysql -h127.0.0.1 -P3306 -uroot -ppassword
{% endcodeblock %}

# 使用自定义MySQL配置文件

默认情况下，MySQL的启动配置文件是/etc/mysql/my.cnf，引用了/etc/mysql/conf.d和/etc/mysql/mysql.conf.d文件夹
想要使用自定义配置文件，可将配置文件挂载到mysql容器的/etc/mysql/conf.d和/etc/mysql/mysql.conf.d目录下
{% codeblock lang:sh %}
docker run --name mysql -p 3307:3306 -v /mnt/share/custom.cnf:/etc/mysql/conf.d/custom.cnf -e MYSQL_ROOT_PASSWORD=password -d mysql:5.7
{% endcodeblock %}

# 使用自定义MySQL配置

## 使用命令行参数来指定配置

{% codeblock lang:sh %}
docker run --name mysql -p 3307:3306 -v /mnt/share/custom.cnf:/etc/mysql/conf.d/custom.cnf -e MYSQL_ROOT_PASSWORD=password -d mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --lower_case_table_names=1
{% endcodeblock %}

## 查看可配参数
{% codeblock lang:sh %}
docker run -it --rm mysql:tag --verbose --hel
{% endcodeblock %}
