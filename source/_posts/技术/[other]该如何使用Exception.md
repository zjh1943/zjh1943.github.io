---
title: '[other]该如何使用Exception'
tags: 
- exception 
- oo
date: 2016-03-07 12:23
categories: 
- 技术
---

摘要：怎么处理别人的Exception？什么时候改抛发自己的Exception？

# Exception有什么好处?

1. 让你的错误信息更完整，接口更统一。
2. 常见的，用函数返回至来标记错误的方式，灵活性很差。


#  Gunjan Doshi的博客

Gunjan Doshi的博客[Best Practices for Exception Handling](http://www.onjava.com/pub/a/onjava/2003/11/19/exceptions.html)谈了一些经验，总结来说就是:

## 如何抛发自己的Exception

1.  如果你的客户端程序能处理它，你就抛发 checked exception；如果你的客户端程序对这个exception除了打印出来什么也做不了，就throw unchecked exception。
2. 如果一个exception你处理不了，请不要单纯的打印出来，最好的方法是继续把它封装成 unchecked exception 继续throw.
3. 如果你define一个新的exception，一定要提供更多的信息，不要让你的class定义只有构造函数。如果觉得没什么更多的信息，那就不要define，直接用 Exception。


## 如何处理别人的Exception

1. 在try{}catch{}后一定要记得清理现场，处理后事。e.g. 如果发生数据库连接异常，一定要在finally里断开连接。
2. 不要用exception做flow control，效率低的一B。
3. 如果某个exception你处理不了，重新抛发它，千万不要包庇隐藏它。
4. 千万不要出现`catch(Exception e)`这样的代码，它会把所有的exception捕捉在这里，你能处理的，不能处理的，全在。
5. 一个exception只log一次。

# The Digital Gabeg 在 Github 上的回答

当且仅当你的方法的最基本的前提假设不满足的时候，才抛发异常。

举个例子：

```java
boolean isList(Object obj);//判断obj是否是List<>.
boolean getLengthOfList(Object obj);//获得obj的长度。
```

现在，假设传入的参数不是`List<>`类型，那么，函数`isList()`就不应该throw exception. 因为这完全在意料之中啊。而函数`getLengthOfList()`的前提假设就是你传进来的参数必须是`List<>`，否则这个函数就木有意义；此时，你传进来的参数，我处理不了，所以要throw exception。



