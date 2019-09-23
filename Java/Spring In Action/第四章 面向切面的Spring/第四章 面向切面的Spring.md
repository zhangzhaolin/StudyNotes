# 第四章 面向切面的Spring

[TOC]

## 面向切面要解决的问题

在软件开发中，散布于应用中多处的功能被称为 **横切关注点**，例如事务、安全、日志、权限控制………通常来讲，这些**横切关注点**从概念上是与业务的应用逻辑相分离的。把这些横切关注点与业务逻辑相分离正是面向切面编程所要解决的问题。

依赖注入（DI）有助于应用对象之间的解耦，而AOP可以实现横切关注点与它们所影响的对象之间的解耦。

横切关注点可以被模块化为特殊的类，这些类被称为**切面**

## 面向切面常用术语

描述切面的常用术语有 ：通知、切点和连接点 。

![通知、切点和连接点](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-8-8/23137316.jpg)

### 通知

在AOP术语中，切面（横切关注点可以被模块化为特殊的类）的工作被称为通知。通知定义了 **切面是什么以及何时使用**，除了描述切面所完成的工作，通知还解决了**何时执行这个工作的问题**。

Spring切面可以应用五种类型的通知 :

- 前置通知 ：在目标方法被 **调用之前** 调用通知功能 
- 后置通知 ：在目标方法 **完成之后** 调用通知，不会关心方法的输出是什么
- 返回通知 ：在目标方法 **成功执行** 之后调用通知
- 异常通知 ：在目标方法 **抛出异常** 后调用通知
- 环绕通知 ：通知包裹了被通知的方法，在目标方法 **调用之前和调用之后** 执行自定义的行为

### 连接点

连接点就是在 **应用执行过程中** 能够 **插入切面** 的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。

### 切点

一个切面不需要通知应用的所有连接点。切点有助于**缩小切面所通知的连接点**。如果说通知定义了切面是什么以及切面在何时使用的话，那么切点就定义了“何处”。

切点的定义会匹配通知所要 **织入** 的一个或者多个连接点。通常使用一个明确的类和方法名称，或者利用正则表达式定义所匹配的类和方法名称来指定这些切点。

### 切面

切面是通知和切点的结合。通知和切点共同定义了切面的全部内容 —— 它是什么，在何时和何处完成其功能。

### 引入

引入允许我们向现有类添加新方法或者属性。在无需修改现有的类的情况下，让他们具有新的行为和状态。

### 织入

织入是**把切面应用到目标对象并创建新的代理的过程**。切面在指定的连接点被织入到目标对象中。在目标对象的生命周期里有多个点可以进行织入 ：

- 编译期 ： 切面在目标类 **编译** 时被织入。这种方式需要有特殊的编译器。例如AspectJ编译器。
- 类加载期 ：切面在目标类 **加载到JVM** 时被织入。这种方式需要有特殊的类加载器（`ClassLoader`）。这类类加载器可以在目标类引入应用之前增强该目标类的字节码。例如AspectJ 5的加载时织入（`LTW`）。
- 运行期 ：切面在 **应用运行的某个某个时刻** 被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。例如Spring AOP。

### 总结

通知包含了需要用于多个应用对象的横切行为（横切关注点）；连接点是程序执行过程中能够应用通知的所有点；切点定义了通知被应用的具体位置（在哪些连接点会得到通知）。

## Spring对AOP的支持

Spring提供了四种类型的AOP支持 ：

- [x] <s>基于代理的经典Spring AOP</s>
- [x] 纯POJO切面（借助`aop`命名空间可以将POJO转换为切面，需要XML配置）
- [x] `@AspectJ`注解驱动的切面（使用注解来完成功能）
- [x] 注入式`AspectJ`切面（如果AOP需求超过了简单的方法调用，如构造器或者属性拦截，那么你需要考虑使用`AspectJ`来实现切面，适用于Spring各版本）

通过在代理类中包裹切面，Spring在运行期把切面织入（通过指定的 **连接点** ）到Spring管理的bean（也就是 **目标对象** ）中。如图，代理类封装了目标类，并拦截被通知方法的调用，再把调用转发给真正的目标bean。当代理拦截到方法调用时，**在调用目标bean方法之前，会执行切面逻辑**。

- 在代理类中包裹切面
- 在运行期间把切面织入到Spring管理的Bean（目标对象）
- 代理类封装目标类，拦截被通知方法的调用
- 在调用目标对象Bean方法之前，执行切面逻辑
- 把调用转发给真正的目标Bean

![SPring的切面由包裹了目标对象的代理类实现。代理类处理方法的调用，执行额外的切面逻辑，并调用目标方法](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-8-12/38529126.jpg)

直到应用需要被代理的bean时，Spring才会创建代理对象。

