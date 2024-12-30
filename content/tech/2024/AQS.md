---
title: "Java并发编程-AQS"
date: 2024-12-07T22:00:00+08:00
tags: ["tech"]
categories: ["tech"]
---



# AQS

AbstractQueuedSynchronizer 抽象队列同步器，构建锁或者其他同步组件的框架。

```java
public abstract class AbstractQueuedSynchronizer  
    extends AbstractOwnableSynchronizer  
    implements java.io.Serializable {}
```
## 成员变量

AQS底层是一个双端队列实现的，有head和tail两个Node节点，还有一个int类型的state变量表示同步状态。主要通过getState，setState，cas的方式设置state来操作这个状态。当线程获取同步状态失败时，AQS会将当前线程信息和等待状态等信息构造成一个Node节点并加入同步队列，同时会阻塞当前线程，当同步状态释放时，会把head的第一个后继节点，也就是队列中的第一个节点的线程唤醒，然后尝试获取同步状态，此时这个节点就成为了head。Node节点的数据结构其实是链表，保存了当前线程，前后节点，和等待状态等信息。


``` java
/**  
 * Head of the wait queue, lazily initialized.  Except for 
 * initialization, it is modified only via method setHead.  Note: 
 * If head exists, its waitStatus is guaranteed not to be 
 * CANCELLED. 
 * 头节点，当前持有锁的线程
 * */
 private transient volatile Node head;  
  
/**  
 * Tail of the wait queue, lazily initialized.  Modified only via 
 * method enq to add new wait node. 
 * 阻塞的尾节点
 */
 private transient volatile Node tail;  
  
/**  
 * The synchronization state.
 * 锁状态（同步状态），0表示没有占用，大于0表示有线程持有锁，可重入锁可以大于1
 * */
 private volatile int state;
```


### Node节点

``` java
static final class Node {  
    /** Marker to indicate a node is waiting in shared mode */
    // 共享模式  
    static final Node SHARED = new Node();  
    /** Marker to indicate a node is waiting in exclusive mode */
    // 独占模式  
    static final Node EXCLUSIVE = null;  
    
	//------- waitStatus的状态 
	
    /** waitStatus value to indicate thread has cancelled */  
    // 此线程取消了争抢这个锁
    static final int CANCELLED =  1;  
    /** waitStatus value to indicate successor's thread needs unparking */  
    static final int SIGNAL    = -1;  
    /** waitStatus value to indicate thread is waiting on condition */  
    static final int CONDITION = -2;  
    /**  
     * waitStatus value to indicate the next acquireShared should     
     * unconditionally propagate     
     * */   
     static final int PROPAGATE = -3;
	 /**  
	 * 等待状态 上面的枚举值
	 * 大于0也就是1时，代表线程取消了等待，抢半天抢不到就不抢了
	 * */
	 volatile int waitStatus;  
	  
	/**  
	 * 前驱节点 
	 * */
	 volatile Node prev;  
	  
	/**  
	 * 后继节点
	 */
	 volatile Node next;  
	  
	/**  
	 * The thread that enqueued this node.  Initialized on 
	 * construction and nulled out after use. 
	 * 线程对象
	 * */
	 volatile Thread thread;  
  
	/**  
	 * 
	 * */
	 Node nextWaiter;
    }
```

## acquire方法
尝试获取锁（tryAcquire），节点构造并加入同步队列（addWaiter）以及在队列中自旋等待（acquireQueued）

```java
public final void acquire(int arg) {  
    if (!tryAcquire(arg) &&  
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))  
        selfInterrupt();  
}
```

### tryAcquire
ReentrantLock实现的

``` java
protected final boolean tryAcquire(int acquires) {  
    /*  
     * Walkthrough:     
     * 1. If read count nonzero or write count nonzero     
     *    and owner is a different thread, fail.     
     * 2. If count would saturate, fail. (This can only     
     *    happen if count is already nonzero.)     
     * 3. Otherwise, this thread is eligible for lock if     
     *    it is either a reentrant acquire or     
     *    queue policy allows it. If so, update state     
     *    and set owner.     
     **/    
    Thread current = Thread.currentThread();  
    int c = getState();  
    int w = exclusiveCount(c);  
    if (c != 0) {  
        // (Note: if c != 0 and w == 0 then shared count != 0)  
        if (w == 0 || current != getExclusiveOwnerThread())  
            return false;  
        if (w + exclusiveCount(acquires) > MAX_COUNT)  
            throw new Error("Maximum lock count exceeded");  
        // Reentrant acquire  
        setState(c + acquires);  
        return true;    }  
    if (writerShouldBlock() ||  
        !compareAndSetState(c, c + acquires))  
        return false;  
    setExclusiveOwnerThread(current);  
    return true;
}
```

### addWaiter
把线程包装成Node，同时加入队列

```java
private Node addWaiter(Node mode) {
	// 把线程包装成Node
    Node node = new Node(Thread.currentThread(), mode);  
    // Try the fast path of enq; backup to full enq on failure
    // 开始尝试在尾部添加，1.当前节点的prev指向原来的尾节点；2.尾节点的next指向当前节点
    // 原来最后一个尾节点 准备作为当前节点的prev  
    Node pred = tail;
    // pred != null 也就是尾节点不是空  
    if (pred != null) {
	    // pred也就是尾节点赋值给当前节点的prev  
        node.prev = pred;
        // 通过cas把当前节点赋值给尾节点的next  
        if (compareAndSetTail(pred, node)) {
		    // cas成功，实现了作为尾节点加入了链表  
            pred.next = node;  
            return node;  
        }  
    }
    // 如果到这里，说明1.pred是空，或者说队列是空；2.cas失败
    // 开始执行enq方法，死循环竞争入队  
    enq(node);  
    return node;  
}
```


