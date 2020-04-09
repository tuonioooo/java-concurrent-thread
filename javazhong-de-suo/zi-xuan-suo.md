# 自旋锁

**自旋锁（Spin lock）**

自旋锁与互斥锁有点类似，只是自旋锁不会引起调用者睡眠，如果自旋锁已经被别的执行单元保持，调用者就一直循环在那里看是否该自旋锁的保持者已经释放了锁，“自旋锁”的作用

是为了解决某项资源的互斥使用。因为自旋锁不会引起调用者睡眠，所以自旋锁的效率远高于互斥锁。

**自旋锁的不足之处**：

自旋锁一直占用着CPU，他在未获得锁的情况下，一直运行（自旋），所以占用着CPU，如果不能在很短的时间内获得锁，这无疑会使CPU效率降低。

在用自旋锁时有可能造成死锁，当递归调用时有可能造成死锁，调用有些其他函数也可能造成死锁，如 copy\_to\_user\(\)、copy\_from\_user\(\)、kmalloc\(\)等。

因此我们要慎重使用自旋锁，自旋锁只有在内核可抢占式或SMP的情况下才真正需要，在单CPU且不可抢占式的内核下，自旋锁的操作为空操作。**自旋锁适用于锁使用者保持锁时间比较短的情况下，若持锁时间太长，性能降低**

> 互斥锁：
>
> 用于保护临界区，确保同一时间只有一个线程访问数据。对共享资源的访问，先对互斥量进行加锁，如果互斥量已经上锁，调用线程会阻塞，直到互斥量被解锁。在完成了对共享资源的访问后，要对互斥量进行解锁。

**Java实现自旋锁的场景：**

依据自旋锁的特点：

* 轻量级操作，无需挂起线程
* 特别吃CPU，如果线程在临界区的操作比较耗时或者线程对临界区的竞争很激烈，那还是老老实实用普通的锁

```text
public class SpinLock implements Lock {

    /**
     * 锁持有线程, null表示锁未被任何线程持有
     */
    private final AtomicReference<Thread> owner = new AtomicReference<Thread>();

    /**
     * owner持有锁次数
     */
    private int holdCount;

    @Override
    public void lock() {
        final AtomicReference<Thread> owner = this.owner;

        final Thread current = Thread.currentThread();
        if (owner.get() == current) { // 当前线程已持有锁, 增加持有计数即可
            ++holdCount;
            return;
        }

        while (!owner.compareAndSet(null, current)) {
        }

        holdCount = 1;
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        final AtomicReference<Thread> owner = this.owner;

        final Thread current = Thread.currentThread();
        if (owner.get() == current) {
            ++holdCount;
            return;
        }

        while (!owner.compareAndSet(null, current)) {
            // 响应中断
            if (current.isInterrupted()) {
                current.interrupt(); // 重设中断标志
                throw new InterruptedException();
            }
        }

        holdCount = 1;
    }

    @Override
    public boolean tryLock() {
        boolean locked =  owner.compareAndSet(null, Thread.currentThread());
        if (locked) {
            holdCount = 1;
        }
        return locked;
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        final AtomicReference<Thread> owner = this.owner;
        final Thread current = Thread.currentThread();
        if (owner.get() == current) {
            ++holdCount;
            return true;
        }

        final long start = System.nanoTime();
        final long timeoutNanos = unit.toNanos(time);
        while (!owner.compareAndSet(null, current)) {
            // 响应中断
            if (current.isInterrupted()) {
                current.interrupt();
                throw new InterruptedException();
            }
            // 判断是否超时
            long elapsed = System.nanoTime() - start;
            if (elapsed >= timeoutNanos) {
                return false;
            }
        }

        holdCount = 1;
        return true;
    }

    @Override
    public void unlock() {
        final AtomicReference<Thread> owner = this.owner;
        final Thread current = Thread.currentThread();

        if (owner.get() != current) {
            throw new IllegalMonitorStateException();
        }
        // 持有多少次, 就必须释放多少次
        if (--holdCount == 0) {
            owner.set(null);
        }
    }

    @Override
    public Condition newCondition() {
        throw new UnsupportedOperationException();
    }
}
```

