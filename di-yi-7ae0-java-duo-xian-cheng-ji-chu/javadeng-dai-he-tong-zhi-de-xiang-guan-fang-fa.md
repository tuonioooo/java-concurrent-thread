# **Java线程等待和通知的相关方法**

一个线程修改了一个对象的值，而另一个线程感知到了变化，然后进行相应的操作，整个过程开始于一个线程，而最终执行又是另一个线程。前者是生产者，后者就是消费者，这种模式隔离了“做什么”（what）和“怎么做”（How），在功能层面上实现了解耦，体系结构上具备了良好的伸缩性，但是在Java语言中如何实现类似的功能呢？

简单的办法是让消费者线程不断地循环检查变量是否符合预期，如下面代码所示，在while循环中设置不满足的条件，如果条件满足则退出while循环，从而完成消费者的工作。

```
while (value != desire) {
    Thread.sleep(1000);
}
doSomething();
```

上面这段伪代码在条件不满足时就睡眠一段时间，这样做的目的是防止过快的“无效”尝试，这种方式看似能够解实现所需的功能，但是却存在如下问题。

1）难以确保及时性。在睡眠时，基本不消耗处理器资源，但是如果睡得过久，就不能及时发现条件已经变化，也就是及时性难以保证。

2）难以降低开销。如果降低睡眠的时间，比如休眠1毫秒，这样消费者能更加迅速地发现条件变化，但是却可能消耗更多的处理器资源，造成了无端的浪费。

以上两个问题，看似矛盾难以调和，但是Java通过内置的等待/通知机制能够很好地解决这个矛盾并实现所需的功能。等待/通知的相关方法是任意Java对象都具备的，因为这些方法被定义在所有对象的超类java.lang.Object上，方法和描述如表4-2所示。

表4-2　等待/通知的相关方法![](/assets/import-4-2.png)

等待/通知机制，是指一个线程A调用了对象O的wait\(\)方法进入等待状态，而另一个线程B调用了对象O的notify\(\)或者notifyAll\(\)方法，线程A收到通知后从对象O的wait\(\)方法返回，进而执行后续操作。上述两个线程通过对象O来完成交互，而对象上的wait\(\)和notify/notifyAll\(\)的关系就如同开关信号一样，用来完成等待方和通知方之间的交互工作。

在代码清单4-11所示的例子中，创建了两个线程——WaitThread和NotifyThread，前者检查flag值是否为false，如果符合要求，进行后续操作，否则在lock上等待，后者在睡眠了一段时间后对lock进行通知，示例如下所示。

代码清单4-11　WaitNotify.java

```
package com.boot.admin;

import java.text.SimpleDateFormat;
import java.util.concurrent.TimeUnit;

public class WaitNotify {
    static boolean flag = true;
    static Object lock = new Object();

    public static void main(String[] args) throws Exception {
        Thread waitThread = new Thread(new Wait(), "WaitThread");
        waitThread.start();
        TimeUnit.SECONDS.sleep(1);
        Thread notifyThread = new Thread(new Notify(), "NotifyThread");
        notifyThread.start();
    }

    static class Wait implements Runnable {
        public void run() {
// 加锁，拥有lock的Monitor
            synchronized (lock) {
// 当条件不满足时，继续wait，同时释放了lock的锁
                while (flag) {
                    try {
                        System.out.println(Thread.currentThread() + " flag is true. wait
                        @ " + new SimpleDateFormat(" HH:
                        mm:
                        ss ").format(new Date()));
                        lock.wait();
                    } catch (InterruptedException e) {
                    }
                }
// 条件满足时，完成工作
                System.out.println(Thread.currentThread() + " flag is false. running
                @ " + new SimpleDateFormat(" HH:
                mm:
                ss ").format(new Date()));
            }
        }
    }

    static class Notify implements Runnable {
        public void run() {
// 加锁，拥有lock的Monitor
            synchronized (lock) {
// 获取lock的锁，然后进行通知，通知时不会释放lock的锁，
// 直到当前线程释放了lock后，WaitThread才能从wait方法中返回
                System.out.println(Thread.currentThread() + " hold lock. notify @ " +
                        new SimpleDateFormat("HH:mm:ss").format(new Date()));
                lock.notifyAll();
                flag = false;
                SleepUtils.second(5);
            }
// 再次加锁
            synchronized (lock) {
                System.out.println(Thread.currentThread() + " hold lock again. sleep
                @ " + new SimpleDateFormat(" HH:
                mm:
                ss ").format(new Date()));
                SleepUtils.second(5);
            }
        }
    }
}
```

```
输出如下（输出内容可能不同，主要区别在时间上）。
Thread[WaitThread,5,main] flag is true. wait @ 22:23:03
Thread[NotifyThread,5,main] hold lock. notify @ 22:23:04
Thread[NotifyThread,5,main] hold lock again. sleep @ 22:23:09
Thread[WaitThread,5,main] flag is false. running @ 22:23:14
```

上述第3行和第4行输出的顺序可能会互换，而上述例子主要说明了调用wait\(\)、notify\(\)以及notifyAll\(\)时需要注意的细节，如下。

1）使用wait\(\)、notify\(\)和notifyAll\(\)时需要先对调用对象加锁。

