[TOC]

# Spring MVC配置的替代方案

## 自定义`DispatcherServlet`配置

```java
public class WebApplicationInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{RootConfig.class};
    }

    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

}
```

在`AbstracAnnotationConfigDispatcherInitializer`将`DispatcherServlet`注册到`servlet`容器之后，就会调用`customizeRegistration()`，并将`servlet`注册后得到的`Registration.Dynamic`传递进来。通过重载`customizeRegistration()`方法，我们可以对`DispatcherServlet`进行额外的配置 。

例如，我们需要处理`multipart`请求和文件上传 ，那么，我们可以重载`customizeRegistration()`方法来设置`MultipartConfigElement`

```java
@Overirde
protected void customizeRegistration(ServletRegistration.Dynamic registration){
    registration.setMultipartConfig(new MultipartConfigElement("/tmp/spittr/uploads"));
}
```

除了设置`multipart`外，还可以进行以下配置：

```java
public interface Dynamic extends ServletRegistration, javax.servlet.Registration.Dynamic {
	void setLoadOnStartup(int var1);
	Set<String> setServletSecurity(ServletSecurityElement var1);
	void setMultipartConfig(MultipartConfigElement var1);
	void setRunAsRole(String var1);
}
```

## 添加其他的`Servlet`和`Filter`

通过继承`AbstractAnnotationConfigDispatcherServletInitializer`来创建`DispatcherServlet`和`ContextLoaderListener`。如果想要注册其他的`Servlet`、`Filter`或者`Listener`的话，最简单的一个方法是实现`WebApplicationInitializer`接口并注册一个`Servlet`：

```java
public class MyServletInitializer implements WebApplicationInitializer {
    public void onStartup(ServletContext servletContext) throws ServletException {
        // 注册servlet
        ServletRegistration.Dynamic myServlet = servletContext.addServlet("myServlet",MyServlet.class);
        // 映射servlet
        myServlet.addMapping("/custom/**");
    }
}
```

类似的，我们还可以创建新的`WebApplicationInitializer`实现来注册`Listener`和`Filter` ：

```java
public void onStartup(ServletContext servletContext) throws ServletException {
	ServletRegistration.Dynamic myServlet = servletContext.addServlet("myServlet",MyServlet.class);
	myServlet.addMapping("/custom/**");
	FilterRegistration.Dynamic myFilter = servletContext.addFilter("myFilter",MyFilter.class);
    myFilter.addMappingForUrlPatterns(null,false,"/custom/**");
}
```

不过，如果你只是注册`Filter`并将其映射到`DispatcherServlet`上的话，还有一种快捷方式，在`AbstractAnnotationConfigDispatcherServletInitializer`实现类中重载`getServletFilters()`方法：

```java
@Override
protected Filter[] getServletFilters() {
	return new Filter[]{MyFilter.class};
}
```

# 处理`multipart`形式的数据

## 配置`multipart`解析器

- 如果采用`Servlet`初始化类来配置`DispatcherServlet`的话，这个初始化类实现了`WebApplicationInitializer` ：

```java
public class MyServletInitializer implements WebApplicationInitializer{
    public void onStartup(ServletContext servletContext) throws ServletException {
        ServletRegistration.Dynamic myServlet = servletContext.addServlet("myServlet",MyServlet.class);
        myServlet.setMultipartConfig(
                new MultipartConfigElement("/tmp/spittr/uploads",
                        2*1024,4*1024,0));
        myServlet.addMapping("/custom/**");
    }
}
```

- 如果继承了`AbstractAnnotationConfigDispatcherServletInitializer` ：

```java
@Override
 protected void customizeRegistration(ServletRegistration.Dynamic registration) {
	registration.setMultipartConfig(
	new MultipartConfigElement("/tmp/spittr/uploads",
                        2*1024,4*1024,0));
}
```

## 处理`Multipart`请求

假设我们需要在注册的时候要求用户上传一张自己的头像 ：

```html
<body>
    <div class="ui text container">
        <form class="ui form" style="padding-top: 5%;" action="/spitter/register" method="post">
            <div class="field">
                <img class="ui centered small circular image" src="/resources/images/matthew.png">
            </div>
            <div class="field">
                <label>姓氏</label>
                <input type="text" id="姓氏" name="firstName" placeholder="姓氏">
            </div>
            <div class="field">
                <label>名称</label>
                <input type="text" name="lastName" placeholder="名称">
            </div>
            <div class="field">
                <label>用户名</label>
                <input type="text" name="userName" placeholder="用户名">
            </div>
            <div class="field">
                <label>密码</label>
                <input type="password" name="passWord" placeholder="密码" autocomplete="">
            </div>
            <div class="field">
                <label>图片上传</label>
                <input type="file" name="file" accept="image/jpeg,image/png,image/gif"/>
            </div>
            <div class="field">
                <div class="ui checkbox">
                    <input type="checkbox" tabindex="0" class="hidden">
                    <label>我同意本条款和条件</label>
                </div>
            </div>
            <div class="ui error message">

            </div>
            <div class="ui primary button" id="submit">提交</div>
        </form>
    </div>
</body>
```

