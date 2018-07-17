# 自己的Java笔记——第一篇 HashMap
## 前言
首先，这是自己的开的不知道多少个坑的笔记了（汗颜），这些笔记都是自己在Java这条道路上的**初学阶段**所思所想，并没有多少含金量～

想第一篇记一记HashMap，没有为什么，任性。

首先，HashMap的基本方法应该都知道了，`put(key,value)`以及`get(key)`，我前几天在笔试的时候就遇到了一道统计各个英文单词数量的，就用到了HashMap（ 应该还有更简单的方法，以后慢慢来吧 ^ _ ^ ）

```
String words = "Synchronized";
// 先把大写的字符转化成为小写的
words = words.toLowerCase();
Map<Character,Integer> result = new HashMap<>();
for(char word:words.toCharArray()) {
if(result.containsKey(word)) {
	result.put(word, result.get(word)+1);
}else {
	result.put(word, 1);
}
}
System.out.println(result.toString());
```

另外值得注意的一个基础问题是HashMap的key、value都不能是基本数据类型（即不能为byte short int long char float double boolean）,但是很特别的一点是key和value竟然可以是null

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
这个规律不知道是哪位大神发现的啊~反正记住就对了

- 0异或A结果都为A ： 0 ^ 1 = 1   0 ^ 0 = 0
- 1异或A结果都为A的相反数 ： 1 ^ 1 = 0  1 ^ 0 = 0
- A异或A结果都为0 （这个比较好理解吧 相同位置下数字相同位0）

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

接着 是 : 

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

总的来说，HashMap的结构是：数组+链表+红黑树 （PS : 我用的是10版本的）

![HashMap中链表的节点](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-17/68941977.jpg)

![节点组成的数组](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-17/67744525.jpg)

![红黑二叉树](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-17/1457116.jpg)

目前，现将红黑二叉树看成是一个不一般的二叉树吧（我能说现阶段不知道什么是红黑二叉树嘛XD）。以上三张图分别就是组成HashMap的三种数据结构了。table就是的数组（这个数组用来存放链表以及红黑树的第一个元素的地址），我们将数组中的各个元素叫做HashMap的桶，称

然后，再来看一下几个HashMap中几个重要的字段 : 

```
	/**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    /**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;
    /**
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;
```

一个一个的来分析 

首先是第一个 `DEFAULT_INITIAL_CAPACITY` 代表的是HashMap默认初始数组长度为16