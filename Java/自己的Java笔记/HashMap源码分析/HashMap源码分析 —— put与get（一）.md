# HashMap源码分析 —— put与get（一）
## 前言
首先，这是自己的开的不知道多少个坑的笔记了（汗颜），这些笔记都是自己在Java这条道路上的**初学阶段**所思所想，并没有多少含金量～

想第一篇记一记HashMap，没有为什么，任性。

首先，HashMap的基本方法应该都知道了，`put(key,value)`以及`get(key)`。
另外值得注意的一个基础问题是HashMap的key、value都不能是基本数据类型（即不能为byte short int long char float double boolean）,但是很特别的一点是key和value可以为null

## 1. 慢着 开始之前复习下异或和右移运算符
### 1.1 异或的基本运算
异或是二元按位运算符的一种，最基本的概念是在**当前位**下，两个二进制相同则为0，不同则为1

```
public static void main(String []args){
	int i = -5;
	int j = 5;
	System.out.println("i : " + Integer.toBinaryString(i));
	System.out.println("j : " + Integer.toBinaryString(j));
	System.out.println("i ^ j : " + Integer.toBinaryString(i ^ j));
	System.out.println("i ^ j : " + (i ^ j));
	/**
	*  output :
	*  i : 11111111111111111111111111111011
	*  j : 101
	*  i ^ j : 11111111111111111111111111111110
	*  i ^ j : -2
	*/
}
```

### 1.2 异或规律

- 0异或A结果都为A ： 0 ^ 1 = 1   0 ^ 0 = 0
- 1异或A结果都为A的相反数 ： 1 ^ 1 = 0  1 ^ 0 = 0
- A异或A结果都为0

### 1.3 异或进阶

