# 过期的suspend\(\)、resume\(\)和stop\(\)

大家对于CD机肯定不会陌生，如果把它播放音乐比作一个线程的运作，那么对音乐播放做出的暂停、恢复和停止操作对应在线程Thread的API就是suspend\(\)、resume\(\)和stop\(\)。在代码清单1所示的例子中，创建了一个线程PrintThread，它以1秒的频率进行打印，而主线程对其进行暂停、恢复和停止操作。

清单1：

```
package com.ise.api.thread;

import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.concurrent.TimeUnit;

public class Deprecated {
    public static void main(String[] args) throws Exception {
        DateFormat format = new SimpleDateFormat("HH:mm:ss");
        Thread printThread = new Thread(new Runner(), "PrintThread");
        printThread.setDaemon(true);
        printThread.start();
        TimeUnit.SECONDS.sleep(3);
        // 将PrintThread进行暂停，输出内容工作停止
        printThread.suspend();
        System.out.println("main suspend PrintThread at " + format.format(new Date()));
        TimeUnit.SECONDS.sleep(3);
        // 将PrintThread进行恢复，输出内容继续
        printThread.resume();
        System.out.println("main resume PrintThread at " + format.format(new Date()));
        TimeUnit.SECONDS.sleep(3);
        // 将PrintThread进行终止，输出内容停止
        printThread.stop();
        System.out.println("main stop PrintThread at " + format.format(new Date()));
        TimeUnit.SECONDS.sleep(3);
    }

    static class Runner implements Runnable {
        @Override
        public void run() {
            DateFormat format = new SimpleDateFormat("HH:mm:ss");
            while (true) {
                System.out.println(Thread.currentThread().getName() + " Run at " +
                        format.format(new Date()));
                SleepUtils.second(1);
            }
        }
    }
}
```

输出如下（输出内容中的时间与示例执行的具体时间相关）。

```
PrintThread Run at 17:34:36
PrintThread Run at 17:34:37
PrintThread Run at 17:34:38
main suspend PrintThread at 17:34:39
main resume PrintThread at 17:34:42
PrintThread Run at 17:34:42
PrintThread Run at 17:34:43
PrintThread Run at 17:34:44
main stop PrintThread at 17:34:45
```

在执行过程中，PrintThread运行了3秒，随后被暂停，3秒后恢复，最后经过3秒被终止。  
通过示例的输出可以看到，suspend\(\)、resume\(\)和stop\(\)方法完成了线程的暂停、恢复和终  
止工作，而且非常“人性化”。但是这些API是过期的，也就是不建议使用的。  
不建议使用的原因主要有：以suspend\(\)方法为例，在调用后，线程不会释放已经占有的资  
源（比如锁），而是占有着资源进入睡眠状态，这样容易引发死锁问题。同样，stop\(\)方法在终结  
一个线程时不会保证线程的资源正常释放，通常是没有给予线程完成资源释放工作的机会，  
因此会导致程序可能工作在不确定状态下。

**注意：**　正因为suspend\(\)、resume\(\)和stop\(\)方法带来的副作用，这些方法才被标注为不建  
议使用的过期方法，而暂停和恢复操作可以用后面提到的等待/通知机制来替代。

