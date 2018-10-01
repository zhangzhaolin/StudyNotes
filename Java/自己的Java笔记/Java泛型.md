# Java泛型

## Java泛型介绍

- Java泛型类
- Java泛型方法
- Java泛型接口
- Java泛型擦除及其他相关内容
- Java泛型通配符

## Java泛型类

`ArrayList`大概是一个非常常见的一个泛型类了，在代码中指定其传入的类型为`<String>`类型，所以只能添加`String`类型的数据，而如果添加其他类型的数据则会报错。

![1538190250627](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-9-30/91917915.jpg)

在这里，`ArrayList`类有一个**类型参数**用来指定元素的类型 ：

```java
ArrayList<String> array = new ArrayList<String>();
```

参照`ArrayList`的源码：

```java
public class ArrayList<E> ... {
    // ...
}
```

可以写出一个简单的泛型类设计 ：

```java
public class DataHolder <T> {
    T item;
    public DataHolder(T item) {
        this.item = item;
    }
    public void setItem(T item){
        this.item = item;
    }
    public T getItem(){
        return this.item;
    }
    public static void main(String[] args) {
        DataHolder<String> holder = new DataHolder<>("something");
        //! holder.setItem(14);
        System.out.println(holder.getItem());
    }
}
```

`DataHolder`类引入了一个 **类型变量T**，用尖括号（`<>`）括起来，并放到类名的后面，泛型类可以有多个类型变量，例如 ：

```java
public class DataHolder<T,K>{
    // ...
}
```

一般的 ：

- 使用`E`表示集合的元素类型
- 使用`K,V`分别表示表的关键字和值
- 使用`T`(或者临近的`U`和`S`)表示“任意类型”

## Java泛型方法

刚才介绍的泛型是作用于整个类的，现在我们介绍泛型方法，泛型方法既可以存在于泛型类中，也可以存在于普通的类中。**如果可以使用泛型方法解决问题，那么应该尽量使用泛型方法**。

例子中使用的仍然是`DataHolder`类，但是我们加入了泛型方法 ：

```java
public class DataHolder <T> {
    T item;
    public DataHolder(T item) {
        this.item = item;
    }
    public void setItem(T item){
        this.item = item;
    }
    public T getItem(){
        return this.item;
    }
    public <T> void printInfo(T e){
        System.out.println(e);
    }
    @Override
    public String toString() {
        return "DataHolder{" +
                "item=" + item +
                '}';
    }
    public static void main(String[] args) {
        DataHolder<String> holder = new DataHolder<>("something");
        holder.printInfo(17);
        holder.printInfo(8.88);
        holder.printInfo(holder);
    }
}

```

注意 ：泛型方法中定义的**类型参数**`T`和泛型类中的类型参数没有任何关系 ！

```java
// 这个T是一个全新的类型，和泛型类中声明的T不是同一种类型!!
public <T> void printInfo(T e){
	System.out.println(e);
}
```

通过主函数调用得知 ：

```java
DataHolder<String> holder = new DataHolder<>("something");
holder.printInfo(17);
holder.printInfo(8.88);
holder.printInfo(holder);
```

泛型方法仍然可以将`int`、`double`、或者其他类型作为形式参数，和泛型类中的**参数类型**是不一样的类型。以下是泛型方法的基本特性 ：

- `public`和**返回值**中间非常重要，可以理解为声明此方法为泛型方法 ：

  ```java
  public <T> T printInfo(T e){
      System.out.println(e);
  	return e;
  }
  ```

- 只有声明了的方法才是泛型方法，泛型类中使用了泛型的成员方法并不是泛型方法
- 表明该方法将使用泛型类型`T`，此时才可以在方法中使用泛型类型`T`
- 和泛型类的定义一样，此处`T`可以随便写为任意标识，例如：`K`、`V`等等

## Java泛型接口

Java泛型接口的定义和Java泛型类基本相同，如下 ：

```java
public interface Generator<T> {
    public T next();
}
```

在泛型接口的实现类中，有两种实现方式 ：

1. 泛型接口未传入泛型参数类型时，和泛型类的声明方式相同，需要将泛型的声明加入到接口实现类中 ：

```java
public class FruitGenerator<T> implements Generator<T> {
    @Override
    public T next() {
        return null;
    }
}
```

