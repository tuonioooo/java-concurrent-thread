olatile写-读的内存语义

volatile写的内存语义如下。

当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内

存。

以上面示例程序VolatileExample为例，假设线程A首先执行writer\(\)方法，随后线程B执行

reader\(\)方法，初始时两个线程的本地内存中的flag和a都是初始状态。图3-17是线程A执行

volatile写后，共享变量的状态示意图。

