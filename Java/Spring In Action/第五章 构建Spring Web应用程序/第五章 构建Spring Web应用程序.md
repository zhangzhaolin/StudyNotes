第五章 构建Spring Web应用程序

[TOC]

## 代码

本章相关代码存放在GIT中 ：[第五章 构建Spring Web应用程序](https://github.com/zhangzhaolin/spring-demo/tree/master/%E7%AC%AC%E4%BA%94%E7%AB%A0%20%E6%9E%84%E5%BB%BASpring%20MVC%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F/%E6%90%AD%E5%BB%BA%E6%9C%80%E5%9F%BA%E7%A1%80%E7%9A%84Spring%20MVC%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F)

## 控制器、模型和视图

- 控制器 ：控制器是一个用于处理请求的Spring组件。在典型的应用程序中可能会有多个控制器。
- 模型 ：控制器在完成逻辑处理后，通常会产生一些信息，这些信息需要返回给用户并在浏览器中显示。这些信息被称为模型。
- 视图 ： 信息需要发送给视图，通常是JSP或者Thymeleaf 等（现在都用HTML了）

## 跟踪Spring MVC请求

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-8-21/34601160.jpg)

- 请求旅程的第一站是`DispatcherServlet`（前端控制器）。
- 请求通过查询一个或者多个**处理器映射**（`handler mapping`）来确定请求的下一站在哪里（需要知道将请求发送给哪一个控制器）
- `DispatcherServlet`会将请求发送给选中的控制器。到了控制器，请求会卸下其**负载**（用户提交信息）并耐心等待控制器处理这些信息。（设计良好的控制器本身只是处理很少甚至不处理工作）
- 控制器所做的最后一件事情就是将模型数据打包，并且标示用于渲染输出的视图名。接下来会将请求连同模型和视图名发送回`DispatcherServlet`。
- 传递给`DispatcherServlet`的视图名仅仅传递了一个**逻辑名称**，这个名字将会查找产生结果的真正视图。`DispatcherServlet`会使用**视图解析器**来将逻辑视图名匹配为一个特定的视图实现。
- `DispatcherServlet`最后一站是视图的实现，在这里他交付模型数据。

## 搭建Spring MVC

### 配置`DispatcherServlet`

```java
public class SpittrWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{RootConfig.class};
    }

    // 指定配置类
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    // 将DispatcherServlet映射到"/"
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
```

### `AbstractAnnotationConfigDispatcherServletInitializer`剖析

在Servlet3.0环境中 ，容器会在类路径中查找实现`ServletContainerInitializer`接口的类，如果能发现的话，那么就会用它来配置`Servlet`容器。Spring提供了这个接口的实现 ：

```java
@HandlesTypes(WebApplicationInitializer.class)
public class SpringServletContainerInitializer implements ServletContainerInitializer {
	// ...
}
```

`SpringServletContainerInitializer`这个类反过来会找实现了`WebApplicationInitializer`基础实现，也就是`AbstractAnnotationConfigDispatcherServletInitializer`。因为我们的`SpittrWebAppInitializer`扩展了`AbstractAnnotationConfigDispatcherServletInitializer`（同时也实现了`WebApplicationInitializer`），因此当部署到Servlet3.0时，容器会自动发现它，并用它来配置应用上下文。

`SpittrWebAppInitializer`类中的第一个方法是`getServletMapping()`，它会将一个或者多个路径映射到`DispatcherServlet`上。在本例中，它映射的是"/"，这表明它会是应用默认的Servlet，它会处理进入应用的**所有请求**。

### 两个应用上下文的故事

在Spring Web应用中，有两种应用上下文 ：`DispatcherServlet`和`ContextLoaderListener`。

我们希望`DispatherServlet`加载包含WEB组件的bean，如控制器、视图解析器以及处理器映射；而`ContextLoaderListener`要加载应用中的其他bean，例如数据层组件、驱动应用后端的中间层等。

事实上，`AbstractAnnotationConfigDispatcherServletInitializer`会同时实现`DispatcherServlet`以及`ContextLoaderListener`。`getServletConfigClass()`方法返回的带有`@Configuration`的类将用来定义`DispatcherServlet`应用上下文的bean。`getRootConfigClass()`方法带有的`@Configuration`的类将会用来配置`ContextLoaderListener`创建的应用上下文的bean。

在本例中，`ContextLoaderListener`定义在`RootConfig.class`中，而`DispatcherServlet`定义在`WebConfig.class`中

> WebConfig.class

