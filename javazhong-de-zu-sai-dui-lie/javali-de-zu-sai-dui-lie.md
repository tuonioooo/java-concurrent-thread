# Java里的阻塞队列

JDK 7提供了7个阻塞队列，如下。

·ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列。

·LinkedBlockingQueue：一个由链表结构组成的有界阻塞队列。

·PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列。

·DelayQueue：一个使用优先级队列实现的无界阻塞队列。

·SynchronousQueue：一个不存储元素的阻塞队列。

·LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。

·LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

1.ArrayBlockingQueue

ArrayBlockingQueue是一个用数组实现的有界阻塞队列。此队列按照先进先出（FIFO）的原

则对元素进行排序。

默认情况下不保证线程公平的访问队列，所谓公平访问队列是指阻塞的线程，可以按照

阻塞的先后顺序访问队列，即先阻塞线程先访问队列。非公平性是对先等待的线程是非公平

的，当队列可用时，阻塞的线程都可以争夺访问队列的资格，有可能先阻塞的线程最后才访问

队列。为了保证公平性，通常会降低吞吐量。我们可以使用以下代码创建一个公平的阻塞队

列。

`ArrayBlockingQueue fairQueue = new ArrayBlockingQueue(1000,true);`

访问者的公平性是使用可重入锁实现的，代码如下。

```
public ArrayBlockingQueue(int capacity, boolean fair) {
if (capacity <= 0)
throw new IllegalArgumentException();
this.items = new Object[capacity];
lock = new ReentrantLock(fair);
notEmpty = lock.newCondition();
notFull = lock.newCondition();
}
```

2.LinkedBlockingQueue

LinkedBlockingQueue是一个用链表实现的有界阻塞队列。此队列的默认和最大长度为

Integer.MAX\_VALUE。此队列按照先进先出的原则对元素进行排序。

3.PriorityBlockingQueue

PriorityBlockingQueue是一个支持优先级的无界阻塞队列。默认情况下元素采取自然顺序

升序排列。也可以自定义类实现compareTo\(\)方法来指定元素排序规则，或者初始化

PriorityBlockingQueue时，指定构造参数Comparator来对元素进行排序。需要注意的是不能保证

同优先级元素的顺序。

4.DelayQueue

DelayQueue是一个支持延时获取元素的无界阻塞队列。队列使用PriorityQueue来实现。队

列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素。

只有在延迟期满时才能从队列中提取元素。

DelayQueue非常有用，可以将DelayQueue运用在以下应用场景。

·缓存系统的设计：可以用DelayQueue保存缓存元素的有效期，使用一个线程循环查询

DelayQueue，一旦能从DelayQueue中获取元素时，表示缓存有效期到了。

·定时任务调度：使用DelayQueue保存当天将会执行的任务和执行时间，一旦从

DelayQueue中获取到任务就开始执行，比如TimerQueue就是使用DelayQueue实现的。

（1）如何实现Delayed接口

DelayQueue队列的元素必须实现Delayed接口。我们可以参考ScheduledThreadPoolExecutor

里ScheduledFutureTask类的实现，一共有三步。

第一步：在对象创建的时候，初始化基本数据。使用time记录当前对象延迟到什么时候可

以使用，使用sequenceNumber来标识元素在队列中的先后顺序。代码如下。

```
private static final AtomicLong sequencer = new AtomicLong(0);
    ScheduledFutureTask(Runnable r, V result, long ns, long period) {
    ScheduledFutureTask(Runnable r, V result, long ns, long period) {
        super(r, result);
        this.time = ns;
        this.period = period;
        this.sequenceNumber = sequencer.getAndIncrement();
}
```

第二步：实现getDelay方法，该方法返回当前元素还需要延时多长时间，单位是纳秒，代码

如下。

```
public long getDelay(TimeUnit unit) {
    return unit.convert(time - now(), TimeUnit.NANOSECONDS);
}
```

通过构造函数可以看出延迟时间参数ns的单位是纳秒，自己设计的时候最好使用纳秒，因

为实现getDelay\(\)方法时可以指定任意单位，一旦以秒或分作为单位，而延时时间又精确不到

纳秒就麻烦了。使用时请注意当time小于当前时间时，getDelay会返回负数。

第三步：实现compareTo方法来指定元素的顺序。例如，让延时时间最长的放在队列的末

尾。实现代码如下。

