# 问题的根源

前面的双重检查锁定示例代码的第7行（instance=new Singleton\(\);）创建了一个对象。这一行代码可以分解为如下的3行伪代码。

* memory = allocate\(\);　　// 1：分配对象的内存空间
* ctorInstance\(memory\);　// 2：初始化对象
* instance = memory;　　// 3：设置instance指向刚分配的内存地址

上面3行伪代码中的2和3之间，可能会被重排序（在一些JIT编译器上，这种重排序是真实发生的，详情见参考文献1的“Out-of-order writes”部分）。2和3之间重排序之后的执行时序如下。

* memory = allocate\(\);　　// 1：分配对象的内存空间
* instance = memory;　　// 3：设置instance指向刚分配的内存地址
* // 注意，此时对象还没有被初始化！
* ctorInstance\(memory\);　// 2：初始化对象

根据《The Java Language Specification,Java SE 7 Edition》（后文简称为Java语言规范），所有线程在执行Java程序时必须要遵守intra-thread semantics。intra-thread semantics保证重排序不会改变单线程内的程序执行结果。换句话说，intra-thread semantics允许那些在单线程内，不会改变单线程程序执行结果的重排序。上面3行伪代码的2和3之间虽然被重排序了，但这个重排序并不会违反intra-thread semantics。这个重排序在没有改变单线程程序执行结果的前提下，可以提高程序的执行性能。

为了更好地理解intra-thread semantics，请看如图3-37所示的示意图（假设一个线程A在构造对象后，立即访问这个对象）。

如图3-37所示，只要保证2排在4的前面，即使2和3之间重排序了，也不会违反intra-threadsemantics。

![](/assets/import-3-37.png)

```
                                                                    图3-37　线程执行时序图
```

![](/assets/import-3-38.png)

```
                                                                      图3-38　多线程执行时序图
```

由于单线程内要遵守intra-thread semantics，从而能保证A线程的执行结果不会被改变。但是，当线程A和B按图3-38的时序执行时，B线程将看到一个还没有被初始化的对象。

回到本文的主题，DoubleCheckedLocking示例代码的第7行（instance=new Singleton\(\);）如果发生重排序，另一个并发执行的线程B就有可能在第4行判断instance不为null。线程B接下来将访问instance所引用的对象，但此时这个对象可能还没有被A线程初始化！表3-6是这个场景的具体执行时序。

表3-6![](/assets/import-3-6.png)

这里A2和A3虽然重排序了，但Java内存模型的intra-thread semantics将确保A2一定会排在

A4前面执行。因此，线程A的intra-thread semantics没有改变，但A2和A3的重排序，将导致线程

B在B1处判断出instance不为空，线程B接下来将访问instance引用的对象。此时，线程B将会访

问到一个还未初始化的对象。

在知晓了问题发生的根源之后，我们可以想出两个办法来实现线程安全的延迟初始化。

1）不允许2和3重排序。

2）允许2和3重排序，但不允许其他线程“看到”这个重排序。

后文介绍的两个解决方案，分别对应于上面这两点。



