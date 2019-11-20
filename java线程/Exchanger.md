

Exchanger 数据交换，一个线程放数据，一个线程取数据。最多只能有一个数据，不会挤压多余一个的数据

## 构造函数

```java
public Exchanger() {
        participant = new Participant();
    }
/**
Participant 是一个ThreadLocal 记录和当前线程相关的数据。
默认生成一个node
**/
 static final class Participant extends ThreadLocal<Node> {
        public Node initialValue() { return new Node(); }
    }
/**
@sun.misc.Contended 防止多余一个的变量放到同一个缓存行上
**/
@sun.misc.Contended static final class Node {
        int index;              // Arena index 数组中的索引
        int bound;              // Last recorded value of Exchanger.bound
        int collides;           // cas失败的次数
        int hash;               // 随机数 做自旋使用
        Object item;            // 设置值的线程
        volatile Object match;  // 记录需要置换的值，在取数线程里进行赋值
        volatile Thread parked; // 记录需要阻塞的线程
    }

```

## exchange 方法

```java
/**
exchange 要么是取值，要么是放值。if条件里的操作
(arena != null ||(v = slotExchange(item, false, 0L)) == null)
第一次操作exchange 的时候，arena是null,则执行slotExchange方法，如果返回null，则执行
(Thread.interrupted() ||(v = arenaExchange(item, false, 0L)) == null))
线程如果没有中断，则执行arenaExchange 方法
**/
public V exchange(V x) throws InterruptedException {
        Object v;
        Object item = (x == null) ? NULL_ITEM : x; // translate null args
        if ((arena != null ||
             (v = slotExchange(item, false, 0L)) == null) &&
            ((Thread.interrupted() || // disambiguates null return
              (v = arenaExchange(item, false, 0L)) == null)))
            throw new InterruptedException();
        return (v == NULL_ITEM) ? null : (V)v;
    }
```

### slotExchange方法

```java
 private final Object slotExchange(Object item, boolean timed, long ns) {
        Node p = participant.get();//获取当前线程中的Node
        Thread t = Thread.currentThread();
        if (t.isInterrupted()) // preserve interrupt status so caller can recheck
            return null;
		//死循环
        for (Node q;;) {
            //第一次 slot 肯定为null。如果放值之后 slot不为null
            if ((q = slot) != null) {
                /**
                如果有其它线程调用exchange进行取值，且置换成功了.  
                置slot为null,取slot中记录的值
                **/
                if (U.compareAndSwapObject(this, SLOT, q, null)) {
                    Object v = q.item;
                    q.match = item;//在取值线程取值过程中，记录当前要被置换出去的值
                    /**
                    如果有阻塞线程，则进行阻塞线程释放.
                    赋值的线程，赋值以后，如果没有取值线程，则赋值线程可能会阻塞，这里记录的
                    就是阻塞的赋值线程，需要在这个时候对赋值线程进行释放
                    **/
                    Thread w = q.parked;
                    if (w != null)
                        U.unpark(w);//进行阻塞线程释放
                    return v;
                }
                //有放值的线程，也有取值的线程。 多个线程进行置换，总有失败的，失败的走这里的逻辑。
                //之后继续执行for循环
                // create arena on contention, but continue until slot null
                if (NCPU > 1 && bound == 0 &&
                    U.compareAndSwapInt(this, BOUND, 0, SEQ))
                    //新建一个数组
                    arena = new Node[(FULL + 2) << ASHIFT];
            }
            //arena 不为null,则直接返回
            else if (arena != null)
                return null; // caller must reroute to arenaExchange
            else {
                //第一次 slot肯定是null
                p.item = item;
                /**
                将slot的值由null置换为p  SLOT记录的是变量slot的偏移量
                置换成功结束for循环，失败的话，接着循环。
                失败的话也说明有其它线程已经抢先一步compareAndSwapObject 成功了
                **/
                if (U.compareAndSwapObject(this, SLOT, null, p))
                    break;
                p.item = null;
            }
        }
		/**
		线程放 值成功，等待被释放。只有取值线程取值，放值的线程才会被释放
		**/
        // await release
        int h = p.hash;
        long end = timed ? System.nanoTime() + ns : 0L;
     	/**
     	自旋次数。如果赋值以后，没有线程来取值，则不会立即阻塞赋值的线程，会进行
     	一定次数的自旋，避免线程状态的切换（比较消耗资源）。如果自旋一定次数后，还
     	没有线程来取值，则进行阻塞
     	自旋次数，如果cpu核心数大于1 则自旋1024此，否则自旋一次
     	**/
        int spins = (NCPU > 1) ? SPINS : 1;
        Object v;
     	//p.match 不为null的时候，说明取值线程已取值成功，可退出循环
        while ((v = p.match) == null) {
            if (spins > 0) {
                h ^= h << 1; h ^= h >>> 3; h ^= h << 10;
                if (h == 0)
                    h = SPINS | (int)t.getId();
                else if (h < 0 && (--spins & ((SPINS >>> 1) - 1)) == 0)
                    Thread.yield();//线程让出cpu资源，进入就绪状态
            }
            //slot 不等于 p 说明正在发生置换动作，这里从新赋值自旋次数
            else if (slot != p)
                spins = SPINS;
            /**
            自旋次数用完了。线程没有中断，arena为null,不存在线程冲突。
            也没有超时
            **/
            else if (!t.isInterrupted() && arena == null &&
                     (!timed || (ns = end - System.nanoTime()) > 0L)) {
                //阻塞当前线程
                U.putObject(t, BLOCKER, this);
                p.parked = t;//记录当前阻塞的线程
                if (slot == p)//说明没有取值线程。则进行阻塞
                    U.park(false, ns);
                /**
                //线程超时，或者被unpark后
               	设置记录阻塞线程的变量为null,设置阻塞线程为null
                **/
                p.parked = null;
                U.putObject(t, BLOCKER, null);
            }
            //设置slot的值为null
            else if (U.compareAndSwapObject(this, SLOT, p, null)) {
                v = timed && ns <= 0L && !t.isInterrupted() ? TIMED_OUT : null;
                break;
            }
        }
     	//设置匹配值为null。无论赋值线程是正常释放还是超时释放
        U.putOrderedObject(p, MATCH, null);
        p.item = null;
        p.hash = h;
        return v;
    }
```

