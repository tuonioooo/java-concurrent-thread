# ForkJoinPool的commonPool相关参数配置

## 概述

ForkJoinPool 主要用于实现“分而治之”的算法，特别是分治之后递归调用的函数，例如 quick sort 等。

  


ForkJoinPool 最适合的是计算密集型的任务，如果存在 I/O，线程间同步，sleep\(\) 等会造成线程长时间阻塞的情况时，最好配合使用 ManagedBlocker。