```java
@Configuration
@EnableWebMVC
@ComponentScan(basePackages = "spittr.web")
public class WebConfig implements WebMvcConfigurer{
    
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer){
        configurer.enable();
    }
    
    @Bean
    public ViewResolver viewResolver(){
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".html");
        resolver.setExposeContextBeansAsAttributes(true);
        return resolver;
    }
    
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/")
                .addResourceLocations("/resources/**");
    }
    
}
```

> RootConfig.class

```java

@Configuration
@Import({DataConfig.class})
@ComponentScan(basePackages = "spittr" ,
    excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION , value = EnableWebMvc.class)
    }
)
public class RootConfig {

}
```

> DataConfig.class —— 数据源配置

```java
@Configuration
@PropertySource("classpath:/application.properties")
public class DataConfig {

    private final Environment environment;

    @Autowired
    public DataConfig(Environment environment) {
        this.environment = environment;
    }

    @Bean
    public DataSource dataSource() throws SQLException {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDbType(environment.getProperty("datasource.dbtype"));
        dataSource.setUrl(environment.getProperty("datasource.url"));
        dataSource.setUsername(environment.getProperty("datasource.username"));
        dataSource.setPassword(environment.getProperty("datasource.password"));
        // 配置WallFilter
        dataSource.setFilters("wall");
        // PSCache
        dataSource.setPoolPreparedStatements(false);
        return dataSource;
    }

    @Bean
    public JdbcOperations jdbcOperations(DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }

}
```

## 编写基本的控制器

```java
@Controller
@RequestMapping(value = "/")
public class HomeController {

    @GetMapping
    public String home(){
        return "home";
    }
}
```

记得还要在pom.xml中加入这一句话 ：

```xml
<packaging>war</packaging>
```

> home.html —— 使用的时semantic ui

```html
<body>
    <div class="ui container">
        <div class="ui two column centered grid" style="padding-top: 5%;">
            <div class="center aligned column">
                <h1>欢迎来到Spittr</h1>
            </div>
            <div class="column row">
                <div class="center aligned column">
                    <a href="/spittles">Spittles</a>
                    ||
                    <a href="/spitter/register">注册</a>
                </div>
            </div>
        </div>
    </div>
</body>
```

之后运行程序，效果图如下所示 ：

![首页](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-8-23/3222114.jpg)

### 传递模型到视图中

> SpittleRepository.java

```java
public interface SpittleRepository{
    /**
     * @param max Spittle中ID的最大个数
     * @param count 最多返回的Spittle个数
     * @return 
     */
    List<Spittle> findSpittles(long max,int count);
}
```

> JdbcSpittleRepository.java

```java
public class JdbcSpittleRepository implements SpittleRepository{
    
    private final JdbcOperations jdbcOperations;
    
    @Autowired
    public JdbcOperations(JdbcOperations jdbcOperations){
        this.jdbcOperations = jdbcOperations;
    }
    
    public List<Spittle> findSpittles(long max,int count){
        return jdbcOperations.query("SELECT * FROM spittle WHERE ID < ? ORDER BY created_at DESC LIMIT ?",new SpittleRowMap(),max,count);
    }
    
    private class SpittleRowMap implements RowMapper<Spittle>{
        @Override
        public Spittle mapRow(ResultSet rs , int rowNum) throws SQLException {
            return new Spittle(
                    rs.getLong("id"),
                    rs.getString("message"),
                    rs.getTimestamp("created_at"),
                    rs.getDouble("latitude"),
                    rs.getDouble("longitude")
            );
        }
    }
}
```

> Spittle.java

```java
public class Spittle {
    private final Long id;
    private final String message;
    private final Timestamp time;
    private Double latitude;
    private Double longitude;
    public Spittle(String message , Timestamp time) {
        this(null,message,time,null,null);
    }
    public Spittle(Long id,String message , Timestamp time , Double latitude , Double longitude) {
        this.id = id;
        this.message = message;
        this.time = time;
        this.latitude = latitude;
        this.longitude = longitude;
    }
	// ...
}
```

> SpittleController.java

```java
@Controller
@RequestMapping(value = "/spittles")
public class SpittleController {

    private SpittleRepository spittleRepository;

    @Autowired
    public SpittleController(SpittleRepository spittleRepository){
        this.spittleRepository = spittleRepository;
    }
    @GetMapping
    public String spittlesGet(){
        return "spittles";
    }
    @PostMapping
    @ResponseBody
    public JsonUtils spittlesPost(){
        return new JsonUtils(ResultEnum.SUCCESS,null,
                spittleRepository.findSpittles(Long.MAX_VALUE,20));
    }
}
```

> JsonUtils.java

```java
public class JsonUtils {
    private ResultEnum result;
    private String message;
    private Object data;
    public JsonUtils(ResultEnum result , String message , Object data) {
        this.result = result;
        this.message = message;
        this.data = data;
    }
	// ...
}
```

