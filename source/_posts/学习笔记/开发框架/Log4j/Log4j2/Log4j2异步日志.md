---
title: Log4j2异步日志
date: 2018-11-16 13:55:28
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
- 异步日志
---

Log4j 2的异步日志详细介绍：{% link Async Loggers https://logging.apache.org/log4j/2.x/manual/async.html %}

{% blockquote Log4j https://logging.apache.org/log4j/2.x/ Apache Log4j 2 %}
Apache Log4j 2是对Log4j的升级，它比其前身Log4j 1.x提供了重大改进，并提供了Logback中可用的许多改进，同时修复了Logback架构中的一些固有问题。
{% endblockquote %}

Log4j 2基于LMAX Disruptor库，实现了一个高性能的异步记录器。在多线程场景中，异步记录器的吞吐量比Log4j 1.x和Logback高18倍，延迟低。

<!-- more -->

# 项目中引入Log4j2依赖

我创建的是Maven项目，按需要在pom文件中添加如下依赖：
{% codeblock pom.xml lang:xml %}
<!-- Log4j2 API接口 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>${log4j2.version}</version>
</dependency>
<!-- Log4j2 API实现 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>${log4j2.version}</version>
</dependency>
<!-- Log4j2 web项目支持 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-web</artifactId>
    <version>${log4j2.version}</version>
</dependency>
<!-- Apache Commons Logging桥接适配器 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-jcl</artifactId>
    <version>${log4j2.version}</version>
</dependency>
<!--SLF4J桥接适配器-->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>${log4j2.version}</version>
</dependency>
{% endcodeblock %}

# 异步日志

## 异步日志介绍

异步日志的实现思路是在单独的线程中执行I/O操作，在调用形如Logger.log的API时，能够更快的返回到主程序中，以提高应用程序的性能。Log4j 2在异步日志领域，做了很多优化和改进。而我们只需要在配置文件中，做一个简单的配置，就可以获得异步日志记录的功能特性。

Log4j 2设计了两种异步日志：

1. Async Appender。内部使用的一个队列（ArrayBlockingQueue）和一个后台线程，日志先存入队列，后台线程从队列中取出日志。阻塞队列容易受到锁竞争的影响，当更多线程同时记录时性能可能会变差。
2. Async Logger。内部使用的是LMAX Disruptor技术，Disruptor是一个无锁的线程间通信库，它不是一个队列，不需要排队，从而产生更高的吞吐量和更低的延迟。

我们采用第二种，也是官方推荐的Async Logger的方式，进行异步日志的配置。

## 异步日志配置

### 项目中引入Disruptor依赖

因为Async Logger使用了Disruptor技术，需要在pom文件中添加如下依赖：
{% codeblock pom.xml lang:xml %}
<!-- Disruptor -->
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.3.7</version>
</dependency>
<!-- 如果你想使用Async Appender，可以考虑引入这个包替代ArrayBlockingQueue -->
<dependency>
    <groupId>com.conversantmedia</groupId>
    <artifactId>disruptor</artifactId>
    <classifier>jdk7</classifier>
    <version>1.2.10</version>
</dependency>
{% endcodeblock %}

注意，Log4j-2.9+需要disruptor-3.3.4.jar或更高版本。在Log4j-2.9之前，需要disruptor-3.0.0.jar或更高版本。

### 全异步

设置一个环境变量，用来开启全异步：

{% codeblock 全异步 lang:bash %}
log4j2.contextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector
{% endcodeblock %}

开启全异步时，日志配置中需要使用普通的Root和Logger元素。如果使用了AsyncRoot或AsyncLogger，将产生不必要的开销。

### 同步异步混合

全异步是官方推荐的，也是性能最佳的方式，但同步异步混合使用，能够提供更大的灵活性。使用AsyncRoot、AsyncLogger、Root、Logger混合配置，可以实现同步异步混合。但是需要注意，配置中只能有一个root元素，也就是只能使用AsyncRoot或Root中的一个。

### 实际配置

#### 自定义插件

Log4j使用插件模式配置组件。因此，无需编写代码来创建和配置Appender，Layout，Pattern Converter等，Log4j自动识别并使用插件。

因此，我们可以基于插件模式，方便的扩展现有功能特性。

我有两个需求：

1. 日志文件做分包处理，按照"应用编码/用户标识/8位年月日.log"的规则细化。其中应用编码和用户标识，在应用启动阶段，是空的，而应用启动阶段是需要输出日志的。因此，我希望在应用编码和用户标识为空时，log4j能识别到这种情况，并给它们赋一个默认值。
2. 日志文件的根路径是可配的，但是我又不希望可配置的参数分散到不同的配置文件中，所以希望日志根路径能够到其他配置文件中读取。

为实现以上需求，我编写了一个自定义插件，基于Log4j提供的上下文解析插件，进行了扩展：

{% codeblock ContextMapWithDefValLookup lang:java  %}
@Plugin(name = "ctxdefval", category = "Lookup")
public class ContextMapWithDefValLookup extends ContextMapLookup {
    public static final String DEFAULT_VALUE_PREFIX = "no";
    @Override
    public String lookup(String key) {
        return this.lookupWithDefaultValue(key, super.lookup(key));
    }
    @Override
    public String lookup(LogEvent event, String key) {
        return this.lookupWithDefaultValue(key, super.lookup(event, key));
    }
    private String lookupWithDefaultValue(String key, String value) {
        // 上下文中没有该字段的情况
        if (StringUtils.isEmpty(value)) {
            // 从统一配置文件中获取，如果获取不到，使用no+key作为默认值（noappid)
            value = ReadProperty.getPlatString(key, DEFAULT_VALUE_PREFIX + key);
        }
        return value;
    }
}
{% endcodeblock %}

#### 配置文件

``` xml log4j2.xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="debug" packages="com.yondervision.plat.common.log.lookup">
    <Appenders>
        <!--打印日志到控制台-->
        <Console name="stdout" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %msg %equals{%x}{[]}{}%xEx%n"/>
        </Console>
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
    </Appenders>
    <Loggers>
        <AsyncRoot level="debug">
            <AppenderRef ref="stdout" level="debug"/>
            <AppenderRef ref="routing" level="info"/>
        </AsyncRoot>
    </Loggers>
</Configuration>
```

说明：

1. 如果Layout中配置成与%C or $class, %F or %file, %l or %location, %L or %line, %M or %method中的一个位置相关的属性，Log4j 2将获取堆栈的快照，并遍历堆栈跟踪以查找位置信息。这是一项昂贵的操作，同步日志中，在获取此堆栈快照时可能会长时间等待。如果不需要位置，则不会获取快照。而异步日志需要在将日志消息传递给另一个线程之前做出此决定，在该点之后，位置信息将丢失。对于异步日志来说，获取堆栈跟踪快照的性能影响甚至更高。因此，默认情况下，异步记录器和异步追加器不包含位置信息。可以通过指定includeLocation =“true”来覆盖异步日志的默认行为。
2. RollingRandomAccessFileAppender类似于标准的RollingFileAppender，内部使用的是ByteBuffer和RandomAccessFile。由于RandomAccessFile可以自由访问文件的任意位置，日志记录通常是向已存在的文件后追加内容，所以使用RandomAccessFile能获得更好的性能。