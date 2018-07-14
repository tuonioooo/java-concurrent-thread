# 什么是线程

现代操作系统在运行一个程序时，会为其创建一个进程。例如，启动一个Java程序，操作系统就会创建一个Java进程。现代操作系统调度的最小单元是线程，也叫轻量级进程（LightWeight Process），在一个进程里可以创建多个线程，这些线程都拥有各自的计数器、堆栈和局部变量等属性，并且能够访问共享的内存变量。处理器在这些线程上高速切换，让使用者感觉到这些线程在同时执行。一个Java程序从main\(\)方法开始执行，然后按照既定的代码逻辑执行，看似没有其他线程参与，但实际上Java程序天生就是多线程程序，因为执行main\(\)方法的是一个名称为main的线程。下面使用JMX来查看一个普通的Java程序包含哪些线程，如代码清单所示。

```
package com.ise.api.thread;

import java.lang.management.ManagementFactory;
import java.lang.management.ThreadInfo;
import java.lang.management.ThreadMXBean;

public class MultiThread {
    public static void main(String[] args) {
        // 获取Java线程管理MXBean
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        // 不需要获取同步的monitor和synchronizer信息，仅获取线程和线程堆栈信息
        ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
        // 遍历线程信息，仅打印线程ID和线程名称信息
        for (ThreadInfo threadInfo : threadInfos) {
            System.out.println("[" + threadInfo.getThreadId() + "] " + threadInfo.getThreadName());
        }
    }
}
```

输出结果：

```
[6] Monitor Ctrl-Break  
[5] Attach Listener   
[4] Signal Dispatcher     // 分发处理发送给JVM信号的线程
[3] Finalizer            // 调用对象finalize方法的线程
[2] Reference Handler   // 清除Reference的线程
[1] main                // main线程，用户程序入口
```

> 线程：有时被称为轻量进程\(Lightweight Process，LWP），是程序执行流的最小单元