因为Spring基于动态代理，所以Spring只支持方法连接点。Spring缺少对字段连接点的支持，无法让创建细粒度的通知，例如拦截对象字段的修改。而且它不支持构造器连接点，无法在bean创建时应用通知。

如果需要方法拦截点之外的拦截点拦截功能，我们可以利用AspectJ来补充Spring AOP的功能。

## 通过切点来选择连接点

### 使用AspectJ的切点表达式语言

在Spring AOP中，使用`AspectJ`的切点表达式语言定义切点，其中`execution`是最重要的描述符 ：

```java
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)
            throws-pattern?)
```

除了返回类型（`ret-type-pattern`）、方法名称（`name-pattern`）以及参数列表（`param-pattern`）之外，其余都是可选的（即含有`?`的都是可选的）。

- `modifiers-pattern?` ：方法修饰符（public、private、默认、protected）
- `ret-type-pattern` ：方法返回类型
- `declaring-type-pattern?` ：方法类所在路径
- `name-pattern` ：方法名
- `param-pattern` ：方法参数类型，可有一个或者多个，用`(..)`表示零个或者任意个参数，多个参数还可以用`,`分隔，也可以用`(*)`表示匹配任意类型的参数
- `throws-pattern?` ：表示方法抛出的异常

> `ret-type-pattern`用来确定方法的返回类型，以匹配连接点。`*`最常用作方法返回类型模式。它匹配任何返回类型。在方法返回给定类型时，全限定类名才会匹配。`name-pattern`匹配方法名称。你可以使用`*`通配符作为`name-pattern`的全部或者部分。如果您指定了`declaring-type-pattern`（方法类所在路径），请在后面加上`.`来链接方法名称。`param-pattern`（方法参数类型）稍微复杂一些：`()`匹配不带参数的方法，`(..)`匹配任意数量（零个或者多个）的参数。`(*)`匹配采用任意类型的一个参数方法。`(*,String)`匹配一个带有两个参数的方法，第一个可以是任何类型，第二个必须是`String`类型。
>
> <span style="text-align:right">翻译自：https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-schema</span>

例如 ：

匹配所有的public方法 ：

```java
execution(public * *(..))
```

匹配所有方法名开头为`set`的 ：

```java
execution(* set*(...))
```

匹配定义在`AccountService`接口类中的所有方法

```java
execution(* com.xyz.service.AccountService.*(...))
```

匹配定义在`service`包下的所有方法

```java
execution(* com.xyz.service.*.*(..))
```

匹配定义在`service`包或者子包下的所有方法

```java
execution(* com.xyz.service..*.*(..))
```

在`service`包下面的所有连接点（在Spring AOP仅仅执行方法）

```
within(com.xyz.service.*)
```

在`service`包和子包下的所有连接点（在Spring AOP仅仅执行方法）

```
within(com.xyz.service..*)
```

实现`AccountService`接口的代理中的所有连接点

```
this(com.xyz.service.AccountService)
```

| Aspect指示器  |                     描述                     |
| :-----------: | :------------------------------------------: |
| `execution()` |                用于匹配连接点                |
|    `arg()`    |          表明连接点参数类型是匹配类          |
|   `@args()`   |             表明参数注解是匹配类             |
|   `this()`    |  匹配一个bean，这个bean是一个指定类型的实例  |
|   `target`    | 匹配一个目标对象，此对象是一个给定类型的实例 |
|  `@target()`  |        匹配对象类需要有指定类型的注解        |
|  `within()`   |           限制连接点匹配指定的类型           |
|  `@within()`  |     匹配方法，该方法需要给定一个特定注解     |
| `@annotation` |           匹配带有指定注解的连接点           |

### 编写切点

假设我们需要编写`Performance`类型 ：

```java
public interface Performance{
    public void perform();
}
```

我们想要编写一个切面，在调用`Performance`类中的`perform`方法时触发通知 ：

```java
execution(* concert.Performance.perform(..))
```



![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-8-14/90244174.jpg)

现在我们假设我们需要配置的切点仅仅匹配`concert`包 ：

```java
execution(public * concert.Performance.perform(..) && within(concert.*))
```

当使用Spring的XML来描述切面时候，我们可以使用`and`来替换`&&`，同样的，`or`和`not`可以替换`||`和`!`

### 在切点中选择bean

Spring中的`bean()`指示器允许我们在切点表达式中使用bean的ID来标识bean。`bean()`使用`bean ID`或`bean`名称作为参数来限制切点只匹配特定的`bean`

例如 ：

```java
execution(* concert.Performance.perform(..) and bean('woodstock'))
```

在上面的例子中，我们希望在执行`perform()`方法时应用通知，但是限制bean的ID为`woodstock`

我们还可以使用非操作作为除了特定ID以外的其他bean应用通知 ：

