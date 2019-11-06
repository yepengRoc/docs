```java
ReentrantReadWriteLock rwLokc = new ReentrantReadWriteLock();
```

```java
//构造函数
public ReentrantReadWriteLock() {
        this(false);
    }
/**
默认false。使用非公平同步方式。
也可以通过构造函数传入truenew ReentrantReadWriteLock(true)使用公平方式
**/
public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);
        writerLock = new WriteLock(this);
    }
```

公平

```java
    /**
     * Fair version of Sync
     */
    static final class FairSync extends Sync {
        final boolean writerShouldBlock() {
            return hasQueuedPredecessors();
        }
        final boolean readerShouldBlock() {
            return hasQueuedPredecessors();
        }
    }
```

非公平

```java
 /**
     * Nonfair version of Sync
     */
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = -8159625535654395037L;
        //非公平模式下，写 需要阻塞
        final boolean writerShouldBlock() {
            return false; // writers can always barge
        }
        final boolean readerShouldBlock() {
            /* 
             队列的的第一个元素是否是独占式。如果是独占式则读锁应该阻塞
             */
            return apparentlyFirstQueuedIsExclusive();
        }
    }
```

公平

```java
/**
通过查看是否有前驱来决定是否阻塞
**/
static final class FairSync extends Sync {
        private static final long serialVersionUID = -2274990926593161451L;
        final boolean writerShouldBlock() {
            return hasQueuedPredecessors();
        }
        final boolean readerShouldBlock() {
            return hasQueuedPredecessors();
        }
    }
```

通过以上可以看出公平和非公平的两个类的实现了各自的writerShouldBlock和readerShouldBlock方法，其它调用逻辑统一在Sync类中实现

## readLock实现

```java
Lock readLock = rwLokc.readLock();
readLock.lock();
....
readLock.unlock();
```

### 加锁

```java
/**
通过调用AQS的acquireShared获取执行权
**/
public void lock() {
            sync.acquireShared(1);
        }
/**
调用当前类中Sync类中的tryAcquireShared 方法
**/
protected final int tryAcquireShared(int unused) {
            /*
            如果state 被其它线程以写锁的方式持有，则读锁获取失败
             * Walkthrough:
             * 1. If write lock held by another thread, fail.
             * 2. Otherwise, this thread is eligible for
             *    lock wrt state, so ask if it should block
             *    because of queue policy. If not, try
             *    to grant by CASing state and updating count.
             *    Note that step does not check for reentrant
             *    acquires, which is postponed to full version
             *    to avoid having to check hold count in
             *    the more typical non-reentrant case.
             * 3. If step 2 fails either because thread
             *    apparently not eligible or CAS fails or count
             *    saturated, chain to version with full retry loop.
             */
            Thread current = Thread.currentThread();
            int c = getState();
    		/**
    		有写锁，且持有写锁的线程不是当前线程
    		**/
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;
            int r = sharedCount(c);
    		//当前线程不应该阻塞。设置写锁记数。因为写锁占用高16位，所以这里加一个SHARED_UNIT
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                //如果还未有读锁记数。这里记录第一个读锁线程
                if (r == 0) {
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {//读锁可重入，获取读锁次数自增
                    firstReaderHoldCount++;
                } else {//firstReader记录线程以外的线程获取读锁
                    HoldCounter rh = cachedHoldCounter;
                    //未有对应记录。或者记录中的线程 和 当前获取读锁的线程不是同一个
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;//持有读锁的次数加1
                }
                return 1;
            }
            return fullTryAcquireShared(current);
        }
		/**
		实体记录当前写锁的线程id ,和线程获取写锁的次数
		**/
 		static final class HoldCounter {
            //记录线程获取写锁的次数
            int count = 0;
            //记录线程id. 使用线程id，而不使用线程引用，是为了避免jvm垃圾回收滞留的问题
            final long tid = getThreadId(Thread.currentThread());
        }
		/**
		readHolds变量是ThreadLocalHoldCounter类，是一个ThreadLocal
		**/
	 static final class ThreadLocalHoldCounter
            extends ThreadLocal<HoldCounter> {
     		/**
     		重写初始化值的方法，如果没有值，则实例化一个HoldCounter
     		**/
            public HoldCounter initialValue() {
                return new HoldCounter();
            }
        }
 /**
         * Full version of acquire for reads, that handles CAS misses
         * and reentrant reads not dealt with in tryAcquireShared.
         */
        final int fullTryAcquireShared(Thread current) {
            /*
             * This code is in part redundant with that in
             * tryAcquireShared but is simpler overall by not
             * complicating tryAcquireShared with interactions between
             * retries and lazily reading hold counts.
             */
            HoldCounter rh = null;
            for (;;) {
                int c = getState();
                //有写锁且不是当前线程
                if (exclusiveCount(c) != 0) {
                    if (getExclusiveOwnerThread() != current)
                        return -1;
                  //如果当前线程阻塞。公平模式下 有前驱。非公平模式下 头结点是独占式
                } else if (readerShouldBlock()) {
                    // Make sure we're not acquiring read lock reentrantly
                    if (firstReader == current) {
                        // assert firstReaderHoldCount > 0;
                    } else {
                        if (rh == null) {
                            rh = cachedHoldCounter;
                            if (rh == null || rh.tid != getThreadId(current)) {
                                rh = readHolds.get();//会new 一个readhold
                                if (rh.count == 0)//如果为0.则移除
                                    readHolds.remove();
                            }
                        }
                        if (rh.count == 0)
                            return -1;
                    }
                }
                if (sharedCount(c) == MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                if (compareAndSetState(c, c + SHARED_UNIT)) {
                    if (sharedCount(c) == 0) {
                        firstReader = current;
                        firstReaderHoldCount = 1;
                    } else if (firstReader == current) {
                        firstReaderHoldCount++;
                    } else {
                        if (rh == null)
                            rh = cachedHoldCounter;
                        if (rh == null || rh.tid != getThreadId(current))
                            rh = readHolds.get();
                        else if (rh.count == 0)
                            readHolds.set(rh);
                        rh.count++;
                        cachedHoldCounter = rh; // cache for release
                    }
                    return 1;
                }
            }
        }

```

### 解锁

```java
/**
调用aqs的releaseShared 方法
**/
public void unlock() {
            sync.releaseShared(1);
        }
/**
调用当前类中Sync中的tryReleaseShared 方法
**/
protected final boolean tryReleaseShared(int unused) {
            Thread current = Thread.currentThread();
    		//当前释放线程是第一个线程，持有锁次数为1，置第一个读线程为null。否则持有锁次数减1
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
                if (firstReaderHoldCount == 1)
                    firstReader = null;
                else
                    firstReaderHoldCount--;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                int count = rh.count;
                if (count <= 1) {
                    readHolds.remove();
                    if (count <= 0)
                        throw unmatchedUnlockException();
                }
                --rh.count;
            }
            for (;;) {
                int c = getState();
                int nextc = c - SHARED_UNIT;
                if (compareAndSetState(c, nextc))
                    // Releasing the read lock has no effect on readers,
                    // but it may allow waiting writers to proceed if
                    // both read and write locks are now free.
                    return nextc == 0;
            }
        }

```

