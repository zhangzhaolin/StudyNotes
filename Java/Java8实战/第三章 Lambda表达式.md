# 第三章 Lambda表达式

## 管中窥豹

可以把Lambda表达式理解为简洁地表示可传递的匿名函数的一种方式：**它没有名字，但它有：参数列表、函数主题、返回类型，可能还有一个可以抛出的异常列表。**

例如，可以利用Lambda表达式，可以更为简洁的创建出`Comparator`对象 ：

![](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E6%88%AA%E5%9B%BE/Java8%E5%AE%9E%E6%88%98/Character3/1.png)

在JDK8出现之前和之后代码看起来更加清晰 ：

![](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E6%88%AA%E5%9B%BE/Java8%E5%AE%9E%E6%88%98/Character3/2.png)

Lambda表达式有三个部分 ：

- 参数列表 —— 在刚刚的例子中，它采用了` Comparator`类中的`compare()`方法的参数（两个`Apple`类）
- 箭头 —— 箭头`->`把参数列表和Lambda主体分隔开
- Lambda主体 —— 在刚刚的例子中，比较两个`Apple`的重量。表达式就是Lambda的**返回值**了

下面列出了五个有效的Lambda表达式的例子 ：

```java
// ①
(String s) -> s.length();
// 第一个Lambda表达式具有一个String类型的参数并返回一个int。Lambda语句已经隐含了return
// ②
(Apple a) -> a.getWeight() > 150;
// 第二个Lambda表达式有一个Apple类型的参数并返回一个boolean
// ③
(int x,int y) -> {
  	System.out.println("Result : ");
    System.out.pringln(x + y);
};
// 第三个Lambda表达式有两个int类型的参数而没有返回值。Lambda表达式可以包含多条语句
// ④
() -> 42;
// 第四个Lambda表达式没有参数，但是返回一个int类型
// ⑤
(Apple a1,Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
// 第五个Lambda表达式有两个Apple类型的参数，返回一个int类型 ：比较两个苹果的重量
```

Lambda的基本语法是 ：

```java
(parameters) -> expression
```

或者 ：

```java
(parameters) -> {statements;}
```

```java
(1) ()->{}
(2) ()->"Raoul"
(3) ()->{return "Mario";}
(4) (Integer i)-> return "Alan" + i
(5) (String s)->{"IronMan";}
```

以上代码中，只有(4)(5)是错误的Lambda

(4) `return`是一个流程控制语句，要使此Lambda有效，需要使用花括号 ：

```java
(Integer i) -> {return "Alan" + i;}
```

(5) `"IronMan"`是一个表达式，而不是一个语句。要使此Lambda有效，你可以去掉花括号和分号 ：

```java
(String s) -> "IronMan"
```

或者可以使用显示语句 ：

```java
(String s) -> {return "IronMan";}
```

## 在哪里以及如何使用Lambda

### 函数式接口

**函数式接口**就是只定义一个抽象方法的接口。

```java
public interface Comparator<T>{
    int compare(T o1,T o2);
}
// ......
public interface Runnable{
    void run();
}
```

Lambda表达式允许你直接以内联的形式为函数式接口的抽象方法提供实现，并把整个表达式作为函数接口的实例（具体地说，是函数式接口的一个具体实现的实例）。你也可以使用匿名类来完成同样地事情 ：

```java
Runnable r1 = () -> System.out.println("Hello World 1");

Runnable r2 = new Runnable(){
    @Override
    public void run(){
        System.out.println("Hello World 2");
    }
};

public static void process(Runnable r){
    r.run();
}

process(r1);
process(r2);
process(()->System.out.println("Hello World 3"));
```

### 函数描述符

函数式接口中抽象方法描述了Lambda表达式的签名，我们将抽象方法所描述的Lambda形式称为**函数描述符**。

Lambda表达式可以被赋值给一个变量，或传递给一个接受函数式接口作为参数的方法，当然，这个Lambda表达式签名要和函数式接口的抽象方法一样。比如 ：

```java
() -> System.out.println("hello world");
```

不接受参数并且返回`void`，这恰恰就是`Runnable`接口中`run`方法的签名。

下面的例子是错误的 ：

```java
Predicate<Apple> p = (Apple apple) -> apple.getWeight();
```

因为`Predicate`源码如下所示 ：

```java
@FunctionalInterface
public interface Predicate{
    boolean test();
}
```

函数签名应该为 ：

```java
(Apple) -> boolean // 返回一个boolean类型的对象 而不是一个Integer
```

## 环绕执行模式

**环绕执行模式**：资源处理（例如处理文件或者数据库）时常见的一个模式就是打开一个资源、做一些处理、然后关闭资源。这个设置和清理阶段总是很类似，并且会围绕着执行处理的那些重要的代码，这就是所谓的*环绕执行*模式

