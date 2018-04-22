# 线程的状态

Java线程在运行的生命周期中可能处于表1所示的6种不同的状态，在给定的一个时刻，线程只能处于其中的一个状态。

表1![](/assets/import.png)

下面我们使用jstack工具（可以选择打开终端，键入jstack或者到JDK安装目录的bin目录下执行命令），尝试查看示例代码运行时的线程信息，更加深入地理解线程状态，示例如代码清单2所示。

清单2

```
package com.ise.api.thread;

public class ThreadState {
    public static void main(String[] args) {
        new Thread(new TimeWaiting(), "TimeWaitingThread").start();
        new Thread(new Waiting(), "WaitingThread").start();
        // 使用两个Blocked线程，一个获取锁成功，另一个被阻塞
        new Thread(new Blocked(), "BlockedThread-1").start();
        new Thread(new Blocked(), "BlockedThread-2").start();
    }

    // 该线程不断地进行睡眠
    static class TimeWaiting implements Runnable {
        @Override
        public void run() {
            while (true) {
                SleepUtils.second(100);
            }
        }
    }

    // 该线程在Waiting.class实例上等待
    static class Waiting implements Runnable {
        @Override
        public void run() {
            while (true) {
                synchronized (Waiting.class) {
                    try {
                        Waiting.class.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    // 该线程在Blocked.class实例上加锁后，不会释放该锁
    static class Blocked implements Runnable {
        public void run() {
            synchronized (Blocked.class) {
                while (true) {
                    SleepUtils.second(100);
                }
            }
        }
    }
}
```

```
package com.ise.api.thread;

import java.util.concurrent.TimeUnit;

public class SleepUtils {
    public static final void second(long seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (InterruptedException e) {
        }
    }
}
```

运行该示例，打开终端或者命令提示符，键入“jps”，输出如下。

```
611
935 Jps
929 ThreadState
270
```