2. 泛型接口传入了泛型参数类型，实现该接口的实现类，所有使用泛型的地方，都需要替换成传入的实参类型 ：

```java
public class FruitGenerator implements Generator<String>{
    @Override
    public String next() {
        return null;
    }
}
```

## Java泛型擦除及相关内容

**虚拟机中不存在泛型，只有普通的类和方法；所有的类型参数都会用它们的限定类型替换，如果没有限定类型替换，那么就用`Object`**，例如 ：

```java
public class Pair<T>{

    private T first;
    private T second;

    public Pair(){
        first = null;
        second = null;
    }

    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }

    public T getFirst() {
        return first;
    }

    public void setFirst(T first) {
        this.first = first;
    }

    public T getSecond() {
        return second;
    }

    public void setSecond(T second) {
        this.second = second;
    }


}
```

这个类的字节码如下所示 ：

```java
public class Pair {
  private Ljava/lang/Object; first
  private Ljava/lang/Object; second
  public <init>()V
   // ....
  public <init>(Ljava/lang/Object;Ljava/lang/Object;)V
   // ....
  public getFirst()Ljava/lang/Object;
   // ....
  public setFirst(Ljava/lang/Object;)V
   // ....
  public getSecond()Ljava/lang/Object;
   // ....
  public setSecond(Ljava/lang/Object;)V
   // ....
}
```

另外一个例子 ——

```java
Class clazz1 = new ArrayList<String>().getClass();
Class clazz2 = new ArrayList<Integer>().getClass();
System.out.println(clazz1);
System.out.println(clazz2);
System.out.println(clazz1 == clazz2);
```

运行结果如下 :

```
class java.util.ArrayList
class java.util.ArrayList
true
```

`clazz1`和`clazz2`是同一种类型`ArrayList`，在运行时我们传入的类型变量`String`、`Integer`都被丢掉了。Java语言泛型在设计的时候为了兼容原来的旧代码，Java的泛型机制使用了“擦除机制”。

```java
public class Main {
    public static void main(String[] args) {
        List<Table> tableList = new ArrayList<>();
        Map<Room,Table> maps = new HashMap<>();
        House<Room> house = new House<>();
        Particle<Long,Double> particle = new Particle<>();
        System.out.println(Arrays.toString(tableList.getClass().getTypeParameters()));
        System.out.println(Arrays.toString(maps.getClass().getTypeParameters()));
        System.out.println(Arrays.toString(house.getClass().getTypeParameters()));
        System.out.println(Arrays.toString(particle.getClass().getTypeParameters()));
    }
}
class Table{}
class Room{}
class House<Q>{}
class Particle<POSITION,MOMENTUM>{}
```

运行结果如下所示 ：

```
[E]
[K, V]
[Q]
[POSITION, MOMENTUM]
```

我们想要获取运行时类的类型参数，但是我们看到运行结果都是「形参」。**在运行期间我们获取不到任何已经声明的类型信息**

<span style="color:red">注意 ：编译器虽然会在编译过程中移除参数的类型信息，但是会保证类或方法内部参数类型的一致性</span>

泛型参数将会被擦除到它的第一个边界（边界可以有多个，重用`extends`关键字，通过它可以给参数类型添加一个边界）。编译器实际上会把类型参数替换为他的第一个边界的类型。如果没有指定参数，那么类型参数将会被擦除到`Object`。下面例子中，可以把泛型参数`T`当成`HasF`使用 ：

```java
public class Manipulator <T extends HasF>{
    T obj;
    public T getObj() {
        return obj;
    }
    public void setObj(T obj) {
        this.obj = obj;
    }
}
interface HasF{
    void f();
}
```

`extends`关键字后后面的类型信息决定了泛型类型能够保留的信息。Java类型擦除只能擦除到`HasF`类型 。

## 🐲 Java泛型约束和局限性

### 不能使用基本类型实例化类型参数

以`Pair`类为例子 ：

```java
public class Pair<T> {
    private T first;
    private T second;
    public Pair(){
        first = null;
        second = null;
    }
    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }
    public T getFirst() {
        return first;
    }
    public void setFirst(T first) {
        this.first = first;
    }
    public T getSecond() {
        return second;
    }
    public void setSecond(T second) {
        this.second = second;
    }
}
```

