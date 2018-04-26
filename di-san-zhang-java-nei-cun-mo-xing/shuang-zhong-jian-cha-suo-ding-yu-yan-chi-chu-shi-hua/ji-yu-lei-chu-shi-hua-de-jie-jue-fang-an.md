基于类初始化的解决方案

JVM在类的初始化阶段（即在Class被加载后，且被线程使用之前），会执行类的初始化。在执行类的初始化期间，JVM会去获取一个锁。这个锁可以同步多个线程对同一个类的初始化。

基于这个特性，可以实现另一种线程安全的延迟初始化方案（这个方案被称之为Initialization On Demand Holder idiom）。

```
public class InstanceFactory {
    private static class InstanceHolder {
        public static Instance instance = new Instance();
    }
    public static Instance getInstance() {
        return InstanceHolder.instance ;　　// 这里将导致InstanceHolder类被初始化
    }
}
```

假设两个线程并发执行getInstance\(\)方法，下面是执行的示意图，如图3-40所示。

图3-40　两个线程并发执行的示意图![](/assets/import-3-40.png)

这个方案的实质是：允许3.8.2节中的3行伪代码中的2和3重排序，但不允许非构造线程（这里指线程B）“看到”这个重排序。

初始化一个类，包括执行这个类的静态初始化和初始化在这个类中声明的静态字段。

根据Java语言规范，在首次发生下列任意一种情况时，一个类或接口类型T将被立即初始化。

1）T是一个类，而且一个T类型的实例被创建。

2）T是一个类，且T中声明的一个静态方法被调用。

3）T中声明的一个静态字段被赋值。

4）T中声明的一个静态字段被使用，而且这个字段不是一个常量字段。

5）T是一个顶级类（Top Level Class，见Java语言规范的§7.6），而且一个断言语句嵌套在T内部被执行。

在InstanceFactory示例代码中，首次执行getInstance\(\)方法的线程将导致InstanceHolder类被初始化（符合情况4）。

由于Java语言是多线程的，多个线程可能在同一时间尝试去初始化同一个类或接口（比如这里多个线程可能在同一时刻调用getInstance\(\)方法来初始化InstanceHolder类）。因此，在Java中初始化一个类或者接口时，需要做细致的同步处理。

Java语言规范规定，对于每一个类或接口C，都有一个唯一的初始化锁LC与之对应。从C到LC的映射，由JVM的具体实现去自由实现。JVM在类初始化期间会获取这个初始化锁，并且每个线程至少获取一次锁来确保这个类已经被初始化过了（事实上，Java语言规范允许JVM的具体实现在这里做一些优化，见后文的说明）。

对于类或接口的初始化，Java语言规范制定了精巧而复杂的类初始化处理过程。Java初始化一个类或接口的处理过程如下（这里对类初始化处理过程的说明，省略了与本文无关的部分；同时为了更好的说明类初始化过程中的同步处理机制，笔者人为的把类初始化的处理过程分为了5个阶段）。

第1阶段：通过在Class对象上同步（即获取Class对象的初始化锁），来控制类或接口的初始化。这个获取锁的线程会一直等待，直到当前线程能够获取到这个初始化锁。假设Class对象当前还没有被初始化（初始化状态state，此时被标记为state=noInitializa-tion），且有两个线程A和B试图同时初始化这个Class对象。图3-41是对应的示意图。

图3-41

![](/assets/import-3-41.png)

表3-7是这个示意图的说明。

                                                                  表3-7　类初始化——第1阶段的执行时序表![](/assets/import-3-7.png)

第2阶段：线程A执行类的初始化，同时线程B在初始化锁对应的condition上等待。

表3-8是这个示意图的说明。

表3-8　类初始化——第2阶段的执行时序表![](/assets/import-3-8.png)![](/assets/import-3-42.png)

图3-42　类初始化——第2阶段

第3阶段：线程A设置state=initialized，然后唤醒在condition中等待的所有线程。

![](/assets/import-3-43.png)

```
                                                             图3-43　类初始化——第3阶段
```



