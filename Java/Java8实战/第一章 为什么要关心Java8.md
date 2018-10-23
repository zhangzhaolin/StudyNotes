# 为什么要关心Java8

## 一个苹果开始

假设有一个苹果类，我们要从苹果列表中筛选出颜色为绿色的那些苹果 ：

![](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E6%88%AA%E5%9B%BE/Java8%E5%AE%9E%E6%88%98/Character1/1.png)

接下来，有人想要选出那些比较重的苹果，于是，你开始再次写了下面的方法，某些行开始了复制粘贴 ：

```java
private static List<Apple> filterHeavyApples(List<Apple> appleList){
    List<Apple> result = new ArrayList<>();
    for (Apple apple:appleList){
        if(apple.getWeight() > 35){
            result.add(apple);
        }
    }
    return result;
}
```

我们发现，这和之间的`filterGreenApples`方法没有明显的差别，只有判断条件有所不同 。

![](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E6%88%AA%E5%9B%BE/Java8%E5%AE%9E%E6%88%98/Character1/2.png)

我们想要写一个公用的方法，无论是什么筛选条件（苹果重量啦、苹果颜色等），都可以进入到这个方法中，然后替换红框的部分 。

这在Java8之前是无法做到的，但是在Java8当中，方法可以被当做值，然后传递到另外的方法中 。



