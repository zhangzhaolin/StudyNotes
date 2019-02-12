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

函数式接口的抽象方法的签名基本上就是Lambda表达式的签名。我们将这种抽象方法叫做**函数描述符**。

Lambda表达式可以被赋值给一个变量，或传递给一个接受函数式接口作为参数的方法，当然，这个Lambda表达式签名要和函数式接口的抽象方法一样。比如 ：

```java
() -> System.out.println("hello world");
```

不接受参数并且返回`void`，这恰恰就是`Runnable`接口中`run`方法的签名。

下面的例子是错误的 ：

```java
Predicate<Apple> p = (Apple apple) -> apple.getWeight(); //! ERROR
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

## 把Lambda付诸实践 —— 环绕执行模式

**环绕执行模式**：资源处理（例如处理文件或者数据库）时常见的一个模式就是打开一个资源、做一些处理、然后关闭资源。这个设置和清理阶段总是很类似，并且会围绕着执行处理的那些重要的代码，这就是所谓的*环绕执行*模式

![](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E6%88%AA%E5%9B%BE/Java8%E5%AE%9E%E6%88%98/Character3/3.png)

下面是读取文件中的一行的代码 ：

```java
private static String processFile() throws IOException {
    try (
            BufferedReader reader = new BufferedReader(new FileReader("E:\\学习内容\\StudyNotes\\Java\\自己的Java笔记\\Java泛型"));
    ) {
        return reader.readLine();
    }
}
```

### 第一步 行为参数化

目前这段代码只能读取文件的一行，但是如果要读取文件的两行，或者是返回某个高频词语呢？你需要一种方法把行为传递给`processFile`，以便它可以利用`BufferedReader`执行不同的行为。

例如，下面代码就是从`BufferedReader`中读取两行：

```java
String result = processFile(reader->reader.readLine() + reader.readLine());
```

### 第二步 使用函数式接口传递行为

现在，我们需要创建一个能够匹配`BuferedReader -> String`，还能够抛出`IOException`的接口 ：

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader bufferedReader) throws IOException;
}
```

现在，你可以把这个接口作为新的`processFile`方法的参数了：

```java
private static String processFile(BufferedReaderProcessor processor) throws IOException {
    // ...
}
```

### 第三步 执行一个行为

任何`BufferedReader -> String`形式的Lambda都可以 作为一个参数来传递，因为他们符合`BufferedReaderProcessor`接口中定义的`process`方法 。

```java
private static String processFile(BufferedReaderProcessor processor) throws IOException {
    try (
            BufferedReader reader = new BufferedReader(new FileReader("E:\\学习内容\\StudyNotes\\Java\\自己的Java笔记\\Java泛型.md"));
    ) {
         return processor.process(reader);
    }
}
```

### 第四步 传递Lambda

现在我们可以传递不同的Lambda重用`processFile`方法，并以不同的方式处理文件 ：

```java
// 处理一行
String oneLine = processFile((reader) -> reader.readLine());
// 处理两行
String twoLine = processFile((reader) -> reader.readLine() + reader.readLine());
```

```java
// 读取全部行
String result = processFile((reader) -> {
    StringBuilder response = new StringBuilder();
    String str = null;
    while ((str = reader.readLine()) != null){
        response.append(str).append("\n");
    }
    return response.toString();
});
```

## 处理函数式接口

函数式接口的抽象方法的签名称为**函数描述符**。为了应用不同的Lambda表达式，我们需要一套能够描述常见函数描述符的函数式接口。Java API中已经有了几个函数式接口 ：

### `Predicate`

`java.util.function.Predicate<T>`接口定义了一个名叫`test`的抽象方法，它接受泛型`T`对象，并返回一个`boolean`。

```java
@FunctionalInterface
public interface Predicate<T>{
    boolean test(T t);
}
```

```java
private static <T> List<T> filter(List<T> list,Predicate<T> predicate){
    List<T> result = new ArrayList<>();
    for(T s : list){
        if (predicate.test(s)){
            result.add(s);
        }
    }
    return result;
}
```

```java
public static void main(String[] args) {
    List<String> listOfString = Arrays.asList(null,"","1","hello",null,"world");
    Predicate<String> nonEmptyStringPredicate = result-> result != null && !result.isEmpty();
    List<String> nonEmptyStringList = filter(listOfString,nonEmptyStringPredicate);
    System.out.println(nonEmptyStringList);
    // [1, hello, world]
}
```

