# 锁内存定义

## 目录

* [锁的释放-获取建立的happens-before关系](suo-de-shi-653e-huo-qu-jian-li-de-happens-before-guan-xi.md)
* [锁的释放和获取的内存语义](suo-de-shi-fang-he-huo-qu-de-nei-cun-yu-yi.md)
* [锁内存语义的实现](suo-nei-cun-yu-yi-de-shi-xian.md)
* [concurrent包的实现](concurrentbao-de-shi-xian.md)

众所周知，锁可以让临界区互斥执行。这里将介绍锁的另一个同样重要，但常常被忽视的功能：锁的内存语义。

