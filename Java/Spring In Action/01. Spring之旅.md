# 第一章  Spring之旅

[TOC]

## 前言

在诞生之初，创建spring的主要目的就是用来**替代更加重量级的企业级Java技术** 。

Spring的**两个核心特性**：

1. 依赖注入（DI）
2. 面向切面编程（AOP）

##  为降低java开发的复杂性，spring采取的四种策略

- 基于[POJO](http://www.jianshu.com/p/6f3e2bd50cb1)的**轻量级**和**最小侵入性编程**
- 通过[依赖注入](http://www.jianshu.com/p/9ea1a9bce19f)和面向接口**实现松耦合**
- 基于切面和惯例进行**声明式编程**
- 通过切面和模板**减少样板式代码**

## 耦合的两面性

>  BraveKnight.class

```java
public class BraveKnight implements Knight {

    //拯救少女请求
    private RescueDamselQuest quest;

    public BraveKnight(){
        this.quest = new RescueDamselQuest();
    }

    @Override
    public void embarkOnQuest() {
        quest.embark();
    }
}
```
`BraveKnight`和`RescueDamselQuest`紧密的耦合在一起。

耦合具有两面性：

1. 一方面，高耦合度代码**难以复用、难以测试、难以理解**，当修复一个bug时，将会出现一个或更多bug。
2. 另一方面，一定程度的耦合度是必须的，完全没有耦合的代码什么也做不了。
3. 为完成实际功能，不同类必须以适当的方式进行交互。总而言之，耦合是必须的，但应当被小心谨慎的管理。

✨ 通过依赖注入（DI），对象的依赖关系将由**系统中负责协调各对象的第三方组件**在创建对象的时候进行设定，对象无需自行创建或管理他们的依赖关系。如图，依赖关系将被自动注入到需要它们的对象当中去。

![依赖关系被自动注入](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/2019/08/20190808164520.png)

## "灵活"的勇敢骑士：构造器注入

BraveKnight.class
```java
public class BraveKnight implements Knight {

    private Quest quest;
	
    // 构造器注入
    public BraveKnight(Quest quest){
        this.quest = quest;
    }

    @Override
    public void embarkOnQuest() {
        quest.embark();
    }
    
}
```

`BraveKnight` 没有自行创建探险任务，**而是在构造的时候把`Quest`作为构造器参数传入**，这是依赖注入的方式之一：**构造器注入** 。

对依赖进行替换的一个最常用的方法就是在测试的时候使用`mock`实现：

```java
class BraveKnightTest {
    @Test
    void knightShouldEmbarkQuest() {
        // 创建一个由mock方法实现的Quest接口
        Quest mockQuest = Mockito.mock(Quest.class);
        // 注入 mock Quest
        BraveKnight braveKnight = new BraveKnight(mockQuest);
        braveKnight.embarkOnQuest();
        // 验证embark()是否仅被调用了一次
        Mockito.verify(mockQuest, new Times(1)).embark();
    }
    
}
```

下面写一个新的Quest实现：

> SlayDragonQuest.class

```java
public class SlayDragonQuest implements AbstractQuest {

    private PrintStream printStream;

    public SlayDragonQuest(PrintStream printStream) {
        this.printStream = printStream;
    }

    @Override
    public void embark() {
        printStream.println("勇士.请和恶龙争斗吧.");
    }

}
```

✨ 构建应用组件之间的协作的行为通常称为**装配**。Spring有多种装配bean的方式，采用XML是很常见的一种装配方式。

### 构造器注入：基于XML进行装配

> knights.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="knight" class="springinaction.knights.BraveKnight">
        <constructor-arg ref="quest"/>
    </bean>

    <bean id="quest" class="springinaction.quest.SlayDragonQuest">
        <constructor-arg value="#{T(System).out}"/>
    </bean>

</beans>
```

其中，`BraveKnight` . `SlayDragonQuest` 被声明为Spring中的bean。就`BraveKnight`来讲，它在构造时传入了对`SlayDragonQuest`的引用，将其作为构造器参数。同时，`SlayDragonQuest` bean 的声明使用了spring表达式语言，将System.out（这是一个PrintStream）传入到了`SlayDragonQuest`的构造器中。

### 对象创建和组装

✨Spring通过**应用上下文（Application Context）**装载bean的定义并把它们组装起来，Spring应用上下文全权负责对象的创建和组装，Spring自带了多种应用上下文的实现，它们之间主要的区别仅仅在于**如何加载配置**。

由于`knight.xml`中的bean是使用XML进行配置的，因此选择`ClassPathXmlApplicationContext`作为应用上下文比较合适。该类加载位于应用程序类路径下的一个或者多个XML配置文件。

> KnightMain.class

```java
public class KnightMain {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("knight.xml");
        Knight knight = (Knight) context.getBean("knight");
        knight.embarkOnQuest();
    }
}
```

`KnightMain.class`过程：

1. 基于`knights.xml`创建了Spring应用上下文
2. 调用应用上下文获取一个ID为`knight`的bean
3. 得到`Knight`引用后，简单调用`embarkOnQuest()`方法执行探险任务

### 构造器注入：基于Java文件

Spring还支持基于Java的配置：

> KnightConfig.class

```java
@Configuration
public class KnightConfig {

    @Bean
    public Quest quest() {
        return new SlayDragonQuest(System.out);
    }

    @Bean
    public Knight knight() {
        return new BraveKnight(quest());
    }
    
}
```
> KnightMain.class

```java
public class KnightMain {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext("springinaction/config");
        Knight knight = (Knight) context.getBean("knight");
        knight.embarkOnQuest();
    }
}
```

## 基于切面进行声明式编程

👏 DI（依赖注入）可以让**互相协作的软件保持松散耦合**，而面向切面编程（AOP）允许你**把遍布应用各处的功能分离出来形成可重用的组件**。

面向切面编程往往被定义为：促进软件系统实现关注点分离的一项技术。

**横切关注点**：系统由许多不同的组件组成，每一个组件负责一块特定功能。除了实现自身核心功能之外，这些组件还经常承担着额外的职责，诸如日志、事务管理和安全这样的系统服务经常融入到自身具有核心业务逻辑的组件中去，这些系统服务通常被称为横向关注点。因为它们会跨越系统的多个组件。

如果将这些关注点分散到多个组件中，代码将会带来双重复杂性：

- 实现系统关注点功能的代码将会分散到多个组件中去；即使把关注点抽象成独立的模块，其他模块仍然需要调用其方法。
- 组件会因为那些与自身核心业务无关的代码而变得混乱。例如一个向地址簿增加地址条目的方法不应该关注它是否是安全的或者是否支持事务。

![关注点的调用经常会散布到各个模块中](https://raw.githubusercontent.com/zhangzhaolin/spring-demo/master/%E5%9B%BE%E7%89%87/%E7%AC%AC%E4%B8%80%E7%AB%A0/%E5%85%B3%E6%B3%A8%E7%82%B9.png)

AOP能够使服务模块化（例如日志模块、安全模块、事务管理模块...），并以声明的方式将它们应用到它们需要影响的组件中去。所造成的结果就是这些组件**会具有更高的内聚性**并且**会更加关注自身的业务**。


## AOP应用

> Minstrel.class

```java
public class Minstrel {
    
