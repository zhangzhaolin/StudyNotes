# 自己的Java笔记 —— 第一篇 HashMap（二）

## 链接
上一节 ：[简书](https://www.jianshu.com/p/8244a21bd2b4)

## 2.3 tableSizeFor函数
先来看tableSizeFor函数的构成 :

```
	/**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

我把这些代码粘贴复制让它们运行了起来 结果不可思议

```
    private static final int MAXIMUM_CAPACITY = 1 << 30;

    public static void main(String[] args) {
        int cap = 18;
        int n = cap - 1; // 17
        System.out.println("n |= n >>> 1 --> " + (n |= n >>> 1));
        System.out.println("n |= n >>> 2 --> " + (n |= n >>> 2));
        System.out.println("n |= n >>> 4 --> " + (n |= n >>> 4));
        System.out.println("n |= n >>> 8 --> " + (n |= n >>> 8));
        System.out.println("n |= n >>> 16 --> " + (n |= n >>> 16));
        int result = (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
        System.out.println(result);
    }
```

结果打印为32 那这些代码是怎样将18转换成为了一个比他大有离他最近的一个二次幂整数呢？另外，右移难道不是越右移越小才对嘛？


```
0000 0000 0001 0001 (17)
0000 0000 0000 1000 (17 >>> 1)
-----------------------
0000 0000 0001 1001
0000 0000 0000 0110 ( >>> 2)
-----------------------
0000 0000 0001 1111
0000 0000 0000 0001 ( >>> 4)
-----------------------
0000 0000 0001 1111
.... .... .... .... ( >>> 8)
-----------------------
0000 0000 0001 1111
.... .... .... .... ....
```

我们发现当到运行至第二行后，就已经定下了解决，不管之后再怎样做或运算，结果会成为`0001 1111`，最会在加一就会成为`0010 0000`也就是32

那若果`n = 01000 0000 0000 0000 0000 0000 0000 0000`呢？也就是说 `cap = (1 >> 30) + 1`，虽然在构造函数中有这样一句话 ：

```
	public HashMap(int initialCapacity, float loadFactor) {
        // ...
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
		// ...
    }
```

但是为什么要这样？

```
0100 0000 0000 0000 || 0000 0000 0000 0000
0010 0000 0000 0000 || 0000 0000 0000 0000 ( >>> 1)
---------------------------------------------------
0110 0000 0000 0000 || 0000 0000 0000 0000
0001 1000 0000 0000 || 0000 0000 0000 0000 ( >>> 2)
---------------------------------------------------
0111 1000 0000 0000 || 0000 0000 0000 0000
0000 0111 1000 0000 || 0000 0000 0000 0000 ( >>> 4)
---------------------------------------------------
0111 1111 1000 0000 || 0000 0000 0000 0000
0000 0000 0111 1111 || 1000 0000 0000 0000 ( >>> 8)
---------------------------------------------------
0111 1111 1111 1111 || 1000 0000 0000 0000
0000 0000 0000 0000 || 0111 1111 1111 1111 ( >>> 16)
---------------------------------------------------
0111 1111 1111 1111 || 1111 1111 1111 1111
```

```
int result = (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
```

如果没有判断的话 n+1就成了负数。所以说 1 >>> 30 是int类型中最大的二次幂整数，如果在构造函数中声名了比他还大的一个数并且没有任何约束条件的话，最后会变成一个悲剧（也就会成为负数）。

通过上面两个异或操作 不难看出`tableSizeFor()`函数就是把二进制中的0变成1，当然不是数字前面的0，而是数字后面的0。

## 2.4 数组的初始化

我们看数组的初始化其实是在做`put()`中进行的(put函数直接调用了`putVal()`)：

```
	transient Node<K,V>[] table;
    // ...
	public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    // ...
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            // 数组的初始化
            n = (tab = resize()).length;
        }
        // ...

    }
```

其实`putVal()`是一个很长的函数，但是数组的初始化只有这几行。

具体再来看`resize()`函数是怎样初始化的 :

```
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        // ...
        else if (oldThr > 0)
	      // 如果你定义了初始容量就走这里了
          newCap = oldThr;
        else {               
        	// 无参构造函数就会到这里进行初始化
			newCap = DEFAULT_INITIAL_CAPACITY;
			newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            // 在这里计算阈值
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
        	// ... 一段昂长的代码    
        }
        return newTab;
    }