关于异或进阶不再赘述，大家可以观看[维基百科中的异或讲解](https://en.wikipedia.org/wiki/Exclusive_or)

值得一提的是面试经常会考到：如何不使用第三个参数来交换a和b的值，这时候就可以使用异或。

```
int a = 10;
int b = 41;
a = b ^ a;
b = b ^ a; // b = b ^ ( b ^ a) = a
a = a ^ b; // a = ( b ^ a ) ^ a = b
```

### 1.4 左移和右移运算符和它们的规律

左移和右移运算符是奇怪的运算操作符，我刚开始学的时候觉得不就是移位嘛，移位就是了，后来看《Java编程思想》的时候才知道原来左移和右移原来有这样那样的限制……

先从最简单的开始说，左移就是低位补0，右移就是高位补符号位，无符号右移运算符就是高位补0（这里不再赘述）

接着 在一般情况下 左移和右移都有下面的规律 :

- a << b = a * (2 ^ b)
- a >> b = a / (2 ^ b)

例如 :

```
public static void main(String []args){
    int i = 5;
    int j = 32;
    int m = 40;
    System.out.println(3 << i);
    System.out.println(3 << j);
    System.out.println(3 << m);
    /**
    * output :
    * 96
    * 3
    * 768
    */
}
```

大家可能发现了，有些结果好像与公式并不符合，这是因为int类型的“限制”

众所周知，在Java中int占四个字节，也就是32位，在做移位操作时，无论是左移还是右移，右操作运算符都不能大于或者等于32（当然，我说的是左边的数值是int类型时），因为一旦超过了32，左边的数值完全变了个样。就拿1 << 32而言：

![1 << 32](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-15/14577629.jpg)

我们发现1 << 32如果按照正常计算的话就会变成0，更别提3 << 32，5 << 91了，如果按照正常流程来的话，只要右操作数大于32，结果都变成了0，当然，Java不可能这样子计算，它采取了一种聪明的操作方式，至于为什么，目前不在讨论范围只内。另一种状况是，如果恰巧将1移到了符号位上，即便移位之前是正的数字，也会变成负的，就像1 << 31 = - 2147483648 那样。所以，在上边说的公式有诸多的限制。

这种聪明的方式就是，在左操作数类型为int时，当右侧数值大于等于32，都可以对右操作数取余操作（ % 32），然后再做移位运算，例如：

3 << 40 = 3 << (40 % 32) = 3 << 8 = 3 * (2 ^ 8) = 768

以上都说的是int类型下的移位操作，如果左边操作数为char、byte、short类型都会被转换成int类型处理，当左操作数是long的时候，右边操作数的最大限制变成了64。


## 2. HashMap

### 2.1 从HashMap结构说起

HashMap结构如下图所示:

![HashMap结构](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-17/75072668.jpg)

总的来说，HashMap的结构是：数组+链表+红黑树 （PS : 我用的是8版本的）

![HashMap中链表的节点](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-17/68941977.jpg)

![节点组成的数组](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-17/67744525.jpg)

![红黑二叉树](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-17/1457116.jpg)

目前，现将红黑二叉树看成是一个不一般的二叉树吧（我能说现阶段不知道什么是红黑二叉树嘛XD）。以上三张图分别就是组成HashMap的三种数据结构了。

然后，再来看一下几个HashMap中几个重要的字段 :

```
/**
* 默认的数组容量(也就是数组的长度) —— 数字必须是二次幂
*/
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
/**
* 数组的最大容量
*/
static final int MAXIMUM_CAPACITY = 1 << 30;
/**
* 当使用无参构造函数时 默认的加载因子
*/
static final float DEFAULT_LOAD_FACTOR = 0.75f;
/**
 * 当某个数组下的链表长度超过8时 链表将会转换为红黑树
*/
static final int TREEIFY_THRESHOLD = 8;
/**
* 当节点的数量超过threshold时会发生扩容 此外 当尚未初始化数组的时候，这个字段也会存放初始数组的容量，如果是0的话表示容量是默认的数组容量(16)
*/
int threshold;
```

什么是加载因子？我搜索了百度，然后一脸懵逼，它是这样介绍的：

> 加载因子是表示Hsah表中元素的填满的程度。

一脸懵逼，两脸茫然，然后，在进行相关搜索，发现这个比较靠谱

> threshold(阈值) = (int)加载因子 * 数组容量

还是有点懵逼！！！意思就是说当 节点数量 > 数组容量 * 加载因子时，数组会扩容.

我们来验证一下吧！！

## 2.2 HashMap的构造函数

我们通过idea发现HashMap有四种构造函数，先看前三种（毕竟最后一种不是重点）：

```
	public HashMap() {
		this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
	}
	public HashMap(int initialCapacity) {
		this(initialCapacity, DEFAULT_LOAD_FACTOR);
	}
	public HashMap(int initialCapacity, float loadFactor) {
		if (initialCapacity < 0)
			throw new IllegalArgumentException("Illegal initial capacity: " +
					initialCapacity);
		if (initialCapacity > MAXIMUM_CAPACITY)
			initialCapacity = MAXIMUM_CAPACITY;
		if (loadFactor <= 0 || Float.isNaN(loadFactor))
			throw new IllegalArgumentException("Illegal load factor: " +
					loadFactor);
		this.loadFactor = loadFactor;
		this.threshold = tableSizeFor(initialCapacity);
	}
```

- 无参构造函数 ： 设置了加载因子为默认加载因子(0.75f)

  PS 再来回顾下加载因子是干嘛的：加载因子是扩容用的 什么时候该扩容呢？就是节点个数大于加载因子✖️数组长度的时候就该扩容了

- 一个参数的构造函数 ： 设置了容量初始容量，然后在源码里面直接this交给了第三个构造参数处理
- 两个参数的构造函数 ： 设置了初始容量以及加载因子，我发现在这个构造函数中好像并没有关于数组初始容量的更多限制啊，刚才源码中不是说初始容量必须是二次幂吗？我传一个13，19，在这个构造参数中也没抛出异常啊!!不急，我们看最后一句 :

> this.threshold = tableSizeFor(initialCapacity);

我们传入的初始容量13，18，41……这些“不规范的”（也就是不是二次幂的）数字传给了`tableSizeFor`这个函数，最终返回给了threshold这个变量中（别忘了，这个变量除了可以记录“扩张阈值”，在数组还没有初始化的时候还可以存放数组的自定义容量，这些都是注释中所标注的东西）。来领略一下`tableSizeFor`的真正魅力，这个函数可以 ：

**把任何一个int类型的、大于0的数字转成一个与之相近并大于这个数字的二次幂整数**

听上去有点绕口，简单点说15进去了成了16，17进去了出来摇身一变成了32（比17大的最小二次幂整数就是32了），非常神奇的一个函数，第二小篇接着看这个函数。


下一小节 ： [自己的Java笔记 —— 第一篇 HashMap（二）](https://github.com/zhangzhaolin/StudyNotes/blob/master/Java/%E8%87%AA%E5%B7%B1%E7%9A%84Java%E7%AC%94%E8%AE%B0/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E2%80%94%E2%80%94%20put%E4%B8%8Eget%EF%BC%88%E4%BA%8C%EF%BC%89.md)
