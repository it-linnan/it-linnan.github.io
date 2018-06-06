---
title: WebSphere下log4j2自定义插件未加载的解决办法
date: 2018-06-05 09:28:01
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
- log4j2
- 自定义插件
---
之前，我们采用引用共享库的方式，解决了WebSphere自带jar包与项目下log4j2相关的jar包冲突的问题。
{% post_link 学习笔记/部署相关/WebSphere/WebSphere自带jar包与应用jar包冲突的解决办法 %}

但是启动工程后发现，log4j2没有加载自定义插件。

在log4j的官网上，我查到了如下介绍（可以跳过这段英文介绍，后面有翻译）：
<!-- more -->
{% blockquote Apache Log4j 2  https://logging.apache.org/log4j/2.x/manual/plugins.html 插件介绍 %}
In Log4j 2 a plugin is declared by adding a @Plugin annotation to the class declaration. During initialization the Configuration will invoke the PluginManager to load the built-in Log4j plugins as well as any custom plugins. The `PluginManager` locates plugins by looking in five places:

1. Serialized plugin listing files on the classpath. These files are generated automatically during the build (more details below).
1. (OSGi only) Serialized plugin listing files in each active OSGi bundle. A `BundleListener` is added on activation to continue checking new bundles after `log4j-core` has started.
1. A comma-separated list of packages specified by the `log4j.plugin.packages` system property.
1. Packages passed to the static `PluginManager.addPackages` method (before Log4j configuration occurs).
1. The packages declared in your log4j2 configuration file.
{% endblockquote %}

大意就是，log4j2支持插件机制，在启动阶段，PluginManager不仅会加载内建插件，也会加载自定义插件。PluginManager会在5个位置查找插件。

我们采用的是，在配置文件中定义插件扫描包位置，自定义插件放在扫描包下。再看官方对插件扫描包位置配置项的说明（只截取部分）：
{% blockquote Apache Log4j 2  https://logging.apache.org/log4j/2.x/manual/configuration.html#ConfigurationSyntax 配置项说明 %}
The configuration element in the XML file accepts several attributes:

| Attribute Name | Description                                                                                                                                                                      |
| :------------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| packages | A comma separated list of package names to search for plugins. `Plugins are only loaded once per classloader` so changing this value may not have any effect upon reconfiguration. |
{% endblockquote %}
注意看我标注的这句话，每个类加载器只加载一次插件。

之前在我们在建立共享库的时候，勾选了【请对此共享库使用隔离的类装入器】，此时共享库将使用独立的类加载器，而我们的自定义插件没有放在共享库中，自然扫描不到。

解决办法是，在共享库的配置中，取消勾选【请对此共享库使用隔离的类装入器】。