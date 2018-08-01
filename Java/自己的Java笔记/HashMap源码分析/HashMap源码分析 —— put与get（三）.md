# HashMap源码分析 —— put与get（三）
## 链接
上一节 ： [自己的Java笔记 —— 第一篇 HashMap（二）](https://github.com/zhangzhaolin/StudyNotes/blob/master/Java/%E8%87%AA%E5%B7%B1%E7%9A%84Java%E7%AC%94%E8%AE%B0/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E2%80%94%E2%80%94%20put%E4%B8%8Eget%EF%BC%88%E4%BA%8C%EF%BC%89.md)
## 2 HashMap的put函数
### 2.5.2 回顾

我们在上两节主要说了以下内容：

- HashMap的存储结构
- HashMap的构造函数（一共有四个我们说了三个）
- HashMap中的几个字段 ： 默认初始容量、阈值、加载因子、最大容量
- HashMap中的几个方法 ： 扰动函数
- `put`函数中如何确定节点的索引 ： (length - 1) & hash
- 初始化数组
- 如何判断key值相等 ： 两个必不可少的条件

主要是`putVal()`函数说了下面的几个东西

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 数组为空的时候初始化数组
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 确定节点所要存储的数组索引，如果当前位置为空，直接占座
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
        	  // 当前数组索引不为空
            Node<K,V> e; K k;
            // 判断key是否相等 如果相等 直接替换
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // ...... 还没有要开始的知识点
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // ...... 还没有要开始的知识点
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
            // 替换
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 这里还没有说
        if (++size > threshold)
            resize();
        // 一个目前没有任何用处的函数
        afterNodeInsertion(evict);
        return null;
    }
```

### 2.5.3 当是一个链表的时候

我们先来看这里 ：

```
p = tab[i = (n - 1) & hash]
        // ...
        Node<K,V> e; K k;
        if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 当p是一个红黑树的时候
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 当p是一个链表的时候
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
```

在这之前，我们需要先知道，当某个节点下面的链表足够多时，需要把链表转成红黑树的形式，所以，头结点可能会是一个红黑树类型的，也可能是一个链表类型的 目前只说当p是一个链表的时候。
(可能说完之后就开始学红黑树了，反正需要学习的东西还很多，努力吧骚年)

```
else {
	for (int binCount = 0; ; ++binCount) {
 		if ((e = p.next) == null) {
 		   // 把新的节点放到链表的尾部
 			p.next = newNode(hash, key, value, null);
        	// 如果链表节点大于等于8 （别忘记binCount是从0计数的）
        	if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
          	// 将链表转换成红黑树
          	treeifyBin(tab, hash);
          break;
       }
       // 如果在遍历链表的过程中发现了key值重复的节点 进行替换
    	if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
	      break;
        p = e;
   }
}
```

上边的代码比较容易理解，就是一个遍历链表的过程 ： 插入到链表的尾部，如果遇到key相同的替换并跳出遍历，如果链表长度大于等于8，将链表转成红黑树

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        // ...
        if (++size > threshold)
            resize();
        // 目前没有卵用的代码
        afterNodeInsertion(evict);
        return null;
    }
```

我们发现，每当put一个节点时候，size都会做++操作，然后判断是否大于阈值，如果大于阈值，就执行`resize()`，不难推测出，`resize()`不仅可以做数组初始化操作，还可以进行数组的扩容，我们来看数组是怎么进行扩容的。

## 2.6 resize()扩容操作

```
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

先来看上面的一段代码 ：

```
		Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        // 如果旧数组不为空
        if (oldCap > 0) {
            // 如果旧的数组容量已经超过了最大容量值，直接将阈值变成最大值，以后都不会扩容了
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 新数组的容量 = 旧的数组容量 * 2
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                // 新的阈值 = 旧的阈值 * 2
                newThr = oldThr << 1; // double threshold
        }
        // 初始化数组走这里
        // 如果构造函数定义了数组初始容量
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        // 如果构造函数没有定义初始容量
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 构造函数中定义了初始容量 在这里计算阈值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        // 阈值（再也不用于存储数组容量了）
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        // 新数组 （可用于数组初始化，也可用于扩容）
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;

