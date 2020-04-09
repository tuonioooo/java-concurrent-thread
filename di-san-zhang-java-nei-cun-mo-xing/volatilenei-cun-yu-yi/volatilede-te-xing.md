# volatile的特性

volatile的特性

理解volatile特性的一个好方法是把对volatile变量的单个读/写，看成是使用同一个锁对这些单个读/写操作做了同步。下面通过具体的示例来说明，示例代码如下：

```text
class VolatileFeaturesExample {
    volatile long vl = 0L; // 使用volatile声明64位的long型变量
    public void set(long l) {
        vl = l; // 单个volatile变量的写
    }
    public void getAndIncrement () {
        vl++; // 复合（多个）volatile变量的读/写
    }
    public long get() {
        return vl; // 单个volatile变量的读
    }
}
```

假设有多个线程分别调用上面程序的3个方法，这个程序在语义上和下面程序等价。

```text
class VolatileFeaturesExample {
    long vl = 0L; // 64位的long型普通变量
    public synchronized void set(long l) { // 对单个的普通变量的写用同一个锁同步
        vl = l;
    }
    public void getAndIncrement () { // 普通方法调用
        long temp = get(); // 调用已同步的读方法
        temp += 1L; // 普通写操作
        set(temp); // 调用已同步的写方法
    }
    public synchronized long get() { // 对单个的普通变量的读用同一个锁同步
        return vl;
    }
}
```

如上面示例程序所示，一个volatile变量的单个读/写操作，与一个普通变量的读/写操作都是使用同一个锁来同步，它们之间的执行效果相同。

锁的happens-before规则保证释放锁和获取锁的两个线程之间的内存可见性，这意味着对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。锁的语义决定了临界区代码的执行具有原子性。

这意味着，即使是64位的long型和double型变量，只要它是volatile变量，对该变量的读/写就具有原子性。如果是多个volatile操作或类似于volatile++这种复合操作，这些操作整体上不具有原子性。简而言之，volatile变量自身具有下列特性。可见性。对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。

·原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。

