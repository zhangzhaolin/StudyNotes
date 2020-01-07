# 快速失败 VS 安全失败

## `fail-fast` 机制

> 如果一个系统，当有异常或者错误发生时就立即中断执行，这种设计称之为 `fail-fast`。相反，如果我们的系统可以在某种异常或者错误发生时继续执行，不会被中断，这种设计被称为 `fail-safe`。

例如一个最简单的 `fail-fast` 例子：

```java
public int divide(int divisor,int dividend){
    if(dividend == 0){
        throw new RuntimeException("dividend can't be null");
    }
    return divisor/dividend;
}
```

## 集合中的 `fail-fast`

在 Java 中的`fail-fast` 机制，通常指的是 Java 集合的一种错误检测机制。当多个线程对部分集合进行结构上的改变的操作时，有可能会产生 `fail-fast` 机制，这个时候会抛出 `ConcurrentModificationException`。

当方法检测到对象的并发修改，但不允许这种修改时就会抛出 `ConcurrentModificationException`。

但是有些情况下，自己的代码并没有在多线程中执行，却抛出了 `ConcurrentModificationException`异常。

## 异常复现

在 Java 中，如果在 `foreach` 循环里对某些集合元素进行元素的 `remove/add` 操作的时候，就会触发 `fail-fast` 机制，进而抛出 `ConcurrentModificationException`。

例如：

```java
List<String> userNames = new ArrayList<String>() {{
    add("Hollis");
    add("hollis");
    add("HollisChuang");
    add("H");
}};

for (String userName : userNames) {
    if (userName.equals("Hollis")) {
        userNames.remove(userName);
    }
}

System.out.println(userNames);
```

会抛出如下异常 ：

```java
Exception in thread "main" java.util.ConcurrentModificationException
at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
at java.util.ArrayList$Itr.next(ArrayList.java:859)
at com.hollis.ForEach.main(ForEach.java:22)
```

其实，这里的 `for` 循环使用了一个语法糖，真正的是使用 `while` 和 `Iterator` 实现的：

> 反编译结果

```java
List<String> userNames = new ArrayList<String>() {
    {
        this.add("Hollis");
        this.add("hollis");
        this.add("HollisChuang");
        this.add("H");
    }
};
Iterator var2 = userNames.iterator();
while(var2.hasNext()) {
    String userName = (String)var2.next();
    if (userName.equals("Hollis")) {
        userNames.remove(userName);
    }
}
System.out.println(userNames);
```

## 异常原理

通过异常堆栈，我们可以追踪到真正抛出异常的代码是：

```java
java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
```

该方法的实现为：

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

### `modCount`

这里的 `modCount` 是 `AbstractList` 中的成员变量，<u>用来记录实际集合被修改的次数</u>：

```java
public abstract class AbstractList<E> ... {
	protected transient int modCount = 0;
}
public class ArrayList<E> extends AbstractList<E> ...{ ... }
```

例如，在 `ArrayList` 之中，我们可以看到 `modCount` 修改情况：

![](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/2020/01TIM%E6%88%AA%E5%9B%BE20200107145635.png)

### `expectedModCount`

`expectedModCount` 是 `Itr` 类中的一个内部成员，而 `Itr` 是 `ArrayList` 中的内部类：

```java
public class ArrayList<E> ... {
    // ... 
    private class Itr implements Iterator<E> {
        int expectedModCount = modCount;
        Itr() {}
        // ...
    }
   	// ...
}
```

`expectedModCount` 值随着 `Itr` 被创建而被初始化为 `modCount`。仅有当通过迭代器对集合进行修改时，该值才会变化：

```java
private class Itr implements Iterator<E> {
	// ...
    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();
        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
	}
}

private class ListItr extends Itr implements ListIterator<E> {
	// ...
    public void add(E e) {
            checkForComodification();
            try {
                int i = cursor;
                ArrayList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
}
```

接下来我们可以看下 `obj.remove("...")` 之中发生了什么事情，为什么会导致 `modCount` 和 `expectedModCount` 不一致。

通过翻阅 `remove()` 的源码，我们发现核心内容如下：

