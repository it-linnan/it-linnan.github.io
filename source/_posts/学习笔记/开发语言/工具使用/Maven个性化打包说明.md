---
title: Maven个性化打包说明
date: 2018-07-17 11:37:51
tags:
- maven
- war
categories:
- 学习笔记
- 开发语言
- 工具使用
keywords:
- maven
- war
- maven-war-plugin
---

以实际项目为例，介绍maven打包插件的使用，实现个性化打包。

# 工程目录结构

## miapp项目分为两个模块

{% asset_img miapp.png %}

## yd-miapp模块为web项目

{% asset_img yd-miapp.png %}

## java代码部分

其中，function目录下，存在不同中心的组件代码
{% asset_img java.png %}

## 静态资源部分

其中，views目录下，存在不同中心的静态资源
{% asset_img webapp.png %}

# 目前war包目录结构

目前war包中包含了全部的组件代码和静态资源，与工程目录结构类似。

# 独立打包后war包目录结构

比如，我们打包的是城市代码为00031400。独立打包时，只包含00031400城市的组件和静态资源。

***注意，00000000和99999999是特殊的城市代码，d00000000存放的是公共资源，d99999999存放的是模板资源。***
{% asset_img yd-miapp-1.0-SNAPSHOT.png %}
<!-- more -->

# 打包操作

## 新建配置文件

在yd-miapp/src/main/resources/filters目录下新建配置文件，配置文件命名规则是product-centerid.xml，比如product-00031400.xml、product-00055500.xml

## 编写配置项

配置文件中的配置项请参考sample-db2、sample-mysql、sample-oracle

## 执行以下maven打包命令

进入miapp目录下，执行以下maven命令。其中centerid是中心编码，比如00031400、00055500
{% codeblock lang:sh %}
mvn clean package -Dp=centerid
mvn clean package -Dp=00031400
{% endcodeblock %}

# 原理

maven提供了一个打包插件，{% link maven-war-plugin https://maven.apache.org/plugins/maven-war-plugin/ %}，负责收集Web应用程序的所有的依赖，类和资源，并将它们打包到war包中。

打包插件提供了一系列参数，通过配置这些参数，可以个性化输出的war包。

我们使用了以下参数来进行独立打包。

| 属性                                                                                                                | 类型   | 支持版本    | 描述                                                                                                                     |
| :-----------------------------------------------------------------------------------------------------------------: | :----: | :---------: | :----------------------------------------------------------------------------------------------------------------------: |
| {% link &lt;warSourceExcludes> https://maven.apache.org/plugins/maven-war-plugin/war-mojo.html#warSourceExcludes %} | String | -           | 在编译周期完成后，向target目录复制文件时忽略的目录列表，用逗号分隔。使用表达式％regex []，可以使用正则表达式语法。       |
| {% link &lt;packagingExcludes> https://maven.apache.org/plugins/maven-war-plugin/war-mojo.html#packagingExcludes %} | String | 2.1-alpha-2 | 最终构建war包之前，从target目录抽取war包文件时忽略的目录列表，用逗号分隔。使用表达式％regex []，可以使用正则表达式语法。 |

我们在独立打包时，想要忽略的就是除当前城市以外的组件和资源文件，而且需要保留d00000000和d99999999目录。因此，使用如下表达式过滤资源。
{% codeblock lang:sh %}
%regex[WEB-INF/classes/function/d(?!(?:${p}|00000000|99999999).+$).+],
%regex[WEB-INF/views/assets/plugins/d(?!(?:${p}|00000000|99999999).+$).+],
%regex[WEB-INF/views/d(?!(?:${p}|00000000|99999999).+$).+]
{% endcodeblock %}

首先，在pom.xml中配置了maven-war-plugin
{% codeblock lang:xml %}
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <configuration>
        <webResources>
            <resource>
                <directory>src/main/webapp</directory>
                <includes>
                    <include>WEB-INF/web.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </webResources>
        <warSourceExcludes>${packagingExcludes}</warSourceExcludes>
        <packagingExcludes>${packagingExcludes}</packagingExcludes>
    </configuration>
</plugin>
{% endcodeblock %}

然后在pom.xml中配置了一个生产环境使用的公共profile。
{% codeblock lang:xml %}
<profile>
    <id>product</id>
    <build>
        <filters>
            <filter>${basedir}/src/main/resources/filters/product-${p}.properties</filter>
        </filters>
    </build>
    <properties>
        <packagingExcludes>
            %regex[WEB-INF/classes/function/d(?!(?:${p}|00000000|99999999).+$).+],
            %regex[WEB-INF/views/assets/plugins/d(?!(?:${p}|00000000|99999999).+$).+],
            %regex[WEB-INF/views/d(?!(?:${p}|00000000|99999999).+$).+]
        </packagingExcludes>
    </properties>
    <activation>
        <property>
            <name>p</name>
            <value></value>
        </property>
    </activation>
</profile>
{% endcodeblock %}

公共profile激活条件是，执行maven命令时，传入参数p（maven命令传参数的方式是：-D参数名=参数值，比如：-Dp=00031400）  
公共profile激活后，会使用product-centerid.properties进行参数初始化，根据资源过滤表达式进行打包，最终实现独立打包。