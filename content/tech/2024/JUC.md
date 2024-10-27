---
title: "Java并发编程-JUC"
date: 2024-10-27T22:00:00+08:00
tags: ["tech"]
categories: ["tech"]
---



# 锁

线程私有的区域（程序计数器，虚拟机栈，本地方法栈）不会出现线程竞争的问题，线程共享的区域（堆，方法区）中，多个线程在竞争其中一些数据时，可能会发生难以预料的异常情况，因此需要锁机制对其进行限制。
锁是一种抽象的概念，在Java代码中每个对象都拥有一把锁，存在对象头中。锁中记录了当前对象被哪个线程占用。
## 对象和对象头
对象头包括MarkWord和ClassPoint

ClassPoint指针指向当前对象类型所在的方法区中的类型数据
MarkWord存放了很多和当前对象运行时状态相关的数据


synchronized通过Javac编译后会生成monitorenter和monitorexit指令


### 无锁
没有对资源锁定，所有线程都能访问同一资源
无竞争：直接访问
有竞争：非锁方式 CAS 
### 偏向锁
 一个对象被加锁，但实际运行时，只有一个线程会获取这个对象锁，那么最理想的方式就是不通过线程切换，也不通过CAS获得锁，CAS多少会耗费一些性能。设想对象能认识这个线程，那么只要这个线程过来，对象就直接把锁交出去，可以理解为对象爱这个线程。
 在对象头的MarkWord中，当锁标志位是01，判断倒数第3个bit是否为1，为1则代表当前的锁状态为偏向锁，0是无锁。如果是偏向锁，再去读取MarkWord的前23个bit，就是线程id，通过线程id来确定当前来获取锁的线程是不是之前的。如果不是之前的，即有多个线程来获取锁，那么就会锁升级为轻量级锁？ 

### 轻量级锁
 不会再通过线程id判断，而是将前30个bit变为，指向线程栈中的锁记录的指针，就是，当一个线程想要获取锁，看到锁标志位是00，那么就知道它是轻量级锁，这时线程会在自己的虚拟机栈中开辟一块lockrecord的空间
### 重量级锁

## CAS
compare and swap 比较并交换，比如现在两个线程同时去获取一个资源，这个资源对象值为0的一瞬间，ab线程都读到了，此时两条线程认为当前状态的值为0，于是他们各自会产生两个值，oldvalue代表读到的资源状态值，newvalue代表想要更新的值。此时ab线程争抢去修改资源对象的值，然后占用。假设a线程先获得时间片，将oldvalue与资源状态值进行compare发现一致，于是将资源对象的值swap为newvalue，而b线程晚了一步，此时资源对象的值已经被a线程修改为了1，所以b线程在compare的时候发现和自己预期的oldvalue值不一样，所以放弃swap操作，但实际应用中，不会让b线程就此放弃，通常会使其进行自旋，不断重试，通常会配置自旋次数防止死循环。

比较并交换的动作必须，同时只有一条线程操作，也就是CAS必须是原子的。CPU原生实现了CAS，上层进行调用即可。就不再依赖锁进行线程同步。

AtomicInteger底层就使用了CAS
incrementAndGet方法直接调用unsafe的getAndAddInt方法

# AQS
AbstractQueuedSynchronizer 抽象队列同步器，构建锁或者其他同步组件的框架。
Java语言实现的，需要手动开启和关闭
### 成员变量
head和tail双向队列，state：volatile修饰的int类型，保证多个线程间的可见性，0表示无锁，1表示有锁。
如果有一个线程a想要获得锁，会先修改state，从0修改为1。此时再有线程b也想要获得锁修改state，就会修改失败，那么线程b就会进入队列等待。再有线程c来也是一样，会进入队列等待，所以底层有一个先进先出的双向队列，内部是双向链表head和tail实现的。当线程a执行完毕之后，就会把state再改为0，同时唤醒队列中的head元素，让head去持有锁。这就是AQS的基本机制。

多个线程争抢资源时，内部使用的时CAS保证了原子性。

AQS可以实现公平锁和非公平锁，线程a释放锁，唤醒队列head，此时有新线程c，head和c同时争抢资源是非公平锁。让新线程c进入队列尾部等待，是公平锁。


``` java
/**  
 * Head of the wait queue, lazily initialized.  Except for 
 * initialization, it is modified only via method setHead.  Note: 
 * If head exists, its waitStatus is guaranteed not to be 
 * CANCELLED. 
 * */
 private transient volatile Node head;  
  
/**  
 * Tail of the wait queue, lazily initialized.  Modified only via 
 * method enq to add new wait node. 
 */
 private transient volatile Node tail;  
  
/**  
 * The synchronization state. 
 * */
 private volatile int state;
```


Node

``` java
static final class Node {  
    /** Marker to indicate a node is waiting in shared mode */  
    static final Node SHARED = new Node();  
    /** Marker to indicate a node is waiting in exclusive mode */  
    static final Node EXCLUSIVE = null;  
  
    /** waitStatus value to indicate thread has cancelled */  
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
	 * */
	 volatile int waitStatus;  
	  
	/**  
	 * 前指针 
	 * */
	 volatile Node prev;  
	  
	/**  
	 * 后指针
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





# ReentrantLock
可重入锁
可中断，可以设置超时时间，可以设置公平锁，支持多个条件的变量
底层就是利用CAS和AQS实现的
继承Lock，只有一个sync变量

``` java
public class ReentrantLock implements Lock, java.io.Serializable {
	/** Synchronizer providing all implementation mechanics */
	private final Sync sync;

}

