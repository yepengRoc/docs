通过两个队列来实现，一个写，一个读，写完之后交换两个队列

通过标识 flag控制读写队列交换

- 第一次没有数据交换  将数据从ture改为false。 因为默认是false修改失败
  直接在等待

数据添加进写队列。将 标识 从false改为true

唤醒 直接在等待。
进行docmmit，这个时候读队列没有数据。

直接进入下一次循环。
进行 将数据从ture改为fals .成功，置换读写队列。直接返回

进行docommit，数据提价完了后。

进行下一次循环。此次写完数据，才可能交换。

进入docommit 完了之后

私服
线程1
flag = false
等待


线程2
flag=true
写入writelist
唤醒等待的线程1

线程1被唤醒，进行数据处理。数据处理完
进行下一次循环
flag 由true改为false
置换 writelist 和readlist