```

## 2.5 HashMap的put操作

下面代码才是`putVal()`的原身 ：

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 对数组进行初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 接下来我们要分析以下代码
        // ------>
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

先看这两行 :

```
	public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    // ...
	final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        // ...
        // 如果数组有位置 就把节点放在这个位置上
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        // ...
	}
```

如何判断某个元素在哪个位置呢？

> p = tab[i = (n-1) & hash]

判断元素在数组的哪个位置位置其实就是 `(length - 1) & hash`

那么 `hash`又是怎么来的？

```
	static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

当key时null的时候，hash值就是0；如果不是null，就计算key的hashCode()，并把结果右移16位，然后把两个做异或处理。这就是最后的hash()值

那为何要这么做？知乎某位大佬给出了如下解释 ：

[JDK 源码中 HashMap 的 hash 方法原理是什么？](JDK 源码中 HashMap 的 hash 方法原理是什么？ - 知乎
https://www.zhihu.com/question/20733617/answer/32513376)

就是说，在确定数组的索引位置时候，如果直接计算hash(key)的话，代价是比较大的。因为hash(key)的取值范围是整个int范围，而HashMap默认的数组容量只有16。所以我们可以将hash(key)做取模处理，也就是 `hash(key) & (length - 1)`。但是，这些又会引发很多的”碰撞”（这个结果很可能将很多的节点放到0号位置）。

```
 0000 0000 0000 0000 || 0000 0000 0000 1111
&0... .... .... .... || .... .... .... 0000
--------------------------------------------
 0000 0000 0000 0000 || 0000 0000 0000 0000
```

我们发现，不管...是0还是1，只要最后四位是0的话，结果都会成为0，这就会造成很多碰撞

根据这个情况，扰动函数出来了：

![扰动函数](https://pic3.zhimg.com/80/4acf898694b8fb53498542dc0c5f765a_hd.png)

将hash()结果的高半部分和低半部分做异或处理，这样就很大程度上减少了碰撞。

好了，知道了元素存放到数组哪个索引下面了，代码就开始判断这个索引是不是空的，如果是空的，元素就是头头了。

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-22/62743503.jpg)

假设某个节点要进行put操作，这个节点的

value="老古董",key="CN-Z17-18-00139",index(hash)=7

然后它就开始判断table[7]是不是有元素占着，如果没有，他就成为了第一张"唱片"（老古董是寻宝游戏的第一张单曲）

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-22/49074801.jpg)

我们再回头看put函数的其他情况 ：

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 数组初始化
        // ...
        // 如果当前索引是空的 占位
        // ...
        // 如果当前有位置 走这里
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // ...
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        // ...
}

```

先来看这一段代码 ：

```
Node < K, V > e; K k;
// ...
// 如果待插入的节点和待在数组上的节点重了 就做替换
if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))) e = p;
// ...
if (e != null) { // existing mapping for key
    V oldValue = e.value;
    if (!onlyIfAbsent || oldValue == null)
			e.value = value;
		// 目前没有意义的一句话
		afterNodeAccess(e);
    return oldValue;
}
```

所以什么时候HashMap会进行节点的替换呢？必须满足以下条件：

- 待插入的和将要被替换的节点必须hash值相等。当然这里的hash值并不是简单的hash(key)，而是经过扰动算法“摧残”过的hash值。
- 待插入的节点的key必须要等于将要被替换的key || 待插入的节点的key不能是null并且待插入和代替换的key的`equals()`要为true