为了能够解析`JsonUtils`，我们还需要引入`json`包 ：

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
	<version>2.9.6</version>
</dependency>
```

前后端交互过程中，我试着加入了最近比较火的`Vue`：

> spittles.html

```html
<body>
    <div class="ui container">
        <div class="ui column centered grid" style="padding-top: 5%">
            <div class="column">
                <div class="ui ordered list" id="spittlesList">
                    <div class="item" v-for="spittle in spittles">
                        {{ spittle.message }}
                        <div class="description">
                            {{ spittle.time | formateDate }}
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>
```

> spittles,js

```javascript
var spittlesList;
$(function () {
    $.ajax({
        type: 'POST',
        url: "/spittles",
        success: function (result) {
            spittlesList = new Vue({
                el : '#spittlesList',
                data : {
                    spittles : result.data
                },
                filters : {
                    formateDate : function (value) {
                        moment.locale("zh_cn");
                        return moment(value).format("lll");
                    }
                }
            })
        },
        dataType: "json",
        error : function (error) {
        }
    });
});
```

效果图如下所示 ：

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-8-24/45708507.jpg)

## 接受请求的输入

### 处理查询参数

```java
public JsonUtils spittlesPost(
            @RequestParam(value = "max" , defaultValue = MAX_LONG_AS_STRING) long max,
            @RequestParam(value = "count" , defaultValue = "10") int count){
        return new JsonUtils(ResultEnum.SUCCESS,null,
                spittleRepository.findSpittles(max,count));
}
```

### 通过路径参数查询

```java
@PostMapping(value = "/show/{spittleId}")
@ResponseBody
public JsonUtils spittlePost(@PathVariable Long spittleId){
	return new JsonUtils(ResultEnum.SUCCESS,null,spittleRepository.findSpittle(spittleId));
}
```

## 处理表单

```java
@Controller
@RequestMapping(value = "/spitter")
public class SpitterController {

    private final SpitterRepository spitterRepository;

    @Autowired
    public SpitterController(SpitterRepository spitterRepository) {
        this.spitterRepository = spitterRepository;
    }

    @GetMapping(value = "/register")
    public String showRegisterForm(){
        return "registerForm";
    }

    @PostMapping(value = "/register")
    @ResponseBody
    public JsonUtils processRegisteration(Spitter spitter){
        spitterRepository.save(spitter);
        return new JsonUtils(ResultEnum.SUCCESS,"","/spitter/user/" + spitter.getUserName());
    }

    @GetMapping(value = "/user/{username}")
    public String showSpitterProfile(){
        return "profile";
    }

    @PostMapping(value = "/user/{username}")
    @ResponseBody
    public JsonUtils spitterProfile(@PathVariable String username){
        return new JsonUtils(ResultEnum.SUCCESS,null,
                spitterRepository.findOneByUserName(username));
    }
}
```

```java
public class Spitter {
	private Long id;
    private String firstName;
    private String lastName;
    private String userName;
    private String passWord;
    private String email;
    // ....
}
```

当`InternalResourceViewResolver`看到视图中的`redirect:`前缀的时候，它就知道要将其解析为重定向的规则，而不是视图的名称。除了可以识别`redirect:`前缀，`InternalResourceResolver`还可以识别`forward:`前缀，请求将会前往指定的路径。

### 校验表单

在校验表单之前，我们需要先导入两个包 ：

```xml
<!-- https://mvnrepository.com/artifact/javax.validation/validation-api -->
        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>2.0.1.Final</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.hibernate.validator/hibernate-validator -->
        <dependency>
            <groupId>org.hibernate.validator</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>6.0.12.Final</version>
        </dependency>
```

之后，我们在实体类中加入如下注解 ：

```java
	@NotNull
    @Size(min = 2 , max = 30)
    private String firstName;

    @NotNull
    @Size(min = 2 , max = 30)
    private String lastName;

    @NotNull
    @Size(min = 5 , max = 16)
    private String userName;

    @NotNull
    @Size(min = 5 , max = 25)
    private String passWord;

    private String email;
```

Controller控制层中 ：

```java
@PostMapping(value = "/register")
@ResponseBody
public JsonUtils processRegisteration(@Valid Spitter spitter, Errors error){
	if (error.hasErrors()){
    	return new JsonUtils(ResultEnum.ERROR,"","/spitter/register");
	}
    spitterRepository.save(spitter);
    return new JsonUtils(ResultEnum.SUCCESS,"","/spitter/user/" + spitter.getUserName());
}
```





                                                                                                                                                                                                                                                                      