```java
execution(* concert.Performance.perform(..) and !bean('woodstock'))
```

## 使用注解创建切面

### 定义切面

```java
@Aspect
public class Audience {
	// 演出之前
    @Before("execution(public * concert.Performance.perform(..))")
    public void silenceCellPhone(){
        System.out.println("手机静音");
    }
	// 演出之前
    @Before("execution(public * concert.Performance.perform(..))")
    public void takeSeats(){
        System.out.println("对号入座");
    }
	// 演出成功之后
    @AfterReturning("execution(public * concert.Performance.perform(..)")
    public void applause(){
        System.out.println("掌声雷动");
    }
	// 演出失败之后
    @AfterThrowing("execution(public * concert.Performance.perform(..))")
    public void demandRefund(){
        System.out.println("我想退款!");
    }

}
```

`Audience`类使用了`@Aspect`注解进行标注，表明该类不仅是一个POJO，**还是一个切面**。`Audience`类中的方法都是用注解来定义切面的具体行为。

AspectJ使用了五个注解来定义通知 ：

|       注解        |                   通知                   |
| :---------------: | :--------------------------------------: |
|     `@After`      | 通知方法在目标方法返回或者抛出异常时调用 |
| `@AfterReturning` |       通知方法在目标方法返回后调用       |
| `@AfterThrowing`  |     通知方法在目标方法抛出异常后调用     |
|     `@Around`     |       通知方法会将目标方法封装起来       |
|     `@Before`     |      通知方法在目标方法调用之前调用      |

在上面的例子中，我们定义了四个切点表达式，这四个表达式完全可以进行整合 ： 

**`PointCut`注解能够在一个`@Aspect`切面内定义可以重复的切点**

```java
@Aspect
public class Audience {
    @Pointcut("execution(public * concert.Performance.perform(..))")
    public void perform(){}
    // 演出之前
    @Before("perform()")
    public void silenceCellPhone(){
        System.out.println("手机静音");
    }
    // 演出之前
    @Before("perform()")
    public void takeSeats(){
        System.out.println("对号入座");
    }
    // 演出成功之后
    @AfterReturning("perform()")
    public void applause(){
        System.out.println("掌声雷动");
    }
    // 演出失败之后
    @AfterThrowing("perform()")
    public void demandRefund(){
        System.out.println("我想退款!");
    }
}

```

我们还需要启动AspectJ的自动代理 ：

如果你使用`JavaConfig`注解的话，你可以在配置类上加上`@EnableAspectJAutoProxy`注解启动自动代理的功能。

```java
// 启动AspectJ自动代理
@EnableAspectJAutoProxy
@Configuration
public class ConcertConfig {
    @Bean
    public Audience audience(){
        return new Audience();
    }
}
```

如果使用XML装配的话，我们需要`<aop:aspectj-autoproxy />`启动自动代理 ：

```java
<bean id="audience" class="concert.Audience"/>
<bean id="musicPerformance" class="concert.MusicPerformance"/>
<!-- 开启自动代理 -->
<aop:aspectj-autoproxy/>
```

Spring的AspectJ自动代理仅仅使用`@Aspect`作为创建切面的指导，**切面依然是基于代理的**。在本质上，它依然是**Spring基于代理的切面**。这一点非常重要，因为这意味着尽管使用的是`@Aspect`注解，但是仍然**限于代理方法的调用**。如果想使用AspectJ的所有能力，我们必须在运行时使用AspectJ并且不依赖Spring来创建基于代理的切面。

### 创建环绕通知

环绕通知能够让你所编写的逻辑将被通知的目标方法（连接点）完全包裹起来。就像是在一个通知方法中同时编写前置后置通知。

```java
@Aspect
public class Audience {

    @Pointcut("execution(public * concert.Performance.perform(..))")
    public void perform(){}

    // 环绕通知
    @Around("perform()")
    public void watchPerformance(ProceedingJoinPoint joinPoint){
        try{
            System.out.println("关闭手机");
            System.out.println("入座");
            // 通过ProceedingJoinPoint来调用被通知的方法
            joinPoint.proceed();
            System.out.println("掌声雷动");
        }catch(Throwable e){
            System.out.println("我要退款");
        }
    }
}
```

`@Around`注解表明`watchPerformance()`方法会作为`performance`切点的环绕通知。**当通知方法需要把控制权交给被通知方法时候，需要调用`ProceedingJoinPoint`的`proceed()`方法**。如果不调用这个方法的话，你的通知会阻塞对被通知方法的调用。

 ### 为通知传递参数

在`BlankDisc`中，我们需要统计磁道被播放的数量 ：**（请注意，Spring AOP无法拦截内部方法的调用）**

