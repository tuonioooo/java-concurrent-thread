# 数据竞争与顺序一致性

当程序未正确同步时，就可能会存在数据竞争。Java内存模型规范对数据竞争的定义如下。

在一个线程中写一个变量，在另一个线程读同一个变量，而且写和读没有通过同步来排序。当代码中包含数据竞争时，程序的执行往往产生违反直觉的结果（前一章的示例正是如此）。如果一个多线程程序能正确同步，这个程序将是一个没有数据竞争的程序。

JMM对正确同步的多线程程序的内存一致性做了如下保证。如果程序是正确同步的，程序的执行将具有顺序一致性（Sequentially Consistent）——即程序的执行结果与该程序在顺序一致性内存模型中的执行结果相同。马上我们就会看到，这对于程序员来说是一个极强的保证。这里的同步是指广义上的同步，包括对常用同步原语（synchronized、volatile和final）的正确使用。
