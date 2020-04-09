# volatile关键字

## volatile关键字

#### volatile 关键字（上）

在 JDK1.2 之前，Java 的内存模型实现总是从主存（即共享内存）读取变量，是不需要进行特别的注意的。而随着 JVM 的成熟和优化，现在在多线程环境下 volatile 关键字的使用变得非常重要。

在当前的 Java 内存模型下，线程可以把变量保存在本地内存（比如机器的寄存器）中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成数据的不一致。

要解决这个问题，就需要把变量声明为 volatile，这就指示 JVM，这个变量是不稳定的，每次使用它都到主存中进行读取。一般说来，多任务环境下，各任务间共享的变量都应该加 volatile 修饰符。

Volatile 修饰的成员变量在每次被线程访问时，都强迫从共享内存中重读该成员变量的值。而且，当成员变量发生变化时，强迫线程将变化值回写到共享内存。这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。

Java 语言规范中指出：为了获得最佳速度，允许线程保存共享成员变量的私有拷贝，而且只当线程进入或者离开同步代码块时才将私有拷贝与共享内存中的原始值进行比较。

这样当多个线程同时与某个对象交互时，就必须注意到要让线程及时的得到共享成员变量的变化。而 volatile 关键字就是提示 JVM：对于这个成员变量，不能保存它的私有拷贝，而应直接与共享成员变量交互。

volatile 是一种稍弱的同步机制，在访问 volatile 变量时不会执行加锁操作，也就不会执行线程阻塞，因此 volatilei 变量是一种比 synchronized 关键字更轻量级的同步机制。

使用建议：在两个或者更多的线程需要访问的成员变量上使用 volatile。当要访问的变量已在 synchronized 代码块中，或者为常量时，没必要使用 volatile。

由于使用 volatile 屏蔽掉了 JVM 中必要的代码优化，所以在效率上比较低，因此一定在必要时才使用此关键字。

### 示例程序 <a id="3e49d1b2350ddf99486da97f59acbacf"></a>

下面给出一段代码，通过其运行结果来说明使用关键字 volatile 产生的差异，但实际上遇到了意料之外的问题：

```text
public class Volatile extends Object implements Runnable {  
    //value变量没有被标记为volatile  
    private int value;    
    //missedIt变量被标记为volatile  
    private volatile boolean missedIt;  
    //creationTime不需要声明为volatile，因为代码执行中它没有发生变化  
    private long creationTime;   

    public Volatile() {  
        value = 10;  
        missedIt = false;  
        //获取当前时间，亦即调用Volatile构造函数时的时间  
        creationTime = System.currentTimeMillis();  
    }  

    public void run() {  
        print("entering run()");  

        //循环检查value的值是否不同  
        while ( value 
<
 20 ) {  
            //如果missedIt的值被修改为true，则通过break退出循环  
            if  ( missedIt ) {  
                //进入同步代码块前，将value的值赋给currValue  
                int currValue = value;  
                //在一个任意对象上执行同步语句，目的是为了让该线程在进入和离开同步代码块时，  
                //将该线程中的所有变量的私有拷贝与共享内存中的原始值进行比较，  
                //从而发现没有用volatile标记的变量所发生的变化  
                Object lock = new Object();  
                synchronized ( lock ) {  
                    //不做任何事  
                }  
                //离开同步代码块后，将此时value的值赋给valueAfterSync  
                int valueAfterSync = value;  
                print("in run() - see value=" + currValue +", but rumor has it that it changed!");  
                print("in run() - valueAfterSync=" + valueAfterSync);  
                break;   
            }  
        }  
        print("leaving run()");  
    }  

    public void workMethod() throws InterruptedException {  
        print("entering workMethod()");  
        print("in workMethod() - about to sleep for 2 seconds");  
        Thread.sleep(2000);  
        //仅在此改变value的值  
        value = 50;  
        print("in workMethod() - just set value=" + value);  
        print("in workMethod() - about to sleep for 5 seconds");  
        Thread.sleep(5000);  
        //仅在此改变missedIt的值  
        missedIt = true;  
        print("in workMethod() - just set missedIt=" + missedIt);  
        print("in workMethod() - about to sleep for 3 seconds");  
        Thread.sleep(3000);  
        print("leaving workMethod()");  
    }  

/* 
*该方法的功能是在要打印的msg信息前打印出程序执行到此所化去的时间，以及打印msg的代码所在的线程 
*/  
    private void print(String msg) {  
        //使用java.text包的功能，可以简化这个方法，但是这里没有利用这一点  
        long interval = System.currentTimeMillis() - creationTime;  
        String tmpStr = "    " + ( interval / 1000.0 ) + "000";       
        int pos = tmpStr.indexOf(".");  
        String secStr = tmpStr.substring(pos - 2, pos + 4);  
        String nameStr = "        " + Thread.currentThread().getName();  
        nameStr = nameStr.substring(nameStr.length() - 8, nameStr.length());      
        System.out.println(secStr + " " + nameStr + ": " + msg);  
    }  

    public static void main(String[] args) {  
        try {  
            //通过该构造函数可以获取实时时钟的当前时间  
            Volatile vol = new Volatile();  

            //稍停100ms，以让实时时钟稍稍超前获取时间，使print（）中创建的消息打印的时间值大于0  
            Thread.sleep(100);    

            Thread t = new Thread(vol);  
            t.start();  

            //休眠100ms，让刚刚启动的线程有时间运行  
            Thread.sleep(100);    
            //workMethod方法在main线程中运行  
            vol.workMethod();  
        } catch ( InterruptedException x ) {  
            System.err.println("one of the sleeps was interrupted");  
        }  
    }  
}
```

