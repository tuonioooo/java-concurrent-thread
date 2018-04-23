锁的释放-获取建立的happens-before关系

锁是Java并发编程中最重要的同步机制。锁除了让临界区互斥执行外，还可以让释放锁的

线程向获取同一个锁的线程发送消息。

下面是锁释放-获取的示例代码。

---

```
class MonitorExample {
    int a = 0;
    public synchronized void writer() {　　　　// 1
        a++;　　　　　　　　　　// 2
    }　　　　　　　　　　　　// 3
    public synchronized void reader() {　　　// 4
        int i = a;　　　　　　　　// 5
……
    }　　　　　　　　　　　　// 6
}
```

---

假设线程A执行writer\(\)方法，随后线程B执行reader\(\)方法。根据happens-before规则，这个

过程包含的happens-before关系可以分为3类。

1）根据程序次序规则，1 happens-before 2,2 happens-before 3;4 happens-before 5,5 happens-

before 6。

2）根据监视器锁规则，3 happens-before 4。

3）根据happens-before的传递性，2 happens-before 5。

上述happens-before关系的图形化表现形式如图1所示。

图1

![](/assets/import-3-5-1-1.png)

在图1中，每一个箭头链接的两个节点，代表了一个happens-before关系。黑色箭头表示

程序顺序规则；橙色箭头表示监视器锁规则；蓝色箭头表示组合这些规则后提供的happens-

before保证。

图3-24表示在线程A释放了锁之后，随后线程B获取同一个锁。在上图中，2 happens-before

5。因此，线程A在释放锁之前所有可见的共享变量，在线程B获取同一个锁之后，将立刻变得

对B线程可见。





