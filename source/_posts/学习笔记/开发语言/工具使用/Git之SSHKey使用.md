---
title: Git之SSHKey使用
date: 2018-06-21 16:27:18
tags:
- git
- ssh key
categories:
- 学习笔记
- 开发语言
- 工具使用
keywords:
- git
- ssh key
---

# SSH key作用

使用SSH Key可以免密登录ssh服务器，在我们日常使用git的时候，一般会使用SSH Key。

# 使用SSH Key的步骤

## 生成SSH Key

打开命令行工具，输入如下命令
{% codeblock lang:sh %}
ssh-keygen -t rsa -C "Linn"
-- 提示如下内容，如果使用默认文件，直接敲回车。或者输入指定文件路径后，敲回车。
-- 请输入保存密钥的文件名，不输入的情况，将默认使用~/.ssh/id_rsa
Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa):
-- 提示如下内容，如果不设置密码，直接敲回车。或者输入密码后，敲回车。
-- 输入密码，不输入表示无密码
Enter passphrase (empty for no passphrase):
-- 提示如下内容，如果不设置密码，直接敲回车。或者输入相同密码后，敲回车。
-- 再次输入密码
Enter same passphrase again:
-- 提示如下内容，密钥生成成功。密钥生成的位置是~/.ssh，其中Linn是私钥，Linn.pub是公钥。
Your identification has been saved in Linn.
Your public key has been saved in Linn.pub.
The key fingerprint is:
SHA256:/Rc6E9x+bExy8rHYkenvIudnWXGZ097JunbOO16xcwY Linn
The key's randomart image is:
+---[RSA 2048]----+
|              E++|
|        . .    .=|
|       . o . . + |
|        +.+.+.*  |
|        S=.*o@=+o|
|          o.B+XX+|
|           .=*o+*|
|            o+.o |
|                 |
+----[SHA256]-----+
{% endcodeblock %}
<!-- more -->

## 添加SSH Key到ssh-agent

执行ssh-add命令
{% codeblock lang:sh %}
ssh-add Linn
{% endcodeblock %}
添加成功
{% codeblock lang:sh %}
Identity added: Linn (Linn)
{% endcodeblock %}
如果出现如下提示，说明ssh-agent服务未启动，需要先启动服务，再执行ssh-add命令
{% codeblock lang:sh %}
Could not open a connection to your authentication agent.
{% endcodeblock %}
启动ssh-agent服务
{% codeblock lang:sh %}
ssh-agent bash
{% endcodeblock %}

## 登录Github或其他Git后台，添加公钥

## 测试

{% codeblock lang:sh %}
ssh git@github.com
{% endcodeblock %}
配置成功
{% codeblock lang:sh %}
PTY allocation request failed on channel 0
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
{% endcodeblock %}

# 关于SSH agent

{% blockquote wiki https://wiki.archlinux.org/index.php/SSH_keys_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#SSH_agents SSH agents %}
如果您的私钥使用密码短语来加密了的话，每一次使用 SSH 密钥对进行登录的时候，您都必须输入正确的密码短语。
而 SSH agent 程序能够将您的已解密的私钥缓存起来，在需要的时候提供给您的 SSH 客户端。这样子，您就只需要将私钥加入 SSH agent 缓存的时候输入一次密码短语就可以了。这为您经常使用 SSH 连接提供了不少便利。
SSH agent 一般会设置成在登录会话的时候自动启动，并在整个会话中保持运行。有不少的 SSH agent 供您选择，我们将为您介绍几种常用的 SSH agent，您可以根据您的需要进行选择。
{% endblockquote %}