基于类初始化的解决方案

JVM在类的初始化阶段（即在Class被加载后，且被线程使用之前），会执行类的初始化。在

执行类的初始化期间，JVM会去获取一个锁。这个锁可以同步多个线程对同一个类的初始化。

基于这个特性，可以实现另一种线程安全的延迟初始化方案（这个方案被称之为

Initialization On Demand Holder idiom）。

public class InstanceFactory {

private static class InstanceHolder {

public static Instance instance = new Instance\(\);

}

public static Instance getInstance\(\) {

return InstanceHolder.instance ;　　// 这里将导致InstanceHolder类被初始化

}

}

假设两个线程并发执行getInstance\(\)方法，下面是执行的示意图，如图3-40所示。

