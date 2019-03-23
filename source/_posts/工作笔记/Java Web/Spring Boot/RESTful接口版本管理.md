---
title: RESTful接口版本管理
date: 2019-03-12 11:36:22
tags:
- java
- Java Web
- Spring Boot
- RESTful
categories:
- 工作笔记
- Java Web
- Spring Boot
keywords:
- rest
- restful
- version
- 接口版本管理
---

# REST简介

REST，全称是`Representational State Transfer`，译为表现层状态转化。REST并不是一个标准，而是一种软件架构风格。

在REST风格下，客户端通过url访问网络上的一个资源，通过HTTP动词请求服务端对资源进行操作。

# 接口版本

服务端接口是会不断变化更新的，一个好的设计，是提供不同版本的接口，而不是在一个接口上进行修改。部署时，服务端包含不同版本的接口，客户端可以依旧使用老版本的接口，也可以随服务端升级到新版本。当所有客户端都升级到新版本时，服务端可以考虑移除旧版本的接口。

在REST风格下，我们可以通过对接口增加版本号的概念，去区分不同版本的接口。版本号有两种表现形式，一种是包含在url中，一种是包含在HTTP头中。例如：

{% codeblock url %}
http://somewhere.com/xxx/v1/user

Accept: application/json; version=v1
{% endcodeblock %}

# Spring Boot starter源码

{% link 接口版本管理源码 https://github.com/it-linnan/api-version-spring-boot-starter %}

# 实现思路

Spring Boot提供了starter的标准，starter完成自动配置，开发者只需要引用starter，就可以实现功能。我们可以通过开发一个api-version-spring-boot-starter，帮助应用快速实现接口版本管理。

Spring Boot对于自定义starter提出的指导有以下几点：

1. 项目包含两个模块，一个是autoconfigure，一个是starter
2. autoconfigure模块包含自动配置相关的代码，和一个清单文件，清单文件中包含自动加载bean的class名
3. starter需引用autoconfigure模块和其他必要的依赖

因此，我们定义了两个模块，一个是autoconfigure，一个是starter，在autoconfigure中完成接口版本管理的功能和自动配置，实现在url中标明版本号。

<!-- more -->

## ApiVersion注解

首先，定义一个注解，注解的作用是标识controller的版本
{% codeblock ApiVersion.java lang:java %}
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ApiVersion {
    /**
     * 版本号
     *
     * @return 版本号
     */
    @AliasFor("value")
    String version() default "";
    /**
     * 版本号
     *
     * @return 版本号
     */
    @AliasFor("version")
    String value() default "";
}
{% endcodeblock %}

## 扩展RequestMappingHandlerMapping

`RequestMappingHandlerMapping`提供了url-controller方法间映射的能力，可以理解为在启动阶段，Spring扫描标注了`@Controller`注解的类，对其中标注了`@RequestMapping`注解的方法进行注册，key为`@RequestMaping`指定的url，value为方法。`@Controller`注解的变体有`@RestController`，`@RequestMapping`注解的变体有`@GetMapping`、`@PostMapping`等。当客户端发起请求时，`DispatcherServlet`在分发请求时，根据启动阶段注册的url和方法映射进行分发。

因此我们需要扩展两个类，一个是`RequestMappingHandlerMapping`，在注册url时，将url注册为统配符的形式，如`{version}/user`。另外一个是`RequestCondition`，提供具体的匹配规则。

具体工作原理是：

1. 服务端启动阶段，Spring扫描Controller，注册能力由`ApiVersionUrlRequestMappingHandlerMapping`提供，将标注了`@ApiVersion`注解的Controller，url统一注册为统配符的形式，形如`{version}/user`，同时将定义的版本号也作为url的属性，注册到Spring容器中
2. 客户端发起请求，请求的url形如`v1/user`
3. `DispathcerServlet`接受到请求，对请求的url与注册的url进行匹配。匹配规则由`ApiVersionUrlRequestCondition`提供，即注册url符合请求的url，且注册版本号与请求的版本号一致，即为匹配成功

{% codeblock ApiVersionUrlRequestCondition.java lang:java %}
@NoArgsConstructor
public class ApiVersionUrlRequestCondition implements RequestCondition<ApiVersionUrlRequestCondition>, ApplicationContextAware {
    @Getter
    private String version;
    private static String regexFormat = "";
    public ApiVersionUrlRequestCondition(String version) {
        this.version = version;
    }
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        if (regexFormat == null || regexFormat.length() == 0) {
            ApiVersionProperties apiVersionProperties = applicationContext.getBean(ApiVersionProperties.class);
            ServerProperties serverProperties = applicationContext.getBean(ServerProperties.class);
            if (serverProperties.getServlet().getContextPath() != null) {
                regexFormat = serverProperties.getServlet().getContextPath();
            }
            regexFormat += "/" + apiVersionProperties.getPrefix() + "%s/**";
        }
    }
    @Override
    public ApiVersionUrlRequestCondition combine(ApiVersionUrlRequestCondition other) {
        return new ApiVersionUrlRequestCondition(other.getVersion());
    }
    @Override
    public ApiVersionUrlRequestCondition getMatchingCondition(HttpServletRequest request) {
        PathMatcher pathMatcher = new AntPathMatcher();
        boolean match = pathMatcher.match(String.format(regexFormat, version), request.getRequestURI());
        return match ? this : null;
    }
    @Override
    public int compareTo(ApiVersionUrlRequestCondition other, HttpServletRequest request) {
        return other.getVersion().compareTo(this.version);
    }
}
{% endcodeblock %}