效果如图所示 （PS 不得不说semantic确实很赞）：

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-8-27/77798882.jpg)

之后，我们打算还要加上点动态效果，例如，在图片上传之后，上面的logo也随之变动 ：

```javascript
$(".ui.form").on("change","input[name='imgLogo']",function () {
	var file = new FileReader();
	file.onload = function(element){
		$("#logo").attr("src",element.target.result);
	};
	file.readAsDataURL(this.files[0]);
});
```

之后，我们就可以处理这张图片了，第一步，我们需要在`AbstractAnnotationConfigDispatcherServletInitializer`实现类中配置`multipart`具体细节 ：

```java
@PropertySource(value = "/application.properties")
public class WebApplicationInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Autowired
    private Environment enviroment;
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{RootConfig.class};
    }
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {
        registration.setMultipartConfig(new MultipartConfigElement(
                "E:/upload",
                4*1024*1024,8*1024*1024,0
        ));
    }
}
```

第二步 ：让`controller`层接收到文件 ：

```java
package controller;

@Controller
@RequestMapping(value = "/spitter")
public class SpitterController {
    // ...
    @PostMapping(value = "/register")
    @ResponseBody
    public JsonUtils processRegisteration(
            @Valid Spitter spitter,
            @RequestPart(value = "imgLogo") MultipartFile image,HttpServletRequest request,
            Errors error) throws IOException {
        if (error.hasErrors()){
            return new JsonUtils(ResultEnum.ERROR,null,error.getAllErrors());
        }

        File file = new File(environment.getProperty("spitter.file"));
        if(!file.exists()){
            file.mkdirs();
        }

        String uuid = UUID.randomUUID().toString();

        String extention = image.getOriginalFilename().substring(image.getOriginalFilename().lastIndexOf("."));

        image.transferTo(new File(environment.getProperty("spitter.file") + "/" + uuid + extention));

        spitter.setImgLogo("/spitter/image/" + uuid);

        spitterRepository.save(spitter);

        return new JsonUtils(ResultEnum.SUCCESS,null,spitter.getUserName());
    }

    @PostMapping(value = "/user/{username}")
    @ResponseBody
    public JsonUtils spitterProfile(@PathVariable String username){
        return new JsonUtils(ResultEnum.SUCCESS,null,
                spitterRepository.findOneByUserName(username));
    }
}
```

第三步：改写`javascript`中的方法 :

```javascript
$(".ui.form").form({
        // ....
    }).api({
        action : "create user",
        method : "POST",
        processData: false,
        contentType: false,
        beforeSend : function(settings){
            settings.data = new FormData($(".ui.form")[0]);
            return settings;
        },
        onResponse : function(response) {
           // .....
        }
    });
```

之后，仿照用户登录，验证成功。

# 处理异常

## 乱码问题

在编写这部分代码过程中，乱码问题困扰了我很长时间，具体需要下面两个配置 ：

```java
public class WebApplicationInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{RootConfig.class};
    }

    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    @Override
    protected Filter[] getServletFilters() {
        return new Filter[]{new CharacterEncodingFilter("utf-8",true)};
    }

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {
        registration.setMultipartConfig(new MultipartConfigElement(
                "E:/upload",
                4*1024*1024,8*1024*1024,0
        ));
    }
}
```

```java
@Configuration
@EnableWebMvc
@ComponentScan(value = {"controller","errors"})
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public ViewResolver viewResolver(){
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/views/");
        resolver.setSuffix(".html");
        resolver.setExposeContextBeansAsAttributes(true);
        return resolver;
    }
	// 编码问题解决
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        StringHttpMessageConverter converter = new StringHttpMessageConverter(UTF_8);
        // 记得还要把json解析器加上...
        converters.add(new MappingJackson2HttpMessageConverter());
        converters.add(converter);
    }

    // 配置静态资源的处理
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/")
                .addResourceLocations("/resources/**");
    }
}
```

在代码运行的时候，我发现编译器idea又出来捣乱了，还需要设置下这个地方 ：

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-8-29/12454940.jpg)

## 编写异常处理的方法

```java
@Controller
public class ...{
	@ExceptionHandler(IncorrectResultSizeDataAccessException.class)
    public ResponseEntity<String> spitterNotFoundHandler(Exception exception){
        exception.printStackTrace();
        return new ResponseEntity<String>("抱歉.查无此人",HttpStatus.NOT_FOUND);
    }
    @ExceptionHandler(SQLIntegrityConstraintViolationException.class)
    public ResponseEntity<String> duplicateSpitterHandler(){
        return new ResponseEntity<String>("用户名已被占用.",HttpStatus.SERVICE_UNAVAILABLE);
    }
}    
```

# 为控制器添加通知

```java
@ControllerAdvice
public class AppWideExceptionHandler{
    
    @ExceptionHandler(...Exception.class)
    public String ....Handler(){
        
    }
    
}
```