### `Consumer`

`java.util.function.Consumer<T>`定义了一个名为`accept`的抽象方法，它接受泛型`T`对象，没有返回（`void`）

```java
@FunctionalInterface
public interface Consumer<T>{
    void accept(T t);
}
```

```java
private static <T> void forEach(List<T> list, Consumer<T> consumer){
    for(T s : list) {
        consumer.accept(s);
    }
}
```

```java
forEach(Arrays.asList(1,2,3,4),(Integer i)->System.out.println(i));
```

### `Function`

`java.util.function.Function<T,R>`接口定义了一个叫做`apply`的方法，它接受一个泛型`T`的对象，并返回一个泛型`R`的对象。

```java
@FunctionalInterface
public interface Function<T, R>{
    R apply(T t);
}
```

```java
private static <T,R> List<R> map(List<T> list,Function<T,R> function){
    List<R> result = new ArrayList<>();
    for(T  s : list){
        result.add(function.apply(s));
    }
    return result;
}
```

```java
List<Integer> stringLength = map(Arrays.asList("hello","world!"),s->s.length());
System.out.println(stringLength);
// [5,6]
```

### 原始类型特化

三个泛型函数式接口 ：`Predicate`、`Consumer`、`Function`只能绑定到引用类型（和泛型擦出机制有关）。

装箱和拆箱是自动完成的，但是装箱后需要更多的内存，并需要额外的内存搜索来获取被包裹的原始值。

Java8为函数式接口带来了专门的版本，以便在输入输出时都是原始类型时避免自动装箱的操作 ：

```java
@FunctionalInterface
public interface IntPredicate{
    boolean test(int value);
}
```

```java
IntPredicate intPredicate = (int i)->i%2 == 0;
intPredicate.test(4);

Predicate<Integer> predicate = (Integer i)->i % 2 == 0;
predicate.test(5);
```

类似的函数式接口还有 ：`LongPredicate`、`DoublePredicate`、`IntFunction<R>`、`IntToDoubleFunction`等

|       使用案例        |                       Lambda例子                        |                   对应的函数式接口                    |
| :-------------------: | :-----------------------------------------------------: | :---------------------------------------------------: |
|      布尔表达式       |         `(List<String> list) -> list.isEmpty()`         |               `Predicate<List<string>`                |
|       创建对象        |                  `() -> new Apple(10)`                  |                   `Supplier<Apple>`                   |
|     消费一个对象      |           `(Apple a)->System.out.println(a)`            |                   `Consumer<Apple>`                   |
| 从一个对象中选择/提取 |                 `(String s)->s.length`                  | `Function<String,Integer>`或者`ToIntFunction<String>` |
|      合并两个值       |                  `(int a,int b)->a*b`                   |                  `IntBinaryOperator`                  |
|     比较两个对象      | `(Apple a1,Apple a2)->a1.getWeight().compareTo(a2.get)` |                  `Comparator<Apple>`                  |

### 异常和函数式接口

Java API中提供的函数式接口都没有抛出受检异常，如果你需要Lambda表达式来抛出异常，有两种方式 ：

- 定义自己的函数式接口，并声明受检异常 ：

```java
@FunctionalInterface
public interface BufferedReaderProcess{
    String process(BufferedReader reader) throws IOException;
}
```

- 调用函数式接口API

```java
Function<BufferedReader,String> f = (BufferedReader reader)-> {
    try {
        return reader.readLine();
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
};
```

## 类型检查、类型推断和限制

当我们在说Lambda表达式时，说它可以为函数式接口生成一个实例。然而，Lambda表达式本身并不包含它在实现哪一个函数接口的信息。

### 类型检查

Lambda的类型是从Lambda上下文推断出来的，上下文（比如，接受它传递的方法的参数，或接受它的值的局部变量）中Lambda表达式需要的类型成为*目标类型* ：

![](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E6%88%AA%E5%9B%BE/Java8%E5%AE%9E%E6%88%98/Character3/4.png)

**类型检查过程可以分解为 ：**

- 找出`filter`方法声明
- 要求它是`Predicate<Apple>`（目标类型）对象的第二个正式参数
- `Predicate<Apple>`是一个函数式接口，定义了了一个叫做`test`的抽象方法
- `test`方法描述了一个函数描述符，它可以接受一个`Apple`对象参数，并返回`boolean`
- `filter`的任何实际参数都必须匹配这个要求

### 特殊的`void`兼容规则

