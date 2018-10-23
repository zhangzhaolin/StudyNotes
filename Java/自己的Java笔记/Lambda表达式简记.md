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

函数式接口里允许使用默认方法 ：

```java
@FunctionInterface
public interface MyLambdaInterface{
    
    void doSomething(String s);
    
    // ! void fxxkError(...);
    
    default void defaultThing(...){
        // ...
    }
}
```

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

<<<<<<< HEAD
![Lambda表达式作为参数](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E6%88%AA%E5%9B%BE/%E8%87%AA%E5%B7%B1%E7%9A%84Java%E7%AC%94%E8%AE%B0/Lambda%E8%A1%A8%E8%BE%BE%E5%BC%8F%E4%BD%9C%E4%B8%BA%E5%87%BD%E6%95%B0%E5%8F%82%E6%95%B0.png)
=======
![Lambda表达式作为参数](https://github.com/zhangzhaolin/StudyNotes/blob/master/%E6%88%AA%E5%9B%BE/%E8%87%AA%E5%B7%B1%E7%9A%84Java%E7%AC%94%E8%AE%B0/Lambda%E8%A1%A8%E8%BE%BE%E5%BC%8F%E4%BD%9C%E4%B8%BA%E5%87%BD%E6%95%B0%E5%8F%82%E6%95%B0.png)
>>>>>>> bc94fa843a12e0640cc79bd33b8aa201fbf6f3af

## 常见的函数式接口

### `Predicate`

<<<<<<< HEAD
```java
Predicate<String> predicate = s -> s.length() > 0;
System.out.println(predicate.test("Hello World"));// true
System.out.println(predicate.negate().test("Hello World"));// false
Predicate<Boolean> notNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;
System.out.println(notNull.test(null));// false
System.out.println(isNull.test(null));// true
Predicate<String> isEmpty = String::isEmpty;
Predicate<String> notEmpty = isEmpty.negate();
System.out.println(isEmpty.test("")); // true
System.out.println(notEmpty.test("")); // false
```

### `Function`

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    // ...
}
```

```java
Function<Integer,Integer> toInteger = e -> {return e+5;};
toInteger = toInteger.andThen(e->{return e*10;});
System.out.println(toInteger.apply(5)); // 100
```

### `Consumer`

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

```java
Consumer<String> shit = (s -> System.out.println("String length = " + s.length()));
shit.accept("test"); // String length = 4
```

与`Function`的区别就是 ，`Consumer`无返回值，`Function`有返回值 。

## 与函数式接口结合

直接看一个实例，假设`Person`的定义和`List<Person>`的值都给定 ：

```java
@AllArgsConstructor
public class Person {
    @Getter
    private String firstName;
    @Getter
    private String lastName;
    @Getter
    private int age;
}
```

```java
List<Person> persons = Arrays.asList(
        new Person("Yixing","Zhao",25),
        new Person("Yanggui","Li",30),
        new Person("Chao","Ma",29)
);
```

=======
```java
Predicate<String> predicate = s -> s.length() > 0;
System.out.println(predicate.test("Hello World"));// true
System.out.println(predicate.negate().test("Hello World"));// false
Predicate<Boolean> notNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;
System.out.println(notNull.test(null));// false
System.out.println(isNull.test(null));// true
Predicate<String> isEmpty = String::isEmpty;
Predicate<String> notEmpty = isEmpty.negate();
System.out.println(isEmpty.test("")); // true
System.out.println(notEmpty.test("")); // false
```

### `Function`

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    // ...
}
```

```java
Function<Integer,Integer> toInteger = e -> {return e+5;};
toInteger = toInteger.andThen(e->{return e*10;});
System.out.println(toInteger.apply(5)); // 100
```

### `Consumer`

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

```

```java
Consumer<String> shit = (s -> System.out.println("String length = " + s.length()));
shit.accept("test"); // String length = 4
```

与`Function`的区别就是 ，`Consumer`无返回值，`Function`有返回值 。

## 与函数式接口结合

直接看一个实例，假设`Person`的定义和`List<Person>`的值都给定 ：

```java
@AllArgsConstructor
public class Person {
    @Getter
    private String firstName;
    @Getter
    private String lastName;
    @Getter
    private int age;
}
```

```java
List<Person> persons = Arrays.asList(
        new Person("Yixing","Zhao",25),
        new Person("Yanggui","Li",30),
        new Person("Chao","Ma",29)
);
```

>>>>>>> bc94fa843a12e0640cc79bd33b8aa201fbf6f3af
有这样一个需求，需要打印出所有`lastName`以`Z`开头的人的`firstName`。

**原生态Lambda写法**：

定义两个函数式接口，定义一个静态函数 ：

```java
@FunctionalInterface
public interface NameChecker {
    boolean check(Person p);
}
// ...
@FunctionalInterface
public interface Executor {
    void execute(Person p);
}
```

```java
private static void checkAndExecute(List<Person> personList,NameChecker nameChecker,Executor executor){
    for (Person person:personList){
        if(nameChecker.check(person)){
            executor.execute(person);
        }
    }
}
```

```java
checkAndExecute(persons,p -> p.getLastName().startsWith("Z"),p -> System.out.println(p.getFirstName()));
```

我们还可以使我们的代码更加的简化 ———