2）调用wait\(\)方法后，线程状态由RUNNING变为WAITING，并将当前线程放置到对象的

等待队列。

3）notify\(\)或notifyAll\(\)方法调用后，等待线程依旧不会从wait\(\)返回，需要调用notify\(\)或notifAll\(\)的线程释放锁之后，等待线程才有机会从wait\(\)返回。

4）notify\(\)方法将等待队列中的一个等待线程从等待队列中移到同步队列中，而notifyAll\(\)方法则是将等待队列中所有的线程全部移到同步队列，被移动的线程状态由WAITING变为BLOCKED。

5）从wait\(\)方法返回的前提是获得了调用对象的锁。从上述细节中可以看到，等待/通知机制依托于同步机制，其目的就是确保等待线程从wait\(\)方法返回时能够感知到通知线程对变量做出的修改。

图4-3描述了上述示例的过程。

![](/assets/import-4-3.png)

图4-3　WaitNotify.java运行过程

在图4-3中，WaitThread首先获取了对象的锁，然后调用对象的wait\(\)方法，从而放弃了锁并进入了对象的等待队列WaitQueue中，进入等待状态。由于WaitThread释放了对象的锁，NotifyThread随后获取了对象的锁，并调用对象的notify\(\)方法，将WaitThread从WaitQueue移到SynchronizedQueue中，此时WaitThread的状态变为阻塞状态。NotifyThread释放了锁之后，WaitThread再次获取到锁并从wait\(\)方法返回继续执行。

**补充：**

**1.notify（）**

具体是怎么个意思呢？就是用来唤醒在此对象上等待的单个线程。说的有点太专业。打个比方，现在有十栋大房子，里面有很多被上了锁的房间，奇怪的是锁都是一样的，更不可思议的是，现在只有一把钥匙。而此时，张三用完钥匙后，就会发出归还钥匙的提醒，就相当于发出notify（）通知，但是要注意的是，此时钥匙还在张三手中，只不过，当张三发出notify（）通知后，JVM从那些整个沉睡的线程，唤醒一个。对应本例子，就是从其余的九栋大房子中唤醒一家，至于提醒谁来拿这把钥匙，就看JVM如何分配资源了。等到张三把钥匙归还后，那个被提醒的哪家，就可以使用该把钥匙来开房间门了。与此相对应的，还有一个notifyAll（）方法。这是什么意思呢，还是本例，张三嗓门大，这时吼了一嗓子，即notifyAll（），所有沉睡的线程全都被吵醒了，当张三归还钥匙后，他们就可以竞争了，注意，刚才是JVM自动分配，而此时是线程之间竞争，比如优先级等等条件，是有区别的。

![](/assets/import-notify.png)

**注意：**_**notify（）方法执行后，并不是立即释放锁，而是等到加锁的代码块执行完后，才开始释放的，相当于本例中，张三只是发出了归还的通知，但是钥匙还没有归还，需要等到代码块，执行完后，才可以归还。**_

**2.wait（）**

```
    这个方法又是怎么个意思呢？当执行到这个方法时，就把钥匙归还，开始睡觉了。Thread.sleep\(\)与Object.wait\(\)二者都可以暂停当前线程，释放CPU控制权，_**主要的区别在于Object.wait\(\)在释放CPU同时，释放了对象锁的控制**_。

    下面来看一个例子，加入我们要实现三个线程之间的同步操作，如何来实现呢？加入有三个线程分别用来输出A/B/C，如何能够三个线程之间顺序执行呢？这时候就需要采取线程之间同步的操作了，详见下面的代码。
```

```
package com.test;


public class MyThreadPrinter2 implements Runnable {

    private String name;
    private Object prev;
    private Object self;

    private MyThreadPrinter2(String name, Object prev, Object self) {
        this.name = name; // A B C
        this.prev = prev; // c a b
        this.self = self; // a b c
    }

    @Override
    public void run() {
        int count = 10;
        while (count > 0) {
            // 加锁，锁的钥匙是prev
            synchronized (prev) {
                // 一把锁，锁的钥匙是self变量
                synchronized (self) {
                    System.out.print(name);
                    count--;
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // 唤醒另一个线程，但是也需要把本线程执行完后，才可以释放锁
                    self.notify(); //a b c
                }

                try {
                    // 释放对象锁，本线程进入休眠状态，等待被唤醒
                    prev.wait();  //睡觉觉了，等待被叫醒吧   // c a b
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }
    }

    public static void main(String[] args) throws Exception {
        Object a = new Object();
        Object b = new Object();
        Object c = new Object();
        MyThreadPrinter2 pa = new MyThreadPrinter2("A", c, a);
        MyThreadPrinter2 pb = new MyThreadPrinter2("B", a, b);
        MyThreadPrinter2 pc = new MyThreadPrinter2("C", b, c);

        new Thread(pa).start();
        // 这样才可以保证按照顺序执行
         Thread.sleep(10);
        new Thread(pb).start();
         Thread.sleep(10);
        new Thread(pc).start();
         Thread.sleep(10);
    }
}
```

参考示例：[https://blog.csdn.net/luckyzhoustar/article/details/48179161](https://blog.csdn.net/luckyzhoustar/article/details/48179161)

