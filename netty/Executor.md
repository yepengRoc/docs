

```java
/**
执行runnable任务。
此接口定义一种任务的运行机制，使得线程的创建 和线程的调度 进行解耦。
通常创建线程任务的过程是
new Thread(new Runnable{
public void run(){
	业务逻辑
}).start();
新的任务执行方式。这里只关心任务执行的逻辑，不关心任务线程的创建。让执行逻辑和线程创建  
进行解耦
executor.execute(new RunnableTask());
RunnableTask 不一定非要是线程执行的内容，也可以是一个正常的方法调用

更加典型的用法是，在提交线程的任务中再次创建一个线程用来执行任务。
  class ThreadPerTaskExecutor implements Executor {
    public void execute(Runnable r) {
      new Thread(r).start();
    }
  }}
  
  还有一些实现一些排序限制当执行任务的时候，示例 顺序提交任务到第二个 second，展示
  出一种复合执行
  class SerialExecutor implements Executor {
  final Queue<Runnable> tasks = new ArrayDeque<Runnable>();
  final Executor executor;
  Runnable active;

  SerialExecutor(Executor executor) {
    this.executor = executor;
  }

  public synchronized void execute(final Runnable r) {
    tasks.offer(new Runnable() {
      public void run() {
        try {
          r.run();
        } finally {
          scheduleNext();
        }
      }
    });
    if (active == null) {
      scheduleNext();
    }
  }

  protected synchronized void scheduleNext() {
    if ((active = tasks.poll()) != null) {
      executor.execute(active);
    }
  }
}}
  
  ExecutorService 是一个广泛使用的接口
  ThreadPoolExecutor 提供可扩展的线程池实现
  Executors 提供了便捷的工厂方法
  **、

public interface Executor
```