    private PrintStream printStream;
    
    
    public Minstrel(PrintStream printStream){
        this.printStream = printStream;
    }
    /**
     *  探险之前调用
     */
    public void singBeforeQuest(){
        printStream.println("探险之前调用.");
    }
    /**
     *  探险之后调用
     */
    public void singAfterQuest(){
        printStream.println("探险之后调用.");
    }
}
```
如果在BraveKnight.class中调用Minstrel的方法的话：
```java
public void embarkOnQuest() {
        minstrel.singBeforeQuest();
        quest.embark();
        minstrel.singAfterQuest();
    }
```
显然有点不正确，`Minstrel` 吟游诗人应该做他分内的事情，而不需要`minstrel`的“提醒”。

我们需要做的是将`Minstrel` 声明为一个切面：

> knights.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="knight" class="springinaction.knights.BraveKnight">
        <constructor-arg ref="quest"/>
    </bean>

    <bean id="quest" class="springinaction.quest.SlayDragonQuest">
        <constructor-arg value="#{T(System).out}"/>
    </bean>

    <bean id="minstrel" class="springinaction.minstrel.Minstrel">
        <constructor-arg value="#{T(System).out}"/>
    </bean>

    <aop:config>
        <aop:aspect ref="minstrel">

            <!-- 定义切点 -->
            <aop:pointcut id="embark" expression="execution(* *.embarkOnQuest(..))"/>
            <!-- 前置通知 -->
            <aop:before method="singBeforeQuest" pointcut-ref="embark"/>
            <!-- 后置通知 -->
            <aop:after method="singAfterQuest" pointcut-ref="embark"/>

        </aop:aspect>
    </aop:config>

</beans>
```

