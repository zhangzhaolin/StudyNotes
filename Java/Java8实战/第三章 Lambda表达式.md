# 第三章 Lambda表达式

## 管中窥豹

可以把Lambda表达式理解为简洁地表示可传递的匿名函数的一种方式：**它没有名字，但它有：参数列表、函数主题、返回类型，可能还有一个可以抛出的异常列表。**

例如，可以利用Lambda表达式，可以更为简洁的创建出`Comparator`对象 ：

![](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E6%88%AA%E5%9B%BE/Java8%E5%AE%9E%E6%88%98/Character3/1.png)

在JDK8出现之前和之后代码看起来更加清晰 ：

![](https://raw.githubusercontent.com/zhangzhaolin/StudyNotes/master/%E6%88%AA%E5%9B%BE/Java8%E5%AE%9E%E6%88%98/Character3/2.png)

