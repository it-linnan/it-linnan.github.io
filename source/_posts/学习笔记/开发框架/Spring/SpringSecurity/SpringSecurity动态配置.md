---
title: Spring Security动态配置
date: 2018-11-08 09:28:08
tags:
- Spring
- Spring Security
categories:
- 学习笔记
- 开发框架
- Spring
- Spring Security
keywords:
- Spring
- Spring Security
- Java Config
---

我们在开发Web应用时，通常希望保护某些资源（页面或数据），对这些资源做安全控制，比如：未登录的用户访问资源时，自动跳转至登录页面；拥有管理员身份的用户，可以访问某些管理界面。

本文介绍一种基于Spring Security的动态配置权限的方法，使用的是JavaConfig的方式。

{% blockquote  Spring https://spring.io/projects/spring-security#overview Spring Security %}
Spring Security是一个功能强大且可高度自定义的身份验证和访问控制框架。具有以下特性：

1. 对身份验证和授权的全面和可扩展的支持
2. 防止会话固定，点击劫持，跨站点请求伪造等攻击
3. Servlet API集成
4. 可选与Spring Web MVC集成
5. 还有很多啊..

{% endblockquote %}
<!-- more -->

# 项目中引入Spring Security依赖

我创建的是Maven项目，在pom文件中添加如下依赖：
{% codeblock pom.xml lang:xml %}
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-core</artifactId>
    <version>${spring-security.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>${spring-security.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>${spring-security.version}</version>
</dependency>
{% endcodeblock %}

# 在web应用中启用Spring Security

## Servlet3.0+

如果应用部署在支持Servlet3.0+环境下，可以使用**AbstractSecurityWebApplicationInitializer**启用Spring Security，代码清单如下：

{% codeblock Spring环境下启用Spring Security lang:java  %}
public class SecurityWebApplicationInitializer extends AbstractSecurityWebApplicationInitializer {
}
{% endcodeblock %}

{% codeblock 非Spring环境下启用Spring Security lang:java  %}
public class SecurityWebApplicationInitializer extends AbstractSecurityWebApplicationInitializer {
    public SecurityWebApplicationInitializer() {
        super(SecurityConfig.class);
    }
}
{% endcodeblock %}

以上，可以理解为，Spring自动注册了某些filter。

## Servlet2.5

一些主流的中间件，仅支持Servlet2.5，比如WebLogic、Web Sphere。如果应用需要部署在这些中间件下，则不能使用以上方式，需要在web.xml中自行配置filter。代码清单如下：
{% codeblock web.xml配置 lang:xml  %}
<filter>
  <filter-name>springSecurityFilterChain</filter-name>
  <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>

<filter-mapping>
  <filter-name>springSecurityFilterChain</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
{% endcodeblock %}

# 基于Spring Security进行安全配置

## 启用Spring Security配置

首先创建一个继承**WebSecurityConfigurerAdapter**抽象类的类，这样就获得了Spring Security提供的默认配置。然后使用**@Configuration**注解使这个类成为Spring的一个配置类，Spring启动时会读取类中的配置。最后使用**@EnableWebSecurity**注解启用Spring Security。

**WebSecurityConfigurerAdapter**抽象类，提供了一个方便的、开箱即用的Spring Security配置。该配置创建一个名为springSecurityFilterChain的Servlet过滤器，它负责应用程序中的所有安全性（保护应用程序URL，验证提交的用户名和密码，重定向到登录表单等）。想要实现自定义配置，只需要覆写相应的方法。

{% codeblock 安全配置类 lang:java  %}
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}
{% endcodeblock %}

接下来，将介绍Spring Security的具体配置，比如如何配置允许哪些用户登录，如何配置用户权限等。

## 具体配置

实现自定义配置，需要覆写基类中的configure方法。
{% codeblock configure lang:java  %}
@Override
protected void configure(HttpSecurity http) throws Exception {
    // TODO:这里写你的具体配置
}
{% endcodeblock %}

### 登录配置

假设你有如下需求：

1. Web应用需要支持表单登录
1. 登录界面或登录请求使用的是一个自定义的url
1. 登录请求中，提交的用户名、密码字段名是自定义的
1. 登录请求提交的参数中除用户名密码以外，还需要验证其他参数
1. 自定义登录成功或失败的后续处理：比如登录成功时，先提示消息，再跳转到首页。

以下代码可以实现上述需求：

{% codeblock 登录配置 lang:java  %}
http.formLogin()
        .loginPage("/login").loginProcessingUrl("/system/login.{ext}")
        .usernameParameter("user_id").passwordParameter("user_password")
        .authenticationDetailsSource(new GetIpWebAuthenticationDetailsSource())
        .successHandler(new SupportAjaxAuthSuccessHandler()).failureHandler(new SupportAjaxAuthFailHandler();
{% endcodeblock %}

接下来，详细说明每行配置的作用。

#### formLogin

**formLogin**方法告知Spring Security开启表单登录，即以一种用户名密码登录的方式。

#### loginPage和loginProcessingUrl

**loginPage**指定登陆页面的url是**/login**，这个配置的作用是当用户未登录时，请求将重定向到该url上，现象是跳转回登录页面。

**loginProcessingUrl**指定登录请求的url是**/system/login.{ext}**，可以通过Ajax的方式将用户名和密码提交到该url上，即一次登录请求。注意，Spring Security默认登录请求是POST类型。
{% codeblock 登录请求 lang:js  %}
$.ajax({
    url : "/system/login.json",
    type : "POST"
});
{% endcodeblock %}

以上两个url中，都可以使用通配符，比如我使用的"**{ext}**，也就是说，可以url可以是**/system/login.json**或**/system/login.xml**，Spring Security会处理符合这个url的所有请求。

#### usernameParameter和passwordParameter

指定登录请求中，提交的用户名、密码字段名分别是**user_id**和**user_password**。
{% codeblock 登录请求 lang:js  %}
$.ajax({
    url : "/system/login.json",
    type : "POST",
    data : {
        'user_id': userName,
        'user_password' : password
    }
});
{% endcodeblock %}

#### authenticationDetailsSource

使用一个自定义的AuthenticationDetailsSource，指定登录请求中，获取哪些数据作为凭证，供验证器验证。

Spring Security通过**WebAuthenticationDetails**类，记录web请求的客户端ip地址和sessionId。它在获取IP地址时，只是简单的调用了**HttpServletRequest**的**getRemoteAddr**方法，在集群环境下，这样获取不能获取到客户端的真实IP，因此我覆写了获取客户端ip地址部分代码，代码清单如下：
{% codeblock GetIpWebAuthenticationDetailsSource lang:java  %}
public class GetIpWebAuthenticationDetailsSource extends WebAuthenticationDetailsSource {
    public GetIpWebAuthenticationDetailsSource() {
    }
    public WebAuthenticationDetails buildDetails(HttpServletRequest context) {
        // 根据请求内容 构造本次请求的详细信息
        return new GetIpWebAuthenticationDetails(context);
    }
}
{% endcodeblock %}

{% codeblock GetIpWebAuthenticationDetails lang:java  %}
public class GetIpWebAuthenticationDetails extends WebAuthenticationDetails {
    private final String remoteAddress;
    public GetIpWebAuthenticationDetails(HttpServletRequest request) {
        super(request);
        // 重新获取客户端地址
        this.remoteAddress = ServletUtil.getIpAddr(request);
    }
    @Override
    public String getRemoteAddress() {
        return this.remoteAddress;
    }
}
{% endcodeblock %}

还可以在自定义的**WebAuthenticationDetails**实现类中，添加更多的属性，用来记录需要验证的参数

#### successHandler和failureHandler

**successHandler**方法指定了登录成功处理器，这里我返回了一个登陆成功的提示信息，同时将用户信息存入session中。
**failureHandler**方法指定了登录失败处理器，这里我返回了一个登录失败的具体的提示信息，比如用户不存在、密码错误。实际上是不推荐这样做，提示信息应该是用户名或密码错误。
{% codeblock successHandler lang:java  %}
public class SupportAjaxAuthSuccessHandler extends SimpleUrlAuthenticationSuccessHandler {
    private final static String RIGHT_CREDENTIALS = "rightCredentials";
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        // 直接返回成功信息
        SecurityHandlerUtil.successResponse(response, RIGHT_CREDENTIALS);
        UserContext user = new UserContext();
        user.setOperId(authentication.getName());
        // 用户信息存入session
        request.getSession().setAttribute("user", user);
    }
}
{% endcodeblock %}

{% codeblock failureHandler lang:java  %}
public class SupportAjaxAuthFailHandler extends SimpleUrlAuthenticationFailureHandler {
    private static final String NOT_FOUND = "notFound";
    private static final String BAD_CREDENTIALS = "badCredentials";
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        // 直接返回错误信息
        if (exception instanceof UsernameNotFoundException) {
            SecurityHandlerUtil.errorResponse(response, NOT_FOUND);
        } else if (exception instanceof BadIpAuthenticationException) {
            SecurityHandlerUtil.errorResponse(response, exception);
        } else {
            SecurityHandlerUtil.errorResponse(response, BAD_CREDENTIALS);
        }
    }
}
{% endcodeblock %}

### 登录数据源配置

一般来说，用户都是存在表中，Spring Security提供了多种数据源，开箱即用。这里我通过扩展**DaoAuthenticationProvider**，获取基类的所有功能，并新增了一个IP白名单的功能，只有IP白名单内的客户端可以使用正确的用户名密码登录。此外，还需要指定从哪个表里查询，查询哪些字段等。为此，Spring Security规定了一个**UserDetailsService**接口，并提供了一系列开箱即用的实现类，而这里我使用了自己的实现类。

配置登录数据源时，需要覆写**configure**的另外一个重载方法，代码清单如下：

{% codeblock 登录数据源配置 lang:java  %}
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.authenticationProvider(authenticationProvider);
}
@Bean
protected AuthenticationProvider authenticationProvider(UserDetailsService apiPlatUserService) {
    IpAuthenticationProvider authenticationProvider = new IpAuthenticationProvider();
    // 指定用户信息业务实现
    authenticationProvider.setUserDetailsService(apiPlatUserService);
    // 指定密码加密方式
    authenticationProvider.setPasswordEncoder(new Md5PasswordEncoder());
    // 显示用户不存在异常
    authenticationProvider.setHideUserNotFoundExceptions(false);
    return authenticationProvider;
}
{% endcodeblock %}

