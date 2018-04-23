# happens-before简介

从JDK 5开始，Java使用新的JSR-133内存模型（除非特别说明，本文针对的都是JSR-133内存模型）。JSR-133使用happens-before的概念来阐述操作之间的内存可见性。在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系。这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。

与程序员密切相关的happens-before规则如下。

* 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。

* 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。

* volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。

* 线程启动规则：Thread 对象的 start\(\)方法 happen—before 此线程的每一个动作。

* 线程终止规则：线程的所有操作都 happen—before 对此线程的终止检测，可以通过 Thread.join\(\)方法结束 Thread.isAlive\(\)的返回值等手段检测到线程已经终止执行。

* 线程中断规则：对线程 interrupt\(\)方法的调用 happen—before 发生于被中断线程的代码检测到中断时事件的发生。

* 对象终结规则：一个对象的初始化完成（构造函数执行结束）happen—before 它的 finalize\(\)方法的开始。

* 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。



注意　两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second）。happens-before的定义很微妙，后文会具体说明happens-before为什么要这么定义。

happens-before与JMM的关系如图1所示。

图1![](/assets/import-happens-before-1.png)

如图1所示，一个happens-before规则对应于一个或多个编译器和处理器重排序规则。对于Java程序员来说，happens-before规则简单易懂，它避免Java程序员为了理解JMM提供的内存

可见性保证而去学习复杂的重排序规则以及这些规则的具体实现方法。

