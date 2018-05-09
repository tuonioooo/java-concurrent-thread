# CompletableFuture与parallelStream\(\)性能差异

parallelStream\(\) 与 CompletableFuture 的性能差异假设一个这样的耗时 1000 毫秒的计算任务

```
private static int getJob() {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
    }
    return 50;
}
```

分别用下面两个方法来测试，任务数可以通过参数来控制

```
private static long testParallelStream(int jobCount) {
    List<Supplier<Integer>> tasks = new ArrayList<>();
    IntStream.rangeClosed(1, jobCount).forEach(value -> tasks.add(Main::getJob));
 
    long start = System.currentTimeMillis();
    int sum = tasks.parallelStream().map(Supplier::get).mapToInt(Integer::intValue).sum();
    return System.currentTimeMillis() - start;
}
 
private static long testCompletableFutureDefaultExecutor(int jobCount) {
    List<CompletableFuture<Integer>> tasks = new ArrayList<>();
    IntStream.rangeClosed(1, jobCount).forEach(value -> tasks.add(CompletableFuture.supplyAsync(Main::getJob)));
 
    long start = System.currentTimeMillis();
    int sum = tasks.stream().map(CompletableFuture::join).mapToInt(Integer::intValue).sum();
    return System.currentTimeMillis() - start;
}
```

写下本文的测试机器用`Runtime.getRuntime().availableProcessors()`得到的 CPU 内核数是 4, 下面是两个方法在不同的并发任务数时一组耗时对照表

| 方法 \| jobCount \| 耗时\(毫秒\) |  1 | 3 | 4 | 5 | 6 | 8 | 9 | 14 | 20 | 21 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
|  testParallelStream |  1009 | 1004 | 1002 | 2005 |  2006 |  2006 | 3012 |  4014 |  5020 |  6018 |
|  testCompletableFutureDefaultExecutor |  1005 |  1005 | 2005 |  2002 |  2008 | 3008 |  3008 |  5016 |  7025 |  7026 |

从上面数据来看，默认情况下，随着任务数的增加，`CompletableFuture`反而比最简单的`parallelStream()`操作方式性能显得越发差一些，是不是有些失望了。慢着，我们是说默认情况下， `parallelStream()`和`CompletableFuture.supplyAsync(job)`的 job 都会分配给默认 `ForkJoinPool.commonPool()`去执行，而这个线程池的大小是 CPU 的内核数，所以它们没有多大区别，甚至`CompletableFuture`的方式比`parallelStream()`更快达到线程的饱和，性能还略微差一些。

可是，不用急，我们还有大招，因为`parallelStream()`不能灵活的干预线程池的大小\(默认为 CPU 内核数\)，要改的话会影响到系统全局的`ForkJoinPool.commonPool()`的大小。可通过指定系统属性 `java.util.concurrent.ForkJoinPool.common.parallelism`来指定这个线程池大小。然而`CompletableFuture`还有第二个版本的 `supplyAsync(supplier, executor)`方法，由第二个参数来指定所使用的线程池，所以我们再定义一个方法，把自定义线程池大小调到 20, 然后重新进行测试并与前面数据对比

```
private static long testCompletableFutureDefaultExecutor(int jobCount) {
    List<CompletableFuture<Integer>> tasks = new ArrayList<>();
    IntStream.rangeClosed(1, jobCount).forEach(value -> tasks.add(CompletableFuture.supplyAsync(Main::getJob)));
 
    long start = System.currentTimeMillis();
    int sum = tasks.stream().map(CompletableFuture::join).mapToInt(Integer::intValue).sum();
    return System.currentTimeMillis() - start;
}
```

测试的结果如下

| 方法 \| jobCount \| 耗时\(毫秒\) | 1 | 3 | 4 | 5 | 6 | 8 | 9 | 14 | 20 | 21 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| testParallelStream | 1009 | 1004 | 1002 | 2005 | 2006 | 2006 | 3012 | 4014 | 5020 | 6018 |
| testCompletableFutureDefaultExecutor | 1005 | 1005 | 2005 | 2002 | 2008 | 3008 | 3008 | 5016 | 7025 | 7026 |
| testCompletableFutureCustomExecutor | 1004 | 1003 | 1004 | 1002 | 1005 | 1001 | 1003 | 1002 | 1003 | 2005 |

  


