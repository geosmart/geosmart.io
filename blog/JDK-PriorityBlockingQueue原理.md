---
title: JDK-PriorityBlockingQueue原理

date: 2017-08-23 10:41:50

tags: [JDK]

categories: 后端开发

---

> 从PriorityBlockingQueue的`概念，结构，参数，源码解析（offer,poll,remove,add,grow），性能，线程安全性，使用场景，常见问题`8个方面进行分析。

> An unbounded BlockingQueue blocking queue that uses the `same ordering rules as class PriorityQueue` and supplies `blocking retrieval operations`.  
* While this queue is logically unbounded, attempted additions may fail due to resource exhaustion (causing OutOfMemoryError). 
* This class does not permit null elements.  
* A priority queue relying on `Comparable natural ordering` also does not permit insertion of `non-comparable` objects (doing so results in ClassCastException).
* This class and its iterator implement all of the optional methods of the Collection and Iterator interfaces.  
* The Iterator provided in method iterator() is `not guaranteed` to traverse the elements of the PriorityBlockingQueue in any particular `order`. If you need ordered traversal, consider using `Arrays.sort(pq.toArray())`.  Also, method drainTo can be used to remove some or all elements in priority  order and place them in another collection.
* Operations on this class make `no guarantees` about the `ordering` of elements with `equal priority`. 
If you need to enforce an ordering, you can define custom classes or comparators that use a secondary key to break ties in primary priority values. 

For example, here is a class that applies `first-in-first-out` tie-breaking to comparable elements. To use it, you would insert a
 `new FIFOEntry(anEntry)` instead of a plain entry object.
 ```java
 class FIFOEntry<E extends Comparable<? super E>>  implements Comparable<FIFOEntry<E>> {
    static final AtomicLong seq = new AtomicLong(0);
    final long seqNum;
    final E entry;
    public FIFOEntry(E entry) {
      seqNum = seq.getAndIncrement();
      this.entry = entry;
    }
    
    public E getEntry() { return entry; }

    public int compareTo(FIFOEntry<E> other) {
      int res = entry.compareTo(other.entry);
      if (res == 0 && other.entry != this.entry)
        res = (seqNum < other.seqNum ? -1 : 1);
      return res;
    }
 }}
 ```
 * This class is a member of the Java Collections Framework

>关键点：与PriorityQueue一样的排序规则，无界队列，实现Queue,Collection,Iterator接口、不允许null键/值、提供阻塞操作、线程安全、不保证队列内元素的顺序；

<!-- more --> 
# 概念


>PriorityQueue的类关系

![priority_queue_hier](img/priority_blocking_queue_hier.png)

>PriorityQueue的类成员

![priority_queue_class](img/priority_blocking_queue_class.png)

# 结构 
# 参数

# 源码解析 
The implementation uses an `array-based binary heap`, with public operations protected with a `single lock`. However, allocation during resizing uses a simple `spinlock` (used only while not holding main lock) in order to allow takes to operate concurrently with allocation.  
This avoids repeated postponement of waiting consumers and consequent element build-up. The need to back away from lock during allocation makes it impossible to simply wrap delegated `java.util.PriorityQueue` operations within a lock, as was done in a previous version of this class. To maintain interoperability, a plain PriorityQueue is still used during serialization, which maintains compatibility at the expense of transiently doubling overhead.
> TODO 搞清楚spinLock间隙锁


# 性能 
# 线程安全性 
# 使用场景 
# 常见问题 
# 参考
* [jdk8.PriorityBlockingQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/PriorityBlockingQueue.html) 
