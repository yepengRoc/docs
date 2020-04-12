https: hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/d975dfffada6/src/share/vm/runtime/synchronizer.hpp



https: hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/d975dfffada6/src/share/vm/runtime/safepoint.cpp

jvm 安全点的概念



开始将系统带入安全点的过程。 Java线程可以处于几种不同的状态，并且 通过不同的机制停止：

1.运行解释更改干预者分派表以强制其检查字节码之间的安全点条件。

 2.以本机代码运行从本机代码返回时，Java线程必须检查安全点_state来查看是否必须阻止。如果 VM线程在本机中看到一个Java线程，不等待该线程阻塞。内存顺序写入和读取安全点状态和Java线程状态很关键。为了保证内存写入相对于彼此序列化， VM线程发出内存屏障指令（在MP系统上）。为了避免发行的开销每个进行本地调用的Java线程的内存屏障线程在更改后执行对单个内存页的写操作线程状态。VM线程执行一系列 mprotect OS调用，强制所有所有先前的写操作要序列化的Java线程。这是在 os :: serialize_thread_states（）调用。事实证明这是比执行membar指令效率更高每次调用本机代码。

 3.运行已编译的代码编译后的代码读取一个全局（安全点轮询）页面，该页面如果我们试图达到安全点，则设置为错误。

 4.封锁被阻止的线程将不允许从阻止条件，直到安全点操作完成。

 5.在VM或状态之间转换如果Java线程当前正在VM中运行或正在过渡在状态之间，安全指向代码将等待线程执行在尝试转换到新状态时阻止自身。









将来，我们可能会：

 1.修改安全点方案，以避免潜在的无限旋转。这很棘手，因为线程退出JVM使用的路径（例如 在JNI调用中）仅存储到其状态字段中。负担放在必须轮询（旋转）的VM线程上。

2.找到在旋转时有用的操作。如果安全点与GC有关 我们可能会主动扫描已经安全的线程堆栈。

3.使用Solaris schedctl检查仍在运行的转换器的状态。 如果所有的变种器都是ONPROC，则没有理由睡觉或屈服。

4. YieldTo（）任何仍在运行但仍在运行但OFFPROC的变量。

5.检查系统饱和度。如果系统未完全饱和，则 简单旋转并避免睡眠屈服。

6.当仍在运行的变种者会合时，他们可以解除休眠状态VMthread。这对于仍在运行的mutator非常有用 安全。VMthread仍必须轮询是否有调出的变量。

7.从time-since-begin开始策略，而不是迭代。

8.考虑将旋转持续时间设为CPU数量的函数： 自旋=（（（（ncpus-1）* M）+ K）+ F（still_running） 或者，不计算外部循环的迭代次数 我们可以计算上面的内部循环中访问的线程数。

9.在Windows上考虑使用SwitchThreadTo（）的返回值 驱动后续的spin / SwitchThreadTo（）/ Sleep（N）决策。