```

## Lock
接口
``` java
public interface Lock {
	// 获取锁，假如当前锁被其他线程占用，将会等待直到获取
	void lock();
	// 获取锁，当前线程在等待锁的时候被中断，会推出等待，并抛出中断异常
	void lockInterruptibly() throws InterruptedException;
	// 尝试获取锁，并立即返回，Boolean类型代表是否获取锁
	boolean tryLock();
	// 在一段时间内获取锁，假如被中断，将会抛出中断异常
	boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
	// 释放锁
	void unlock();
	// 新建一个绑定在当前lock上的Condition对象
	Condition newCondition();
}

```
## 构造方法

无参构造一个非公平锁
带参数的传入true可以构造公平锁 

``` java
/**  
 * Creates an instance of {@code ReentrantLock}.  
 * This is equivalent to using {@code ReentrantLock(false)}.  
 */
public ReentrantLock() {  
    sync = new NonfairSync();  
}  
  
/**  
 * Creates an instance of {@code ReentrantLock} with the  
 * given fairness policy. 
 * 
 * @param fair {@code true} if this lock should use a fair ordering policy  
 */
public ReentrantLock(boolean fair) {  
    sync = fair ? new FairSync() : new NonfairSync();  
}
```

## Sync类

``` java
/**  
 * Base of synchronization control for this lock. Subclassed 
 * into fair and nonfair versions below. Uses AQS state to 
 * represent the number of holds on the lock. 
 * 
 * */
abstract static class Sync extends AbstractQueuedSynchronizer {  
    private static final long serialVersionUID = -5179523762034025860L;  
  
    /**  
     * Performs {@link Lock#lock}. The main reason for subclassing  
     * is to allow fast path for nonfair version.     
     * */    
     abstract void lock();  
  
    /**  
     * Performs non-fair tryLock.  tryAcquire is implemented in     
     * subclasses, but both need nonfair try for trylock method.     
     * */    
    final boolean nonfairTryAcquire(int acquires) {  
        final Thread current = Thread.currentThread();  
        int c = getState();  
        if (c == 0) {  
            if (compareAndSetState(0, acquires)) {  
                setExclusiveOwnerThread(current);  
                return true;            }  
        }  
        else if (current == getExclusiveOwnerThread()) {  
            int nextc = c + acquires;  
            if (nextc < 0) // overflow  
                throw new Error("Maximum lock count exceeded");  
            setState(nextc);  
            return true;        }  
        return false;  
    }  
  
    protected final boolean tryRelease(int releases) {  
        int c = getState() - releases;  
        if (Thread.currentThread() != getExclusiveOwnerThread())  
            throw new IllegalMonitorStateException();  
        boolean free = false;  
        if (c == 0) {  
            free = true;  
            setExclusiveOwnerThread(null);  
        }  
        setState(c);  
        return free;  
    }  
  
    protected final boolean isHeldExclusively() {  
        // While we must in general read state before owner,  
        // we don't need to do so to check if current thread is owner        
        return getExclusiveOwnerThread() == Thread.currentThread();  
    }  
  
    final ConditionObject newCondition() {  
        return new ConditionObject();  
    }  
  
    // Methods relayed from outer class  
  
    final Thread getOwner() {  
        return getState() == 0 ? null : getExclusiveOwnerThread();  
    }  
  
    final int getHoldCount() {  
        return isHeldExclusively() ? getState() : 0;  
    }  
  
    final boolean isLocked() {  
        return getState() != 0;  
    }  
  
    /**  
     * Reconstitutes the instance from a stream (that is, deserializes it).     
     * */    
    private void readObject(java.io.ObjectInputStream s)  
        throws java.io.IOException, ClassNotFoundException {  
        s.defaultReadObject();  
        setState(0); // reset to unlocked state  
    }  
}
```


## NonfairSync类

继承Sync继承AQS，state/head/tail
exclusiveOwnerThread

``` java
/**  
 * Sync object for non-fair locks 
 * */
static final class NonfairSync extends Sync {  
    private static final long serialVersionUID = 7316153563782823691L;  
  
    /**  
     * Performs lock.  Try immediate barge, backing up to normal     
     * acquire on failure.     
     * */    
    final void lock() {  
        if (compareAndSetState(0, 1))  
            setExclusiveOwnerThread(Thread.currentThread());  
        else            
	        acquire(1);  
    }  
  
    protected final boolean tryAcquire(int acquires) {  
        return nonfairTryAcquire(acquires);  
    }  
}
```

# ConcurrentHashMap

#### 1.7分段数组+链表
分段数组：一个个Segment不能扩容，每个Segment对应HashEntry数组可以扩容，HashEntry数组上存的是真的数据，可以链接为链表
put的时候计算元素key的hash值，确认在Segment数组的下标，找到下标之后就会用ReentrantLock加锁，此时再有资源来竞争就会进行CAS，拿到锁的元素，再根据hash值定位出在HashEntry中的位置。
这会有一个问题，就是当有多个key定位到同一个Segment数组下标，只能有一个线程去操作数据，性能不好。
#### 1.8数组+链表+红黑树
利用CAS和synchronize保证并发安全
CAS控制数组节点的添加，就是当hash冲突了，两个元素都要添加到某个下标，会通过CAS保证安全。synchronize只会锁定当前链表或者红黑树的头节点，只要hash不冲突，就不会产生并发问题，效率得到提升。


# CountDownLatch
允许一条或多条线程，等待其他线程中的一组操作完成后，再继续执行



参考

> [寒食君Java并发系列](https://space.bilibili.com/1578320/channel/collectiondetail?sid=313922&spm_id_from=333.788.0.0)
>  [黑马Java面试系列](https://www.bilibili.com/video/BV1yT411H7YK?spm_id_from=333.788.videopod.episodes&vd_source=37ac084d6ff08684dbf1ca0e4d4b2c70&p=97)
>  [JavaGuide](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html)