如果一个Lambda的主体是一个语句表达式，它就和一个返回`void`的函数描述符相兼容（前提条件是参数列表兼容）。

例如，下面两行都是合法的，尽管`List`的`add`方法返回了一个`boolean`，而不是`Consumer`上下文（`T->void`）所要求的`void`：

```java
// Predicate 返回了一个boolean
Predicate<String> predicate = (String s) -> list.add(s);
// Consumer 返回了一个void
Consumer<String> consumer = (String s) -> list.add(s);
```

### 局部变量限制

Lambda表达式允许使用自由变量（不是参数，而是在外层作用于定义的变量），他们被称之为捕获Lambda ：

```java
int portNumber = 32;
Runnable runnable = ()->System.out.println(portNumber);
```

**被Lambda捕获的局部变量必须被显示声明为`final`，或者事实上就是`final`**，所以下面的代码将无法编译 ：

```java
int portNumber = 32;
Runnable runnable = ()->System.out.println(portNumber);
// ! portNumber = 5;
```

## 方法引用

方法引用可以被看作仅仅调用特定方法的Lambda的一种快捷写法。方法引用可以让你重复使用现有的方法定义，并像Lambda一样传递它们，如下是使用方法引用写的排序的例子 ：

```java
// before
inventory.sort((a1,a2)->a1.getWeight().compareTo(a2.getWeight()));
// after
inventory.sort(Comparator.comparing(Apple::getWeight));
```

`Apple::getWeight`方法引用其实就是lambda表达式`(Apple a)->a.getWeight()`的快捷方式。

### 管中窥豹

当你需要使用方法引用时，目标引用放在分隔符`::`前面，方法的名称放在后面 。例如，`Apple::getWeight`就是引用了`Apple`类中定义的方法`getWeight()`。

方法引用**🚫不需要括号**，因为你没有实际调用这个方法 。

| `Lambda` | 等效的方法引用 |
| :------: | :-----------: |
| `(Apple a)->a.getWeight()` | `Apple::getWeight` |
| `()->Thread.currentThread().dumpStack()` | `Thread.currentThread()::dumpStack` |
| **`(str,i)->str.substring(i)`** | `String::substring` |
| `(String s)->System.out.println(s)` | `System.out::println` |

例如，想要对一个字符串`List`进行排序，忽略大小写。`List`的`sort()`方法需要一个`Comparator`作为参数。`Comparator`描述了一个具有一个`(T,T)->int`的**函数描述符** ：

```java
List<String> str = Arrays.asList("a","b","A","B");
str.sort((s1,s2)->s1.compareToIgnoreCase(s2));
```

上述代码可以使用方法引用改写成下面的样子 ：

```java
List<String> str = Arrays.asList("a","b","A","B");
str.sort(String::compareToIgnoreCase);
```

### 三类方法引用

- 指向<u>静态方法</u>的方法引用 。

```
Integer的parseInt方法 ——» Integer::parseInt
```

- 指向<u>任意类型实例方法</u>的方法引用（你在引用一个对象的方法，而这个对象本身是Lambda参数）

```
(String s)->s.toUpperCase()
String::toUpperCase
```

- 指向<u>现有对象的实例方法</u>的方法引用（你在Lambda中调用一个已经存在的外部对象中的方法）

```
()->expensive.getValue()
expensive::getValue
```

### 构造函数引用

`ClassName::new`可以创建构造函数的引用，例如，若一个构造函数没有参数，那么他适合`Supplier`的签名 ：

```
Supplier<Apple> c1 = ()->new Apple();
Apple a1 = c1.get();
```

等价于 ：

```java
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get();
```

如果你的构造函数签名为`Apple(Double weight)`，那么它适合`Function`的签名 ：

```java
Function<Double,Apple> f = (weight)->new Apple(weight);
Apple a2 = f.apply(12d);
```

等价于 ：

```java
Function<Double,Apple> f = Apple::new;
Apple a2 = f.apply(12d);
```

下述代码中，有一个由`Double`组成的`List`中的每一个元素都通过`map`方法传递给了`Apple`的构造函数，得到了一个具有不同重量苹果的`List`：

```java
public class Main {
    public static void main(String[] args) {
        List<Double> weights = Arrays.asList(13d, 4d, 5d);
        List<Apple> list1 = map(weights, Apple::new);
        System.out.println(list1);
	}
    public static <T, R> List<R> map(List<T> list, Function<T, R> f) {
        List<R> result = new ArrayList<>();
        for (T item : list) {
            result.add(f.apply(item));
        }
        return result;
    }
}

```

