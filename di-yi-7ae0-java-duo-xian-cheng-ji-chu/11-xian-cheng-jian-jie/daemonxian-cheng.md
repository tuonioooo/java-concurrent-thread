# Daemon线程（守护线程）

Daemon线程是一种支持型线程，因为它主要被用作程序中后台调度以及支持性工作。这  
意味着，当一个Java虚拟机中不存在非Daemon线程的时候，Java虚拟机将会退出。可以通过调  
用Thread.setDaemon\(true\)将线程设置为Daemon线程。

**注意：**　Daemon属性需要在启动线程之前设置，不能在启动线程之后设置。  
Daemon线程被用作完成支持性工作，但是在Java虚拟机退出时Daemon线程中的finally块  
并不一定会执行，示例如代码清单1所示。

清单1

```
public class Daemon {
public static void main(String[] args) {
    Thread thread = new Thread(new DaemonRunner(), "DaemonRunner");
    thread.setDaemon(true);
    thread.start();
}
static class DaemonRunner implements Runnable {
    @Override
    public void run() {
        try {
            SleepUtils.second(10);
        } finally {
            System.out.println("DaemonThread finally run.");
        }
        }
    }
}
```

运行Daemon程序，可以看到在终端或者命令提示符上没有任何输出。main线程（非  
Daemon线程）在启动了线程DaemonRunner之后随着main方法执行完毕而终止，而此时Java虚拟  
机中已经没有非Daemon线程，虚拟机需要退出。Java虚拟机中的所有Daemon线程都需要立即  
终止，因此DaemonRunner立即终止，但是DaemonRunner中的finally块并没有执行。

> 将线程转换为守护线程可以通过调用Thread对象的setDaemon\(true\)方法来实现。在使用守护线程时需要注意一下几点：
>
> \(1\) thread.setDaemon\(true\)必须在thread.start\(\)之前设置，否则会跑出一个IllegalThreadStateException异常。你不能把正在运行的常规线程设置为守护线程。 
>
> \(2\) 在Daemon线程中产生的新线程也是Daemon的。
>
> \(3\) 守护线程应该永远不去访问固有资源，如文件、数据库，因为它会在任何时候甚至在一个操作的中间发生中断。

```
import java.util.concurrent.TimeUnit;

/**

 *  守护线程

 */

public class Daemons {

 

    /**

     * @param args

     * @throws InterruptedException

     */

    public static void main(String[] args) throws InterruptedException {

 

        Thread d = new Thread(new Daemon());

        d.setDaemon(true); //必须在启动线程前调用

        d.start();

        System.out.println("d.isDaemon() = " + d.isDaemon() + ".");

        TimeUnit.SECONDS.sleep(1);

    }

}

 

 

 

class DaemonSpawn implements Runnable {

    public void run() {

        while (true) {

            Thread.yield();

        }

    }

}

 



class Daemon implements Runnable {

    private Thread[] t = new Thread[10];

    public void run() {

        for (int i=0; i<t.length; i++) {

            t[i] = new Thread(new DaemonSpawn());

            t[i].start();

            System.out.println("DaemonSpawn " + i + " started.");

        }

        for (int i=0; i<t.length; i++) {

            System.out.println("t[" + i + "].isDaemon() = " +

                    t[i].isDaemon() + ".");

        }

        while (true) {

            Thread.yield();

        }

    }

}
```

运行结果：

d.isDaemon\(\) = true.

DaemonSpawn 0 started.

DaemonSpawn 1 started.

DaemonSpawn 2 started.

DaemonSpawn 3 started.

DaemonSpawn 4 started.

DaemonSpawn 5 started.

DaemonSpawn 6 started.

DaemonSpawn 7 started.

DaemonSpawn 8 started.

DaemonSpawn 9 started.

t\[0\].isDaemon\(\) = true.

t\[1\].isDaemon\(\) = true.

t\[2\].isDaemon\(\) = true.

t\[3\].isDaemon\(\) = true.

t\[4\].isDaemon\(\) = true.

t\[5\].isDaemon\(\) = true.

t\[6\].isDaemon\(\) = true.

t\[7\].isDaemon\(\) = true.

t\[8\].isDaemon\(\) = true.

t\[9\].isDaemon\(\) = true.

以上结果说明了守护线程中产生的新线程也是守护线程。

如果将mian函数中的TimeUnit._SECONDS_.sleep\(1\);注释掉，运行结果如下：

d.isDaemon\(\) = true.

DaemonSpawn 0 started.

DaemonSpawn 1 started.

DaemonSpawn 2 started.

DaemonSpawn 3 started.

DaemonSpawn 4 started.

DaemonSpawn 5 started.

DaemonSpawn 6 started.

DaemonSpawn 7 started.

DaemonSpawn 8 started.

DaemonSpawn 9 started.

以上结果说明了如果用户线程已经全部退出运行了，只剩下守护线程存在了，虚拟机也就退出了。下面的例子也说明了这个问题。

```
import java.util.concurrent.TimeUnit;

/**

 * Finally shoud be always run ?

 */

public class DaemonsDontRunFinally {

    /**

     * @param args

     */

    public static void main(String[] args) {

        Thread t = new Thread(new ADaemon());

        t.setDaemon(true);

        t.start();

    }

}



class ADaemon implements Runnable {

 public void run() {

        try {

            System.out.println("start ADaemon...");

            TimeUnit.SECONDS.sleep(1);

        } catch (InterruptedException e) {

            System.out.println("Exiting via InterruptedException");

        } finally {

            System.out.println("This shoud be always run ?");

        }

    }

}
```

运行结果：

start ADaemon...

如果将main函数中的t.setDaemon\(**true**\);注释掉，运行结果如下：

start ADaemon...

This shoud be always run ?