{% codeblock UserDetailsService实现类 lang:java  %}
@Service("apiPlatUserService")
public class UserServiceImpl implements UserDetailsService {
    @Autowired
    private BaseDao baseDao;
    private List<GrantedAuthority> authorities;
    @PostConstruct
    private void init() {
        // 我的应用中没有角色的概念，因此默认为每个用户分配一个管理员角色
        authorities = new ArrayList<GrantedAuthority>();
        authorities.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
    }
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        APP005 app005 = baseDao.selectOneBySql("APP005.selectByPrimaryKey", username, APP005.class);
        if (app005 != null) {
            return new User(app005.getLoginid(), app005.getPassword(), authorities);
        }
        throw new UsernameNotFoundException("用户" + username + "不存在");
    }
}
{% endcodeblock %}

{% codeblock IpAuthenticationProvider lang:java  %}
public class IpAuthenticationProvider extends DaoAuthenticationProvider {
    @Autowired
    private SecurityConfigCache securityConfigCache;
    /**
     * ip匹配器集合
     */
    private List<IpAddressMatcher> ipAddressMatchers;
    @PostConstruct
    private void init() {
        // 获取安全配置缓存中的ip白名单 创建ip匹配器集合
        String ipWhileList = securityConfigCache.getIpWhileList();
        if (StringUtils.isNotEmpty(ipWhileList)) {
            String[] ips = ipWhileList.split(SecurityConfigCache.SEPARATOR);
            ipAddressMatchers = new ArrayList<IpAddressMatcher>(ips.length);
            for (String ip : ips) {
                IpAddressMatcher ipAddressMatcher = new IpAddressMatcher(ip);
                ipAddressMatchers.add(ipAddressMatcher);
            }
        }
    }
    @Override
    protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
        // 先校验IP
        if (ipAddressMatchers != null && ipAddressMatchers.size() > 0) {
            Object details = authentication.getDetails();
            if (details != null && details instanceof WebAuthenticationDetails) {
                WebAuthenticationDetails webAuthenticationDetails = (WebAuthenticationDetails) details;
                // 注意 这里将获得在GetIpWebAuthenticationDetails中重新获取的真实IP
                String remoteAddress = webAuthenticationDetails.getRemoteAddress();
                boolean isValid = false;
                // 遍历ip匹配器集合 有一个匹配则通过
                for (IpAddressMatcher ipAddressMatcher : ipAddressMatchers) {
                    if (ipAddressMatcher.matches(remoteAddress)) {
                        isValid = true;
                        break;
                    }
                }
                if (!isValid) {
                    throw new BadIpAuthenticationException(messages.getMessage("badIp", new String[]{remoteAddress}));
                }
            } else {
                logger.debug("请求附加信息中未获取到ip,咋回事儿呢...");
                throw new BadIpAuthenticationException(messages.getMessage("nullIp"));
            }
        }
        // 校验用户名密码是否正确
        super.additionalAuthenticationChecks(userDetails, authentication);
    }
}
{% endcodeblock %}

