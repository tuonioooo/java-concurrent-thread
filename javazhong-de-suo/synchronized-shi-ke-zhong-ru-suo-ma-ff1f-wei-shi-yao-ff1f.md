# synchronized 是可重入锁吗？为什么？

## 什么是可重入锁？

关于什么是[可重入锁](https://zh.wikipedia.org/wiki/%E5%8F%AF%E9%87%8D%E5%85%A5)，我们先来看一段维基百科的定义。

> 若一个程序或子程序可以“在任意时刻被中断然后操作系统调度执行另外一段代码，这段代码又调用了该子程序不会出错”，则称其为可重入（reentrant或re-entrant）的。即当该子程序正在运行时，执行线程可以再次进入并执行它，仍然获得符合设计时预期的结果。与多线程并发执行的线程安全不同，可重入强调对单个线程执行时重新进入同一个子程序仍然是安全的。

通俗来说：当线程请求一个由其它线程持有的对象锁时，该线程会阻塞，而当线程请求由自己持有的对象锁时，如果该锁是重入锁，请求就会成功，否则阻塞。

## synchronized 是可重入锁吗？

我先给大家一个结论：synchronized 是可重入锁！

假设我们现在不知道它是不是一个可重入锁，那我们就应该想方设法来验证它是不是可重入锁？怎么验证呢？看下面的代码

```
public class Xttblog extends SuperXttblog {
    public static void main(String[] args) {
        Xttblog child = new Xttblog();
        child.doSomething();
    }
 
    public synchronized void doSomething() {
        System.out.println("child.doSomething()" + Thread.currentThread().getName());
        doAnotherThing(); // 调用自己类中其他的synchronized方法
    }
 
    private synchronized void doAnotherThing() {
        super.doSomething(); // 调用父类的synchronized方法
        System.out.println("child.doAnotherThing()" + Thread.currentThread().getName());
    }
}
 
class SuperXttblog {
    public synchronized void doSomething() {
        System.out.println("father.doSomething()" + Thread.currentThread().getName());
    }
}
```

上面的代码也不是随便写的，我是根据维基百科的定义写出这段代码来验证它。现在运行一下上面的代码，我们看一下结果：

```
child.doSomething()Thread-5492
father.doSomething()Thread-5492
child.doAnotherThing()Thread-5492
```

现在可以验证出 synchronized 是可重入锁了吧！因为这些方法输出了相同的线程名称，表明即使递归使用synchronized也没有发生死锁，证明其是可重入的。

还看不懂？那我就再解释下！

这里的对象锁只有一个，就是 child 对象的锁，当执行 child.doSomething 时，该线程获得 child 对象的锁，在 doSomething 方法内执行 doAnotherThing 时再次请求child对象的锁，因为synchronized 是重入锁，所以可以得到该锁，继续在 doAnotherThing 里执行父类的 doSomething 方法时第三次请求 child 对象的锁，同样可得到。如果不是重入锁的话，那这后面这两次请求锁将会被一直阻塞，从而导致死锁。

所以在 java 内部，同一线程在调用自己类中其他 synchronized 方法/块或调用父类的 synchronized 方法/块都不会阻碍该线程的执行。就是说同一线程对同一个对象锁是可重入的，而且同一个线程可以获取同一把锁多次，也就是可以多次重入。因为java线程是基于“每线程（per-thread）”，而不是基于“每调用（per-invocation）”的（java中线程获得对象锁的操作是以线程为粒度的，per-invocation 互斥体获得对象锁的操作是以每调用作为粒度的）。

## 可重入锁的实现原理？

看到这里，你终于明白了 synchronized 是一个可重入锁。但是面试官要再问你，可重入锁的原理是什么？

对不起，你又卡壳了。

那么我现在先给你说一下，可重入锁的原理。具体我们后面再写 ReentrantLock 的时候来验证或看它源码。

重入锁实现可重入性原理或机制是：每一个锁关联一个线程持有者和计数器，当计数器为 0 时表示该锁没有被任何线程持有，那么任何线程都可能获得该锁而调用相应的方法；当某一线程请求成功后，JVM会记下锁的持有线程，并且将计数器置为 1；此时其它线程请求该锁，则必须等待；而该持有锁的线程如果再次请求这个锁，就可以再次拿到这个锁，同时计数器会递增；当线程退出同步代码块时，计数器会递减，如果计数器为 0，则释放该锁。