声明一个简单切面的方法 :
- 首先，把 `Minstrel` 声明为一个bean
- 在 `<aop:config></aop:config>` 中编写切面
- 在 `<aop:aspect></aop:aspect>` 中引用名为 `minstrel` 的bean，使其声明为一个切面
- 使用 `<aop:pointcut/>` 定义切入点，并配置`expression`属性来选择所应用的通知
- 使用 `<aop:before/>` 定义 **前置通知**
- 使用 `<aop:after/>` 定义 **后置通知**


## 使用模板消除样板式代码

为了实现简单的功能或者任务，不得不重复写一些重复冗长的代码，比如查询数据库。这就是样板式代码 ：

```java
public class BookMain {
    public static void main(String[] args) {
        List<Book> books = new ArrayList<>();
        try (
                Connection connection =
                        DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/graduate?serverTimezone=Asia/Shanghai", "xxxxx", "xxxxx");
                PreparedStatement preparedStatement = connection.prepareStatement("SELECT * FROM book");
                ResultSet resultSet = preparedStatement.executeQuery();
        ) {
            while (resultSet.next()) {
                Book book = new Book();
                book.setId(resultSet.getLong("id"));
                book.setAuthor(resultSet.getString("author"));
                book.setDescription(resultSet.getString("description"));
                book.setIsbn(resultSet.getString("isbn"));
                book.setTitle(resultSet.getString("title"));
                books.add(book);
            }
        } catch (SQLException e) {
        }
        books.forEach(System.out::println);
    }
}
```

如果说使用了Spring的`JdbcTemplate`的话，事情会变得比较简单 ： 

```java
public Book getBookById(Long id) {
    String sql = "SELECT * FROM BOOK WHERE ID = ?";
    Book book = new Book();
    RowMapper<Book> mapper = (rs, rowNum) -> {
        book.setAuthor(rs.getString("author"));
        book.setDescription(rs.getString("description"));
        book.setId(rs.getLong("id"));
        book.setIsbn(rs.getString("isbn"));
        book.setTitle(rs.getString("title"));
        return book;
    };
    return jdbcTemplate.queryForObject(sql, mapper, id);
}
```

spring通过**面向 POJO 编程、DI、切面和模板技术**来简化 Java 开发中的复杂性。


## 容纳你的Bean：Spring容器

✨ 在基于Spring的应用中，你的应用对象生存于Spring容器（container）中，Spring容器负责**创建对象，装配它们，配置它们，并管理它们的整个生命周期，从生存到死亡**。

**容器是Spring框架的核心**，Spring容器使用DI管理构成应用的组件，它会创建相互协作的组件之间的关联。

**Spring的容器并不是一个，Spring自带了多个容器实现**，可以归为两个不同的类型：**bean工厂和应用上下文**。

bean工厂是最简单的容器，提供基本的依赖注入支持。应用上下文基于`BeanFactory`构建，并提供应用框架级别的服务，例如从属性文件解析文本以及发布应用事件给感兴趣的事件监听者。