### arenaExchange方法

```java
 private final Object arenaExchange(Object item, boolean timed, long ns) {
        Node[] a = arena;
        Node p = participant.get();
        for (int i = p.index;;) {                      // access slot at i
            int b, m, c; long j;                       // j is raw array offset
            //j 记录的是数组的偏移，即数组地址
            Node q = (Node)U.getObjectVolatile(a, j = (i << ASHIFT) + ABASE);
            //取值操作
            if (q != null && U.compareAndSwapObject(a, j, q, null)) {
                Object v = q.item;                     // release
                q.match = item;
                Thread w = q.parked;
                if (w != null)
                    U.unpark(w);
                return v;
            }
            else if (i <= (m = (b = bound) & MMASK) && q == null) {
                p.item = item;   //赋值操作                      // offer
                if (U.compareAndSwapObject(a, j, null, p)) {
                    long end = (timed && m == 0) ? System.nanoTime() + ns : 0L;
                    Thread t = Thread.currentThread(); // wait
                    for (int h = p.hash, spins = SPINS;;) {
                        Object v = p.match;
                        if (v != null) {
                            U.putOrderedObject(p, MATCH, null);
                            p.item = null;             // clear for next use
                            p.hash = h;
                            return v;
                        }
                        else if (spins > 0) {
                            h ^= h << 1; h ^= h >>> 3; h ^= h << 10; // xorshift
                            if (h == 0)                // initialize hash
                                h = SPINS | (int)t.getId();
                            else if (h < 0 &&          // approx 50% true
                                     (--spins & ((SPINS >>> 1) - 1)) == 0)
                                Thread.yield();        // two yields per wait
                        }
                        else if (U.getObjectVolatile(a, j) != p)
                            spins = SPINS;       // releaser hasn't set match yet
                        else if (!t.isInterrupted() && m == 0 &&
                                 (!timed ||
                                  (ns = end - System.nanoTime()) > 0L)) {
                            U.putObject(t, BLOCKER, this); // emulate LockSupport
                            p.parked = t;              // minimize window
                            if (U.getObjectVolatile(a, j) == p)
                                U.park(false, ns);
                            p.parked = null;
                            U.putObject(t, BLOCKER, null);
                        }
                        else if (U.getObjectVolatile(a, j) == p &&
                                 U.compareAndSwapObject(a, j, p, null)) {
                            if (m != 0)  //收缩              // try to shrink
                                U.compareAndSwapInt(this, BOUND, b, b + SEQ - 1);
                            p.item = null;
                            p.hash = h;
                            i = p.index >>>= 1;        // descend
                            if (Thread.interrupted())
                                return null;
                            if (timed && m == 0 && ns <= 0L)
                                return TIMED_OUT;
                            break;                     // expired; restart
                        }
                    }
                }
                else
                    p.item = null;                     // clear offer
            }
            else {
                if (p.bound != b) {                    // stale; reset
                    p.bound = b;
                    p.collides = 0;
                    i = (i != m || m == 0) ? m : m - 1;
                }
                else if ((c = p.collides) < m || m == FULL ||
                         !U.compareAndSwapInt(this, BOUND, b, b + SEQ + 1)) {
                    p.collides = c + 1;//冲突次数增加
                    i = (i == 0) ? m : i - 1;          // cyclically traverse
                }
                else
                    i = m + 1;                         // grow
                p.index = i;
            }
        }
    }
```