{% codeblock ApiVersionUrlRequestMappingHandlerMapping.java lang:java %}
@Slf4j
public class ApiVersionUrlRequestMappingHandlerMapping extends RequestMappingHandlerMapping {
    public static final String VERSION_PREFIX = "/{version}";
    public static final String PATH_KEY = "path";
    public static final String VALUE_CACHE_KEY = "valueCache";
    @Override
    protected RequestCondition<ApiVersionUrlRequestCondition> getCustomTypeCondition(Class<?> handlerType) {
        ApiVersion apiVersion = AnnotationUtils.findAnnotation(handlerType, ApiVersion.class);
        return createCondition(apiVersion);
    }
    @Override
    protected RequestCondition<ApiVersionUrlRequestCondition> getCustomMethodCondition(Method method) {
        ApiVersion apiVersion = AnnotationUtils.findAnnotation(method, ApiVersion.class);
        return createCondition(apiVersion);
    }
    @Override
    protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
        RequestMappingInfo info = createRequestMappingInfo(method);
        if (info != null) {
            RequestMappingInfo typeInfo = createRequestMappingInfo(handlerType);
            if (typeInfo != null) {
                info = typeInfo.combine(info);
            }
        }
        return info;
    }
    @Nullable
    private RequestMappingInfo createRequestMappingInfo(AnnotatedElement element) {
        RequestMapping requestMapping = AnnotatedElementUtils.findMergedAnnotation(element, RequestMapping.class);
        ApiVersion apiVersion = AnnotationUtils.findAnnotation(element, ApiVersion.class);
        if (apiVersion != null) {
            InvocationHandler invocationHandler = Proxy.getInvocationHandler(requestMapping);
            Class<? extends InvocationHandler> clazz = invocationHandler.getClass();
            try {
                Field declaredField = clazz.getDeclaredField(VALUE_CACHE_KEY);
                declaredField.setAccessible(true);
                Map<String, Object> valueCache = (Map) declaredField.get(invocationHandler);
                String[] path = requestMapping.path();
                if (path != null) {
                    for (int i = 0; i < path.length; i++) {
                        path[i] = VERSION_PREFIX + path[i];
                    }
                    valueCache.put(PATH_KEY, path);
                    declaredField.set(invocationHandler, valueCache);
                }
            } catch (NoSuchFieldException | IllegalAccessException e) {
                log.error("这是不可能发生的事。。", e);
            }
        }
        RequestCondition<?> condition = (element instanceof Class ?
                getCustomTypeCondition((Class<?>) element) : getCustomMethodCondition((Method) element));
        return (requestMapping != null ? createRequestMappingInfo(requestMapping, condition) : null);
    }
    private RequestCondition<ApiVersionUrlRequestCondition> createCondition(ApiVersion apiVersion) {
        return apiVersion == null ? null : new ApiVersionUrlRequestCondition(apiVersion.version());
    }
}
{% endcodeblock %}

## 配置RequestMappingHandlerMapping

自定义了`RequestMappingHandlerMapping`后，我们需要注册自定义处理类，让Spring容器使用自定义处理类进行url注册。注册自定义处理类的方法非常简单，只需要扩展`WebMvcRegistrations`，覆写`getRequestMappingHandlerMapping`方法即可。

{% codeblock ApiVersionUrlWebConfig.java lang:java %}
public class ApiVersionUrlWebConfig implements WebMvcRegistrations {
    @Override
    public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
        return new ApiVersionUrlRequestMappingHandlerMapping();
    }
}
{% endcodeblock %}

## 配置版本号前缀

考虑到有些人习惯将版本号定义为`v1`，有的人则习惯定义为`1`，我们还需要提供版本号前缀的配置项。

{% codeblock ApiVersionProperties.java lang:java %}
@Data
@ConfigurationProperties(prefix = "api.version")
public class ApiVersionProperties {
    /**
     * 全局版本号前缀
     */
    private String prefix = "v";
}
{% endcodeblock %}

## 自动配置

Spring Boot starter的核心思想是自动配置，因此，我们需要注册以上所有的bean到Spring容器中。

### 自动配置类

{% codeblock ApiVersionHttpConfiguration.java lang:java %}
@Configuration
@EnableConfigurationProperties
@ConditionalOnWebApplication
public class ApiVersionHttpConfiguration {
    @Bean
    public ApiVersionUrlWebConfig apiVersionUrlWebConfig() {
        return new ApiVersionUrlWebConfig();
    }
    @ConditionalOnMissingBean
    @Bean
    public ApiVersionProperties apiVersionProperties() {
        return new ApiVersionProperties();
    }
    @ConditionalOnMissingBean
    @Bean
    public ApiVersionUrlRequestCondition apiVersionUrlRequestCondition() {
        return new ApiVersionUrlRequestCondition();
    }
}
{% endcodeblock %}

### 配置清单

Spring规定了清单文件的路径resources/META-IN/spring.factories，我们需要在清单文件中标注自动配置类的路径

{% codeblock spring.factories lang:properties %}
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
priv.ln.api.version.spring.boot.autoconfigure.ApiVersionHttpConfiguration
{% endcodeblock %}

# 在Spring Boot项目中使用接口版本管理

在pom.xml中添加依赖，引用starter。

在Controler中添加注解
{% codeblock lang:java %}
@ApiVersion("1")
@Controller
@RequestMapping("/xxx")
public class XxxController{
}
{% endcodeblock %}

访问该接口时，通过url`/v1/xxx`访问