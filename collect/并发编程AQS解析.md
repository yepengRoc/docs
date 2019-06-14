---
typora-root-url: ..\image
---

### 非公平方式获取锁



```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
```

通过lock来查看非公平锁的实现

lock.lock()调用方法

```
sync.lock();
```

调用静态内部类NonfairSync 中的lock方法

```
final void lock() {
	/**
	设置全局状态state 为1。如果设置成功，则设置当前线程为拥有锁的线程
	**/
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else//设置全局变量失败
        acquire(1);
}
```

```
/**
如果尝试获取锁失败，且添加进等待队列失败。则中断
**/
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

```
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```

```
/**
	因为是非公平的方式获取锁。所以再前面compareAndSetState(0, 1) 失败（失败表示当时的state肯定不为0）之后，这里再次进行state值的判断，有可能前面获取锁的线程已经释放锁了。
	如果state=0 标识无线程获取锁，则当前线程再次通过compareAndSetState(0, 1) 设置自己为锁的拥有者
	如果线程就是拥有者，就读state值进行累加，实现可重入锁（可重入锁：获取锁的线程可重复多次获取同一把锁）
**/
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {//说明锁已释放
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {//说明当前线程就是锁的拥有者
        int nextc = c + acquires;
        if (nextc < 0) // overflow  int 越界 4字节 32bit
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

```
/**
  把获取锁的线程构建成双向非循环队列的一个节点
  这里的mode是标识锁的类型，是共享锁 还是排它锁。对应的是
  AbstracQueuedSynchroizer中Node定义中的两个final变量
    static final Node SHARED = new Node();
    static final Node EXCLUSIVE = null;
    
    拿到tail节点
    新节点的pre = tail
    新节点 pre.next = 新节点
    如果tail节点为null。说明当前队列还没有节点
**/
private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
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
    /**
    队列为空。则通过死循环进行设置。因为是多线程。在设置的时候可能tail已经被别的线程设置有值了。
    队列头节点是一个空的Node节点
    **/
     private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
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
/**
  方法名字直译过来就是获取队列。意思是：在这个队列中线程都可以进行锁的获取
  如果当前线程的前一个节点是头节点。尝试获取锁。
  获取成功则设置线程为头节点
  获取失败，说明
**/
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            /**
            shouldParkAfterFailedAcquire
            1前驱节点不是头节点。
            2前驱节点是头节点。但是获取锁失败了
              前驱节点是头节点说明已经是获取锁的状态，不可能是取消状态
              说明前驱节点还没有释放锁。前驱如果没有被取消。如果状态是0 则失败，继续循环，如果不为0则返回true. 
            parkAndCheckInterrupt 通过jdk native方法。阻塞当前线程0秒
            如果线程在阻塞的时候被中断。则是真的中断。-- 温故
            是lock.interrupt
            还是thread.interrupt会抛异常。
            wait 的时候会抛异常。并清除中断位
            
            **/
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)//抛异常了。取消当前节点
            cancelAcquire(node);
    }
}
/**

**/
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)//前驱节点是获取锁的合法状态
            return true;
        if (ws > 0) {//如果前驱节点被取消了。则从新寻找一个没有取消的前驱节点
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {//设置前驱节点 节点状态为 -1. 默认节点状态为0
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
    //取消节点
    private void cancelAcquire(Node node) {
        if (node == null)
            return;

        node.thread = null;
        // Skip cancelled predecessors
        Node pred = node.prev;
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;
       
        Node predNext = pred.next;
        node.waitStatus = Node.CANCELLED;
        // If we are the tail, remove ourselves.
        if (node == tail && compareAndSetTail(node, pred)) {
            compareAndSetNext(pred, predNext, null);
        } else {
            // If successor needs signal, try to set pred's next-link
            // so it will get one. Otherwise wake it up to propagate.
            int ws;
            if (pred != head &&
                ((ws = pred.waitStatus) == Node.SIGNAL ||
                 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                pred.thread != null) {
                Node next = node.next;
                if (next != null && next.waitStatus <= 0)
                	//因为前面有可能都是取消的。prenext也是取消的。因此要设置成node的next
                    compareAndSetNext(pred, predNext, next);
            } else {
                unparkSuccessor(node);
            }

            node.next = node; // help GC
        }
    }
    
    
```