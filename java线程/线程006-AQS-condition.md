## 线程等待-唤醒

### 线程等待

```java
/**
线程等待。是释放执行权的
**/
public final void await() throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
            Node node = addConditionWaiter();
    		//释放执行权，进行后续节点唤醒
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            while (!isOnSyncQueue(node)) {
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                //中断
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                //清理取消节点
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                //从新中断
                reportInterruptAfterWait(interruptMode);
        }
 
 
```

fullyRelease(node) 分析

```java
final int fullyRelease(Node node) {
        boolean failed = true;
        try {
            int savedState = getState();
            //release失败，则抛出异常。如果当前节点已经执行完逻辑了。则会进入取消。还有其它可能
            if (release(savedState)) {
                failed = false;
                return savedState;
            } else {
                throw new IllegalMonitorStateException();
            }
        } finally {
            //置当前节点为取消状态
            if (failed)
                node.waitStatus = Node.CANCELLED;
        }
    }
```

isOnSyncQueue 分析

```java
/**
这个时候node 还是一个aqs节点。判断是否在同步队列中
**/
final boolean isOnSyncQueue(Node node) {
        if (node.waitStatus == Node.CONDITION || node.prev == null)
            return false;
        if (node.next != null) // If has successor, it must be on queue
            return true;
        /*
         * node.prev can be non-null, but not yet on queue because
         * the CAS to place it on queue can fail. So we have to
         * traverse from tail to make sure it actually made it.  It
         * will always be near the tail in calls to this method, and
         * unless the CAS failed (which is unlikely), it will be
         * there, so we hardly ever traverse much.
         */
        return findNodeFromTail(node);
    }

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

checkInterruptWhileWaiting(node) 方法分析

```java
private int checkInterruptWhileWaiting(Node node) {
            return Thread.interrupted() ?
                (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
                0;
        }

final boolean transferAfterCancelledWait(Node node) {
        if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
            enq(node);
            return true;
        }
        /*
         * If we lost out to a signal(), then we can't proceed
         * until it finishes its enq().  Cancelling during an
         * incomplete transfer is both rare and transient, so just
         * spin.
         */
        while (!isOnSyncQueue(node))
            Thread.yield();
        return false;
    }
        
```



```java

/**
和AQs阻塞队列底层使用的node节点一样
如果当前线程是第一个触发等待，则构建node节点
如果不是第一个触发等待，则执行 unlinkCancelledWaiters
AQS是双向非循环节点。condition是单向非循环队列
aqs通过node节点中的next pre构建双向队列，nextWaiter记录当前是独占还是共享模式
condition通过node中的nextwaiter构建单向队列
**/
private Node addConditionWaiter() {
            Node t = lastWaiter;
            // If lastWaiter is cancelled, clean out.
            if (t != null && t.waitStatus != Node.CONDITION) {
                unlinkCancelledWaiters();
                t = lastWaiter;
            }
            Node node = new Node(Thread.currentThread(), Node.CONDITION);
            if (t == null)
                firstWaiter = node;
            else
                t.nextWaiter = node;
            lastWaiter = node;
            return node;
        }
/**
去除掉condition队列中取消的节点
**/
private void unlinkCancelledWaiters() {
            Node t = firstWaiter;
            Node trail = null;//此节点作为一个指针备份，往后不断遍历
            while (t != null) {
                //获取首节点的下一个等待节点
                Node next = t.nextWaiter;
                /**
                首节点状态不是-2 说明被取消
                从新设置首节点。如果一直不为-2，则整个对列都会遍历完。如果  
                       为-2 则走else逻辑 trail = t。之后还有取消，则走else逻  
                       trail.nextWaiter = next。剔除掉队列中所有取消节点
                **/
                if (t.waitStatus != Node.CONDITION) {
                    t.nextWaiter = null;
                    if (trail == null)
                        firstWaiter = next;
                    else
                        trail.nextWaiter = next;
                    if (next == null)
                        lastWaiter = trail;
                }
                else//首节点状态为-2.则通过trail 记录当前操作节点
                    trail = t;
                t = next;
            }
        }
```





### 线程唤醒-一次唤醒一个

```java
 public final void signal() {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node first = firstWaiter;
            if (first != null)
                doSignal(first);
        }
/**
从头节点开始唤醒。如果唤醒成功。则终止。头结点唤醒失败，则不停往后查找可以唤醒的节点
**/
 private void doSignal(Node first) {
            do {
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
               	    first.nextWaiter = null;
            } while (!transferForSignal(first) &&
                     (first = firstWaiter) != null);
        }
/**
**/
final boolean transferForSignal(Node node) {
        /*
         * If cannot change waitStatus, the node has been cancelled.
         如果不能改变等待节点的状态，则节点被取消。设置节点状态为0。
         这里不能设置成功，说明节点的状态已经改变了
         */
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;

        /*
         * Splice onto queue and try to set waitStatus of predecessor to
         * indicate that thread is (probably) waiting. If cancelled or
         * attempt to set waitStatus fails, wake up to resync (in which
         * case the waitStatus can be transiently and harmlessly wrong).
         通过aqs的入队方法enq 把等待节点加入aqs队列。
         如果当前节点取消了，或者当前节点的状态为-1(争夺执行权合法状态)
         则进行当前节点的唤醒动作。
         */
    
        Node p = enq(node);
        int ws = p.waitStatus;
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }

```

### 唤醒线程-唤醒所有

```java
  public final void signalAll() {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node first = firstWaiter;
            if (first != null)
                doSignalAll(first);
        }
/**
和单个唤醒一样。只是这里是遍历所有节点都唤醒一遍
**/
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