在实例化此类的时候，需要这样写 ：

```java
Pair<Double> pair = new Pair<>(); // OK
Pair<double> pair = new Pair<>(); // ERROR
```

不能使用`Pair<double>`的直接原因是因为类型擦除，擦除之后，`Pair`类中含有`Object`类型的域，而`Object`不能转换为`double`类型。（或者任何其他基本类型数据）

### 运行时类型查询只适用于原始类型

首先，解释下什么是原始类型 —— 原始类型就是在删除类型参数（擦除）之后的泛型类型名。一般是限定类型，如果没有限定类型，那么就是`Object`。例如 ：`Pair<T>`的原始类型就是`Object`，而`Interval<T extends Comparable>`的原始类型为`Comparable`。

以下代码均不允许运行 ：

```java
if (a instanceof Pair<Stirng>){...} //ERROR
if (a instanceof Pair<T>){...	}
```

`getClass`方法总是返回原始类型 ：

```java
Pair<Double> pair = new Pair<>();
Pair<String> str = new Pair<>();
System.out.println(str.getClass() == pair.getClass()); // true
```

因为两次调用的`getClass()`都将会返回`Pair.class`

### 不能创建参数化类型的数组

例如 ：

```java
Pair<String>[] table = new Pair<String>[10]; //!!ERROR
```

假设编译没有出现错误，那么虚拟机会擦除泛型 ：

```java
Pair<Object>[] table = new Pair<Object>[10];
```

那么，下面赋值 ：

```java
table[0] = new Pair<Integer>();
```

就能够通过数组存储检查了，但是如果读取的时候，会提示类型转换错误。

🍭 但是可以声明通配类型的数组 ：

```java
// 不安全但是IDE没有报错
Pair<String>[] pairs = (Pair<String>[]) new Pair<?>[10];
Object[] objects = pairs;
objects[0] = new Pair<>(1,2);
System.out.println(pairs[0].getFirst());
```

以上代码编译器不会报错，但是运行时会出现错误。

### `Varargs`警告

考虑下面一个简单的方法，这个方法的参数个数是可变的 ：

```java
public final <T> void display(T... products){
    System.out.println(Arrays.toString(products));
}
```

为了调用这个方法，虚拟机内部必须创建一个泛型类型数组，但是，对于这种情况，**只会得到一个警告**

可以使用两种方式消除这种警告 ：

第一种 ——

```java
@SafeVarargs
public final <T> void display(T... products){
    System.out.println(Arrays.toString(products));
}
```

第二种 ——

```java
@SuppressWarnings("unchecked")
public static void main(String[] args) {
    // .....
    var.display(list);
}
```

### 不能实例化类型变量

例如，下面的构造器是非法的 ：

```java
public Pair(){this.first = new T();this.second = new T();} //ERROR
```

有两种解决方法 ：

第一种有一个前提，JDK8+，让调用者提供一个**构造器表达式** ：

```java
Pair<String> pair = Pair.makePair(String::new);
```

```java
public static <T> Pair<T> makePair(Supplier<T> constr){
    return new Pair<T>(constr.get(),constr.get());
}
```

其中，`makePair`方法接受一个`Supplier<T>`，这是一个函数式接口，表示一个无参数而且返回类型为`T`的函数

第二种是比较传统的方式 ：

```java
public static <T> Pair<T> makePair(Class<T> clazz){
    try {
        return new Pair<T>(clazz.newInstance(),clazz.newInstance());
    } catch (InstantiationException | IllegalAccessException e) {
        e.printStackTrace();
        return null;
    }
}
```

然后如下调用 ：

```java
Pair<String> pair = Pair.makePair(String.class);
```

### 不能构造泛型数组

就像不能构造泛型实例一样，也不能构造泛型数组 ：

```java
public static <T extends Comparable> T[] minmax(T... a){T[]mm = new T[2];} // ERROR
```

如果数组仅仅作为一个类的私有实例域，就可以将这个数组声明为`Object[]`，并在获取元素的时候进行类型转换，例如`ArrayList`类 ：

```java
public class ArrayList{
    transient Object[] elementData; // non-private to simplify nested class access
    // ...
    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }
}
```

如果数组是某个方法的临时变量 ：就可以用两种方式 ——

