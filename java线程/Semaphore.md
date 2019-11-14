# Semaphore

Semaphore信号量。通过构造函数传值（num）,允许同时有num个线程进行执行。这里的num可翻译为许可，  只有获取许可的线程才能执行,未获取许可的线程则阻塞（待确定），只有获取许可的线程释放许可，阻塞线程才  可执行

## 构造函数

```java
Semaphore semaphore = new Semaphore(2);
/**
通过构造函数传值，初始化一个Semaphore 有多少许可。默认是非公平方式
通过第二个参数传入 boolean值来控制使用公平方式获取许可，还是非公平方式获取许可
**/
public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }
    
public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

## 底层实现

```java
/**
继承AQS，重写nonfairTryAcquireShared  tryReleaseShared 方法
**/
abstract static class Sync extends AbstractQueuedSynchronizer {
 
        Sync(int permits) {
            setState(permits);
        }

        final int getPermits() {
            return getState();
        }
		/**
		非公平方式获取需求。不判断是否有前驱，直接对state值做处理
		**/
        final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }

        protected final boolean tryReleaseShared(int releases) {
            for (;;) {
                int current = getState();
                int next = current + releases;
                if (next < current) // overflow
                    throw new Error("Maximum permit count exceeded");
                if (compareAndSetState(current, next))
                    return true;
            }
        }
		//减少许可
        final void reducePermits(int reductions) {
            for (;;) {
                int current = getState();
                int next = current - reductions;
                if (next > current) // underflow
                    throw new Error("Permit count underflow");
                if (compareAndSetState(current, next))
                    return;
            }
        }
		//许可清0
        final int drainPermits() {
            for (;;) {
                int current = getState();
                if (current == 0 || compareAndSetState(current, 0))
                    return current;
            }
        }
    }

    /**
     * 非公平方式。
     */
    static final class NonfairSync extends Sync {
        NonfairSync(int permits) {
            super(permits);
        }
		/**
		获取许可调用的是 Sync中的 nonfairTryAcquireShared 方法
		**/
        protected int tryAcquireShared(int acquires) {
            return nonfairTryAcquireShared(acquires);
        }
    }

    /**
     * 贡品方式
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = 2014338818796000944L;

        FairSync(int permits) {
            super(permits);
        }
		/**
		重写tryAcquireShared
		**/
        protected int tryAcquireShared(int acquires) {
            for (;;) {
                //如果当前线程有前驱，则获取许可失败。所谓公平就是先进先出   
                if (hasQueuedPredecessors())
                    return -1;
                //当前许可数量
                int available = getState();
                //减去当前获取的许可数
                int remaining = available - acquires;
                /**
                如果剩余许可 小于0 或者 设置state许可数成功，
                则返回。
                如果没有许可了，这个时候state是0，再次调用获取
                许可的方法则 remaining 就为负数了
                **/
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
    }
```

## 获取许可

```java
 semaphore.acquire();
/**
调用aqs中的acquireSharedInterruptibly 方法。在此方法中会调用  
Semaphore 内部类FairSync NonfairSync 中 tryAcquireShared 方法
**/
public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
```

## 释放许可

```java
/**
调用aqs中的acquireSharedInterruptibly 方法。在此方法中会调用  
Semaphore 内部类Sync 中 tryReleaseShared 方法
**/
public void release() {
        sync.releaseShared(1);
    }
```

