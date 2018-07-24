# 自己的Java笔记 —— 第一篇 HashMap（三）
## 链接
上一节 ： [自己的Java笔记 —— 第一篇 HashMap（二](https://www.jianshu.com/p/97bd35ec4f8d)
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

