# CountDownLatch

## 概述

java.util.concurrent.CountDownLatch 是一个并发构造，它允许一个或多个线程等待其他线程完成操作。

## 示例：

假如有这样一个需求：我们需要解析一个Excel里多个sheet的数据，此时可以考虑使用多线程，每个线程解析一个sheet里的数据，等到所有的sheet都解析完之后，程序需要提示解析完成。在这个需求中，要实现主线程等待所有线程完成sheet的解析操作，最简单的做法是使用join\(\)方法，如代码清单8-1所示。

> 代码清单8-1　JoinCountDownLatchTest.java

```
public class JoinCountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        Thread parser1 = new Thread(new Runnable() {
            @Override
            public void run() {
            }
        });
        Thread parser2 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("parser2 finish");
            }
        });
        parser1.start();
        parser2.start();
        parser1.join();
        parser2.join();
        System.out.println("all parser finish");
    }
}
```

join用于让当前执行线程等待join线程执行结束。其实现原理是不停检查join线程是否存

活，如果join线程存活则让当前线程永远等待。其中，wait（0）表示永远等待下去，代码片段如

下。

