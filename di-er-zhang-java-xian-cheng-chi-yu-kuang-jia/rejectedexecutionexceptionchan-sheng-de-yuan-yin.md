# RejectedExecutionException

引发java.util.concurrent.RejectedExecutionException主要有两种原因：

1. 线程池显示的调用了shutdown\(\)之后，再向线程池提交任务的时候，如果你配置的拒绝策略是ThreadPoolExecutor.AbortPolicy的话，这个异常就被会抛出来。

2. 当你的排队策略为有界队列，并且配置的拒绝策略是ThreadPoolExecutor.AbortPolicy，当线程池的线程数量已经达到了maximumPoolSize的时候，你再向它提交任务，就会抛出ThreadPoolExecutor.AbortPolicy异常。



