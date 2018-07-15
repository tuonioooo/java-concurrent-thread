# Thread.join\(\)的使用

如果一个线程A执行了thread.join\(\)语句，其含义是：当前线程A等待thread线程终止之后才

从thread.join\(\)返回。线程Thread除了提供join\(\)方法之外，还提供了join\(long millis\)和join\(long

millis,int nanos\)两个具备超时特性的方法。这两个超时方法表示，如果线程thread在给定的超时

时间里没有终止，那么将会从该超时方法中返回。

在代码清单4-13所示的例子中，创建了10个线程，编号0~9，每个线程调用前一个线程的

join\(\)方法，也就是线程0结束了，线程1才能从join\(\)方法中返回，而线程0需要等待main线程结

束。

> 代码清单4-13　Join.java

```
import java.util.concurrent.TimeUnit;

public class Join {
    public static void main(String[] args) throws Exception {
        Thread previous = Thread.currentThread();
        for (int i = 0; i < 10; i++) {
// 每个线程拥有前一个线程的引用，需要等待前一个线程终止，才能从等待中返回
            Thread thread = new Thread(new Domino(previous), String.valueOf(i));
            thread.start();
            previous = thread;
        }
        TimeUnit.SECONDS.sleep(5);
        System.out.println(Thread.currentThread().getName() + " terminate.");
    }
    static class Domino implements Runnable {
        private Thread thread;
        public Domino(Thread thread) {
            this.thread = thread;
        }
        public void run() {
            try {
                thread.join();
            } catch (InterruptedException e) {
            }
            System.out.println(Thread.currentThread().getName() + " terminate.");
        }
    }
}
```

```
输出如下。
main terminate.
0 terminate.
1 terminate.
2 terminate.
3 terminate.
4 terminate.
5 terminate.
6 terminate.
7 terminate.
8 terminate.
9 terminate.
```

从上述输出可以看到，每个线程终止的前提是前驱线程的终止，每个线程等待前驱线程

终止后，才从join\(\)方法返回，这里涉及了等待/通知机制（等待前驱线程结束，接收前驱线程结

束通知）。

代码清单4-14是JDK中Thread.join\(\)方法的源码（进行了部分调整）。

> 代码清单4-14　Thread.java

```
// 加锁当前线程对象
public final synchronized void join() throws InterruptedException {
// 条件不满足，继续等待
while (isAlive()) {
    wait(0);
}
// 条件符合，方法返回
}
```

当线程终止时，会调用线程自身的notifyAll\(\)方法，会通知所有等待在该线程对象上的线

程。可以看到join\(\)方法的逻辑结构与4.3.3节中描述的等待/通知经典范式一致，即加锁、循环

和处理逻辑3个步骤。

