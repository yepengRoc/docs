EventLoopGroup



```java
//继承于EventExecutorGroup
/**
进行channel注册。
在事件循环组内，获取select上下一个事件（稍后的事件）
**/

interface EventLoopGroup extends EventExecutorGroup
/**
通过next() 方法返回 EventExecutor。
并在负责维护 EventExecutor的生命周期，在全局范围内
**/
interface EventExecutorGroup extends ScheduledExecutorService, Iterable<EventExecutor>
/**
定时执行任务 
**/
interface ScheduledExecutorService extends ExecutorService
/**
提供终止任务的方法，通过返回 Future，追踪一个或多个异步任务的执行
shutdown 方法允许之前提交的任务执行完后再关闭。shutdownNow尝试关闭正在执行的任务。
**/
interface ExecutorService extends Executor
    
public interface Executor

```



EventExecutor

```java
/**
通过此可方便插线事件循环组内线程的执行。并提供通用的方式进行访问
**/
interface EventExecutor extends EventExecutorGroup
```