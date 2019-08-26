---
title: JDK-PriorityQueue原理

date: 2017-08-23 10:41:50

tags: [JDK]

categories: 后端开发

---

> 从PriorityQueue的`概念，结构，参数，源码解析（offer,poll,remove,add,grow），性能，线程安全性，使用场景，常见问题`8个方面进行分析。

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

# 结构
一维数组

* Priority queue represented as a `balanced binary heap`:
* the two children of queue[n] are queue[2*n+1] and queue[2*(n+1)].  
* The priority queue is `ordered by comparator`, or by the elements' `natural ordering`, 
* if comparator is null: For each node n in the heap and each descendant d of n, n <= d.  
* The element with the `lowest value` is in queue[0], assuming the queue is nonempty.
```java
    // non-private to simplify nested class access
    transient Object[] queue; 
```

# 参数
* `initialCapacity`：初始化容量，默认为`11`；
* `comparator`:用于队列中元素排序；
* 构造函数：新建1个空的队列；
```java
    public PriorityQueue(int initialCapacity, Comparator<? super E> comparator) {
        this.queue = new Object[initialCapacity];
        this.comparator = comparator;
    }
```
* 如果是由`SortedSet`,`PriorityQueue`这种有序的结构构建优先队列，直接`Arrays.copyOf`把数据复制到queue数组中；
* 如果是由无序数组构建优先队列，需要把数据复制到queue数组中后，执行`构建堆(heapify)`操作；

# 源码解析
## heapify-构建堆
```java
    /**
     * Establishes the heap invariant (described above) in the entire tree,
     * assuming nothing about the order of the elements prior to the call.
     */
    @SuppressWarnings("unchecked")
    private void heapify() {
        //从最后一个非叶子节点（父亲节点）开始遍历所有父节点，直到堆顶
        for (int i = (size >>> 1) - 1; i >= 0; i--){
            //下沉（将3 or 2者中较大元素下沉）
            siftDown(i, (E) queue[i]);
        }
    }
```

### siftDown-下沉
```java

    /**
     * Inserts item x at position k, maintaining heap invariant by demoting x down the tree repeatedly
     * until it is less than or equal to its children or is a leaf.
     *
     * @param k the position to fill
     * @param x the item to insert
     */
    private void siftDown(int k, E x) {
        if (comparator != null) {
            //按自定义顺序swap下沉
            siftDownUsingComparator(k, x);
        } else {
            //按字典顺序swap下沉
            siftDownComparable(k, x);
        }
    }
```

> 按字典顺序swap下沉
```java
    private void siftDownComparable(int k, E x) {
        Comparable<? super E> key = (Comparable<? super E>) x;
        int half = size >>> 1;
        //二叉树结构，下标大于size/2都是叶子节点，其他的节点都有子节点。
        //循环至最后一个非叶子节点：loop while a non-leaf
        while (k < half) {
            //假设left节点为child中的最小值节点
            int child = (k << 1) + 1;
            int right = child + 1;
            Object c = queue[child];
            //right没超过数组大小，且right<left，则最小为right
            if (right < size && ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0) {
                c = queue[child = right];
            }
            //如果parent节点<min(left,right),则不需要swap
            if (key.compareTo((E) c) <= 0) {
                break;
            }
            //否则swap parent节点和min(left,right)的节点
            queue[k] = c;
            //当前父节点取最小值的index
            k = child;
        }
        //当前节点赋值到n轮swap后的最小值
        //或者当前节点没有子节点，则k是叶子节点的下标，没有比它更小的了，直接赋值即可
        queue[k] = key;
    }
```

> 按自定义顺序swap下沉，与siftDownComparable类似
```java
    @SuppressWarnings("unchecked")
    private void siftDownUsingComparator(int k, E x) {
        int half = size >>> 1;
        while (k < half) {
            int child = (k << 1) + 1;
            Object c = queue[child];
            int right = child + 1;
            if (right < size &&
                comparator.compare((E) c, (E) queue[right]) > 0)
                c = queue[child = right];
            if (comparator.compare(x, (E) c) <= 0)
                break;
            queue[k] = c;
            k = child;
        }
        queue[k] = x;
    }
```
## offer
## add
## pop
## remove
## peek

# 参考
* [jdk8.PriorityQueue](https://docs.oracle.com/javase/8/docs/api/java/util/PriorityQueue.html) 