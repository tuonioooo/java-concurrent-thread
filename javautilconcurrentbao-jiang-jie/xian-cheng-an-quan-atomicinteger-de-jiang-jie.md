# 线程安全AtomicInteger的讲解

参考：[https://blog.csdn.net/tuoni123/article/details/80215749](https://blog.csdn.net/tuoni123/article/details/80215749)

```text
package concurrent;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.stream.IntStream;

/**
 * @author tuonioooo
 */
public class Atomic1 {

    private static final int NUM_INCREMENTS = 1000;

    private static AtomicInteger atomicInt = new AtomicInteger(0);

    public static void main(String[] args) {
        testIncrementOfFor();
        testIncrement();
        testAccumulate();
        testAccumulateOfFor();
        testUpdate();

    }

    private static void testUpdate() {
        atomicInt.set(0);

        ExecutorService executor = Executors.newFixedThreadPool(2);

        IntStream.range(0, NUM_INCREMENTS)
                .forEach(i -> {
                    Runnable task = () ->
                            atomicInt.updateAndGet(n -> n + 2);//每次进行+2操作
                    executor.submit(task);
                });

        ConcurrentUtils.stop(executor);

        System.out.format("Update: %d\n", atomicInt.get());
    }

    private static void testAccumulate() {
        atomicInt.set(0);

        ExecutorService executor = Executors.newFixedThreadPool(2);

        IntStream.range(0, NUM_INCREMENTS)
                .forEach(i -> {
                    Runnable task = () ->
                            atomicInt.accumulateAndGet(i, (n, m) -> n + m);//循环值之后操作
                    executor.submit(task);
                });

        ConcurrentUtils.stop(executor);

        System.out.format("Accumulate: %d\n", atomicInt.get());
    }

    private static void testIncrement() {
        atomicInt.set(0);
        ExecutorService executor = Executors.newFixedThreadPool(2);

        long startMillis = System.currentTimeMillis();

        IntStream.range(0, NUM_INCREMENTS)
                .forEach(i -> executor.submit(atomicInt::incrementAndGet));//+1操作

        ConcurrentUtils.stop(executor);

        long endMillis = System.currentTimeMillis();

        System.out.format("totalTime=%d\n", endMillis-startMillis);
        System.out.format("Increment: Expected=%d; Is=%d\n", NUM_INCREMENTS, atomicInt.get());
        System.out.format("Increment: Expected=%d; Is=%d\n", NUM_INCREMENTS, atomicInt.getAndIncrement());


    }

    private static void testIncrementOfFor(){
        long startMillis = System.currentTimeMillis();
        for (int i = 0; i <NUM_INCREMENTS ; i++) {
            atomicInt.incrementAndGet();//+1操作
        }
        long endMillis = System.currentTimeMillis();
        System.out.format("totalTime=%d\n", endMillis-startMillis);
        System.out.format("testIncrementOfFor: Expected=%d; Is=%d\n", NUM_INCREMENTS, atomicInt.get());
        System.out.format("testIncrementOfFor: Expected=%d; Is=%d\n", NUM_INCREMENTS, atomicInt.getAndIncrement());

    }

    private static void testAccumulateOfFor(){
        int sum = 0;
        for (int i = 0; i < NUM_INCREMENTS; i++) {
            sum +=i;
        }
        System.out.println("sum = " + sum);

        long startMillis = System.currentTimeMillis();
        for (int i = 0; i <NUM_INCREMENTS ; i++) {
            atomicInt.accumulateAndGet(i, (n, m) -> n + m); // 计算 NUM_INCREMENTS循环内的值之和
        }
        long endMillis = System.currentTimeMillis();
        System.out.format("totalTime=%d\n", endMillis-startMillis);
        System.out.format("testAccumulateOfFor: Expected=%d; Is=%d\n", NUM_INCREMENTS, atomicInt.get());
        System.out.format("testAccumulateOfFor: Expected=%d; Is=%d\n", NUM_INCREMENTS, atomicInt.getAndIncrement());

    }

}
```

