



```java
 private final Object slotExchange(Object item, boolean timed, long ns) {
        Node p = participant.get();
        Thread t = Thread.currentThread();
        if (t.isInterrupted()) // preserve interrupt status so caller can recheck
            return null;

        for (Node q;;) {
            if ((q = slot) != null) {//slot不为null
                /**
                如果有其它线程调用exchange进行置换，且置换成功了.  
                值slot为null,取slot中记录的值
                
                **/
                if (U.compareAndSwapObject(this, SLOT, q, null)) {
                    Object v = q.item;
                    q.match = item;//记录当前要被置换出去的值
                    Thread w = q.parked;//如果有阻塞线程
                    if (w != null)
                        U.unpark(w);//进行阻塞线程释放
                    return v;
                }
                //有放值的线程，也有取值的线程。 多个线程进行置换，总有失败的，失败的走这里的逻辑
                // create arena on contention, but continue until slot null
                if (NCPU > 1 && bound == 0 &&
                    U.compareAndSwapInt(this, BOUND, 0, SEQ))
                    arena = new Node[(FULL + 2) << ASHIFT];
            }
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

        // await release
        int h = p.hash;
        long end = timed ? System.nanoTime() + ns : 0L;
        int spins = (NCPU > 1) ? SPINS : 1;
        Object v;
        while ((v = p.match) == null) {
            if (spins > 0) {
                h ^= h << 1; h ^= h >>> 3; h ^= h << 10;
                if (h == 0)
                    h = SPINS | (int)t.getId();
                else if (h < 0 && (--spins & ((SPINS >>> 1) - 1)) == 0)
                    Thread.yield();
            }
            else if (slot != p)
                spins = SPINS;
            else if (!t.isInterrupted() && arena == null &&
                     (!timed || (ns = end - System.nanoTime()) > 0L)) {
                U.putObject(t, BLOCKER, this);
                p.parked = t;
                if (slot == p)
                    U.park(false, ns);
                p.parked = null;
                U.putObject(t, BLOCKER, null);
            }
            else if (U.compareAndSwapObject(this, SLOT, p, null)) {
                v = timed && ns <= 0L && !t.isInterrupted() ? TIMED_OUT : null;
                break;
            }
        }
        U.putOrderedObject(p, MATCH, null);
        p.item = null;
        p.hash = h;
        return v;
    }
```