```java
public class BlankDisc implements CompactDisc {

    private String title;
    private String artist;
    private List<String> tracks;
    public BlankDisc(String title,String artist,List<String> tracks) {
        this.title = title;
        this.artist = artist;
        this.tracks = tracks;
    }
    public String getTitle() {
        return title;
    }
    public String getArtist() {
        return artist;
    }
    public List<String> getTracks() {
        return tracks;
    }
    public void play() {
        for (int trackNumber = 0;trackNumber < tracks.size();trackNumber ++){
            // 请注意，Spring AOP无法拦截内部方法的调用
            // ! playTrack(trackNumber);
            ((CompactDisc) (AopContext.currentProxy())).playTrack(trackNumber);
        }
    }
    public void playTrack(int trackNumber) {
        System.out.println("track "+ trackNumber + " : " + tracks.get(trackNumber));
    }
}
```

我们定义`TrackCounter`来描述切面 ：

```java
@Aspect
@Component
public class TrackCounter {

    public Map<Integer, Integer> trackCounts = new HashMap<>();

    @Pointcut("execution(* soundsystem.BlankDisc.playTrack(int))  && args(trackNumber)")
    public void trackedPlay(int trackNumber) {
    }

    @AfterReturning(value = "trackedPlay(trackNumber)", argNames = "trackNumber")
    public void countTrack(int trackNumber) {
        trackCounts.compute(trackNumber, (k, v) -> v == null ? 1 : v + 1);
        System.out.println(trackCounts);
    }

    public int getPlayCount(int trackNumber) {
        return trackCounts.getOrDefault(trackNumber, 0);
    }
}
```

切点表达式中的`args(trackNumber)`表明 ：**传递给连接点的`int`类型的参数也会传递到通知方法中**。参数的名称为`trackNumber`，与切点方法签名中的参数相匹配。在`@AfterReturing("trackNumber")`表达式下面，切点方法和切点定义的参数名一致。

### 通过注解引入新功能

TODO

### 缺陷

面向注解的切面声明有一个明显的劣势：你必须要为通知类添加注解，为了做到这一点，必须要有源代码。

## 在XML中声明切面

把bean声明为一个切面时，必须从`<aop:config>`元素开始配置。在`<aop:config>`中，我们可以声明一或多个通知器、切面或者切点。在下面的例子中，使用了`<aop:aspect>`声明了一个简单的切面。

### 前置后置通知

```xml
<bean id="audience" class="concert.Audience"/>

<!-- 切面配置 -->
<!-- 顶层的aop配置元素 -->    
<aop:config>
        <!-- 定制一个切面 -->
        <aop:aspect ref="audience">
			<!-- 定义一个切点 -->
            <aop:pointcut id="perform" expression="execution(public * concert.Performance.perform(..))"/>

            <aop:before method="takeSeats" pointcut-ref="perform"/>

            <aop:before method="silenceCellPhone" pointcut-ref="perform"/>

            <aop:after-returning method="applause" pointcut-ref="perform"/>

            <aop:after-throwing method="demandRefund" pointcut-ref="perform"/>

        </aop:aspect>
</aop:config>
```

### 环绕通知

```xml
<bean id="musicPerformance" class="concert.MusicPerformance"/>
<bean id="audience" class="concert.Audience"/>
<aop:config>
	<aop:aspect ref="audience">
 		<aop:pointcut id="performance" expression="execution(public * concert.Performance.perform())"/>
 		<aop:around method="execute" pointcut-ref="performance"/>
    </aop:aspect>
</aop:config>
```

### 为通知传递参数

```xml
<beans>	
	<bean id="blankDisc" class="soundsystem.BlankDisc"
          c:_0="${disc.title}" c:_1="${disc.artist}" c:_2-ref="blankDiscList"/>

    <bean id="cdPlayer" class="soundsystem.CDPlayer"/>

    <bean id="trackCounter" class="soundsystem.TrackCounter"/>

    <util:list id="blankDiscList">
        <value>老古董</value>
        <value>大千世界</value>
        <value>如约而至</value>
        <value>柳成荫</value>
    </util:list>

    <context:property-placeholder location="classpath:/application.properties"/>

    <aop:aspectj-autoproxy/>

    <import resource="classpath:/aopconfig.xml"/>
</beans>
```

```xml
<aop:config>
    <aop:aspect ref="trackCounter">
        <aop:after-returning method="countTrack" arg-names="trackNumber"
                             pointcut="execution(* soundsystem.CompactDisc.playTrack(int)) and args(trackNumber)"/>
    </aop:aspect>
</aop:config>
```

### 通过切面引入新的功能

TODO

## 注入AspectJ切面

TODO

## 总结

![](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E7%BB%98%E5%9B%BE/Spring%E5%AE%9E%E6%88%98/%E9%9D%A2%E5%90%91%E5%88%87%E9%9D%A2%E7%9A%84Spring.png)



