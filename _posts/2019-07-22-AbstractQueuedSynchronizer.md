AbstractQueuedSynchronizer队列同步器中维护了一个FIFO的双向链表作为同步队列，当在线程中调用JAVA提供的或者自定义锁的lock方法获取同步状态失败时，会将获取同步状态失败的线程引用以及以及获取同步状态的模式（独占还是共享）构成节点信息并添加到链表尾部。

AbstractQueuedSynchronizer队列同步器提供了很多获取同步状态的方法，其中可以分为**独占模式获取同步状态**、**共享模式获取同步状态**两大类，每个分类中包含**不响应中断**（acquire(int arg)、acquireShared(int)）、**响应中断**（acquireInterruptibly(int arg)、acquireSharedInterruptibly(int arg)）、**超时获取**（tryAcquireNanos(int arg, long nanosTimeout)、tryAcquireSharedNanos(int arg, long nanosTimeout)），其中超时获取也属于响应中断。

无论哪种获取同步状态的方法，其流程都是获取同步状态，获取同步状态失败后构造节点加入到队列中，接着自旋检查并尝试获取同步状态。

独占模式与共享模式的区别在于判断获取同步状态是否成功的方式，因为独占式只有一个线程可以获取到同步状态，而共享模式则允许多个线程，所以共享模式判断是否获取同步状态成功的方式主要是获取同步状态的线程数是否到达指定阀值，或者同步状态已经被独占模式锁获取。

在自旋时是否响应中断，取决于在自旋时检测到线程已经被中断时是返回中断标识(interrupted = true;)还是直接抛出异常(throw new InterruptedException();)。

超时获取是在自旋的开始计算出超时的deadline，然后检查当前时间跟deadline之间的**差值**并与spinForTimeoutThreshold（1000L）的比较，如果差值较大则对当前线程执行parkNanos操作，将当前线程挂起指定的时间，线程在到达指定的时间或者在前驱节点释放同步状态时被激活，激活之后，先检查线程是被中断，如果中断则抛出异常，否则继续尝试获取同步状态。

除了超时获取状态外，其他的线程在自旋过程中都会被LockSupport.park()，直到前驱节点执行完成并调用LockSupport.unpark()来进行唤醒，而超时获取同步状态在指定时间内如果没有获取到同步状态则会返回false或响应线程的中断操作.	

接下来通过代码来看看AbstractQueuedSynchronizer的实现

## 构造节点并添加到队列尾部

因为同时会有个线程尝试获取同步状态，而获取到同步状态的只有一个，失败的会有多个。因此通过CAS操作来确保多线程下的一致性。

```java
private Node addWaiter(Node mode) {
    // 根据当前线程构造节点信息
    Node node = new Node(Thread.currentThread(), mode);
    // 尝试快速插入到队列尾部
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```
```java
// 通过类似死循环的方式来保证节点的正确添加，只有通过CAS将节点设置为尾节点时，当前线程才能从该方法中返回
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // 初始化
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

## 判断在获取同步状态失败后是否执行park操作

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        /*
         * This node has already set status asking a release
         * to signal it, so it can safely park.
         */
        return true;
    if (ws > 0) {
        /*
         * Predecessor was cancelled. Skip over predecessors and
         * indicate retry.
         */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        /*
         * waitStatus must be 0 or PROPAGATE.  Indicate that we
         * need a signal, but don't park yet.  Caller will need to
         * retry to make sure it cannot acquire before parking.
         */
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

## 线程执行park操作并判断线程是否被中断

LockSupport.park方法在线程已经被中断时不生效，所以如果在执行parkAndCheckInterrupt之前线程已经被中断，则直接返回true，否则则需要其他线程调用LockSupport.unpark来恢复线程，并检查线程是否被中断。

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

## 独占式自旋检查-不响应中断

节点进入同步队列后，就进入一个自旋的过程，每个节点都在自身是否可以尝试获取同步状态

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            
            // 当前驱节点是首节点时尝试获取同步状态
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            
            // 如果首节点不是同步状态，或者尝试获取同步状态失败就阻塞当前节点
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

## 独占式自旋检查-响应中断

```java
private void doAcquireInterruptibly(int arg)
    throws InterruptedException {
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

## 独占式自旋检查-超时设定

```java
private boolean doAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return true;
            }
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

## 共享模式的自旋检查-不响应中断

```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

## 共享式自旋检查-响应中断

```java
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

### 共享式自旋检查-超时设定

```java
private boolean doAcquireSharedNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return true;
                }
            }
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```


### 同步状态释放


当首节点执行操作完成后，需要释放同步状态，并通知后继节点，让后继节点尝试获取同步状态。

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

```java
/**
 * Wakes up node's successor, if one exists.
 *
 * @param node the node
 */
private void unparkSuccessor(Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        
        // 从尾节点开始，从后往前查找符合条件的节点
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```


