# HashMap源码分析 —— put和get（总）

## HashMap的put函数总结

仿照美团文章，我也做了一个有关put函数的整理 ：

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-7-27/13348411.jpg)

## 一问一答

- 当两个对象的hashcode()相同时会发生什么?

当两个对象的hashcode相同，首先能确定的是这两个对象一定会在数组的同一个位置，因为hashmap是通过干扰函数以及数组长度做与运算来计算索引值的。当然，如果两个对象的key相同或者说equals(key)相同，那么HashMap会留下后面的一个，前面的被后面的覆盖了。

- 如果两个键的hashcode相同，你如何获取值对象?

同上，在两个节点会在同一个位置，要么是以红黑树的形式，要么以链表的形式存放。如果以链表的形式存放的话，就遍历链表并通过key.equals()的方式来获取.如果以红黑树的形式存放的话...(balabala)

- hash的实现方式是什么？为什么要这么做？

hash的实现方式是根据key的hashCode，然后hashCode的高十六位和第十六位做异或。这样做主要有两个原因，第一个原因是可能用户Override了hashCode函数，导致hashCode结果不怎么样，第二个原因是因为这样做异或处理可以减少碰撞，以默认的数组容量16距离来说，如果16直接与hashCode做与运算的话，并且hashCode的末四位都是0，那么极易可能会发生碰撞。

- HashMap 的 table 的容量如何确定？loadFactor 是什么？ 该容量如何变化？

table默认是1 << 4即16，如果是自定义初始容量的话，那么用户自定义的容量会经过函数处理为2的次方，table最大容量为 1 << 30，loadFactor为加载因子，默认为0.75，加载因子在扩容的时候一般都扩大两倍，table也会变为原来的两倍。

- 请简要说下put操作？

太多了，看图吧 ^ _ ^

## 参考
- [Java 8系列之重新认识HashMap](https://tech.meituan.com/java_hashmap.html)
- http://rkhcy.github.io/2017/12/03/%E5%9B%BE%E8%A7%A3HashMap(%E4%B8%80)/

## 目录

- [put和get(一)](https://github.com/zhangzhaolin/StudyNotes/blob/master/Java/%E8%87%AA%E5%B7%B1%E7%9A%84Java%E7%AC%94%E8%AE%B0/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E2%80%94%E2%80%94%20put%E4%B8%8Eget%EF%BC%88%E4%B8%80%EF%BC%89.md)
- [put和get(二)](https://github.com/zhangzhaolin/StudyNotes/blob/master/Java/%E8%87%AA%E5%B7%B1%E7%9A%84Java%E7%AC%94%E8%AE%B0/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E2%80%94%E2%80%94%20put%E4%B8%8Eget%EF%BC%88%E4%BA%8C%EF%BC%89.md)
- [put和get(三)](https://github.com/zhangzhaolin/StudyNotes/blob/master/Java/%E8%87%AA%E5%B7%B1%E7%9A%84Java%E7%AC%94%E8%AE%B0/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E2%80%94%E2%80%94%20put%E4%B8%8Eget%EF%BC%88%E4%B8%89%EF%BC%89.md)
- [put和get(四)](https://github.com/zhangzhaolin/StudyNotes/blob/master/Java/%E8%87%AA%E5%B7%B1%E7%9A%84Java%E7%AC%94%E8%AE%B0/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E2%80%94%E2%80%94%20put%E4%B8%8Eget%EF%BC%88%E5%9B%9B%EF%BC%89.md)
