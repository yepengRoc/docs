CyclicBarrier 字面意思循环障碍。是说可以循环利用

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
/**
在对应线程的逻辑中调用
**/
class 线程 implements Runnable {
	 public void run() {
         try {
           cyclicBarrier.await();
           //业务代码
          } catch (InterruptedException ex) {
            return;
         } catch (BrokenBarrierException ex) {
           return;
         }
       }
    }
}


```

## 构造函数和参数说明

```java
 	/**
 	Generation 英文意思是代。
 	count为0时，重置count为parties，表示新一轮循环的开始。会从新 new Generation
 	此类中有一个参数 broken 中断或 count为0。或调用 reset方法时进行赋值
 	**/
private static class Generation {
        boolean broken = false;
    }

    /** 底层通过reentrantlock实现*/
    private final ReentrantLock lock = new ReentrantLock();
    /** 线程阻塞在condition上*/
    private final Condition trip = lock.newCondition();
    /** cyclicbarrier初始化时，需要指定一次循环要阻塞的线程。这里用final修饰
    一旦赋值则不可变。
    final赋值要么定义变量的时候指定值；要么在构造函数中赋值
    */
    private final int parties;
    /* 达到阻塞线程数 parties时，在唤醒所有线程之前执行的动作，是一个线程*/
    private final Runnable barrierCommand;
    /** generation 记录当前代的终端情况 */
    private Generation generation = new Generation();

    /**
     * parties值的拷贝，当count--为0时，表示需要唤醒所有阻塞线程。
     重置count为 parties，开始新一轮或新一代的循环使用
     */
    private int count;

public CyclicBarrier(int parties) {
        this(parties, null);
    }
/**
parties 记录当前障碍阻塞的线程数，当阻塞线程数等于parties，阻塞线程才开始run各自的业务逻辑。
	parties的值设置以后不会变化，当阻塞线程都开始执行后，重置count的值为parties
barrierAction 当阻塞线程数等于parties，阻塞线程开始run之前执行的动作
count 阻塞记数。未有阻塞，则 count = parties 。如果当阻塞线程数等于parties count=0.
	这里是通过判断count=0 来启动所有阻塞线程。
**/
  public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }
```

## 阻塞方法

```java
public int await() throws InterruptedException, BrokenBarrierException {
        try {
            return dowait(false, 0L);
        } catch (TimeoutException toe) {
            throw new Error(toe); // cannot happen
        }
    }
/**
底层通过reentrantlock实现
**/
private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
          
            final Generation g = generation;
						//本次循环被中断了
            if (g.broken)
                throw new BrokenBarrierException();
						//线程被中断
            if (Thread.interrupted()) {
                breakBarrier();
                throw new InterruptedException();
            }
						//count减1
            int index = --count;
          	//为0表示阻塞线程数达到设置的值，开始唤醒所有线程
            if (index == 0) {  // tripped
                boolean ranAction = false;
                try {
                  //唤醒所有线程前的一个动作，也是一个线程
                    final Runnable command = barrierCommand;
                    if (command != null)
                        command.run();
                    ranAction = true;
                    nextGeneration();
                    return 0;
                } finally {
                  //线程非正常结束，则执行障碍中断
                    if (!ranAction)
                        breakBarrier();
                }
            }

          	//count不为0时，则进行阻塞操作。通过condition的await方法进行阻塞
            for (;;) {
                try {
                    if (!timed)
                        trip.await();
                    else if (nanos > 0L)
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                  //如果阻塞过程中线程被别的线程调用中断方法，且在当前轮循环中，没有被中断过
                  //设置当前代中断标示。唤醒所有线程。 其它情况则执行线程中断操作
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        // We're about to finish waiting even if we had not
                        // been interrupted, so this interrupt is deemed to
                        // "belong" to subsequent execution.
                        Thread.currentThread().interrupt();
                    }
                }
							//当前代中断
                if (g.broken)
                    throw new BrokenBarrierException();
							//当前代已更换
                if (g != generation)
                    return index;

                if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            lock.unlock();
        }
    }
/**
开始下一次循环。唤醒所有线程。重置count的值，标示新的一代
**/
private void nextGeneration() {
        // signal completion of last generation
        trip.signalAll();
        // set up next generation
        count = parties;
        generation = new Generation();
    }
/**
中断操作，设置当前循环的中断标示。唤醒所有阻塞线程
**/
  private void breakBarrier() {
        generation.broken = true;
        count = parties;
        trip.signalAll();
    }
```

