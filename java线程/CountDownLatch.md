CountDownLatch 在构造函数里设置一个数字，当调用

## 构造函数

```java
/**
通过构造函数设置AQS中的state值
**/
CountDownLatch countDownLatch = new CountDownLatch(5);
public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }
 Sync(int count) {
            setState(count);
        }
```

## 底层实现

```java
/**
这里实现了Aqs，复写了 tryAcquireShared 和 tryReleaseShared 
通过这里可以看出。CountDownLatch是一个共享式锁
**/
private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }
		/**
		获取执行权的时候，如果AQS中的state值 为0，则成功，否则则失败
		**/
        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }
		/**
		阻塞线程唤醒。开始执行
		**/
        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                /**
                等于0的时候说明 state已经被其它线程通过countDown调用处理为0了。  
                当线程再次调用countDown 时，则调用失败，因为state的值已经为0 了 
                这里也说明 new CountDownLatch(数字) 的时候，构造函数里的数字
                不能设置为0，要不一直是失败状态，线程无法执行
                **/
                if (c == 0)
                    return false;
                int nextc = c-1;
                /**
                当state 由非0数字修改为0，则返nextc == 0 返回true,所有阻塞线程执行
                **/
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }
```

## countDown ()方法讲解

```java
countDownLatch.countDown();
/**
调用aqs的 releaseShared方法，在这个方法里，会调用CountDownLatch 中内部类Sync 中的tryReleaseShared
 方法
**/
public void countDown() {
        sync.releaseShared(1);
    }
```

### await() 方法讲解

```java
/**
调用aqs的doAcquireSharedInterruptibly，在这个方法里  
会调用CountDownLatch 中内部类Sync 中的tryReleaseShared
**/
public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
```



## 总结

CountDownLatch 在构造的时候传入了一个数字 设置aqs中states的值，此数字的值不可重置，减为0 后就不能再使用了，不可循环利用。

使用的时候调用 await 方法，使线程进行阻塞，然后再调用 countDown 使 aqs中states 减1，直到减为0，唤醒所有线程。可以用来使所有线程一起工作，也可以用来使所有线程一起结束