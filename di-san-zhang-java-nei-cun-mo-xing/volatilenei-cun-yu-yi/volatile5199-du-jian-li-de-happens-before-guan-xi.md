# volatile写-读建立的happens-before关系

上面讲的是volatile变量自身的特性，对程序员来说，volatile对线程的内存可见性的影响比volatile自身的特性更为重要，也更需要我们去关注。

从JSR-133开始（即从JDK5开始），volatile变量的写-读可以实现线程之间的通信。从内存语义的角度来说，volatile的写-读与锁的释放-获取有相同的内存效果：volatile写和锁的释放有相同的内存语义；volatile读与锁的获取有相同的内存语义。

请看下面使用volatile变量的示例代码：

---

```
class VolatileExample {
    int a = 0;
    volatile boolean flag = false;
    public void writer() {
        a = 1;　　　　　// 1
        flag = true;　　　// 2
    }
    public void reader() {
        if (flag) {　　　　// 3
            int i = a;　　// 4
……
        }
    }
}
```

---

假设线程A执行writer\(\)方法之后，线程B执行reader\(\)方法。

根据happens-before规则，这个过程建立的happens-before关系可以分为3类：

1）根据程序次序规则，1 happens-before 2;3 happens-before 4。

2）根据volatile规则，2 happens-before 3。

3）根据happens-before的传递性规则，1 happens-before 4。

上述happens-before关系的图形化表现形式如下图

![](/assets/import-3-4-2-1.png)

在上图中，每一个箭头链接的两个节点，代表了一个happens-before关系。黑色箭头表示程序顺序规则；橙色箭头表示volatile规则；蓝色箭头表示组合这些规则后提供的happens-before保证。

这里A线程写一个volatile变量后，B线程读同一个volatile变量。A线程在写volatile变量之前所有可见的共享变量，在B线程读同一个volatile变量后，将立即变得对B线程可见。

> 注意　本文统一用粗实线标识组合后产生的happens-before关系。



