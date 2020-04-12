https://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/95bccc5efc6a/src/share/vm/oops/markOop.hpp

markOop描述对象的头

请注意，标记不是真正的标记，而只是一个word(32 位虚拟你一个word 是4 byte 32bit;64位虚拟机 一个word 是8byte 64bit)。

由于历史原因，它被放置在oop层次中

对象标头的位格式（以下为最高有效位，以下为大字节序布局）：

```java
//  32 bits:
//  --------
//             hash:25 ------------>| age:4    biased_lock:1 lock:2 (normal object)
//             JavaThread*:23 epoch:2 age:4    biased_lock:1 lock:2 (biased object)
//             size:32 ------------------------------------------>| (CMS free block)
//             PromotedObject*:29 ---------->| promo_bits:3 ----->| (CMS promoted object)
//
//  64 bits:
//  --------
//  unused:25 hash:31 -->| unused:1   age:4    biased_lock:1 lock:2 (normal object)
//  JavaThread*:54 epoch:2 unused:1   age:4    biased_lock:1 lock:2 (biased object)
//  PromotedObject*:61 --------------------->| promo_bits:3 ----->| (CMS promoted object)
//  size:64 ----------------------------------------------------->| (CMS free block)
//
//  unused:25 hash:31 -->| cms_free:1 age:4    biased_lock:1 lock:2 (COOPs && normal object)
//  JavaThread*:54 epoch:2 cms_free:1 age:4    biased_lock:1 lock:2 (COOPs && biased object)
//  narrowOop:32 unused:24 cms_free:1 unused:4 promo_bits:3 ----->| (COOPs && CMS promoted object)
//  unused:21 size:35 -->| cms_free:1 unused:7 ------------------>| (COOPs && CMS free block)
//
```

-- hash包含标识哈希值：最大值为 31位，请参见os :: random（）。另外，64位虚拟机需要哈希值不大于32位，因为它们不会正确生成一个大于该值的掩码：请参见library_call.cpp和c1_CodePatterns_sparc.cpp。

--偏向锁模式用于将锁偏向给定线程。当此模式设置为低三位时，锁定要么偏向给定线程，要么偏向“匿名”偏向，表示可能偏向。当锁定偏向给定的线程的时候，锁定和解锁可以由该线程执行，而无需使用原子操作。取消锁的偏向后，它将恢复为正常如下所述的锁定方案。



（状态头中放置了过多的信息）

请注意，我们超载了“解锁”状态头的含义。因为我们从age中借用了一个bit，所以我们可以确保绝对不会看到这种偏向解锁对象。

还要注意，偏向状态通常包含age bit （age年龄 新生代或老年代）包含在对象头中。清理时间的大幅增加 看到这些bit缺失和任意age的时间分配给所有偏向的对象，因为它们倾向于消耗eden半空间的相当大的一部分，不是立即晋升（老年代），导致复制数量增加 执行。运行时系统将所有JavaThread *指针对齐一个非常大的值（当前为128字节（32bVM）或256字节（64bVM））为age big和epoch bit腾出空间（用于支持有偏向的锁定），并针对64bVM中的CMS“释放”位（+ COOP）。

```java
//    [JavaThread* | epoch | age | 1 | 01]       lock is biased toward given thread 锁偏向给定线程
//    [0           | epoch | age | 1 | 01]       lock is anonymously biased 锁匿名偏向
```

这两个锁定位用于描述三种状态：锁定/解锁和监视。

```java
//    [ptr             | 00]  locked             ptr points to real header on stack ptr指向堆栈上的真实对象头
//    [header      | 0 | 01]  unlocked           regular object header 常规对象头
//    [ptr             | 10]  monitor            inflated lock (header is wapped out) 膨胀的锁（将对象头换出）
//    [ptr             | 11]  marked             used by markSweep to mark an object
//                                               not valid at any other time  markSweep用于标记对象在任何其他时间无效
```

java对象头分析

https://www.jianshu.com/p/9c19eb0ea4d8