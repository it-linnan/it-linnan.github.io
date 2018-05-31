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
---

1. MySQL官网下载安装包 https://www.mysql.com/cn/downloads/
1. 解压到本地目录
1. 新建my.ini
1. cd到bin目录下
1. 初始化数据，执行 mysqld --initialize（mysqld --initialize-insecure --user=mysql;据说执行这句话可以免密陆）
1. 注册windows服务，mysqld --install MySQL --defaultsfile="d:\DataBase\MySql\mysql-5.7.16-winx64\my.ini"
1. 启动mysql服务 net start mysql
1. 登陆mysql，mysql -u root -p
1. 提示输入密码，初始无密码，直接回车
1. 切换数据库，use mysql
1. 修改密码，update user set password = password('newpsw') where user = 'roo';
1. 刷新权限表，flush privileges;
1. 修改主机名，允许所有ip连接，update user set hosts = '%' where user = 'root';
1. exit退出mysql
1. 如需关闭服务，net stop mysql

注意：

1. 5.7版本之后，初始化数据后，会自动分配一个密码，密码可在一个.err文件中查看日志获取
1. 在my.ini中配置一项可以不使用密码登陆，在[mysqld]下输入skip-grant-tables

{% codeblock %}
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