# JVM001

eden区

survivor区

新生代young generation  发生的gc 叫minor gc

老年代old generation   发生的gc 叫major gc 或full gc

新生代发生gc后存活下来的对象会移到老年代

永久代 permanet generation 也成方法区（method area）.用来存储常量以及字符常量，也会发生gc，这里的gc也叫 full gc

1. 老年代对新生代的引用

   老年代中有一个card table(大小为512byte的块)，老年代对新生代的引用会记录在这里，当新生代执行gc的时候，查询card table来知道是否被回收，不用查询整个老年代，card table 由一个write barrier来管理。以空间换时间，减少gc时间



## 新生代

新生代分为一个 eden区 两个 survivor区。比例为8：1：1

