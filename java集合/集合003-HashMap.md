hashMap的不安全操作

多线程情况下，hashMap扩容的时候，会引起死循环

```java
void transfer(Entry[] newTable) {
    Entry[] src = table;
    int newCapacity = newTable.length;
    for (int j = 0; j < src.length; j++) {
        Entry<K,V> e = src[j];
        if (e != null) {
            src[j] = null;
            do {
                Entry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            } while (e != null);
        }
    }
}
```

执行到数组链表的时候

值1---》值2  即 值1.next = 值2

线程1执行了

```java
Entry<K,V> e = src[j];//值1
Entry<K,V> next = e.next;//值2
```

然后切换到线程b执行，一直到执行完，因为hash冲突链表采用的是头插法，所以执行完后的结果是

值2.next=值1

然后切换到线程1执行

```java
//e	在线程1 中为值1 此时newTable[i]的位置为值2 此操作过后 值1.next=值2。
 e.next = newTable[i];
//值1 又放回了 i位置
 newTable[i] = e;
//赋值之后e 这里为值2
 e = next;
```

这里就形成了死循环

值2.next=值1

值1.next=值2