#### enq
通过“死循环”来保证节点的正确添加，在“死循环”中只有通过CAS将节点设置成为尾节点之后，当前线程才能从该方法返回，否则，当前线程不断地尝试设置。

``` java
private Node enq(final Node node) {  
    for (;;) {  
        Node t = tail;
        // 队列为空  
        if (t == null) { // Must initialize
	        // cas初始化头节点  
            if (compareAndSetHead(new Node()))
	            // tail也先指向head，没有return，继续循环  
                tail = head;  
        } else {
	        // 和addWaiter一样的操作 
            node.prev = t;  
            if (compareAndSetTail(t, node)) {  
                t.next = node;  
                return t;  
            }  
        }  
    }  
}
```


### acquireQueued 
挂起 唤醒获取锁
参数node，经过addWaiter(Node.EXCLUSIVE)，此时已经进入阻塞队列

```java
final boolean acquireQueued(final Node node, int arg) {  
    boolean failed = true;  
    try {  
        boolean interrupted = false;  
        for (;;) {
	        // predecessor方法获取node的前驱节点
            final Node p = node.predecessor();
            // 前驱节点是头节点（当前节点是队列的首节点）  再尝试获取锁
            if (p == head && tryAcquire(arg)) {
	            // 成功后将该节点设置为头节点  
                setHead(node);
                // 旧的头节点next置空，也就是和链表没关系了，会被垃圾回收  
                p.next = null; // help GC  
                failed = false;  
                return interrupted;  
            }
            // 执行到这里就是 1.不是首节点；2.获取锁失败
              
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

#### shouldParkAfterFailedAcquire
获取锁失败后，是否需要挂起当前线程
返回true为需要挂起

``` java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {  
    // 前驱节点的等待状态
    int ws = pred.waitStatus;
    // 为SIGNAL 就是正常 返回
    if (ws == Node.SIGNAL)  
        /*  
         * This node has already set status asking a release         
         * to signal it, so it can safely park.         
         * */        
        return true; 
    
    if (ws > 0) {
	    // 前驱节点 waitStatus大于0 说明前驱节点取消了排队  
        /*  
         * Predecessor was cancelled. Skip over predecessors and         
         * indicate retry.         
         * */        
        do {
	        // 循环找到一个正常的前驱节点，作为当前节点的前驱节点  
            node.prev = pred = pred.prev;  
        } while (pred.waitStatus > 0);  
        pred.next = node;  
    } else {
	    // 前驱节点的waitStatus不等于-1和1，那也就是只可能是0，-2，-3
        /*  
         * waitStatus must be 0 or PROPAGATE.  Indicate that we         
         * need a signal, but don't park yet.  Caller will need to         
         * retry to make sure it cannot acquire before parking.         
         * */
         // 用CAS将前驱节点的waitStatus设置为Node.SIGNAL(也就是-1) 从等待队列加入同步队列？
         compareAndSetWaitStatus(pred, ws, Node.SIGNAL);  
    }  
    return false;  
}
```

#### parkAndCheckInterrupt
挂起线程，shouldParkAfterFailedAcquire返回true时会执行

```java
private final boolean parkAndCheckInterrupt() {  
    LockSupport.park(this);  
    return Thread.interrupted();  
}
```

## release方法
解锁

```java
public final boolean release(int arg) {
	// 尝试释放
    if (tryRelease(arg)) {
	    // 进来证明可以释放了  
        Node h = head; 
        // 头节点不为空，状态不是0 
        if (h != null && h.waitStatus != 0)
	        // 执行唤醒  
            unparkSuccessor(h);  
        return true;    }  
    return false;  
}
```


### tryRelease
ReentrantLock实现的

``` java
protected final boolean tryRelease(int releases) {
	// 当前状态-1 
    int c = getState() - releases;  
    if (Thread.currentThread() != getExclusiveOwnerThread())  
        throw new IllegalMonitorStateException();  
    boolean free = false;
    // 是不是重入锁，如果c==0，也就是说没有嵌套锁了，可以释放了，否则还不能释放掉
    if (c == 0) {  
        free = true;  
        setExclusiveOwnerThread(null);  
    }  
    setState(c);  
    return free;  
}
```


### unparkSuccessor
唤醒后继节点

``` java
private void unparkSuccessor(Node node) {
	// node是head头节点
    /*  
     * If status is negative (i.e., possibly needing signal) try     
     * to clear in anticipation of signalling.  It is OK if this     
     * fails or if status is changed by waiting thread.     
     * */    
    int ws = node.waitStatus;
    // head的等待状态小于0，改为0  
    if (ws < 0)  
        compareAndSetWaitStatus(node, ws, 0);  
  
    /*  
     * Thread to unpark is held in successor, which is normally     
     * just the next node.  But if cancelled or apparently null,     
     * traverse backwards from tail to find the actual     
     * non-cancelled successor.     
     * */
    // 开始唤醒后继节点，但有可能后继节点取消了等待>0
    Node s = node.next;  
    if (s == null || s.waitStatus > 0) {  
        s = null;
        // 循环从队尾一直向前找，找到最前面那个（不知道为什么不从头开始找？考虑并发吗？）
        for (Node t = tail; t != null && t != node; t = t.prev)  
            if (t.waitStatus <= 0)  
                s = t;  
    }  
    if (s != null)
	    // 唤醒线程  
        LockSupport.unpark(s.thread);  
}
```