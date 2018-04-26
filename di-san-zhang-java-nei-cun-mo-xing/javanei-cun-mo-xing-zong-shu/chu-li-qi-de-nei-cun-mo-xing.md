# 处理器的内存模型

顺序一致性内存模型是一个理论参考模型，JMM和处理器内存模型在设计时通常会以顺序一致性内存模型为参照。在设计时，JMM和处理器内存模型会对顺序一致性模型做一些放松，因为如果完全按照顺序一致性模型来实现处理器和JMM，那么很多的处理器和编译器优化都要被禁止，这对执行性能将会有很大的影响。

根据对不同类型的读/写操作组合的执行顺序的放松，可以把常见处理器的内存模型划分为如下几种类型。

* 放松程序中写-读操作的顺序，由此产生了Total Store Ordering内存模型（简称为TSO）。
* 在上面的基础上，继续放松程序中写-写操作的顺序，由此产生了Partial Store Order内存模型（简称为PSO）。
* 在前面两条的基础上，继续放松程序中读-写和读-读操作的顺序，由此产生了Relaxed Memory Order内存模型（简称为RMO）和PowerPC内存模型。

注意，这里处理器对读/写操作的放松，是以两个操作之间不存在数据依赖性为前提的（因为处理器要遵守as-if-serial语义，处理器不会对存在数据依赖性的两个内存操作做重排序）。

表3-7-1-1展示了常见处理器内存模型的细节特征如下。

表3-7-1-1　处理器内存模型的特征表

![](/assets/import-3-7-1-1.png)

从表3-7-1-1中可以看到，所有处理器内存模型都允许写-读重排序，原因在第1章已经说明过：它们都使用了写缓存区。写缓存区可能导致写-读操作重排序。同时，我们可以看到这些处理器内存模型都允许更早读到当前处理器的写，原因同样是因为写缓存区。由于写缓存区仅对当前处理器可见，这个特性导致当前处理器可以比其他处理器先看到临时保存在自己写缓存区中的写。

表3-7-1-1中的各种处理器内存模型，从上到下，模型由强变弱。越是追求性能的处理器，内存模型设计得会越弱。因为这些处理器希望内存模型对它们的束缚越少越好，这样它们就可以做尽可能多的优化来提高性能。

由于常见的处理器内存模型比JMM要弱，Java编译器在生成字节码时，会在执行指令序列的适当位置插入内存屏障来限制处理器的重排序。同时，由于各种处理器内存模型的强弱不同，为了在不同的处理器平台向程序员展示一个一致的内存模型，JMM在不同的处理器中需要插入的内存屏障的数量和种类也不相同。图3-48展示了JMM在不同处理器内存模型中需要插入的内存屏障的示意图。

JMM屏蔽了不同处理器内存模型的差异，它在不同的处理器平台之上为Java程序员呈现了一个一致的内存模型。
