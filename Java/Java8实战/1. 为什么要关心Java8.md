# 第一章 为什么要关心Java8

## 前言

### 行为参数化

行为参数化，就是可以帮助我们处理频繁变化需求的一种**软件开发模式**。通俗的说，就是拿出一个代码块，把它准备好，而不去执行它。这个代码块以后可以被程序的其他部分调用，这就意味着我们需要延迟这段代码的执行。

在`Java 8`之前需要使用匿名类来实现行为参数化。

### ✍函数式编程

函数式编程：Java8里面将代码传递给方法的功能（同时也能够返回代码并将其包含在数据结构中），还让我们使用一整套新技巧。

## 一个苹果开始

假设有一个苹果类，我们要从苹果列表中筛选出颜色为绿色的那些苹果 ：

![](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E6%88%AA%E5%9B%BE/Java8%E5%AE%9E%E6%88%98/Character1/1.png)

接下来，有人想要选出那些比较重的苹果，于是，你开始再次写了下面的方法，某些行开始了复制粘贴 ：

```java
private static List<Apple> filterHeavyApples(List<Apple> appleList){
    List<Apple> result = new ArrayList<>();
    for (Apple apple:appleList){
        if(apple.getWeight() > 35){
            result.add(apple);
        }
    }
    return result;
}
```

我们发现，这和之间的`filterGreenApples`方法没有明显的差别，只有判断条件有所不同 。

![](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E6%88%AA%E5%9B%BE/Java8%E5%AE%9E%E6%88%98/Character1/2.png)

我们想要写一个公用的方法，无论是什么筛选条件（苹果重量啦、苹果颜色等），都可以进入到这个方法中，然后替换红框的部分 。

这在Java8之前是无法做到的，但是在Java8当中，方法可以被当做值，然后传递到另外的方法中 。

![](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E6%88%AA%E5%9B%BE/Java8%E5%AE%9E%E6%88%98/Character1/3.png)

代码如下所示 ：

```java
public static void main(String[] args) {
    List<Apple> inventory = Arrays.asList(
            new Apple(40d,Color.BLUE),new Apple(15.7,Color.BLUE),new Apple(32.1,Color.RED)
    );
    System.out.println(filterApples(inventory,AppleSort::isGreenApple));
}

private static List<Apple> filterApples(List<Apple> appleList, Predicate<Apple> predicate){
    List<Apple> result = new ArrayList<>();
    for (Apple apple:appleList){
        if (predicate.test(apple)){
            result.add(apple);
        }
    }
    return result;
}

private static boolean isGreenApple(Apple apple){
    return apple.getColor() == Color.BLUE;
}
private static boolean isHeavyApple(Apple apple){
    return apple.getWeight() > 35;
}
```

## 从传递方法到Lambda

我们可以看到代码中的`isGreenApple()`、`isHeavyApple()`方法，这些短方法到后期可能会越来越多……但是，Java8也解决了这个问题，它引入了一套新记法（匿名函数或Lambda）：

```java
// System.out.println(filterApples(inventory,AppleSort::isGreenApple));
filterApples(inventory,apple -> apple.getColor() == Color.BLUE);
```

或者 ：

```java
filterApples(inventory,apple -> apple.getWeight() > 50)
```

甚至可以这样 ：

```java
filterApples(inventory,apple -> apple.getWeight() > 35 || apple.getColor() == Color.BLUE);
```

## 流

### 什么是『数据流』

数据流：数据流是指一组有顺序的、有起点和终点的字节集合，程序从键盘中接受数据或者向文件中写入数据，或者在网络连接上的读写操作，都可以使用数据流来完成。

几乎每一个Java应用都会处理集合，例如，你需要从一个列表中筛选出金额较高的交易，并且按照货币进行分组。（代码中的`Currency`是JDK自带类型）

```java
public class Transaction {
    private BigDecimal price;
    private Currency currency;
}
```

```java
List<Transaction> transactions = Arrays.asList(
        new Transaction(BigDecimal.valueOf(2000),Currency.getInstance(Locale.TAIWAN)),
        new Transaction(BigDecimal.valueOf(11000),Currency.getInstance(Locale.CHINA)),
        new Transaction(BigDecimal.valueOf(5000),Currency.getInstance(Locale.US)),
        new Transaction(BigDecimal.valueOf(3500),Currency.getInstance(Locale.CHINA)),
        new Transaction(BigDecimal.valueOf(4000),Currency.getInstance(Locale.TAIWAN)),
        new Transaction(BigDecimal.valueOf(500),Currency.getInstance(Locale.CHINA))
);
```

在JDK8之前，你需要写下面的方法 ：

```java
Map<Currency,List<Transaction>> transactionsByCurrencies = new HashMap<>();
for(Transaction transaction:transactions){
    if (transaction.getPrice().compareTo(BigDecimal.valueOf(1000)) > 0) {
        Currency currency = transaction.getCurrency();
        List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);
        if (transactionsForCurrency == null){
            transactionsForCurrency = new ArrayList<>();
            transactionsByCurrencies.put(currency,transactionsForCurrency);
        }
        transactionsForCurrency.add(transaction);
    }
}
// 打印结果
System.out.println(transactionsByCurrencies);
```

在JDK8中加入了Stream API，你现在可以这样解决问题了 ：

```java
Map<Currency,List<Transaction>> transactionsByCurrencies = transactions.stream()
        .filter(transaction -> transaction.getPrice()
        // 筛选金额较高的交易
        .compareTo(BigDecimal.valueOf(1000)) > 0)
        // 按照货币进行分组
        .collect(Collectors.groupingBy(Transaction::getCurrency));
System.out.println(transactionsByCurrencies);
```

我们暂时不需要理解这段代码，只需要知道有了流之后，代码量可以大幅度减少。

新的`Stream API`和Java现有的集合API行为差不多：他们都能够访问数据项目的序列。`Collection`主要是为了存储和访问数据，而`Stream`主要用于描述对数据的计算。 `Stream`提倡使用并行处理一个`Stream`元素，筛选一个`Collection`最快方法就是将其转换成`Stream` ，然后再转换回`List` 。

『利用`Stream`和`Lambda`』

顺序处理 ：

```java
List<Apple> heavyApples = inventory.stream().filter(apple -> apple.getWeight().compareTo(30d) > 0).collect(Collectors.toList());
```

并行处理 ：

```java
List<Apple> heavyApples = inventory.parallelStream().filter(apple -> apple.getWeight().compareTo(30d) > 0)
                    .collect(Collectors.toList());
```

## 默认方法

接口类如今可以包含实现类没有提供实现的方法签名了 。

例如，在Java8里，你可以直接对`List`调用`sort()`方法，它是用Java8 List接口中如下所示的默认方法实现的 ：

```java
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray();
    Arrays.sort(a, (Comparator) c);
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {
        i.next();
        i.set((E) e);
    }
}
```



