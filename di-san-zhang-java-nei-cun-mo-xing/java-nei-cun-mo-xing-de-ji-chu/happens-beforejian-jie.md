# happens-before简介

从JDK 5开始，Java使用新的JSR-133内存模型（除非特别说明，本文针对的都是JSR-133内存模型）。JSR-133使用happens-before的概念来阐述操作之间的内存可见性。在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系。这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。

与程序员密切相关的happens-before规则如下。

·程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。

·监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。

·volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。

·传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。

注意　两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个

操作之前执行！happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一

个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second）。

happens-before的定义很微妙，后文会具体说明happens-before为什么要这么定义。

happens-before与JMM的关系如图3-5所示。

