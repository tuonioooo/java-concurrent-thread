JSR-133为什么要增强volatile的内存语义

在JSR-133之前的旧Java内存模型中，虽然不允许volatile变量之间重排序，但旧的Java内

存模型允许volatile变量与普通变量重排序。在旧的内存模型中，VolatileExample示例程序可能

被重排序成下列时序来执行，如图1所示。

图1

![](/assets/import-3-4-5-1.png)

