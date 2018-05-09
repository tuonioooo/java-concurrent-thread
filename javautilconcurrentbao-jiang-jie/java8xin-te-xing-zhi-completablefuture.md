# \*\*CompletableFuture\*\*



CompletableFuture是java8中添加的一个类，这个类主要的作用就是提供了新的方式来完成异步处理,包括合成和组合事件的非阻塞方式。

CompletableFuture类实现了CompletionStage和Future接口。Future是Java 5添加的类，用来描述一个异步计算的结果，但是获取一个结果时方法较少,要么通过轮询isDone，确认完成后，调用get\(\)获取值，要么调用get\(\)设置一个超时时间。但是这个get\(\)方法会阻塞住调用线程，这种阻塞的方式显然和我们的异步编程的初衷相违背。

为了解决这个问题，JDK吸收了guava的设计思想，加入了Future的诸多扩展功能形成了CompletableFuture。