### 登出配置

假设你有如下需求：

1. Web应用需要支持登出
1. 登出请求使用的是一个自定义的url
1. 自定义登出成功或失败的后续处理：比如登出成功时，先提示消息，再跳转到登录页

以下代码可以实现上述需求：

{% codeblock 登出配置 lang:java  %}
http.logout()
        .logoutUrl("/system/logout.{ext}")
        .logoutSuccessHandler(new SupportAjaxLogoutSuccessHandler());
{% endcodeblock %}

接下来，详细说明每行配置的作用。

#### logout

**logout**方法告知Spring Security支持登出处理，登出时，Spring Security自动清除用户缓存并使session失效。

#### logoutUrl

**logoutUrl**指定登出请求的url是**/system/logout.{ext}**，可以通过Ajax的方式请求该url上，即一次登出请求。注意，Spring Security默认登出请求是POST类型。
{% codeblock 登出请求 lang:js  %}
$.ajax({
    url : "/system/logout.json",
    type : "POST"
});
{% endcodeblock %}

### 资源访问限制

一般来说，web应用都会做角色-菜单配置，即拥有某身份的用户可以访问响应的资源。而这个配置，通常不是硬编码的，可能使在表中配置的，也可能是在配置文件中配置的。代码清单如下：

{% codeblock 资源权限配置 lang:java  %}
ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry authorizeRequests = http.authorizeRequests();
// 获取资源-权限映射 权限使用Spring Security提供的安全表达式
Map<RequestMatcher, String> requestConfigs = securityConfigCache.getRequestConfigs();
for (Map.Entry<RequestMatcher, String> requestConfig : requestConfigs.entrySet()) {
    authorizeRequests.requestMatchers(requestConfig.getKey()).access(requestConfig.getValue());
}
{% endcodeblock %}

