# HashMap源码分析 —— put与get（四）

## 链接

上一节 : [自己的Java笔记 —— 第一篇 HashMap（三）](https://github.com/zhangzhaolin/StudyNotes/blob/master/Java/%E8%87%AA%E5%B7%B1%E7%9A%84Java%E7%AC%94%E8%AE%B0/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E2%80%94%E2%80%94%20put%E4%B8%8Eget%EF%BC%88%E4%B8%89%EF%BC%89.md)


## 2.6 resize()扩容操作

```
    Node<K,V> loHead = null, loTail = null;
    Node<K,V> hiHead = null, hiTail = null;
    Node<K,V> next;
    do {
        next = e.next;
        // 还是原来的索引值
        if ((e.hash & oldCap) == 0) {
            if (loTail == null)
                loHead = e;
            else
                loTail.next = e;
            loTail = e;
        }
        // 不是原来的索引值了 需要迁移
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

不难看出，`loHead`和`loTail`两个节点分别记录不需要移动的链表的头部和尾部，`hiHead`和`hiTail`分别记录需要移动的链表头部和尾部.

假设在扩容的时候某个数组下有这样一个链表 :

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-25/28481902.jpg)

其中，假设天蓝色部分的不需要挪动，红色部分的需要挪动

第一步 : 建立`loHead` `loTail` `hiHead` `hiTail`四个节点

第二步 ：

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-25/48045779.jpg)

第三步 :

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-25/14539380.jpg)

...

第N步 :

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-25/66532230.jpg)

最后一步 :

把以`loHead`为首的链表放到数组的原位置，把以`hiHead`为首的链表放到`原位置+oldCap`的位置，这就是链表的扩容操作

下面图片是美团技术团队的put函数的流程总结 :

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-27/76780290.jpg)

## 2.7 脚步加快 —— get()函数

相对于put()函数 get()函数要简单的多 ：

```
	public V get(Object key) {
		Node<K,V> e;
		return (e = getNode(hash(key), key)) == null ? null : e.value;
	}
	final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

首先看下条件 :

```
// 数组不能是空的 数组的长度必须要大于0 数组当前索引有节点存在
if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
	// ...           
}
```

然后，如果第一个节点正好就是要找的，那就直接返回吧 :

```
if (first.hash == hash && // always check first node
	((k = first.key) == key || (key != null && key.equals(k))))
	return first;
```

```
if ((e = first.next) != null) {
	// 如果是红黑树
	if (first instanceof TreeNode)
    	return ((TreeNode<K,V>)first).getTreeNode(hash, key);
   // 如果是链表 就循环 直到最后一个节点
   do {
       if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
	       return e;
    } while ((e = e.next) != null);

}
```

## 2.8 HashMap的最后一个构造函数

最后一个构造函数如下 ：

```
public HashMap(Map<? extends K, ? extends V> m) {
   // 加载因子直接设置成默认的
   this.loadFactor = DEFAULT_LOAD_FACTOR;
   putMapEntries(m, false);
}
```

注意形式参数中的泛型

```
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            // 其实构造函数时 table是null
            if (table == null) { // pre-size
                // 初始化数据的容量
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            else if (s > threshold)
                resize();
            // 遍历要插入的map中的每一个节点 并把它们插入到调用它的map中
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }

```

其实除了构造器中调用了`putMapEntries`函数之外，还有`putAll()`函数调用了它

下一节 ： [put与get函数(总)](https://github.com/zhangzhaolin/StudyNotes/blob/master/Java/%E8%87%AA%E5%B7%B1%E7%9A%84Java%E7%AC%94%E8%AE%B0/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E2%80%94%E2%80%94put%E5%92%8Cget%EF%BC%88%E6%80%BB%EF%BC%89.md)
