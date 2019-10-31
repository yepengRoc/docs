reetrantlock是一个可重入的互斥锁。功能和synchronized一样，但是更灵活强大

## 构造方法

```java
	/**
	默认无参构造方法，使用的是非公平同步方式实现
	**/
	public ReentrantLock() {
        sync = new NonfairSync();
    }
    /**
    	有参构造方法。传入true则使用公平同步方式实现，false 非公平方式实现。
    	非公平同步方式下，效率会更高，因为可以减少线程阻塞和唤醒的上下文切换，但是容易造成线程饥饿
    	公平方式下，效率不高，但是能保证每个线程都能执行到
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

## 同步的基础AQS

```java
/**
内部抽象类Sync 继承AQS,实现通用方法的封装
**/
abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;
        /**
         * 留给子类实现
         */
        abstract void lock();
        /**
         * 非公平方式获取锁。没有任何前驱判断
         */
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
		//锁的释放
        protected final boolean tryRelease(int releases) {
            //持有锁的次数减1
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            //持有锁的次数为0，标识没有线程占有锁。锁完全释放
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
		//是否当前线程持有锁
        protected final boolean isHeldExclusively() {
            // While we must in general read state before owner,
            // we don't need to do so to check if current thread is owner
            return getExclusiveOwnerThread() == Thread.currentThread();
        }
		//AQS中的ConditionObject 对象
        final ConditionObject newCondition() {
            return new ConditionObject();
        }

        // Methods relayed from outer class
		
        final Thread getOwner() {
            return getState() == 0 ? null : getExclusiveOwnerThread();
        }
		//锁被持有的次数
        final int getHoldCount() {
            return isHeldExclusively() ? getState() : 0;
        }
		//锁是否是被线程持有状态
        final boolean isLocked() {
            return getState() != 0;
        }

        /**
         * 序列化的时候使用。
         */
        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }
```



## 公平同步方式

```java
 static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;
		//调用AQS的acquire 进行锁（执行权）的获取
        final void lock() {
            acquire(1);
        }

        /**
        	tryAcquire 由自定义锁的实现类进行实现
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();//获取当前锁被获取的次数
            if (c == 0) {
                /**
                检查是否有比自己更早的节点。
                没有的话，通过cas设置锁获取次数为acquires，设置锁持有者为当前线程
                **/
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            //如果当前锁持有的次数大于0，且持有锁的线程是当前线程。因为锁可以重入，所以这里继续对
            //锁的持有次数加1
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

## 非公平同步方式

```java
static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * 直接尝试获取执行权，获取失败则入等待队列。没有任何前驱判断
         */
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

