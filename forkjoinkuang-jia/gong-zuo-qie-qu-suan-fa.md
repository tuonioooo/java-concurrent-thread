# 工作窃取算法

工作窃取（work-stealing）算法是指某个线程从其他队列里窃取任务来执行。那么，为什么

需要使用工作窃取算法呢？假如我们需要做一个比较大的任务，可以把这个任务分割为若干

互不依赖的子任务，为了减少线程间的竞争，把这些子任务分别放到不同的队列里，并为每个

队列创建一个单独的线程来执行队列里的任务，线程和队列一一对应。比如A线程负责处理A

队列里的任务。但是，有的线程会先把自己队列里的任务干完，而其他线程对应的队列里还有

任务等待处理。干完活的线程与其等着，不如去帮其他线程干活，于是它就去其他线程的队列

里窃取一个任务来执行。而在这时它们会访问同一个队列，所以为了减少窃取任务线程和被

窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿

任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行。

工作窃取的运行流程如图6-7所示。

![](../.gitbook/assets/import-6-7.png)

工作窃取算法的优点：充分利用线程进行并行计算，减少了线程间的竞争。

工作窃取算法的缺点：在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并

且该算法会消耗了更多的系统资源，比如创建多个线程和多个双端队列。

