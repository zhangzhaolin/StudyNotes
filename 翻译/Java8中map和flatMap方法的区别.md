# Java8中map和flatMap方法的区别？

> 原文地址 ：
>
> https://stackoverflow.com/questions/26684562/whats-the-difference-between-map-and-flatmap-methods-in-java-8

## 问题

在Java8当中，[`Stream.map`](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#map-java.util.function.Function-)和[`Stream.flatMap`](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#flatMap-java.util.function.Function-)方法的区别是什么 ？

## 回答

### Stuart Marks

`map`和`flatMap`都可以应用于`Stream<T>`并返回`Stream<R>`。不同之处在于`map`操作为每一个输入值生成一个输出值，而`flatMap`为每一个输入值生成任意数量的输出值（0个或者多个）。

这体现在每个操作的参数中。

`map`操作接收一个`Function`，输入流中的每一个值都会调用它并产生一个结果值，结果值会发送到输出流中。

`flatMap`提供一个功能：从概念上想要“消耗”一个值并产生任意数量的值。然而，在Java中，返回任意数量值的方法很麻烦，因为方法只能返回零或者一个值。可以想象一个API，`flatMap`的`mapper`函数接收一个值并返回值的数组或列表，然后再送到输出流中。鉴于这是streams库，表示任意数量返回值的不二之选就是 ：`mapper`函数本身就返回一个流！映射器返回的流中的值会从流中排出并传递到输出流。每次调用`mapper`函数返回的值的“集合”在输出流中根本不被区分，因此输出被称之为“扁平化”。

典型的应用就是：如果想要发送空值，`flatMap`的`mapper`函数就返回`Stream.empty()`，或者像`Stream.of("a","b","c")`那样返回多个值。但是当然可以返回任何一个流。

### Dici

`Stream.flatMap`，通过名字可以猜测，是一个`map`和`flat`操作的组合。这意味着，你首先需要将一个函数应用于元素，然后将其“展平”。而`Stream.map`仅仅将函数应用于流而没有平展流。

为了理解流的扁平化，考虑像`[[1,2,3],[4,5,6],[7,8,9]]`这样具有“两个层次”的结构。扁平化意味着将其转换为“一级”结构：`[1,2,3,4,5,6,7,8,9]`

### rudy-vissers

我想要举两个例子来得到更加确切的观点 ：

```java
@Test
public void convertStringToUpperCaseStreams() {
    List<String> collected = Stream
        	.of("a", "b", "hello") // String流 
            .map(String::toUpperCase) // 返回一个流，流中包含了将给定函数应用于流中元素的结果
            .collect(Collectors.toList());
    assertEquals(asList("A", "B", "HELLO"), collected);
}
```

第一个例子没有特殊之处，应用一个`Function`并返回大写形式的`String`。

第二个例子是使用`flatMap`：

```java
public void testFlatMap() throws Exception {
    List<Integer> together =
            Stream.of(Arrays.asList(1, 2), Arrays.asList(3, 4)) // Stream<List<Integer>>
                    .flatMap((List<Integer> integers) -> integers.stream())
                    .map((Integer i) -> i + 1)
                    .collect(Collectors.toList());
    assertEquals(together, Arrays.asList(2, 3, 4, 5));
}
```

在第二个例子中，传递了一个List流，**它并不是Integer流！**

如果必须使用转换函数（通过映射），则首先必须将Stream展平为其他内容（Integer）

如果删除了`flatMap`方法，那么则会返回以下错误 ：

```
Operator '+' cannot be applied to 'java.util.List<java.lang.Integer>', 'int'
```

无法在整数列表上使用操作符 “+”！


