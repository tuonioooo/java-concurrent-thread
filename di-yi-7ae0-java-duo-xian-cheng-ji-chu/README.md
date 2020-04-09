# Java多线程基础

## 目录

* [线程简介](11-xian-cheng-jian-jie/)
* [启动和终止线程](qi-dong-he-zhong-zhi-xian-cheng/)
* [多线程实现方式](1duo-xian-cheng-shi-xian-fang-shi.md)
* [多线程环境下，局部变量和全局变量都会共享吗？](duo-xian-cheng-huan-jing-xia-ff0c-ju-bu-bian-liang-he-quan-ju-bian-liang-du-hui-gong-xiang-ma-ff1f.md)
* [Java线程间的协助和通信](javaxian-cheng-zhi-jian-de-tong-xin/)
* [实战应用](shi-zhan-ying-yong/)

Java从诞生开始就明智地选择了内置对多线程的支持，这使得Java语言相比同一时期的其他语言具有明显的优势。线程作为操作系统调度的最小单元，多个线程能够同时执行，这将显著提升程序性能，在多核环境中表现得更加明显。但是，过多地创建线程和对线程的不当管理也容易造成问题。本章将着重介绍Java并发编程的基础知识，从启动一个线程到线程间不同的通信方式，最后通过简单的线程池示例以及应用（简单的Web服务器）来串联本章所介绍的内容。

