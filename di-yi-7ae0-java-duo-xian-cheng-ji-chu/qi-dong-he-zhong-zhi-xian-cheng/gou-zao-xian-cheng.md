# 构造线程

在运行线程之前首先要构造一个线程对象，线程对象在构造的时候需要提供线程所需要的属性，如线程所属的线程组、线程优先级、是否是Daemon线程等信息。代码清单1所示的代码摘自java.lang.Thread中对线程进行初始化的部分。

清单1

```text
private void init(ThreadGroup g, Runnable target, String name, long stackSize, AccessControlContext acc) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }
        // 当前线程就是该线程的父线程
        Thread parent = currentThread();
        this.group = g;
        // 将daemon、priority属性设置为父线程的对应属性
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        this.name = name.toCharArray();
        this.target = target;
        setPriority(priority);
        // 将父线程的InheritableThreadLocal复制过来
        if (parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals = ThreadLocal.createInheritedMap(parent.
                    inheritableThreadLocals);
        // 分配一个线程ID
        tid = nextThreadID();
    }
```

在上述过程中，一个新构造的线程对象是由其parent线程来进行空间分配的，而child线程  
继承了parent是否为Daemon、优先级和加载资源的contextClassLoader以及可继承的  
ThreadLocal，同时还会分配一个唯一的ID来标识这个child线程。至此，一个能够运行的线程对  
象就初始化好了，在堆内存中等待着运行。

