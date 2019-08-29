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
* `size`:记录队列中元素个数；
* `modCount`:记录队列修改次数；
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
    @SuppressWarnings("unchecked")
    private void siftDownComparable(int parent, E x) {
        Comparable<? super E> parentVal = (Comparable<? super E>) x;
        System.out.println(String.format("start siftdown,parent[%s]=%s", parent, x));
        int half = size >>> 1;
        //二叉树结构，下标大于size/2都是叶子节点，其他的节点都有子节点。
        //循环直到k没有子节点：loop while a non-leaf
        while (parent < half) {
            dump();
            //假设left节点为child中的最小值节点
            int left = (parent << 1) + 1;
            int right = left + 1;
            System.out.println(String.format("handle parent[%s]=%s,left[%s]=%s,right[%s]=%s", parent, parentVal, left, queue[left], right, queue[right]));
            Object minVal = queue[left];
            //存在right，且right<left，则最小为right
            if (right < size && ((Comparable<? super E>) minVal).compareTo((E) queue[right]) > 0) {
                System.out.println(String.format("min(left(%s),right(%s))=%s", minVal, queue[right], queue[right]));
                left = right;
                minVal = queue[right];
            }
            //如果parent节点<min(left,right),则不需要swap
            if (parentVal.compareTo((E) minVal) <= 0) {
                System.out.println(String.format("parent(%s)<min(%s),break", parentVal, minVal));
                break;
            }
            System.out.println(String.format("swap parent(%s)<->min(%s)", queue[parent], minVal));
            //否则swap parent节点和min(left,right)的节点
            queue[parent] = minVal;
            System.out.println(String.format("now parent[%s]=%s", parent, minVal));
            //当前父节点取最小值的index继续loop
            parent = left;
            System.out.println(String.format("set parent idx=%s to loop", left));
        }
        //1.当前节点没有子节点，则k是叶子节点的下标，没有比它更小的了，直接赋值即可
        //2.当前节点下沉n轮后，将节点的值放到最终不需要再交换的位置（没有比它更小的或者到达叶子节点）
        System.out.println(String.format("end siftdown,set parent[%s]=%s", parent, parentVal));
        queue[parent] = parentVal;
        dump();
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
```java
    /**
     * The number of times this priority queue has been structurally modified 
     */
    transient int modCount = 0; 
```
```java
    /** 
     * 节点插入到队列 
     */
    public boolean offer(E e) {
        if (e == null) {
            throw new NullPointerException();
        }
        //修改次数+1
        modCount++;
        int i = size;
        if (i >= queue.length) {
            //队列已满时，按50%动态扩容
            grow(i + 1);
        }
        size = i + 1;
        if (i == 0) {
            //队列为空时
            queue[0] = e;
        } else {
            //上浮调整堆顺序
            siftUp(i, e);
        }
        return true;
    }
```
>队列已满时，按50%动态扩容
```java
    /**
     * Increases the capacity of the array.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        int oldCapacity = queue.length;
        // Double size if small; else grow by 50%
        int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                (oldCapacity + 2) :
                (oldCapacity >> 1));
        // overflow-conscious code
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        queue = Arrays.copyOf(queue, newCapacity);
    }
```

> 节点上浮调整
```java
    
    /**
     * Inserts item x at position k, 
     * maintaining heap invariant by promoting x up the tree until it is greater than or equal to its parent, or is the root. 
     * 为保持堆的性质，将插入元素x一路上浮，直到满足x节点值>=父节点值，或者到达根节点；
     * @param k the position to fill 插入位置
     * @param x the item to insert 插入元素
     */
    private void siftUp(int k, E x) {
        if (comparator != null) {
            siftUpUsingComparator(k, x);
        } else {
            siftUpComparable(k, x);
        }
    }
```
    
> 按字典顺序swap上浮
```java
    @SuppressWarnings("unchecked")
    private void siftUpComparable(int k, E x) {
        Comparable<? super E> key = (Comparable<? super E>) x;
        //从当前节点循环上浮到堆顶节点
        while (k > 0) {
            //k节点的父节点索引
            int parent = (k - 1) >>> 1;
            //k节点的父节点值
            Object e = queue[parent];
            //比较k节点与父节点的值大小，父节点值较小时，终止遍历
            if (key.compareTo((E) e) >= 0) {
                break;
            }
            //父节点值较大时，交换k节点与父节点值
            queue[k] = e;
            //当前节点移到父节点，继续向上遍历
            k = parent;
        }
        //将当前节点值赋给交换后的父节点
        queue[k] = key;
    }
```
> 按自定义顺序swap上浮，与siftUpComparable类似
```java
    @SuppressWarnings("unchecked")
    private void siftUpUsingComparator(int k, E x) {
        while (k > 0) {
            int parent = (k - 1) >>> 1;
            Object e = queue[parent];
            if (comparator.compare(x, (E) e) >= 0)
                break;
            queue[k] = e;
            k = parent;
        }
        queue[k] = x;
    }

```
## add
## pop
## remove
## peek

# 参考
* [jdk8.PriorityQueue](https://docs.oracle.com/javase/8/docs/api/java/util/PriorityQueue.html) 
* [堆排序](https://juejin.im/post/5cba5cb9518825327e23f078)
* [java集合之PriorityQueue源码分析](https://juejin.im/post/5cba5cb9518825327e23f078)