按照以上的理论来分析，由于 value 变量不是 volatile 的，因此它在 main 线程中的改变不会被 Thread-0 线程（在 main 线程中新开启的线程）马上看到，因此 Thread-0 线程中的 while 循环不会直接退出，它会继续判断 missedIt 的值，由于 missedIt 是 volatile 的，当 main 线程中改变了 missedIt 时，Thread-0 线程会立即看到该变化，那么 if 语句中的代码便得到了执行的机会，由于此时 Thread-0 依然没有看到 value 值的变化，因此，currValue 的值为 10，继续向下执行，进入同步代码块，因为进入前后要将该线程内的变量值与共享内存中的原始值对比，进行校准，因此离开同步代码块后，Thread-0 便会察觉到 value 的值变为了 50，那么后面的 valueAfterSync 的值便为 50，最后从 break 跳出循环，结束 Thread-0 线程。

#### 意料之外的问题

但实际的执行结果如下：

![](http://wiki.jikexueyuan.com/project/java-concurrency/images/unexpectedresults.jpg)

从结果中可以看出，Thread-0 线程并没有进入 while 循环，说明 Thread-0 线程在 value 的值发生变化后，missedIt 的值发生变化前，便察觉到了 value 值的变化，从而退出了 while 循环。这与理论上的分析不符，我便尝试注释掉 value 值发生改变与 missedIt 值发生改变之间的线程休眠代码 Thread.sleep\(5000\)，以确保Thread-0 线程在 missedIt 的值发生改变前，没有时间察觉到 value 值的变化。但执行的结果与上面大同小异（可能有一两行顺序不同，但依然不会打印出 if 语句中的输出信息）。

#### 问题分析

在 JDK1.7~JDK1.3 之间的版本上输出结果与上面基本大同小异，只有在 JDK1.2 上才得到了预期的结果，即Thread-0 线程中的 while 循环是从 if 语句中退出的，这说明 Thread-0 线程没有及时察觉到 value 值的变化。

这里需要注意：volatile 是针对 JIT 带来的优化，因此 JDK1.2 以前的版本基本不用考虑，另外，在 JDK1.3.1 开始，开始运用 HotSpot 虚拟机，用来代替 JIT。因此，是不是 HotSpot 的问题呢？这里需要再补充一点：

JIT 或 HotSpot 编译器在 server 模式和 client 模式编译不同，server 模式为了使线程运行更快，如果其中一个线程更改了变量 boolean flag 的值，那么另外一个线程会看不到，因为另外一个线程为了使得运行更快所以从寄存器或者本地 cache 中取值，而不是从内存中取值，那么使用 volatile 后，就告诉不论是什么线程，被volatile修饰的变量都要从内存中取值。

对于非 volatile 修饰的变量，尽管 jvm 的优化，会导致变量的可见性问题，但这种可见性的问题也只是在短时间内高并发的情况下发生，CPU 执行时会很快刷新 Cache，一般的情况下很难出现，而且出现这种问题是不可预测的，与 jvm, 机器配置环境等都有关。

## volatile 关键字（下）

在《Volatile 关键字（上）》一文中遗留了一个问题，就是 volatile 只修饰了 missedIt 变量，而没修饰value 变量，但是在线程读取 value 的值的时候，也读到的是最新的数据。

下面讲解问题出现的原因。

首先明确一点：假如有两个线程分别读写 volatile 变量时，线程 A 写入了某 volatile 变量，线程 B 在读取该 volatile 变量时，便能看到线程 A 对该 volatile 变量的写入操作，关键在这里，它不仅会看到对该 volatile 变量的写入操作，A 线程在写 volatile 变量之前所有可见的共享变量，在 B 线程读同一个 volatile 变量后，都将立即变得对 B 线程可见。

回过头来看文章中出现的问题，由于程序中 volatile 变量 missedIt 的写入操作在 value 变量写入操作之后，而且根据 volatile 规则，又不能重排序，因此，在线程 B 读取由线程 A 改变后的 missedIt 之后，它之前的 value 变量在线程 A 的改变也对线程 B 变得可见了。

我们颠倒一下 value=50 和 missedIt=true 这两行代码试下，即 missedIt=true 在前，value=50 在后，这样便会得到我们想要的结果：value 值的改变不会被看到。

这应该是 JDK1.2 之后对 volatile 规则做了一些修订的结果。

修改后的代码如下：

```text
public class Volatile extends Object implements Runnable {  
    //value变量没有被标记为volatile  
    private int value;    
    //missedIt变量被标记为volatile  
    private volatile boolean missedIt;  
    //creationTime不需要声明为volatile，因为代码执行中它没有发生变化  
    private long creationTime;   

    public Volatile() {  
        value = 10;  
        missedIt = false;  
        //获取当前时间，亦即调用Volatile构造函数时的时间  
        creationTime = System.currentTimeMillis();  
    }  

    public void run() {  
        print("entering run()");  

        //循环检查value的值是否不同  
        while ( value 
<
 20 ) {  
            //如果missedIt的值被修改为true，则通过break退出循环  
            if  ( missedIt ) {  
                //进入同步代码块前，将value的值赋给currValue  
                int currValue = value;  
                //在一个任意对象上执行同步语句，目的是为了让该线程在进入和离开同步代码块时，  
                //将该线程中的所有变量的私有拷贝与共享内存中的原始值进行比较，  
                //从而发现没有用volatile标记的变量所发生的变化  
                Object lock = new Object();  
                synchronized ( lock ) {  
                    //不做任何事  
                }  
                //离开同步代码块后，将此时value的值赋给valueAfterSync  
                int valueAfterSync = value;  
                print("in run() - see value=" + currValue +", but rumor has it that it changed!");  
                print("in run() - valueAfterSync=" + valueAfterSync);  
                break;   
            }  
        }  
        print("leaving run()");  
    }  

    public void workMethod() throws InterruptedException {  
        print("entering workMethod()");  
        print("in workMethod() - about to sleep for 2 seconds");  
        Thread.sleep(2000);  
        //仅在此改变value的值  
        missedIt = true;  
//      value = 50;  
        print("in workMethod() - just set value=" + value);  
        print("in workMethod() - about to sleep for 5 seconds");  
        Thread.sleep(5000);  
        //仅在此改变missedIt的值  
//      missedIt = true;  
        value = 50;  
        print("in workMethod() - just set missedIt=" + missedIt);  
        print("in workMethod() - about to sleep for 3 seconds");  
        Thread.sleep(3000);  
        print("leaving workMethod()");  
    }  

/* 
*该方法的功能是在要打印的msg信息前打印出程序执行到此所化去的时间，以及打印msg的代码所在的线程 
*/  
    private void print(String msg) {  
        //使用java.text包的功能，可以简化这个方法，但是这里没有利用这一点  
        long interval = System.currentTimeMillis() - creationTime;  
        String tmpStr = "    " + ( interval / 1000.0 ) + "000";       
        int pos = tmpStr.indexOf(".");  
        String secStr = tmpStr.substring(pos - 2, pos + 4);  
        String nameStr = "        " + Thread.currentThread().getName();  
        nameStr = nameStr.substring(nameStr.length() - 8, nameStr.length());      
        System.out.println(secStr + " " + nameStr + ": " + msg);  
    }  

    public static void main(String[] args) {  
        try {  
            //通过该构造函数可以获取实时时钟的当前时间  
            Volatile vol = new Volatile();  

            //稍停100ms，以让实时时钟稍稍超前获取时间，使print（）中创建的消息打印的时间值大于0  
            Thread.sleep(100);    

            Thread t = new Thread(vol);  
            t.start();  

            //休眠100ms，让刚刚启动的线程有时间运行  
            Thread.sleep(100);    
            //workMethod方法在main线程中运行  
            vol.workMethod();  
        } catch ( InterruptedException x ) {  
            System.err.println("one of the sleeps was interrupted");  
        }  
    }  
}
```

运行结果如下：

![](http://wiki.jikexueyuan.com/project/java-concurrency/images/modifyresults.jpg)

很明显，这其实并不符合使用 volatile 的第二个条件：**该变量要没有包含在具有其他变量的不变式中**。因此，在这里使用 volatile 是不安全的。

附上一篇讲述 volatile 关键字正确使用的很好的文章：[http://www.ibm.com/developerworks/cn/java/j-jtp06197.html](http://www.ibm.com/developerworks/cn/java/j-jtp06197.html)