![BeanFactory和ApplicationContext](https://raw.githubusercontent.com/zhangzhaolin/spring-demo/master/%E5%9B%BE%E7%89%87/%E7%AC%AC%E4%B8%80%E7%AB%A0/BeanFactory%E5%92%8CApplicationContext.jpg)

应用上下文使用的要更多更频繁一些。

## 使用应用上下文

- `AnnotationConfigApplicationContext` ： 从一个或多个**基于Java的配置类**中加载Spring应用上下文。
- `AnnotationConfigWebApplicationContext`：从一个或多个**基于Java配置类**中加载Spring web应用上下文。
- `ClassPathXmlApplicationContext`：从classpath路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类资源。
- `FileSystemXmlApplicationContext`：从文件系统下的一个或多个XML配置文件中加载上下文定义。
- `XmlWebApplicationContext`：从WEB应用下的一个或多个XML配置文件中加载上下文定义。

无论是从类中还是从文件系统中加载应用上下文都是类似的：
例如，下面是从类路径中加载：
```java
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("/knights.xml");
```
下面是从文件系统中加载
```java
FileSystemXmlApplicationContext context = new FileSystemXmlApplicationContext("c:/knights.xml");
```
下面是Java配置文件加载：
```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(KnightConfig.class);
```

## bean 的声明周期

![bean的声明周期](https://raw.githubusercontent.com/zhangzhaolin/spring-demo/master/%E5%9B%BE%E7%89%87/%E7%AC%AC%E4%B8%80%E7%AB%A0/bean%E7%9A%84%E5%A3%B0%E6%98%8E%E5%91%A8%E6%9C%9F.jpg)

bean的声明周期经常会被问到，但是却又很难记忆 （建议把接下来的打印下来每天记忆）：

>作者：MOBIN-F
>链接：https://www.zhihu.com/question/38597960/answer/77600561
>来源：知乎
>著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

>- Spring对Bean进行实例化（相当于程序中的newXx()）
>- Spring将值和Bean的引用注入进Bean对应的属性中
>- 如果Bean实现了`BeanNameAware`接口，Spring将Bean的ID传递给`setBeanName()`方法（实现`BeanNameAware`接口的主要是为了通过Bean的引用来获得Bean的ID，一般业务中是很少有用到Bean的ID的）
>- 如果Bean实现了`BeanFactoryAware`接口，Spring将调用`setBeanDactory(BeanFactory bf)`方法并把`BeanFactory`容器实例作为参数传入。（**实现`BeanFactoryAware`主要目的是为了获取Spring容器，如Bean通过Spring容器发布事件等）**
>- 如果Bean实现了`ApplicationContextAware`接口，Spring容器将调用`setApplicationContext(ApplicationContext ctx)`方法，把应用上下文作为参数传入。**(作用与BeanFactory类似都是为了获取Spring容器，不同的是Spring容器在调用`setApplicationContext()`方法时会把它自己作为`setApplicationContext`的参数传入，而Spring容器在调用`setBeanDactory`前需要程序员自己指定（注入）`setBeanFactory`里的参数`BeanFactory`)**
>- 如果Bean实现了`BeanPostProcessor`接口，Spring将调用它们的`postProcessBeforeInitialization`（预初始化）方法 **(作用是在Bean实例创建成功后对进行增强处理，如对Bean进行修改，增加某个功能)**
>- 如果Bean实现了`InitializingBean`接口，Spring将调用它们的`afterPropertiesSet`方法，作用与在配置文件中对Bean使用`init-method`声明初始化的作用一样，都是在Bean的全部属性设置成功后执行的初始化方法。
>- 如果Bean实现了`BeanPostProcess`接口，Spring将调用它们的`postProcessAfterInitialization`（后初始化）方法（**作用与6的一样，只不过6是在Bean初始化前执行的，而这个是在Bean初始化后执行的，时机不同 )**
>- 经过以上的工作后，Bean将一直驻留在应用上下文中给应用使用，直到应用上下文被销毁
>
>- 如果Bean实现了`DisposableBean`接口，Spring将调用它的`destory`方法，作用与在配置文件中对Bean使用destory-method属性的作用一样，都是在Bean实例销毁前执行的方法。

## Spring模块

![Spring模块](https://raw.githubusercontent.com/zhangzhaolin/spring-demo/823096970bcc9b0ca95a6d862661525911d4ff44/%E5%9B%BE%E7%89%87/%E7%AC%AC%E4%B8%80%E7%AB%A0/spring%E6%A8%A1%E5%9D%97.png)