其中，securityConfigCache是一个由Spring管理的bean，缓存了表中的配置。在初始化该bean时，从表中查询角色-菜单配置，构造一个资源-权限映射。

### 自定义异常处理器

{% codeblock 自定义异常处理器 lang:java  %}
http.exceptionHandling()
        .accessDeniedHandler(new UnauthorizedAccessDeniedHandler())
        .authenticationEntryPoint(new UnauthorizedEntryPoint());
{% endcodeblock %}

#### accessDeniedHandler

**accessDeniedHandler**方法指定了拒绝请求处理器。也就是指定当用户无权限访问资源时，如何对请求做出响应。

请求大体分为两种，一种是请求页面，一种是请求数据。而对这两种请求的拒绝处理也是不一样的，请求页面时，需要将请求重定向到登录界面，提示用户登录；请求数据时，则返回一个错误信息，提示用户无权访问。代码清单如下：

{% codeblock UnauthorizedAccessDeniedHandler lang:java  %}
public class UnauthorizedAccessDeniedHandler implements AccessDeniedHandler {
    protected final Logger LOGGER = LogManager.getLogger(getClass());
    public static final String UNAUTHORIZED = "unauthorized";
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException exception) throws IOException, ServletException {
        LOGGER.error("当前登录用户无权限访问", exception);
        SecurityHandlerUtil.handleAnyUnthorizedRequest(request, response, UNAUTHORIZED);
    }
}
{% endcodeblock %}

