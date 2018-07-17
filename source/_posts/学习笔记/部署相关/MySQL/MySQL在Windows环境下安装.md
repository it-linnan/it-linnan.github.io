---
title: MySQL在Windows环境下安装
date: 2018-05-30 09:20:03
tags:
- mysql
- windows
categories:
- 学习笔记
- 部署相关
- MySQL
keywords:
- mysql
- windows
---

# 下载官方安装包

{% link MySQL https://www.mysql.com/cn/downloads/ %}

# 解压到本地目录

# 新建my.ini

{% codeblock lang:sh %}
my.ini
[mysql]
default-character-set=utf8
[mysqld]
basedir =d:\DataBase\MySql\mysql-5.7.16-winx64
datadir =d:\DataBase\MySql\mysql-5.7.16-winx64\data
port =3306
character-set-server=utf8
[client]
default-character-set=utf8
{% endcodeblock %}

# cd到bin目录下

# 初始化数据，执行命令

{% codeblock lang:sh %}
mysqld --initialize
-- 据说执行这句话可以免密陆
mysqld --initialize-insecure --user=mysql
{% endcodeblock %}
<!-- more -->

# 注册windows服务

{% codeblock lang:sh %}
mysqld --install MySQL --defaultsfile="d:\DataBase\MySql\mysql-5.7.16-winx64\my.ini"
{% endcodeblock %}

# 启动mysql服务

{% codeblock lang:sh %}
net start mysql
{% endcodeblock %}

# 登陆mysql

{% codeblock lang:sh %}
mysql -u root -p
{% endcodeblock %}

# 提示输入密码，初始无密码，直接回车

# 切换数据库

{% codeblock lang:sql %}
use mysql
{% endcodeblock %}

# 修改密码

{% codeblock lang:sql %}
update user set password = password('newpsw') where user = 'root';
{% endcodeblock %}

# 刷新权限表

{% codeblock lang:sql %}
flush privileges;
{% endcodeblock %}

# 修改主机名，允许所有ip连接

{% codeblock lang:sql %}
update user set hosts = '%' where user = 'root';
{% endcodeblock %}

# 退出mysql

{% codeblock lang:sh %}
exit
{% endcodeblock %}

# 如需关闭服务

{% codeblock lang:sh %}
net stop mysql
{% endcodeblock %}

# 注意

## 5.7版本之后，初始化数据后，会自动分配一个密码，密码可在一个.err文件中查看日志获取

## 在my.ini中配置一项可以不使用密码登陆，在[mysqld]下输入skip-grant-tables