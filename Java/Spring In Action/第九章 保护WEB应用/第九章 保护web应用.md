[TOC]

# `spring security` 简介

`Spring Security`是基于Spring的应用程序提供声明式安全保护的安全性框架。它能够在**Web请求级别和方法调用级别处理身份认证和授权**。因为基于Spring框架，所以它充分利用了依赖注入和面向切面的技术。

## Spring security的几大模块

|        模块         |                             描述                             |
| :-----------------: | :----------------------------------------------------------: |
|         ACL         |            支持通过访问控制列表为域对象提供安全性            |
|    切面(Aspects)    | 当使用`Spring Security`注解时，会使用基于`AspectJ`的切面，而不是使用标准的`Spring AOP` |
| 配置(Configuration) |       包含通过XML和Java配置`Spring Security`的支持功能       |
|     核心(Core)      |                 提供`Spring Security`基本库                  |
| 加密(Cryptography)  |                  提供了加密和密码编码的功能                  |
|        LDAP         |                     支持基于LDAP进行认证                     |
|       OpenID        |                 支持使用OpenID进行集中式认证                 |
|       标签库        |                 `Spring Securiy`的JSP标签库                  |
|         Web         |     提供了`Spring Security`基于`Filter`的`Web`安全性支持     |

应用程序的类路径下至少要包含`Core`和`Configuration`两个模块。

## 过滤web请求

`DelegatingFilterProxy`是一个特殊的`Servlet Filter`，他将工作委托给一个`javax.servlet.Filter`的实现类，这个实现类作为一个`<bean>`注册在Spring应用的上下文中 ：

![DelegatingFilterProxy把Filter的处理委托给Spring应用上下文所定义的一个代理Filter bean](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-8-30/85586956.jpg)

在xml中配置如下所示：

```xml
<filter>
	<filer-name>springSecurityFilterChain</filer-name>
    <filter-class>
    	org.springframework.web.filter.DelegatingFilterProxy
    </filter-class>
</filter>
```

如果用java方式来配置的话 ：

```java
public class SecurityWebInitializer extends AbstractSecurityWebApplicationInitializer {
}
```

**`AbstractSecurityWebApplicationInitializer`实现了`WebApplicationInitializer`，因此Spring会发现它，并用它在Web容器中注册`DelegatingFilterProxy`，会拦截发往应用的请求，并将请求委托给ID为`springSecurityFilterChain`的bean** 。`springSecurityFilterChain`本身是另一个特殊的Filter，它也被称为`FilterChainProxy`，它可以链接任意一个或者其他的Filter 。

## 编写简单的安全性配置

```java
@Configuration
// 启动web安全功能
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter{
    
}
```

我们可以通过重载`WebSecurityConfigurerAdapter`中的一个或多个方法来实现。我们可以通过重载`WebSecurityConfigurerAdapter`的三个`configure()`方法来配置Web安全性：

|                   方法                    |                   描述                    |
| :---------------------------------------: | :---------------------------------------: |
|         `configure(WebSecurity)`          | 通过重载，配置`Spring Security`的Filter链 |
|         `configure(HttpSecurity)`         |   通过重载，配置如何通过拦截器保护请求    |
| `configure(AuthenticationManagerBuilder)` |      通过重载，配置`user-detail`服务      |

默认的`configuer(HttpSecurity)`实际上为 ：
```java
protected void configure(HttpSecurity http) throws Exception{
    http
        .authorizeRequests()
        	.anyRequest().authenticated()
        	.and()
        .formLogin().and()
        .httpBasic();
}
```

这个默认配置指定了该如何保护HTTP请求，以及客户端认证用户的方案。通过调用`authorizeRequest()`和`anyRequest().authenticated()`就会要求所有进入应用的HTTP请求都要进行认证。

同时，因为我们没有配置`configure(AuthenticationManagerBuilder)`方法，所以没有用户存储支撑认证过程。没有用户存储，实际上就等于没有用户。

# 选择查询用户详细信息的服务

## 使用基于内存的用户存储

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
	auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
		.withUser("admin").password(new BCryptPasswordEncoder().encode("admin"))
        .roles("USER","ADMIN")
	.and()
		.withUser("user").password(new BCryptPasswordEncoder().encode("user"))
        .roles("USER");
}
```

通过调用`inMemoryAuthentication()`就能弃用内存用户存储，调用`withUser()`方法为内存用户存储新的用户 。

`roles()`方法是`authorities()`方法的简写形式。**`roles()`方法所给定的值都会添加一个`ROLE_`前缀，并将其作为权限授予给用户 。**

## 基于数据库表进行认证

```java
protected void configure(AuthenticationManagerBuilder auth) throws Exception{
    auth.jdbcAuthentication().dataSource(datasource)
        .usersByUsernameQuery("SELECT username,password,true FROM spitter where username = ?")
        .authoritiesByUsernameQuery("SELECT username,'ROLE_USER' from spitter where username = ?")
        .passwordEncoder(new BCryptPasswordEncoder());
}
```

## 配置自定义的用户服务

```java
// ...
@Autowired
private SpitterRepository spitterRepository;
// ...
protected void configure(AuthenticationManagerBuilder auth) throws Exception{
    auth.userDetailsService(spitterRepository).passwordEncoder(new BCryptPasswordEncoder());
}
```

```java
@Repository
public class JdbcSpitterRepository implements SpitterRepository {
    // ...
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        Spitter spitter = findOneByUserName(s);
        if (spitter != null) {
            List<GrantedAuthority> authorities = new ArrayList<>();
            authorities.add(new SimpleGrantedAuthority("ROLE_USER"));
            return new User(spitter.getUserName(),spitter.getPassWord(),authorities);
        }

        throw new UsernameNotFoundException("用户名为 : " + s + "没有找到.");
    }
}
```

为了防止字符串乱码，还需要配置如下内容：

```java
public class SecurityWebInitializer extends AbstractSecurityWebApplicationInitializer{
    @Override
    protected void beforeSpringSecurityFilterChain(ServletContext servletContext){
        FilterRegistration.Dynamic filter = servletContext.addFilter("encodingFilter",new CharacterEncodingFilter("utr-8"),true);
        filter.addMappingForUrlPatterns(null,false,"/*");
    }
}
```




















































































