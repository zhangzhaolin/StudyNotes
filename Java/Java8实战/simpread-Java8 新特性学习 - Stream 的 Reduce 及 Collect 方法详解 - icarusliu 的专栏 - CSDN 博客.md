> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://blog.csdn.net/icarusliu/article/details/79504602 版权声明：本文为博主原创文章，未经博主允许不得转载。 https://blog.csdn.net/icarusliu/article/details/79504602

Stream 的使用方法在 [http://blog.csdn.net/icarusliu/article/details/79495534](http://blog.csdn.net/icarusliu/article/details/79495534) 一文中已经做了初步的介绍，但它的 Reduce 及 Collect 方法由于较为复杂未进行总结，现单独对这两个方法进行学习。
为简化理解，部分可以采用 Lambda 语法的地方采用了原始的语法；

# <a></a>0\. 涉及知识

大部分涉及到的知识在 [http://blog.csdn.net/icarusliu/article/details/79495534](http://blog.csdn.net/icarusliu/article/details/79495534) 中已经进行了介绍，还有几个新的类在下面将会涉及到，先行理解：

## <a></a>0.1 BiFunction

它是一个函数式接口，包含的函数式方法定义如下：

```
R apply(T t, U u);
```

可见它与 Function 不同点在于它接收两个输入返回一个输出； 而 Function 接收一个输入返回一个输出。
注意它的两个输入、一个输出的类型可以是不一样的。

## <a></a>0.2 BinaryOperator

它实际上就是继承自 BiFunction 的一个接口；我们看下它的定义：

```
public interface BinaryOperator<T> extends BiFunction<T,T,T>
```

上面已经分析了，BiFunction 的三个参数可以是一样的也可以不一样；而 BinaryOperator 就直接限定了其三个参数必须是一样的；
因此 BinaryOperator 与 BiFunction 的区别就在这。
它表示的就是两个相同类型的输入经过计算后产生一个同类型的输出。

## <a></a>0.3 BiConsumer

也是一个函数式接口，它的定义如下：

```
public interface BiConsumer<T, U> {

    /**
     * Performs this operation on the given arguments.
     *
     * @param t the first input argument
     * @param u the second input argument
     */
    void accept(T t, U u);
}
```

可见它就是一个两个输入参数的 Consumer 的变种。计算没有返回值。

# <a></a>1\. Reduce

Reduce 中文含义为：减少、缩小；而 Stream 中的 Reduce 方法干的正是这样的活：根据一定的规则将 Stream 中的元素进行计算后返回一个唯一的值。
它有三个变种，输入参数分别是一个参数、二个参数以及三个参数；

## <a></a>1.1 一个参数的 Reduce

定义如下：

```
Optional<T> reduce(BinaryOperator<T> accumulator)
```

假设 Stream 中的元素 a[0]/a[1]/a[2]…a[n - 1]，它表达的计算含义，使用 Java 代码来表述如下：

```
T result = a[0];  
for (int i = 1; i < n; i++) {
    result = accumulator.apply(result, a[i]);  
}
return result;  
```

也就是说，a[0] 与 a[1] 进行二合运算，结果与 a[2] 做二合运算，一直到最后与 a[n-1] 做二合运算。

可见，reduce 在求和、求最大最小值等方面都可以很方便的实现，代码如下，注意其返回的结果是一个 Optional 对象：

```
Stream<Integer> s = Stream.of(1, 2, 3, 4, 5, 6);
/**
 * 求和，也可以写成Lambda语法：
 * Integer sum = s.reduce((a, b) -> a + b).get();
 */
Integer sum = s.reduce(new BinaryOperator<Integer>() {
    @Override
    public Integer apply(Integer integer, Integer integer2) {
        return integer + integer2;
    }
}).get();

/**
 * 求最大值，也可以写成Lambda语法：
 * Integer max = s.reduce((a, b) -> a >= b ? a : b).get();
 */
Integer max = s.reduce(new BinaryOperator<Integer>() {
    @Override
    public Integer apply(Integer integer, Integer integer2) {
        return integer >= integer2 ? integer : integer2;
    }
}).get(); 
```

当然可做的事情更多，如将一系列数中的正数求和、将序列中满足某个条件的数一起做某些计算等。

## <a></a>1.2 两个参数的 Reduce

其定义如下：

```
T reduce(T identity, BinaryOperator<T> accumulator)
```

相对于一个参数的方法来说，它多了一个 T 类型的参数；实际上就相当于需要计算的值在 Stream 的基础上多了一个初始化的值。
同理，当对 n 个元素的数组进行运算时，其表达的含义如下：

```
T result = identity; 
for (int i = 0; i < n; i++) {
    result = accumulator.apply(result, a[i]);  
}
return result;  
```

注意区分与一个参数的 Reduce 方法的不同：它多了一个初始化的值，因此计算的顺序是 identity 与 a[0] 进行二合运算，结果与 a[1] 再进行二合运算，最终与 a[n-1] 进行二合运算。
因此它与一参数时的应用场景类似，不同点是它使用在可能需要某些初始化值的场景中。

使用示例，如要将一个 String 类型的 Stream 中的所有元素连接到一起并在最前面添加 [value] 后返回：

```
Stream<String> s = Stream.of("test", "t1", "t2", "teeeee", "aaaa", "taaa");
/**
 * 以下结果将会是：　[value]testt1t2teeeeeaaaataaa
 * 也可以使用Lambda语法：
 * System.out.println(s.reduce("[value]", (s1, s2) -> s1.concat(s2)));
 */
System.out.println(s.reduce("[value]", new BinaryOperator<String>() {
    @Override
    public String apply(String s, String s2) {
        return s.concat(s2);
    }
})); 
```

## <a></a>1.3 三个参数的 Reduce

三个参数时是最难以理解的。
先来看其定义：

```
<U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner)
```

分析下它的三个参数：

*   identity: 一个初始化的值；这个初始化的值其类型是泛型 U，与 Reduce 方法返回的类型一致；注意此时 Stream 中元素的类型是 T，与 U 可以不一样也可以一样，这样的话操作空间就大了；不管 Stream 中存储的元素是什么类型，U 都可以是任何类型，如 U 可以是一些基本数据类型的包装类型 Integer、Long 等；或者是 String，又或者是一些集合类型 ArrayList 等；后面会说到这些用法。
*   accumulator: 其类型是 BiFunction，输入是 U 与 T 两个类型的数据，而返回的是 U 类型；也就是说返回的类型与输入的第一个参数类型是一样的，而输入的第二个参数类型与 Stream 中元素类型是一样的。
*   combiner: 其类型是 BinaryOperator，支持的是对 U 类型的对象进行操作；

第三个参数 combiner 主要是使用在并行计算的场景下；如果 Stream 是非并行时，第三个参数实际上是不生效的。
因此针对这个方法的分析需要分并行与非并行两个场景。

### <a></a>1.3.1 非并行

如果 Stream 是非并行的，combiner 不生效；
其计算过程与两个参数时的 Reduce 基本是一致的。
如 Stream 中包含了 N 个元素，其计算过程使用 Java 代码表述如下：

```
U result = identity;  
for (T element：a) {
    result = accumulator.apply(result, element);  
}
return result; 
```

这个含义与 1.2 中的含义基本是一样的——除了类型上，Result 的类型是 U，而 Element 的类型是 T！如果 U 与 T 一样，那么与 1.2 就是完全一样的；
就是因为不一样，就存在很多种用法了。如假设 U 的类型是 ArrayList，那么可以将 Stream 中所有元素添加到 ArrayList 中再返回了，如下示例：

```
/**
 * 以下reduce生成的List将会是[aa, ab, c, ad]
 * Lambda语法：
 *  System.out.println(s1.reduce(new ArrayList<String>(), (r, t) -> {r.add(t); return r; }, (r1, r2) -> r1));
 */
Stream<String> s1 = Stream.of("aa", "ab", "c", "ad");
System.out.println(s1.reduce(new ArrayList<String>(),
        new BiFunction<ArrayList<String>, String, ArrayList<String>>() {
            @Override
            public ArrayList<String> apply(ArrayList<String> u, String s) {
                u.add(s);
                return u;
            }
        }, new BinaryOperator<ArrayList<String>>() {
            @Override
            public ArrayList<String> apply(ArrayList<String> strings, ArrayList<String> strings2) {
                return strings;
            }
        }));
```

也可以进行元素过滤，即[模拟](https://www.baidu.com/s?wd=%E6%A8%A1%E6%8B%9F&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd) Stream 中的 Filter 函数：

```
/**
 * 模拟Filter查找其中含有字母a的所有元素，打印结果将是aa ab ad
 * lambda语法：
 * s1.reduce(new ArrayList<String>(), (r, t) -> {if (predicate.test(t)) r.add(t);  return r; },
        (r1, r2) -> r1).stream().forEach(System.out::println);
 */
Stream<String> s1 = Stream.of("aa", "ab", "c", "ad");
Predicate<String> predicate = t -> t.contains("a");
s1.reduce(new ArrayList<String>(), new BiFunction<ArrayList<String>, String, ArrayList<String>>() {
            @Override
            public ArrayList<String> apply(ArrayList<String> strings, String s) {
                if (predicate.test(s)) strings.add(s);
                return strings;
            }
        },
        new BinaryOperator<ArrayList<String>>() {
            @Override
            public ArrayList<String> apply(ArrayList<String> strings, ArrayList<String> strings2) {
                return strings;  
            }
        }).stream().forEach(System.out::println);
```

注意由于是非并行的，第三个参数实际上没有什么意义，可以指定 r1 或者 r2 为其返回值，甚至可以指定 null 为返回值。

### <a></a>1.3.2 并行

当 Stream 是并行时，第三个参数就有意义了，它会将不同线程计算的结果调用 combiner 做汇总后返回。
注意由于采用了并行计算，前两个参数与非并行时也有了差异！
举个简单点的例子，计算 4+1+2+3 的结果，其中 4 是初始值：

```
/**
 * lambda语法：
 * System.out.println(Stream.of(1, 2, 3).parallel().reduce(4, (s1, s2) -> s1 + s2
 , (s1, s2) -> s1 + s2));
 **/
System.out.println(Stream.of(1, 2, 3).parallel().reduce(4, new BiFunction<Integer, Integer, Integer>() {
            @Override
            public Integer apply(Integer integer, Integer integer2) {
                return integer + integer2;
            }
        }
        , new BinaryOperator<Integer>() {
            @Override
            public Integer apply(Integer integer, Integer integer2) {
                return integer + integer2;
            }
        }));
```

并行时的计算结果是 18，而非并行时的计算结果是 10！
为什么会这样？
先分析下非并行时的计算过程；第一步计算 4 + 1 = 5，第二步是 5 + 2 = 7，第三步是 7 + 3 = 10。按 1.3.1 中所述来理解没有什么疑问。
那问题就是非并行的情况与理解有不一致的地方了！
先分析下它可能是通过什么方式来并行的？按非并行的方式来看它是分了三步的，每一步都要依赖前一步的运算结果！那应该是没有办法进行并行计算的啊！可实际上现在并行计算出了结果并且关键其结果与非并行时是不一致的！
那要不就是理解上有问题，要不就是这种方式在并行计算上存在 BUG。
暂且认为其不存在 BUG，先来看下它是怎么样出这个结果的。猜测初始值 4 是存储在一个变量 result 中的；并行计算时，线程之间没有影响，因此每个线程在调用第二个参数 BiFunction 进行计算时，直接都是使用 result 值当其第一个参数（由于 Stream 计算的延迟性，在调用最终方法前，都不会进行实际的运算，因此每个线程取到的 result 值都是原始的 4），因此计算过程现在是这样的：线程 1：1 + 4 = 5；线程 2：2 + 4 = 6；线程 3：3 + 4 = 7；Combiner 函数： 5 + 6 + 7 = 18！
通过多种情况的测试，其结果都符合上述推测！

如以下示例：

```
/**
 * lambda语法：
 * System.out.println(Stream.of(1, 2, 3).parallel().reduce(4, (s1, s2) -> s1 + s2
 , (s1, s2) -> s1 * s2));
 */
System.out.println(Stream.of(1, 2, 3).parallel().reduce(4, new BiFunction<Integer, Integer, Integer>() {
            @Override
            public Integer apply(Integer integer, Integer integer2) {
                return integer + integer2;
            }
        }
        , new BinaryOperator<Integer>() {
            @Override
            public Integer apply(Integer integer, Integer integer2) {
                return integer * integer2;
            }
        }));
```

以上示例输出的结果是 210！
它表示的是，使用 4 与 1、2、3 中的所有元素按 (s1,s2) -> s1 + s2(accumulator) 的方式进行第一次计算，得到结果序列 4+1, 4+2, 4+3，即 5、6、7；然后将 5、6、7 按 combiner 即(s1, s2) -> s1 * s2 的方式进行汇总，也就是 5 * 6 * 7 = 210。
使用函数表示就是：(4+1) * (4+2) * (4+3) = 210;

reduce 的这种写法可以与以下写法结果相等（但过程是不一样的，三个参数时会进行并行处理）：

```
System.out.println(Stream.of(1, 2, 3).map(n -> n + 4).reduce((s1, s2) -> s1 * s2));
```

这种方式有助于理解并行三个参数时的场景，实际上就是第一步使用 accumulator 进行转换（它的两个输入参数一个是 identity, 一个是序列中的每一个元素），由 N 个元素得到 N 个结果；第二步是使用 combiner 对第一步的 N 个结果做汇总。

但这里需要注意的是，如果第一个参数的类型是 ArrayList 等对象而非基本数据类型的包装类或者 String，第三个函数的处理上可能容易引起误解，如以下示例：

```
/**
 * 模拟Filter查找其中含有字母a的所有元素，打印结果将是aa ab ad
 * lambda语法：
 * s1.parallel().reduce(new ArrayList<String>(), (r, t) -> {if (predicate.test(t)) r.add(t);  return r; },
 (r1, r2) -> {System.out.println(r1==r2); return r2; }).stream().forEach(System.out::println);
 */
Stream<String> s1 = Stream.of("aa", "ab", "c", "ad");
Predicate<String> predicate = t -> t.contains("a");
s1.parallel().reduce(new ArrayList<String>(), new BiFunction<ArrayList<String>, String, ArrayList<String>>() {
            @Override
            public ArrayList<String> apply(ArrayList<String> strings, String s) {
                if (predicate.test(s)) {
                    strings.add(s);
                }

                return strings;
            }
        },
        new BinaryOperator<ArrayList<String>>() {
            @Override
            public ArrayList<String> apply(ArrayList<String> strings, ArrayList<String> strings2) {
                System.out.println(strings == strings2);
                return strings;
            }
        }).stream().forEach(System.out::println);
```

其中 System.out.println(r1==r2) 这句打印的结果是什么呢？经过运行后发现是 True！
为什么会这样？这是因为每次第二个参数也就是 accumulator 返回的都是第一个参数中 New 的 ArrayList 对象！因此 combiner 中传入的永远都会是这个对象，这样 r1 与 r2 就必然是同一样对象！
因此如果按理解的，combiner 是将不同线程操作的结果汇总起来，那么一般情况下上述代码就会这样写 (lambda)：　

```
Stream<String> s1 = Stream.of("aa", "ab", "c", "ad");

//模拟Filter查找其中含有字母a的所有元素，由于使用了r1.addAll(r2)，其打印结果将不会是预期的aa ab ad
Predicate<String> predicate = t -> t.contains("a");
s1.parallel().reduce(new ArrayList<String>(), (r, t) -> {if (predicate.test(t)) r.add(t);  return r; },
        (r1, r2) -> {r1.addAll(r2); return r1; }).stream().forEach(System.out::println);
```

这个时候出来的结果与预期的结果就完全不一样了，要多了很多元素！

# <a></a>3.1.2.7 collect

collect 含义与 Reduce 有点相似；
先看其定义：

```
<R> R collect(Supplier<R> supplier,
              BiConsumer<R, ? super T> accumulator,
              BiConsumer<R, R> combiner);
```

仍旧先分析其参数（参考其 JavaDoc）：

*   supplier：动态的提供初始化的值；创建一个可变的结果容器（JAVADOC）；对于并行计算，这个方法可能被调用多次，每次返回一个新的对象；
*   accumulator：类型为 BiConsumer，注意这个接口是没有返回值的；它必须将一个元素放入结果容器中（JAVADOC）。
*   combiner：类型也是 BiConsumer，因此也没有返回值。它与三参数的 Reduce 类型，只是在并行计算时汇总不同线程计算的结果。它的输入是两个结果容器，必须将第二个结果容器中的值全部放入第一个结果容器中（JAVADOC）。

可见 Collect 与分并行与非并行两种情况。
下面对并行情况进行分析。
直接使用上面 Reduce 模拟 Filter 的示例进行演示 (使用 lambda 语法）：

```
/**
 * 模拟Filter查找其中含有字母a的所有元素，打印结果将是aa ab ad
 */
Stream<String> s1 = Stream.of("aa", "ab", "c", "ad");
Predicate<String> predicate = t -> t.contains("a");
System.out.println(s1.parallel().collect(() -> new ArrayList<String>(),
        (array, s) -> {if (predicate.test(s)) array.add(s); },
        (array1, array2) -> array1.addAll(array2)));
```

根据以上分析，这边理解起来就很容易了：每个线程都创建了一个结果容器 ArrayList，假设每个线程处理一个元素，那么处理的结果将会是 [aa],[ab],[],[ad] 四个结果容器（ArrayList）；最终再调用第三个 BiConsumer 参数将结果全部 Put 到第一个 List 中，因此返回结果就是打印的结果了。

JAVADOC 中也在强调结果容器（result container) 这个，那是否除集合类型，其结果 R 也可以是其它类型呢？
先看基本类型，由于 BiConsumer 不会有返回值，如果是基本数据类型或者 String，在 BiConsumer 中加工后的结果都无法在这个函数外体现，因此是没有意义的。
那其它非集合类型的 Java 对象呢？如果对象中包含有集合类型的属性，也是可以处理的；否则，处理上也没有任何意义，combiner 对象使用一个 Java 对象来更新另外一个对象？至少目前我没有想到这个有哪些应用场景。它不同 Reduce，Reduce 在 Java 对象上是有应用场景的，就因为 Reduce 即使是并行情况下，也不会创建多个初始化对象，combiner 接收的两个参数永远是同一个对象，如假设人有很多条参加会议的记录，这些记录没有在人本身对象里面存储而在另外一个对象中；人本身对象中只有一个属性是最早参加会议时间，那就可以使用 reduce 来对这个属性进行更新。当然这个示例不够完美，它能使用其它更快的方式实现，但至少通过 Reduce 是能够实现这一类型的功能的。

<link href="https://csdnimg.cn/release/phoenix/mdeditor/markdown_views-7b4cdcb592.css" rel="stylesheet">