# 阻塞队列的实现原理

如果队列是空的，消费者会一直等待，当生产者添加元素时，消费者是如何知道当前队列

有元素的呢？如果让你来设计阻塞队列你会如何设计，如何让生产者和消费者进行高效率的

通信呢？让我们先来看看JDK是如何实现的。

使用通知模式实现。所谓通知模式，就是当生产者往满的队列里添加元素时会阻塞住生

产者，当消费者消费了一个队列中的元素后，会通知生产者当前队列可用。通过查看JDK源码

发现ArrayBlockingQueue使用了Condition来实现，代码如下。

```text
private final Condition notFull;
private final Condition notEmpty;
public ArrayBlockingQueue(int capacity, boolean fair) {
// 省略其他代码
notEmpty = lock.newCondition();
notFull = lock.newCondition();
}
public void put(E e) throws InterruptedException {
checkNotNull(e);
final ReentrantLock lock = this.lock;
lock.lockInterruptibly();
try {
while (count == items.length)
notFull.await();
insert(e);
} finally {
lock.unlock();
}
}
public E take() throws InterruptedException {
final ReentrantLock lock = this.lock;
lock.lockInterruptibly();
try {
while (count == 0)
notEmpty.await();
return extract();
} finally {
lock.unlock();
}
}
private void insert(E x) {
items[putIndex] = x;
putIndex = inc(putIndex);
++count;
notEmpty.signal();
}
当往队列里插入一个元素时，如果队列不可用，那么阻塞生产者主要通过
LockSupport.park（this）来实现。
public final void await() throws InterruptedException {
if (Thread.interrupted())
throw new InterruptedException();
Node node = addConditionWaiter();
int savedState = fullyRelease(node);
int interruptMode = 0;
while (!isOnSyncQueue(node)) {
LockSupport.park(this);
if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
break;
}
if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
interruptMode = REINTERRUPT;
if (node.nextWaiter != null) // clean up if cancelled
unlinkCancelledWaiters();
if (interruptMode != 0)
reportInterruptAfterWait(interruptMode);
}
```

继续进入源码，发现调用setBlocker先保存一下将要阻塞的线程，然后调用unsafe.park阻塞

当前线程。

```text
public static void park(Object blocker) {
Thread t = Thread.currentThread();
setBlocker(t, blocker);
unsafe.park(false, 0L);
setBlocker(t, null);
}
```

unsafe.park是个native方法，代码如下。

public native void park\(boolean isAbsolute, long time\);

park这个方法会阻塞当前线程，只有以下4种情况中的一种发生时，该方法才会返回。

·与park对应的unpark执行或已经执行时。“已经执行”是指unpark先执行，然后再执行park

的情况。

·线程被中断时。

·等待完time参数指定的毫秒数时。

·异常现象发生时，这个异常现象没有任何原因。

继续看一下JVM是如何实现park方法：park在不同的操作系统中使用不同的方式实现，在

Linux下使用的是系统方法pthread\_cond\_wait实现。实现代码在JVM源码路径

src/os/linux/vm/os\_linux.cpp里的os::PlatformEvent::park方法，代码如下。

```text
void os::PlatformEvent::park() {
int v ;
for (;;) {
v = _Event ;
if (Atomic::cmpxchg (v-1, &_Event, v) == v) break ;
}
guarantee (v >= 0, "invariant") ;
if (v == 0) {
// Do this the hard way by blocking ...
int status = pthread_mutex_lock(_mutex);
assert_status(status == 0, status, "mutex_lock");
guarantee (_nParked == 0, "invariant") ;
++ _nParked ;
while (_Event < 0) {
status = pthread_cond_wait(_cond, _mutex);
// for some reason, under 2.7 lwp_cond_wait() may return ETIME ...
// Treat this the same as if the wait was interrupted
if (status == ETIME) { status = EINTR; }
assert_status(status == 0 || status == EINTR, status, "cond_wait");
}
-- _nParked ;
// In theory we could move the ST of 0 into _Event past the unlock(),
// but then we'd need a MEMBAR after the ST.
_Event = 0 ;
status = pthread_mutex_unlock(_mutex);
assert_status(status == 0, status, "mutex_unlock");
}
guarantee (_Event >= 0, "invariant") ;
}
}
```

pthread\_cond\_wait是一个多线程的条件变量函数，cond是condition的缩写，字面意思可以

理解为线程在等待一个条件发生，这个条件是一个全局变量。这个方法接收两个参数：一个共

享变量\_cond，一个互斥量\_mutex。而unpark方法在Linux下是使用pthread\_cond\_signal实现的。

park方法在Windows下则是使用WaitForSingleObject实现的。想知道pthread\_cond\_wait是如何实

现的，可以参考glibc-2.5的nptl/sysdeps/pthread/pthread\_cond\_wait.c。

当线程被阻塞队列阻塞时，线程会进入WAITING（parking）状态。我们可以使用jstack dump

阻塞的生产者线程看到这点，如下。

```text
"main" prio=5 tid=0x00007fc83c000000 nid=0x10164e000 waiting on condition [0x000000010164d000]
java.lang.Thread.State: WAITING (parking)
at sun.misc.Unsafe.park(Native Method)
parking to wait for <0x0000000140559fe8> (a java.util.concurrent.locks.
AbstractQueuedSynchronizer$ConditionObject)
at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.
await(AbstractQueuedSynchronizer.java:2043)
at java.util.concurrent.ArrayBlockingQueue.put(ArrayBlockingQueue.java:324)
at blockingqueue.ArrayBlockingQueueTest.main(ArrayBlockingQueueTest.java
```

