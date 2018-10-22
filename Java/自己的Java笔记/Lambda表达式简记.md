# Lambda表达式

## 函数式接口

函数式接口就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口 。例如 ：

```java
@FunctionalInterface
public interface MyLambdaInterface {
    void doSomethingShit(String s);
}
```

`@FunctionalInterface`主要用于 **编译级错误检查**，加上该注解之后，当你写的接口不符合函数式接口时，编译器会报错 。

## 什么是`Lambda`表达式

如果想把“一块代码”赋值给一个Java变量 ：

![Lambda表达式1](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-10-22/28591788.jpg)

在Java8之前，是无法做到的，但是利用lambda表达式，就可以做到了。

![lambda表达式2](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-10-22/22937999.jpg)

这样，我们就非常“优雅”的把“一块代码”赋值给了一个变量 。而这块代码，或者说，“这个被赋值给一个变量的函数”，就是一个Lambda表达式 。

那么变量`aBlockOfCode`的类型应该是什么 ？

在Java8当中，**所有的Lambda类型都是一个接口，而Lambda本身（也就是那一段函数），需要是这个接口的实现** 。简而言之，**Lambda本身就是一个接口的实现**

![Lambda表达式3](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-10-22/81019055.jpg)

这样，我们就得到了一个完整的Lambda表达式声明 ：

```java
MyLambdaInterface lambda = (s) -> System.out.println(s);
```

## Lambda表达式的作用

最直观的作用是使得代码更加的简洁 ：

![Lambda使得代码更加简洁](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-10-22/656831.jpg)

由于Lambda可以直接赋值给一个变量，我们就可以直接把Lambda作为参数给函数 ：

```java

```

