---
typora-root-url: ..\image
---

## 排它锁

### 非公平锁获取锁

#### 锁的获取



```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
```

通过lock来查看非公平锁的实现

lock.lock()调用方法

```java
sync.lock();
```

调用静态内部类NonfairSync 中的lock方法

```java
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

```java
/**
如果尝试获取锁失败，且添加进等待队列失败。则中断
**/
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();//中断如果没有中断处理，则线程会正常处理完。走释放流程
}
```

```java
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```

```java
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

```java
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
                return interrupted;//如果线程阻塞，则执行真正的阻塞
            }
            /**
            shouldParkAfterFailedAcquire
            1前驱节点不是头节点。
            2前驱节点是头节点。但是获取锁失败了
              前驱节点是头节点说明已经是获取锁的状态，不可能是取消状态
              说明前驱节点还没有释放锁。前驱如果没有被取消。如果状态是0 则失败，继续循环，如果不为0则返回true. 
            parkAndCheckInterrupt 通过jdk native方法。阻塞当前线程 底层0就是永久阻塞
            如果线程在阻塞的时候被中断。则是真的中断。-- 温故
            是lock.interrupt
            还是thread.interrupt会抛异常。
            wait 的时候会抛异常。并清除中断位
            
            还有unpark方法可以先调用。即便这个时候
            https://www.jianshu.com/p/ceb8870ef2c5 博客讲解park方法
            还没有执行park,如果这个时候执行park，则线程不会阻塞，直接继续执行，因为先行执行过一次uppark
            这里parkAandCheckInterrupt 在park的时候，如果先行执行了uppark,则这里不会进行阻塞，
            继续执行。
            如果阻塞的过程 线程执行了interrupt,则会抛异常。如果没有阻塞，则返回真实的阻塞状态
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
    /**
    
    **/
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

### 非公平锁的释放

```java
public void unlock() {
    sync.release(1);
}
/**
尝试释放锁。如果释放成功
如果当前节点是头节点。且当前节点的状态是-1
-1表明当前节点被阻塞过，相对应的next的也被阻塞
所以要通过upark释放唤醒后续节点
**/
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
尝试释放节点。如果节点是初始化状态。则直接更新全局状态为0
后续加入线程可进行锁的竞争
**/
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
/**
如果当前节点是取消状态。则从尾部开始查找第一个不为取消的节点
个人理解：因为线程是实时变化的。所以从此刻的尾部开始查找。
有可能next已经获取成功了，不需要unpark.
即便状态等于0
因为当前头节点还在执行h.waitstatus != 0
所以next的节点肯定最后会进行 -1设置。通过unpark进行唤醒
**/
    private void unparkSuccessor(Node node) {
       
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }

```

### 公平锁

公平锁的获取和释放和非公平锁一样，只是严格按照先进先出进行处理的

## 共享锁

### 锁的获取

```java
/**
共享锁-核心方法
tryAcquireShared 由子类进行实现
doAcquireSharedInterruptibly 统一实现
**/
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}
  
private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {//如果前面的节点是头节点
                    int r = tryAcquireShared(arg);//如果许可大于0。通过release释放了许可
                    if (r >= 0) {
                        //阻塞在头节点的下一个节点则会设置自己为头结点。并唤醒自己的后继节点
                        setHeadAndPropagate(node, r);//设置当前节点为头节点，并唤醒后继节点
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
private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head; // Record old head for check below
        setHead(node);
        //h == null 队列初始化的时候
        if (propagate > 0 || h == null || h.waitStatus < 0) {
            Node s = node.next;
            if (s == null || s.isShared())
                doReleaseShared();
        }
    }
```

### 锁的释放

```java
public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
/**
如果后继节点是 -1，预备状态，则唤醒后继节点。

如果这个时候没有许可。h== head ,则跳出
后节点阻塞于
int r = tryAcquireShared(arg);//如果许可大于0 
等待许可的位置，一直空循环。直到有释放。后继节点抢占head位置，唤醒自己的后续

如果这个时候有许可，则不停的唤醒 后继节点，后继节点设置自己为头节点，
所以h == head 可能一直为false。
**/
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```

通过countdownlatch来查看下公平锁的实现

```java
CountDownLatch countDownLatch = new CountDownLatch();
countDownLatch.await();//执行此方法后线程都阻塞
countDownLatch.countDown();//执行此方法后。知道全局变量state=0 线程才全部启动
```

### 获取锁

```java
countDownLatch.await();
  public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }


```

### 释放锁

```java
countDownLatch.countDown();
 public void countDown() {
        sync.releaseShared(1);
    }
    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }

```

这里通过分析Semaphore 来分析共享锁

## conditon

### Condition.await()

```java
/***
线程进入等待
是一个单项列表
condition是在lock.lock()中操作，
说明此时的线程已经获取了锁。
await需要释放 资源
通过while循环使线程阻塞
**/
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
  	//将线程放到等待队列 队尾
    Node node = addConditionWaiter();
  //释放线程持有的锁，并唤醒后继线程
    int savedState = fullyRelease(node);
    int interruptMode = 0;
  /**
  如果线程没有进入等待同步队列。则在condition构建的等待队列上
  则进入while结构进行阻塞
  **/
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
  //说明中断了。如果不是在clh中 中断。则设置中断模式为 condition上的中断
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
/**
判断线程在clh 队列 还是信号队列
**/
 final boolean isOnSyncQueue(Node node) {
        if (node.waitStatus == Node.CONDITION || node.prev == null)
            return false;
   //node.waitestatus != node.condition 
        if (node.next != null) // If has successor, it must be on queue
            return true;
       //从clh队列尾部开始查找。如果找到则返回true 否则返回false
        return findNodeFromTail(node);
    }
//查找逻辑
private boolean findNodeFromTail(Node node) {
        Node t = tail;
        for (;;) {
            if (t == node)
                return true;
            if (t == null)
                return false;
            t = t.prev;
        }
    }
```

### Condition.single()

```java
/**
如果不是独占锁，则抛出异常。如果占用锁的线程 不是当前线程
**/
public final void signal() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
        private void doSignal(Node first) {
            do {
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                first.nextWaiter = null;
            } while (!transferForSignal(first) &&
                     (first = firstWaiter) != null);
        }
    final boolean transferForSignal(Node node) {
       
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;

        Node p = enq(node);
        int ws = p.waitStatus;
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }
```

### Condition.singleAll()

```java
public final void signalAll() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignalAll(first);
}
private void doSignalAll(Node first) {
            lastWaiter = firstWaiter = null;
            do {
                Node next = first.nextWaiter;
                first.nextWaiter = null;
                transferForSignal(first);
                first = next;
            } while (first != null);
        }
```