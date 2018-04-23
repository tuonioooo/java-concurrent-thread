volatile写-读建立的happens-before关系

上面讲的是volatile变量自身的特性，对程序员来说，volatile对线程的内存可见性的影响

比volatile自身的特性更为重要，也更需要我们去关注。

从JSR-133开始（即从JDK5开始），volatile变量的写-读可以实现线程之间的通信。

从内存语义的角度来说，volatile的写-读与锁的释放-获取有相同的内存效果：volatile写和

锁的释放有相同的内存语义；volatile读与锁的获取有相同的内存语义。

请看下面使用volatile变量的示例代码。

