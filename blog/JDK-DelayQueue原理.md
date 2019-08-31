---
title: JDK-DelayQueue原理

date: 2017-08-31 10:41:50

tags: [JDK]

categories: 后端开发

---

> 从DelayQueue的`概念，结构，参数，源码解析（offer,poll,remove,add,grow），性能，线程安全性，使用场景，常见问题`8个方面进行分析。

>* An `unbounded blocking queue` of `Delayed elements`, in which an element can only be taken when its delay has expired. 
>* The head of the queue is that Delayed element whose delay expired furthest in the past. 
>* If no delay has expired there is no head and poll will return null. 
>* Expiration occurs when an element's `getDelay`(TimeUnit.NANOSECONDS) method returns a value less than or equal to zero. 
>* Even though `unexpired elements cannot be removed` using take or poll, they are otherwise treated as normal elements. 
>* For example, the size method returns the count of both expired and unexpired elements. 
>* This queue does `not permit null elements`.
>* This class and its iterator implement all of the optional methods of the `Collection` and `Iterator` interfaces.
>* The Iterator provided in method iterator() is `not guaranteed` to traverse the elements of the DelayQueue in `any particular order`.
>* This class is a member of the Java Collections Framework.

>关键点：基于priority queue，无界阻塞队列，实现Collection,Iterator接口、不允许null键/值；

<!-- more --> 
