---
title: JDK-PriorityQueue原理

date: 2017-08-23 10:41:50

tags: [JDK]

categories: 后端开发

---

> 从PriorityQueue的`概念，结构，参数，性能，线程安全性，源码解析（offer,poll,remove,add,grow），使用场景，常见问题`8个方面进行分析。

>* An `unbounded priority queue` based on a `priority heap`. 
>* The elements of the priority queue are ordered according to their `natural ordering`, or by a `Comparator provided` at queue >construction time, depending on which constructor is used. 
>* A priority queue does `not permit null elements`. 
>* A priority queue relying on natural ordering also does `not permit insertion of non-comparable objects` (doing so may result in >ClassCastException).
>* The `head of this queue` is the `least element` with respect to the `specified ordering`. If multiple elements are tied for >least value, the head is one of those elements -- ties are broken arbitrarily. 
>* The queue retrieval operations `poll`, `remove`, `peek`, and element access the element at the head of the queue.
>* `A priority queue is unbounded`, but has an `internal capacity` governing the size of an `array` used to store the elements on >the queue. It is always at least as large as the queue size. As elements are added to a priority queue, its `capacity grows >automatically`. The details of the growth policy are not specified.
>* This class and its iterator implement all of the optional methods of the `Collection and Iterator interfaces`. 
>* The Iterator provided in method iterator() is `not guaranteed` to traverse the elements of the priority queue in any particular >`order`. If you need ordered traversal, consider using `Arrays.sort(pq.toArray())`.
>* Note that this implementation is `not synchronized`. Multiple threads should not access a PriorityQueue instance concurrently if >any of the threads modifies the queue. Instead, use the `thread-safe PriorityBlockingQueue` class.
>* Implementation note: this implementation provides 
>    * `O(log(n)) time` for the `enqueuing` and `dequeuing` methods (`offer, poll, remove() and add`); 
>    * `linear time` for the `remove(Object) and contains(Object)` methods; 
>    * `constant time` for the retrieval methods (`peek, element, and size`).
>This class is a member of the `Java Collections Framework`.

>关键点：基于priority heap，无界队列，实现Queue,Collection,Iterator接口、不允许null键/值、非线程安全、enqueue和dequeue都是O(long(n))，remove和contains是O(n)，peek是O(1)；

<!-- more --> 
# 概念
>* 优先队列跟普通的队列不一样，普通队列遵循FIFO规则出队入队，而优先队列每次都是优先级最高出队。
>* 优先队列内部维护着一个堆，每次取数据的时候都从堆顶取，这是优先队列的基本工作原理。
* jdk的优先队列使用PriorityQueue这个类，使用者可以自己定义优先级规则。
>PriorityQueue的类关系
![priority_queue_hier](img/priority_queue_hier.png)

>PriorityQueue的类成员
![priority_queue_class](img/priority_queue_class1.png)

# 参考
* [jdk8.PriorityQueue](https://docs.oracle.com/javase/8/docs/api/java/util/PriorityQueue.html) 