```java
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

我们可以发现，这里只修改了 `modCount` ，而没有修改 `expectedModCount`。

![](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/2019/11/20191128102833.png)

总结一下就是，在使用增强型 `for` 循环时，也就是说，使用 `Iterator` 遍历集合时（增强型 `for` 循环仅仅是语法糖），使用集合类自己的修改方法会导致 `modCount` 和`expectedModCount` 不一致。

### 如何解决

我们可以通过上述的代码观察到，在 `Itr` 类中，有一个 `remove()` 方法，该方法先是会调用 `ArrayList.remove()` 方法，修改了 `modCount` 变量，紧接着采用 `expectedModCount = modCount;` 语句保证 `expectedModCount` 和 `modCount` 相等。因此，我们可以使用 `Itr.remove()` 方法来修改集合。同理，可以使用 `ListItr.add()` 添加集合：

```java
java.util.Iterator<String> iterator = userNames.iterator();
while (iterator.hasNext()) {
    String str = iterator.next();
    if ("Hollis".equals(str)) {
        iterator.remove();
    }
}
```

## `fail-safe` 机制

为了避免触发 `fail-fast` 机制导致异常，我们可以使用 Java 中提供的采用了 `fail-safe` 机制的集合类。

`fail-safe` 的集合容器在进行操作的时候，不是直接在集合内容上操作，而是先复制原有的集合内容，在拷贝的集合中进行操作。

`java.util.concurrent` 包下的容器都是 `fail-safe` 的，<u>可以在多线程并发使用，并发修改</u>。同时也可以在 `foreach` 中进行 `add/remove` 操作。

例如，`CopyOnWriteArrayList` 这个 `fail-safe` 集合类：

```java
CopyOnWriteArrayList<String> userNamesSafe = new CopyOnWriteArrayList<String>() {
    {
        add("Hollis");
        add("hollis");
        add("HollisChuang");
        add("H");
    }
};
for (String userName : userNamesSafe) {
    if (userName.equals("Hollis")) {
        userNamesSafe.remove(userName);
    }
}
System.out.println(userNamesSafe);
```

以上代码，使用了 `CopyOnWriteArrayList` 代替 `ArrayList`，就不会引发异常。

`fail-safe` 集合中，对集合的修改都是先拷贝一份副本，然后在副本上进行操作，而不是直接对原集合进行修改。并且这些修改方法，都是通过加锁来控制并发的。

因此，`CopyOnWriteArrayList` 中的迭代器在迭代的过程中不需要做 `fail-fast` 的并发检测。（`fail-fast` 主要目的时就是识别并发，然后通过异常的方式通知用户）。

虽然，通过这种技术避免了 `ConcurrentModificationException`，但是迭代器并不能访问到修改之后的内容：

```java
public static void main(String[] args) {
    List<String> userNames = new CopyOnWriteArrayList<String>() {{
        add("Hollis");
        add("hollis");
        add("HollisChuang");
        add("H");
    }};

    Iterator it = userNames.iterator();

    for (String userName : userNames) {
        if (userName.equals("Hollis")) {
            userNames.remove(userName);
        }
    }

    System.out.println(userNames);

    while(it.hasNext()){
        System.out.println(it.next());
    }
}
```

这是因为，`CopyOnWriteArrayList` 类的 `Iterator` 拿到的是快照，而不是实时的数据：

```java
static final class COWIterator<E> implements ListIterator<E> {
    private final Object[] snapshot;
    private COWIterator(Object[] elements, int initialCursor) {
    	cursor = initialCursor;
    	snapshot = elements;
	}
}
```

## 总结

- `fail-fast`：快速失败，在发生异常时立即中断执行；`fail-safe`：安全失败，程序在发生某种错误或者异常时能够继续执行，不会被中断。
- 在 Java 集合中，当多个线程同时对集合进行结构上的改变操作时，可能会产生 `fail-fast`，并抛出 `ConcurrentModificationException` 异常。
- 当单个线程对集合进行结构修改时，也可能会产生上述异常。原因是因为使用 `for-each` 或者 `Iterator` 进行集合遍历时，若使用集合自带的方法对集合进行修改，只会引起 `modCount` 的变化，而不会引起`expectedModCount`的变化，当两个参数不相等时，会抛出异常。
- `java.util.concurrent` 包下的集合都是 `fail-safe` 的。可以在 `for-each` 中使用自带的 `remove/add` 对集合进行修改而不会抛出异常。

## 参考资料

- [一不小心就让Java开发踩坑的fail-fast是个什么鬼？]( https://juejin.im/post/5cb683d6518825186d65402c )

