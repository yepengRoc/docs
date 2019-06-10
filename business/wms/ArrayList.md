# ArrayList

### 实现

​		底层通过数组实现。数组用transient进行修饰。transient关键字修饰的，当数组在序列化的时候，数组中的null不会进行序列化。

### 扩容

每次扩容1.5倍

### fast-fail机制

​		维护属性modecount,每次在修改列表的时候，modecount++,修改完成后比较修改前的modecount和修改后的modecount是否一致，不一致则抛异常。说明存在并发情况

