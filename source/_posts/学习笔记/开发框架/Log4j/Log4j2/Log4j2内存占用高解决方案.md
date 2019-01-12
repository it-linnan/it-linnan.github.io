---
title: Log4j2内存占用高解决方案
date: 2019-01-12 20:37:23
tags:
- Log4j
- Log4j2
categories:
- 学习笔记
- 开发框架
- Log4j
- Log4j2
keywords:
- Log4j
- Log4j2
- 内存占用
- oom
- gc
---

# Log4j2异步日志回顾

之前，我们在应用中使用Log4j2的异步日志，获得了很好的表现。
{% post_link 学习笔记/开发框架/Log4j2/Log4j2异步日志 %}

# 内存占用过高

我们在生产环境使用了`kubernetes`技术，在应用部署到k8s后，监控了一段时间，发现每天都有内存溢出的现象。而当应用心跳地址多次无法访问后，k8s自动帮我们重启了应用，但是重启并不能从根本上解决问题。

在某一次，容器内存占用很高时，我们将内存使用情况dump下来，使用IBM的工具分析了一下，发现是`Log4j2`的某个类所使用的`ConcurrentHashMap`集合很大，占用内存很高，这时终于发现了问题原因。

<!-- more -->

# 问题分析过程

首先，占用内存过高的类是`RoutingAppender`，现象是`RoutingAppender`类中使用的`ConcurrentHashMap`集合过大，先简单介绍一下`RoutingAppdener`。

RoutingAppender相当于一个路由器，分析每次写日志的操作，然后将它们路由到子Appender上。路由的依据是一个路由key，路由key需要预先配置，每次写日志时，RoutingAppender会计算路由key，如果配置了路由key对应的子Appender，那么就路由到子Appender上，由子Appender负责真正的日志输出。

我阅读了一下`RoutingAppender`的源码。`RoutingAppender`有两个变量是`ConcurrentMap`类型的：

{% codeblock pom.xml lang:java %}
private final ConcurrentMap<String, AppenderControl> appenders = new ConcurrentHashMap<>();
private final ConcurrentMap<Object, Object> scriptStaticVariables = new ConcurrentHashMap<>();
{% endcodeblock %}

其中，`appenders`变量存放的是子Appender，key是解析后的路由key。

可达鸭眉头一皱，发现事情并不简单。

之前，我们在使用异步日志时配置过RoutingAppender，现在回忆一下是如何配置的：

``` xml RoutingAppender
<Routing name="routing">
    <Routes pattern="$${ctxdefval:appid}$${ctxdefval:userid}$${date:yyyy-MM-dd}">
        <Route>
            <!-- 输出到日志文件 -->
            <RollingRandomAccessFile name="RollingFile"
                                        fileName="${ctxdefval:logpath}/${ctxdefval:appid}/${date:yyyy-MM-dd}/${ctxdefval:userid}.log"
                                        filePattern="${ctxdefval:logpath}/${ctxdefval:appid}/${date:yyyy-MM-dd}/${ctxdefval:userid}-%i.log.gz">
                <PatternLayout pattern="%d{HH:mm:ss.SSS} [%-5level] [%tid.%tn] %msg %equals{%x}{[]}{}%xEx%n"/>
                <Policies>
                    <!-- 日志达到50MB时打包 -->
                    <SizeBasedTriggeringPolicy size="50 MB"/>
                </Policies>
            </RollingRandomAccessFile>
        </Route>
    </Routes>
</Routing>
```

可以看到，我们配置的路由key规则是一个表达式（如下），RoutingAppender在工作时，会解析这个表达式，比如当前上下文中，应用id是app，userid是user，日期是2019-01-12，解析后的路由key就是appuser2019-01-12。这时，会将子Appender存放到map中，key是解析后的路由key。我们可以预想一下，随着时间的增长，这个map会越来越大。

{% codeblock 路由key规则 lang:java %}
$${ctxdefval:appid}$${ctxdefval:userid}$${date:yyyy-MM-dd}
{% endcodeblock %}

那么问题应该就出现在这里啦。

# 解决办法

我想到的解决办法，就是定期清理这个map了。所以我继续阅读源码，发现了一个根据路由key清理map的方法，查找调用这个方法的类，惊讶的发现log4j已经为我们提供了清理插件。

清理插件是`IdlePurgePolicy`，具体的实现细节就直接阅读源码吧，或者看javadoc，说的很清楚了，可以配置的参数如下：

|参数名|描述|
|:--:|:--:|
|timeToLive|appender闲置时间，超过这个闲置时间就会被移除|
|checkInterval|检查的时间间隔，与timeToLive相乘就是真正的清理周期。不配置的话，清理周期就是timeToLive的值|
|timeUnit|时间单位，可选的配置项需参考枚举类TimeUnit|

插件需要配置到Routing节点下，我配置的是10分钟清理一次：

{% codeblock log4j2.xml lang:xml %}
<IdlePurgePolicy timeToLive="10" timeUnit="minutes"/>
{% endcodeblock %}