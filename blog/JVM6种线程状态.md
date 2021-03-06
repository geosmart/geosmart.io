# JVM6种线程状态
## NEW-初始状态
A thread that has not yet started is in this state.

## RUNNABLE-运行中
A thread executing in the Java virtual machine is in this state.

## BLOCKED-阻塞
A thread that is blocked waiting for a monitor lock is in this state.

## WAITING-等待
A thread that is waiting indefinitely for another thread to perform a particular action is in this state.

## TIMED_WAITING-超时等待
A thread that is waiting for another thread to perform an action for up to a specified waiting time is in this state.

## TERMINATED-终止
A thread that has exited is in this state.
 
```java

/**
 *A thread can be in only one state at a given point in time. 
 These states are virtual machine states which do not reflect any operating system thread states.
 *
 * @since 1.5
 */
public enum State {
    /**
     * Thread state for a thread which has not yet started.
     */
    NEW,

    /**
     * Thread state for a runnable thread. 
     * A thread in the runnable state is executing in the Java virtual machine but it may be waiting for other resources from the operating system such as processor.
     */
    RUNNABLE,

    /**
     * Thread state for a thread blocked waiting for a monitor lock. 
     * A thread in the blocked state is waiting for a monitor lock to enter a synchronized block/method 
     * or reenter a synchronized block/method after calling Object.wait.
     */
    BLOCKED,

    /**
     * Thread state for a waiting thread. A thread is in the waiting state due to calling one of the following methods:
     * Object.wait with no timeout
     * Thread.join with no timeout
     * LockSupport.park
     * A thread in the waiting state is waiting for another thread to perform a particular action. 
     * For example, a thread that has called Object.wait() on an object is waiting for another thread to call Object.notify() or Object.notifyAll() on that object. 
     * A thread that has called Thread.join() is waiting for a specified thread to terminate.
     */
    WAITING,

    /**
     * Thread state for a waiting thread with a specified waiting time. 
     * A thread is in the timed waiting state due to calling one of the following methods with a specified positive waiting time:
     * Thread.sleep
     * Object.wait with timeout
     * Thread.join with timeout
     * LockSupport.parkNanos
     * LockSupport.parkUntil
     */
    TIMED_WAITING,

    /**
     * Thread state for a terminated thread. 
     * The thread has completed execution.
     */
    TERMINATED;
}

```
