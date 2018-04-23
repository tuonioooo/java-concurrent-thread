concurrent包的实现

由于Java的CAS同时具有volatile读和volatile写的内存语义，因此Java线程之间的通信现

在有了下面4种方式。

1）A线程写volatile变量，随后B线程读这个volatile变量。

2）A线程写volatile变量，随后B线程用CAS更新这个volatile变量。

3）A线程用CAS更新一个volatile变量，随后B线程用CAS更新这个volatile变量。

4）A线程用CAS更新一个volatile变量，随后B线程读这个volatile变量。

Java的CAS会使用现代处理器上提供的高效机器级别的原子指令，这些原子指令以原子

方式对内存执行读-改-写操作，这是在多处理器中实现同步的关键（从本质上来说，能够支持

原子性读-改-写指令的计算机，是顺序计算图灵机的异步等价机器，因此任何现代的多处理器

都会去支持某种能对内存执行原子性读-改-写操作的原子指令）。同时，volatile变量的读/写和

CAS可以实现线程之间的通信。把这些特性整合在一起，就形成了整个concurrent包得以实现

的基石。如果我们仔细分析concurrent包的源代码实现，会发现一个通用化的实现模式。

首先，声明共享变量为volatile。

然后，使用CAS的原子条件更新来实现线程之间的同步。

同时，配合以volatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的

通信。

AQS，非阻塞数据结构和原子变量类（java.util.concurrent.atomic包中的类），这些concurrent

包中的基础类都是使用这种模式来实现的，而concurrent包中的高层类又是依赖于这些基础类

来实现的。从整体来看，concurrent包的实现示意图如3-5-4-1所示。

图3-5-4-1![](/assets/import-3-5-4-1.png)

