类NioEventLoop 中的静态代码块，解决了jdk中nio中的bug

```
// Workaround for JDK NIO bug.
//
// See:
// - http://bugs.sun.com/view_bug.do?bug_id=6427854  jdkbug地址
// - https://github.com/netty/netty/issues/203  bug解决方案
static {
    final String key = "sun.nio.ch.bugLevel";
    final String buglevel = SystemPropertyUtil.get(key);
    if (buglevel == null) {
        try {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                @Override
                public Void run() {
                    System.setProperty(key, "");
                    return null;
                }
            });
        } catch (final SecurityException e) {
            logger.debug("Unable to get/set System Property: " + key, e);
        }
    }
```



解决方案是，在获取属性sun.nio.ch.bugLevel 的时候，如果为null,则手动设置一个空字符串



http://bugs.sun.com/view_bug.do?bug_id=6427854 

bug开始于：java version "1.5.0_06"

解决于：jdk 7b08

bug产生的原因：非线程安全导致的

sun.nio.ch.Util

```
private static String bugLevel = null;

    static boolean atBugLevel(String bl) {		// package-private
        if (bugLevel == null) {
            if (!sun.misc.VM.isBooted())
                return false;
            java.security.PrivilegedAction pa =
                new GetPropertyAction("sun.nio.ch.bugLevel");
// the next line can reset bugLevel to null
            bugLevel = (String)AccessController.doPrivileged(pa);
            if (bugLevel == null)
                bugLevel = "";
        }
        return (bugLevel != null) && bugLevel.equals(bl);
    }
两个线程同时执行到了if (buglevel == null)
第一个线程执行到了return (bugLevel != null) && bugLevel.equals(bl)
 已经执行了bugLevel != null，还没执行 bugLevel.equals(bl)
第二个线程执行到了bugLevel = (String)AccessController.doPrivileged(pa) 如果没有设置sun.nio.ch.bugLevel 则这里会把bugLevel赋值为null

第一个线程bugLevel.equals(bl) 判断的时候抛NPE
```