第一种同样需要JDK8+版本 ：

```java
public static <T extends Comparable> T[] minmax(IntFunction<T[]> constr,T... a){
    T[] mm = constr.apply(2);
    // ...
}
```

```java
String[] ss = ArrayAlg.minmax(String[]::new,"Tom","Smith","Harry");
```

另外比较老的方式是利用反射机制 ：

```java
public static <T extends Comparable> T[] minmax(Class<T[]> clazz,T... a){
    T[] mm = Array.newInstance(clazz,2);
    // ...
}
```

### 不能在静态域或方法中引用类型变量

### 不能抛出或捕获泛型类的实例

既不能抛出也不能捕获泛型类对象，甚至泛型类不能够扩展`Throwable`：

```java
public class Problem<T> extends Throwable{...} ///ERROR
```

在`catch`子句中不能使用类型变量 ：

```java
catch(T t){...}	//ERROR
```

但是，在异常规范中使用类型变量是允许的 ：

```java
public static <T extends Throwable> void work() throws T{
    try {
        // ... somework
    }catch (Throwable t){
        throw t;
    }
}
```

### 注意擦除后的冲突

当泛型类型被擦除后，无法创建引发冲突的方法 ：

```java
public class Pair<T>{
    // !!!ERROR
    public boolean equals(T value){
        // ...
    }
}
```

因为和`Object.equals()`引发冲突了

## Java泛型的通配符

### 上界通配符`<? extends T>`

`<T extends BoundingType>`表示T应该是绑定类型的*子类型*。T和绑定类型可以是类，也可以是接口。一个类型变量或通配符可以有多个限定，例如 ：

```java
T extends Comparable & Serializable
```

```java
class Fruit{}
class Apple extends Fruit{}
class Plate<T>{
    T item;
    public Plate(T t){
        this.item = t;
    }
    public T getItem() {
        return item;
    }
    public void setItem(T item) {
        this.item = item;
    }
}
```

下面，我们想定义一个水果盘子 ：

```java
Plant<Fruit> plant = new Plant<Apple>(new Apple());
```

但是，上面的代码会无法编译 ：

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-9-30/99967816.jpg)

从图片可知，就算容器中的类型之间存在继承关系，但是`Plate`和`Plate`两个容器之间是不存在继承关系的，Java可以设计成`<? extends T>`让两个容器之间存在继承关系 ：

```java
Plate<? extends Fruit> plant = new Plate<Apple>(new Apple());
```

看一个更为详细的例子 ：

```java
class Food{}
class Fruit extends Food {}
class Meat extends Food {}
class Apple extends Fruit {}
class Banana extends Fruit {}
class Pork extends Meat{}
class Beef extends Meat{}
class RedApple extends Apple {}
class GreenApple extends Apple {}
```

`Plate<? extends Fruit>`覆盖下面红框中的部分 ：

![1538207510165](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-9-30/50021287.jpg)

但是如果我们想要向盘子里添加数据，例如 ：

```java
plate.setItem(new Fruit());
p.set(new Apple());
```

你就会发现编译器报错。`<? extends Fruit>`会使往盘子里放东西的`set()`方法失效，但是取东西的`get()`方法还有效。

因为 ：

```java
Plate<? extends Fruit> plant
```

可能会有多种含义 ：

```java
Plate<? extends Fruit> plant = new Plate<Fruit>();
Plate<? extends Fruit> plant = new Plate<RedApple>();
Plate<? extends Fruit> plant = new Plate<Banana>();
```

- 当我们尝试set一个`Fruit`时，`plant`可能指向`new Plate<Banana>`
- 当我们尝试set一个`Banana`时，`plant`可能指向`new Plate<Fruit>`
- ……

所以对于实现了`<? extends T>`的类只可以向外提供元素

但是上界通配符是允许读取操作的：

```java
Fruit fruit = plate.getItem();
System.out.println(fruit);
```

### 下界通配符`<? super T>`

`Plate<? super Fruit>`覆盖红色框部分 ：

![1538209506061](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-9-30/79818120.jpg)

下图代码就会编译出错 ：

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-9-30/69673043.jpg)

因为`Apple`是`Fruit`的子类。

但是这样做是正确的 ：

```java
Plate<? super Fruit> plate = new Plate(new Apple());
```

