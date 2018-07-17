# 如何让多线程下的类安全

1. 不要跨线程访问共享变量
2. 使共享变量是final类型的
3. 使共享变量只读
4. 将共享变量的操作加上同步

> ## 注意 {#注意}
>
> 1. 对于volatile声明的数值类型变量进行运算, 往往是不安全的\(volatile只能保证可见性,不能保证原子性\)
> 2. 使用普通同步容器\(Vector, Hashtable\)的迭代器, 需要外部锁来保证其原子性。原因是
>    **普通同步容器产生的迭代器是非线程安全**的。在并发编程中, 需要容器支持的时候, 优先考虑使用jdk并发容器
> 3. ConcurrentHashMap, CopyOnWriteArrayList并发容器的迭代器,以及全范围的size\(\), isEmpty\(\) 都表现出弱一致性。他们只能标示容器当时的一个数据状态，无法完整响应容器之后的变化和修改。


