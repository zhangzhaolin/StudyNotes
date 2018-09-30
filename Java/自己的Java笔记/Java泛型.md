# Java泛型

## Java泛型介绍

- Java泛型类
- Java泛型方法
- Java泛型接口
- Java泛型擦除及其他相关内容
- Java泛型通配符

## Java泛型类

`ArrayList`大概是一个非常常见的一个泛型类了，在代码中指定其传入的类型为`<String>`类型，所以只能添加`String`类型的数据，而如果添加其他类型的数据则会报错。

![1538190250627](C:\Users\zhang\AppData\Roaming\Typora\typora-user-images\1538190250627.png)

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

先看下面的例子 ：

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

### Java泛型擦除缺陷及补救措施

泛型类型不可以显式的运用在运行时类型的操作当中。例如：转型、`instanceof`和`new`，因为在运行时，所有参数的类型信息都丢失了，类似如下代码是无法通过编译的 ：

```java
public class Erased<T> {
    private static final int SIZE = 100;
    public static void f(Object arg){
        // ! if(arg instanceof T){}
        // ! T var = new T();
        // ! T[] array = new T[SIZE];
        // ! T[] array = (T)new Object[SIZE];
    }
}
```

### 类型判断问题

我们可以使用如下代码解决泛型的类型信息由于擦除无法判断问题 ：

```java
public class GenericType<T> {
    Class<?> classType;
    public GenericType(Class<?> type){
        classType = type;
    }
    public boolean isInstance(Object object){
        return classType.isInstance(object);
    }
    public static void main(String[] args) {
        GenericType<A> generator = new GenericType<>(A.class);
        System.out.println(generator.isInstance(new A()));
        System.out.println(generator.isInstance(new B()));
    }
}
class A{}
class B{}
```

### 创建类型实例

泛型中不能使用`new T`的原因有两个，第一个是不知道`T`的类型；第二是因为不知道`T`是否包含无参构造函数，我们可以使用显示的工厂模式 ：

```java
public class Main {
    public static void main(String[] args) {
        Creater<Integer> creater = new Creater<>();
        System.out.println(creater.newInstance(new IntegerFactory()));
    }
}
interface Factory<T>{
    T create();
}
class Creater<T>{
    T instance;
    public <F extends Factory<T>> T newInstance(F f){
        instance = f.create();
        return instance;
    }

}
class IntegerFactory implements Factory<Integer>{
    @Override
    public Integer create() {
        return 9;
    }
}
```

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

![1538206459450](C:\Users\zhang\AppData\Roaming\Typora\typora-user-images\1538206459450.png)

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

![1538207510165](C:\Users\zhang\AppData\Roaming\Typora\typora-user-images\1538207510165.png)

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

![1538209506061](C:\Users\zhang\AppData\Roaming\Typora\typora-user-images\1538209506061.png)

下图代码就会编译出错 ：

![1538210584784](C:\Users\zhang\AppData\Roaming\Typora\typora-user-images\1538210584784.png)

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



## 参考及引用

- [深入理解Java泛型](https://juejin.im/post/5b614848e51d45355d51f792#heading-7)
- 『Java核心技术 卷一』

