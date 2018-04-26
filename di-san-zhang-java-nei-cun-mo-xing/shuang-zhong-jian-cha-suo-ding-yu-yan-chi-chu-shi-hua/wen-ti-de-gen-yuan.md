问题的根源

前面的双重检查锁定示例代码的第7行（instance=new Singleton\(\);）创建了一个对象。这一

行代码可以分解为如下的3行伪代码。

memory = allocate\(\);　　// 1：分配对象的内存空间

ctorInstance\(memory\);　// 2：初始化对象

instance = memory;　　// 3：设置instance指向刚分配的内存地址

上面3行伪代码中的2和3之间，可能会被重排序（在一些JIT编译器上，这种重排序是真实

发生的，详情见参考文献1的“Out-of-order writes”部分）。2和3之间重排序之后的执行时序如

下。

memory = allocate\(\);　　// 1：分配对象的内存空间

instance = memory;　　// 3：设置instance指向刚分配的内存地址

// 注意，此时对象还没有被初始化！

ctorInstance\(memory\);　// 2：初始化对象

根据《The Java Language Specification,Java SE 7 Edition》（后文简称为Java语言规范），所有

线程在执行Java程序时必须要遵守intra-thread semantics。intra-thread semantics保证重排序不会

改变单线程内的程序执行结果。换句话说，intra-thread semantics允许那些在单线程内，不会改

变单线程程序执行结果的重排序。上面3行伪代码的2和3之间虽然被重排序了，但这个重排序

并不会违反intra-thread semantics。这个重排序在没有改变单线程程序执行结果的前提下，可以

提高程序的执行性能。
