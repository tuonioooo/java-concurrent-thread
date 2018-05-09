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

![](/assets/import-completablefuture-1.png)

从上面数据来看，默认情况下，随着任务数的增加，`CompletableFuture`反而比最简单的`parallelStream()`操作方式性能显得越发差一些，是不是有些失望了。慢着，我们是说默认情况下， `parallelStream()`和`CompletableFuture.supplyAsync(job)`的 job 都会分配给默认 `ForkJoinPool.commonPool()`去执行，而这个线程池的大小是 CPU 的内核数，所以它们没有多大区别，甚至`CompletableFuture`的方式比`parallelStream()`更快达到线程的饱和，性能还略微差一些。

可是，不用急，我们还有大招，因为`parallelStream()`不能灵活的干预线程池的大小\(默认为 CPU 内核数\)，要改的话会影响到系统全局的`ForkJoinPool.commonPool()`的大小。可通过指定系统属性 `java.util.concurrent.ForkJoinPool.common.parallelism`来指定这个线程池大小。然而`CompletableFuture`还有第二个版本的 `supplyAsync(supplier, executor)`方法，由第二个参数来指定所使用的线程池，所以我们再定义一个方法，把自定义线程池大小调到 20, 然后重新进行测试并与前面数据对比

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

![](/assets/import-complatablefuture-2.png)

现在 `CompletableFuture`的优越性就体现出来了，任务数在没有超过线程池大小 20 的时候，仿佛在执行一个任务一样，所需耗时一保持在 1000 毫秒左右，任务数在 21 的时候才发生第一次线程等待的情况，所以耗时为`2005`毫秒。

继续观察上面那个耗时对照表，结合执行每个方法时的线程池大小来解读上面的测试结果，简单点来讲上面标记为绿色的数字开始出现了任务等待线程池中的可用线程，当把线程池调大一些并发性能就越高。当然不是线程数越大越好，因为我们这儿例子中的计算任务很简单，所以线程再大些也无妨。

对于如何选择合适的线程池大小，这里不深究了，需权衡到计算任务的耗时，是否有 I/O 的等，有一个公式是

```
Nthreads = Ncpu * Ucpu * (1 + W/C)

Ncpu 是 CPU 内核数，通过 Runtime.getRuntime().availableProcessors() 得到的值
Ucpu 是期望的 CPU 的使用率(介于 0 与 1 之间)
W/C 是等待时间与任务执行时间的比率
```

下面是本篇测试用的完整代码：

```
package cc.unmi;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.Executor;
import java.util.concurrent.ForkJoinPool;
import java.util.function.Supplier;
import java.util.stream.IntStream;

import static java.lang.String.format;

public class Main {

    private static int PROCESSORS = Runtime.getRuntime().availableProcessors();

    public static void main(String[] args) {
        System.out.println("Processors: " + PROCESSORS);

        Arrays.asList(-3, -1, 0, 1, 2, 4, 5, 10, 16, 17).forEach(offset -> {
            int jobNum = PROCESSORS + offset;
            System.out.println(
                format("When %s tasks => stream: %s, parallelStream: %s, future default: %s, future custom: %s",
                    jobNum, testStream(jobNum), testParallelStream(jobNum),
                    testCompletableFutureDefaultExecutor(jobNum), testCompletableFutureCustomExecutor(jobNum)));
        });

    }

    private static long testStream(int jobCount) {
        List<Supplier<Integer>> tasks = new ArrayList<>();
        IntStream.rangeClosed(1, jobCount).forEach(value -> tasks.add(Main::getJob));

        long start = System.currentTimeMillis();
        int sum = tasks.stream().map(Supplier::get).mapToInt(Integer::intValue).sum();
        checkSum(sum, jobCount);
        return System.currentTimeMillis() - start;
    }

    private static long testParallelStream(int jobCount) {
        List<Supplier<Integer>> tasks = new ArrayList<>();
        IntStream.rangeClosed(1, jobCount).forEach(value -> tasks.add(Main::getJob));

        long start = System.currentTimeMillis();
        int sum = tasks.parallelStream().map(Supplier::get).mapToInt(Integer::intValue).sum();
        checkSum(sum, jobCount);
        return System.currentTimeMillis() - start;
    }

    private static long testCompletableFutureDefaultExecutor(int jobCount) {
        List<CompletableFuture<Integer>> tasks = new ArrayList<>();
        IntStream.rangeClosed(1, jobCount).forEach(value -> tasks.add(CompletableFuture.supplyAsync(Main::getJob)));

        long start = System.currentTimeMillis();
        int sum = tasks.stream().map(CompletableFuture::join).mapToInt(Integer::intValue).sum();
        checkSum(sum, jobCount);
        return System.currentTimeMillis() - start;
    }

    private static long testCompletableFutureCustomExecutor(int jobCount) {
        Executor executor = new ForkJoinPool(20);

        List<CompletableFuture<Integer>> tasks = new ArrayList<>();
        IntStream.rangeClosed(1, jobCount).forEach(value -> tasks.add(CompletableFuture.supplyAsync(Main::getJob, executor)));

        long start = System.currentTimeMillis();
        int sum = tasks.stream().map(CompletableFuture::join).mapToInt(Integer::intValue).sum();
        checkSum(sum, jobCount);
        return System.currentTimeMillis() - start;
    }

    private static int getJob() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
        }

        return 50;
    }

    private static void checkSum(int sum, int jobNum) {
        if(sum != 50 * jobNum) {
            throw new AssertionError("Result Error");
        }
    }
}
```

运行后控制台输出

```
Processors: 4
When 1 tasks => stream: 1007, parallelStream: 1009, future default: 1005, future custom: 1004
When 3 tasks => stream: 3014, parallelStream: 1004, future default: 1005, future custom: 1003
When 4 tasks => stream: 4010, parallelStream: 1002, future default: 2005, future custom: 1004
When 5 tasks => stream: 5013, parallelStream: 2005, future default: 2002, future custom: 1002
When 6 tasks => stream: 6019, parallelStream: 2006, future default: 2008, future custom: 1005
When 8 tasks => stream: 8026, parallelStream: 2006, future default: 3008, future custom: 1001
When 9 tasks => stream: 9028, parallelStream: 3012, future default: 3008, future custom: 1003
When 14 tasks => stream: 14045, parallelStream: 4014, future default: 5016, future custom: 1002
When 20 tasks => stream: 20049, parallelStream: 5020, future default: 7025, future custom: 1003
When 21 tasks => stream: 21057, parallelStream: 6018, future default: 7026, future custom: 2005
```

不同机器上跑的需依据实际 CPU 内核和所定制的线程池大小数来调整每次测试的任务数，即须修改

`Arrays.asList(-3, -1, 0, 1, 2, 4, 5, 10, 16, 17)`

中的测试参数来覆盖某些边界值

