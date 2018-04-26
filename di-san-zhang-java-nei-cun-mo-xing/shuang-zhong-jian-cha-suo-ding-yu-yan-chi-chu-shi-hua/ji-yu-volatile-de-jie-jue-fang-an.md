基于volatile的解决方案

对于前面的基于双重检查锁定来实现延迟初始化的方案（指DoubleCheckedLocking示例代

码），只需要做一点小的修改（把instance声明为volatile型），就可以实现线程安全的延迟初始

化。请看下面的示例代码。

```
public class SafeDoubleCheckedLocking {

    private volatile static Instance instance;

    public static Instance getInstance() {

        if (instance == null) {

            synchronized (SafeDoubleCheckedLocking.class) {

            if (instance == null)

            instance = new Instance(); // instance为volatile，现在没问题了

            }

        }

        return instance;

    }

}
```

注意

这个解决方案需要JDK 5或更高版本（因为从JDK 5开始使用新的JSR-133内存模型规范，这个规范增强了volatile的语义）。

当声明对象的引用为volatile后，3.8.2节中的3行伪代码中的2和3之间的重排序，在多线程环境中将会被禁止。上面示例代码将按如下的时序执行，如图3-39所示。

图3-39![](/assets/import-3-39.png)这个方案本质上是通过禁止图3-39中的2和3之间的重排序，来保证线程安全的延迟初始

化。

