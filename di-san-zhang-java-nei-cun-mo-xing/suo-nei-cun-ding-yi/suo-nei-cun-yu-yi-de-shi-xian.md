# 锁内存语义的实现

本文将借助ReentrantLock的源代码，来分析锁内存语义的具体实现机制。

请看下面的示例代码：

---

```
class ReentrantLockExample {
    int a = 0;
    ReentrantLock lock = new ReentrantLock();
    public void writer() {
        lock.lock();　　　　// 获取锁
        try {
            a++;
        } f　　inally {
            lock.unlock();　　// 释放锁
        }
    }
    public void reader () {
        lock.lock();　　　　// 获取锁
        try {
            int i = a;
……
        } f　　inally {
            lock.unlock();　// 释放锁
        }
    }
}
```

---

在ReentrantLock中，调用lock\(\)方法获取锁；调用unlock\(\)方法释放锁。

ReentrantLock的实现依赖于Java同步器框架AbstractQueuedSynchronizer（本文简称之为

AQS）。AQS使用一个整型的volatile变量（命名为state）来维护同步状态，马上我们会看到，这

个volatile变量是ReentrantLock内存语义实现的关键。

图3-5-3-1是ReentrantLock的类图（仅画出与本文相关的部分）。

图3-5-3-1

![](/assets/import-3-5-3-1.png)

ReentrantLock分为公平锁和非公平锁，我们首先分析公平锁。

使用公平锁时，加锁方法lock\(\)调用轨迹如下。

1）ReentrantLock:lock\(\)。

2）FairSync:lock\(\)。

3）AbstractQueuedSynchronizer:acquire\(int arg\)。

4）ReentrantLock:tryAcquire\(int acquires\)。

在第4步真正开始加锁，下面是该方法的源代码：

---

```
protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();　　　　// 获取锁的开始，首先读volatile变量state
        if (c == 0) {
            if (isFirst(current) &&
                    compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)　
            throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
```

---

从上面源代码中我们可以看出，加锁方法首先读volatile变量state。

在使用公平锁时，解锁方法unlock\(\)调用轨迹如下。

1）ReentrantLock:unlock\(\)。

2）AbstractQueuedSynchronizer:release\(int arg\)。

3）Sync:tryRelease\(int releases\)。

在第3步真正开始释放锁，下面是该方法的源代码：

---

```
protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);　　　　　// 释放锁的最后，写volatile变量state
        return free;
    }
```

---

从上面的源代码可以看出，在释放锁的最后写volatile变量state。

公平锁在释放锁的最后写volatile变量state，在获取锁时首先读这个volatile变量。根据volatile的happens-before规则，释放锁的线程在写volatile变量之前可见的共享变量，在获取锁的线程读取同一个volatile变量后将立即变得对获取锁的线程可见。

现在我们来分析非公平锁的内存语义的实现。非公平锁的释放和公平锁完全一样，所以这里仅仅分析非公平锁的获取。使用非公平锁时，加锁方法lock\(\)调用轨迹如下。

1）ReentrantLock:lock\(\)。

2）NonfairSync:lock\(\)。

3）AbstractQueuedSynchronizer:compareAndSetState\(int expect,int update\)。

---

```
protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```

---

该方法以原子操作的方式更新state变量，本文把Java的compareAndSet\(\)方法调用简称为CAS。

JDK文档对该方法的说明如下：如果当前状态值等于预期值，则以原子方式将同步状态设置为给定的更新值。此操作具有volatile读和写的内存语义。这里我们分别从编译器和处理器的角度来分析，CAS如何同时具有volatile读和volatile写的内存语义。前文我们提到过，编译器不会对volatile读与volatile读后面的任意内存操作重排序；编译器不会对volatile写与volatile写前面的任意内存操作重排序。组合这两个条件，意味着为了同时实现volatile读和volatile写的内存语义，编译器不能对CAS与CAS前面和后面的任意内存操

作重排序。下面我们来分析在常见的intel X86处理器中，CAS是如何同时具有volatile读和volatile写的内存语义的。

下面是sun.misc.Unsafe类的compareAndSwapInt\(\)方法的源代码。

```
public final native boolean compareAndSwapInt(Object o, long offset,int expected,int x);
```

可以看到，这是一个本地方法调用。这个本地方法在openjdk中依次调用的c++代码为：

unsafe.cpp，atomic.cpp和atomic\_windows\_x86.inline.hpp。这个本地方法的最终实现在openjdk的如下位置：openjdk-7-fcs-src-b147-27\_jun\_2011\openjdk\hotspot\src\os\_cpu\windows\_x86\vm\atomic\_windows\_x86.inline.hpp（对应于Windows操作系统，X86处理器）。下面是对应于intel X86处理器的源代码的片段。

```
inline jint Atomic::cmpxchg (jint exchange_value, volatile jint* dest,
jint compare_value) {
// alternative for InterlockedCompareExchange
int mp = os::is_MP();
__asm {
mov edx, dest
mov ecx, exchange_value
mov eax, compare_value
LOCK_IF_MP(mp)
cmpxchg dword ptr [edx], ecx
}
}
```

如上面源代码所示，程序会根据当前处理器的类型来决定是否为cmpxchg指令添加lock前缀。如果程序是在多处理器上运行，就为cmpxchg指令加上lock前缀（Lock Cmpxchg）。反之，如果程序是在单处理器上运行，就省略lock前缀（单处理器自身会维护单处理器内的顺序一致性，不需要lock前缀提供的内存屏障效果）。

intel的手册对lock前缀的说明如下。

1）确保对内存的读-改-写操作原子执行。在Pentium及Pentium之前的处理器中，带有lock前缀的指令在执行期间会锁住总线，使得其他处理器暂时无法通过总线访问内存。很显然，这会带来昂贵的开销。从Pentium 4、Intel Xeon及P6处理器开始，Intel使用缓存锁定（Cache Locking）来保证指令执行的原子性。缓存锁定将大大降低lock前缀指令的执行开销。

2）禁止该指令，与之前和之后的读和写指令重排序。

3）把写缓冲区中的所有数据刷新到内存中。上面的第2点和第3点所具有的内存屏障效果，足以同时实现volatile读和volatile写的内存

语义。经过上面的分析，现在我们终于能明白为什么JDK文档说CAS同时具有volatile读和volatile写的内存语义了。

现在对公平锁和非公平锁的内存语义做个总结：

* 公平锁和非公平锁释放时，最后都要写一个volatile变量state。
* 公平锁获取时，首先会去读volatile变量。
* 非公平锁获取时，首先会用CAS更新volatile变量，这个操作同时具有volatile读和volatile写的内存语义。
* 从本文对ReentrantLock的分析可以看出，锁释放-获取的内存语义的实现至少有下面两种方式。

1）利用volatile变量的写-读所具有的内存语义。

2）利用CAS所附带的volatile读和volatile写的内存语义。

