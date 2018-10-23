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



