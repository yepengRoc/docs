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
          } catch (InterruptedException ex) {
            return;
         } catch (BrokenBarrierException ex) {
           return;
         }
       }
    }
}


```

构造函数

```java
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