如果你具有两个参数的构造函数`Apple(Color color,Double weight)`，那么他就适合`BiFunction`接口的签名 ：

```java
BiFunction<Color, Double, Apple> biFunction = Apple::new;
// 与下面一句等价
BiFunction<Color, Double, Apple> biFunction = (Color c,Double b)->new Apple(c,b);
Apple a = biFunction.apply(Color.BLUE, 24d);
```

## 复合Lambda表达式的有用方法

### 比较器复合

```java
List<Apple> inventory = Arrays.asList(
        new Apple(Color.BLUE, 12d),
        new Apple(Color.RED, 5d),
        new Apple(Color.YELLOW, 30d),
        new Apple(Color.RED, 12d)
);
Comparator<Apple> c = Comparator.comparing(Apple::getWeight);
// 逆序
inventory.sort(Comparator.comparing(Apple::getWeight).reversed());
System.out.println("逆序 ： " + inventory);
// 比较器链
inventory.sort(Comparator.comparing(Apple::getWeight)
        .reversed()
        .thenComparing(Apple::getColor));
System.out.println("按照重量从大到小&按照颜色排序 : " + inventory);
```

### 谓词复合

谓词复合有这样几个函数 ：`negate`、`and`和`or`

```java
Apple apple = new Apple(Color.RED, 15d);
Predicate<Apple> redApple = (t) -> t.getColor() == Color.RED;
Predicate<Apple> notRedPredicate = redApple.negate();
// false
System.out.println(notRedPredicate.test(apple));
Predicate<Apple> redAndHeavyApple = redApple.and(item -> item.getWeight() > 10d);
// true
System.out.println(redAndHeavyApple.test(apple));
apple.setColor(Color.BLUE);
apple.setWeight(4d);
// false
System.out.println(redAndHeavyApple.test(apple));
Predicate<Apple> redAndHeavyAppleOrBlue = redApple.and(item -> item.getWeight() > 10d)
        .or(item -> item.getColor() == Color.BLUE);
// true
System.out.println(redAndHeavyAppleOrBlue.test(apple));
```

### 函数符合

```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> 2 * x;
Function<Integer, Integer> h = f.andThen(g); // 相当于 g(f(x))
System.out.println(h.apply(3)); // 8
h = f.compose(g); // 相当于 f(g(x))
System.out.println(h.apply(3)); // 7
```

对用`String`表示的一封信做出转换 ：

```java
public class Letter {
    public static String addHeader(String text) {
        return "From Zhang : " + text;
    }
    public static String addFooter(String text) {
        return text + "\n 至上.";
    }
    public static String checkSpelling(String text) {
        return text.replaceAll("Labda", "lambda");
    }
    public static void main(String[] args) {
        Function<String, String> addHeader = Letter::addHeader;
        Function<String, String> transformation = addHeader.andThen(Letter::checkSpelling)
                .andThen(Letter::addFooter);
        String result = transformation.apply("我喜欢用Labda");
        //From Zhang : 我喜欢用lambda
 		// 至上.
        System.out.println(result);
        transformation = addHeader.andThen(Letter::addFooter);
        result = transformation.apply("我喜欢用Lambda~");
        //From Zhang : 我喜欢用Lambda~
 		// 至上.
        System.out.println(result);
    }
}
```

## 小结

- Lambda表达式可以看做是一种匿名函数：它没有名称，但有参数列表、函数主题、返回类型，可能还有一个可以抛出的异常的列表。
- **函数式接口**就是仅仅声明了一个抽象方法的接口
- Java8提供了常见的函数式接口，放在`java.util.function`包里，包括`Predicate<T>`、`Function<T,R>`、`Supplier<T>`、`Consumer<T>`和`BinaryOperator<T>`
- 为了避免装箱操作，对`Predicate<T>`和`Function<T,R>`等通用函数式接口的原型类型特化 ：`IntPredicate`、`IntToLongFunction`等
- 环绕执行模式（即在方法所必需的代码中间，你需要执行点儿什么操作，比如资源分配和清理）可以配合Lambda提高灵活性和可重用性
- Lambda表达式所需要代表的类型称为目标类型
- 方法引用让你重复使用现在的方法实现并直接传递它们
- `Comparator`、`Predicate`和`Function`等函数式接口都有几个可以用来结合Lambda表达式的默认方法