{% codeblock SecurityHandlerUtil lang:java  %}
@Component
public class SecurityHandlerUtil {
    public static void handleAnyUnthorizedRequest(HttpServletRequest request, HttpServletResponse response, String errorCode) throws IOException {
        // 请求为ajax的情况 返回错误信息
        if (SecurityHandlerUtil.isAjaxRequest(request)) {
            SecurityHandlerUtil.errorResponse(response, errorCode);
        } else {
            // 其他情况 跳转回登录页
            response.sendRedirect(request.getContextPath() + SecurityConfig.LOGIN);
        }
    }
    public static boolean isAjaxRequest(HttpServletRequest request) {
        String ajaxFlag = request.getHeader("X-Requested-With");
        return ajaxFlag != null && "XMLHttpRequest".equals(ajaxFlag);
    }
    public static void errorResponse(HttpServletResponse response, String msgCode) throws IOException {
        writeResponse(response, "9999", getMessage(msgCode));
    }
    public static void successResponse(HttpServletResponse response, String msgCode) throws IOException {
        writeResponse(response, "0000", getMessage(msgCode));
    }
    public static void writeResponse(HttpServletResponse response, String statusCode, String msg) throws IOException {
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("code", statusCode);
        jsonObject.put("msg", msg);
        response.setCharacterEncoding("utf-8");
        response.setContentType("application/json; charset=utf-8");
        PrintWriter writer = response.getWriter();
        writer.write(jsonObject.toString());
    }
    public static String getMessage(String code) {
        return messageSource.getMessage(code, null, null);
    }
    @Autowired
    private void setMessageSource(MessageSource messageSource) {
        SecurityHandlerUtil.messageSource = messageSource;
    }
}
{% endcodeblock %}

#### authenticationEntryPoint

**authenticationEntryPoint**方法指定了用户未登录时的入口点。由于用户尚未通过身份验证，因此需要返回一个响应，指示用户必须先进行身份验证。响应同样是根据请求类型决定，如果请求页面，将重定向到登录页；如果请求数据，将返回错误消息。

{% codeblock UnauthorizedEntryPoint lang:java  %}
public class UnauthorizedEntryPoint implements AuthenticationEntryPoint {
    protected final Logger LOGGER = LogManager.getLogger(getClass());
    public static final String CREDENTIALS_EXPIRED = "credentialsExpired";
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        LOGGER.error("用户未登录", authException);
        SecurityHandlerUtil.handleAnyUnthorizedRequest(request, response, CREDENTIALS_EXPIRED);
    }
}
{% endcodeblock %}

### 自定义错误信息数据源

{% codeblock 错误信息数据源 lang:java  %}
@Bean
protected ResourceLoaderAware messageSource() {
    ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
    messageSource.setBasename("classpath:spring/security/messages");
    return messageSource;
}
{% endcodeblock %}

### 攻击保护配置

#### 同源访问

{% codeblock 同源访问 lang:java  %}
http.headers()
        .addHeaderWriter(new XFrameOptionsHeaderWriter(XFrameOptionsHeaderWriter.XFrameOptionsMode.SAMEORIGIN));
{% endcodeblock %}

#### 防crsf

{% codeblock 防crsf lang:java  %}
http.crsf();
{% endcodeblock %}

#### session保护

{% codeblock session保护 lang:java  %}
http.sessionManagement()
        .sessionFixation().migrateSession();
{% endcodeblock %}