```
public int compareTo(Delayed other) {
    if (other == this)　　// compare zero ONLY if same object
    return 0;
    if (other instanceof ScheduledFutureTask) {
    ScheduledFutureTask<> x = (ScheduledFutureTask<>)other;
    long diff = time - x.time;
    if (diff < 0)
    return -1;
    else if (diff > 0)
    return 1;
    else if (sequenceNumber < x.sequenceNumber)
    return -1;
    else
    return 1;
    }
    long d = (getDelay(TimeUnit.NANOSECONDS) -other.getDelay(TimeUnit.NANOSECONDS));
    return (d == 0) 0 : ((d < 0) -1 : 1);
    }
```

（2）如何实现延时阻塞队列

延时阻塞队列的实现很简单，当消费者从队列里获取元素时，如果元素没有达到延时时

间，就阻塞当前线程。

```
long delay = first.getDelay(TimeUnit.NANOSECONDS);
if (delay <= 0)
return q.poll();
else if (leader != null)
available.await();
else {
Thread thisThread = Thread.currentThread();
leader = thisThread;
try {
available.awaitNanos(delay);
} finally {
if (leader == thisThread)
leader = null;
}
}
```

代码中的变量leader是一个等待获取队列头部元素的线程。如果leader不等于空，表示已

经有线程在等待获取队列的头元素。所以，使用await\(\)方法让当前线程等待信号。如果leader

等于空，则把当前线程设置成leader，并使用awaitNanos\(\)方法让当前线程等待接收信号或等

待delay时间。

5.SynchronousQueue

SynchronousQueue是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，

否则不能继续添加元素。

它支持公平访问队列。默认情况下线程采用非公平性策略访问队列。使用以下构造方法

可以创建公平性访问的SynchronousQueue，如果设置为true，则等待的线程会采用先进先出的

顺序访问队列。

```
public SynchronousQueue(boolean fair) {
    transferer = fair new TransferQueue() : new TransferStack();
}
```

SynchronousQueue可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费

者线程。队列本身并不存储任何元素，非常适合传递性场景。SynchronousQueue的吞吐量高于

LinkedBlockingQueue和ArrayBlockingQueue。

6.LinkedTransferQueue

LinkedTransferQueue是一个由链表结构组成的无界阻塞TransferQueue队列。相对于其他阻

塞队列，LinkedTransferQueue多了tryTransfer和transfer方法。

（1）transfer方法

如果当前有消费者正在等待接收元素（消费者使用take\(\)方法或带时间限制的poll\(\)方法

时），transfer方法可以把生产者传入的元素立刻transfer（传输）给消费者。如果没有消费者在等

待接收元素，transfer方法会将元素存放在队列的tail节点，并等到该元素被消费者消费了才返

回。transfer方法的关键代码如下。

Node pred = tryAppend\(s, haveData\);

return awaitMatch\(s, pred, e, \(how == TIMED\), nanos\);

第一行代码是试图把存放当前元素的s节点作为tail节点。第二行代码是让CPU自旋等待

消费者消费元素。因为自旋会消耗CPU，所以自旋一定的次数后使用Thread.yield\(\)方法来暂停

当前正在执行的线程，并执行其他线程。

（2）tryTransfer方法

tryTransfer方法是用来试探生产者传入的元素是否能直接传给消费者。如果没有消费者等

待接收元素，则返回false。和transfer方法的区别是tryTransfer方法无论消费者是否接收，方法

立即返回，而transfer方法是必须等到消费者消费了才返回。

对于带有时间限制的tryTransfer（E e，long timeout，TimeUnit unit）方法，试图把生产者传入

的元素直接传给消费者，但是如果没有消费者消费该元素则等待指定的时间再返回，如果超

时还没消费元素，则返回false，如果在超时时间内消费了元素，则返回true。

7.LinkedBlockingDeque

LinkedBlockingDeque是一个由链表结构组成的双向阻塞队列。所谓双向队列指的是可以

从队列的两端插入和移出元素。双向队列因为多了一个操作队列的入口，在多线程同时入队

时，也就减少了一半的竞争。相比其他的阻塞队列，LinkedBlockingDeque多了addFirst、

addLast、offerFirst、offerLast、peekFirst和peekLast等方法，以First单词结尾的方法，表示插入、

获取（peek）或移除双端队列的第一个元素。以Last单词结尾的方法，表示插入、获取或移除双

端队列的最后一个元素。另外，插入方法add等同于addLast，移除方法remove等效于

removeFirst。但是take方法却等同于takeFirst，不知道是不是JDK的bug，使用时还是用带有First

和Last后缀的方法更清楚。

在初始化LinkedBlockingDeque时可以设置容量防止其过度膨胀。另外，双向阻塞队列可以

运用在“工作窃取”模式中。

