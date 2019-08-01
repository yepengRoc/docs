方法上加锁，锁是当前实例对象

静态同步方法加锁，锁是当前类的class对象

普通方法加锁，锁是指定实例对象



同步方法块通过moniterenter moniterexit实现锁

同步方法 通过置access_flags 为1来实现锁



1. 锁的优化 

- 锁粗化

- 锁消除

- 偏向锁

- 适应性自旋

参考[jvm内部细节1](https://www.cnblogs.com/javaminer/p/3889023.html)

参考[jvm内部细节2](https://www.cnblogs.com/javaminer/p/3889023.html)



java对象头

markwork 是在类实例化的时候，被添加到实例对象上的

markwork 设计成一个非固定的数据结构，以便存储更多的数据



moniter 是线程的私有数据结构





