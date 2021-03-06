# CompletableFuture讲解

CompletableFuture是java8中添加的一个类，这个类主要的作用就是提供了新的方式来完成异步处理,包括合成和组合事件的非阻塞方式。

CompletableFuture类实现了CompletionStage和Future接口。Future是Java 5添加的类，用来描述一个异步计算的结果，但是获取一个结果时方法较少,要么通过轮询isDone，确认完成后，调用get\(\)获取值，要么调用get\(\)设置一个超时时间。但是这个get\(\)方法会阻塞住调用线程，这种阻塞的方式显然和我们的异步编程的初衷相违背。

为了解决这个问题，JDK吸收了guava的设计思想，加入了Future的诸多扩展功能形成了CompletableFuture。

CompletableFuture提供更多的特性：

1. 将两个异步计算的结果组合成一个结果，并且这两个异步计算时相互独立的在不同到线程中，并且第二个计算依赖第一个计算。
2. 等待所有异步任务的完成。
3. 等待所有异步任务中任意一个完成并且获得计算结果。

4.编程式的完成异步任务。（手动提供一个结果）

5.异步任务完成响应事件。

github demo示例：

[https://github.com/tuonioooo/java8-examples-master](https://github.com/tuonioooo/java8-examples-master)

参考：

[https://www.jianshu.com/p/6f3ee90ab7d3](https://www.jianshu.com/p/6f3ee90ab7d3)

[https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html?is-external=true](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html?is-external=true)

## 目录

* [CompletableFuture接口详解](completablefuture4e0029.md) 
* [CompletableFuture接口详解2](completablefuturejie-kou-xiang-jie-2.md)
* [CompletableFuture与parallelStream\(\)性能差异](completablefutureyu-parallelstream-xing-neng-cha-yi.md)