```

但是如果扩容只创建一个新的数组也不行啊，看接下来怎么把“钉子户”挪到新的地方去：

```
        if (oldTab != null) {
            // 遍历旧的数组 看有没有钉子户
            for (int j = 0; j < oldCap; ++j) {
                // 用一个e来表示数组上的节点
                Node<K,V> e;
                // 先来判断一下数组上有没有节点 只要不为空
                if ((e = oldTab[j]) != null) {
                	   // 只要不为空 先把头结点制空
                    oldTab[j] = null;
                    // 如果下面没有节点了 只有他孤身一人
                    if (e.next == null)
                        // 重新计算索引值
                        newTab[e.hash & (newCap - 1)] = e;
                    // 如果你是一个红黑树的头结点
                    else if (e instanceof TreeNode)
                    	// 看起来是红黑树的一些操作
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    // 如果你是一个链表 （长度不要大于8）
                    // 你就应该进行以下操作
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
```

扩容时，如果是一个链表，将会进行以下操作 ：

```
Node<K,V> loHead = null, loTail = null;
Node<K,V> hiHead = null, hiTail = null;
Node<K,V> next;
do {
    next = e.next;
    if ((e.hash & oldCap) == 0) {
        if (loTail == null)
            loHead = e;
        else
            loTail.next = e;
        loTail = e;
    }
    else {
        if (hiTail == null)
            hiHead = e;
        else
            hiTail.next = e;
        hiTail = e;
    }
} while ((e = next) != null);
if (loTail != null) {
    loTail.next = null;
    newTab[j] = loHead;
}
if (hiTail != null) {
    hiTail.next = null;
    newTab[j + oldCap] = hiHead;
}
```

我们先来看这一行 ：

```
if ((e.hash & oldCap) == 0) {
    // ...
}
```

我们好像在前面见过类似的 是重新计算索引的 ：

```
e.hash & (newCap - 1)
```

那么`e.hash & oldCap`是做什么的呢？

我们先做一个示例 : oldCap = 32 newCap = 64

下面是重新计算索引的示例 ：

```
newCap = 64 newCap - 1 = 63

  0000 0000 0001 1111 oldCap - 1 旧索引值
  0000 0000 0011 1111 newCap - 1 新索引值
& 0010 1010 01?0 0110 hash(key)
```
我们发现，某个节点是否需要挪动位置，完全取决于?的位置 当?为0的时候 不需要挪动位置，当?为1的时候，需要挪动位置。

然后，HashMap的开发人员发现了更为高级的判断方式 :

```
  0000 0000 0010 0000 oldCap
& 0101 1101 01?0 1010 hash(key)
```
无论hash值是多少，如果?为0，那oldCap&hash结果就为0,如果?为1，那么oldCap&hash的结果就不是0，所以，在判断某个节点是否需要挪动位置的时候，oldCap&hash和(newCap-1)&hash的效果是一样的。

美团的博客论坛中有更为详细的解释 : [Java 8系列之重新认识HashMap
](https://tech.meituan.com/java_hashmap.html)

> 经过观测可以发现，我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。看下图可以明白这句话的意思，n为table的长度，图（a）表示扩容前的key1和key2两种key确定索引位置的示例，图（b）表示扩容后key1和key2两种key确定索引位置的示例，其中hash1是key1对应的哈希与高位运算结果。

![](https://tech.meituan.com/img/java-hashmap/hashMap%201.8%20%E5%93%88%E5%B8%8C%E7%AE%97%E6%B3%95%E4%BE%8B%E5%9B%BE1.png)

> 元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

![哈希算法例图](https://tech.meituan.com/img/java-hashmap/hashMap%201.8%20%E5%93%88%E5%B8%8C%E7%AE%97%E6%B3%95%E4%BE%8B%E5%9B%BE2.png)

综上所述，oldCap & hash可以判断元素是否需要重新移动位置。

下一篇 ：[HashMap源码分析-put和get(四)](https://github.com/zhangzhaolin/StudyNotes/blob/master/Java/%E8%87%AA%E5%B7%B1%E7%9A%84Java%E7%AC%94%E8%AE%B0/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E2%80%94%E2%80%94%20put%E4%B8%8Eget%EF%BC%88%E5%9B%9B%EF%BC%89.md)