可以在`plate`中存入数据 ：

```java
plate.setItem(new Apple());
plate.setItem(new Banana());
// ! plate.setItem(new Meat());
```

但是读取的话，只能是`Object`类型 ，因为 ：

下界通配符规定了元素的最小粒度，必须是`T`以及其基类，向其中存储`	T`和派生类都是可以的，因为它们都可以隐式转换为`T`类型。但是向外读取就不好控制了，里面存储的都是`T`和其派生类，无法转换为任何一种类型，只有`Object`才能装下。

### `PECS`原则

- 上界`<? extends T>`只能向外获取，但是不能向内存储，适合频繁向外面读取内容的场景。
- 下界`<? super T>`不影响向内存储，但是向外取的时候只能放`Object`对象，适合经常向里面插入数据的场景

### `<?>`无限通配符

`Pair<?>`类型有以下方法 ：

```java
? getFirst();
void setFirst(? first);
```

其中，`getFirst`的返回值只能赋值给`Object`，而`setFirst`方法不能被调用，甚至不能用`Object`调用 。

`Pair<?>`和`Pair`的本质区别在于 —— `Pair`可以用任意`Object`对象调用`setObject`方法 。

## 其他

### 桥方法

假设有这样一个类 ：

```java
public class Pair<T> {
    private T first;
    private T second;
    public Pair(){
        first = null;
        second = null;
    }
    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }
    public T getFirst() {
        return first;
    }
    public void setFirst(T first) {
        this.first = first;
    }
    public T getSecond() {
        return second;
    }
    public void setSecond(T second) {
        this.second = second;
    }
}
```

还有它的一个子类 ：

```java
class DateInterval extends Pair<LocalDate>{
    @Override
    public void setSecond(LocalDate second) {
        if (second.compareTo(getFirst()) > 0){
            super.setSecond(second);
        }
    }
}
```

我们知道，虚拟机在运行期间会擦除类型参数，会替换为他们的限定类型；如果没有限定类型，那么就用`Object`：

那么，`Pair<T>`在虚拟机中是如下所示的 ：

```java
public class Pair{

    private Object first;
    private Object second;

    public Pair(){
        first = null;
        second = null;
    }

    public Pair(Object first, Object second) {
        this.first = first;
        this.second = second;
    }

    public Object getFirst() {
        return first;
    }

    public void setFirst(Object first) {
        this.first = first;
    }

    public Object getSecond() {
        return second;
    }

    public void setSecond(Object second) {
        this.second = second;
    }
}
```

`DateInterval`泛型擦除之后如下所示 ：

```java
class DateInterval extends Pair{
    @Override
    public void setSecond(LocalDate second) {
        if (second.compareTo(getFirst()) > 0){
            super.setSecond(second);
        }
    }
}
```

但是，`Pair`中的`setSecond`的方法显然是这样的 ：

```java
public void setSecond(Object second){}
```

类型擦除和`@Override`之间发生了冲突，要解决这个问题，编译器在`DateInterval`中生成了一个 **桥方法** ：

```java
public void setSecond(Object second){
    setSecond((LocalDate)setSecond);
}
```

字节码如下所示 ：

```java
public synthetic bridge setSecond(Ljava/lang/Object;)V
   L0
    LINENUMBER 35 L0
    ALOAD 0
    ALOAD 1
    CHECKCAST java/time/LocalDate
    INVOKEVIRTUAL DateInterval.setSecond (Ljava/time/LocalDate;)V
    RETURN
   L1
    LOCALVARIABLE this LDateInterval; L0 L1 0
    MAXSTACK = 2
    MAXLOCALS = 2
```

假设在`DateInterval`中覆盖了`getSecond()`方法 ：

```java
@Override
public LocalDate getSecond() {
    return super.getSecond();
}
```

那么在`DateInterval`中就有两个`getSecond`方法了 ：

```java
LocalDate getSecond();
Object getSecond();
```

在Java中，这样编写代码是不合法的；但是在虚拟机中，用参数类型和返回类型确定一个方法 。

总之，桥方法被合成来保持多态。

## 参考及引用

- [深入理解Java泛型](https://juejin.im/post/5b614848e51d45355d51f792#heading-7)
- 『Java核心技术 卷一』

