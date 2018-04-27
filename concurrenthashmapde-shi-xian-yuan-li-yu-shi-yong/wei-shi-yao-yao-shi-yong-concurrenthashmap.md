为什么要使用ConcurrentHashMap

在并发编程中使用HashMap可能导致程序死循环。而使用线程安全的HashTable效率又非

常低下，基于以上两个原因，便有了ConcurrentHashMap的登场机会。

（1）线程不安全的HashMap

在多线程环境下，使用HashMap进行put操作会引起死循环，导致CPU利用率接近100%，所

以在并发情况下不能使用HashMap。例如，执行以下代码会引起死循环。

